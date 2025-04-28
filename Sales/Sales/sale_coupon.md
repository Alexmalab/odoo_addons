# Odoo Module: sale_coupon

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Sale Coupon",
    'summary': "Use discount coupons in sales orders",
    'description': """Integrate coupon mechanism in sales orders.""",
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['sale_management'],
    'data': [
        'security/ir.model.access.csv',
        'wizard/sale_coupon_apply_code_views.xml',
        'wizard/sale_coupon_generate_views.xml',
        'views/sale_order_views.xml',
        'views/sale_coupon_views.xml',
        'views/sale_coupon_program_views.xml',
        'views/res_config_settings_views.xml',
        'report/sale_coupon_report.xml',
        'report/sale_coupon_report_templates.xml',
        'data/sale_coupon_email_data.xml',
    ],
    'demo': [
        'data/sale_coupon_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\sale_coupon_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="product_product_10_percent_discount" model="product.product">
            <field name="name">10.0% discount on total amount</field>
            <field name="type">service</field>
            <field name="taxes_id" eval="False"/>
            <field name="supplier_taxes_id" eval="False"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="invoice_policy">order</field>
            <field name="default_code">10PERCENTDISC</field>
            <field name="categ_id" ref="product.product_category_3"/>
        </record>

        <record id="10_percent_auto_applied" model="sale.coupon.program">
            <field name="name">Code for 10% on orders</field>
            <field name="promo_code_usage">code_needed</field>
            <field name="promo_code">10pc</field>
            <field name="discount_apply_on">on_order</field>
            <field name="discount_type">percentage</field>
            <field name="discount_percentage">10.0</field>
            <field name="program_type">promotion_program</field>
            <field name="discount_line_product_id" ref="sale_coupon.product_product_10_percent_discount"/>
            <field name="validity_duration">0</field>
        </record>

        <record id="product_product_free_large_cabinet" model="product.product">
            <field name="name">Free Product - Large Cabinet</field>
            <field name="type">service</field>
            <field name="taxes_id" eval="False"/>
            <field name="supplier_taxes_id" eval="False"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="invoice_policy">order</field>
            <field name="default_code">FREELARGECABINET</field>
            <field name="categ_id" ref="product.product_category_3"/>
        </record>

        <record id="3_cabinets_plus_1_free" model="sale.coupon.program">
            <field name="name">Buy 3 large cabinets, get one for free</field>
            <field name="promo_code_usage">no_code_needed</field>
            <field name="discount_apply_on">on_order</field>
            <field name="reward_type">product</field>
            <field name="program_type">promotion_program</field>
            <field name="reward_product_id" ref="product.product_product_6"></field>
            <field name="rule_min_quantity">3</field>
            <field name="rule_products_domain">[["name","ilike","large cabinet"]]</field>
            <field name="discount_line_product_id" ref="sale_coupon.product_product_free_large_cabinet"/>
            <field name="validity_duration">0</field>
        </record>

        <record id="10_percent_coupon" model="sale.coupon.program">
            <field name="name">10% Discount</field>
            <field name="promo_code_usage">code_needed</field>
            <field name="discount_apply_on">on_order</field>
            <field name="discount_type">percentage</field>
            <field name="discount_percentage">10.0</field>
            <field name="program_type">coupon_program</field>
        </record>
    </data>
</odoo>

```

## File: data\sale_coupon_email_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
   <data noupdate="1">
      <record id="sale_coupon.mail_template_sale_coupon" model="mail.template">
         <field name="name">Coupon: Send by Email</field>
         <field name="model_id" ref="sale_coupon.model_sale_coupon"/>
         <field name="subject">Your reward coupon from ${object.program_id.company_id.name} </field>
         <field name="email_from">${object.program_id.company_id.email | safe}</field>
         <field name="partner_to">${object.order_id.partner_id.id or object.partner_id.id}</field>
         <field name="lang">${object.partner_id.lang}</field>
         <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="width:100%; margin:0px auto;"><tbody>
    <tr><td valign="top" style="text-align: center; font-size: 14px;">
        % if object.partner_id.name:
        Congratulations ${object.partner_id.name},<br />
        % endif

        Here is your reward from ${object.program_id.company_id.name}.<br />

        % if object.program_id.reward_type == 'discount':
            % if object.program_id.discount_type == 'fixed_amount':
            <span style="font-size: 50px; color: #875A7B; font-weight: bold;">
                ${'%s' % format_amount(object.program_id.discount_fixed_amount, object.program_id.currency_id)}
            </span><br />
            <strong style="font-size: 24px;">off on your next order</strong><br />
            %else
            <span style="font-size: 50px; color: #875A7B; font-weight: bold;">
                ${object.program_id.discount_percentage} %
            </span>
            % if object.program_id.discount_apply_on == 'specific_products'
                <br />
                % if len(object.program_id.discount_specific_product_ids) != 1
                % set display_specific_products = True
                <strong style="font-size: 24px;">
                    on some products*
                </strong>
                % else
                <strong style="font-size: 24px;">
                    ${'on %s' % object.program_id.discount_specific_product_ids.name}
                </strong>
                % endif
            % elif object.program_id.discount_apply_on == 'cheapest_product':
            <br /><strong style="font-size: 24px;">
                off on the cheapest product
            </strong>
            % else
            <br /><strong style="font-size: 24px;">
                off on your next order
            </strong>
            % endif
            <br />
            % endif
        % elif object.program_id.reward_type == 'product':
            <span style="font-size: 36px; color: #875A7B; font-weight: bold;">
                ${'get %s free %s' % (object.program_id.reward_product_quantity, object.program_id.reward_product_id.name)}
            </span><br />
            <strong style="font-size: 24px;">on your next order</strong><br />
        % elif object.program_id.reward_type == 'free_shipping':
            <span style="font-size: 36px; color: #875A7B; font-weight: bold;">
                get free shipping
            </span><br />
            <strong style="font-size: 24px;">on your next order</strong><br />
        % endif
    </td></tr>
    <tr style="margin-top: 16px"><td valign="top" style="text-align: center; font-size: 14px;">
        Use this promo code
        % if object.expiration_date:
            before ${object.expiration_date}
        % endif
        <p style="margin-top: 16px;">
            <strong style="padding: 16px 8px 16px 8px; border-radius: 3px; background-color: #F1F1F1;">
                ${object.code}
            </strong>
        </p>
        % if object.program_id.rule_min_quantity not in [0, 1]
        <span style="font-size: 14px;">
            Minimum purchase of ${object.program_id.rule_min_quantity} products
        </span><br />
        % endif
        % if object.program_id.rule_minimum_amount != 0.00
        <span style="font-size: 14px;">
            Valid for purchase above ${object.program_id.company_id.currency_id.symbol}${'%0.2f' % object.program_id.rule_minimum_amount |float}
        </span><br />
        % endif
        % if display_specific_products
        <span>
            *Valid for following products: ${', '.join(object.program_id.discount_specific_product_ids.mapped('name'))}
        </span><br />
        % endif
        <br/>
        Thank you,
        <br/>
        % if object.order_id.user_id:
            ${object.order_id.user_id.signature | safe}
        % endif
    </td></tr>
</tbody></table>
            </field>
            <field name="report_template" ref="report_coupon_code"/>
            <field name="report_name">Your Coupon Code</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
      </record>

        <record id="expire_coupon_cron" model="ir.cron">
            <field name="name">Coupon: expire coupon based on date</field>
            <field name="model_id" ref="sale_coupon.model_sale_coupon"/>
            <field name="state">code</field>
            <field name="code">model.cron_expire_coupon()</field>
            <field name="active" eval="True"/>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
        </record>
   </data>
</odoo>
```

## File: models\sale_coupon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import random
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _


class SaleCoupon(models.Model):
    _name = 'sale.coupon'
    _description = "Sales Coupon"
    _rec_name = 'code'

    @api.model
    def _generate_code(self):
        """Generate a 20 char long pseudo-random string of digits for barcode
        generation.

        A decimal serialisation is longer than a hexadecimal one *but* it
        generates a more compact barcode (Code128C rather than Code128A).

        Generate 8 bytes (64 bits) barcodes as 16 bytes barcodes are not 
        compatible with all scanners.
         """
        return str(random.getrandbits(64))

    code = fields.Char(default=_generate_code, required=True, readonly=True)
    expiration_date = fields.Date('Expiration Date', compute='_compute_expiration_date')
    state = fields.Selection([
        ('reserved', 'Reserved'),
        ('new', 'Valid'),
        ('used', 'Consumed'),
        ('expired', 'Expired')
        ], required=True, default='new')
    partner_id = fields.Many2one('res.partner', "For Customer")
    program_id = fields.Many2one('sale.coupon.program', "Program", required=True, ondelete="cascade")
    order_id = fields.Many2one('sale.order', 'Order Reference', readonly=True,
        help="The sales order from which coupon is generated")
    sales_order_id = fields.Many2one('sale.order', 'Applied on order', readonly=True,
        help="The sales order on which the coupon is applied")
    discount_line_product_id = fields.Many2one('product.product', related='program_id.discount_line_product_id', readonly=False,
        help='Product used in the sales order to apply the discount.')

    _sql_constraints = [
        ('unique_coupon_code', 'unique(code)', 'The coupon code must be unique!'),
    ]

    def _compute_expiration_date(self):
        self.expiration_date = 0
        for coupon in self.filtered(lambda x: x.program_id.validity_duration > 0):
            coupon.expiration_date = (coupon.create_date + relativedelta(days=coupon.program_id.validity_duration)).date()

    def _check_coupon_code(self, order):
        message = {}
        applicable_programs = order._get_applicable_programs()
        if self.state in ('used', 'expired') or \
           (self.expiration_date and self.expiration_date < fields.Datetime.now().date()):
            message = {'error': _('This coupon %s has been used or is expired.') % (self.code)}
        elif self.state == 'reserved':
            message = {'error': _('This coupon %s exists but the origin sales order is not validated yet.') % (self.code)}
        # Minimum requirement should not be checked if the coupon got generated by a promotion program (the requirement should have only be checked to generate the coupon)
        elif self.program_id.program_type == 'coupon_program' and not self.program_id._filter_on_mimimum_amount(order):
            message = {'error': _('A minimum of %s %s should be purchased to get the reward') % (self.program_id.rule_minimum_amount, self.program_id.currency_id.name)}
        elif not self.program_id.active:
            message = {'error': _('The coupon program for %s is in draft or closed state') % (self.code)}
        elif self.partner_id and self.partner_id != order.partner_id:
            message = {'error': _('Invalid partner.')}
        elif self.program_id in order.applied_coupon_ids.mapped('program_id'):
            message = {'error': _('A Coupon is already applied for the same reward')}
        elif order.code_promo_program_id.reward_type == self.program_id.reward_type == 'free_shipping':
            message = {'error': _('Free shipping has already been applied.')}
        elif self.program_id._is_global_discount_program() and order._is_global_discount_already_applied():
            message = {'error': _('Global discounts are not cumulable.')}
        elif self.program_id.reward_type == 'product' and not order._is_reward_in_order_lines(self.program_id):
            message = {'error': _('The reward products should be in the sales order lines to apply the discount.')}
        elif not self.program_id._is_valid_partner(order.partner_id):
            message = {'error': _("The customer doesn't have access to this reward.")}
        # Product requirement should not be checked if the coupon got generated by a promotion program (the requirement should have only be checked to generate the coupon)
        elif self.program_id.program_type == 'coupon_program' and not self.program_id._filter_programs_on_products(order):
            message = {'error': _("You don't have the required product quantities on your sales order. All the products should be recorded on the sales order. (Example: You need to have 3 T-shirts on your sales order if the promotion is 'Buy 2, Get 1 Free').")}
        else:
            if self.program_id not in applicable_programs and self.program_id.promo_applicability == 'on_current_order':
                message = {'error': _('At least one of the required conditions is not met to get the reward!')}
        return message

    def action_coupon_sent(self):
        """ Open a window to compose an email, with the edi invoice template
            message loaded by default
        """
        self.ensure_one()
        template = self.env.ref('sale_coupon.mail_template_sale_coupon', False)
        compose_form = self.env.ref('mail.email_compose_message_wizard_form', False)
        ctx = dict(
            default_model='sale.coupon',
            default_res_id=self.id,
            default_use_template=bool(template),
            default_template_id=template.id,
            default_composition_mode='comment',
            custom_layout='mail.mail_notification_light',
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

    def cron_expire_coupon(self):
        self._cr.execute("""
            SELECT C.id FROM SALE_COUPON as C
            INNER JOIN SALE_COUPON_PROGRAM as P ON C.program_id = P.id
            WHERE C.STATE in ('reserved', 'new')
                AND P.validity_duration > 0
                AND C.create_date + interval '1 day' * P.validity_duration < now()""")

        expired_ids = [res[0] for res in self._cr.fetchall()]
        self.browse(expired_ids).write({'state': 'expired'})

```

## File: models\sale_coupon_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.safe_eval import safe_eval


class SaleCouponProgram(models.Model):
    _name = 'sale.coupon.program'
    _description = "Sales Coupon Program"
    _inherits = {'sale.coupon.rule': 'rule_id', 'sale.coupon.reward': 'reward_id'}
    # We should apply 'discount' promotion first to avoid offering free product when we should not.
    # Eg: If the discount lower the SO total below the required threshold
    # Note: This is only revelant when programs have the same sequence (which they have by default)
    _order = "sequence, reward_type"

    name = fields.Char(required=True, translate=True)
    active = fields.Boolean('Active', default=True, help="A program is available for the customers when active")
    rule_id = fields.Many2one('sale.coupon.rule', string="Coupon Rule", ondelete='restrict', required=True)
    reward_id = fields.Many2one('sale.coupon.reward', string="Reward", ondelete='restrict', required=True, copy=False)
    sequence = fields.Integer(copy=False,
        help="Coupon program will be applied based on given sequence if multiple programs are " +
        "defined on same condition(For minimum amount)")
    maximum_use_number = fields.Integer(help="Maximum number of sales orders in which reward can be provided")
    program_type = fields.Selection([
        ('promotion_program', 'Promotional Program'),
        ('coupon_program', 'Coupon Program'),
        ],
        help="""A promotional program can be either a limited promotional offer without code (applied automatically)
                or with a code (displayed on a magazine for example) that may generate a discount on the current
                order or create a coupon for a next order.

                A coupon program generates coupons with a code that can be used to generate a discount on the current
                order or create a coupon for a next order.""")
    promo_code_usage = fields.Selection([
        ('no_code_needed', 'Automatically Applied'),
        ('code_needed', 'Use a code')],
        help="Automatically Applied - No code is required, if the program rules are met, the reward is applied (Except the global discount or the free shipping rewards which are not cumulative)\n" +
             "Use a code - If the program rules are met, a valid code is mandatory for the reward to be applied\n")
    promo_code = fields.Char('Promotion Code', copy=False,
        help="A promotion code is a code that is associated with a marketing discount. For example, a retailer might tell frequent customers to enter the promotion code 'THX001' to receive a 10%% discount on their whole order.")
    promo_applicability = fields.Selection([
        ('on_current_order', 'Apply On Current Order'),
        ('on_next_order', 'Apply On Next Order')],
        default='on_current_order', string="Applicability")
    coupon_ids = fields.One2many('sale.coupon', 'program_id', string="Generated Coupons", copy=False)
    coupon_count = fields.Integer(compute='_compute_coupon_count')
    order_count = fields.Integer(compute='_compute_order_count')
    company_id = fields.Many2one('res.company', string="Company", default=lambda self: self.env.company)
    currency_id = fields.Many2one(string="Currency", related='company_id.currency_id', readonly=True)
    validity_duration = fields.Integer(default=1,
        help="Validity duration for a coupon after its generation")

    @api.constrains('promo_code')
    def _check_promo_code_constraint(self):
        """ Program code must be unique """
        for program in self.filtered(lambda p: p.promo_code):
            domain = [('id', '!=', program.id), ('promo_code', '=', program.promo_code)]
            if self.search(domain):
                raise ValidationError(_('The program code must be unique!'))

    # The api.depends is handled in `def modified` of `sale_coupon/models/sale_order.py`
    def _compute_order_count(self):
        product_data = self.env['sale.order.line'].read_group([('product_id', 'in', self.mapped('discount_line_product_id').ids)], ['product_id'], ['product_id'])
        mapped_data = dict([(m['product_id'][0], m['product_id_count']) for m in product_data])
        for program in self:
            program.order_count = mapped_data.get(program.discount_line_product_id.id, 0)

    @api.depends('coupon_ids')
    def _compute_coupon_count(self):
        coupon_data = self.env['sale.coupon'].read_group([('program_id', 'in', self.ids)], ['program_id'], ['program_id'])
        mapped_data = dict([(m['program_id'][0], m['program_id_count']) for m in coupon_data])
        for program in self:
            program.coupon_count = mapped_data.get(program.id, 0)

    @api.onchange('promo_code_usage')
    def _onchange_promo_code_usage(self):
        if self.promo_code_usage == 'no_code_needed':
            self.promo_code = False

    @api.onchange('reward_product_id')
    def _onchange_reward_product_id(self):
        if self.reward_product_id:
            self.reward_product_uom_id = self.reward_product_id.uom_id

    @api.onchange('discount_type')
    def _onchange_discount_type(self):
        if self.discount_type == 'fixed_amount':
            self.discount_apply_on = 'on_order'

    @api.model
    def create(self, vals):
        program = super(SaleCouponProgram, self).create(vals)
        if not vals.get('discount_line_product_id', False):
            discount_line_product_id = self.env['product.product'].create({
                'name': program.reward_id.display_name,
                'type': 'service',
                'taxes_id': False,
                'supplier_taxes_id': False,
                'sale_ok': False,
                'purchase_ok': False,
                'invoice_policy': 'order',
                'lst_price': 0, #Do not set a high value to avoid issue with coupon code
            })
            program.write({'discount_line_product_id': discount_line_product_id.id})
        return program

    def write(self, vals):
        res = super(SaleCouponProgram, self).write(vals)
        reward_fields = [
            'reward_type', 'reward_product_id', 'discount_type', 'discount_percentage',
            'discount_apply_on', 'discount_specific_product_ids', 'discount_fixed_amount'
        ]
        if any(field in reward_fields for field in vals):
            self.mapped('discount_line_product_id').write({'name': self[0].reward_id.display_name})
        return res

    def unlink(self):
        for program in self.filtered(lambda x: x.active):
            raise UserError(_('You can not delete a program in active state'))
        return super(SaleCouponProgram, self).unlink()

    def toggle_active(self):
        super(SaleCouponProgram, self).toggle_active()
        for program in self:
            program.discount_line_product_id.active = program.active
        coupons = self.filtered(lambda p: not p.active and p.promo_code_usage == 'code_needed').mapped('coupon_ids')
        coupons.filtered(lambda x: x.state != 'used').write({'state': 'expired'})

    def action_view_sales_orders(self):
        self.ensure_one()
        orders = self.env['sale.order.line'].search([('product_id', '=', self.discount_line_product_id.id)]).mapped('order_id')
        return {
            'name': _('Sales Orders'),
            'view_mode': 'tree,form',
            'res_model': 'sale.order',
            'search_view_id': [self.env.ref('sale.sale_order_view_search_inherit_quotation').id],
            'type': 'ir.actions.act_window',
            'domain': [('id', 'in', orders.ids)],
            'context': dict(self._context, create=False)
        }

    def _is_global_discount_program(self):
        self.ensure_one()
        return self.promo_applicability == 'on_current_order' and \
               self.reward_type == 'discount' and \
               self.discount_type == 'percentage' and \
               self.discount_apply_on == 'on_order'

    def _keep_only_most_interesting_auto_applied_global_discount_program(self):
        '''Given a record set of programs, remove the less interesting auto
        applied global discount to keep only the most interesting one.
        We should not take promo code programs into account as a 10% auto
        applied is considered better than a 50% promo code, as the user might
        not know about the promo code.
        '''
        programs = self.filtered(lambda p: p._is_global_discount_program() and p.promo_code_usage == 'no_code_needed')
        if not programs: return self
        most_interesting_program = max(programs, key=lambda p: p.discount_percentage)
        # remove least interesting programs
        return self - (programs - most_interesting_program)

    def _check_promo_code(self, order, coupon_code):
        message = {}
        if self.maximum_use_number != 0 and self.order_count >= self.maximum_use_number:
            message = {'error': _('Promo code %s has been expired.') % (coupon_code)}
        elif not self._filter_on_mimimum_amount(order):
            message = {'error': _('A minimum of %s %s should be purchased to get the reward') % (self.rule_minimum_amount, self.currency_id.name)}
        elif self.promo_code and self.promo_code == order.promo_code:
            message = {'error': _('The promo code is already applied on this order')}
        elif self in order.no_code_promo_program_ids:
            message = {'error': _('The promotional offer is already applied on this order')}
        elif not self.active:
            message = {'error': _('Promo code is invalid')}
        elif self.rule_date_from and self.rule_date_from > fields.Datetime.now() or self.rule_date_to and fields.Datetime.now() > self.rule_date_to:
            message = {'error': _('Promo code is expired')}
        elif order.promo_code and self.promo_code_usage == 'code_needed':
            message = {'error': _('Promotionals codes are not cumulative.')}
        elif self.reward_type == 'free_shipping' and order.applied_coupon_ids.filtered(lambda c: c.program_id.reward_type == 'free_shipping'):
            message = {'error': _('Free shipping has already been applied.')}
        elif self._is_global_discount_program() and order._is_global_discount_already_applied():
            message = {'error': _('Global discounts are not cumulative.')}
        elif self.promo_applicability == 'on_current_order' and self.reward_type == 'product' and not order._is_reward_in_order_lines(self):
            message = {'error': _('The reward products should be in the sales order lines to apply the discount.')}
        elif not self._is_valid_partner(order.partner_id):
            message = {'error': _("The customer doesn't have access to this reward.")}
        elif not self._filter_programs_on_products(order):
            message = {'error': _("You don't have the required product quantities on your sales order. If the reward is same product quantity, please make sure that all the products are recorded on the sales order (Example: You need to have 3 T-shirts on your sales order if the promotion is 'Buy 2, Get 1 Free'.")}
        elif self.promo_applicability == 'on_current_order' and not self.env.context.get('applicable_coupon'):
            applicable_programs = order._get_applicable_programs()
            if self not in applicable_programs:
                message = {'error': _('At least one of the required conditions is not met to get the reward!')}
        return message

    def _compute_program_amount(self, field, currency_to):
        self.ensure_one()
        return self.currency_id._convert(getattr(self, field), currency_to, self.company_id, fields.Date.today())

    @api.model
    def _filter_on_mimimum_amount(self, order):
        no_effect_lines = order._get_no_effect_on_threshold_lines()
        order_amount = {
            'amount_untaxed' : order.amount_untaxed - sum(line.price_subtotal for line in no_effect_lines),
            'amount_tax' : order.amount_tax - sum(line.price_tax for line in no_effect_lines)
        }
        program_ids = list()
        for program in self:
            if program.reward_type != 'discount':
                # avoid the filtered
                lines = self.env['sale.order.line']
            else:
                lines = order.order_line.filtered(lambda line:
                    line.product_id == program.discount_line_product_id or
                    line.product_id == program.reward_id.discount_line_product_id or
                    (program.program_type == 'promotion_program' and line.is_reward_line)
                )
            untaxed_amount = order_amount['amount_untaxed'] - sum(line.price_subtotal for line in lines)
            tax_amount = order_amount['amount_tax'] - sum(line.price_tax for line in lines)
            program_amount = program._compute_program_amount('rule_minimum_amount', order.currency_id)
            if program.rule_minimum_amount_tax_inclusion == 'tax_included' and program_amount <= (untaxed_amount + tax_amount) or program_amount <= untaxed_amount:
                program_ids.append(program.id)

        return self.env['sale.coupon.program'].browse(program_ids)

    @api.model
    def _filter_on_validity_dates(self, order):
        return self.filtered(lambda program:
            (not program.rule_date_from or program.rule_date_from <= fields.Datetime.now())
            and
            (not program.rule_date_to or program.rule_date_to >= fields.Datetime.now())
        )

    @api.model
    def _filter_promo_programs_with_code(self, order):
        '''Filter Promo program with code with a different promo_code if a promo_code is already ordered'''
        return self.filtered(lambda program: program.promo_code_usage == 'code_needed' and program.promo_code != order.promo_code)

    def _filter_unexpired_programs(self, order):
        return self.filtered(
            lambda program: program.maximum_use_number == 0
            or program.order_count < program.maximum_use_number
            or program
            in (order.code_promo_program_id + order.no_code_promo_program_ids)
        )

    def _filter_programs_on_partners(self, order):
        return self.filtered(lambda program: program._is_valid_partner(order.partner_id))

    def _filter_programs_on_products(self, order):
        """
        To get valid programs according to product list.
        i.e Buy 1 imac + get 1 ipad mini free then check 1 imac is on cart or not
        or  Buy 1 coke + get 1 coke free then check 2 cokes are on cart or not
        """
        order_lines = order.order_line.filtered(lambda line: line.product_id) - order._get_reward_lines()
        products = order_lines.mapped('product_id')
        products_qties = dict.fromkeys(products, 0)
        for line in order_lines:
            products_qties[line.product_id] += line.product_uom_qty
        valid_program_ids = list()
        for program in self:
            if not program.rule_products_domain:
                valid_program_ids.append(program.id)
                continue
            valid_products = program._get_valid_products(products)
            if not valid_products:
                # The program can be directly discarded
                continue
            ordered_rule_products_qty = sum(products_qties[product] for product in valid_products)
            # Avoid program if 1 ordered foo on a program '1 foo, 1 free foo'
            if program.promo_applicability == 'on_current_order' and \
               program.reward_type == 'product' and program._get_valid_products(program.reward_product_id):
                ordered_rule_products_qty -= program.reward_product_quantity
            if ordered_rule_products_qty >= program.rule_min_quantity:
                valid_program_ids.append(program.id)
        return self.browse(valid_program_ids)

    def _filter_not_ordered_reward_programs(self, order):
        """
        Returns the programs when the reward is actually in the order lines
        """
        programs = self.env['sale.coupon.program']
        for program in self:
            if program.reward_type == 'product' and \
               not order.order_line.filtered(lambda line: line.product_id == program.reward_product_id):
                continue
            elif program.reward_type == 'discount' and program.discount_apply_on == 'specific_products' and \
               not order.order_line.filtered(lambda line: line.product_id in program.discount_specific_product_ids):
                continue
            programs |= program
        return programs

    @api.model
    def _filter_programs_from_common_rules(self, order, next_order=False):
        """ Return the programs if every conditions is met
            :param bool next_order: is the reward given from a previous order
        """
        programs = self
        # Minimum requirement should not be checked if the coupon got generated by a promotion program (the requirement should have only be checked to generate the coupon)
        if not next_order:
            programs = programs and programs._filter_on_mimimum_amount(order)
        if not self.env.context.get("no_outdated_coupons"):
            programs = programs and programs._filter_on_validity_dates(order)
        programs = programs and programs._filter_unexpired_programs(order)
        programs = programs and programs._filter_programs_on_partners(order)
        # Product requirement should not be checked if the coupon got generated by a promotion program (the requirement should have only be checked to generate the coupon)
        if not next_order:
            programs = programs and programs._filter_programs_on_products(order)

        programs_curr_order = programs.filtered(lambda p: p.promo_applicability == 'on_current_order')
        programs = programs.filtered(lambda p: p.promo_applicability == 'on_next_order')
        if programs_curr_order:
            # Checking if rewards are in the SO should not be performed for rewards on_next_order
            programs += programs_curr_order._filter_not_ordered_reward_programs(order)
        return programs

    def _is_valid_partner(self, partner):
        if self.rule_partners_domain and self.rule_partners_domain != '[]':
            domain = safe_eval(self.rule_partners_domain) + [('id', '=', partner.id)]
            return bool(self.env['res.partner'].search_count(domain))
        else:
            return True

    def _is_valid_product(self, product):
        # VFE TODO remove in master
        # NOTE: if you override this method, think of also overriding _get_valid_products
        # we also encourage the use of _get_valid_products as its execution is faster
        if self.rule_products_domain:
            domain = safe_eval(self.rule_products_domain) + [('id', '=', product.id)]
            return bool(self.env['product.product'].search_count(domain))
        else:
            return True

    def _get_valid_products(self, products):
        if self.rule_products_domain:
            domain = safe_eval(self.rule_products_domain)
            return products.filtered_domain(domain)
        return products

```

## File: models\sale_coupon_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class SaleCouponReward(models.Model):
    _name = 'sale.coupon.reward'
    _description = "Sales Coupon Reward"
    _rec_name = 'reward_description'

    # VFE FIXME multi company
    """Rewards are not restricted to a company...
    You could have a reward_product_id limited to a specific company A.
    But still use this reward as reward of a program of company B...
    """
    reward_description = fields.Char('Reward Description')
    reward_type = fields.Selection([
        ('discount', 'Discount'),
        ('product', 'Free Product'),
        ], string='Reward Type', default='discount',
        help="Discount - Reward will be provided as discount.\n" +
        "Free Product - Free product will be provide as reward \n" +
        "Free Shipping - Free shipping will be provided as reward (Need delivery module)")
    # Product Reward
    reward_product_id = fields.Many2one('product.product', string="Free Product",
        help="Reward Product")
    reward_product_quantity = fields.Integer(string="Quantity", default=1, help="Reward product quantity")
    # Discount Reward
    discount_type = fields.Selection([
        ('percentage', 'Percentage'),
        ('fixed_amount', 'Fixed Amount')], default="percentage",
        help="Percentage - Entered percentage discount will be provided\n" +
        "Amount - Entered fixed amount discount will be provided")
    discount_percentage = fields.Float(string="Discount", default=10,
        help='The discount in percentage, between 1 to 100')
    discount_apply_on = fields.Selection([
        ('on_order', 'On Order'),
        ('cheapest_product', 'On Cheapest Product'),
        ('specific_products', 'On Specific Products')], default="on_order",
        help="On Order - Discount on whole order\n" +
        "Cheapest product - Discount on cheapest product of the order\n" +
        "Specific products - Discount on selected specific products")
    discount_specific_product_ids = fields.Many2many('product.product', string="Products",
        help="Products that will be discounted if the discount is applied on specific products")
    discount_max_amount = fields.Float(default=0,
        help="Maximum amount of discount that can be provided")
    discount_fixed_amount = fields.Float(string="Fixed Amount", help='The discount in fixed amount')
    reward_product_uom_id = fields.Many2one(related='reward_product_id.product_tmpl_id.uom_id', string='Unit of Measure', readonly=True)
    discount_line_product_id = fields.Many2one('product.product', string='Reward Line Product', copy=False,
        help="Product used in the sales order to apply the discount. Each coupon program has its own reward product for reporting purpose")

    @api.constrains('discount_percentage')
    def _check_discount_percentage(self):
        if self.filtered(lambda reward: reward.discount_type == 'percentage' and (reward.discount_percentage < 0 or reward.discount_percentage > 100)):
            raise ValidationError(_('Discount percentage should be between 1-100'))

    def name_get(self):
        """
        Returns a complete description of the reward
        """
        result = []
        for reward in self:
            reward_string = ""
            if reward.reward_type == 'product':
                reward_string = _("Free Product - %s" % (reward.reward_product_id.name))
            elif reward.reward_type == 'discount':
                if reward.discount_type == 'percentage':
                    reward_percentage = str(reward.discount_percentage)
                    if reward.discount_apply_on == 'on_order':
                        reward_string = _("%s%% discount on total amount" % (reward_percentage))
                    elif reward.discount_apply_on == 'specific_products':
                        if len(reward.discount_specific_product_ids) > 1:
                            reward_string = _("%s%% discount on products" % (reward_percentage))
                        else:
                            reward_string = _("%s%% discount on %s" % (reward_percentage, reward.discount_specific_product_ids.name))
                    elif reward.discount_apply_on == 'cheapest_product':
                        reward_string = _("%s%% discount on cheapest product" % (reward_percentage))
                elif reward.discount_type == 'fixed_amount':
                    program = self.env['sale.coupon.program'].search([('reward_id', '=', reward.id)])
                    reward_string = _("%s %s discount on total amount" % (str(reward.discount_fixed_amount), program.currency_id.name))
            result.append((reward.id, reward_string))
        return result

```

## File: models\sale_coupon_rules.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools.safe_eval import safe_eval


class SaleCouponRule(models.Model):
    _name = 'sale.coupon.rule'
    _description = "Sales Coupon Rule"

    rule_date_from = fields.Datetime(string="Start Date", help="Coupon program start date")
    rule_date_to = fields.Datetime(string="End Date", help="Coupon program end date")
    rule_partners_domain = fields.Char(string="Based on Customers", help="Coupon program will work for selected customers only")
    rule_products_domain = fields.Char(string="Based on Products", default=[['sale_ok', '=', True]], help="On Purchase of selected product, reward will be given")
    rule_min_quantity = fields.Integer(string="Minimum Quantity", default=1,
        help="Minimum required product quantity to get the reward")
    rule_minimum_amount = fields.Float(default=0.0, help="Minimum required amount to get the reward")
    rule_minimum_amount_tax_inclusion = fields.Selection([
        ('tax_included', 'Tax Included'),
        ('tax_excluded', 'Tax Excluded')], default="tax_excluded")

    @api.constrains('rule_date_to', 'rule_date_from')
    def _check_rule_date_from(self):
        if any(applicability for applicability in self
               if applicability.rule_date_to and applicability.rule_date_from
               and applicability.rule_date_to < applicability.rule_date_from):
            raise ValidationError(_('The start date must be before the end date'))

    @api.constrains('rule_minimum_amount')
    def _check_rule_minimum_amount(self):
        if self.filtered(lambda applicability: applicability.rule_minimum_amount < 0):
            raise ValidationError(_('Minimum purchased amount should be greater than 0'))

    @api.constrains('rule_min_quantity')
    def _check_rule_min_quantity(self):
        if not self.rule_min_quantity > 0:
            raise ValidationError(_('Minimum quantity should be greater than 0'))

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools.misc import formatLang


class SaleOrder(models.Model):
    _inherit = "sale.order"

    applied_coupon_ids = fields.One2many('sale.coupon', 'sales_order_id', string="Applied Coupons", copy=False)
    generated_coupon_ids = fields.One2many('sale.coupon', 'order_id', string="Offered Coupons", copy=False)
    reward_amount = fields.Float(compute='_compute_reward_total')
    no_code_promo_program_ids = fields.Many2many('sale.coupon.program', string="Applied Immediate Promo Programs",
        domain="[('promo_code_usage', '=', 'no_code_needed'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", copy=False)
    code_promo_program_id = fields.Many2one('sale.coupon.program', string="Applied Promo Program",
        domain="[('promo_code_usage', '=', 'code_needed'), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", copy=False)
    promo_code = fields.Char(related='code_promo_program_id.promo_code', help="Applied program code", readonly=False)

    @api.depends('order_line')
    def _compute_reward_total(self):
        for order in self:
            order.reward_amount = sum([line.price_subtotal for line in order._get_reward_lines()])

    def _get_no_effect_on_threshold_lines(self):
        self.ensure_one()
        lines = self.env['sale.order.line']
        return lines

    def recompute_coupon_lines(self):
        for order in self:
            order._remove_invalid_reward_lines()
            if order.state != 'cancel':
                order._create_new_no_code_promo_reward_lines()
            order._update_existing_reward_lines()

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        order = super(SaleOrder, self).copy(default)
        reward_line = order._get_reward_lines()
        if reward_line:
            reward_line.unlink()
            order._create_new_no_code_promo_reward_lines()
        return order

    def action_confirm(self):
        self.generated_coupon_ids.write({'state': 'new'})
        self.applied_coupon_ids.write({'state': 'used'})
        self._send_reward_coupon_mail()
        return super(SaleOrder, self).action_confirm()

    def action_cancel(self):
        res = super(SaleOrder, self).action_cancel()
        self.generated_coupon_ids.write({'state': 'expired'})
        self.applied_coupon_ids.write({'state': 'new'})
        self.applied_coupon_ids.sales_order_id = False
        self.recompute_coupon_lines()
        return res

    def action_draft(self):
        res = super(SaleOrder, self).action_draft()
        self.generated_coupon_ids.write({'state': 'reserved'})
        return res

    def _get_reward_lines(self):
        self.ensure_one()
        return self.order_line.filtered(lambda line: line.is_reward_line)

    def _is_reward_in_order_lines(self, program):
        self.ensure_one()
        order_quantity = sum(self.order_line.filtered(lambda line:
            line.product_id == program.reward_product_id).mapped('product_uom_qty'))
        return order_quantity >= program.reward_product_quantity

    def _is_global_discount_already_applied(self):
        applied_programs = self.no_code_promo_program_ids + \
                           self.code_promo_program_id + \
                           self.applied_coupon_ids.mapped('program_id')
        return applied_programs.filtered(lambda program: program._is_global_discount_program())

    def _get_reward_values_product(self, program):
        price_unit = self.order_line.filtered(lambda line: program.reward_product_id == line.product_id)[0].price_reduce

        order_lines = (self.order_line - self._get_reward_lines()).filtered(lambda x: program._get_valid_products(x.product_id))
        max_product_qty = sum(order_lines.mapped('product_uom_qty')) or 1
        total_qty = sum(self.order_line.filtered(lambda x: x.product_id == program.reward_product_id).mapped('product_uom_qty'))
        # Remove needed quantity from reward quantity if same reward and rule product
        if program._get_valid_products(program.reward_product_id):
            # number of times the program should be applied
            program_in_order = max_product_qty // (program.rule_min_quantity + program.reward_product_quantity)
            # multipled by the reward qty
            reward_product_qty = program.reward_product_quantity * program_in_order
            # do not give more free reward than products
            reward_product_qty = min(reward_product_qty, total_qty)
            if program.rule_minimum_amount:
                order_total = sum(order_lines.mapped('price_total')) - (program.reward_product_quantity * program.reward_product_id.lst_price)
                reward_product_qty = min(reward_product_qty, order_total // program.rule_minimum_amount)
        else:
            program_in_order = max_product_qty // program.rule_min_quantity
            reward_product_qty = min(program.reward_product_quantity * program_in_order, total_qty)

        reward_qty = min(int(int(max_product_qty / program.rule_min_quantity) * program.reward_product_quantity), reward_product_qty)
        # Take the default taxes on the reward product, mapped with the fiscal position
        taxes = program.reward_product_id.taxes_id.filtered(lambda t: t.company_id.id == self.company_id.id)
        if self.fiscal_position_id:
            taxes = self.fiscal_position_id.map_tax(taxes)
        return {
            'product_id': program.discount_line_product_id.id,
            'price_unit': - price_unit,
            'product_uom_qty': reward_qty,
            'is_reward_line': True,
            'name': _("Free Product") + " - " + program.reward_product_id.name,
            'product_uom': program.reward_product_id.uom_id.id,
            'tax_id': [(4, tax.id, False) for tax in taxes],
        }

    def _get_paid_order_lines(self):
        """ Returns the sale order lines that are not reward lines.
            It will also return reward lines being free product lines. """
        free_reward_product = self.env['sale.coupon.program'].search([('reward_type', '=', 'product')]).mapped('discount_line_product_id')
        return self.order_line.filtered(lambda x: not x.is_reward_line or x.product_id in free_reward_product)

    def _get_base_order_lines(self, program):
        """ Returns the sale order lines not linked to the given program.
        """
        return self.order_line.filtered(lambda x: not (x.is_reward_line and x.product_id == program.discount_line_product_id))

    def _get_reward_values_discount_fixed_amount(self, program):
        total_amount = sum(self._get_base_order_lines(program).mapped('price_total'))
        fixed_amount = program._compute_program_amount('discount_fixed_amount', self.currency_id)
        if total_amount < fixed_amount:
            return total_amount
        else:
            return fixed_amount

    def _get_coupon_program_domain(self):
        return []

    def _get_cheapest_line(self):
        # Unit prices tax included
        return min(self.order_line.filtered(lambda x: not x.is_reward_line and x.price_reduce > 0), key=lambda x: x['price_reduce'])

    def _get_reward_values_discount_percentage_per_line(self, program, line):
        discount_amount = line.product_uom_qty * line.price_reduce * (program.discount_percentage / 100)
        return discount_amount

    def _get_max_reward_values_per_tax(self, program, taxes):
        lines = self.order_line.filtered(lambda l: l.tax_id == taxes and l.product_id != program.discount_line_product_id)
        return sum(lines.mapped(lambda l: l.price_reduce * l.product_uom_qty))

    def _get_reward_values_fixed_amount(self, program):
        discount_amount = self._get_reward_values_discount_fixed_amount(program)

        # In case there is a tax set on the promotion product, we give priority to it.
        # This allow manual overwrite of taxes for promotion.
        if program.discount_line_product_id.taxes_id:
            line_taxes = self.fiscal_position_id.map_tax(program.discount_line_product_id.taxes_id) if self.fiscal_position_id else program.discount_line_product_id.taxes_id
            return [{
                'name': _("Discount: ") + program.name,
                'product_id': program.discount_line_product_id.id,
                'price_unit': -discount_amount,
                'product_uom_qty': 1.0,
                'product_uom': program.discount_line_product_id.uom_id.id,
                'is_reward_line': True,
                'tax_id': [(4, tax.id, False) for tax in line_taxes],
            }]

        lines = self._get_paid_order_lines()
        # Remove Free Lines
        lines = lines.filtered('price_reduce')
        reward_lines = {}

        tax_groups = set([line.tax_id for line in lines])
        max_discount_per_tax_groups = {tax_ids: self._get_max_reward_values_per_tax(program, tax_ids) for tax_ids in tax_groups}

        for tax_ids in sorted(tax_groups, key=lambda tax_ids: max_discount_per_tax_groups[tax_ids], reverse=True):

            if discount_amount <= 0:
                return reward_lines.values()

            curr_lines = lines.filtered(lambda l: l.tax_id == tax_ids)
            lines_price = sum(curr_lines.mapped(lambda l: l.price_reduce * l.product_uom_qty))
            lines_total = sum(curr_lines.mapped('price_total'))

            discount_line_amount_price = min(max_discount_per_tax_groups[tax_ids], (discount_amount * lines_price / lines_total))

            if not discount_line_amount_price:
                continue

            discount_amount -= discount_line_amount_price * lines_total / lines_price

            tax_name = ""
            if len(tax_ids) == 1:
                tax_name = " - " + _("On product with following tax: ") + ', '.join(tax_ids.mapped('name'))
            elif len(tax_ids) > 1:
                tax_name = " - " + _("On product with following taxes: ") + ', '.join(tax_ids.mapped('name'))

            reward_lines[tax_ids] = {
                'name': _("Discount: ") + program.name + tax_name,
                'product_id': program.discount_line_product_id.id,
                'price_unit': -discount_line_amount_price,
                'product_uom_qty': 1.0,
                'product_uom': program.discount_line_product_id.uom_id.id,
                'is_reward_line': True,
                'tax_id': [(4, tax.id, False) for tax in tax_ids],
                }
        return reward_lines.values()

    def _get_reward_values_discount(self, program):
        if program.discount_type == 'fixed_amount':
            return self._get_reward_values_fixed_amount(program)
        else:
            return self._get_reward_values_percentage_amount(program)

    def _get_reward_values_percentage_amount(self, program):
        # Invalidate multiline fixed_price discount line as they should apply after % discount
        fixed_price_products = self._get_applied_programs().filtered(
            lambda p: p.discount_type == 'fixed_amount'
        ).mapped('discount_line_product_id')
        self.order_line.filtered(lambda l: l.product_id in fixed_price_products).write({'price_unit': 0})

        reward_dict = {}
        lines = self._get_paid_order_lines()
        amount_total = sum([any(line.tax_id.mapped('price_include')) and line.price_total or line.price_subtotal
                            for line in self._get_base_order_lines(program)])
        if program.discount_apply_on == 'cheapest_product':
            line = self._get_cheapest_line()
            if line:
                discount_line_amount = min(line.price_reduce * (program.discount_percentage / 100), amount_total)
                if discount_line_amount:
                    taxes = line.tax_id
                    if self.fiscal_position_id:
                        taxes = self.fiscal_position_id.map_tax(taxes)

                    reward_dict[line.tax_id] = {
                        'name': _("Discount: ") + program.name,
                        'product_id': program.discount_line_product_id.id,
                        'price_unit': - discount_line_amount if discount_line_amount > 0 else 0,
                        'product_uom_qty': 1.0,
                        'product_uom': program.discount_line_product_id.uom_id.id,
                        'is_reward_line': True,
                        'tax_id': [(4, tax.id, False) for tax in taxes],
                    }
        elif program.discount_apply_on in ['specific_products', 'on_order']:
            if program.discount_apply_on == 'specific_products':
                # We should not exclude reward line that offer this product since we need to offer only the discount on the real paid product (regular product - free product)
                free_product_lines = self.env['sale.coupon.program'].search([('reward_type', '=', 'product'), ('reward_product_id', 'in', program.discount_specific_product_ids.ids)]).mapped('discount_line_product_id')
                lines = lines.filtered(lambda x: x.product_id in (program.discount_specific_product_ids | free_product_lines))

            # when processing lines we should not discount more than the order remaining total
            currently_discounted_amount = 0
            for line in lines:
                discount_line_amount = min(self._get_reward_values_discount_percentage_per_line(program, line), amount_total - currently_discounted_amount)

                if discount_line_amount:

                    if line.tax_id in reward_dict:
                        reward_dict[line.tax_id]['price_unit'] -= discount_line_amount
                    else:
                        taxes = line.tax_id
                        if self.fiscal_position_id:
                            taxes = self.fiscal_position_id.map_tax(taxes)

                        tax_name = ""
                        if len(taxes) == 1:
                            tax_name = " - " + _("On product with following tax: ") + ', '.join(taxes.mapped('name'))
                        elif len(taxes) > 1:
                            tax_name = " - " + _("On product with following taxes: ") + ', '.join(taxes.mapped('name'))

                        reward_dict[line.tax_id] = {
                            'name': _("Discount: ") + program.name + tax_name,
                            'product_id': program.discount_line_product_id.id,
                            'price_unit': - discount_line_amount if discount_line_amount > 0 else 0,
                            'product_uom_qty': 1.0,
                            'product_uom': program.discount_line_product_id.uom_id.id,
                            'is_reward_line': True,
                            'tax_id': [(4, tax.id, False) for tax in taxes],
                        }
                        currently_discounted_amount += discount_line_amount

        # If there is a max amount for discount, we might have to limit some discount lines or completely remove some lines
        max_amount = program._compute_program_amount('discount_max_amount', self.currency_id)
        if max_amount > 0:
            amount_already_given = 0
            for val in list(reward_dict):
                amount_to_discount = amount_already_given + reward_dict[val]["price_unit"]
                if abs(amount_to_discount) > max_amount:
                    reward_dict[val]["price_unit"] = - (max_amount - abs(amount_already_given))
                    add_name = formatLang(self.env, max_amount, currency_obj=self.currency_id)
                    reward_dict[val]["name"] += "( " + _("limited to ") + add_name + ")"
                amount_already_given += reward_dict[val]["price_unit"]
                if reward_dict[val]["price_unit"] == 0:
                    del reward_dict[val]
        return reward_dict.values()

    def _get_reward_line_values(self, program):
        self.ensure_one()
        self = self.with_context(lang=self.partner_id.lang)
        program = program.with_context(lang=self.partner_id.lang)
        if program.reward_type == 'discount':
            return self._get_reward_values_discount(program)
        elif program.reward_type == 'product':
            return [self._get_reward_values_product(program)]

    def _create_reward_line(self, program):
        self.write({'order_line': [(0, False, value) for value in self._get_reward_line_values(program)]})

    def _create_reward_coupon(self, program):
        # if there is already a coupon that was set as expired, reactivate that one instead of creating a new one
        coupon = self.env['sale.coupon'].search([
            ('program_id', '=', program.id),
            ('state', '=', 'expired'),
            ('partner_id', '=', self.partner_id.id),
            ('order_id', '=', self.id),
            ('discount_line_product_id', '=', program.discount_line_product_id.id),
        ], limit=1)
        if coupon:
            coupon.write({'state': 'reserved'})
        else:
            coupon = self.env['sale.coupon'].sudo().create({
                'program_id': program.id,
                'state': 'reserved',
                'partner_id': self.partner_id.id,
                'order_id': self.id,
                'discount_line_product_id': program.discount_line_product_id.id
            })
        self.generated_coupon_ids |= coupon
        return coupon

    def _send_reward_coupon_mail(self):
        template = self.env.ref('sale_coupon.mail_template_sale_coupon', raise_if_not_found=False)
        if template:
            for order in self:
                for coupon in order.generated_coupon_ids:
                    order.message_post_with_template(
                        template.id, composition_mode='comment',
                        model='sale.coupon', res_id=coupon.id,
                        email_layout_xmlid='mail.mail_notification_light',
                    )

    def _get_applicable_programs(self):
        """
        This method is used to return the valid applicable programs on given order.
        """
        self.ensure_one()
        programs = self.env['sale.coupon.program'].with_context(
            no_outdated_coupons=True
        ).search([
            ('company_id', 'in', [self.company_id.id, False]),
            '|', ('rule_date_from', '=', False), ('rule_date_from', '<=', fields.Datetime.now()),
            '|', ('rule_date_to', '=', False), ('rule_date_to', '>=', fields.Datetime.now()),
        ], order="id")._filter_programs_from_common_rules(self)
        # no impact code...
        # should be programs = programs.filtered if we really want to filter...
        # if self.promo_code:
        #     programs._filter_promo_programs_with_code(self)
        return programs

    def _get_applicable_no_code_promo_program(self):
        self.ensure_one()
        programs = self.env['sale.coupon.program'].with_context(
            no_outdated_coupons=True,
            applicable_coupon=True,
        ).search([
            ('promo_code_usage', '=', 'no_code_needed'),
            '|', ('rule_date_from', '=', False), ('rule_date_from', '<=', fields.Datetime.now()),
            '|', ('rule_date_to', '=', False), ('rule_date_to', '>=', fields.Datetime.now()),
            '|', ('company_id', '=', self.company_id.id), ('company_id', '=', False),
        ])._filter_programs_from_common_rules(self)
        return programs

    def _get_applied_coupon_program_coming_from_another_so(self):
        # TODO: Remove me in master as no more used
        pass

    def _get_valid_applied_coupon_program(self):
        self.ensure_one()
        # applied_coupon_ids's coupons might be coming from:
        #   * a coupon generated from a previous order that benefited from a promotion_program that rewarded the next sale order.
        #     In that case requirements to benefit from the program (Quantity and price) should not be checked anymore
        #   * a coupon_program, in that case the promo_applicability is always for the current order and everything should be checked (filtered)
        programs = self.applied_coupon_ids.mapped('program_id').filtered(lambda p: p.promo_applicability == 'on_next_order')._filter_programs_from_common_rules(self, True)
        programs += self.applied_coupon_ids.mapped('program_id').filtered(lambda p: p.promo_applicability == 'on_current_order')._filter_programs_from_common_rules(self)
        return programs

    def _create_new_no_code_promo_reward_lines(self):
        '''Apply new programs that are applicable'''
        self.ensure_one()
        order = self
        programs = order._get_applicable_no_code_promo_program()
        programs = programs._keep_only_most_interesting_auto_applied_global_discount_program()
        for program in programs:
            # VFE REF in master _get_applicable_no_code_programs already filters programs
            # why do we need to reapply this bunch of checks in _check_promo_code ????
            # We should only apply a little part of the checks in _check_promo_code...
            error_status = program._check_promo_code(order, False)
            if not error_status.get('error'):
                if program.promo_applicability == 'on_next_order':
                    order.state != 'cancel' and order._create_reward_coupon(program)
                elif program.discount_line_product_id.id not in self.order_line.mapped('product_id').ids:
                    self.write({'order_line': [(0, False, value) for value in self._get_reward_line_values(program)]})
                order.no_code_promo_program_ids |= program

    def _update_existing_reward_lines(self):
        '''Update values for already applied rewards'''
        def update_line(order, lines, values):
            '''Update the lines and return them if they should be deleted'''
            lines_to_remove = self.env['sale.order.line']
            # move global coupons to the end of the SO
            if program.discount_apply_on == 'on_order':
                values['sequence'] = max(order.order_line.mapped('sequence')) + 1

            # Check commit 6bb42904a03 for next if/else
            # Remove reward line if price or qty equal to 0
            if values['product_uom_qty'] and values['price_unit']:
                lines.write(values)
            else:
                if program.reward_type != 'free_shipping':
                    # Can't remove the lines directly as we might be in a recordset loop
                    lines_to_remove += lines
                else:
                    values.update(price_unit=0.0)
                    lines.write(values)
            return lines_to_remove

        self.ensure_one()
        order = self
        applied_programs = order._get_applied_programs_with_rewards_on_current_order()
        for program in applied_programs.sorted(lambda ap: (ap.discount_type == 'fixed_amount', ap.discount_apply_on == 'on_order')):
            values = order._get_reward_line_values(program)
            lines = order.order_line.filtered(lambda line: line.product_id == program.discount_line_product_id)
            if program.reward_type == 'discount':
                lines_to_remove = lines
                lines_to_add = []
                lines_to_keep = []
                # Values is what discount lines should really be, lines is what we got in the SO at the moment
                # 1. If values & lines match, we should update the line (or delete it if no qty or price?)
                #    As removing a lines remove all the other lines linked to the same program, we need to save them
                #    using lines_to_keep
                # 2. If the value is not in the lines, we should add it
                # 3. if the lines contains a tax not in value, we should remove it
                for value in values:
                    value_found = False
                    for line in lines:
                        # Case 1.
                        if not len(set(line.tax_id.mapped('id')).symmetric_difference(set([v[1] for v in value['tax_id']]))):
                            value_found = True
                            # Working on Case 3.
                            # update_line update the line to the correct value and returns them if they should be unlinked
                            update_to_remove = update_line(order, line, value)
                            if not update_to_remove:
                                lines_to_keep += [(0, False, value)]
                                lines_to_remove -= line
                    # Working on Case 2.
                    if not value_found:
                        lines_to_add += [(0, False, value)]
                # Case 3.
                line_update = []
                if lines_to_remove:
                    line_update += [(3, line_id, 0) for line_id in lines_to_remove.ids]
                    line_update += lines_to_keep
                line_update += lines_to_add
                order.write({'order_line': line_update})
            else:
                update_line(order, lines, values[0]).unlink()

    def _remove_invalid_reward_lines(self):
        """ Find programs & coupons that are not applicable anymore.
            It will then unlink the related reward order lines.
            It will also unset the order's fields that are storing
            the applied coupons & programs.
            Note: It will also remove a reward line coming from an archive program.
        """
        self.ensure_one()
        order = self

        applied_programs = order._get_applied_programs()
        applicable_programs = self.env['sale.coupon.program']
        if applied_programs:
            applicable_programs = order._get_applicable_programs() + order._get_valid_applied_coupon_program()
            applicable_programs = applicable_programs._keep_only_most_interesting_auto_applied_global_discount_program()
        programs_to_remove = applied_programs - applicable_programs

        reward_product_ids = applied_programs.discount_line_product_id.ids
        # delete reward line coming from an archived coupon (it will never be updated/removed when recomputing the order)
        invalid_lines = order.order_line.filtered(lambda line: line.is_reward_line and line.product_id.id not in reward_product_ids)

        if programs_to_remove:
            product_ids_to_remove = programs_to_remove.discount_line_product_id.ids

            if product_ids_to_remove:
                # Invalid generated coupon for which we are not eligible anymore ('expired' since it is specific to this SO and we may again met the requirements)
                self.generated_coupon_ids.filtered(lambda coupon: coupon.program_id.discount_line_product_id.id in product_ids_to_remove).write({'state': 'expired'})

            # Reset applied coupons for which we are not eligible anymore ('valid' so it can be use on another )
            coupons_to_remove = order.applied_coupon_ids.filtered(lambda coupon: coupon.program_id in programs_to_remove)
            coupons_to_remove.write({'state': 'new'})

            # Unbind promotion and coupon programs which requirements are not met anymore
            order.no_code_promo_program_ids -= programs_to_remove
            order.code_promo_program_id -= programs_to_remove

            if coupons_to_remove:
                order.applied_coupon_ids -= coupons_to_remove

            # Remove their reward lines
            if product_ids_to_remove:
                invalid_lines |= order.order_line.filtered(lambda line: line.product_id.id in product_ids_to_remove)

        invalid_lines.unlink()

    def _get_applied_programs_with_rewards_on_current_order(self):
        # Need to add filter on current order. Indeed, it has always been calculating reward line even if on next order (which is useless and do calculation for nothing)
        # This problem could not be noticed since it would only update or delete existing lines related to that program, it would not find the line to update since not in the order
        # But now if we dont find the reward line in the order, we add it (since we can now have multiple line per  program in case of discount on different vat), thus the bug
        # mentionned ahead will be seen now
        return self.no_code_promo_program_ids.filtered(lambda p: p.promo_applicability == 'on_current_order') + \
               self.applied_coupon_ids.mapped('program_id') + \
               self.code_promo_program_id.filtered(lambda p: p.promo_applicability == 'on_current_order')

    def _get_applied_programs_with_rewards_on_next_order(self):
        return self.no_code_promo_program_ids.filtered(lambda p: p.promo_applicability == 'on_next_order') + \
            self.code_promo_program_id.filtered(lambda p: p.promo_applicability == 'on_next_order')

    def _get_applied_programs(self):
        """Returns all applied programs on current order:

        Expected to return same result than:

            self._get_applied_programs_with_rewards_on_current_order()
            +
            self._get_applied_programs_with_rewards_on_next_order()
        """
        return self.code_promo_program_id + self.no_code_promo_program_ids + self.applied_coupon_ids.mapped('program_id')

    def _get_invoice_status(self):
        # Handling of a specific situation: an order contains
        # a product invoiced on delivery and a promo line invoiced
        # on order. We would avoid having the invoice status 'to_invoice'
        # if the created invoice will only contain the promotion line
        super()._get_invoice_status()
        for order in self.filtered(lambda order: order.invoice_status == 'to invoice'):
            paid_lines = order._get_paid_order_lines()
            if not any(line.invoice_status == 'to invoice' for line in paid_lines):
                order.invoice_status = 'no'

    def _get_invoiceable_lines(self, final=False):
        """ Ensures we cannot invoice only reward lines.

        Since promotion lines are specified with service products,
        those lines are directly invoiceable when the order is confirmed
        which can result in invoices containing only promotion lines.

        To avoid those cases, we allow the invoicing of promotion lines
        iff at least another 'basic' lines is also invoiceable.
        """
        invoiceable_lines = super()._get_invoiceable_lines(final)
        reward_lines = self._get_reward_lines()
        if invoiceable_lines <= reward_lines:
            return self.env['sale.order.line'].browse()
        return invoiceable_lines


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    is_reward_line = fields.Boolean('Is a program reward line')

    def unlink(self):
        related_program_lines = self.env['sale.order.line']
        # Reactivate coupons related to unlinked reward line
        for line in self.filtered(lambda line: line.is_reward_line):
            coupons_to_reactivate = line.order_id.applied_coupon_ids.filtered(
                lambda coupon: coupon.program_id.discount_line_product_id == line.product_id
            )
            coupons_to_reactivate.write({'state': 'new'})
            line.order_id.applied_coupon_ids -= coupons_to_reactivate
            # Remove the program from the order if the deleted line is the reward line of the program
            # And delete the other lines from this program (It's the case when discount is split per different taxes)
            related_program = self.env['sale.coupon.program'].search([('discount_line_product_id', '=', line.product_id.id)])
            if related_program:
                line.order_id.no_code_promo_program_ids -= related_program
                line.order_id.code_promo_program_id -= related_program
                related_program_lines |= line.order_id.order_line.filtered(lambda l: l.product_id.id == related_program.discount_line_product_id.id) - line
        return super(SaleOrderLine, self | related_program_lines).unlink()

    def _compute_tax_id(self):
        reward_lines = self.filtered('is_reward_line')
        super(SaleOrderLine, self - reward_lines)._compute_tax_id()
        # Discount reward line is split per tax, the discount is set on the line but not on the product
        # as the product is the generic discount line.
        # In case of a free product, retrieving the tax on the line instead of the product won't affect the behavior.
        for line in reward_lines:
            fpos = line.order_id.fiscal_position_id or line.order_id.partner_id.property_account_position_id
            # If company_id is set, always filter taxes by the company
            taxes = line.tax_id.filtered(lambda r: not line.company_id or r.company_id == line.company_id)
            line.tax_id = fpos.map_tax(taxes, line.product_id, line.order_id.partner_shipping_id) if fpos else taxes

    def _get_display_price(self, product):
        # A product created from a promotion does not have a list_price.
        # The price_unit of a reward order line is computed by the promotion, so it can be used directly
        if self.is_reward_line:
            return self.price_unit
        return super()._get_display_price(product)

    # Invalidation of `sale.coupon.program.order_count`
    # `test_program_rules_validity_dates_and_uses`,
    # Overriding modified is quite hardcore as you need to know how works the cache and the invalidation system,
    # but at least the below works and should be efficient.
    # Another possibility is to add on product.product a one2many to sale.order.line 'order_line_ids',
    # and then add the depends @api.depends('discount_line_product_id.order_line_ids'),
    # but I am not sure this will as efficient as the below.
    def modified(self, fnames, *args, **kwargs):
        super(SaleOrderLine, self).modified(fnames, *args, **kwargs)
        if 'product_id' in fnames:
            Program = self.env['sale.coupon.program'].sudo()
            field_order_count = Program._fields['order_count']
            programs = self.env.cache.get_records(Program, field_order_count)
            if programs:
                products = self.filtered('is_reward_line').mapped('product_id')
                for program in programs:
                    if program.discount_line_product_id in products:
                        self.env.cache.invalidate([(field_order_count, program.ids)])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_coupon
from . import sale_coupon_reward
from . import sale_coupon_rules
from . import sale_coupon_program
from . import sale_order

```

## File: report\sale_coupon_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class CouponReport(models.AbstractModel):
    _name = 'report.sale_coupon.report_coupon'
    _description = 'Sales Coupon Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['sale.coupon'].browse(docids)
        return {
            'doc_ids': docs.ids,
            'doc_model': 'sale.coupon',
            'data': data,
            'docs': docs,
        }

```

## File: report\sale_coupon_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <report
            id="report_coupon_code" 
            model="sale.coupon"
            name="sale_coupon.report_coupon_i18n"
            file="sale_coupon.report_coupon_i18n"
            report_type="qweb-pdf"
            string="Coupon Code"/>
    </data>
</odoo>

```

## File: report\sale_coupon_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_coupon">
                    <t t-call="web.internal_layout">
                    <div class="card">
                    <div class="card-body">
                    <div class="page">
                        <div class="row text-center">
                            <div class="o_offer col-lg-12">
                                <h4 t-if="o.partner_id.name">
                                    Congratulations
                                    <t t-esc="o.partner_id.name"/>,
                                </h4>
                                <t t-set="text">OFF ON YOUR NEXT ORDER!</t>
                                <t t-if="o.program_id.reward_type == 'discount'">
                                    <h1 t-if="o.program_id.discount_type == 'fixed_amount'" class="text-success">
                                        <strong><span t-field="o.program_id.discount_fixed_amount" t-options='{"widget": "monetary", "display_currency": o.program_id.currency_id}'/></strong>
                                    </h1>
                                    <h1 t-if="o.program_id.discount_type == 'percentage'" class="text-success">
                                        <strong><span t-field="o.program_id.discount_percentage"/>%</strong>
                                    </h1>
                                    <t t-if="o.program_id.discount_apply_on == 'specific_products'">
                                        <t t-if="len(o.program_id.discount_specific_product_ids) > 1">
                                            <t t-set="text">OFF ON SOME PRODUCTS*</t>
                                            <t t-set="display_specific_products" t-value="True"/>
                                        </t>
                                        <t t-else="">
                                            <t t-set="text">OFF ON %s</t>
                                            <t t-set="text" t-value="text % o.program_id.discount_specific_product_ids.name.upper()"/>
                                        </t>
                                    </t>
                                    <t t-if="o.program_id.discount_apply_on == 'cheapest_product'">
                                        <t t-set="text">OFF ON THE CHEAPEST PRODUCT</t>
                                    </t>
                                </t>
                                <t t-if="o.program_id.reward_type == 'product'">
                                    <t t-set="text">GET %s FREE %s ON YOUR NEXT ORDER!</t>
                                    <t t-set="text" t-value="text % (o.program_id.reward_product_quantity, o.program_id.reward_product_id.name.upper())"/>
                                </t>
                                <t t-if="o.program_id.reward_type == 'free_shipping'">
                                    <t t-set="text">GET FREE SHIPPING ON YOUR NEXT ORDER!</t>
                                </t>
                                <h3 t-esc="text"/>
                                <h3 t-if="o.expiration_date">
                                    Use this promo code before
                                    <strong t-field="o.expiration_date" t-options='{"format": "d MMMM y"}'/>
                                </h3>
                                <h3>
                                    CODE:
                                    <strong class="bg-success" t-esc="o.code"></strong>
                                </h3>
                                <h4 t-if="o.program_id.rule_min_quantity > 1">
                                    Minimum purchase of
                                    <strong t-esc="o.program_id.rule_min_quantity"/> products
                                </h4>
                                <h4 t-if="o.program_id.rule_minimum_amount">
                                    Valid for purchase above
                                    <strong t-esc="o.program_id.rule_minimum_amount" t-options="{'widget': 'monetary', 'display_currency': o.program_id.currency_id}"/>
                                </h4>
                                <p t-if="display_specific_products">
                                    <small>
                                        *Valid for following products: <t t-esc="', '.join(o.program_id.discount_specific_product_ids.mapped('name')).upper()"/>
                                    </small>
                                </p>
                                <br/>
                                <img alt="Barcode" t-att-src="'/report/barcode/Code128/%s' % o.code"/>
                                <br/>
                                <div class="mt64">
                                    <div class="col-6 text-right">
                                    <img alt="Logo" t-att-src="'/logo?company=%d' % (o.program_id.company_id)" t-att-alt="'%s' % (o.program_id.company_id.name)" style="border:0px solid transparent; height: 50; width: 200px;" height="50"/>
                                    </div>
                                    <div class="col-6 text-left">
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

    <template id="report_coupon_i18n">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang or o.env.lang)"/>
                <t t-call="sale_coupon.report_coupon" t-lang="o.partner_id.lang or o.env.lang"/>
            </t>
        </t>
    </template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_coupon_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_program_salesman,salesman,model_sale_coupon_program,sales_team.group_sale_salesman,1,0,0,0
access_program_manager,program manager,model_sale_coupon_program,sales_team.group_sale_manager,1,1,1,1
access_applicability_salesman,salesman,model_sale_coupon_rule,sales_team.group_sale_salesman,1,0,0,0
access_applicability_manager,program manager,model_sale_coupon_rule,sales_team.group_sale_manager,1,1,1,0
access_coupon_salesman,salesman,model_sale_coupon,sales_team.group_sale_salesman,1,1,0,0
access_coupon_manager,program manager,model_sale_coupon,sales_team.group_sale_manager,1,1,1,0
access_reward_salesman,salesman,model_sale_coupon_reward,sales_team.group_sale_salesman,1,0,0,0
access_reward_manager,program manager,model_sale_coupon_reward,sales_team.group_sale_manager,1,1,1,0

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.coupon</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='sale_coupon']" position="after">
                <div class="content-group">
                    <div class="mt8" attrs="{'invisible': [('module_sale_coupon', '=', False)]}">
                        <button name="%(sale_coupon.sale_coupon_program_action_promo_program)d" icon="fa-arrow-right" type="action" string="Promotion Programs" class="btn-link"/>
                    </div>
                    <div class="mt8" attrs="{'invisible': [('module_sale_coupon', '=', False)]}">
                        <button name="%(sale_coupon.sale_coupon_program_action_coupon_program)d" icon="fa-arrow-right" type="action" string="Coupon Programs" class="btn-link"/>
                    </div>
                 </div>
             </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_coupon_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Common form view between the coupon programs and the promotion programs -->
    <record id="sale_coupon_program_view_form_common" model="ir.ui.view">
        <field name="name">sale.coupon.program.common.form</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <form string="Coupon Program">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" type="object" icon="fa-usd" name="action_view_sales_orders">
                            <field name="order_count" string="Sales" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div name="title" class="oe_left">
                        <label class="oe_edit_only" for="name" string="Program Name"/>
                    </div>
                    <group>
                        <group name="conditions" string="Conditions">
                            <field name="active" invisible="1"/>
                            <field name="program_type" invisible="1"/>
                            <field name="rule_products_domain" placeholder="Select product" widget="domain" options="{'model': 'product.product', 'in_dialog': true}"/>
                            <label string="Quantity" for="rule_min_quantity" attrs="{'invisible': [('rule_products_domain', '=', False)]}"/>
                            <div attrs="{'invisible': [('rule_products_domain', '=',False)]}">
                                <field name="rule_min_quantity" class="oe_inline"/>
                            </div>
                            <label string="Minimum Purchase Of" for="rule_minimum_amount" />
                            <div name="rule_minimum_amount" class="o_row">
                                <field name="currency_id" invisible="1"/>
                                <field name="rule_minimum_amount" widget='monetary' options="{'currency_field': 'currency_id'}"/>
                                <field name="rule_minimum_amount_tax_inclusion" required="1"/>
                            </div>
                            <field name="company_id" placeholder="Select company" groups="base.group_multi_company" required="1"></field>
                        </group>
                        <group name="validity" string="Validity"/>
                    </group>
                    <group string="Rewards">
                        <group name='reward'>
                            <field name="reward_type" string="Reward" widget="radio"/>
                            <field name="discount_line_product_id" attrs="{'invisible': [('discount_line_product_id', '=', False)]}" readonly="True"/>
                        </group>
                        <group>
                            <field name="reward_product_id" attrs="{'invisible': [('reward_type', 'in', ('discount', 'free_shipping'))], 'required': [('reward_type', '=', 'product')]}" placeholder="Select reward product"/>
                            <label string="Quantity" for="reward_product_quantity" attrs="{'invisible': ['|', ('reward_type', 'in', ('discount', 'free_shipping')), ('reward_product_id', '=',False)]}"/>
                            <div attrs="{'invisible': ['|', ('reward_type', 'in', ('discount', 'free_shipping')),('reward_product_id', '=',False)]}">
                                <field name="reward_product_quantity" class="oe_inline"/>
                                <field name="reward_product_uom_id" class="oe_inline"/>
                            </div>
                            <label string="Apply Discount" for="discount_type" attrs="{'invisible': [('reward_type', 'in', ('product', 'free_shipping'))]}"/>
                            <div attrs="{'invisible': [('reward_type', 'in', ('product', 'free_shipping'))]}">
                                <field name="discount_type" class="oe_inline" attrs="{'required': [('reward_type','=','discount')]}"/>
                                <field name="discount_percentage" attrs="{'invisible': [('discount_type', '!=', 'percentage')],'required': [('discount_type', '=', 'percentage')]}" class="oe_inline"/>
                                <span attrs="{'invisible': [('discount_type', '!=', 'percentage')],'required': [('discount_type', '=', 'percentage')]}" class="oe_inline">%</span>
                            </div>
                            <label string="Fixed Amount" for="discount_fixed_amount" attrs="{'invisible': ['|',('discount_type', '!=', 'fixed_amount'), ('reward_type', '!=', 'discount')]}" />
                            <div attrs="{'invisible': ['|',('discount_type', '!=', 'fixed_amount'), ('reward_type', '!=', 'discount')]}">
                                <field name="discount_fixed_amount" class="oe_inline" attrs="{'required':[('discount_type', '=', 'fixed_amount')]}" widget='monetary' options="{'currency_field': 'currency_id'}"/>
                            </div>
                            <field name="discount_apply_on" attrs="{'invisible':
                            ['|', ('reward_type', 'in', ('product', 'free_shipping')), ('discount_type', '!=', 'percentage')]}" widget="radio"/>
                            <field name="discount_specific_product_ids" widget='many2many_tags' attrs="{'invisible': ['|', '|', ('discount_apply_on', '!=', 'specific_products'),('discount_type', '!=', 'percentage'), ('reward_type', 'in', ('product', 'free_shipping'))], 'required': [('reward_type', '=', 'discount'),('discount_apply_on', '=', 'specific_products'),('discount_type', '=', 'percentage')]}" placeholder="Select products"/>
                            <label for="discount_max_amount" string="Max Discount Amount" attrs="{'invisible': ['|', ('reward_type', 'in', ('product', 'free_shipping')), ('discount_type', '!=', 'percentage')]}"/>
                            <div attrs="{'invisible': ['|', ('reward_type', 'in', ('product', 'free_shipping')), ('discount_type', '!=', 'percentage')]}">
                                <field name="discount_max_amount" class="oe_inline" widget='monetary' options="{'currency_field': 'currency_id'}"/>
                                <span class="oe_grey">if 0, no limit</span>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <!-- Coupon Program -->
    <record id="sale_coupon_program_view_form" model="ir.ui.view">
        <field name="name">sale.coupon.program.form</field>
        <field name="model">sale.coupon.program</field>
        <field name="inherit_id" ref="sale_coupon_program_view_form_common"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="before">
                <header>
                    <button type="action" name="%(sale_coupon.sale_coupon_generate_action)d"
                            string="Generate Coupon" attrs="{'invisible': [('active', '=', False)]}"/>
                    <button type="action" name="%(sale_coupon.sale_coupon_generate_action)d"
                            string="Generate Coupon" attrs="{'invisible': [('active', '=', True)]}" class="oe_highlight"/>
                </header>
            </xpath>
            <xpath expr="//field[@name='order_count']/.." position="before">
                <button class="oe_stat_button" type="action" icon="fa-ticket" name="%(sale_coupon.sale_coupon_action)d">
                    <field name="coupon_count" string="Coupons" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//div[@name='title']" position="inside">
                <h1><field name="name" class="oe_title" placeholder="Coupon Program Name..." height="20px"/></h1>
            </xpath>
            <xpath expr="//group[@name='validity']" position="inside">
                <label for="validity_duration" string="Validity Duration"/>
                <div>
                    <field name="validity_duration" class="oe_inline"/>
                    <span class="o_form_label oe_inline"> Days</span> <span class="oe_grey">if 0, infinite use</span>
                </div>
            </xpath>
        </field>
    </record>

    <record id="sale_coupon_program_view_tree" model="ir.ui.view">
        <field name="name">sale.coupon.program.tree</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <tree>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="active"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="sale_coupon_program_view_search" model="ir.ui.view">
        <field name="name">sale.coupon.program.search</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Expired" name="expired" domain="[('rule_date_to', '&lt;', datetime.datetime.now())]" help="Expired Programs"/>
                <separator/>
                <filter string="Active" name="active" domain="[('active', '=', True)]" help="Opened Programs"/>
                <filter string="Closed" name="closed" domain="[('active', '=', False)]" help="Closed Programs"/>
            </search>
        </field>
    </record>

    <record id="view_sale_coupon_program_kanban" model="ir.ui.view">
        <field name="name">sale.coupon.program.kanban</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
               <field name="name" />
                <field name="active"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                            <div class="text-center">
                               <strong><span><field name="name"/></span></strong>
                            </div>
                            <hr class="mt4 mb4"/>
                            <div class="row">
                                <div class="col-4 text-center"><strong>Coupons</strong></div>
                                <div class="col-4 text-center"><strong>Sales</strong></div>
                                <div class="col-4 text-center"><strong>Active</strong></div>
                                <div class="col-4 text-center"><field name="coupon_count"/></div>
                                <div class="col-4 text-center"><field name="order_count"/></div>
                                <div class="col-4 text-center">
                                    <field name="active" widget="boolean"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="sale_coupon_program_action_coupon_program" model="ir.actions.act_window">
        <field name="name">Coupon Programs</field>
        <field name="res_model">sale.coupon.program</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="sale_coupon_program_view_search"/>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'tree'}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('sale_coupon_program_view_form')})]"/>
        <field name="domain">[('program_type','=', 'coupon_program')]</field>
        <field name="context">{
            'default_program_type': 'coupon_program',
            'promo_code_usage': 'code_needed',
            'search_default_opened': 1
            }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new coupon program
            </p><p>
                Generate and share coupon codes with your customers to get discounts or free products.
             </p>
        </field>
    </record>

    <menuitem id="menu_coupon_type_config" action="sale_coupon_program_action_coupon_program" parent="sale.product_menu_catalog" name="Coupon Programs" groups="sales_team.group_sale_manager" sequence="5"/>


    <!-- Promotion Program -->

    <record id="sale_coupon_program_view_promo_program_form" model="ir.ui.view">
        <field name="name">sale.coupon.promotion.program.form</field>
        <field name="model">sale.coupon.program</field>
        <field name="inherit_id" ref="sale_coupon_program_view_form_common"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='title']" position="inside">
                <h1><field name="name" class="oe_title" placeholder="Promotion Program Name..." height="20px"/></h1>
            </xpath>
            <xpath expr="//field[@name='order_count']/.." position="before">
                <button class="oe_stat_button" type="action" icon="fa-ticket" name="%(sale_coupon.sale_coupon_action)d" attrs="{'invisible': [('promo_applicability', '=', 'on_current_order')]}">
                    <field name="coupon_count" string="Coupons" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//group[@name='reward']" position="before">
                <field name="sequence" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='program_type']" position="after">
                <field name="rule_partners_domain" placeholder="Select customer" widget="domain" options="{'model': 'res.partner', 'in_dialog': true}"/>
            </xpath>
            <xpath expr="//div[@name='rule_minimum_amount']" position="after">
                <field name="promo_code_usage" widget="radio"/>
                <field name="promo_code" attrs="{'required': [('promo_code_usage', '=', 'code_needed')], 'invisible': [('promo_code_usage', '=', 'no_code_needed')]}"/>
            </xpath>
            <xpath expr="//group[@name='validity']" position="inside">
                <label string="Apply on First" for="maximum_use_number" class="oe_inline"/>
                <div>
                    <field name="maximum_use_number" class="oe_inline"/>
                    <span> Orders</span>
                    <span class="oe_grey"> if 0, infinite use</span>
                </div>
                <field name="rule_date_from" class="oe_inline"/>
                <field name="rule_date_to" class="oe_inline"/>
            </xpath>
            <xpath expr="//group[@name='reward']" position="before">
                <group>
                    <field name="promo_applicability" widget="radio"/>
                </group>
                <group>
                    <span class="oe_grey">
                        <b>Apply on Current Order -</b> Reward will be applied on current order.<br/>
                        <b>Apply on Next Order -</b> Generate a coupon for a next order.
                    </span>
                </group>
            </xpath>
        </field>
    </record>

    <record id="sale_coupon_program_view_promo_program_tree" model="ir.ui.view">
        <field name="name">sale.coupon.promotion.program.tree</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <tree>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="promo_code_usage"/>
                <field name="active"/>
            </tree>
        </field>
    </record>

    <record id="sale_coupon_program_view_promo_program_search" model="ir.ui.view">
        <field name="name">sale.coupon.promotion.program.search</field>
        <field name="model">sale.coupon.program</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Active" name="active" domain="[('active', '=', True)]" help="Opened Programs"/>
                <filter string="Closed" name="closed" domain="[('active', '=', False)]" help="Closed Programs"/>
            </search>
        </field>
    </record>

    <record id="sale_coupon_program_action_promo_program" model="ir.actions.act_window">
        <field name="name">Promotion Programs</field>
        <field name="res_model">sale.coupon.program</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'tree'}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('sale_coupon_program_view_promo_program_form')})]"/>
        <field name="search_view_id" ref="sale_coupon_program_view_promo_program_search"/>
        <field name="domain">[('program_type', '=', 'promotion_program')]</field>
        <field name="context">{
            'default_program_type': 'promotion_program',
            'default_promo_code_usage': 'no_code_needed',
            'default_validity_duration': 0,
            'search_default_opened': 1
            }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new promotion program
            </p><p>
                Build up promotion programs to attract more customers with discounts, free products, free delivery, etc.
                You can share promotion codes or grant the promotions automatically if some conditions are met.
             </p>
        </field>
    </record>

    <menuitem action="sale_coupon_program_action_promo_program" id="menu_promotion_type_config" parent="sale.product_menu_catalog" name="Promotion Programs" groups="sales_team.group_sale_manager" sequence="4"/>

</odoo>

```

## File: views\sale_coupon_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_coupon_view_tree" model="ir.ui.view">
        <field name="name">sale.coupon.tree</field>
        <field name="model">sale.coupon</field>
        <field name="arch" type="xml">
            <tree string="Coupons" create="false" edit="false" delete="false">
                <field name="code"/>
                <field name="expiration_date"/>
                <field name="program_id"/>
                <field name="partner_id"/>
                <field name="order_id"/>
                <field name="state"/>
            </tree>
        </field>
    </record>

    <record id="sale_coupon_action" model="ir.actions.act_window">
        <field name="name">Coupons</field>
        <field name="res_model">sale.coupon</field>
        <field name="view_id" ref="sale_coupon_view_tree"/>
        <field name="domain">[('program_id', '=', active_id)]</field>
        <field name="context">{}</field>
    </record>


    <record id="sale_coupon_view_form" model="ir.ui.view">
        <field name="name">sale.coupon.form</field>
        <field name="model">sale.coupon</field>
        <field name="arch" type="xml">
            <form string="Coupons" create="false" edit="false" delete="false">
                <header>
                     <button name="action_coupon_sent" type="object" string="Send by Email" class="oe_highlight"/>
                    <field name="state" widget="statusbar" statusbar_visible="new,used,expired" context="{'state': state}"/>
                </header>
                <sheet>
                    <group>
                        <field name="code"/>
                        <field name="expiration_date"/>
                        <field name="partner_id"/>
                        <field name="order_id"/>
                        <field name="sales_order_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
   <record id="sale_order_view_form" model="ir.ui.view">
         <field name="name">sale.order.form.inherit.coupon</field>
         <field name="model">sale.order</field>
         <field name="inherit_id" ref="sale.view_order_form" />
         <field name="priority">10</field>
         <field name="arch" type="xml">
            <xpath expr="//group[@name='note_group']" position="before">
                <div class="oe_right">
                    <button name="%(sale_coupon.sale_coupon_apply_code_action)d" class="btn btn-secondary"
                            string="Coupon" type="action" groups="base.group_user" states="draft,sent,sale"/>
                    <button name="recompute_coupon_lines" class="btn btn-secondary" string="Promotions"
                            help="When clicked, the content of the order will be checked to detect (and apply) possible promotion programs."
                            type="object" states="draft,sent,sale"/>
                </div>
            </xpath>
         </field>
    </record>

    <record id="sale_order_action" model="ir.actions.act_window">
        <field name="name">Sales Order</field>
        <field name="res_model">sale.order</field>
        <field name="view_id" ref="sale.view_order_tree"/>
        <field name="domain">[('state', '!=', 'cancel')]</field>
        <field name="context">{}</field>
    </record>
</odoo>

```

## File: wizard\sale_coupon_apply_code.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class SaleCouponApplyCode(models.TransientModel):
    _name = 'sale.coupon.apply.code'
    _rec_name = 'coupon_code'
    _description = 'Sales Coupon Apply Code'

    coupon_code = fields.Char(string="Code", required=True)

    def process_coupon(self):
        """
        Apply the entered coupon code if valid, raise an UserError otherwise.
        """
        sales_order = self.env['sale.order'].browse(self.env.context.get('active_id'))
        error_status = self.apply_coupon(sales_order, self.coupon_code)
        if error_status.get('error', False):
            raise UserError(error_status.get('error', False))
        if error_status.get('not_found', False):
            raise UserError(error_status.get('not_found', False))

    def apply_coupon(self, order, coupon_code):
        error_status = {}
        program_domain = order._get_coupon_program_domain()
        program_domain = expression.AND([program_domain, [('promo_code', '=', coupon_code)]])
        program = self.env['sale.coupon.program'].search(program_domain)
        if program:
            error_status = program._check_promo_code(order, coupon_code)
            if not error_status:
                if program.promo_applicability == 'on_next_order':
                    # Avoid creating the coupon if it already exist
                    if program.discount_line_product_id.id not in order.generated_coupon_ids.filtered(lambda coupon: coupon.state in ['new', 'reserved']).mapped('discount_line_product_id').ids:
                        coupon = order._create_reward_coupon(program)
                        return {
                            'generated_coupon': {
                                'reward': coupon.program_id.discount_line_product_id.name,
                                'code': coupon.code,
                            }
                        }
                else:  # The program is applied on this order
                    # Only link the promo program if reward lines were created
                    order_line_count = len(order.order_line)
                    order._create_reward_line(program)
                    if order_line_count < len(order.order_line):
                        order.code_promo_program_id = program
        else:
            coupon = self.env['sale.coupon'].search([('code', '=', coupon_code)], limit=1)
            if coupon:
                error_status = coupon._check_coupon_code(order)
                if not error_status:
                    # Consume coupon only if reward lines were created
                    order_line_count = len(order.order_line)
                    order._create_reward_line(coupon.program_id)
                    if order_line_count < len(order.order_line):
                        order.applied_coupon_ids += coupon
                        coupon.write({'state': 'used'})
            else:
                error_status = {'not_found': _('The code %s is invalid') % (coupon_code)}
        return error_status

```

## File: wizard\sale_coupon_apply_code_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_coupon_apply_code_view_form" model="ir.ui.view">
        <field name="name">sale.coupon.apply.code.form</field>
        <field name="model">sale.coupon.apply.code</field>
        <field name="arch" type="xml">
            <form string="Apply coupon">
                <group>
                    <group>
                        <field name="coupon_code"/>
                    </group>
                </group>
                <footer>
                    <button name="process_coupon" string="Apply" type="object" class="oe_highlight"/>
                    <button special="cancel" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sale_coupon_apply_code_action" model="ir.actions.act_window">
        <field name="name">Enter Promotion or Coupon Code</field>
        <field name="res_model">sale.coupon.apply.code</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="sale_coupon_apply_code_view_form"/>
     </record>
</odoo>

```

## File: wizard\sale_coupon_generate.py

```python
# -*- coding: utf-8 -*-

from odoo import _, api, fields, models
from odoo.tools.safe_eval import safe_eval


class SaleCouponGenerate(models.TransientModel):
    _name = 'sale.coupon.generate'
    _description = 'Generate Sales Coupon'

    nbr_coupons = fields.Integer(string="Number of Coupons", help="Number of coupons", default=1)
    generation_type = fields.Selection([
        ('nbr_coupon', 'Number of Coupons'),
        ('nbr_customer', 'Number of Selected Customers')
        ], default='nbr_coupon')
    partners_domain = fields.Char(string="Customer", default='[]')

    def generate_coupon(self):
        """Generates the number of coupons entered in wizard field nbr_coupons
        """
        program = self.env['sale.coupon.program'].browse(self.env.context.get('active_id'))

        vals = {'program_id': program.id}

        if self.generation_type == 'nbr_coupon' and self.nbr_coupons > 0:
            for count in range(0, self.nbr_coupons):
                self.env['sale.coupon'].create(vals)

        if self.generation_type == 'nbr_customer' and self.partners_domain:
            for partner in self.env['res.partner'].search(safe_eval(self.partners_domain)):
                vals.update({'partner_id': partner.id})
                coupon = self.env['sale.coupon'].create(vals)
                context = dict(lang=partner.lang)
                subject = _('%s, a coupon has been generated for you') % (partner.name)
                del context
                template = self.env.ref('sale_coupon.mail_template_sale_coupon', raise_if_not_found=False)
                if template:
                    template.send_mail(coupon.id, email_values={'email_from': self.env.user.email or '', 'subject': subject})

```

## File: wizard\sale_coupon_generate_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_coupon_generate_view_form" model="ir.ui.view">
        <field name="name">sale.coupon.generate.form</field>
        <field name="model">sale.coupon.generate</field>
        <field name="arch" type="xml">
            <form string="Generate Coupons">
                <group>
                    <field name="generation_type" widget="radio"/>
                    <field name="partners_domain" attrs="{'invisible': [('generation_type', '!=', 'nbr_customer')]}" widget="domain" options="{'model': 'res.partner'}"/>
                    <field name="nbr_coupons" attrs="{'invisible': [('generation_type', '!=', 'nbr_coupon')]}"/>
                </group>
                <footer>
                    <button name="generate_coupon" type="object" string="Generate" class="oe_highlight"/>
                    <button special="cancel" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sale_coupon_generate_action" model="ir.actions.act_window">
        <field name="name">Number of Coupons To Generate</field>
        <field name="res_model">sale.coupon.generate</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="sale_coupon_generate_view_form"/>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_coupon_apply_code
from . import sale_coupon_generate

```

