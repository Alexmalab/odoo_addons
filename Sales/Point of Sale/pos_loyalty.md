# Odoo Module: pos_loyalty

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def uninstall_hook(env):
    """Delete loyalty history record accessing pos order on uninstall."""
    env['loyalty.history'].search([('order_model', '=', 'pos.order')]).unlink()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Point of Sale - Coupons & Loyalty",
    'version': '2.0',
    'category': 'Sales/Point Of Sale',
    'sequence': 6,
    'summary': 'Use Coupons, Gift Cards and Loyalty programs in Point of Sale',
    'depends': ['loyalty', 'point_of_sale'],
    'data': [
        'security/ir.model.access.csv',
        'data/default_barcode_patterns.xml',
        'data/gift_card_data.xml',
        'views/loyalty_card_views.xml',
        'views/loyalty_mail_views.xml',
        'views/pos_loyalty_menu_views.xml',
        'views/res_config_settings_view.xml',
        'views/loyalty_program_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/pos_loyalty_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'pos_loyalty/static/src/portal/*',
        ],
        'point_of_sale._assets_pos': [
            'pos_loyalty/static/src/**/*',
            ('remove', 'pos_loyalty/static/src/portal/*'),
        ],
        'web.assets_tests': [
            'pos_loyalty/static/tests/tours/**/*',
        ],
    },
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: data\default_barcode_patterns.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="barcode_rule_coupon" model="barcode.rule">
        <field name="name">Coupon &amp; Gift Card Barcodes</field>
        <field name="barcode_nomenclature_id" ref="barcodes.default_barcode_nomenclature"/>
        <field name="sequence">50</field>
        <field name="type">coupon</field>
        <field name="encoding">any</field>
        <!-- Old Gift Cards might start with 044 -->
        <field name="pattern">043|044</field>
    </record>
</odoo>

```

## File: data\gift_card_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty.gift_card_product_50" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="taxes_id" eval="False"/>
    </record>
    <record id="loyalty.ewallet_product_50" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="taxes_id" eval="False"/>
    </record>
    <!-- Gift Cards -->
    <record id="loyalty.gift_card_program" model="loyalty.program">
        <field name="pos_report_print_id" ref="loyalty.report_gift_card"/>
    </record>
</odoo>

```

## File: data\pos_loyalty_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- 15% next order -->
        <record id="15_pc_on_next_order" model="loyalty.program">
            <field name="name">15% on next order</field>
            <field name="program_type">next_order_coupons</field>
            <field name="trigger">auto</field>
            <field name="applies_on">future</field>
            <field name="pos_config_ids" eval="[(6,0,[ref('point_of_sale.pos_config_main')])]"/>
            <field name="portal_visible">True</field>
            <field name="portal_point_name">Coupon point(s)</field>
        </record>

        <record id="15_pc_on_next_order_rule" model="loyalty.rule">
            <field name="minimum_amount">100</field>
            <field name="program_id" ref="pos_loyalty.15_pc_on_next_order"/>
        </record>
        
        <record id="15_pc_on_next_order_reward" model="loyalty.reward">
            <field name="discount_mode">percent</field>
            <field name="discount">15</field>
            <field name="discount_applicability">order</field>
            <field name="program_id" ref="pos_loyalty.15_pc_on_next_order"/>
        </record>

        <record id="loyalty.3_cabinets_plus_1_free" model="loyalty.program">
            <field name="pos_config_ids" eval="[(6,0,[ref('point_of_sale.pos_config_main')])]" />
        </record>

        <record id="loyalty.10_percent_with_code" model="loyalty.program">
            <field name="pos_config_ids" eval="[(6,0,[ref('point_of_sale.pos_config_main')])]" />
        </record>

        <record id="loyalty.10_percent_coupon" model="loyalty.program">
            <field name="pos_config_ids" eval="[(6,0,[ref('point_of_sale.pos_config_main')])]" />
        </record>

        <function name="create" model="loyalty.generate.wizard">
            <value model="loyalty.generate.wizard" eval="dict(
                obj().default_get(list(obj().fields_get())),
                **{
                    'coupon_qty': 10,
                    'program_id': ref('loyalty.10_percent_coupon'),
                }
            )"/>
        </function>

        <!-- Create 10 coupons for the 10% coupon program based on the created record above -->
        <function name="generate_coupons" model="loyalty.generate.wizard">
            <value model="loyalty.generate.wizard" eval="obj().search([('coupon_qty', '=', 10)]).id"/>
        </function>

        <function name="unlink" model="loyalty.generate.wizard">
            <value model="loyalty.generate.wizard" eval="obj().search([('coupon_qty', '=', 10)]).id"/>
        </function>
    </data>

    <record id="simple_pen" model="product.product">
        <field name="available_in_pos">True</field>
        <field name="list_price">1.20</field>
        <field name="name">Simple Pen</field>
        <field name="weight">0.01</field>
        <field name="default_code">CONS_0002</field>
        <field name="uom_id" ref="uom.product_uom_unit" />
        <field name="uom_po_id" ref="uom.product_uom_unit" />
        <field name="image_1920" type="base64" file="pos_loyalty/static/img/simple_pen.png"/>
    </record>

    <!-- Loyalty program -->
    <record id="loyalty_program" model="loyalty.program">
        <field name="name">Loyalty Program</field>
        <field name="program_type">loyalty</field>
        <field name="applies_on">both</field>
        <field name="trigger">auto</field>
        <field name="portal_visible">True</field>
        <field name="portal_point_name">Loyalty Points</field>
    </record>

    <record id="loyalty_program_rule" model="loyalty.rule">
        <field name="reward_point_mode">money</field>
        <field name="reward_point_amount">10</field>
        <field name="program_id" ref="pos_loyalty.loyalty_program"/>
    </record>

    <record id="loyalty_program_reward" model="loyalty.reward">
        <field name="reward_type">product</field>
        <field name="required_points">5</field>
        <field name="reward_product_id" ref="pos_loyalty.simple_pen"/>
        <field name="program_id" ref="pos_loyalty.loyalty_program"/>
    </record>

    <record id="product.product_product_6" model="product.product">
        <field name="available_in_pos">True</field>
    </record>

</odoo>

```

## File: models\barcode_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class BarcodeRule(models.Model):
    _inherit = 'barcode.rule'

    type = fields.Selection(selection_add=[('coupon', 'Coupon')], ondelete={'coupon': 'set default'})

```

## File: models\loyalty_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api

class LoyaltyCard(models.Model):
    _name = 'loyalty.card'
    _inherit = ['loyalty.card', 'pos.load.mixin']

    source_pos_order_id = fields.Many2one('pos.order', "PoS Order Reference",
        help="PoS order where this coupon was generated.")

    @api.model
    def _load_pos_data_domain(self, data):
        return [('program_id', 'in', [program["id"] for program in data["loyalty.program"]['data']])]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['partner_id', 'code', 'points', 'program_id', 'expiration_date']

    def _has_source_order(self):
        return super()._has_source_order() or bool(self.source_pos_order_id)

    def _get_default_template(self):
        self.ensure_one()
        if self.source_pos_order_id:
            return self.env.ref('pos_loyalty.mail_coupon_template', False)
        return super()._get_default_template()

    def _get_mail_partner(self):
        return super()._get_mail_partner() or self.sudo().source_pos_order_id.partner_id

    def _get_signature(self):
        return self.source_pos_order_id.user_id.signature or super()._get_signature()

    def _compute_use_count(self):
        super()._compute_use_count()
        read_group_res = self.env['pos.order.line']._read_group(
            [('coupon_id', 'in', self.ids)], ['coupon_id'], ['__count'])
        count_per_coupon = {coupon.id: count for coupon, count in read_group_res}
        for card in self:
            card.use_count += count_per_coupon.get(card.id, 0)

```

## File: models\loyalty_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class LoyaltyMail(models.Model):
    _inherit = 'loyalty.mail'

    pos_report_print_id = fields.Many2one('ir.actions.report', string="Print Report", domain=[('model', '=', 'loyalty.card')],
        help="The report action to be executed when creating a coupon/gift card/loyalty card in the PoS.",
    )

```

## File: models\loyalty_program.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError

class LoyaltyProgram(models.Model):
    _name = 'loyalty.program'
    _inherit = ['loyalty.program', 'pos.load.mixin']

    # NOTE: `pos_config_ids` satisfies an excpeptional use case: when no PoS is specified, the loyalty program is
    # applied to every PoS. You can access the loyalty programs of a PoS using _get_program_ids() of pos.config
    pos_config_ids = fields.Many2many('pos.config', compute="_compute_pos_config_ids", store=True, readonly=False, string="Point of Sales", help="Restrict publishing to those shops.")
    pos_order_count = fields.Integer("PoS Order Count", compute='_compute_pos_order_count')
    pos_ok = fields.Boolean("Point of Sale", default=True)
    pos_report_print_id = fields.Many2one('ir.actions.report', string="Print Report", domain=[('model', '=', 'loyalty.card')], compute='_compute_pos_report_print_id', inverse='_inverse_pos_report_print_id', readonly=False,
        help="This is used to print the generated gift cards from PoS.")

    @api.model
    def _load_pos_data_domain(self, data):
        config_id = self.env['pos.config'].browse(data['pos.config']['data'][0]['id'])
        return [('id', 'in', config_id._get_program_ids().ids)]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return [
            'name', 'trigger', 'applies_on', 'program_type', 'pricelist_ids', 'date_from',
            'date_to', 'limit_usage', 'max_usage', 'is_nominative', 'portal_visible',
            'portal_point_name', 'trigger_product_ids', 'rule_ids', 'reward_ids'
        ]

    @api.depends("communication_plan_ids.pos_report_print_id")
    def _compute_pos_report_print_id(self):
        for program in self:
            program.pos_report_print_id = program.communication_plan_ids.pos_report_print_id[:1]

    def _inverse_pos_report_print_id(self):
        for program in self:
            if program.program_type not in ("gift_card", "ewallet"):
                continue

            if program.pos_report_print_id:
                if not program.mail_template_id:
                    mail_template_label = program._fields.get('mail_template_id').get_description(self.env)['string']
                    pos_report_print_label = program._fields.get('pos_report_print_id').get_description(self.env)['string']
                    raise UserError(_(
                        "You must set '%(mail_template)s' before setting '%(report)s'.",
                        mail_template=mail_template_label,
                        report=pos_report_print_label,
                    ))
                else:
                    if not program.communication_plan_ids:
                        program.communication_plan_ids = self.env['loyalty.mail'].create({
                            'program_id': program.id,
                            'trigger': 'create',
                            'mail_template_id': program.mail_template_id.id,
                            'pos_report_print_id': program.pos_report_print_id.id,
                        })
                    else:
                        program.communication_plan_ids.write({
                            'trigger': 'create',
                            'pos_report_print_id': program.pos_report_print_id.id,
                        })

    @api.depends('pos_ok')
    def _compute_pos_config_ids(self):
        for program in self:
            if not program.pos_ok:
                program.pos_config_ids = False

    def _compute_pos_order_count(self):
        query = """
            SELECT program.id, SUM(orders_count)
            FROM loyalty_program program
                JOIN loyalty_reward reward ON reward.program_id = program.id
                JOIN LATERAL (
                    SELECT COUNT(DISTINCT orders.id) AS orders_count
                    FROM pos_order orders
                        JOIN pos_order_line order_lines ON order_lines.order_id = orders.id
                        WHERE order_lines.reward_id = reward.id
                ) agg ON TRUE
                WHERE program.id = ANY(%s)
                    GROUP BY program.id
                """
        self._cr.execute(query, (self.ids,))
        res = self._cr.dictfetchall()
        res = {k['id']: k['sum'] for k in res}

        for rec in self:
            rec.pos_order_count = res.get(rec.id) or 0

    def _compute_total_order_count(self):
        super()._compute_total_order_count()
        for program in self:
            program.total_order_count += program.pos_order_count

```

## File: models\loyalty_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api
import ast
import json

class LoyaltyReward(models.Model):
    _name = 'loyalty.reward'
    _inherit = ['loyalty.reward', 'pos.load.mixin']

    def _get_discount_product_values(self):
        res = super()._get_discount_product_values()
        for vals in res:
            vals.update({'taxes_id': False})
        return res

    @api.model
    def _load_pos_data_domain(self, data):
        config_id = self.env['pos.config'].browse(data['pos.config']['data'][0]['id'])
        return [('program_id', 'in', config_id._get_program_ids().ids)]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['description', 'program_id', 'reward_type', 'required_points', 'clear_wallet', 'currency_id',
                'discount', 'discount_mode', 'discount_applicability', 'all_discount_product_ids', 'is_global_discount',
                'discount_max_amount', 'discount_line_product_id', 'reward_product_id',
                'multi_product', 'reward_product_ids', 'reward_product_qty', 'reward_product_uom_id', 'reward_product_domain']

    def _load_pos_data(self, data):
        domain = self._load_pos_data_domain(data)
        fields = self._load_pos_data_fields(data['pos.config']['data'][0]['id'])
        rewards = self.search_read(domain, fields, load=False)
        for reward in rewards:
            reward['reward_product_domain'] = self._replace_ilike_with_in(reward['reward_product_domain'])
        return {
            'data': rewards,
            'fields': fields,
        }

    def _get_reward_product_domain_fields(self, config_id):
        fields = set()
        config = self.env['pos.config'].browse(config_id)
        search_domain = [('program_id', 'in', config._get_program_ids().ids)]
        domains = self.search_read(search_domain, fields=['reward_product_domain'], load=False)
        for domain in filter(lambda d: d['reward_product_domain'] != "null", domains):
            domain = json.loads(domain['reward_product_domain'])
            for condition in self._parse_domain(domain).values():
                field_name, _, _ = condition
                fields.add(field_name)
        return fields

    def _replace_ilike_with_in(self, domain_str):
        if domain_str == "null":
            return domain_str

        domain = json.loads(domain_str)

        for index, condition in self._parse_domain(domain).items():
            field_name, operator, value = condition
            field = self.env['product.product']._fields.get(field_name)

            if field and field.type == 'many2one' and operator in ('ilike', 'not ilike'):
                comodel = self.env[field.comodel_name]
                matching_ids = list(comodel._search([('display_name', operator, value)]))

                new_operator = 'in' if operator == 'ilike' else 'not in'
                domain[index] = [field_name, new_operator, matching_ids]

        return json.dumps(domain)

    def _parse_domain(self, domain):
        parsed_domain = {}

        for index, condition in enumerate(domain):
            if isinstance(condition, (list, tuple)) and len(condition) == 3:
                parsed_domain[index] = condition
        return parsed_domain

    def unlink(self):
        if len(self) == 1 and self.env['pos.order.line'].sudo().search_count([('reward_id', 'in', self.ids)], limit=1):
            return self.action_archive()
        return super().unlink()

```

## File: models\loyalty_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression

class LoyaltyRule(models.Model):
    _name = 'loyalty.rule'
    _inherit = ['loyalty.rule', 'pos.load.mixin']

    valid_product_ids = fields.Many2many(
        'product.product', "Valid Products", compute='_compute_valid_product_ids',
        help="These are the products that are valid for this rule.")
    any_product = fields.Boolean(
        compute='_compute_valid_product_ids', help="Technical field, whether all product match")

    promo_barcode = fields.Char("Barcode", compute='_compute_promo_barcode', store=True, readonly=False,
        help="A technical field used as an alternative to the promo code. "
        "This is automatically generated when the promo code is changed."
    )

    @api.model
    def _load_pos_data_domain(self, data):
        config_id = self.env['pos.config'].browse(data['pos.config']['data'][0]['id'])
        return [('program_id', 'in', config_id._get_program_ids().ids)]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['program_id', 'valid_product_ids', 'any_product', 'currency_id',
            'reward_point_amount', 'reward_point_split', 'reward_point_mode',
            'minimum_qty', 'minimum_amount', 'minimum_amount_tax_mode', 'mode', 'code']

    @api.depends('product_ids', 'product_category_id', 'product_tag_id', 'product_domain')  # TODO later: product tags
    def _compute_valid_product_ids(self):
        for key, rules in self.grouped(lambda rule: (
            tuple(rule.product_ids.ids),
            rule.product_category_id.id,
            rule.product_tag_id.id,
            '' if rule.product_domain in ('[]', "[['sale_ok', '=', True]]") else rule.product_domain,
        )).items():
            if any(key):
                domain = expression.AND([[('available_in_pos', '=', True)], rules[:1]._get_valid_product_domain()])
                rules.valid_product_ids = self.env['product.product'].search(domain, order="id")
                rules.any_product = False
            else:
                rules.valid_product_ids = self.env['product.product']
                rules.any_product = True

    @api.depends('code')
    def _compute_promo_barcode(self):
        for rule in self:
            rule.promo_barcode = self.env['loyalty.card']._generate_code()

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError

class PosConfig(models.Model):
    _inherit = 'pos.config'

    # NOTE: this funtions acts as a m2m field with loyalty.program model. We do this to handle an excpetional use case:
    # When no PoS is specified at a loyalty program form, this program is applied to every PoS (instead of none)
    def _get_program_ids(self):
        return self.env['loyalty.program'].search(['&', ('pos_ok', '=', True), '|', ('pos_config_ids', '=', self.id), ('pos_config_ids', '=', False)])

    def _check_before_creating_new_session(self):
        self.ensure_one()
        # Check validity of programs before opening a new session
        invalid_reward_products_msg = ''
        for reward in self._get_program_ids().reward_ids:
            if reward.reward_type == 'product':
                for product in reward.reward_product_ids:
                    if product.available_in_pos:
                        continue
                    invalid_reward_products_msg += "\n\t"
                    invalid_reward_products_msg += _(
                        "Program: %(name)s, Reward Product: `%(reward_product)s`",
                        name=reward.program_id.name,
                        reward_product=product.name,
                    )
        gift_card_programs = self._get_program_ids().filtered(lambda p: p.program_type == 'gift_card')
        for product in gift_card_programs.mapped('rule_ids.valid_product_ids'):
            if product.available_in_pos:
                continue
            invalid_reward_products_msg += "\n\t"
            invalid_reward_products_msg += _(
                "Program: %(name)s, Rule Product: `%(rule_product)s`",
                name=reward.program_id.name,
                rule_product=product.name,
            )

        if invalid_reward_products_msg:
            prefix_error_msg = _("To continue, make the following reward products available in Point of Sale.")
            raise UserError(f"{prefix_error_msg}\n{invalid_reward_products_msg}")
        if gift_card_programs:
            for gc_program in gift_card_programs:
                # Do not allow a gift card program with more than one rule or reward, and check that they make sense
                if len(gc_program.reward_ids) > 1:
                    raise UserError(_('Invalid gift card program. More than one reward.'))
                elif len(gc_program.rule_ids) > 1:
                    raise UserError(_('Invalid gift card program. More than one rule.'))
                rule = gc_program.rule_ids
                if rule.reward_point_amount != 1 or rule.reward_point_mode != 'money':
                    raise UserError(_('Invalid gift card program rule. Use 1 point per currency spent.'))
                reward = gc_program.reward_ids
                if reward.reward_type != 'discount' or reward.discount_mode != 'per_point' or reward.discount != 1:
                    raise UserError(_('Invalid gift card program reward. Use 1 currency per point discount.'))
                if not gc_program.mail_template_id:
                    raise UserError(_('There is no email template on the gift card program and your pos is set to print them.'))
                if not gc_program.pos_report_print_id:
                    raise UserError(_('There is no print report on the gift card program and your pos is set to print them.'))

        return super()._check_before_creating_new_session()

    def use_coupon_code(self, code, creation_date, partner_id, pricelist_id):
        self.ensure_one()
        # Points desc so that in coupon mode one could use a coupon multiple times
        coupon = self.env['loyalty.card'].search(
            [('program_id', 'in', self._get_program_ids().ids),
             '|', ('partner_id', 'in', (False, partner_id)), ('program_type', '=', 'gift_card'),
             ('code', '=', code)],
            order='partner_id, points desc', limit=1)
        program = coupon.program_id
        if not coupon or not program.active:
            return {
                'successful': False,
                'payload': {
                    'error_message': _('This coupon is invalid (%s).', code),
                },
            }
        check_date = fields.Date.from_string(creation_date[:11])
        today_date = fields.Date.context_today(self)
        error_message = False
        if (
            (coupon.expiration_date and coupon.expiration_date < check_date)
            or (program.date_to and program.date_to < today_date)
            or (program.limit_usage and program.total_order_count >= program.max_usage)
        ):
            error_message = _("This coupon is expired (%s).", code)
        elif program.date_from and program.date_from > today_date:
            error_message = _("This coupon is not yet valid (%s).", code)
        elif (
            not program.reward_ids or
            not any(r.required_points <= coupon.points for r in program.reward_ids)
        ):
            error_message = _("No reward can be claimed with this coupon.")
        elif program.pricelist_ids and pricelist_id not in program.pricelist_ids.ids:
            error_message = _("This coupon is not available with the current pricelist.")
        elif coupon and program.program_type == 'promo_code':
            error_message = _("This programs requires a code to be applied.")

        if error_message:
            return {
                'successful': False,
                'payload': {
                    'error_message': error_message,
                },
            }

        return {
            'successful': True,
            'payload': {
                'program_id': program.id,
                'coupon_id': coupon.id,
                'coupon_partner_id': coupon.partner_id.id,
                'points': coupon.points,
                'has_source_order': coupon._has_source_order(),
            },
        }

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from odoo import _, models
from odoo.tools import float_compare
import base64

class PosOrder(models.Model):
    _inherit = 'pos.order'

    def validate_coupon_programs(self, point_changes, new_codes):
        """
        This is called upon validating the order in the pos.

        This will check the balance for any pre-existing coupon to make sure that the rewards are in fact all claimable.
        This will also check that any set code for coupons do not exist in the database.
        """
        point_changes = {int(k): v for k, v in point_changes.items()}
        coupon_ids_from_pos = set(point_changes.keys())
        coupons = self.env['loyalty.card'].browse(coupon_ids_from_pos).exists().filtered('program_id.active')
        coupon_difference = set(coupons.ids) ^ coupon_ids_from_pos
        if coupon_difference:
            return {
                'successful': False,
                'payload': {
                    'message': _('Some coupons are invalid. The applied coupons have been updated. Please check the order.'),
                    'removed_coupons': list(coupon_difference),
                }
            }
        for coupon in coupons:
            if float_compare(coupon.points, -point_changes[coupon.id], 2) == -1:
                return {
                    'successful': False,
                    'payload': {
                        'message': _('There are not enough points for the coupon: %s.', coupon.code),
                        'updated_points': {c.id: c.points for c in coupons}
                    }
                }
        # Check existing coupons
        coupons = self.env['loyalty.card'].search([('code', 'in', new_codes)])
        if coupons:
            return {
                'successful': False,
                'payload': {
                    'message': _('The following codes already exist in the database, perhaps they were already sold?\n%s',
                        ', '.join(coupons.mapped('code'))),
                }
            }
        return {
            'successful': True,
            'payload': {},
        }

    def add_loyalty_history_lines(self, coupon_data, coupon_updates):
        id_mapping = {item['old_id']: int(item['id']) for item in coupon_updates}
        history_lines_create_vals = []
        for coupon in coupon_data:
            card_id = id_mapping.get(int(coupon['card_id'], False)) or int(coupon['card_id'])
            if not self.env['loyalty.card'].browse(card_id).exists():
                continue
            issued = coupon['won']
            cost = coupon['spent']
            if (issued or cost) and card_id > 0:
                history_lines_create_vals.append({
                    'card_id': card_id,
                    'order_model': self._name,
                    'order_id': self.id,
                    'description': _('Onsite %s', self.display_name),
                    'used': cost,
                    'issued': issued,
                })
        self.env['loyalty.history'].create(history_lines_create_vals)

    def confirm_coupon_programs(self, coupon_data):
        """
        This is called after the order is created.

        This will create all necessary coupons and link them to their line orders etc..

        It will also return the points of all concerned coupons to be updated in the cache.
        """
        get_partner_id = lambda partner_id: partner_id and self.env['res.partner'].browse(partner_id).exists() and partner_id or False
        # Keys are stringified when using rpc
        coupon_data = {int(k): v for k, v in coupon_data.items()}

        self._check_existing_loyalty_cards(coupon_data)
        # Map negative id to newly created ids.
        coupon_new_id_map = {k: k for k in coupon_data.keys() if k > 0}

        # Create the coupons that were awarded by the order.
        coupons_to_create = {k: v for k, v in coupon_data.items() if k < 0 and not v.get('giftCardId')}
        coupon_create_vals = [{
            'program_id': p['program_id'],
            'partner_id': get_partner_id(p.get('partner_id', False)),
            'code': p.get('code') or p.get('barcode') or self.env['loyalty.card']._generate_code(),
            'points': 0,
            'expiration_date': p.get('date_to', False),
            'source_pos_order_id': self.id,
            'expiration_date': p.get('expiration_date')
        } for p in coupons_to_create.values()]

        # Pos users don't have the create permission
        new_coupons = self.env['loyalty.card'].with_context(action_no_send_mail=True).sudo().create(coupon_create_vals)

        # We update the gift card that we sold when the gift_card_settings = 'scan_use'.
        gift_cards_to_update = [v for v in coupon_data.values() if v.get('giftCardId')]
        updated_gift_cards = self.env['loyalty.card']
        for coupon_vals in gift_cards_to_update:
            gift_card = self.env['loyalty.card'].browse(coupon_vals.get('giftCardId'))
            gift_card.write({
                'points': coupon_vals['points'],
                'source_pos_order_id': self.id,
                'partner_id': get_partner_id(coupon_vals.get('partner_id', False)),
            })
            updated_gift_cards |= gift_card

        # Map the newly created coupons
        for old_id, new_id in zip(coupons_to_create.keys(), new_coupons):
            coupon_new_id_map[new_id.id] = old_id

        # We need a sudo here because this can trigger `_compute_order_count` that require access to `sale.order.line`
        all_coupons = self.env['loyalty.card'].sudo().browse(coupon_new_id_map.keys()).exists()
        lines_per_reward_code = defaultdict(lambda: self.env['pos.order.line'])
        for line in self.lines:
            if not line.reward_identifier_code:
                continue
            lines_per_reward_code[line.reward_identifier_code] |= line
        for coupon in all_coupons:
            if coupon.id in coupon_new_id_map:
                # Coupon existed previously, update amount of points.
                coupon.points += coupon_data[coupon_new_id_map[coupon.id]]['points']
            for reward_code in coupon_data[coupon_new_id_map[coupon.id]].get('line_codes', []):
                lines_per_reward_code[reward_code].coupon_id = coupon
        # Send creation email
        new_coupons.with_context(action_no_send_mail=False)._send_creation_communication()
        # Reports per program
        report_per_program = {}
        coupon_per_report = defaultdict(list)
        # Important to include the updated gift cards so that it can be printed. Check coupon_report.
        for coupon in new_coupons | updated_gift_cards:
            if coupon.program_id not in report_per_program:
                report_per_program[coupon.program_id] = coupon.program_id.communication_plan_ids.\
                    filtered(lambda c: c.trigger == 'create').pos_report_print_id
            for report in report_per_program[coupon.program_id]:
                coupon_per_report[report.id].append(coupon.id)
        return {
            'coupon_updates': [{
                'old_id': coupon_new_id_map[coupon.id],
                'id': coupon.id,
                'points': coupon.points,
                'code': coupon.code,
                'program_id': coupon.program_id.id,
                'partner_id': coupon.partner_id.id,
            } for coupon in all_coupons if coupon.program_id.is_nominative],
            'program_updates': [{
                'program_id': program.id,
                'usages': program.sudo().total_order_count,
            } for program in all_coupons.program_id],
            'new_coupon_info': [{
                'program_name': coupon.program_id.name,
                'expiration_date': coupon.expiration_date,
                'code': coupon.code,
            } for coupon in new_coupons if (
                coupon.program_id.applies_on == 'future'
                # Don't send the coupon code for the gift card and ewallet programs.
                # It should not be printed in the ticket.
                and coupon.program_id.sudo().program_type not in ['gift_card', 'ewallet']
            )],
            'coupon_report': coupon_per_report,
        }

    def _check_existing_loyalty_cards(self, coupon_data):
        coupon_key_to_modify = []
        for coupon_id, coupon_vals in coupon_data.items():
            partner_id = coupon_vals.get('partner_id', False)
            if partner_id:
                partner_coupons = self.env['loyalty.card'].search(
                    [('partner_id', '=', partner_id), ('program_type', '=', 'loyalty')])
                existing_coupon_for_program = partner_coupons.filtered(lambda c: c.program_id.id == coupon_vals['program_id'])
                if existing_coupon_for_program:
                    coupon_vals['coupon_id'] = existing_coupon_for_program[0].id
                    coupon_key_to_modify.append([coupon_id, existing_coupon_for_program[0].id])
        for old_key, new_key in coupon_key_to_modify:
            coupon_data[new_key] = coupon_data.pop(old_key)

    def _get_fields_for_order_line(self):
        fields = super(PosOrder, self)._get_fields_for_order_line()
        fields.extend(['is_reward_line', 'reward_id', 'coupon_id', 'reward_identifier_code', 'points_cost'])
        return fields

    def _add_mail_attachment(self, name, ticket, basic_receipt):
        attachment = super()._add_mail_attachment(name, ticket, basic_receipt)
        gift_card_programs = self.config_id._get_program_ids().filtered(lambda p: p.program_type == 'gift_card' and
                                                                                  p.pos_report_print_id)
        if gift_card_programs:
            gift_cards = self.env['loyalty.card'].search([('source_pos_order_id', '=', self.id),
                                                          ('program_id', 'in', gift_card_programs.mapped('id'))])
            if gift_cards:
                for program in gift_card_programs:
                    filtered_gift_cards = gift_cards.filtered(lambda gc: gc.program_id == program)
                    if filtered_gift_cards:
                        action_report = program.pos_report_print_id
                        report = action_report._render_qweb_pdf(action_report.report_name, filtered_gift_cards.mapped('id'))
                        filename = name + '.pdf'
                        gift_card_pdf = self.env['ir.attachment'].create({
                            'name': filename,
                            'type': 'binary',
                            'datas': base64.b64encode(report[0]),
                            'store_fname': filename,
                            'res_model': 'pos.order',
                            'res_id': self.ids[0],
                            'mimetype': 'application/x-pdf'
                        })
                        attachment += [(4, gift_card_pdf.id)]

        return attachment

```

## File: models\pos_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api

class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    is_reward_line = fields.Boolean(
        help="Whether this line is part of a reward or not.")
    reward_id = fields.Many2one(
        'loyalty.reward', "Reward", ondelete='restrict',
        help="The reward associated with this line.", index='btree_not_null')
    coupon_id = fields.Many2one(
        'loyalty.card', "Coupon", ondelete='restrict',
        help="The coupon used to claim that reward.")
    reward_identifier_code = fields.Char(help="""
        Technical field used to link multiple reward lines from the same reward together.
    """)
    points_cost = fields.Float(help="How many point this reward cost on the coupon.")

    def _is_not_sellable_line(self):
        return super().is_not_sellable_line() or self.reward_id

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['is_reward_line', 'reward_id', 'reward_identifier_code', 'points_cost', 'coupon_id']
        return params

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api

class PosSession(models.Model):
    _inherit = 'pos.session'

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        data += ['loyalty.program', 'loyalty.rule', 'loyalty.reward', 'loyalty.card']
        return data

```

## File: models\product_product.py

```python
import logging

from odoo import api, models
from odoo.exceptions import AccessError

_logger = logging.getLogger(__name__)


class ProductProduct(models.Model):
    _inherit = 'product.product'

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['all_product_tag_ids']

        # add missing product fields used in the reward_product_domain
        missing_fields = self.env['loyalty.reward']._get_reward_product_domain_fields(config_id) - set(params)

        if missing_fields:
            params.extend([field for field in missing_fields if field in self._fields])

        return params

    def _load_pos_data(self, data):
        res = super()._load_pos_data(data)
        config_id = self.env['pos.config'].browse(data['pos.config']['data'][0]['id'])
        try:
            rewards = config_id._get_program_ids().reward_ids
            reward_products = rewards.discount_line_product_id | rewards.reward_product_ids | rewards.reward_product_id
            trigger_products = config_id._get_program_ids().filtered(lambda p: p.program_type in ['ewallet', 'gift_card']).trigger_product_ids

            loyalty_product_ids = set(reward_products.ids + trigger_products.ids)
            classic_product_ids = {product['id'] for product in res['data']}
            products = self.env['product.product'].browse(list(loyalty_product_ids - classic_product_ids))
            products = products.read(fields=res['fields'], load=False)
            self._process_pos_ui_product_product(products, config_id)

            data['pos.session']['data'][0]['_pos_special_products_ids'] += [product.id for product in reward_products if product.id not in [p["id"] for p in res['data']]]
            res['data'].extend(products)
        except AccessError as e:
            _logger.warning('Cannot load loyalty products into the PoS \n%s', e)

        return res

```

## File: models\res_partner.py

```python
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    loyalty_card_count = fields.Integer(groups='base.group_user,point_of_sale.group_pos_user')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import barcode_rule
from . import loyalty_card
from . import loyalty_mail
from . import loyalty_program
from . import loyalty_reward
from . import loyalty_rule
from . import pos_config
from . import pos_order_line
from . import pos_order
from . import pos_session
from . import product_product
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_program_pos_user,Loyalty Program (PoS User),loyalty.model_loyalty_program,point_of_sale.group_pos_user,1,0,0,0
access_program_pos_manager,Loyalty Program (PoS Manager),loyalty.model_loyalty_program,point_of_sale.group_pos_manager,1,1,1,1
access_applicability_pos_user,Loyalty Rule (PoS User),loyalty.model_loyalty_rule,point_of_sale.group_pos_user,1,0,0,0
access_applicability_pos_manager,Loyalty Rule (PoS Manager),loyalty.model_loyalty_rule,point_of_sale.group_pos_manager,1,1,1,1
access_coupon_pos_user,Loyalty (PoS User),loyalty.model_loyalty_card,point_of_sale.group_pos_user,1,1,0,0
access_coupon_pos_manager,Loyalty (PoS Manager),loyalty.model_loyalty_card,point_of_sale.group_pos_manager,1,1,1,0
access_reward_pos_user,Loyalty Reward (PoS User),loyalty.model_loyalty_reward,point_of_sale.group_pos_user,1,0,0,0
access_reward_pos_manager,Loyalty Reward (PoS Manager),loyalty.model_loyalty_reward,point_of_sale.group_pos_manager,1,1,1,1
access_communication_pos_user,Loyalty Communication (PoS User),loyalty.model_loyalty_mail,point_of_sale.group_pos_user,1,0,0,0
access_communication_pos_manager,Loyalty Communication (PoS Manager),loyalty.model_loyalty_mail,point_of_sale.group_pos_manager,1,1,1,1
access_sale_coupon_generate,Coupon Generation,loyalty.model_loyalty_generate_wizard,point_of_sale.group_pos_user,1,1,1,0
access_loyalty_history_pos_user,Loyalty History (Pos User),loyalty.model_loyalty_history,point_of_sale.group_pos_user,1,1,1,0
access_loyalty_card_update_balance_pos_user,Loyalty Card Update Balance (Pos User),loyalty.model_loyalty_card_update_balance,point_of_sale.group_pos_user,1,1,1,0

```

## File: static\src\overrides\components\control_buttons\control_buttons.js

```javascript
import { ControlButtons } from "@point_of_sale/app/screens/product_screen/control_buttons/control_buttons";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";
import { _t } from "@web/core/l10n/translation";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { makeAwaitable } from "@point_of_sale/app/store/make_awaitable_dialog";
import { patch } from "@web/core/utils/patch";

patch(ControlButtons.prototype, {
    _getEWalletRewards(order) {
        const claimableRewards = order.getClaimableRewards();
        return claimableRewards.filter((reward_line) => {
            const coupon = this.pos.models["loyalty.card"].get(reward_line.coupon_id);
            return (
                coupon &&
                reward_line.reward.program_id.program_type == "ewallet" &&
                !coupon.isExpired()
            );
        });
    },
    _getEWalletPrograms() {
        return this.pos.models["loyalty.program"].filter((p) => p.program_type == "ewallet");
    },
    async onClickWallet() {
        const order = this.pos.get_order();
        const eWalletPrograms = this._getEWalletPrograms();
        const orderTotal = order.get_total_with_tax();
        const eWalletRewards = this._getEWalletRewards(order);
        if (eWalletRewards.length === 0 && orderTotal >= 0) {
            this.dialog.add(AlertDialog, {
                title: _t("No valid eWallet found"),
                body: _t(
                    "You either have not created an eWallet or all your eWallets have expired."
                ),
            });
            return;
        }
        if (orderTotal < 0 && eWalletPrograms.length >= 1) {
            let selectedProgram = null;
            if (eWalletPrograms.length == 1) {
                selectedProgram = eWalletPrograms[0];
            } else {
                selectedProgram = await makeAwaitable(this.dialog, SelectionPopup, {
                    title: _t("Refund with eWallet"),
                    list: eWalletPrograms.map((program) => ({
                        id: program.id,
                        item: program,
                        label: program.name,
                    })),
                });
            }
            if (selectedProgram) {
                this.pos.addLineToCurrentOrder(
                    {
                        product_id: selectedProgram.trigger_product_ids[0],
                        _e_wallet_program_id: selectedProgram,
                        price_unit: -orderTotal,
                    },
                    {}
                );
            }
        } else if (eWalletRewards.length >= 1) {
            let eWalletReward = null;
            if (eWalletRewards.length == 1) {
                eWalletReward = eWalletRewards[0];
            } else {
                eWalletReward = await makeAwaitable(this.dialog, SelectionPopup, {
                    title: _t("Use eWallet to pay"),
                    list: eWalletRewards.map(({ reward, coupon_id }) => ({
                        id: reward.id,
                        item: { reward, coupon_id },
                        label: `${reward.description} (${reward.program_id.name})`,
                    })),
                });
            }
            if (eWalletReward) {
                const result = order._applyReward(
                    eWalletReward.reward,
                    eWalletReward.coupon_id,
                    {}
                );
                if (result !== true) {
                    // Returned an error
                    this.dialog.add(AlertDialog, {
                        title: _t("Error"),
                        body: result,
                    });
                }
                this.pos.updateRewards();
            }
        }
    },
    async clickPromoCode() {
        this.dialog.add(TextInputPopup, {
            title: _t("Enter Code"),
            placeholder: _t("Gift card or Discount code"),
            getPayload: async (code) => {
                code = code.trim();
                if (code !== "") {
                    const res = await this.pos.activateCode(code);
                    if (res !== true) {
                        this.notification.add(res, { type: "danger" });
                    }
                }
            },
        });
    },

    getPotentialRewards() {
        const order = this.pos.get_order();
        // Claimable rewards excluding those from eWallet programs.
        // eWallet rewards are handled in the eWalletButton.
        let rewards = [];
        if (order) {
            const claimableRewards = order.getClaimableRewards();
            rewards = claimableRewards.filter(
                ({ reward }) => reward.program_id.program_type !== "ewallet"
            );
        }
        const result = {};
        const discountRewards = rewards.filter(({ reward }) => reward.reward_type == "discount");
        const freeProductRewards = rewards.filter(({ reward }) => reward.reward_type == "product");
        const potentialFreeProductRewards = this.pos.getPotentialFreeProductRewards();
        const avaiRewards = [
            ...potentialFreeProductRewards,
            ...discountRewards,
            ...freeProductRewards, // Free product rewards at the end of array to prioritize them
        ];

        for (const reward of avaiRewards) {
            result[reward.reward.id] = reward;
        }

        return Object.values(result);
    },

    /**
     * Applies the reward on the current order, if multiple products can be claimed opens a popup asking for which one.
     *
     * @param {Object} reward
     * @param {Integer} coupon_id
     */
    async _applyReward(reward, coupon_id, potentialQty) {
        const order = this.pos.get_order();
        order.uiState.disabledRewards.delete(reward.id);

        const args = {};
        if (reward.reward_type === "product" && reward.multi_product) {
            const productsList = reward.reward_product_ids.map((product_id) => ({
                id: product_id.id,
                label: product_id.display_name,
                item: product_id,
            }));
            const selectedProduct = await makeAwaitable(this.dialog, SelectionPopup, {
                title: _t("Please select a product for this reward"),
                list: productsList,
            });
            if (!selectedProduct) {
                return false;
            }
            args["product"] = selectedProduct;
        }
        if (
            (reward.reward_type == "product" && reward.program_id.applies_on !== "both") ||
            (reward.program_id.applies_on == "both" && potentialQty)
        ) {
            await this.pos.addLineToCurrentOrder(
                {
                    product_id: args["product"] || reward.reward_product_ids[0],
                    qty: potentialQty || 1,
                },
                {}
            );
            return true;
        } else {
            const result = order._applyReward(reward, coupon_id, args);
            if (result !== true) {
                // Returned an error
                this.notification.add(result);
            }
            this.pos.updateRewards();
            return result;
        }
    },
    async clickRewards() {
        const rewards = this.getPotentialRewards();
        if (rewards.length >= 1) {
            const rewardsList = rewards.map((reward) => ({
                id: reward.reward.id,
                label: reward.reward.program_id.name,
                description: `Add "${reward.reward.description}"`,
                item: reward,
            }));
            this.dialog.add(SelectionPopup, {
                title: _t("Available rewards"),
                list: rewardsList,
                getPayload: (selectedReward) => {
                    this._applyReward(
                        selectedReward.reward,
                        selectedReward.coupon_id,
                        selectedReward.potentialQty
                    );
                },
            });
        }
    },
});

```

## File: static\src\overrides\components\control_buttons\control_buttons.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_loyalty.ControlButtons" t-inherit="point_of_sale.ControlButtons" t-inherit-mode="extension">
        <xpath expr="//t[@t-if='props.showRemainingButtons']/div/OrderlineNoteButton" position="after">
            <t t-if="pos.models['loyalty.program'].some((p) => p.program_type == 'ewallet')">
                <t t-set="_orderTotal" t-value="pos.get_order().get_total_with_tax()" />
                <t t-set="_eWalletPrograms" t-value="_getEWalletPrograms()" />
                <t t-set="_eWalletRewards" t-value="_getEWalletRewards(pos.get_order())" />
                <button t-att-class="buttonClass"
                    t-on-click="onClickWallet"
                    t-attf-class="{{(_orderTotal lt 0 and _eWalletPrograms.length gte 1) or _eWalletRewards.length gte 1 ? 'highlight text-action': ''}}">
                    <i class="fa fa-credit-card me-1" />
                    <t t-if="_orderTotal lt 0 and _eWalletPrograms.length">eWallet Refund</t>
                    <t t-elif="_eWalletRewards.length">eWallet Pay</t>
                    <t t-else="">eWallet</t>
                </button>
            </t>
            <t t-if="pos.models['loyalty.program'].some((p) => ['coupons', 'promotion', 'gift_card', 'promo_code', 'next_order_coupons'].includes(p.program_type))">
                <button t-att-class="buttonClass"
                    t-on-click="() => this.clickPromoCode()">
                    <i class="fa fa-barcode me-1"/>Enter Code
                </button>
            </t>
            <t t-if="pos.models['loyalty.program'].length">
                <button class="control-button"
                    t-att-class="buttonClass"
                    t-attf-class="{{getPotentialRewards().length ? 'highlight text-action' : 'disabled'}}"
                    t-on-click="() => this.clickRewards()">
                    <i class="fa fa-star me-1"/>Reward
                </button>
            </t>
        </xpath>
        <xpath expr="//t[@t-if='props.showRemainingButtons']/div/OrderlineNoteButton" position="after">
            <t t-if="pos.models['loyalty.program'].some((p) => ['coupons', 'promotion'].includes(p.program_type))">
                <button class="btn btn-secondary btn-lg py-5" t-att-class="{'disabled': !pos.get_order().isProgramsResettable()}"
                    t-on-click="() => this.pos.resetPrograms()">
                    <i class="fa fa-star me-1"/>Reset Programs
                </button>
            </t>
        </xpath>
        <xpath expr="//button[hasclass('more-btn')]" position="attributes">
            <attribute name="t-attf-class">{{ getPotentialRewards().length ? 'active text-action' : '' }}</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_coupon.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt')]//div[hasclass('before-footer')]" position="inside">
            <t t-foreach="props.data.loyaltyStats or []" t-as="_loyaltyStat" t-key="_loyaltyStat.couponId">
                <!-- Show only if portal_visible. -->
                <div t-if="_loyaltyStat.program.portal_visible and (_loyaltyStat.points.won || _loyaltyStat.points.spent)" class='loyalty'>
                    <span class="pos-receipt-center-align">
                        <div>--------------------------------</div>
                    </span>
                    <t t-if='_loyaltyStat.points.won'>
                        <div><t t-esc="_loyaltyStat.points.name"/> Won: <span t-esc='_loyaltyStat.points.won' class="pos-receipt-right-align"/></div>
                    </t>
                    <t t-if='_loyaltyStat.points.spent'>
                        <div><t t-esc="_loyaltyStat.points.name"/> Spent: <span t-esc='_loyaltyStat.points.spent' class="pos-receipt-right-align"/></div>
                    </t>
                    <!-- Don't use points.total, it's wrong in this context (after the order synced). -->
                    <!-- Show balance as it's updated during _postPushOrderResolve. -->
                    <t t-if='_loyaltyStat.points.balance'>
                        <div>Balance <t t-esc="_loyaltyStat.points.name"/>: <span t-esc='_loyaltyStat.points.balance' class="pos-receipt-right-align"/></div>
                    </t>
                </div>
            </t>
            <t t-if="props.data.partner">
                <br/>
                <div>Customer <span t-esc='props.data.partner.name' class='pos-receipt-right-align'/></div>
            </t>
            <t t-if="props.data.new_coupon_info and props.data.new_coupon_info.length !== 0">
                <div class="pos-coupon-rewards">
                    <div>------------------------</div>
                    <br/>
                    <div>
                        Coupon Codes
                    </div>
                    <t t-foreach="props.data.new_coupon_info" t-as="coupon_info" t-key="coupon_info.code">
                        <div class="coupon-container">
                            <div style="font-size: 110%;">
                                <t t-esc="coupon_info['program_name']"/>
                            </div>
                            <div>
                                <span>Valid until: </span> 
                                <t t-if="coupon_info['expiration_date']">
                                    <t t-esc="coupon_info['expiration_date']"/>
                                </t>
                                <t t-else="">
                                    no expiration
                                </t>
                            </div>
                            <div>
                                <img t-att-src="'/report/barcode/Code128/'+coupon_info['code']" style="width:200px;height:50px" alt="Barcode"/>
                            </div>
                            <div>
                                <t t-esc="coupon_info['code']"/>
                            </div>
                        </div>
                    </t>
                </div>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\partner_line\partner_line.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { PartnerLine } from "@point_of_sale/app/screens/partner_list/partner_line/partner_line";
import { patch } from "@web/core/utils/patch";
import { formatFloat } from "@web/core/utils/numbers";

patch(PartnerLine.prototype, {
    setup() {
        super.setup(...arguments);
        this.pos = usePos();
    },
    _getLoyaltyPointsRepr(loyaltyCard) {
        const program = loyaltyCard.program_id;
        if (program.program_type === "ewallet") {
            return `${program.name}: ${this.env.utils.formatCurrency(loyaltyCard.points)}`;
        }
        const balanceRepr = formatFloat(loyaltyCard.points, { digits: [69, 2] });
        if (program.portal_visible) {
            return `${balanceRepr} ${program.portal_point_name}`;
        }
        return _t("%s Points", balanceRepr);
    },
});

```

## File: static\src\overrides\components\partner_line\partner_line.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="pos_loyalty.PartnerLine" t-inherit="point_of_sale.PartnerLine" t-inherit-mode="extension">
        <xpath expr="//td[hasclass('partner-line-balance')]" position="inside">
            <t t-set="_loyaltyCards" t-value="pos.getLoyaltyCards(props.partner)" />
            <t t-foreach="_loyaltyCards" t-as="_loyaltyCard" t-key="_loyaltyCard.id">
                <div class="pos-right-align">
                    <t t-esc="_getLoyaltyPointsRepr(_loyaltyCard)"/>
                </div>
            </t>
        </xpath>
    </t>

 </templates>

```

## File: static\src\overrides\components\partner_list_screen\partner_list_screen.js

```javascript
import { PartnerList } from "@point_of_sale/app/screens/partner_list/partner_list";
import { patch } from "@web/core/utils/patch";

patch(PartnerList.prototype, {
    /**
     * Needs to be set to true to show the loyalty points in the partner list.
     * @override
     */
    get isBalanceDisplayed() {
        return true;
    },

    async searchPartner() {
        const res = await super.searchPartner();
        const programIds = this.pos.models["loyalty.program"].getAll().map((p) => p.id);
        const coupons = await this.pos.fetchCoupons(
            [
                ["partner_id", "in", res.map((partner) => partner.id)],
                ["program_id.active", "=", true],
                ["program_id", "in", programIds],
                ["points", ">", 0],
            ],
            0
        );
        this.pos.computePartnerCouponIds(coupons);
        return res;
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { omit } from "@web/core/utils/objects";
import { useService } from "@web/core/utils/hooks";

patch(PaymentScreen.prototype, {
    setup() {
        super.setup(...arguments);
        this.report = useService("report");
    },
    //@override
    async validateOrder(isForceValidate) {
        const pointChanges = {};
        const newCodes = [];
        for (const pe of Object.values(this.currentOrder.uiState.couponPointChanges)) {
            if (pe.coupon_id > 0) {
                pointChanges[pe.coupon_id] = pe.points;
            } else if (pe.barcode && !pe.giftCardId) {
                // New coupon with a specific code, validate that it does not exist
                newCodes.push(pe.barcode);
            }
        }
        for (const line of this.currentOrder._get_reward_lines()) {
            if (line.coupon_id.id < 1) {
                continue;
            }
            if (!pointChanges[line.coupon_id.id]) {
                pointChanges[line.coupon_id.id] = -line.points_cost;
            } else {
                pointChanges[line.coupon_id.id] -= line.points_cost;
            }
        }
        if (!(await this._isOrderValid(isForceValidate))) {
            return;
        }
        // No need to do an rpc if no existing coupon is being used.
        if (Object.keys(pointChanges || {}).length > 0 || newCodes.length) {
            try {
                const { successful, payload } = await this.pos.data.call(
                    "pos.order",
                    "validate_coupon_programs",
                    [[], pointChanges, newCodes]
                );
                // Payload may contain the points of the concerned coupons to be updated in case of error. (So that rewards can be corrected)
                if (payload && payload.updated_points) {
                    for (const pointChange of Object.entries(payload.updated_points)) {
                        const coupon = this.pos.models["loyalty.card"].get(pointChange[0]);
                        if (coupon) {
                            coupon.points = pointChange[1];
                        }
                    }
                }
                if (payload && payload.removed_coupons) {
                    for (const couponId of payload.removed_coupons) {
                        const coupon = this.pos.models["loyalty.card"].get(couponId);
                        coupon && coupon.delete();
                    }
                }
                if (!successful) {
                    this.dialog.add(AlertDialog, {
                        title: _t("Error validating rewards"),
                        body: payload.message,
                    });
                    return;
                }
            } catch {
                // Do nothing with error, while this validation step is nice for error messages
                // it should not be blocking.
            }
        }
        await super.validateOrder(...arguments);
    },
    /**
     * @override
     */
    async _postPushOrderResolve(order, server_ids) {
        const orders = this.pos.models["pos.order"]
            .readMany(server_ids)
            .filter((o) => !["draft", "cancel"].includes(o.state));
        for (const order of orders) {
            await this._postProcessLoyalty(order);
        }
        return super._postPushOrderResolve(order, server_ids);
    },
    async _postProcessLoyalty(order) {
        // Compile data for our function
        const ProgramModel = this.pos.models["loyalty.program"];
        const rewardLines = order._get_reward_lines();
        const partner = order.get_partner();
        let couponData = Object.values(order.uiState.couponPointChanges).reduce((agg, pe) => {
            agg[pe.coupon_id] = Object.assign({}, pe, {
                points: pe.points - order._getPointsCorrection(ProgramModel.get(pe.program_id)),
            });
            const program = ProgramModel.get(pe.program_id);
            if (
                (program.is_nominative || program.program_type == "next_order_coupons") &&
                partner
            ) {
                agg[pe.coupon_id].partner_id = partner.id;
            }
            if (program.program_type != "loyalty") {
                agg[pe.coupon_id].expiration_date = program.date_to || pe.expiration_date;
            }
            return agg;
        }, {});
        for (const line of rewardLines) {
            const reward = line.reward_id;
            const couponId = line.coupon_id.id;
            if (!couponData[couponId]) {
                couponData[couponId] = {
                    points: 0,
                    program_id: reward.program_id.id,
                    coupon_id: couponId,
                    barcode: false,
                };
                if (reward.program_type != "loyalty") {
                    couponData[couponId].expiration_date = reward.program_id.date_to;
                }
            }
            if (!couponData[couponId].line_codes) {
                couponData[couponId].line_codes = [];
            }
            if (!couponData[couponId].line_codes.includes(line.reward_identifier_code)) {
                !couponData[couponId].line_codes.push(line.reward_identifier_code);
            }
            couponData[couponId].points -= line.points_cost;
        }
        // We actually do not care about coupons for 'current' programs that did not claim any reward, they will be lost if not validated
        couponData = Object.fromEntries(
            Object.entries(couponData)
                .filter(([key, value]) => {
                    const program = ProgramModel.get(value.program_id);
                    if (program.applies_on === "current") {
                        return value.line_codes && value.line_codes.length;
                    }
                    return true;
                })
                .map(([key, value]) => [key, omit(value, "appliedRules")])
        );
        if (Object.keys(couponData || {}).length > 0) {
            const payload = await this.pos.data.call("pos.order", "confirm_coupon_programs", [
                order.id,
                couponData,
            ]);
            if (payload.coupon_updates) {
                for (const couponUpdate of payload.coupon_updates) {
                    // The following code is a workaround to update the id of an existing record.
                    // It's so ugly.
                    // FIXME: Find a better way of updating the id of an existing record.
                    // It would be better if we can do this:
                    // const coupon = this.pos.models["loyalty.card"].get(couponUpdate.old_id);
                    // coupon.update({ id: couponUpdate.id, points: couponUpdate.points })

                    if (couponUpdate.old_id == couponUpdate.id) {
                        // just update the points
                        const coupon = this.pos.models["loyalty.card"].get(couponUpdate.id);

                        if (!coupon) {
                            await this.pos.data.read("loyalty.card", [couponUpdate.id]);
                        } else {
                            coupon.update({ points: couponUpdate.points });
                        }
                    } else {
                        // create a new coupon and delete the old one
                        const coupon = this.pos.models["loyalty.card"].create({
                            id: couponUpdate.id,
                            code: couponUpdate.code,
                            program_id: this.pos.models["loyalty.program"].get(
                                couponUpdate.program_id
                            ),
                            partner_id: this.pos.models["res.partner"].get(couponUpdate.partner_id),
                            points: couponUpdate.points,
                        });

                        // Before deleting the old coupon, update the order lines that use it.
                        for (const line of order.lines) {
                            if (line.coupon_id?.id == couponUpdate.old_id) {
                                line.update({ coupon_id: coupon });
                            }
                        }

                        this.pos.models["loyalty.card"].get(couponUpdate.old_id)?.delete();
                    }
                }
            }

            const loyaltyPoints = Object.keys(couponData).map((coupon_id) => ({
                order_id: order.id,
                card_id: coupon_id,
                spent: couponData[coupon_id].points < 0 ? -couponData[coupon_id].points : 0,
                won: couponData[coupon_id].points > 0 ? couponData[coupon_id].points : 0,
            }));

            const couponUpdates = payload.coupon_updates.map((item) => ({
                id: item.id,
                old_id: item.old_id,
            }));
            this.pos.data.call("pos.order", "add_loyalty_history_lines", [
                [this.currentOrder.id],
                loyaltyPoints,
                couponUpdates,
            ]);
            // Update the usage count since it is checked based on local data
            if (payload.program_updates) {
                for (const programUpdate of payload.program_updates) {
                    const program = ProgramModel.get(programUpdate.program_id);
                    if (program) {
                        program.total_order_count = programUpdate.usages;
                    }
                }
            }
            if (payload.coupon_report) {
                for (const [actionId, active_ids] of Object.entries(payload.coupon_report)) {
                    await this.report.doAction(actionId, active_ids);
                }
                order.has_pdf_gift_card = Object.keys(payload.coupon_report).length > 0;
            }
            order.new_coupon_info = payload.new_coupon_info;
        }
    },
});

```

## File: static\src\overrides\components\product_screen\product_screen.js

```javascript
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useBarcodeReader } from "@point_of_sale/app/barcode/barcode_reader_hook";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";

patch(ProductScreen.prototype, {
    setup() {
        super.setup(...arguments);
        this.notification = useService("notification");
        useBarcodeReader({
            coupon: this._onCouponScan,
        });
    },
    async _onCouponScan(code) {
        // IMPROVEMENT: Ability to understand if the scanned code is to be paid or to be redeemed.
        const res = await this.pos.activateCode(code.base_code);
        if (res !== true) {
            this.notification.add(res, { type: "danger" });
        }
    },
    async _barcodeProductAction(code) {
        await super._barcodeProductAction(code);
        this.pos.updateRewards();
    },
    async _barcodeGS1Action(code) {
        await super._barcodeGS1Action(code);
        this.pos.updateRewards();
    },
    async _barcodePartnerAction(code) {
        await super._barcodePartnerAction(code);
        this.pos.updateRewards();
    },
});

```

## File: static\src\overrides\components\product_screen\order_summary\order_summary.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { OrderSummary } from "@point_of_sale/app/screens/product_screen/order_summary/order_summary";
import { patch } from "@web/core/utils/patch";
import { ask } from "@point_of_sale/app/store/make_awaitable_dialog";
import { useService } from "@web/core/utils/hooks";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { ManageGiftCardPopup } from "@pos_loyalty/utils/manage_giftcard_popup/manage_giftcard_popup";

patch(OrderSummary.prototype, {
    setup() {
        super.setup(...arguments);
        this.notification = useService("notification");
    },
    async updateSelectedOrderline({ buffer, key }) {
        const selectedLine = this.currentOrder.get_selected_orderline();
        if (key === "-") {
            if (selectedLine && selectedLine._e_wallet_program_id) {
                // Do not allow negative quantity or price in a gift card or ewallet orderline.
                // Refunding gift card or ewallet is not supported.
                this.notification.add(
                    _t("You cannot set negative quantity or price to gift card or ewallet."),
                    4000
                );
                return;
            }
        }
        if (
            selectedLine &&
            selectedLine.is_reward_line &&
            !selectedLine.manual_reward &&
            (key === "Backspace" || key === "Delete")
        ) {
            const reward = selectedLine.reward_id;
            const confirmed = await ask(this.dialog, {
                title: _t("Deactivating reward"),
                body: _t(
                    "Are you sure you want to remove %s from this order?\n You will still be able to claim it through the reward button.",
                    reward.description
                ),
                cancelLabel: _t("No"),
                confirmLabel: _t("Yes"),
            });
            if (confirmed) {
                buffer = null;
            } else {
                // Cancel backspace
                return;
            }
        }
        return super.updateSelectedOrderline({ buffer, key });
    },
    /**
     * 1/ Perform the usual set value operation (super._setValue(val)) if the line being modified
     * is not a reward line or if it is a reward line, the `val` being set is '' or 'remove' only.
     *
     * 2/ Update activated programs and coupons when removing a reward line.
     *
     * 3/ Trigger 'update-rewards' if the line being modified is a regular line or
     * if removing a reward line.
     *
     * @override
     */
    _setValue(val) {
        const selectedLine = this.currentOrder.get_selected_orderline();
        if (!selectedLine) {
            return;
        }
        if (selectedLine.is_reward_line && val === "remove") {
            this.currentOrder.uiState.disabledRewards.add(selectedLine.reward_id.id);
            const coupon = selectedLine.coupon_id;
            if (
                coupon &&
                coupon.id > 0 &&
                this.currentOrder._code_activated_coupon_ids.find((c) => c.code === coupon.code)
            ) {
                coupon.delete();
            }
        }
        if (
            !selectedLine ||
            !selectedLine.is_reward_line ||
            (selectedLine.is_reward_line && ["", "remove"].includes(val))
        ) {
            super._setValue(val);
        }
        if (!selectedLine.is_reward_line || (selectedLine.is_reward_line && val === "remove")) {
            this.pos.updateRewards();
        }
    },

    async _showDecreaseQuantityPopup() {
        const result = await super._showDecreaseQuantityPopup();
        if (result) {
            this.pos.updateRewards();
        }
    },

    /**
     * Updates the order line with the gift card information:
     * 1. Reduce the quantity if greater than one, otherwise remove the order line.
     * 2. Add a new order line with updated gift card code and points, removing any existing related couponPointChanges.
     */
    async _updateGiftCardOrderline(code, points) {
        let selectedLine = this.currentOrder.get_selected_orderline();
        const product = selectedLine.product_id;

        if (selectedLine.get_quantity() > 1) {
            selectedLine.set_quantity(selectedLine.get_quantity() - 1);
        } else {
            this.currentOrder.removeOrderline(selectedLine);
        }

        const program = this.pos.models["loyalty.program"].find(
            (p) => p.program_type === "gift_card"
        );
        const existingCouponIds = Object.keys(this.currentOrder.uiState.couponPointChanges).filter(
            (key) => {
                const change = this.currentOrder.uiState.couponPointChanges[key];
                return (
                    change.points === product.lst_price &&
                    change.program_id === program.id &&
                    change.product_id === product.id &&
                    !change.manual
                );
            }
        );
        if (existingCouponIds.length) {
            const couponId = existingCouponIds.shift();
            delete this.currentOrder.uiState.couponPointChanges[couponId];
        }

        await this.pos.addLineToCurrentOrder({ product_id: product }, { price_unit: points });
        selectedLine = this.currentOrder.get_selected_orderline();
        selectedLine.gift_code = code;
    },

    manageGiftCard() {
        this.dialog.add(ManageGiftCardPopup, {
            title: _t("Sell/Manage physical gift card"),
            placeholder: _t("Enter Gift Card Number"),
            getPayload: async (code, points, expirationDate) => {
                points = parseFloat(points);
                if (isNaN(points)) {
                    console.error("Invalid amount value:", points);
                    return;
                }
                code = code.trim();
                const res = await this.pos.data.searchRead(
                    "loyalty.card",
                    ["&", ["program_type", "=", "gift_card"], ["code", "=", code]],
                    []
                );
                if (res.length > 0) {
                    this.notification.add(_t("This Gift card has already been sold."), {
                        type: "danger",
                    });
                    return;
                }

                // check for duplicate code
                if (this.currentOrder.duplicateCouponChanges(code)) {
                    this.dialog.add(ConfirmationDialog, {
                        title: _t("Validation Error"),
                        body: _t("A coupon/loyalty card must have a unique code."),
                    });
                    return;
                }

                await this._updateGiftCardOrderline(code, points);
                this.currentOrder.processGiftCard(code, points, expirationDate);

                // update indexedDB
                this.pos.data.syncDataWithIndexedDB(this.pos.data.records);
            },
        });
    },

    clickLine(ev, orderline) {
        if (orderline.isSelected() && orderline.getEWalletGiftCardProgramType() === "gift_card") {
            return;
        } else {
            super.clickLine(ev, orderline);
        }
    },
});

```

## File: static\src\overrides\components\product_screen\order_summary\order_summary.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_loyalty.OrderSummary" t-inherit="point_of_sale.OrderSummary" t-inherit-mode="extension">
		<xpath expr="//Orderline" position="inside" >
            <li t-if="line.isGiftCardOrEWalletReward()">
                Current Balance: <t t-esc="line.getGiftCardOrEWalletBalance()"/>
            </li>
            <t t-if="!line.isGiftCardOrEWalletReward() and line.getEWalletGiftCardProgramType() === 'gift_card'">
                <a t-if="!line.gift_code" class="text-wrap text-primary" t-on-click="manageGiftCard">Sell physical gift card?</a>
                <div t-if="line.gift_code" class="text-wrap" t-esc="line.gift_code"/>
            </t>
        </xpath>
        <xpath expr="//OrderWidget/t[@t-set-slot='details']" position="inside">
            <t t-foreach="pos.get_order()?.getLoyaltyPoints() or []" t-as="_loyaltyStat" t-key="_loyaltyStat.couponId">
                <div t-if="_loyaltyStat.points.won || _loyaltyStat.points.spent" class="d-flex justify-content-between mt-2 px-3 py-2 rounded-3 bg-white">
                    <div t-esc="_loyaltyStat.points.name" class="loyalty-points-title fs-4 fw-bolder"/>
                    <div class="d-flex gap-1 fw-bold">
                        <div t-if='_loyaltyStat.points.balance' class="loyalty-points loyalty-points-balance">
                            <span class='value'><t t-esc='_loyaltyStat.points.balance'/></span>
                        </div>
                        <div t-if='_loyaltyStat.points.won' class="loyalty-points loyalty-points-won">
                            <span class='value text-success'>+<t t-esc='_loyaltyStat.points.won'/></span>
                        </div>
                        <div t-if='_loyaltyStat.points.spent' class="loyalty-points loyalty-points-spent">
                            <span class='value text-danger'>-<t t-esc='_loyaltyStat.points.spent'/></span>
                        </div>
                        =
                        <div class="loyalty-points loyalty-points-totaltext-end fw-bolder">
                            <span class='value'><t t-esc='_loyaltyStat.points.total'/></span>
                        </div>
                    </div>
                </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

/**
 * Prevent refunding ewallet/gift card lines.
 */
patch(TicketScreen.prototype, {
    setup() {
        super.setup(...arguments);
        this.notification = useService("notification");
    },
    _onUpdateSelectedOrderline() {
        const order = this.getSelectedOrder();
        if (!order) {
            return this.numberBuffer.reset();
        }
        const selectedOrderlineId = this.getSelectedOrderlineId();
        const orderline = order.lines.find((line) => line.id == selectedOrderlineId);
        if (orderline && this._isEWalletGiftCard(orderline)) {
            this._showNotAllowedRefundNotification();
            return this.numberBuffer.reset();
        }
        return super._onUpdateSelectedOrderline(...arguments);
    },
    _prepareAutoRefundOnOrder(order) {
        const selectedOrderlineId = this.getSelectedOrderlineId();
        const orderline = order.lines.find((line) => line.id == selectedOrderlineId);
        if (this._isEWalletGiftCard(orderline)) {
            this._showNotAllowedRefundNotification();
            return false;
        }
        return super._prepareAutoRefundOnOrder(...arguments);
    },
    _showNotAllowedRefundNotification() {
        this.notification.add(
            _t(
                "Refunding a top up or reward product for an eWallet or gift card program is not allowed."
            ),
            5000
        );
    },
    _isEWalletGiftCard(orderline) {
        if (orderline.is_reward_line) {
            const reward = orderline.reward_id;
            const program = reward && reward.program_id;
            if (program && ["gift_card", "ewallet"].includes(program.program_type)) {
                return true;
            }
        }
        return false;
    },
});

```

## File: static\src\overrides\models\data_service_options.js

```javascript
import { DataServiceOptions } from "@point_of_sale/app/models/data_service_options";
import { patch } from "@web/core/utils/patch";

patch(DataServiceOptions.prototype, {
    get databaseTable() {
        return {
            ...super.databaseTable,
            "loyalty.card": {
                key: "id",
                condition: (record) => {
                    return record["<-pos.order.line.coupon_id"].find(
                        (l) => !(l.order_id?.finalized && typeof l.order_id.id === "number")
                    );
                },
            },
        };
    },
    get pohibitedAutoLoadedModels() {
        return [
            ...super.pohibitedAutoLoadedModels,
            "loyalty.program",
            "loyalty.rule",
            "loyalty.reward",
        ];
    },
});

```

## File: static\src\overrides\models\loyalty_card.js

```javascript
import { registry } from "@web/core/registry";
import { Base } from "@point_of_sale/app/models/related_models";

const { DateTime } = luxon;

export class LoyaltyCard extends Base {
    static pythonModel = "loyalty.card";

    isExpired() {
        // If no expiration date is set, the card is not expired
        if (!this.expiration_date) {
            return false;
        }

        return DateTime.fromISO(this.expiration_date).toMillis() < DateTime.now().toMillis();
    }
}

registry.category("pos_available_models").add(LoyaltyCard.pythonModel, LoyaltyCard);

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";
import { roundDecimals, roundPrecision } from "@web/core/utils/numbers";
import { _t } from "@web/core/l10n/translation";
import { loyaltyIdsGenerator } from "./pos_store";
import { compute_price_force_price_include } from "@point_of_sale/app/models/utils/tax_utils";
const { DateTime } = luxon;

function _newRandomRewardCode() {
    return (Math.random() + 1).toString(36).substring(3);
}

let pointsForProgramsCountedRules = {};

/**
 * Calculate the number of free items based on the given number
 * of items `number_items` and the rule: buy `n` take `m`.
 *
 * e.g.
 *```
 *      rule: buy 2 take 1                    rule: buy 2 take 3
 *     +------------+--------+--------+      +------------+--------+--------+
 *     |number_items| charged|    free|      |number_items| charged|    free|
 *     +------------+--------+--------+      +------------+--------+--------+
 *     |           1|       1|       0|      |           1|       1|       0|
 *     |           2|       2|       0|      |           2|       2|       0|
 *     |           3|       2|       1|      |           3|       2|       1|
 *     |           4|       3|       1|      |           4|       2|       2|
 *     |           5|       4|       1|      |           5|       2|       3|
 *     |           6|       4|       2|      |           6|       3|       3|
 *     |           7|       5|       2|      |           7|       4|       3|
 *     |           8|       6|       2|      |           8|       4|       4|
 *     |           9|       6|       3|      |           9|       4|       5|
 *     |          10|       7|       3|      |          10|       4|       6|
 *     +------------+--------+--------+      +------------+--------+--------+
 * ```
 *
 * @param {number} numberItems number of items
 * @param {number} n items to buy
 * @param {number} m item for free
 * @returns {number} number of free items
 */
function computeFreeQuantity(numberItems, n, m) {
    const factor = Math.trunc(numberItems / (n + m));
    const free = factor * m;
    const charged = numberItems - free;
    // adjust the calculated free quantities
    const x = (factor + 1) * n;
    const y = x + (factor + 1) * m;
    const adjustment = x <= charged && charged < y ? charged - x : 0;
    return Math.floor(free + adjustment);
}

patch(PosOrder, {
    extraFields: {
        ...(PosOrder.extraFields || {}),
        _code_activated_coupon_ids: {
            model: "pos.order",
            name: "_code_activated_coupon_ids",
            relation: "loyalty.card",
            type: "one2many",
            local: true,
        },
    },
});

patch(PosOrder.prototype, {
    setup() {
        super.setup(...arguments);
        // Always start with invalid coupons so that coupon for this
        // order is properly assigned. @see _checkMissingCoupons
        this.invalidCoupons = true;
        this.uiState = {
            ...this.uiState,
            disabledRewards: this.uiState.disabledRewards || new Set(),
            codeActivatedProgramRules: this.uiState.codeActivatedProgramRules || [],
            couponPointChanges: this.uiState.couponPointChanges || {},
        };
        const oldCouponMapping = {};
        if (Object.keys(this.uiState.couponPointChanges).length === 0) {
            for (const [key, pe] of Object.entries(this.uiState.couponPointChanges)) {
                if (!this.models["loyalty.program"].get(pe.program_id)) {
                    // Remove points changes for programs that are not available anymore.
                    delete this.uiState.couponPointChanges[key];
                    continue;
                }
                if (pe.coupon_id > 0) {
                    continue;
                }
                const newId = loyaltyIdsGenerator();
                delete oldCouponMapping[pe.coupon_id];
                pe.coupon_id = newId;
                this.uiState.couponPointChanges[newId] = pe;
            }
        }
    },
    setupState(vals) {
        super.setupState(...arguments);
        this.uiState.disabledRewards = new Set(vals?.disabledRewards || []);
    },
    serializeState() {
        const state = super.serializeState(...arguments);
        state.disabledRewards = [...(this.uiState.disabledRewards || [])];
        return state;
    },
    /** @override */
    getEmailItems() {
        return super
            .getEmailItems(...arguments)
            .concat(this.has_pdf_gift_card ? [_t("the gift cards")] : []);
    },

    /**
     * We need to update the rewards upon changing the partner as it may impact the points available
     *  for rewards.
     *
     * @override
     */
    set_partner(partner) {
        const oldPartner = this.get_partner();
        super.set_partner(partner);
        if (this.uiState.couponPointChanges && oldPartner !== this.get_partner()) {
            // Remove couponPointChanges for cards in is_nominative programs.
            // This makes sure that counting of points on loyalty and ewallet programs is updated after partner changes.
            const loyaltyProgramIds = new Set(
                this.models["loyalty.program"]
                    .filter((program) => program.is_nominative)
                    .map((program) => program.id)
            );
            for (const [key, pointChange] of Object.entries(this.uiState.couponPointChanges)) {
                if (loyaltyProgramIds.has(pointChange.program_id)) {
                    delete this.uiState.couponPointChanges[key];
                }
            }
        }
    },
    wait_for_push_order() {
        return (
            Object.keys(this.uiState.couponPointChanges || {}).length > 0 ||
            this._get_reward_lines().length ||
            super.wait_for_push_order(...arguments)
        );
    },
    /**
     * Add additional information for our ticket, such as new coupons and loyalty point gains.
     *
     * @override
     */
    export_for_printing(baseUrl, headerData) {
        const result = super.export_for_printing(...arguments);
        if (this.get_partner()) {
            result.loyaltyStats = this.getLoyaltyPoints();
            result.partner = this.get_partner();
        }
        result.new_coupon_info = this.new_coupon_info;
        return result;
    },
    //@override
    _get_ignored_product_ids_total_discount() {
        const productIds = super._get_ignored_product_ids_total_discount(...arguments);
        const giftCardPrograms = this.models["loyalty.program"].filter(
            (p) => p.program_type === "gift_card"
        );
        for (const program of giftCardPrograms) {
            const giftCardProductId = [...program.rule_ids[0].valid_product_ids][0];
            if (giftCardProductId) {
                productIds.push(giftCardProductId);
            }
        }
        return productIds;
    },
    get_orderlines() {
        const orderlines = super.get_orderlines(this, arguments);
        const rewardLines = [];
        const nonRewardLines = [];

        for (const line of orderlines) {
            if (line.is_reward_line) {
                rewardLines.push(line);
            } else {
                nonRewardLines.push(line);
            }
        }

        return [...nonRewardLines, ...rewardLines];
    },
    _get_reward_lines() {
        if (this.lines) {
            return this.lines.filter((line) => line.is_reward_line);
        }
        return this.lines;
    },
    _get_regular_order_lines() {
        if (this.lines) {
            return this.lines.filter((line) => !line.is_reward_line && !line.refunded_orderline_id);
        }
        return this.lines;
    },
    get_last_orderline() {
        const orderLines = this.lines.filter((line) => !line.is_reward_line);
        return orderLines[orderLines.length - 1];
    },
    set_pricelist(pricelist) {
        const oldPricelist = this.pricelist_id;
        super.set_pricelist(...arguments);
        if (this.uiState.couponPointChanges && oldPricelist !== pricelist) {
            // Remove couponPointChanges for cards in no longer available programs.
            // This makes sure that counting of points on loyalty and ewallet programs is updated after pricelist changes.
            const loyaltyProgramIds = new Set(
                this.models["loyalty.program"]
                    .filter(
                        (program) =>
                            program.pricelist_ids.length > 0 &&
                            (!pricelist ||
                                !program.pricelist_ids.some((pl) => pl.id === pricelist.id))
                    )
                    .map((program) => program.id)
            );
            for (const [key, pointChange] of Object.entries(this.uiState.couponPointChanges)) {
                if (loyaltyProgramIds.has(pointChange.program_id)) {
                    delete this.uiState.couponPointChanges[key];
                }
            }
        }
    },
    _resetPrograms() {
        this.uiState.disabledRewards = new Set();
        this.uiState.codeActivatedProgramRules = [];
        this.uiState.couponPointChanges = {};
        for (const rewardLine of this.lines.filter((line) => line.is_reward_line)) {
            rewardLine.delete();
        }
        this.update({ _code_activated_coupon_ids: [["clear"]] });
    },
    /**
     * Refreshes the currently applied rewards, if they are not applicable anymore they are removed.
     */
    _updateRewardLines() {
        if (!this.lines.length) {
            return;
        }
        const rewardLines = this._get_reward_lines();
        if (!rewardLines.length) {
            return;
        }
        const productRewards = [];
        const otherRewards = [];
        const paymentRewards = []; // Gift card and ewallet rewards are considered payments and must stay at the end
        for (const line of rewardLines) {
            const claimedReward = {
                reward: line.reward_id,
                coupon_id: line.coupon_id?.id,
                args: {
                    product: line._reward_product_id,
                    price: line.price_unit,
                    quantity: line.qty,
                    cost: line.points_cost,
                },
                reward_identifier_code: line.reward_identifier_code,
            };
            if (
                claimedReward.reward.program_id.program_type === "gift_card" ||
                claimedReward.reward.program_id.program_type === "ewallet"
            ) {
                paymentRewards.push(claimedReward);
            } else if (claimedReward.reward.reward_type === "product") {
                productRewards.push(claimedReward);
            } else if (
                !otherRewards.some(
                    (reward) =>
                        reward.reward_identifier_code === claimedReward.reward_identifier_code
                )
            ) {
                otherRewards.push(claimedReward);
            }
            line.delete();
        }
        const allRewards = productRewards.concat(otherRewards).concat(paymentRewards);
        const allRewardsMerged = [];
        allRewards.forEach((reward) => {
            if (reward.reward.reward_type == "discount") {
                allRewardsMerged.push(reward);
            } else {
                const reward_index = allRewardsMerged.findIndex(
                    (item) =>
                        item.reward.id === reward.reward.id && item.args.price === reward.args.price
                );
                if (reward_index > -1) {
                    allRewardsMerged[reward_index].args.quantity += reward.args.quantity;
                    allRewardsMerged[reward_index].args.cost += reward.args.cost;
                } else {
                    allRewardsMerged.push(reward);
                }
            }
        });

        for (const claimedReward of allRewardsMerged) {
            // For existing coupons check that they are still claimed, they can exist in either `couponPointChanges` or `codeActivatedCoupons`
            if (
                !this._code_activated_coupon_ids.find(
                    (coupon) => coupon.id === claimedReward.coupon_id
                ) &&
                !this.uiState.couponPointChanges[claimedReward.coupon_id]
            ) {
                continue;
            }
            if (
                claimedReward.reward.program_id.program_type === "coupons" &&
                this.lines.find(
                    (rewardline) => rewardline.reward_id?.id === claimedReward.reward.id
                )
            ) {
                continue;
            }
            this._applyReward(claimedReward.reward, claimedReward.coupon_id, claimedReward.args);
        }
    },
    /**
     * @typedef {{ won: number, spend: number, total: number, balance: number, name: string}} LoyaltyPoints
     * @typedef {{ couponId: number, program: object, points: LoyaltyPoints}} LoyaltyStat
     * @returns {Array<LoyaltyStat>}
     */
    getLoyaltyPoints() {
        // map: couponId -> LoyaltyPoints
        const loyaltyPoints = {};
        for (const pointChange of Object.values(this.uiState.couponPointChanges)) {
            const { coupon_id, points, program_id } = pointChange;
            const program = this.models["loyalty.program"].get(program_id);
            if (program.program_type !== "loyalty") {
                // Not a loyalty program, skip
                continue;
            }
            const loyaltyCard =
                this.models["loyalty.card"].get(coupon_id) ||
                this.models["loyalty.card"].create({
                    id: coupon_id,
                    points: 0,
                });
            let [won, spent, total] = [0, 0, 0];
            const balance = loyaltyCard.points;
            won += points - this._getPointsCorrection(program);
            if (coupon_id !== 0) {
                for (const line of this._get_reward_lines()) {
                    if (line.coupon_id.id === coupon_id) {
                        spent += line.points_cost;
                    }
                }
            }
            total = balance + won - spent;
            const name = program.portal_visible ? program.portal_point_name : _t("Points");
            loyaltyPoints[coupon_id] = {
                won: parseFloat(won.toFixed(2)),
                spent: parseFloat(spent.toFixed(2)),
                // Display total when order is ongoing.
                total: parseFloat(total.toFixed(2)),
                // Display balance when order is done.
                balance: parseFloat(balance.toFixed(2)),
                name,
                program,
            };
        }
        return Object.entries(loyaltyPoints).map(([couponId, points]) => ({
            couponId,
            points,
            program: points.program,
        }));
    },
    /**
     * The points in the couponPointChanges for free product reward is not correct.
     * It doesn't take into account the points from the `free` product. Use this method
     * to compute the necessary correction.
     * @param {*} program
     * @returns {number}
     */
    _getPointsCorrection(program) {
        const rewardLines = this.lines.filter((line) => line.is_reward_line);
        let res = 0;
        for (const rule of program.rule_ids) {
            for (const line of rewardLines) {
                const reward = line.reward_id;
                if (this._validForPointsCorrection(reward, line, rule)) {
                    if (rule.reward_point_mode === "money") {
                        res -= roundPrecision(
                            rule.reward_point_amount * line.get_price_with_tax(),
                            0.01
                        );
                    } else if (rule.reward_point_mode === "unit") {
                        res += rule.reward_point_amount * line.get_quantity();
                    }
                }
            }
        }
        return res;
    },
    /**
     * Checks if a reward line is valid for points correction.
     *
     * The function evaluates three conditions:
     * 1. The reward type must be 'product'.
     * 2. The reward line must be part of the rule.
     * 3. The reward line and the rule must be associated with the same program.
     */
    _validForPointsCorrection(reward, line, rule) {
        // Check if the reward type is free product
        if (reward.reward_type !== "product") {
            return false;
        }

        // Check if the rule's reward point mode is order then not valid for correction
        if (rule.reward_point_mode === "order") {
            return false;
        }

        // Check if the reward line is part of the rule
        if (!(rule.any_product || rule.validProductIds.has(line._reward_product_id?.id))) {
            return false;
        }

        // Check if the reward line and the rule are associated with the same program
        if (rule.program_id.id !== reward.program_id.id) {
            return false;
        }
        return true;
    },
    /**
     * @returns {number} The points that are left for the given coupon for this order.
     */
    //FIXME use of pos
    _getRealCouponPoints(coupon_id) {
        let points = 0;
        const dbCoupon = this.models["loyalty.card"].get(coupon_id);
        if (dbCoupon) {
            points += dbCoupon.points;
        }
        Object.values(this.uiState.couponPointChanges).some((pe) => {
            if (pe.coupon_id === coupon_id) {
                if (this.models["loyalty.program"].get(pe.program_id).applies_on !== "future") {
                    points += pe.points;
                }
                // couponPointChanges is not supposed to have a coupon multiple times
                return true;
            }
            return false;
        });
        for (const line of this.get_orderlines()) {
            if (line.is_reward_line && line.coupon_id.id === coupon_id) {
                points -= line.points_cost;
            }
        }
        return points;
    },
    _programIsApplicable(program) {
        if (
            program.trigger === "auto" &&
            !program.rule_ids.find(
                (rule) =>
                    rule.mode === "auto" || this.uiState.codeActivatedProgramRules.includes(rule.id)
            )
        ) {
            return false;
        }
        if (
            program.trigger === "with_code" &&
            !program.rule_ids.find((rule) =>
                this.uiState.codeActivatedProgramRules.includes(rule.id)
            )
        ) {
            return false;
        }
        if (program.is_nominative && !this.get_partner()) {
            return false;
        }
        if (program.date_from && program.date_from.startOf("day") > DateTime.now()) {
            return false;
        }
        if (program.date_to && program.date_to.endOf("day") < DateTime.now()) {
            return false;
        }
        if (program.limit_usage && program.total_order_count >= program.max_usage) {
            return false;
        }
        if (
            program.pricelist_ids.length > 0 &&
            (!this.pricelist_id ||
                !program.pricelist_ids.some((pl) => pl.id === this.pricelist_id.id))
        ) {
            return false;
        }
        return true;
    },
    /**
     * Computes how much points each program gives.
     *
     * @param {Array} programs list of loyalty.program
     * @returns {Object} Containing the points gained per program
     */
    pointsForPrograms(programs) {
        pointsForProgramsCountedRules = {};
        const orderLines = this.get_orderlines();
        const linesPerRule = {};
        for (const line of orderLines) {
            const reward = line.reward_id;
            const isDiscount = reward && reward.reward_type === "discount";
            const rewardProgram = reward && reward.program_id;
            // Skip lines for automatic discounts.
            if (isDiscount && rewardProgram.trigger === "auto") {
                continue;
            }
            for (const program of programs) {
                // Skip lines for the current program's discounts.
                if (isDiscount && rewardProgram.id === program.id) {
                    continue;
                }
                for (const rule of program.rule_ids) {
                    // Skip lines to which the rule doesn't apply.
                    if (rule.any_product || rule.validProductIds.has(line.product_id.id)) {
                        if (!linesPerRule[rule.id]) {
                            linesPerRule[rule.id] = [];
                        }
                        linesPerRule[rule.id].push(line);
                    }
                }
            }
        }
        const result = {};
        for (const program of programs) {
            let points = 0;
            const splitPoints = [];
            for (const rule of program.rule_ids) {
                if (
                    rule.mode === "with_code" &&
                    !this.uiState.codeActivatedProgramRules.includes(rule.id)
                ) {
                    continue;
                }
                const linesForRule = linesPerRule[rule.id] ? linesPerRule[rule.id] : [];
                const amountWithTax = linesForRule.reduce(
                    (sum, line) => sum + line.get_price_with_tax(),
                    0
                );
                const amountWithoutTax = linesForRule.reduce(
                    (sum, line) => sum + line.get_price_without_tax(),
                    0
                );
                const amountCheck =
                    (rule.minimum_amount_tax_mode === "incl" && amountWithTax) || amountWithoutTax;
                if (rule.minimum_amount > amountCheck) {
                    continue;
                }
                let totalProductQty = 0;
                // Only count points for paid lines.
                const qtyPerProduct = {};
                let orderedProductPaid = 0;
                for (const line of orderLines) {
                    if (
                        ((!line.reward_product_id &&
                            (rule.any_product || rule.validProductIds.has(line.product_id.id))) ||
                            (line.reward_product_id &&
                                (rule.any_product ||
                                    rule.validProductIds.has(line._reward_product_id?.id)))) &&
                        !line.ignoreLoyaltyPoints({ program })
                    ) {
                        // We only count reward products from the same program to avoid unwanted feedback loops
                        if (line.is_reward_line) {
                            const reward = line.reward_id;
                            if (
                                program.id === reward.program_id.id ||
                                ["gift_card", "ewallet"].includes(reward.program_id.program_type)
                            ) {
                                continue;
                            }
                        }
                        const lineQty = line._reward_product_id
                            ? -line.get_quantity()
                            : line.get_quantity();
                        if (qtyPerProduct[line._reward_product_id || line.get_product().id]) {
                            qtyPerProduct[line._reward_product_id || line.get_product().id] +=
                                lineQty;
                        } else {
                            qtyPerProduct[line._reward_product_id?.id || line.get_product().id] =
                                lineQty;
                        }
                        orderedProductPaid += line.get_price_with_tax();
                        if (!line.is_reward_line) {
                            totalProductQty += lineQty;
                        }
                    }
                }
                if (totalProductQty < rule.minimum_qty) {
                    // Should also count the points from negative quantities.
                    // For example, when refunding an ewallet payment. See TicketScreen override in this addon.
                    continue;
                }
                if (!(program.id in pointsForProgramsCountedRules)) {
                    pointsForProgramsCountedRules[program.id] = [];
                }
                pointsForProgramsCountedRules[program.id].push(rule.id);
                if (
                    program.applies_on === "future" &&
                    rule.reward_point_split &&
                    rule.reward_point_mode !== "order"
                ) {
                    // In this case we count the points per rule
                    if (rule.reward_point_mode === "unit") {
                        splitPoints.push(
                            ...Array.apply(null, Array(totalProductQty)).map((_) => ({
                                points: rule.reward_point_amount,
                            }))
                        );
                    } else if (rule.reward_point_mode === "money") {
                        for (const line of orderLines) {
                            if (
                                line.is_reward_line ||
                                !rule.validProductIds.has(line.product_id.id) ||
                                line.get_quantity() <= 0 ||
                                line.ignoreLoyaltyPoints({ program })
                            ) {
                                continue;
                            }
                            const pointsPerUnit = roundPrecision(
                                (rule.reward_point_amount * line.get_price_with_tax()) /
                                    line.get_quantity(),
                                0.01
                            );
                            if (pointsPerUnit > 0) {
                                splitPoints.push(
                                    ...Array.apply(null, Array(line.get_quantity())).map(() => {
                                        if (line._gift_barcode && line.get_quantity() == 1) {
                                            return {
                                                points: pointsPerUnit,
                                                barcode: line._gift_barcode,
                                                giftCardId: line._gift_card_id.id,
                                            };
                                        }
                                        return { points: pointsPerUnit };
                                    })
                                );
                            }
                        }
                    }
                } else {
                    // In this case we add on to the global point count
                    if (rule.reward_point_mode === "order") {
                        points += rule.reward_point_amount;
                    } else if (rule.reward_point_mode === "money") {
                        // NOTE: unlike in sale_loyalty this performs a round half-up instead of round down
                        points += roundPrecision(
                            rule.reward_point_amount * orderedProductPaid,
                            0.01
                        );
                    } else if (rule.reward_point_mode === "unit") {
                        points += rule.reward_point_amount * totalProductQty;
                    }
                }
            }
            const res = points || program.program_type === "coupons" ? [{ points }] : [];
            if (splitPoints.length) {
                res.push(...splitPoints);
            }
            result[program.id] = res;
        }
        return result;
    },
    /**
     * @returns {Array} List of lines composing the global discount
     */
    _getGlobalDiscountLines() {
        return this.get_orderlines().filter(
            (line) => line.reward_id && line.reward_id.is_global_discount
        );
    },
    /**
     * Returns the number of product items in the order based on the given rule.
     * @param {*} rule
     */
    _computeNItems(rule) {
        return this._get_regular_order_lines().reduce((nItems, line) => {
            let increment = 0;
            if (rule.any_product || rule.validProductIds.has(line.product_id.id)) {
                increment = line.get_quantity();
            }
            return nItems + increment;
        }, 0);
    },
    /**
     * Checks whether this order is allowed to generate rewards
     * from the given coupon program.
     * @param {*} couponProgram
     */
    _canGenerateRewards(couponProgram, orderTotalWithTax, orderTotalWithoutTax) {
        for (const rule of couponProgram.rule_ids) {
            const amountToCompare =
                rule.minimum_amount_tax_mode == "incl" ? orderTotalWithTax : orderTotalWithoutTax;
            if (rule.minimum_amount > amountToCompare) {
                return false;
            }
            const nItems = this._computeNItems(rule);
            if (rule.minimum_qty > nItems) {
                return false;
            }
        }
        return true;
    },
    /**
     * @param {Integer} coupon_id (optional) Coupon id
     * @param {Integer} program_id (optional) Program id
     * @returns {Array} List of {Object} containing the coupon_id and reward keys
     */
    getClaimableRewards(coupon_id = false, program_id = false, auto = false) {
        const couponPointChanges = this.uiState.couponPointChanges;
        const excludedCouponIds = Object.keys(couponPointChanges)
            .filter((id) => couponPointChanges[id].manual && couponPointChanges[id].existing_code)
            .map((id) => couponPointChanges[id].coupon_id);

        const allCouponPrograms = Object.values(this.uiState.couponPointChanges)
            .filter((pe) => !excludedCouponIds.includes(pe.coupon_id))
            .map((pe) => ({
                program_id: pe.program_id,
                coupon_id: pe.coupon_id,
            }))
            .concat(
                this._code_activated_coupon_ids.map((coupon) => ({
                    program_id: coupon.program_id.id,
                    coupon_id: coupon.id,
                }))
            );
        const result = [];
        const totalWithTax = this.get_total_with_tax();
        const totalWithoutTax = this.get_total_without_tax();
        const totalIsZero = totalWithTax === 0;
        const globalDiscountLines = this._getGlobalDiscountLines();
        const globalDiscountPercent = globalDiscountLines.length
            ? globalDiscountLines[0].reward_id.discount
            : 0;
        for (const couponProgram of allCouponPrograms) {
            const program = this.models["loyalty.program"].get(couponProgram.program_id);
            if (
                program.pricelist_ids.length > 0 &&
                (!this.pricelist_id ||
                    !program.pricelist_ids.some((pl) => pl.id === this.pricelist_id.id))
            ) {
                continue;
            }
            if (program.trigger == "with_code") {
                // For coupon programs, the rules become conditions.
                // Points to purchase rewards will only come from the scanned coupon.
                if (!this._canGenerateRewards(program, totalWithTax, totalWithoutTax)) {
                    continue;
                }
            }
            if (
                (coupon_id && couponProgram.coupon_id !== coupon_id) ||
                (program_id && couponProgram.program_id !== program_id)
            ) {
                continue;
            }
            const points = this._getRealCouponPoints(couponProgram.coupon_id);
            for (const reward of program.reward_ids) {
                if (points < reward.required_points) {
                    continue;
                }
                // Skip if the reward program is of type 'coupons' and there is already an reward orderline linked to the current reward to avoid multiple reward apply
                if (
                    reward.program_id.program_type === "coupons" &&
                    this.lines.find((rewardline) => rewardline.reward_id?.id === reward.id)
                ) {
                    continue;
                }
                if (auto && this.uiState.disabledRewards.has(reward.id)) {
                    continue;
                }
                // Try to filter out rewards that will not be claimable anyway.
                if (reward.is_global_discount && reward.discount <= globalDiscountPercent) {
                    continue;
                }
                if (reward.reward_type === "discount" && totalIsZero) {
                    continue;
                }
                let unclaimedQty;
                if (reward.reward_type === "product") {
                    if (!reward.multi_product) {
                        const product = reward.reward_product_id;
                        if (!product) {
                            continue;
                        }
                        unclaimedQty = this._computeUnclaimedFreeProductQty(
                            reward,
                            couponProgram.coupon_id,
                            product,
                            points
                        );
                    }
                    if (!unclaimedQty || unclaimedQty <= 0) {
                        continue;
                    }
                }
                result.push({
                    coupon_id: couponProgram.coupon_id,
                    reward: reward,
                    potentialQty: unclaimedQty,
                });
            }
        }
        return result;
    },
    /**
     * TODO JCB: make the second parameter not id, but the loyalty.card object itself.
     * Applies a reward to the order, `pos.updateRewards` is expected to be called right after.
     *
     * @param {loyalty.reward} reward
     * @param {Integer} coupon_id
     * @param {Object} args Reward options
     * @returns True if everything went right or an error message
     */
    _applyReward(reward, coupon_id, args) {
        if (this._getRealCouponPoints(coupon_id) < reward.required_points) {
            return _t("There are not enough points on the coupon to claim this reward.");
        }
        if (reward.is_global_discount) {
            const globalDiscountLines = this._getGlobalDiscountLines();
            if (globalDiscountLines.length) {
                const rewardId = globalDiscountLines[0].reward_id;
                if (rewardId != reward.id && rewardId.discount >= reward.discount) {
                    return _t("A better global discount is already applied.");
                } else if (rewardId != rewardId.id) {
                    for (const line of globalDiscountLines) {
                        line.delete();
                    }
                }
            }
        }
        args = args || {};
        const rewardLines = this._getRewardLineValues({
            reward: reward,
            coupon_id: coupon_id,
            product: args["product"] || null,
            price: args["price"] || null,
            quantity: args["quantity"] || null,
            cost: args["cost"] || null,
        });
        if (!Array.isArray(rewardLines)) {
            return rewardLines; // Returned an error.
        }
        if (!rewardLines.length) {
            return _t("The reward could not be applied.");
        }
        for (const rewardLine of rewardLines) {
            const prepareRewards = {
                ...rewardLine,
                reward_id: rewardLine.reward_id,
                coupon_id: this.models["loyalty.card"].get(rewardLine.coupon_id),
                tax_ids: rewardLine.tax_ids.map((tax) => ["link", tax]),
            };
            this.models["pos.order.line"].create({
                ...prepareRewards,
                order_id: this,
                price_type: "manual",
            });
        }
        return true;
    },
    /**
     * Checks if there are any existing manual changes or new coupon additions for the given coupon code
     */
    duplicateCouponChanges(code) {
        return Object.keys(this.uiState.couponPointChanges).some((key) => {
            const change = this.uiState.couponPointChanges[key];
            return (
                (change.existing_code === code && change.manual) ||
                (change.code === code && change.coupon_id < 0)
            );
        });
    },
    /**
     * Processes a gift card by creating a new gift card.
     *
     * @param {String} newGiftCardCode gift card code as a string if new gift card to be created.
     * @param {number} points number of points to assign to the gift card.
     */
    processGiftCard(newGiftCardCode, points, expirationDate) {
        const partner_id = this.partner_id?.id || false;
        const product_id = this.get_selected_orderline().product_id.id;
        const program = this.models["loyalty.program"].find((p) => p.program_type === "gift_card");

        let couponId;
        const couponData = {
            program_id: program?.id,
            points: points,
            manual: true,
            product_id: product_id,
        };

        // Fetch all coupon_ids for the specified points and not manually created, that are associated with the gift card program
        const applicableCouponIds = Object.keys(this.uiState.couponPointChanges).filter((key) => {
            const change = this.uiState.couponPointChanges[key];
            return (
                change.points === points &&
                change.program_id === program.id &&
                change.product_id === product_id &&
                !change.manual
            );
        });

        if (newGiftCardCode) {
            couponId = applicableCouponIds.shift() || loyaltyIdsGenerator();
            couponData.coupon_id = couponId;
            couponData.code = newGiftCardCode;
            couponData.partner_id = partner_id;
            couponData.expiration_date = expirationDate;
        }

        this.uiState.couponPointChanges[couponId] = couponData;
    },
    /**
     * @param {loyalty.reward} reward
     * @returns the discountable and discountable per tax for this discount on order reward.
     */
    _getDiscountableOnOrder(reward) {
        let discountable = 0;
        const discountablePerTax = {};
        for (const line of this.get_orderlines()) {
            if (!line.get_quantity()) {
                continue;
            }

            const taxKey = ["ewallet", "gift_card"].includes(reward.program_id.program_type)
                ? line.tax_ids.map((t) => t.id)
                : line.tax_ids.filter((t) => t.amount_type !== "fixed").map((t) => t.id);
            discountable += line.get_price_with_tax();
            if (!discountablePerTax[taxKey]) {
                discountablePerTax[taxKey] = 0;
            }
            discountablePerTax[taxKey] += line.get_base_price();
        }
        return { discountable, discountablePerTax };
    },
    /**
     * @returns the order's cheapest line
     */
    _getCheapestLine() {
        const filtered_lines = this.get_orderlines().filter(
            (line) => !line.comboParent && !line.reward_id && line.get_quantity
        );
        return filtered_lines.toSorted(
            (lineA, lineB) => lineA.getComboTotalPrice() - lineB.getComboTotalPrice()
        )[0];
    },
    /**
     * @returns the discountable and discountable per tax for this discount on cheapest reward.
     */
    _getDiscountableOnCheapest(reward) {
        const cheapestLine = this._getCheapestLine();
        if (!cheapestLine) {
            return { discountable: 0, discountablePerTax: {} };
        }
        const taxKey = cheapestLine.tax_ids.map((t) => t.id);
        return {
            discountable: cheapestLine.getComboTotalPriceWithoutTax(),
            discountablePerTax: Object.fromEntries([
                [taxKey, cheapestLine.getComboTotalPriceWithoutTax()],
            ]),
        };
    },
    /**
     * @param {loyalty.reward} reward
     * @returns all lines to which the reward applies.
     */
    _getSpecificDiscountableLines(reward) {
        const discountableLines = [];
        const applicableProductIds = new Set(reward.all_discount_product_ids.map((p) => p.id));
        for (const line of this.get_orderlines()) {
            if (!line.get_quantity()) {
                continue;
            }
            if (
                applicableProductIds.has(line.get_product().id) ||
                applicableProductIds.has(line._reward_product_id?.id)
            ) {
                discountableLines.push(line);
            }
        }
        return discountableLines;
    },
    /**
     * For a 'specific' type of discount it is more complicated as we have to make sure that we never
     *  discount more than what is available on a per line basis.
     * @param {loyalty.reward} reward
     * @returns the discountable and discountable per tax for this discount on specific reward.
     */
    _getDiscountableOnSpecific(reward) {
        const applicableProductIds = new Set(reward.all_discount_product_ids.map((p) => p.id));
        const linesToDiscount = [];
        const discountLinesPerReward = {};
        const orderLines = this.get_orderlines();
        const orderProducts = orderLines.map((line) => line.product_id.id);
        const remainingAmountPerLine = {};
        for (const line of orderLines) {
            if (!line.get_quantity() || !line.price_unit) {
                continue;
            }
            remainingAmountPerLine[line.uuid] = line.get_price_with_tax();
            const product_id = line.combo_parent_id?.product_id.id || line.get_product().id;
            if (
                applicableProductIds.has(product_id) ||
                (line._reward_product_id && applicableProductIds.has(line._reward_product_id.id))
            ) {
                linesToDiscount.push(line);
            } else if (line.reward_id) {
                const lineReward = line.reward_id;
                const lineRewardApplicableProductsIds = new Set(
                    lineReward.all_discount_product_ids.map((p) => p.id)
                );
                if (
                    lineReward.id === reward.id ||
                    (orderProducts.some(
                        (product) =>
                            lineRewardApplicableProductsIds.has(product) &&
                            applicableProductIds.has(product)
                    ) &&
                        lineReward.reward_type === "discount" &&
                        lineReward.discount_mode != "percent")
                ) {
                    linesToDiscount.push(line);
                }
                if (!discountLinesPerReward[line.reward_identifier_code]) {
                    discountLinesPerReward[line.reward_identifier_code] = [];
                }
                discountLinesPerReward[line.reward_identifier_code].push(line);
            }
        }

        let cheapestLine = false;
        for (const lines of Object.values(discountLinesPerReward)) {
            const lineReward = lines[0].reward_id;
            if (lineReward.reward_type !== "discount") {
                continue;
            }
            let discountedLines = orderLines;
            if (lineReward.discount_applicability === "cheapest") {
                cheapestLine = cheapestLine || this._getCheapestLine();
                discountedLines = [cheapestLine];
            } else if (lineReward.discount_applicability === "specific") {
                discountedLines = this._getSpecificDiscountableLines(lineReward);
            }
            if (!discountedLines.length) {
                continue;
            }
            if (lineReward.discount_mode === "percent") {
                const discount = lineReward.discount / 100;
                for (const line of discountedLines) {
                    if (line.reward_id) {
                        continue;
                    }
                    if (lineReward.discount_applicability === "cheapest") {
                        remainingAmountPerLine[line.uuid] *= 1 - discount / line.get_quantity();
                    } else {
                        remainingAmountPerLine[line.uuid] *= 1 - discount;
                    }
                }
            }
        }

        let discountable = 0;
        const discountablePerTax = {};
        for (const line of linesToDiscount) {
            discountable += remainingAmountPerLine[line.uuid];
            const taxKey = line.tax_ids.map((t) => t.id);
            if (!discountablePerTax[taxKey]) {
                discountablePerTax[taxKey] = 0;
            }
            discountablePerTax[taxKey] +=
                line.get_base_price() *
                (remainingAmountPerLine[line.uuid] / line.get_price_with_tax());
        }
        return { discountable, discountablePerTax };
    },
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the reward lines
     */
    _getRewardLineValues(args) {
        const reward = args["reward"];
        if (reward.reward_type === "discount") {
            return this._getRewardLineValuesDiscount(args);
        } else if (reward.reward_type === "product") {
            return this._getRewardLineValuesProduct(args);
        }
        // NOTE: we may reach this step if for some reason there is a free shipping reward
        return [];
    },
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the discount lines
     */
    _getRewardLineValuesDiscount(args) {
        //LINK
        const reward = args["reward"];
        const coupon_id = args["coupon_id"];
        const rewardAppliesTo = reward.discount_applicability;
        let getDiscountable;
        if (rewardAppliesTo === "order") {
            getDiscountable = this._getDiscountableOnOrder.bind(this);
        } else if (rewardAppliesTo === "cheapest") {
            getDiscountable = this._getDiscountableOnCheapest.bind(this);
        } else if (rewardAppliesTo === "specific") {
            getDiscountable = this._getDiscountableOnSpecific.bind(this);
        }
        if (!getDiscountable) {
            return _t("Unknown discount type");
        }
        let { discountable, discountablePerTax } = getDiscountable(reward);
        discountable = Math.min(this.get_total_with_tax(), discountable);
        if (!discountable) {
            return [];
        }
        let maxDiscount = reward.discount_max_amount || Infinity;
        if (reward.discount_mode === "per_point") {
            // Rewards cannot be partially offered to customers
            const points = ["ewallet", "gift_card"].includes(reward.program_id.program_type)
                ? this._getRealCouponPoints(coupon_id)
                : Math.floor(this._getRealCouponPoints(coupon_id) / reward.required_points) *
                  reward.required_points;
            maxDiscount = Math.min(maxDiscount, reward.discount * points);
        } else if (reward.discount_mode === "per_order") {
            maxDiscount = Math.min(maxDiscount, reward.discount);
        } else if (reward.discount_mode === "percent") {
            maxDiscount = Math.min(maxDiscount, discountable * (reward.discount / 100));
        }
        const rewardCode = _newRandomRewardCode();
        let pointCost = reward.clear_wallet
            ? this._getRealCouponPoints(coupon_id)
            : reward.required_points;
        if (reward.discount_mode === "per_point" && !reward.clear_wallet) {
            pointCost = Math.min(maxDiscount, discountable) / reward.discount;
        }
        // These are considered payments and do not require to be either taxed or split by tax
        const discountProduct = reward.discount_line_product_id;
        if (["ewallet", "gift_card"].includes(reward.program_id.program_type)) {
            const new_price = compute_price_force_price_include(
                discountProduct.taxes_id,
                -Math.min(maxDiscount, discountable),
                discountProduct,
                this.config._product_default_values,
                this.company,
                this.currency,
                this.models
            );

            return [
                {
                    product_id: discountProduct,
                    price_unit: new_price,
                    qty: 1,
                    reward_id: reward,
                    is_reward_line: true,
                    coupon_id: coupon_id,
                    points_cost: pointCost,
                    reward_identifier_code: rewardCode,
                    tax_ids: discountProduct.taxes_id,
                },
            ];
        }
        const discountFactor = discountable ? Math.min(1, maxDiscount / discountable) : 1;
        const result = Object.entries(discountablePerTax).reduce((lst, entry) => {
            // Ignore 0 price lines
            if (!entry[1]) {
                return lst;
            }
            let taxIds = entry[0] === "" ? [] : entry[0].split(",").map((str) => parseInt(str));
            taxIds = this.models["account.tax"].filter((tax) => taxIds.includes(tax.id));

            lst.push({
                product_id: discountProduct,
                price_unit: -(Math.min(this.get_total_with_tax(), entry[1]) * discountFactor),
                qty: 1,
                reward_id: reward,
                is_reward_line: true,
                coupon_id: coupon_id,
                points_cost: 0,
                reward_identifier_code: rewardCode,
                tax_ids: taxIds,
            });
            return lst;
        }, []);
        if (result.length) {
            result[0]["points_cost"] = pointCost;
        }
        return result;
    },
    _isRewardProductPartOfRules(reward, product) {
        return (
            reward.program_id.rule_ids.filter(
                (rule) => rule.any_product || rule.validProductIds.has(product.id)
            ).length > 0
        );
    },
    /**
     * Tries to compute how many free product can be given out for the given product.
     * Contrary to sale_loyalty, the product must be in the order lines in order to give it out
     *  (resulting in discount lines for the product's value).
     * As such we need to approximate the effect of removing 1 quantity on the counting of points in order
     *  to avoid feedback loops between giving a product and it removing the required points for it.
     *
     * @param {loyalty.reward} reward
     * @param {Integer} coupon_id
     * @param {Product} product
     * @returns {Integer} Available quantity to be given as reward for the given product
     */
    _computeUnclaimedFreeProductQty(reward, coupon_id, product, remainingPoints) {
        let claimed = 0;
        let available = 0;
        let shouldCorrectRemainingPoints = false;
        for (const line of this.get_orderlines()) {
            if (
                reward.reward_product_ids.map((reward) => reward.id).includes(product.id) &&
                reward.reward_product_ids.map((reward) => reward.id).includes(line.get_product().id)
            ) {
                if (this._get_reward_lines() == 0) {
                    if (line.get_product() === product) {
                        available += line.get_quantity();
                    }
                } else {
                    available += line.get_quantity();
                }
            } else if (
                reward.reward_product_ids
                    .map((reward) => reward.id)
                    .includes(line._reward_product_id?.id)
            ) {
                if (line.reward_id.id == reward.id) {
                    remainingPoints += line.points_cost;
                    claimed += line.get_quantity();
                } else {
                    shouldCorrectRemainingPoints = true;
                }
            }
        }
        let freeQty;
        if (reward.program_id.trigger == "auto") {
            if (
                this._isRewardProductPartOfRules(reward, product) &&
                reward.program_id.applies_on !== "future"
            ) {
                // OPTIMIZATION: Pre-calculate the factors for each reward-product combination during the loading.
                // For points not based on quantity, need to normalize the points to compute free quantity.
                const appliedRulesIds = this.uiState.couponPointChanges[coupon_id].appliedRules;
                const appliedRules =
                    appliedRulesIds !== undefined
                        ? reward.program_id.rule_ids.filter((rule) =>
                              appliedRulesIds.includes(rule.id)
                          )
                        : reward.program_id.rule_ids;
                let factor = 0;
                let orderPoints = 0;
                for (const rule of appliedRules) {
                    if (rule.any_product || rule.validProductIds.has(product.id)) {
                        if (rule.reward_point_mode === "order") {
                            orderPoints += rule.reward_point_amount;
                        } else if (rule.reward_point_mode === "money") {
                            factor += roundPrecision(
                                rule.reward_point_amount * product.lst_price,
                                0.01
                            );
                        } else if (rule.reward_point_mode === "unit") {
                            factor += rule.reward_point_amount;
                        }
                    }
                }
                if (factor === 0) {
                    freeQty = Math.floor(
                        (remainingPoints / reward.required_points) * reward.reward_product_qty
                    );
                } else {
                    const correction = shouldCorrectRemainingPoints
                        ? this._getPointsCorrection(reward.program_id)
                        : 0;
                    freeQty = computeFreeQuantity(
                        (remainingPoints - correction - orderPoints) / factor,
                        reward.required_points / factor,
                        reward.reward_product_qty
                    );
                    freeQty += Math.floor(
                        (orderPoints / reward.required_points) * reward.reward_product_qty
                    );
                }
            } else {
                freeQty = Math.floor(
                    (remainingPoints / reward.required_points) * reward.reward_product_qty
                );
            }
        } else if (reward.program_id.trigger == "with_code") {
            freeQty = Math.floor(
                (remainingPoints / reward.required_points) * reward.reward_product_qty
            );
        }
        return Math.min(available, freeQty) - claimed;
    },
    _computePotentialFreeProductQty(reward, product, remainingPoints) {
        if (reward.program_id.trigger == "auto") {
            if (
                this._isRewardProductPartOfRules(reward, product) &&
                reward.program_id.applies_on !== "future"
            ) {
                const line = this.get_orderlines().find(
                    (line) => line._reward_product_id?.id === product.id
                );
                // Compute the correction points once even if there are multiple reward lines.
                // This is because _getPointsCorrection is taking into account all the lines already.
                const claimedPoints = line ? this._getPointsCorrection(reward.program_id) : 0;
                return Math.floor((remainingPoints - claimedPoints) / reward.required_points) > 0
                    ? reward.reward_product_qty
                    : 0;
            } else {
                return Math.floor(
                    (remainingPoints / reward.required_points) * reward.reward_product_qty
                );
            }
        } else if (reward.program_id.trigger == "with_code") {
            return Math.floor(
                (remainingPoints / reward.required_points) * reward.reward_product_qty
            );
        }
    },
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the reward lines
     */
    _getRewardLineValuesProduct(args) {
        const reward = args["reward"];
        const product =
            reward.reward_product_ids.find((p) => p.id === args["product"]?.id) ||
            reward.reward_product_ids[0];

        const points = this._getRealCouponPoints(args["coupon_id"]);
        const unclaimedQty = this._computeUnclaimedFreeProductQty(
            reward,
            args["coupon_id"],
            product,
            points
        );
        if (unclaimedQty <= 0) {
            return _t("There are not enough products in the basket to claim this reward.");
        }
        const claimable_count = reward.clear_wallet
            ? 1
            : Math.min(
                  Math.ceil(unclaimedQty / reward.reward_product_qty),
                  Math.floor(points / reward.required_points)
              );
        const cost = reward.clear_wallet
            ? points
            : Math.min(claimable_count * reward.required_points, args["cost"] || Infinity);
        // In case the reward is the product multiple times, give it as many times as possible
        const freeQuantity = Math.min(
            unclaimedQty,
            reward.reward_product_qty * claimable_count,
            args["quantity"] || Infinity
        );
        return [
            {
                product_id: reward.discount_line_product_id,
                price_unit: -roundDecimals(
                    product.get_price(this.pricelist_id, freeQuantity),
                    this.currency.decimal_places
                ),
                tax_ids: product.taxes_id,
                qty: freeQuantity,
                reward_id: reward,
                is_reward_line: true,
                _reward_product_id: product,
                coupon_id: args["coupon_id"],
                points_cost: cost,
                reward_identifier_code: _newRandomRewardCode(),
            },
        ];
    },
    isProgramsResettable() {
        const array = [
            this.uiState.disabledRewards,
            this.uiState.codeActivatedProgramRules,
            Object.keys(this.uiState.couponPointChanges),
            this._get_reward_lines(),
        ];
        return array.some((elem) => elem.length > 0);
    },
    removeOrderline(lineToRemove) {
        if (lineToRemove.is_reward_line) {
            // Remove any line that is part of that same reward aswell.
            const linesToRemove = this.get_orderlines().filter(
                (line) =>
                    line.reward_id === lineToRemove.reward_id &&
                    line.coupon_id === lineToRemove.coupon_id &&
                    line.reward_identifier_code === lineToRemove.reward_identifier_code
            );
            for (const line of linesToRemove) {
                line.delete();
            }
            return true;
        } else {
            return super.removeOrderline(lineToRemove);
        }
    },
    getSortedOrderlines() {
        const lines = super.getSortedOrderlines();
        if (this.config.orderlines_sequence_in_cart_by_category && this.lines.length) {
            const rewardLines = [];
            const resultLines = [];

            lines.forEach((line) => {
                if (line.is_reward_line) {
                    rewardLines.push(line);
                } else {
                    resultLines.push(line);
                }
            });

            rewardLines.forEach((line) => {
                if (line.reward_id.reward_type === "discount") {
                    resultLines.splice(resultLines.length, 0, line);
                } else if (line.reward_id.reward_type === "product") {
                    const rewardProductIndex = resultLines.findIndex(
                        (rewardLine) =>
                            line.reward_id?.reward_product_id?.id === rewardLine.product_id.id
                    );
                    resultLines.splice(rewardProductIndex + 1, 0, line);
                }
            });
            return resultLines;
        }
        return lines;
    },
});

```

## File: static\src\overrides\models\pos_order_line.js

```javascript
import { PosOrderline } from "@point_of_sale/app/models/pos_order_line";
import { formatCurrency } from "@point_of_sale/app/models/utils/currency";
import { patch } from "@web/core/utils/patch";

patch(PosOrderline, {
    extraFields: {
        ...(PosOrderline.extraFields || {}),
        _e_wallet_program_id: {
            model: "pos.order.line",
            name: "_e_wallet_program_id",
            relation: "loyalty.program",
            type: "many2one",
            local: true,
        },
        gift_code: {
            model: "pos.order.line",
            name: "gift_code",
            type: "char",
            local: true,
        },
        _gift_barcode: {
            model: "pos.order.line",
            name: "_gift_barcode",
            type: "char",
            local: true,
        },
        _gift_card_id: {
            model: "pos.order.line",
            name: "_gift_card_id",
            relation: "loyalty.card",
            type: "many2one",
            local: true,
        },
        _reward_product_id: {
            model: "pos.order.line",
            name: "_reward_product_id",
            relation: "product.product",
            type: "many2one",
            local: true,
        },
    },
});

patch(PosOrderline.prototype, {
    serialize(options = {}) {
        const json = super.serialize(...arguments);
        if (options.orm && json.coupon_id < 0) {
            json.coupon_id = undefined;
        }
        return json;
    },
    setOptions(options) {
        if (options.eWalletGiftCardProgram) {
            this.update({ _e_wallet_program_id: options.eWalletGiftCardProgram });
        }
        if (options.giftBarcode) {
            this.update({ _gift_barcode: options.giftBarcode });
        }
        if (options.giftCardId) {
            this.update({ _gift_card_id: this.models["loyalty.card"].get(options.giftCardId) });
        }
        return super.setOptions(...arguments);
    },
    getEWalletGiftCardProgramType() {
        return this._e_wallet_program_id && this._e_wallet_program_id.program_type;
    },
    ignoreLoyaltyPoints({ program }) {
        return (
            ["gift_card", "ewallet"].includes(program.program_type) &&
            this._e_wallet_program_id?.id !== program.id
        );
    },
    isGiftCardOrEWalletReward() {
        const coupon = this.coupon_id;
        if (!coupon || !this.is_reward_line) {
            return false;
        }
        return ["ewallet", "gift_card"].includes(coupon.program_id?.program_type);
    },
    getGiftCardOrEWalletBalance() {
        const coupon = this.coupon_id;
        return formatCurrency(coupon?.points || 0, this.currency);
    },
    getDisplayClasses() {
        return {
            ...super.getDisplayClasses(),
            "fst-italic": this.is_reward_line,
        };
    },
    getDisplayData() {
        if (!this.order_id) {
            return;
        }
        return super.getDisplayData();
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { _t } from "@web/core/l10n/translation";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { Domain, InvalidDomainError } from "@web/core/domain";
import { ask, makeAwaitable } from "@point_of_sale/app/store/make_awaitable_dialog";
import { Mutex } from "@web/core/utils/concurrency";
import { effect } from "@web/core/utils/reactive";
import { batched } from "@web/core/utils/timing";
import { serializeDate } from "@web/core/l10n/dates";

let nextId = -1;
const mutex = new Mutex();
const updateRewardsMutex = new Mutex();
const pointsForProgramsCountedRules = {};
const { DateTime } = luxon;

export function loyaltyIdsGenerator() {
    return nextId--;
}

function inverted(fn) {
    return (arg) => !fn(arg);
}

patch(PosStore.prototype, {
    async setup() {
        this.couponByLineUuidCache = {};
        this.rewardProductByLineUuidCache = {};
        await super.setup(...arguments);

        effect(
            batched((orders) => {
                const order = Array.from(orders.values()).find(
                    (order) => order.uuid === this.selectedOrderUuid
                );

                if (order) {
                    this.updateOrder(order);
                }
            }),
            [this.data.records["pos.order"]]
        );
    },
    async updateOrder(order) {
        // Read value to trigger effect
        order?.lines?.length;
        await this.orderUpdateLoyaltyPrograms();
    },
    async selectPartner(partner) {
        const res = await super.selectPartner(partner);
        await this.updateRewards();
        return res;
    },
    async selectPricelist(pricelist) {
        await super.selectPricelist(pricelist);
        await this.updateRewards();
    },
    async resetPrograms() {
        const order = this.get_order();
        order._resetPrograms();
        await this.orderUpdateLoyaltyPrograms();
        await this.updateRewards();
    },
    async orderUpdateLoyaltyPrograms() {
        if (!this.get_order()) {
            return;
        }

        await this.checkMissingCoupons();
        await this.updatePrograms();
    },
    updateRewards() {
        // Calls are not expected to take some time besides on the first load + when loyalty programs are made applicable
        if (this.models["loyalty.program"].length === 0) {
            return;
        }

        const order = this.get_order();
        if (!order || order.finalized) {
            return;
        }
        updateRewardsMutex.exec(() => {
            return this.orderUpdateLoyaltyPrograms().then(async () => {
                // Try auto claiming rewards
                const claimableRewards = order.getClaimableRewards(false, false, true);
                let changed = false;
                for (const { coupon_id, reward } of claimableRewards) {
                    if (
                        reward.program_id.reward_ids.length === 1 &&
                        !reward.program_id.is_nominative &&
                        (reward.reward_type !== "product" ||
                            (reward.reward_type == "product" && !reward.multi_product))
                    ) {
                        if (
                            (reward.reward_type == "product" &&
                                reward.program_id.applies_on !== "both") ||
                            (reward.program_id.applies_on == "both" && reward.reward_product_qty)
                        ) {
                            this.addLineToCurrentOrder({
                                product_id: reward.reward_product_id,
                                qty: reward.reward_product_qty || 1,
                            });
                        }
                        order._applyReward(reward, coupon_id);
                        changed = true;
                    }
                }
                // Rewards may impact the number of points gained
                if (changed) {
                    await this.orderUpdateLoyaltyPrograms();
                }
                order._updateRewardLines();
            });
        });
    },
    async couponForProgram(program) {
        const order = this.get_order();
        if (program.is_nominative) {
            return await this.fetchLoyaltyCard(program.id, order.get_partner().id);
        }
        // This type of coupons don't need to really exist up until validating the order, so no need to cache
        return this.models["loyalty.card"].create({
            id: loyaltyIdsGenerator(),
            code: null,
            program_id: program,
            partner_id: order.partner_id,
            points: 0,
        });
    },
    /**
     * Update our couponPointChanges, meaning the points/coupons each program give etc.
     */
    async updatePrograms() {
        const order = this.get_order();
        // 'order.delivery_provider_id' check is used for UrbanPiper orders (as loyalty points and rewards are not allowed for UrbanPiper orders)
        if (!order || order.delivery_provider_id) {
            return;
        }
        const changesPerProgram = {};
        const programsToCheck = new Set();
        // By default include all programs that are considered 'applicable'
        for (const program of this.models["loyalty.program"].getAll()) {
            if (order._programIsApplicable(program)) {
                programsToCheck.add(program.id);
            }
        }
        for (const pe of Object.values(order.uiState.couponPointChanges)) {
            if (!changesPerProgram[pe.program_id]) {
                changesPerProgram[pe.program_id] = [];
                programsToCheck.add(pe.program_id);
            }
            changesPerProgram[pe.program_id].push(pe);
        }
        for (const coupon of order._code_activated_coupon_ids) {
            programsToCheck.add(coupon.program_id.id);
        }
        const programs = [...programsToCheck].map((programId) =>
            this.models["loyalty.program"].get(programId)
        );
        const pointsAddedPerProgram = order.pointsForPrograms(programs);
        for (const program of this.models["loyalty.program"].getAll()) {
            // Future programs may split their points per unit paid (gift cards for example), consider a non applicable program to give no points
            const pointsAdded = order._programIsApplicable(program)
                ? pointsAddedPerProgram[program.id]
                : [];
            // For programs that apply to both (loyalty) we always add a change of 0 points, if there is none, since it makes it easier to
            //  track for claimable rewards, and makes sure to load the partner's loyalty card.
            if (program.is_nominative && !pointsAdded.length && order.get_partner()) {
                pointsAdded.push({ points: 0 });
            }
            const oldChanges = changesPerProgram[program.id] || [];
            // Update point changes for those that exist
            for (
                let idx = 0;
                idx < Math.min(pointsAdded.length, oldChanges.length) && !oldChanges[idx].manual;
                idx++
            ) {
                Object.assign(oldChanges[idx], pointsAdded[idx]);
            }
            if (pointsAdded.length < oldChanges.length) {
                const removedIds = oldChanges.map((pe) => pe.coupon_id);
                order.uiState.couponPointChanges = Object.fromEntries(
                    Object.entries(order.uiState.couponPointChanges).filter(([k, pe]) => {
                        return !removedIds.includes(pe.coupon_id);
                    })
                );
            } else if (pointsAdded.length > oldChanges.length) {
                const pointsCount = pointsAdded.reduce((acc, pointObj) => {
                    const { points, barcode = "" } = pointObj;
                    const key = barcode ? `${points}-${barcode}` : `${points}`;
                    acc[key] = (acc[key] || 0) + 1;
                    return acc;
                }, {});

                oldChanges.forEach((pointObj) => {
                    const { points, barcode = "" } = pointObj;
                    const key = barcode ? `${points}-${barcode}` : `${points}`;
                    if (pointsCount[key] && pointsCount[key] > 0) {
                        pointsCount[key]--;
                    }
                });

                // Get new points added which are not in oldChanges
                const newPointsAdded = [];
                Object.keys(pointsCount).forEach((key) => {
                    const [points, barcode = ""] = key.split("-");
                    while (pointsCount[key] > 0) {
                        newPointsAdded.push({ points: Number(points), barcode });
                        pointsCount[key]--;
                    }
                });

                for (const pa of newPointsAdded) {
                    const coupon = await this.couponForProgram(program);
                    const couponPointChange = {
                        points: pa.points,
                        program_id: program.id,
                        coupon_id: coupon.id,
                        barcode: pa.barcode,
                        appliedRules: pointsForProgramsCountedRules[program.id],
                    };
                    if (program && program.program_type === "gift_card") {
                        couponPointChange.product_id =
                            order.get_selected_orderline()?.product_id.id;
                        couponPointChange.expiration_date = serializeDate(
                            luxon.DateTime.now().plus({ year: 1 })
                        );
                        couponPointChange.code = order.get_selected_orderline()?.gift_code;
                        couponPointChange.partner_id = order.get_partner()?.id;
                    }

                    order.uiState.couponPointChanges[coupon.id] = couponPointChange;
                }
            }
        }

        // Also remove coupons from _code_activated_coupon_ids if their program applies_on current orders and the program does not give any points
        const toUnlink = order._code_activated_coupon_ids.filter(
            inverted((coupon) => {
                const program = coupon.program_id;
                if (
                    program.applies_on === "current" &&
                    pointsAddedPerProgram[program.id].length === 0
                ) {
                    return false;
                }
                return true;
            })
        );
        order.update({ _code_activated_coupon_ids: [["unlink", ...toUnlink]] });
    },
    async activateCode(code) {
        const order = this.get_order();
        const rule = this.models["loyalty.rule"].find((rule) => {
            return rule.mode === "with_code" && (rule.promo_barcode === code || rule.code === code);
        });
        let claimableRewards = null;
        let coupon = null;
        if (rule) {
            const date_order = DateTime.fromSQL(order.date_order);
            if (
                rule.program_id.date_from &&
                date_order < rule.program_id.date_from.startOf("day")
            ) {
                return _t("That promo code program is not yet valid.");
            }
            if (rule.program_id.date_to && date_order > rule.program_id.date_to.endOf("day")) {
                return _t("That promo code program is expired.");
            }
            const program_pricelists = rule.program_id.pricelist_ids;
            if (
                program_pricelists.length > 0 &&
                (!order.pricelist_id ||
                    !program_pricelists.some((pr) => pr.id === order.pricelist_id.id))
            ) {
                return _t("That promo code program requires a specific pricelist.");
            }
            if (order.uiState.codeActivatedProgramRules.includes(rule.id)) {
                return _t("That promo code program has already been activated.");
            }
            order.uiState.codeActivatedProgramRules.push(rule.id);
            await this.orderUpdateLoyaltyPrograms();
            claimableRewards = order.getClaimableRewards(false, rule.program_id.id);
        } else {
            if (order._code_activated_coupon_ids.find((coupon) => coupon.code === code)) {
                return _t("That coupon code has already been scanned and activated.");
            }
            const customerId = order.get_partner() ? order.get_partner().id : false;
            const { successful, payload } = await this.data.call("pos.config", "use_coupon_code", [
                [this.config.id],
                code,
                order.date_order,
                customerId,
                order.pricelist_id ? order.pricelist_id.id : false,
            ]);
            if (successful) {
                // Allow rejecting a gift card that is not yet paid.
                const program = this.models["loyalty.program"].get(payload.program_id);
                if (program && program.program_type === "gift_card" && !payload.has_source_order) {
                    const confirmed = await ask(this.dialog, {
                        title: _t("Unpaid gift card"),
                        body: _t(
                            "This gift card is not linked to any order. Do you really want to apply its reward?"
                        ),
                    });
                    if (!confirmed) {
                        return _t("Unpaid gift card rejected.");
                    }
                }
                // TODO JCB: It's possible that the coupon is already loaded. We should check for that.
                //   - At the moment, creating a new one with existing id creates a duplicate.
                coupon = this.models["loyalty.card"].create({
                    id: payload.coupon_id,
                    code: code,
                    program_id: this.models["loyalty.program"].get(payload.program_id),
                    partner_id: this.models["res.partner"].get(payload.partner_id),
                    points: payload.points,
                    // TODO JCB: make the expiration_date work.
                    // expiration_date: payload.expiration_date,
                });
                order.update({ _code_activated_coupon_ids: [["link", coupon]] });
                await this.orderUpdateLoyaltyPrograms();
                claimableRewards = order.getClaimableRewards(coupon.id);
            } else {
                return payload.error_message;
            }
        }
        if (claimableRewards && claimableRewards.length === 1) {
            if (
                claimableRewards[0].reward.reward_type !== "product" ||
                !claimableRewards[0].reward.multi_product
            ) {
                order._applyReward(claimableRewards[0].reward, claimableRewards[0].coupon_id);
                this.updateRewards();
            }
        }
        if (!rule && order.lines.length === 0 && coupon) {
            return _t(
                "Gift Card: %s\nBalance: %s",
                code,
                this.env.utils.formatCurrency(coupon.points)
            );
        }
        return true;
    },
    async checkMissingCoupons() {
        // This function must stay sequential to avoid potential concurrency errors.
        const order = this.get_order();
        await mutex.exec(async () => {
            if (!order.invalidCoupons) {
                return;
            }
            order.invalidCoupons = false;
            order.uiState.couponPointChanges = Object.fromEntries(
                Object.entries(order.uiState.couponPointChanges).filter(([k, pe]) =>
                    this.models["loyalty.card"].get(pe.coupon_id)
                )
            );
        });
    },
    async addLineToCurrentOrder(vals, opt = {}, configure = true) {
        const product = vals.product_id;
        const order = this.get_order();
        const linkedPrograms = (
            this.models["loyalty.program"].getBy("trigger_product_ids", product.id) || []
        ).filter((p) => ["gift_card", "ewallet"].includes(p.program_type));
        let selectedProgram = null;
        if (linkedPrograms.length > 1) {
            selectedProgram = await makeAwaitable(this.dialog, SelectionPopup, {
                title: _t("Select program"),
                list: linkedPrograms.map((program) => ({
                    id: program.id,
                    item: program,
                    label: program.name,
                })),
            });
            if (!selectedProgram) {
                return;
            }
        } else if (linkedPrograms.length === 1) {
            selectedProgram = linkedPrograms[0];
        }

        const orderTotal = this.get_order().get_total_with_tax();
        if (
            selectedProgram &&
            ["gift_card", "ewallet"].includes(selectedProgram.program_type) &&
            orderTotal < 0
        ) {
            opt.price_unit = -orderTotal;
        }
        if (selectedProgram && selectedProgram.program_type == "gift_card") {
            const shouldProceed = await this._setupGiftCardOptions(selectedProgram, opt);
            if (!shouldProceed) {
                return;
            }
        } else if (selectedProgram && selectedProgram.program_type == "ewallet") {
            const shouldProceed = await this.setupEWalletOptions(selectedProgram, opt);
            if (!shouldProceed) {
                return;
            }
        }
        const potentialRewards = this.getPotentialFreeProductRewards();
        const rewardsToApply = [];
        for (const reward of potentialRewards) {
            for (const reward_product_id of reward.reward.reward_product_ids) {
                if (reward_product_id.id == product.id) {
                    rewardsToApply.push(reward);
                }
            }
        }

        // move price_unit from opt to vals
        if (opt.price_unit !== undefined) {
            vals.price_unit = opt.price_unit;
            delete opt.price_unit;
        }

        const result = await super.addLineToCurrentOrder(vals, opt, configure);

        await this.updatePrograms();
        if (rewardsToApply.length == 1) {
            const reward = rewardsToApply[0];
            order._applyReward(reward.reward, reward.coupon_id, { product });
        }
        this.updateRewards();

        return result;
    },
    /**
     * Sets up the options for the gift card product.
     * @param {object} program
     * @param {object} options
     * @returns {Promise<boolean>} whether to proceed with adding the product or not
     */
    async _setupGiftCardOptions(program, options) {
        options.quantity = 1;
        options.merge = false;
        options.eWalletGiftCardProgram = program;

        return true;
    },
    async setupEWalletOptions(program, options) {
        options.quantity = 1;
        options.merge = false;
        options.eWalletGiftCardProgram = program;
        return true;
    },
    /**
     * Returns the reward such that when its reward product is added
     * in the order, it will be added as free. That is, when added,
     * it comes with the corresponding reward product line.
     */
    async pay() {
        const currentOrder = this.get_order();
        const eWalletLine = currentOrder
            .get_orderlines()
            .find((line) => line.getEWalletGiftCardProgramType() === "ewallet");

        if (eWalletLine && !currentOrder.get_partner()) {
            const confirmed = await ask(this.dialog, {
                title: _t("Customer needed"),
                body: _t("eWallet requires a customer to be selected"),
            });
            if (confirmed) {
                await this.selectPartner();
            }
        } else {
            return super.pay(...arguments);
        }
    },
    getPotentialFreeProductRewards() {
        const order = this.get_order();
        const allCouponPrograms = Object.values(order.uiState.couponPointChanges)
            .map((pe) => {
                return {
                    program_id: pe.program_id,
                    coupon_id: pe.coupon_id,
                };
            })
            .concat(
                order._code_activated_coupon_ids.map((coupon) => {
                    return {
                        program_id: coupon.program_id.id,
                        coupon_id: coupon.id,
                    };
                })
            );
        const result = [];
        for (const couponProgram of allCouponPrograms) {
            const program = this.models["loyalty.program"].get(couponProgram.program_id);
            if (
                program.pricelist_ids.length > 0 &&
                (!order.pricelist_id ||
                    !program.pricelist_ids.some((pl) => pl.id === order.pricelist_id.id))
            ) {
                continue;
            }

            const points = order._getRealCouponPoints(couponProgram.coupon_id);
            const hasLine = order.lines.filter((line) => !line.is_reward_line).length > 0;
            for (const reward of program.reward_ids.filter(
                (reward) => reward.reward_type == "product"
            )) {
                if (points < reward.required_points) {
                    continue;
                }
                // Loyalty program (applies_on == 'both') should needs an orderline before it can apply a reward.
                const considerTheReward =
                    program.applies_on !== "both" || (program.applies_on == "both" && hasLine);
                if (reward.reward_type === "product" && considerTheReward) {
                    let hasPotentialQty = true;
                    let potentialQty;
                    for (const { id } of reward.reward_product_ids) {
                        const product = this.models["product.product"].get(id);
                        potentialQty = order._computePotentialFreeProductQty(
                            reward,
                            product,
                            points
                        );
                        if (potentialQty <= 0) {
                            hasPotentialQty = false;
                        }
                    }
                    if (hasPotentialQty) {
                        result.push({
                            coupon_id: couponProgram.coupon_id,
                            reward: reward,
                            potentialQty,
                        });
                    }
                }
            }
        }
        return result;
    },

    //@override
    async processServerData() {
        await super.processServerData();

        this.partnerId2CouponIds = {};

        this.computeDiscountProductIdsForAllRewards({
            model: "product.product",
            ids: Array.from(this.data.records["product.product"].keys()),
        });

        this.models["product.product"].addEventListener(
            "create",
            this.computeDiscountProductIdsForAllRewards.bind(this)
        );

        for (const program of this.models["loyalty.program"].getAll()) {
            if (program.date_to) {
                program.date_to = DateTime.fromISO(program.date_to);
            }
            if (program.date_from) {
                program.date_from = DateTime.fromISO(program.date_from);
            }
        }

        for (const rule of this.models["loyalty.rule"].getAll()) {
            rule.validProductIds = new Set(rule.raw.valid_product_ids);
        }

        this.models["loyalty.card"].addEventListener("create", (records) => {
            records = records.ids.map((record) => this.models["loyalty.card"].get(record));
            this.computePartnerCouponIds(records);
        });
        this.computePartnerCouponIds();
    },

    computeDiscountProductIdsForAllRewards(data) {
        const products = this.models[data.model].readMany(data.ids);
        for (const reward of this.models["loyalty.reward"].getAll()) {
            this.compute_discount_product_ids(reward, products);
        }
    },

    computePartnerCouponIds(loyaltyCards = null) {
        const cards = loyaltyCards || this.models["loyalty.card"].getAll();
        for (const card of cards) {
            if (!card.partner_id || card.id < 0) {
                continue;
            }

            if (!this.partnerId2CouponIds[card.partner_id.id]) {
                this.partnerId2CouponIds[card.partner_id.id] = new Set();
            }

            this.partnerId2CouponIds[card.partner_id.id].add(card.id);
        }
    },

    compute_discount_product_ids(reward, products) {
        const reward_product_domain = JSON.parse(reward.reward_product_domain);
        if (!reward_product_domain) {
            return;
        }

        const domain = new Domain(reward_product_domain);

        try {
            reward.update({
                all_discount_product_ids: [
                    ["link", ...products.filter((p) => domain.contains(p.serialize()))],
                ],
            });
        } catch (error) {
            if (!(error instanceof InvalidDomainError || error instanceof TypeError)) {
                throw error;
            }
            const index = this.models["loyalty.reward"].indexOf(reward);
            if (index != -1) {
                this.dialog.add(AlertDialog, {
                    title: _t("A reward could not be loaded"),
                    body: _t(
                        'The reward "%s" contain an error in its domain, your domain must be compatible with the PoS client',
                        this.models["loyalty.reward"].getAll()[index].description
                    ),
                });

                this.models["loyalty.reward"].delete(reward.id);
            }
        }
    },
    async initServerData() {
        await super.initServerData(...arguments);
        if (this.selectedOrderUuid) {
            this.updateRewards();
        }
    },
    /**
     * Fetches `loyalty.card` records from the server and adds/updates them in our cache.
     *
     * @param {domain} domain For the search
     * @param {int} limit Default to 1
     */
    async fetchCoupons(domain, limit = 1) {
        return await this.data.searchRead(
            "loyalty.card",
            domain,
            this.data.fields["loyalty.card"],
            { limit }
        );
    },
    /**
     * Fetches a loyalty card for the given program and partner, put in cache afterwards
     *  if a matching card is found in the cache, that one is used instead.
     * If no card is found a local only card will be created until the order is validated.
     *
     * @param {int} programId
     * @param {int} partnerId
     */
    async fetchLoyaltyCard(programId, partnerId) {
        const coupon = this.models["loyalty.card"].find(
            (c) => c.partner_id?.id === partnerId && c.program_id?.id === programId
        );
        if (coupon) {
            return coupon;
        }
        const fetchedCoupons = await this.fetchCoupons([
            ["partner_id", "=", partnerId],
            ["program_id", "=", programId],
        ]);
        let dbCoupon = fetchedCoupons.length > 0 ? fetchedCoupons[0] : null;
        if (!dbCoupon) {
            dbCoupon = await this.models["loyalty.card"].create({
                id: loyaltyIdsGenerator(),
                code: null,
                program_id: this.models["loyalty.program"].get(programId),
                partner_id: this.models["res.partner"].get(partnerId),
                points: 0,
                expiration_date: null,
            });
        }
        return dbCoupon;
    },
    getLoyaltyCards(partner) {
        const loyaltyCards = [];
        if (this.partnerId2CouponIds[partner.id]) {
            this.partnerId2CouponIds[partner.id].forEach((couponId) =>
                loyaltyCards.push(this.models["loyalty.card"].get(couponId))
            );
        }
        return loyaltyCards;
    },
    /**
     * IMPROVEMENT: It would be better to update the local order object instead of creating a new one.
     *   - This way, we don't need to remember the lines linked to negative coupon ids and relink them after pushing the order.
     */
    async preSyncAllOrders(orders) {
        await super.preSyncAllOrders(orders);

        for (const order of orders) {
            Object.assign(
                this.couponByLineUuidCache,
                order.lines.reduce((agg, line) => {
                    if (line.coupon_id && line.coupon_id.id < 0) {
                        return { ...agg, [line.uuid]: line.coupon_id.id };
                    } else {
                        return agg;
                    }
                }, {})
            );
            Object.assign(
                this.rewardProductByLineUuidCache,
                order.lines.reduce((agg, line) => {
                    if (line._reward_product_id) {
                        return { ...agg, [line.uuid]: line._reward_product_id.id };
                    } else {
                        return agg;
                    }
                }, {})
            );
        }
    },
    postSyncAllOrders(orders) {
        super.postSyncAllOrders(orders);

        for (const order of orders) {
            for (const line of order.lines) {
                if (line.uuid in this.couponByLineUuidCache) {
                    line.update({
                        coupon_id: this.models["loyalty.card"].get(
                            this.couponByLineUuidCache[line.uuid]
                        ),
                    });
                }
            }
            for (const line of order.lines) {
                if (line.uuid in this.rewardProductByLineUuidCache) {
                    line.update({
                        _reward_product_id: this.models["product.product"].get(
                            this.rewardProductByLineUuidCache[line.uuid]
                        ),
                    });
                }
            }
        }
    },
});

```

## File: static\src\portal\loyalty_card_dialog.xml

```xml
<templates>
    <t t-inherit="loyalty.portal_loyalty_card_dialog" t-inherit-mode="extension">
        <div name="history_lines" position="before">
            <div
                t-if="props.program.program_type == 'loyalty'"
                class="d-flex align-items-center flex-column"
            >
                <img
                    t-att-src="`/report/barcode/Code128/${props.card.code}?&amp;width=350&amp;height=100`"
                    style="width:400px;height:100px"
                />
                <span t-out="props.card.code" class="fs-5"/>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\utils\manage_giftcard_popup\manage_giftcard_popup.js

```javascript
import { Component, onMounted, useRef, useState } from "@odoo/owl";
import { Dialog } from "@web/core/dialog/dialog";
import { useService } from "@web/core/utils/hooks";
import { DateTimeInput } from "@web/core/datetime/datetime_input";
import { serializeDate } from "@web/core/l10n/dates";

export class ManageGiftCardPopup extends Component {
    static template = "pos_loyalty.ManageGiftCardPopup";
    static components = { Dialog, DateTimeInput };
    static props = {
        title: String,
        placeholder: { type: String, optional: true },
        rows: { type: Number, optional: true },
        getPayload: Function,
        close: Function,
    };
    static defaultProps = {
        startingValue: "",
        placeholder: "",
        rows: 1,
    };

    setup() {
        this.ui = useState(useService("ui"));
        this.state = useState({
            inputValue: this.props.startingValue,
            amountValue: "",
            error: false,
            amountError: false,
            expirationDate: luxon.DateTime.now().plus({ year: 1 }),
        });
        this.inputRef = useRef("input");
        this.amountInputRef = useRef("amountInput");
        onMounted(this.onMounted);
    }

    onMounted() {
        // Removing the main "DateTimeInput" component's class "o_input" and
        // adding the CSS classes "form-control" and "form-control-lg" for styling the form input with Bootstrap.
        const expirationDateInput = document.querySelector(".o_exp_date_container").children[1];
        expirationDateInput.classList.remove("o_input");
        expirationDateInput.classList.add("form-control", "form-control-lg");
        this.inputRef.el.focus();
    }

    addBalance() {
        if (!this.validateCode()) {
            return;
        }
        this.props.getPayload(
            this.state.inputValue,
            parseFloat(this.state.amountValue),
            this.state.expirationDate ? serializeDate(this.state.expirationDate) : false
        );
        this.props.close();
    }

    close() {
        this.props.close();
    }

    validateCode() {
        const { inputValue, amountValue } = this.state;
        if (inputValue.trim() === "") {
            this.state.error = true;
            return false;
        }
        if (amountValue.trim() === "") {
            this.state.amountError = true;
            return false;
        }
        return true;
    }

    onExpDateChange(date) {
        this.state.expirationDate = date;
    }
}

```

## File: static\src\utils\manage_giftcard_popup\manage_giftcard_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_loyalty.ManageGiftCardPopup">
        <Dialog title="props.title" size="'md'">
            <input id="code" t-att-rows="props.rows" class="form-control form-control-lg mx-auto" type="text" t-model="state.inputValue" t-ref="input" t-att-placeholder="props.placeholder" t-att-style="state.error ? 'border-color: red;' : ''" />
            <div class="mt-3 d-flex">
                <div t-attf-class="col align-items-center d-flex {{!ui.isSmall? 'me-2 w-50': ''}}">
                    <div class="col-form-label text-center pe-0 me-4 fs-5">Amount</div>
                    <div t-attf-class="{{ui.isSmall? 'flex-grow-1' : ''}}">
                        <input id="amount" class="form-control form-control-lg" type="number" t-model="state.amountValue" t-ref="amountInput" placeholder="Enter amount" t-att-style="state.amountError ? 'border-color: red;' : ''"/>
                    </div>
                </div>
                <div t-if="!ui.isSmall" class="d-flex ms-2 w-50 o_exp_date_container">
                    <div class="col-form-label text-center pe-0 me-4 fs-5">Expiration</div>
                    <DateTimeInput
                        type="'date'"
                        value="state.expirationDate"
                        onChange.bind="onExpDateChange" />
                </div>
            </div>
            <div t-if="ui.isSmall" class="d-flex my-2 o_exp_date_container">
                <div class="col-form-label text-center pe-0 me-2 fs-5">Expiration</div>
                <DateTimeInput
                    type="'date'"
                    value="state.expirationDate"
                    onChange.bind="onExpDateChange" />
            </div>
            <t t-set-slot="footer">
                <button class="btn btn-primary o-default-button" t-on-click="addBalance">Add Balance</button>
            </t>
        </Dialog>
    </t>
</templates>

```

## File: views\loyalty_card_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_card_view_form_inherit_pos_loyalty" model="ir.ui.view">
        <field name="name">loyalty.card.view.form.inherit.pos.loyalty</field>
        <field name="model">loyalty.card</field>
        <field name="inherit_id" ref="loyalty.loyalty_card_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="source_pos_order_id" readonly="1" invisible="not source_pos_order_id"/>
            </xpath>
        </field>
    </record>    
</odoo>

```

## File: views\loyalty_mail_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="loyalty_mail_view_tree_inherit_pos_loyalty" model="ir.ui.view">
        <field name="name">loyalty.mail.view.list.inherit.pos.loyalty</field>
        <field name="model">loyalty.mail</field>
        <field name="inherit_id" ref="loyalty.loyalty_mail_view_tree"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="pos_report_print_id" invisible="trigger != 'create'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="loyalty_program_view_form_inherit_pos_loyalty" model="ir.ui.view">
        <field name="name">loyalty.program.view.form.inherit.pos.loyalty</field>
        <field name="model">loyalty.program</field>
        <field name="inherit_id" ref="loyalty.loyalty_program_view_form"/>
        <field name="arch" type="xml">
            <field name="mail_template_id" position="after">
                <field name="pos_report_print_id" invisible="program_type != 'gift_card'" />
            </field>
            <xpath expr="//label[@for='available_on']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="inside">
                <span class="d-inline-flex text-break">
                    <field name="pos_ok"/>
                    <label for="pos_ok"/>
                </span>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="after">
                <field name="pos_config_ids" string="Point of Sale" widget="many2many_tags" invisible="not pos_ok" options="{'create': False}" placeholder="All PoS"/>
            </xpath>
        </field>
    </record>

    <record id="loyalty_program_view_tree_inherit_pos_loyalty" model="ir.ui.view">
        <field name="name">loyalty.program.view.list.inherit.pos.loyalty</field>
        <field name="model">loyalty.program</field>
        <field name="inherit_id" ref="loyalty.loyalty_program_view_tree"/>
        <field name="arch" type="xml">
            <field name="company_id" position="before">
                <field name="pos_config_ids" widget="many2many_tags"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\pos_loyalty_menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        id="menu_discount_loyalty_type_config"
        action="loyalty.loyalty_program_discount_loyalty_action"
        name="Discount &amp; Loyalty"
        parent="point_of_sale.pos_config_menu_catalog"
        groups="point_of_sale.group_pos_manager"
        sequence="91"
    />

    <menuitem
        id="menu_gift_ewallet_type_config"
        action="loyalty.loyalty_program_gift_ewallet_action"
        name="Gift cards &amp; eWallet"
        parent="point_of_sale.pos_config_menu_catalog"
        groups="point_of_sale.group_pos_manager"
        sequence="92"
    />
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_view_form_inherit_pos_loyalty" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_loyalty</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='pos-loyalty']" position="inside">
                <div class="content-group" invisible="not module_loyalty">
                    <div class="mt8">
                        <button name="%(loyalty.loyalty_program_discount_loyalty_action)d" icon="oi-arrow-right" type="action" string="Discount &amp; Loyalty" class="btn-link"/>
                    </div>
                    <div class="mt8" id="button_loyalty_program" invisible="is_kiosk_mode">
                        <button name="%(loyalty.loyalty_program_gift_ewallet_action)d" icon="oi-arrow-right" type="action" string="Gift cards &amp; eWallet" class="btn-link"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_form" model="ir.ui.view">
        <field name="name">res.partner.view.buttons</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="loyalty.res_partner_form"/>
        <field name="arch" type="xml">
            <button name="action_view_loyalty_cards" position="attributes">
                <attribute name="groups" separator="," add="point_of_sale.group_pos_user"/>
            </button>
        </field>
    </record>

</odoo>

```

