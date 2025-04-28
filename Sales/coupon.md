# Odoo Module: coupon

Category: Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Coupon",
    'summary': "Use discount coupons in different sales channels.",
    'description': """Integrate coupon mechanism in orders.""",
    'category': 'Sales',
    'version': '1.0',
    'depends': ['account'],
    'data': [
        'wizard/coupon_generate_views.xml',
        'security/ir.model.access.csv',
        'security/coupon_security.xml',
        'views/coupon_views.xml',
        'views/coupon_program_views.xml',
        'report/coupon_report.xml',
        'report/coupon_report_templates.xml',
        'data/coupon_email_data.xml',
    ],
    'demo': [
        'demo/coupon_demo.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\coupon_email_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
   <data noupdate="1">
      <record id="coupon.mail_template_sale_coupon" model="mail.template">
         <field name="name">Coupon: Send by Email</field>
         <field name="model_id" ref="coupon.model_coupon_coupon"/>
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
        % if object.order_id.user_id.signature:
            <br />
            ${object.order_id.user_id.signature | safe}
        % endif
    </td></tr>
</tbody></table>
            </field>
            <field name="report_template" ref="report_coupon_code"/>
            <field name="report_name">Your Coupon Code</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
      </record>

        <record id="expire_coupon_cron" model="ir.cron">
            <field name="name">Coupon: expire coupon based on date</field>
            <field name="model_id" ref="coupon.model_coupon_coupon"/>
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

## File: models\coupon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import random
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _

from uuid import uuid4

class Coupon(models.Model):
    _name = 'coupon.coupon'
    _description = "Coupon"
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
        return str(uuid4())[:22]

    code = fields.Char(default=_generate_code, required=True, readonly=True)
    expiration_date = fields.Date('Expiration Date', compute='_compute_expiration_date')
    state = fields.Selection([
        ('reserved', 'Pending'),
        ('new', 'Valid'),
        ('sent', 'Sent'),
        ('used', 'Used'),
        ('expired', 'Expired'),
        ('cancel', 'Cancelled')
    ], required=True, default='new')
    partner_id = fields.Many2one('res.partner', "For Customer")
    program_id = fields.Many2one('coupon.program', "Program")
    discount_line_product_id = fields.Many2one('product.product', related='program_id.discount_line_product_id', readonly=False,
        help='Product used in the sales order to apply the discount.')

    _sql_constraints = [
        ('unique_coupon_code', 'unique(code)', 'The coupon code must be unique!'),
    ]

    @api.depends('create_date', 'program_id.validity_duration')
    def _compute_expiration_date(self):
        self.expiration_date = 0
        for coupon in self.filtered(lambda x: x.program_id.validity_duration > 0):
            coupon.expiration_date = (coupon.create_date + relativedelta(days=coupon.program_id.validity_duration)).date()

    def action_coupon_sent(self):
        """ Open a window to compose an email, with the edi invoice template
            message loaded by default
        """
        self.ensure_one()
        template = self.env.ref('coupon.mail_template_sale_coupon', False)
        compose_form = self.env.ref('mail.email_compose_message_wizard_form', False)
        ctx = dict(
            default_model='coupon.coupon',
            default_res_id=self.id,
            default_use_template=bool(template),
            default_template_id=template.id,
            default_composition_mode='comment',
            custom_layout='mail.mail_notification_light',
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

    def action_coupon_cancel(self):
        self.state = 'cancel'

    def cron_expire_coupon(self):
        self._cr.execute("""
            SELECT C.id FROM COUPON_COUPON as C
            INNER JOIN COUPON_PROGRAM as P ON C.program_id = P.id
            WHERE C.STATE in ('reserved', 'new', 'sent')
                AND P.validity_duration > 0
                AND C.create_date + interval '1 day' * P.validity_duration < now()""")

        expired_ids = [res[0] for res in self._cr.fetchall()]
        self.browse(expired_ids).write({'state': 'expired'})

```

## File: models\coupon_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError

import ast


class CouponProgram(models.Model):
    _name = 'coupon.program'
    _description = "Coupon Program"
    _inherits = {'coupon.rule': 'rule_id', 'coupon.reward': 'reward_id'}
    # We should apply 'discount' promotion first to avoid offering free product when we should not.
    # Eg: If the discount lower the SO total below the required threshold
    # Note: This is only revelant when programs have the same sequence (which they have by default)
    _order = "sequence, reward_type"

    name = fields.Char(required=True, translate=True)
    active = fields.Boolean('Active', default=True, help="A program is available for the customers when active")
    rule_id = fields.Many2one('coupon.rule', string="Coupon Rule", ondelete='restrict', required=True)
    reward_id = fields.Many2one('coupon.reward', string="Reward", ondelete='restrict', required=True, copy=False)
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
        ('on_next_order', 'Send a Coupon')],
        default='on_current_order', string="Applicability")
    coupon_ids = fields.One2many('coupon.coupon', 'program_id', string="Generated Coupons", copy=False)
    coupon_count = fields.Integer(compute='_compute_coupon_count')
    company_id = fields.Many2one('res.company', string="Company", default=lambda self: self.env.company)
    currency_id = fields.Many2one(string="Currency", related='company_id.currency_id', readonly=True)
    validity_duration = fields.Integer(default=30,
        help="Validity duration for a coupon after its generation")

    @api.constrains('promo_code')
    def _check_promo_code_constraint(self):
        """ Program code must be unique """
        for program in self.filtered(lambda p: p.promo_code):
            domain = [('id', '!=', program.id), ('promo_code', '=', program.promo_code)]
            if self.search(domain):
                raise ValidationError(_('The program code must be unique!'))

    @api.depends('coupon_ids')
    def _compute_coupon_count(self):
        coupon_data = self.env['coupon.coupon'].read_group([('program_id', 'in', self.ids)], ['program_id'], ['program_id'])
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
        program = super(CouponProgram, self).create(vals)
        if not vals.get('discount_line_product_id', False):
            values = program._get_discount_product_values()
            discount_line_product_id = self.env['product.product'].create(values)
            program.write({'discount_line_product_id': discount_line_product_id.id})
        return program

    def write(self, vals):
        res = super(CouponProgram, self).write(vals)
        if not self:
            return res
        reward_fields = [
            'reward_type', 'reward_product_id', 'discount_type', 'discount_percentage',
            'discount_apply_on', 'discount_specific_product_ids', 'discount_fixed_amount'
        ]
        if any(field in reward_fields for field in vals):
            for program in self:
                program.discount_line_product_id.write({'name': program.reward_id.display_name})
        return res

    def unlink(self):
        if self.filtered('active'):
            raise UserError(_('You can not delete a program in active state'))
        # get reference to rule and reward
        rule = self.rule_id
        reward = self.reward_id
        # unlink the program
        super(CouponProgram, self).unlink()
        # then unlink the rule and reward
        rule.unlink()
        reward.unlink()
        return True

    def toggle_active(self):
        super(CouponProgram, self).toggle_active()
        for program in self:
            program.discount_line_product_id.active = program.active
        coupons = self.filtered(lambda p: not p.active and p.promo_code_usage == 'code_needed').mapped('coupon_ids')
        coupons.filtered(lambda x: x.state != 'used').write({'state': 'expired'})

    def _compute_program_amount(self, field, currency_to):
        self.ensure_one()
        return self.currency_id._convert(self[field], currency_to, self.company_id, fields.Date.today())

    def _is_valid_partner(self, partner):
        if self.rule_partners_domain and self.rule_partners_domain != '[]':
            domain = ast.literal_eval(self.rule_partners_domain) + [('id', '=', partner.id)]
            return bool(self.env['res.partner'].search_count(domain))
        else:
            return True

    def _is_valid_product(self, product):
        """Check if the given product is valid for the program.

        :param product: record of product.product
        :rtype: bool
        """
        return bool(self._get_valid_products(product))

    def _get_valid_products(self, products):
        """Get valid products for the program.

        :param products: records of product.product
        :return: valid products recordset
        """
        if self.rule_products_domain and self.rule_products_domain != "[]":
            domain = ast.literal_eval(self.rule_products_domain)
            return products.filtered_domain(domain)
        return products

    def _get_discount_product_values(self):
        return {
            'name': self.reward_id.display_name,
            'type': 'service',
            'taxes_id': False,
            'supplier_taxes_id': False,
            'sale_ok': False,
            'purchase_ok': False,
            'lst_price': 0, #Do not set a high value to avoid issue with coupon code
        }

```

## File: models\coupon_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class CouponReward(models.Model):
    _name = 'coupon.reward'
    _description = "Coupon Reward"
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
        help='The discount in percentage, between 1 and 100')
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
                reward_string = _("Free Product - %s", reward.reward_product_id.name)
            elif reward.reward_type == 'discount':
                if reward.discount_type == 'percentage':
                    reward_percentage = str(reward.discount_percentage)
                    if reward.discount_apply_on == 'on_order':
                        reward_string = _("%s%% discount on total amount", reward_percentage)
                    elif reward.discount_apply_on == 'specific_products':
                        if len(reward.discount_specific_product_ids) > 1:
                            reward_string = _("%s%% discount on products", reward_percentage)
                        else:
                            reward_string = _(
                                "%(percentage)s%% discount on %(product_name)s",
                                percentage=reward_percentage,
                                product_name=reward.discount_specific_product_ids.name
                            )
                    elif reward.discount_apply_on == 'cheapest_product':
                        reward_string = _("%s%% discount on cheapest product", reward_percentage)
                elif reward.discount_type == 'fixed_amount':
                    program = self.env['coupon.program'].search([('reward_id', '=', reward.id)])
                    reward_string = _(
                        "%(amount)s %(currency)s discount on total amount",
                        amount=reward.discount_fixed_amount,
                        currency=program.currency_id.name
                    )
            result.append((reward.id, reward_string))
        return result

```

## File: models\coupon_rules.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class CouponRule(models.Model):
    _name = 'coupon.rule'
    _description = "Coupon Rule"

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

    _sql_constraints = [
        ('check_coupon_rule_dates', 'check(rule_date_from < rule_date_to)', 'The start date must be before the end date!'),
    ]

    @api.constrains('rule_minimum_amount')
    def _check_rule_minimum_amount(self):
        if self.filtered(lambda applicability: applicability.rule_minimum_amount < 0):
            raise ValidationError(_('Minimum purchased amount should be greater than 0'))

    @api.constrains('rule_min_quantity')
    def _check_rule_min_quantity(self):
        if not self.rule_min_quantity > 0:
            raise ValidationError(_('Minimum quantity should be greater than 0'))

```

## File: models\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def send_mail(self, **kwargs):
        for wizard in self:
            if self._context.get('mark_coupon_as_sent') and wizard.model == 'coupon.coupon' and wizard.partner_ids:
                # Mark coupon as sent in sudo, as helpdesk users don't have the right to write on coupons
                self.env[wizard.model].sudo().browse(wizard.res_id).state = 'sent'
        return super().send_mail(**kwargs)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_compose_message
from . import coupon
from . import coupon_reward
from . import coupon_rules
from . import coupon_program

```

## File: report\coupon_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class CouponReport(models.AbstractModel):
    _name = 'report.coupon.report_coupon'
    _description = 'Sales Coupon Report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['coupon.coupon'].browse(docids)
        return {
            'doc_ids': docs.ids,
            'doc_model': 'coupon.coupon',
            'data': data,
            'docs': docs,
        }

```

## File: report\coupon_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="report_coupon_code" model="ir.actions.report">
            <field name="name">Coupon Code</field>
            <field name="model">coupon.coupon</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">coupon.report_coupon_i18n</field>
            <field name="report_file">coupon.report_coupon_i18n</field>
            <field name="binding_model_id" ref="model_coupon_coupon"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: report\coupon_report_templates.xml

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
                                <t t-set="text">on your next order</t>
                                <h4>Here is your reward from <t t-esc="o.program_id.company_id.name"/>.</h4>
                                <t t-if="o.program_id.reward_type == 'discount'">
                                    <t t-set="trans_text">off %s</t>
                                    <t t-set="text" t-value="trans_text % text"/>
                                    <h1 t-if="o.program_id.discount_type == 'fixed_amount'" style="font-size: 55px; color: #875A7B">
                                        <strong><span t-field="o.program_id.discount_fixed_amount" t-options='{"widget": "monetary", "display_currency": o.program_id.currency_id}'/></strong>
                                    </h1>
                                    <h1 t-if="o.program_id.discount_type == 'percentage'" style="font-size: 55px; color: #875A7B">
                                        <strong><span t-field="o.program_id.discount_percentage"/> %</strong>
                                    </h1>
                                    <t t-if="o.program_id.discount_apply_on == 'specific_products'">
                                        <t t-if="len(o.program_id.discount_specific_product_ids) > 1">
                                            <t t-set="text">off on some products*</t>
                                            <t t-set="display_specific_products" t-value="True"/>
                                        </t>
                                        <t t-else="">
                                            <t t-set="trans_text">off on %s</t>                                            
                                            <t t-set="text" t-value="trans_text % o.program_id.discount_specific_product_ids.name"/>
                                        </t>
                                    </t>
                                    <t t-if="o.program_id.discount_apply_on == 'cheapest_product'">
                                        <t t-set="text">off on the cheapest product</t>
                                    </t>
                                </t>
                                <t t-if="o.program_id.reward_type == 'product'">
                                    <strong style="font-size: 55px; color: #875A7B">
                                        <t t-esc="'get %s free %s' % (o.program_id.reward_product_quantity, o.program_id.reward_product_id.name)"/>
                                    </strong>
                                </t>
                                <t t-if="o.program_id.reward_type == 'free_shipping'">
                                    <strong style="font-size: 55px; color: #875A7B">get free shipping</strong>
                                </t>
                                <h1 class="font-weight-bold" style="font-size: 34px" t-esc="text"/>
                                <br/>
                                <h4 t-if="o.expiration_date">
                                    Use this promo code before
                                    <span t-field="o.expiration_date" t-options='{"format": "yyyy-MM-d"}'/>
                                </h4>
                                <h2 class="mt32" style="margin-top: 32px">
                                    <strong class="bg-light" t-esc="o.code" style="padding: 20px 10px;"></strong>
                                </h2>
                                <h4 t-if="o.program_id.rule_min_quantity > 1">
                                    <span>Minimum purchase of</span>
                                    <strong t-esc="o.program_id.rule_min_quantity"/> <span>products</span>
                                </h4>
                                <h4 t-if="o.program_id.rule_minimum_amount">
                                    <span>Valid for purchase above</span>
                                    <strong t-esc="o.program_id.rule_minimum_amount" t-options="{'widget': 'monetary', 'display_currency': o.program_id.currency_id}"/>
                                </h4>
                                <p t-if="display_specific_products">
                                    <small>
                                        *Valid for following products: <t t-esc="', '.join(o.program_id.discount_specific_product_ids.mapped('name'))"/>
                                    </small>
                                </p>
                                <br/>
                                <img alt="Barcode" t-att-src="'/report/barcode/Code128/%s' % o.code"/>
                                <br/><br/>
                                <h4>Thank you,</h4>
                                <br/>
                                <div class="mt32">
                                    <div class="text-center">
                                        <img alt="Logo" t-att-src="'/logo?company=%d' % (o.program_id.company_id)" t-att-alt="'%s' % (o.program_id.company_id.name)" style="border:0px solid transparent; height: 50; width: 200px;" height="50"/>
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

    <template id="report_coupon_i18n">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="o" t-value="o.with_context(lang=o.partner_id.lang or o.env.lang)"/>
                <t t-call="coupon.report_coupon" t-lang="o.partner_id.lang or o.env.lang"/>
            </t>
        </t>
    </template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import coupon_report

```

## File: security\coupon_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="sale_coupon_generate_rule" model="ir.rule">
        <field name="name">Generate Sales Coupon Rule</field>
        <field name="model_id" ref="model_coupon_generate_wizard"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_coupon_coupon,access_coupon_coupon,model_coupon_coupon,base.group_user,0,0,0,0
access_coupon_reward,access_coupon_reward,model_coupon_reward,base.group_user,0,0,0,0
access_coupon_rule,access_coupon_rule,model_coupon_rule,base.group_user,0,0,0,0
access_coupon_program,access_coupon_program,model_coupon_program,base.group_user,0,0,0,0
access_coupon_generate_wizard,access_coupon_generate_wizard,model_coupon_generate_wizard,base.group_user,0,0,0,0


```

## File: views\coupon_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Common form view between the coupon programs and the promotion programs -->
    <record id="coupon_program_view_form_common" model="ir.ui.view">
        <field name="name">coupon.program.common.form</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <form string="Coupon Program">
                <sheet>
                    <div class="oe_button_box" name="button_box" />
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div name="title" class="oe_left">
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

    <record id="coupon_program_view_coupon_program_form" model="ir.ui.view">
        <field name="name">coupon.program.form</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon_program_view_form_common"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="before">
                <header>
                    <button type="action" name="%(coupon.coupon_generate_action)d"
                            string="Generate Coupon" attrs="{'invisible': [('active', '=', False)]}"/>
                    <button type="action" name="%(coupon.coupon_generate_action)d"
                            string="Generate Coupon" attrs="{'invisible': [('active', '=', True)]}" class="oe_highlight"/>
                </header>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="action" icon="fa-ticket" name="%(coupon.coupon_action)d">
                    <field name="coupon_count" string="Coupons" widget="statinfo"/>
                </button>
            </xpath>
            <xpath expr="//div[@name='title']" position="inside">
                <label class="oe_edit_only" for="name" string="Coupon Program Name"/>
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

    <record id="coupon_program_view_tree" model="ir.ui.view">
        <field name="name">coupon.program.tree</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <tree sample="1">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="active"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="coupon_program_view_search" model="ir.ui.view">
        <field name="name">coupon.program.search</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Expired" name="expired" domain="[('rule_date_to', '&lt;', datetime.datetime.now())]" help="Expired Programs"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="view_coupon_program_kanban" model="ir.ui.view">
        <field name="name">coupon.program.kanban</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
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
                                <div class="col-4 text-center coupon-count-label"><strong>Coupons</strong></div>
                                <div class="col-4 text-center active-label"><strong>Active</strong></div>
                                <div class="col-4 text-center coupon-count-value"><field name="coupon_count"/></div>
                                <div class="col-4 text-center active-value">
                                    <field name="active" widget="boolean"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="coupon_program_action_coupon_program" model="ir.actions.act_window">
        <field name="name">Coupon Programs</field>
        <field name="res_model">coupon.program</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="coupon_program_view_search"/>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'tree'}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('coupon_program_view_coupon_program_form')})]"/>
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

    <!-- Promotion Program -->

    <record id="coupon_program_view_promo_program_form" model="ir.ui.view">
        <field name="name">coupon.promotion.program.form</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon_program_view_form_common"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='title']" position="inside">
                <label class="oe_edit_only" for="name" string="Promotion Program Name"/>
                <h1><field name="name" class="oe_title" placeholder="Promotion Program Name..." height="20px"/></h1>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="action" icon="fa-ticket" name="%(coupon.coupon_action)d" attrs="{'invisible': [('promo_applicability', '=', 'on_current_order')]}">
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
            </xpath>
        </field>
    </record>

    <record id="coupon_program_view_promo_program_tree" model="ir.ui.view">
        <field name="name">coupon.promotion.program.tree</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <tree>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="promo_code_usage"/>
                <field name="active"/>
            </tree>
        </field>
    </record>

    <record id="coupon_program_view_promo_program_search" model="ir.ui.view">
        <field name="name">coupon.promotion.program.search</field>
        <field name="model">coupon.program</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="coupon_program_action_promo_program" model="ir.actions.act_window">
        <field name="name">Promotion Programs</field>
        <field name="res_model">coupon.program</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'tree'}),
            (0, 0, {'view_mode': 'form', 'view_id': ref('coupon_program_view_promo_program_form')})]"/>
        <field name="search_view_id" ref="coupon_program_view_promo_program_search"/>
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
</odoo>

```

## File: views\coupon_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="coupon_view_tree" model="ir.ui.view">
        <field name="name">coupon.coupon.tree</field>
        <field name="model">coupon.coupon</field>
        <field name="arch" type="xml">
            <tree string="Coupons" create="false" edit="false" delete="false">
                <field name="code"/>
                <field name="expiration_date"/>
                <field name="program_id"/>
                <field name="partner_id"/>
                <field name="state"/>
            </tree>
        </field>
    </record>

    <record id="coupon_action" model="ir.actions.act_window">
        <field name="name">Coupons</field>
        <field name="res_model">coupon.coupon</field>
        <field name="view_id" ref="coupon_view_tree"/>
        <field name="domain">[('program_id', '=', active_id)]</field>
        <field name="context">{}</field>
    </record>


    <record id="coupon_view_form" model="ir.ui.view">
        <field name="name">coupon.coupon.form</field>
        <field name="model">coupon.coupon</field>
        <field name="arch" type="xml">
            <form string="Coupons" create="false" edit="false" delete="false">
                <header>
                    <button name="action_coupon_sent" type="object" string="Send by Email" class="oe_highlight" attrs="{'invisible': [('state', 'not in', ['new', 'sent'])]}"/>
                    <button name="action_coupon_cancel" type="object" string="Cancel" class="oe_highlight" attrs="{'invisible': [('state', '!=', 'new')]}"/>
                    <field name="state" widget="statusbar" statusbar_visible="new,sent,used,expired" context="{'state': state}"/>
                </header>
                <sheet>
                    <group>
                        <field name="code"/>
                        <field name="expiration_date"/>
                        <field name="partner_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\coupon_generate.py

```python
# -*- coding: utf-8 -*-

from odoo import _, api, fields, models

import ast
from odoo.osv import expression


class CouponGenerate(models.TransientModel):
    _name = 'coupon.generate.wizard'
    _description = 'Generate Coupon'

    nbr_coupons = fields.Integer(string="Number of Coupons", help="Number of coupons", default=1)
    generation_type = fields.Selection([
        ('nbr_coupon', 'Number of Coupons'),
        ('nbr_customer', 'Number of Selected Customers')
        ], default='nbr_coupon')
    partners_domain = fields.Char(string="Customer", default='[]')
    has_partner_email = fields.Boolean(compute='_compute_has_partner_email')

    def generate_coupon(self):
        """Generates the number of coupons entered in wizard field nbr_coupons
        """
        program = self.env['coupon.program'].browse(self.env.context.get('active_id'))

        vals = {'program_id': program.id}

        if self.generation_type == 'nbr_coupon' and self.nbr_coupons > 0:
            for count in range(0, self.nbr_coupons):
                self.env['coupon.coupon'].create(vals)

        if self.generation_type == 'nbr_customer' and self.partners_domain:
            for partner in self.env['res.partner'].search(ast.literal_eval(self.partners_domain)):
                vals.update({'partner_id': partner.id, 'state': 'sent' if partner.email else 'new'})
                coupon = self.env['coupon.coupon'].create(vals)
                context = dict(lang=partner.lang)
                subject = _('%s, a coupon has been generated for you') % (partner.name)
                del context
                template = self.env.ref('coupon.mail_template_sale_coupon', raise_if_not_found=False)
                if template:
                    email_values = {'email_from': self.env.user.email or '', 'subject': subject}
                    template.send_mail(coupon.id, email_values=email_values, notif_layout='mail.mail_notification_light')

    @api.depends('partners_domain')
    def _compute_has_partner_email(self):
        for record in self:
            partners_domain = ast.literal_eval(record.partners_domain)
            if partners_domain == [['', '=', 1]]:
                # The field name is not clear. It actually means "all partners have email".
                # If domain is not set, we don't want to show the warning "there is a partner without email".
                # So, we explicitly set value to True
                record.has_partner_email = True
                continue
            domain = expression.AND([partners_domain, [('email', '=', False)]])
            record.has_partner_email = self.env['res.partner'].search_count(domain) == 0

```

## File: wizard\coupon_generate_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="coupon_generate_view_form" model="ir.ui.view">
        <field name="name">coupon.generate.wizard.form</field>
        <field name="model">coupon.generate.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate Coupons">
                <group>
                    <field name="has_partner_email" invisible="1"/>
                    <field name="generation_type" widget="radio"/>
                    <field name="partners_domain" attrs="{'invisible': [('generation_type', '!=', 'nbr_customer')]}" widget="domain" options="{'model': 'res.partner'}"/>
                    <field name="nbr_coupons" attrs="{'invisible': [('generation_type', '!=', 'nbr_coupon')]}"/>
                </group>
                <div role="alert" class="alert alert-warning" attrs="{'invisible': ['|', ('generation_type', '!=', 'nbr_customer'), ('has_partner_email', '=', True)]}">
                    Some selected customers do not have an email address and will not receive the coupon.
                </div>
                <footer>
                    <button name="generate_coupon" type="object" string="Generate" class="oe_highlight"/>
                    <button special="cancel" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="coupon_generate_action" model="ir.actions.act_window">
        <field name="name">Number of Coupons To Generate</field>
        <field name="res_model">coupon.generate.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="view_id" ref="coupon_generate_view_form"/>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import coupon_generate

```

