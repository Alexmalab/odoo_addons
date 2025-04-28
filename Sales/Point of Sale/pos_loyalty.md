# Odoo Module: pos_loyalty

Category: Sales/Point Of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import tests

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
        'point_of_sale._assets_pos': [
            'pos_loyalty/static/src/**/*',
        ],
        'web.assets_tests': [
            'pos_loyalty/static/tests/tours/**/*',
        ],
    },
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

from odoo import fields, models

class LoyaltyCard(models.Model):
    _inherit = 'loyalty.card'

    source_pos_order_id = fields.Many2one('pos.order', "PoS Order Reference",
        help="PoS order where this coupon was generated.")

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.tools import unique
from odoo.exceptions import UserError

class LoyaltyProgram(models.Model):
    _inherit = 'loyalty.program'

    # NOTE: `pos_config_ids` satisfies an excpeptional use case: when no PoS is specified, the loyalty program is
    # applied to every PoS. You can access the loyalty programs of a PoS using _get_program_ids() of pos.config
    pos_config_ids = fields.Many2many('pos.config', compute="_compute_pos_config_ids", store=True, readonly=False, string="Point of Sales", help="Restrict publishing to those shops.")
    pos_order_count = fields.Integer("PoS Order Count", compute='_compute_pos_order_count')
    pos_ok = fields.Boolean("Point of Sale", default=True)
    pos_report_print_id = fields.Many2one('ir.actions.report', string="Print Report", domain=[('model', '=', 'loyalty.card')], compute='_compute_pos_report_print_id', inverse='_inverse_pos_report_print_id', readonly=False,
        help="This is used to print the generated gift cards from PoS.")

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
                    raise UserError(_("You must set '%s' before setting '%s'.", mail_template_label, pos_report_print_label))
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
                WITH reward_to_orders_count AS (
                 SELECT reward.id                    AS lr_id,
                        COUNT(DISTINCT pos_order.id) AS orders_count
                   FROM pos_order_line line
                   JOIN pos_order ON line.order_id = pos_order.id
                   JOIN loyalty_reward reward ON line.reward_id = reward.id
               GROUP BY lr_id
              ),
              program_to_reward AS (
                 SELECT reward.id  AS reward_id,
                        program.id AS program_id
                   FROM loyalty_program program
                   JOIN loyalty_reward reward ON reward.program_id = program.id
                  WHERE program.id = ANY (%s)
              )
       SELECT program_to_reward.program_id,
              SUM(reward_to_orders_count.orders_count)
         FROM program_to_reward
    LEFT JOIN reward_to_orders_count ON reward_to_orders_count.lr_id = program_to_reward.reward_id
     GROUP BY program_to_reward.program_id
                """
        self._cr.execute(query, (self.ids,))
        res = self._cr.dictfetchall()
        res = {k['program_id']: k['sum'] for k in res}

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

from odoo import models

class LoyaltyReward(models.Model):
    _inherit = 'loyalty.reward'

    def _get_discount_product_values(self):
        res = super()._get_discount_product_values()
        for vals in res:
            vals.update({'taxes_id': False})
        return res

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
from odoo.tools import ustr

class LoyaltyRule(models.Model):
    _inherit = 'loyalty.rule'

    valid_product_ids = fields.Many2many(
        'product.product', "Valid Products", compute='_compute_valid_product_ids',
        help="These are the products that are valid for this rule.")
    any_product = fields.Boolean(
        compute='_compute_valid_product_ids', help="Technical field, whether all product match")

    promo_barcode = fields.Char("Barcode", compute='_compute_promo_barcode', store=True, readonly=False,
        help="A technical field used as an alternative to the promo code. "
        "This is automatically generated when the promo code is changed."
    )

    @api.depends('product_ids', 'product_category_id', 'product_tag_id') #TODO later: product tags
    def _compute_valid_product_ids(self):
        domain_products = {}
        for rule in self:
            if rule.product_ids or\
                rule.product_category_id or\
                rule.product_tag_id or\
                rule.product_domain not in ('[]', "[['sale_ok', '=', True]]"):
                domain = rule._get_valid_product_domain()
                domain = expression.AND([[('available_in_pos', '=', True)], domain])
                product_ids = domain_products.get(ustr(domain))
                if product_ids is None:
                    product_ids = self.env['product.product'].search(domain, order="id")
                    domain_products[ustr(domain)] = product_ids
                rule.valid_product_ids = product_ids
                rule.any_product = False
            else:
                rule.any_product = True
                rule.valid_product_ids = self.env['product.product']

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

    gift_card_settings = fields.Selection(
        [
            ("create_set", "Generate PDF cards"),
            ("scan_use", "Scan existing cards"),
        ],
        string="Gift Cards settings",
        default="create_set",
        help="Defines the way you want to set your gift cards.",
    )
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
                if self.gift_card_settings == "create_set":
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
            'code': p.get('barcode') or self.env['loyalty.card']._generate_code(),
            'points': 0,
            'expiration_date': p.get('date_to', False),
            'source_pos_order_id': self.id,
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
                'usages': program.total_order_count,
            } for program in all_coupons.program_id],
            'new_coupon_info': [{
                'program_name': coupon.program_id.name,
                'expiration_date': coupon.expiration_date,
                'code': coupon.code,
            } for coupon in new_coupons if (
                coupon.program_id.applies_on == 'future'
                # Don't send the coupon code for the gift card and ewallet programs.
                # It should not be printed in the ticket.
                and coupon.program_id.program_type not in ['gift_card', 'ewallet']
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

    def _add_mail_attachment(self, name, ticket):
        attachment = super()._add_mail_attachment(name, ticket)
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

from odoo import fields, models

class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    is_reward_line = fields.Boolean(
        help="Whether this line is part of a reward or not.")
    reward_id = fields.Many2one(
        'loyalty.reward', "Reward", ondelete='restrict',
        help="The reward associated with this line.")
    coupon_id = fields.Many2one(
        'loyalty.card', "Coupon", ondelete='restrict',
        help="The coupon used to claim that reward.")
    reward_identifier_code = fields.Char(help="""
        Technical field used to link multiple reward lines from the same reward together.
    """)
    points_cost = fields.Float(help="How many point this reward cost on the coupon.")

    def _order_line_fields(self, line, session_id=None):
        res = super()._order_line_fields(line, session_id)
        # coupon_id may be negative in case of new coupons, they will be added after validating the order.
        if 'coupon_id' in res[2] and res[2]['coupon_id'] < 1:
            res[2].pop('coupon_id')
        return res

    def _is_not_sellable_line(self):
        return super().is_not_sellable_line() or self.reward_id

    def _export_for_ui(self, orderline):
        result = super()._export_for_ui(orderline)
        result['is_reward_line'] = orderline.is_reward_line
        result['reward_id'] = orderline.reward_id.id
        result['coupon_id'] = orderline.coupon_id.id
        result['reward_identifier_code'] = orderline.reward_identifier_code
        result['points_cost'] = orderline.points_cost
        return result

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import OR
import ast
import json

class PosSession(models.Model):
    _inherit = 'pos.session'

    def _pos_ui_models_to_load(self):
        result = super()._pos_ui_models_to_load()
        if self.config_id._get_program_ids():
            result += [
                'loyalty.program',
                'loyalty.rule',
                'loyalty.reward',
            ]
        return result

    def _loader_params_loyalty_program(self):
        return {
            'search_params': {
                'domain': [('id', 'in', self.config_id._get_program_ids().ids)],
                'fields': [
                    'name', 'trigger', 'applies_on', 'program_type', 'pricelist_ids', 'date_from',
                    'date_to', 'limit_usage', 'max_usage', 'is_nominative', 'portal_visible',
                    'portal_point_name', 'trigger_product_ids',
                ],
            },
        }

    def _loader_params_loyalty_rule(self):
        return {
            'search_params': {
                'domain': [('program_id', 'in', self.config_id._get_program_ids().ids)],
                'fields': ['program_id', 'valid_product_ids', 'any_product', 'currency_id',
                    'reward_point_amount', 'reward_point_split', 'reward_point_mode',
                    'minimum_qty', 'minimum_amount', 'minimum_amount_tax_mode', 'mode', 'code'],
            }
        }

    def _loader_params_loyalty_reward(self):
        return {
            'search_params': {
                'domain': [('program_id', 'in', self.config_id._get_program_ids().ids)],
                'fields': ['description', 'program_id', 'reward_type', 'required_points', 'clear_wallet', 'currency_id',
                    'discount', 'discount_mode', 'discount_applicability', 'all_discount_product_ids', 'is_global_discount',
                    'discount_max_amount', 'discount_line_product_id',
                    'multi_product', 'reward_product_ids', 'reward_product_qty', 'reward_product_uom_id', 'reward_product_domain'],
            }
        }

    def _get_pos_ui_loyalty_program(self, params):
        return self.env['loyalty.program'].search_read(**params['search_params'])

    def _get_pos_ui_loyalty_rule(self, params):
        return self.env['loyalty.rule'].search_read(**params['search_params'])

    def _get_pos_ui_loyalty_reward(self, params):
        rewards = self.env['loyalty.reward'].search_read(**params['search_params'])
        for reward in rewards:
            reward['reward_product_domain'] = self._replace_ilike_with_in(reward['reward_product_domain'])
        return rewards

    def _replace_ilike_with_in(self, domain_str):
        if domain_str == "null":
            return domain_str

        domain = json.loads(domain_str)
        for index, condition in enumerate(domain):
            if isinstance(condition, (list, tuple)) and len(condition) == 3:
                field_name, operator, value = condition
                field = self.env['product.product']._fields.get(field_name)

                if field and field.type == 'many2one' and operator in ('ilike', 'not ilike'):
                    comodel = self.env[field.comodel_name]
                    matching_ids = list(comodel._name_search(value, [], operator, limit=None))

                    new_operator = 'in' if operator == 'ilike' else 'not in'
                    domain[index] = [field_name, new_operator, matching_ids]

        return json.dumps(domain)

    def _get_pos_ui_product_product(self, params):
        result = super()._get_pos_ui_product_product(params)
        self = self.with_context(**params['context'])
        rewards = self.config_id._get_program_ids().reward_ids
        products = rewards.discount_line_product_id | rewards.reward_product_ids
        products |= self.config_id._get_program_ids().filtered(lambda p: p.program_type == 'ewallet').trigger_product_ids
        # Only load products that are not already in the result
        products = list(set(products.ids) - set(product['id'] for product in result))
        products = self.env['product.product'].search_read([('id', 'in', products)], fields=params['search_params']['fields'])
        self._process_pos_ui_product_product(products)
        result.extend(products)
        return result

    def _get_pos_ui_res_partner(self, params):
        partners = super()._get_pos_ui_res_partner(params)
        self._set_loyalty_cards(partners)
        return partners

    def get_pos_ui_res_partner_by_params(self, custom_search_params):
        partners = super().get_pos_ui_res_partner_by_params(custom_search_params)
        self._set_loyalty_cards(partners)
        return partners

    def _set_loyalty_cards(self, partners):
        # Map partner_id to its loyalty cards from all loyalty programs.
        loyalty_programs = self.config_id._get_program_ids().filtered(lambda p: p.program_type == 'loyalty')
        loyalty_card_fields = ['points', 'code', 'program_id']
        partner_id_to_loyalty_card = {}
        for partner, *field_values in self.env['loyalty.card']._read_group(
            domain=[('partner_id', 'in', [p['id'] for p in partners]), ('program_id', 'in', loyalty_programs.ids)],
            groupby=['partner_id'],
            aggregates=[f'{field_name}:array_agg' for field_name in loyalty_card_fields] + ['id:array_agg'],
        ):
            # field_values = [(a1, a2, ...), (b1, b2, ...), ..., (id1, id2, ...)]
            loyalty_cards = {}
            for *values, id_ in zip(*field_values):
                # values = [ak, bk, ...], id_ = idk
                loyalty_cards[id_] = dict(zip(loyalty_card_fields, values))
            partner_id_to_loyalty_card[partner.id] = loyalty_cards

        # Assign loyalty cards to each partner to load.
        for partner in partners:
            partner['loyalty_cards'] = partner_id_to_loyalty_card.get(partner['id'], {})

        return partners

    def _pos_data_process(self, loaded_data):
        super()._pos_data_process(loaded_data)

        # Additional post processing to link gift card and ewallet programs
        # to their rules' products.
        # Important because points from their products are only counted once.
        product_id_to_program_ids = {}
        for program in self.config_id._get_program_ids():
            if program.program_type in ['gift_card', 'ewallet']:
                for product in program.trigger_product_ids:
                    product_id_to_program_ids.setdefault(product['id'], [])
                    product_id_to_program_ids[product['id']].append(program['id'])

        loaded_data['product_id_to_program_ids'] = product_id_to_program_ids
        product_product_fields = self.env['product.product'].fields_get(self._loader_params_product_product()['search_params']['fields'])
        loaded_data['field_types'] = {
            'product.product': {f:v['type'] for f, v in product_product_fields.items()}
        }

    def _loader_params_product_product(self):
        params = super()._loader_params_product_product()
        # this is usefull to evaluate reward domain in frontend
        params['search_params']['fields'].append('all_product_tag_ids')
        return params

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_gift_card_settings = fields.Selection(related='pos_config_id.gift_card_settings', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
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
from . import res_config_settings

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

```

## File: static\src\app\control_buttons\e_wallet_button\e_wallet_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { useService } from "@web/core/utils/hooks";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component } from "@odoo/owl";

export class eWalletButton extends Component {
    static template = "point_of_sale.eWalletButton";

    setup() {
        this.popup = useService("popup");
        this.pos = usePos();
    }

    _getEWalletRewards(order) {
        const claimableRewards = order.getClaimableRewards();
        return claimableRewards.filter((reward_line) => {
            const coupon = this.pos.couponCache[reward_line.coupon_id];
            return coupon && reward_line.reward.program_id.program_type == 'ewallet' && !coupon.isExpired();
        });
    }
    _getEWalletPrograms() {
        return this.pos.programs.filter((p) => p.program_type == "ewallet");
    }
    async _onClickWalletButton() {
        const order = this.pos.get_order();
        const eWalletPrograms = this.pos.programs.filter((p) => p.program_type == "ewallet");
        const orderTotal = order.get_total_with_tax();
        const eWalletRewards = this._getEWalletRewards(order);
        if (eWalletRewards.length === 0 && orderTotal >= 0) {
            this.popup.add(ErrorPopup, {
                title: _t('No valid eWallet found'),
                body: _t('You either have not created an eWallet or all your eWallets have expired.'),
            });
            return;
        }
        if (orderTotal < 0 && eWalletPrograms.length >= 1) {
            let selectedProgram = null;
            if (eWalletPrograms.length == 1) {
                selectedProgram = eWalletPrograms[0];
            } else {
                const { confirmed, payload } = await this.popup.add(SelectionPopup, {
                    title: _t("Refund with eWallet"),
                    list: eWalletPrograms.map((program) => ({
                        id: program.id,
                        item: program,
                        label: program.name,
                    })),
                });
                if (confirmed) {
                    selectedProgram = payload;
                }
            }
            if (selectedProgram) {
                const eWalletProduct = this.pos.db.get_product_by_id(
                    selectedProgram.trigger_product_ids[0]
                );
                this.pos.addProductFromUi(eWalletProduct, {
                    price: -orderTotal,
                    merge: false,
                    eWalletGiftCardProgram: selectedProgram,
                });
            }
        } else if (eWalletRewards.length >= 1) {
            let eWalletReward = null;
            if (eWalletRewards.length == 1) {
                eWalletReward = eWalletRewards[0];
            } else {
                const { confirmed, payload } = await this.popup.add(SelectionPopup, {
                    title: _t("Use eWallet to pay"),
                    list: eWalletRewards.map(({ reward, coupon_id }) => ({
                        id: reward.id,
                        item: { reward, coupon_id },
                        label: `${reward.description} (${reward.program_id.name})`,
                    })),
                });
                if (confirmed) {
                    eWalletReward = payload;
                }
            }
            if (eWalletReward) {
                const result = order._applyReward(
                    eWalletReward.reward,
                    eWalletReward.coupon_id,
                    {}
                );
                if (result !== true) {
                    // Returned an error
                    this.popup.add(ErrorPopup, {
                        title: _t("Error"),
                        body: result,
                    });
                }
                order._updateRewards();
            }
        }
    }
    _shouldBeHighlighted(orderTotal, eWalletPrograms, eWalletRewards) {
        return (orderTotal < 0 && eWalletPrograms.length >= 1) || eWalletRewards.length >= 1;
    }
    _getText(orderTotal, eWalletPrograms, eWalletRewards) {
        if (orderTotal < 0 && eWalletPrograms.length >= 1) {
            return _t("eWallet Refund");
        } else if (eWalletRewards.length >= 1) {
            return _t("eWallet Pay");
        } else {
            return _t("eWallet");
        }
    }
}

ProductScreen.addControlButton({
    component: eWalletButton,
    condition: function () {
        return this.pos.programs.filter((p) => p.program_type == "ewallet").length > 0;
    },
});

```

## File: static\src\app\control_buttons\e_wallet_button\e_wallet_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="point_of_sale.eWalletButton">
        <t t-set="_order" t-value="pos.get_order()" />
        <t t-set="_orderTotal" t-value="_order.get_total_with_tax()" />
        <t t-set="_eWalletPrograms" t-value="_getEWalletPrograms()" />
        <t t-set="_eWalletRewards" t-value="_getEWalletRewards(_order)" />
        <span class="control-button btn btn-light rounded-0 fw-bolder" t-att-class="_shouldBeHighlighted(_orderTotal, _eWalletPrograms, _eWalletRewards) ? 'highlight' : ''" t-on-click="_onClickWalletButton">
            <i class="fa fa-credit-card me-1"></i>
            <span> </span>
            <span><t t-esc="_getText(_orderTotal, _eWalletPrograms, _eWalletRewards)" /></span>
        </span>
    </t>

 </templates>

```

## File: static\src\app\control_buttons\promo_code_button\promo_code_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useService } from "@web/core/utils/hooks";
import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class PromoCodeButton extends Component {
    static template = "pos_loyalty.PromoCodeButton";

    setup() {
        this.pos = usePos();
        this.popup = useService("popup");
    }

    async click() {
        let { confirmed, payload: code } = await this.popup.add(TextInputPopup, {
            title: _t("Enter Code"),
            startingValue: "",
            placeholder: _t("Gift card or Discount code"),
        });
        if (confirmed) {
            code = code.trim();
            if (code !== "") {
                this.pos.get_order().activateCode(code);
            }
        }
    }
}

ProductScreen.addControlButton({
    component: PromoCodeButton,
    condition: function () {
        return this.pos.programs.some((p) =>
            ["coupons", "promotion", "gift_card", "promo_code", "next_order_coupons"].includes(p.program_type)
        );
    },
});

```

## File: static\src\app\control_buttons\promo_code_button\promo_code_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_loyalty.PromoCodeButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-on-click="() => this.click()">
            <i class="fa fa-barcode me-1"></i>
            <span> </span>
            <span>Enter Code</span>
        </button>
    </t>

</templates>

```

## File: static\src\app\control_buttons\reset_programs_button\reset_programs_button.js

```javascript
/** @odoo-module **/

import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class ResetProgramsButton extends Component {
    static template = "pos_loyalty.ResetProgramsButton";

    setup() {
        this.pos = usePos();
    }
    _isDisabled() {
        return !this.pos.get_order().isProgramsResettable();
    }
    click() {
        this.pos.get_order()._resetPrograms();
    }
}

ProductScreen.addControlButton({
    component: ResetProgramsButton,
    condition: function () {
        return this.pos.programs.some((p) =>
            ["coupons", "promotion"].includes(p.program_type)
        );
    },
});

```

## File: static\src\app\control_buttons\reset_programs_button\reset_programs_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_loyalty.ResetProgramsButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-att-class="{'disabled': _isDisabled()}" t-on-click="() => this.click()">
            <i class="fa fa-star me-1"></i>
            <span> </span>
            <span>Reset Programs</span>
        </button>
    </t>

</templates>

```

## File: static\src\app\control_buttons\reward_button\reward_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useService } from "@web/core/utils/hooks";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component } from "@odoo/owl";

export class RewardButton extends Component {
    static template = "pos_loyalty.RewardButton";

    setup() {
        this.popup = useService("popup");
        this.pos = usePos();
        this.notification = useService("pos_notification");
    }

    /**
     * If rewards are the same, prioritize the one from freeProductRewards.
     * Make sure that the reward is claimable first.
     */
    _mergeFreeProductRewards(freeProductRewards, potentialFreeProductRewards) {
        const result = [];
        for (const reward of potentialFreeProductRewards) {
            if (!freeProductRewards.find((item) => item.reward.id === reward.reward.id)) {
                result.push(reward);
            }
        }
        return freeProductRewards.concat(result);
    }

    _getPotentialRewards() {
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
        const discountRewards = rewards.filter(({ reward }) => reward.reward_type == "discount");
        const freeProductRewards = rewards.filter(({ reward }) => reward.reward_type == "product");
        const potentialFreeProductRewards = this.pos.getPotentialFreeProductRewards();
        return discountRewards.concat(
            this._mergeFreeProductRewards(freeProductRewards, potentialFreeProductRewards)
        );
    }

    _isDisabled() {}

    hasClaimableRewards() {
        return this._getPotentialRewards().length > 0;
    }

    /**
     * Applies the reward on the current order, if multiple products can be claimed opens a popup asking for which one.
     *
     * @param {Object} reward
     * @param {Integer} coupon_id
     */
    async _applyReward(reward, coupon_id, potentialQty) {
        const order = this.pos.get_order();
        order.disabledRewards.delete(reward.id);

        const args = {};
        if (reward.reward_type === "product" && reward.multi_product) {
            const productsList = reward.reward_product_ids.map((product_id) => ({
                id: product_id,
                label: this.pos.db.get_product_by_id(product_id).display_name,
                item: product_id,
            }));
            const { confirmed, payload: selectedProduct } = await this.popup.add(SelectionPopup, {
                title: _t("Please select a product for this reward"),
                list: productsList,
            });
            if (!confirmed) {
                return false;
            }
            args["product"] = selectedProduct;
        }
        if (
            (reward.reward_type == "product" && reward.program_id.applies_on !== "both") ||
            (reward.program_id.applies_on == "both" && potentialQty)
        ) {
            this.pos.addProductToCurrentOrder(
                args["product"] || reward.reward_product_ids[0],
                { quantity: potentialQty || 1 }
            );
            return true;
        } else {
            const result = order._applyReward(reward, coupon_id, args);
            if (result !== true) {
                // Returned an error
                this.notification.add(result);
            }
            order._updateRewards();
            return result;
        }
    }

    async click() {
        const rewards = this._getPotentialRewards();
        if (rewards.length >= 1) {
            const rewardsList = rewards.map((reward) => ({
                id: reward.reward.id,
                label: reward.reward.description,
                description: reward.reward.program_id.name,
                item: reward,
            }));
            const { confirmed, payload: selectedReward } = await this.popup.add(SelectionPopup, {
                title: _t("Please select a reward"),
                list: rewardsList,
            });
            if (confirmed) {
                return this._applyReward(
                    selectedReward.reward,
                    selectedReward.coupon_id,
                    selectedReward.potentialQty
                );
            }
        }
        return false;
    }
}

ProductScreen.addControlButton({
    component: RewardButton,
    condition: function () {
        return this.pos.programs.length > 0;
    },
});

```

## File: static\src\app\control_buttons\reward_button\reward_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="pos_loyalty.RewardButton">
        <button class="control-button btn btn-light rounded-0 fw-bolder" t-attf-class="{{hasClaimableRewards() ? 'highlight text-action' : 'disabled'}}" t-on-click="() => this.click()">
            <i class="fa fa-star me-1"></i>
            <span> </span>
            <span>Reward</span>
        </button>
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
                        <br/>
                        <div t-esc='_loyaltyStat.program.name' class="pos-receipt-title" />
                        <br />
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
                    <br />
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
/** @odoo-module **/

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
        const program = this.pos.program_by_id[loyaltyCard.program_id];
        if (program.program_type === "ewallet") {
            return `${program.name}: ${this.env.utils.formatCurrency(loyaltyCard.balance)}`;
        }
        const balanceRepr = formatFloat(loyaltyCard.balance, { digits: [69, 2] });
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
/** @odoo-module */

import { PartnerListScreen } from "@point_of_sale/app/screens/partner_list/partner_list";
import { patch } from "@web/core/utils/patch";

patch(PartnerListScreen.prototype, {
    /**
     * Needs to be set to true to show the loyalty points in the partner list.
     * @override
     */
    get isBalanceDisplayed() {
        return true;
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { PosLoyaltyCard } from "@pos_loyalty/overrides/models/loyalty";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";

patch(PaymentScreen.prototype, {
    //@override
    async validateOrder(isForceValidate) {
        const pointChanges = {};
        const newCodes = [];
        for (const pe of Object.values(this.currentOrder.couponPointChanges)) {
            if (pe.coupon_id > 0) {
                pointChanges[pe.coupon_id] = pe.points;
            } else if (pe.barcode && !pe.giftCardId) {
                // New coupon with a specific code, validate that it does not exist
                newCodes.push(pe.barcode);
            }
        }
        for (const line of this.currentOrder._get_reward_lines()) {
            if (line.coupon_id < 1) {
                continue;
            }
            if (!pointChanges[line.coupon_id]) {
                pointChanges[line.coupon_id] = -line.points_cost;
            } else {
                pointChanges[line.coupon_id] -= line.points_cost;
            }
        }
        if (!(await this._isOrderValid(isForceValidate))) {
            return;
        }
        // No need to do an rpc if no existing coupon is being used.
        if (Object.keys(pointChanges || {}).length > 0 || newCodes.length) {
            try {
                const { successful, payload } = await this.orm.call(
                    "pos.order",
                    "validate_coupon_programs",
                    [[], pointChanges, newCodes]
                );
                // Payload may contain the points of the concerned coupons to be updated in case of error. (So that rewards can be corrected)
                const { couponCache } = this.pos;
                if (payload && payload.updated_points) {
                    for (const pointChange of Object.entries(payload.updated_points)) {
                        if (couponCache[pointChange[0]]) {
                            couponCache[pointChange[0]].balance = pointChange[1];
                        }
                    }
                }
                if (payload && payload.removed_coupons) {
                    for (const couponId of payload.removed_coupons) {
                        if (couponCache[couponId]) {
                            delete couponCache[couponId];
                        }
                    }
                    this.currentOrder.codeActivatedCoupons =
                        this.currentOrder.codeActivatedCoupons.filter(
                            (coupon) => !payload.removed_coupons.includes(coupon.id)
                        );
                }
                if (!successful) {
                    this.popup.add(ErrorPopup, {
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
        // Compile data for our function
        const { program_by_id, reward_by_id, couponCache } = this.pos;
        const rewardLines = order._get_reward_lines();
        const partner = order.get_partner();
        let couponData = Object.values(order.couponPointChanges).reduce((agg, pe) => {
            agg[pe.coupon_id] = Object.assign({}, pe, {
                points: pe.points - order._getPointsCorrection(program_by_id[pe.program_id]),
            });
            const program = program_by_id[pe.program_id];
            if (program.is_nominative && partner) {
                agg[pe.coupon_id].partner_id = partner.id;
            }
            if (program.program_type != 'loyalty') {
                agg[pe.coupon_id].date_to = program.date_to;
            }
            return agg;
        }, {});
        for (const line of rewardLines) {
            const reward = reward_by_id[line.reward_id];
            if (!couponData[line.coupon_id]) {
                couponData[line.coupon_id] = {
                    points: 0,
                    program_id: reward.program_id.id,
                    coupon_id: line.coupon_id,
                    barcode: false,
                };
                if (reward.program_type != 'loyalty') {
                    couponData[line.coupon_id].date_to = reward.program_id.date_to;
                }
            }
            if (!couponData[line.coupon_id].line_codes) {
                couponData[line.coupon_id].line_codes = [];
            }
            if (!couponData[line.coupon_id].line_codes.includes(line.reward_identifier_code)) {
                !couponData[line.coupon_id].line_codes.push(line.reward_identifier_code);
            }
            couponData[line.coupon_id].points -= line.points_cost;
        }
        // We actually do not care about coupons for 'current' programs that did not claim any reward, they will be lost if not validated
        couponData = Object.fromEntries(
            Object.entries(couponData).filter(([key, value]) => {
                const program = program_by_id[value.program_id];
                if (program.applies_on === "current") {
                    return value.line_codes && value.line_codes.length;
                }
                return true;
            })
        );
        if (Object.keys(couponData || []).length > 0) {
            const payload = await this.orm.call("pos.order", "confirm_coupon_programs", [
                server_ids,
                couponData,
            ]);
            if (payload.coupon_updates) {
                for (const couponUpdate of payload.coupon_updates) {
                    let dbCoupon = couponCache[couponUpdate.old_id];
                    if (dbCoupon) {
                        dbCoupon.id = couponUpdate.id;
                        dbCoupon.balance = couponUpdate.points;
                        dbCoupon.code = couponUpdate.code;
                    } else {
                        dbCoupon = new PosLoyaltyCard(
                            couponUpdate.code,
                            couponUpdate.id,
                            couponUpdate.program_id,
                            couponUpdate.partner_id,
                            couponUpdate.points
                        );
                    }
                    delete couponCache[couponUpdate.old_id];
                    couponCache[couponUpdate.id] = dbCoupon;
                }
            }
            // Update the usage count since it is checked based on local data
            if (payload.program_updates) {
                for (const programUpdate of payload.program_updates) {
                    const program = program_by_id[programUpdate.program_id];
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
        return super._postPushOrderResolve(order, server_ids);
    },
});

```

## File: static\src\overrides\components\product_screen\product_screen.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { useBarcodeReader } from "@point_of_sale/app/barcode/barcode_reader_hook";
import { patch } from "@web/core/utils/patch";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";
import { useService } from "@web/core/utils/hooks";

patch(ProductScreen.prototype, {
    setup() {
        super.setup(...arguments);
        this.notification = useService("pos_notification");
        useBarcodeReader({
            coupon: this._onCouponScan,
        });
    },
    _onCouponScan(code) {
        // IMPROVEMENT: Ability to understand if the scanned code is to be paid or to be redeemed.
        this.currentOrder.activateCode(code.base_code);
    },
    async updateSelectedOrderline({ buffer, key }) {
        const selectedLine = this.currentOrder.get_selected_orderline();
        if (key === "-") {
            if (selectedLine && selectedLine.eWalletGiftCardProgram) {
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
            const reward = this.pos.reward_by_id[selectedLine.reward_id];
            const { confirmed } = await this.popup.add(ConfirmPopup, {
                title: _t("Deactivating reward"),
                body: _t(
                    "Are you sure you want to remove %s from this order?\n You will still be able to claim it through the reward button.",
                    reward.description
                ),
                cancelText: _t("No"),
                confirmText: _t("Yes"),
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
        if (
            !selectedLine ||
            !selectedLine.is_reward_line ||
            (selectedLine.is_reward_line && ["", "remove"].includes(val))
        ) {
            super._setValue(val);
        }
        if (!selectedLine) {
            return;
        }
        if (selectedLine.is_reward_line && val === "remove") {
            this.currentOrder.disabledRewards.add(selectedLine.reward_id);
            const { couponCache } = this.pos;
            const coupon = couponCache[selectedLine.coupon_id];
            if (
                coupon &&
                coupon.id > 0 &&
                this.currentOrder.codeActivatedCoupons.find((c) => c.code === coupon.code)
            ) {
                delete couponCache[selectedLine.coupon_id];
                this.currentOrder.codeActivatedCoupons.splice(
                    this.currentOrder.codeActivatedCoupons.findIndex((coupon) => {
                        return coupon.id === selectedLine.coupon_id;
                    }),
                    1
                );
            }
        }
        if (!selectedLine.is_reward_line || (selectedLine.is_reward_line && val === "remove")) {
            this.currentOrder._updateRewards();
        }
    },
    async _barcodeProductAction(code) {
        await super._barcodeProductAction(code);
        this.currentOrder._updateRewards();
    },
    async _barcodeGS1Action(code) {
        await super._barcodeGS1Action(code);
        this.currentOrder._updateRewards();
    },

    async _showDecreaseQuantityPopup() {
        const result = await super._showDecreaseQuantityPopup();
        if (result) {
            this.currentOrder._updateRewards();
        }
    },
});

```

## File: static\src\overrides\components\product_screen\product_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_loyalty.ProductScreen" t-inherit="point_of_sale.ProductScreen" t-inherit-mode="extension">
		<xpath expr="//Orderline" position="inside" >
            <li t-if="line.isGiftCardOrEWalletReward()">
                Current Balance: <t t-esc="line.getGiftCardOrEWalletBalance()"/>
            </li>
        </xpath>
        <xpath expr="//OrderWidget/t[@t-set-slot='details']" position="inside">
            <t t-foreach="pos.get_order()?.getLoyaltyPoints() or []" t-as="_loyaltyStat" t-key="_loyaltyStat.couponId">
                <div t-if="_loyaltyStat.points.won || _loyaltyStat.points.spent" class="w-100 border-top mt-auto fw-bolder d-flex flex-column px-3 py-2 bg-200">
                    <div t-esc="_loyaltyStat.points.name" class="loyalty-points-title text-center mb-1" />
                    <div class="d-flex justify-content-around gap-2 mt-1">
                        <div t-if='_loyaltyStat.points.balance' class="loyalty-points-balance d-flex flex-column align-items-center justify-content-center flex-grow-1 rounded bg-300 px-3 py-2" >
                            <span class="text-muted">Points Balance</span>
                            <span class='value'><t t-esc='_loyaltyStat.points.balance'/></span>
                        </div>
                        <div t-if='_loyaltyStat.points.won' class="loyalty-points-won d-flex flex-column align-items-center justify-content-center flex-grow-1 rounded bg-300 px-3 py-2" >
                            <span class="text-muted">Points Won</span>
                            <span class='value text-success '>+<t t-esc='_loyaltyStat.points.won'/></span>
                        </div>
                        <div t-if='_loyaltyStat.points.spent' class="loyalty-points-spent d-flex flex-column align-items-center justify-content-center flex-grow-1 rounded bg-300 px-3 py-2" >
                            <span class="text-muted">Points Spent</span>
                            <span class='value text-danger'>-<t t-esc='_loyaltyStat.points.spent'/></span>
                        </div>
                        <div class="loyalty-points-total d-flex flex-column align-items-center justify-content-center flex-grow-1 rounded bg-300 px-3 py-2">
                            <span class="text-muted">New Total</span>
                            <span class='value text-primary'><t t-esc='_loyaltyStat.points.total'/></span>
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
/** @odoo-module **/

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
        this.notification = useService("pos_notification");
    },
    _onUpdateSelectedOrderline() {
        const order = this.getSelectedOrder();
        if (!order) {
            return this.numberBuffer.reset();
        }
        const selectedOrderlineId = this.getSelectedOrderlineId();
        const orderline = order.orderlines.find((line) => line.id == selectedOrderlineId);
        if (orderline && this._isEWalletGiftCard(orderline)) {
            this._showNotAllowedRefundNotification();
            return this.numberBuffer.reset();
        }
        return super._onUpdateSelectedOrderline(...arguments);
    },
    _prepareAutoRefundOnOrder(order) {
        const selectedOrderlineId = this.getSelectedOrderlineId();
        const orderline = order.orderlines.find((line) => line.id == selectedOrderlineId);
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
        const linkedProgramIds = this.pos.productId2ProgramIds[orderline.product.id];
        if (linkedProgramIds) {
            return linkedProgramIds.length > 0;
        }
        if (orderline.is_reward_line) {
            const reward = this.pos.reward_by_id[orderline.reward_id];
            const program = reward && reward.program_id;
            if (program && ["gift_card", "ewallet"].includes(program.program_type)) {
                return true;
            }
        }
        return false;
    },
});

```

## File: static\src\overrides\models\loyalty.js

```javascript
/** @odoo-module **/

import { Order, Orderline } from "@point_of_sale/app/store/models";
import { Mutex } from "@web/core/utils/concurrency";
import { roundDecimals, roundPrecision } from "@web/core/utils/numbers";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { ConfirmPopup } from "@point_of_sale/app/utils/confirm_popup/confirm_popup";

const { DateTime } = luxon;
const mutex = new Mutex(); // Used for sequential cache updates
const updateRewardsMutex = new Mutex();

function _newRandomRewardCode() {
    return (Math.random() + 1).toString(36).substring(3);
}

let nextId = -1;

let pointsForProgramsCountedRules = {};

export class PosLoyaltyCard {
    /**
     * @param {string} code coupon code
     * @param {number} id id of loyalty.card, negative if it is cache local only
     * @param {number} program_id id of loyalty.program
     * @param {number} partner_id id of res.partner
     * @param {number} balance points on the coupon, not counting the order's changes
     * @param {string} expiration_date
     */
    constructor(code, id, program_id, partner_id, balance, expiration_date = false) {
        this.code = code;
        this.id = id || nextId--;
        this.program_id = program_id;
        this.partner_id = partner_id;
        this.balance = balance;
        this.expiration_date = expiration_date && new Date(expiration_date);
    }

    isExpired() {
        return this.expiration_date && this.expiration_date < new Date();
    }
}

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

patch(Orderline.prototype, {
    export_as_JSON() {
        const result = super.export_as_JSON(...arguments);
        result.is_reward_line = this.is_reward_line;
        result.reward_id = this.reward_id;
        result.reward_product_id = this.reward_product_id;
        result.coupon_id = this.coupon_id;
        result.reward_identifier_code = this.reward_identifier_code;
        result.points_cost = this.points_cost;
        result.giftBarcode = this.giftBarcode;
        result.giftCardId = this.giftCardId;
        result.eWalletGiftCardProgramId = this.eWalletGiftCardProgram
            ? this.eWalletGiftCardProgram.id
            : null;
        return result;
    },
    init_from_JSON(json) {
        if (json.is_reward_line) {
            this.is_reward_line = json.is_reward_line;
            this.reward_id = json.reward_id;
            this.reward_product_id = json.reward_product_id;
            // Since non existing coupon have a negative id, of which the counter is lost upon reloading
            //  we make sure that they are kept the same between after a reload between the order and the lines.
            this.coupon_id = this.order.oldCouponMapping[json.coupon_id] || json.coupon_id;
            this.reward_identifier_code = json.reward_identifier_code;
            this.points_cost = json.points_cost;
        }
        this.giftBarcode = json.giftBarcode;
        this.giftCardId = json.giftCardId;
        this.eWalletGiftCardProgram = this.pos.program_by_id[json.eWalletGiftCardProgramId];
        super.init_from_JSON(...arguments);
    },
    getEWalletGiftCardProgramType() {
        return this.eWalletGiftCardProgram && this.eWalletGiftCardProgram.program_type;
    },
    ignoreLoyaltyPoints({ program }) {
        return (
            ["gift_card", "ewallet"].includes(program.program_type) &&
            this.eWalletGiftCardProgram &&
            this.eWalletGiftCardProgram.id !== program.id
        );
    },
});

patch(Order.prototype, {
    setup() {
        super.setup(...arguments);
        this._initializePrograms({});
        // Always start with invalid coupons so that coupon for this
        // order is properly assigned. @see _checkMissingCoupons
        this.invalidCoupons = true;
    },

    /** @override */
    getEmailItems() {
        return super
            .getEmailItems(...arguments)
            .concat(this.has_pdf_gift_card ? [_t("the gift cards")] : []);
    },

    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        json.disabledRewards = [...this.disabledRewards];
        json.codeActivatedProgramRules = this.codeActivatedProgramRules;
        json.codeActivatedCoupons = this.codeActivatedCoupons;
        json.couponPointChanges = this.couponPointChanges;
        return json;
    },
    init_from_JSON(json) {
        this.couponPointChanges = json.couponPointChanges;
        // Remapping of coupon_id for both couponPointChanges and Orderline.coupon_id
        this.oldCouponMapping = {};
        if (this.couponPointChanges) {
            for (const [key, pe] of Object.entries(this.couponPointChanges)) {
                if (!this.pos.program_by_id[pe.program_id]) {
                    // Remove points changes for programs that are not available anymore.
                    delete this.couponPointChanges[key];
                    continue;
                }
                if (pe.coupon_id > 0) {
                    continue;
                }
                const newId = nextId--;
                this.oldCouponMapping[pe.coupon_id] = newId;
                pe.coupon_id = newId;
                this.couponPointChanges[newId] = pe;
                delete this.couponPointChanges[key];
            }
        }
        super.init_from_JSON(...arguments);
        delete this.oldCouponMapping;
        this.disabledRewards = new Set(json.disabledRewards);
        this.codeActivatedProgramRules = json.codeActivatedProgramRules;
        this.codeActivatedCoupons = json.codeActivatedCoupons;
    },
    async pay() {
        const eWalletLine = this.get_orderlines().find(
            (line) => line.getEWalletGiftCardProgramType() === "ewallet"
        );
        if (eWalletLine && !this.get_partner()) {
            const { confirmed } = await this.env.services.popup.add(ConfirmPopup, {
                title: _t("Customer needed"),
                body: _t("eWallet requires a customer to be selected"),
            });
            if (confirmed) {
                const { confirmed, payload: newPartner } =
                    await this.env.services.pos.showTempScreen("PartnerListScreen", {
                        partner: null,
                    });
                if (confirmed) {
                    this.set_partner(newPartner);
                }
            }
        } else {
            return super.pay(...arguments);
        }
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
        if (this.couponPointChanges && oldPartner !== this.get_partner()) {
            // Remove couponPointChanges for cards in is_nominative programs.
            // This makes sure that counting of points on loyalty and ewallet programs is updated after partner changes.
            const loyaltyProgramIds = new Set(
                this.pos.programs
                    .filter((program) => program.is_nominative)
                    .map((program) => program.id)
            );
            for (const [key, pointChange] of Object.entries(this.couponPointChanges)) {
                if (loyaltyProgramIds.has(pointChange.program_id)) {
                    delete this.couponPointChanges[key];
                }
            }
            this._updateRewards();
        }
    },
    wait_for_push_order() {
        return (
            Object.keys(this.couponPointChanges || {}).length > 0 ||
            this._get_reward_lines().length ||
            super.wait_for_push_order(...arguments)
        );
    },
    /**
     * Add additional information for our ticket, such as new coupons and loyalty point gains.
     *
     * @override
     */
    export_for_printing() {
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
        const giftCardPrograms = this.pos.programs.filter((p) => p.program_type === "gift_card");
        for (const program of giftCardPrograms) {
            const giftCardProductId = [...program.rules[0].valid_product_ids][0];
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
        if (this.orderlines) {
            return this.orderlines.filter((line) => line.is_reward_line);
        }
        return this.orderlines;
    },
    _get_regular_order_lines() {
        if (this.orderlines) {
            return this.orderlines.filter(
                (line) => !line.is_reward_line && !line.refunded_orderline_id
            );
        }
        return this.orderlines;
    },
    get_last_orderline() {
        const orderLines = this.orderlines.filter((line) => !line.is_reward_line);
        return orderLines[orderLines.length - 1];
    },
    set_pricelist(pricelist) {
        const oldPricelist = this.pricelist;
        super.set_pricelist(...arguments);
        if (this.couponPointChanges && oldPricelist !== pricelist) {
            // Remove couponPointChanges for cards in no longer available programs.
            // This makes sure that counting of points on loyalty and ewallet programs is updated after pricelist changes.
            const loyaltyProgramIds = new Set(
                this.pos.programs
                    .filter(
                        (program) =>
                            program.pricelist_ids.length > 0 &&
                            (!pricelist || !program.pricelist_ids.includes(pricelist.id))
                    )
                    .map((program) => program.id)
            );
            for (const [key, pointChange] of Object.entries(this.couponPointChanges)) {
                if (loyaltyProgramIds.has(pointChange.program_id)) {
                    delete this.couponPointChanges[key];
                }
            }
        }
        this._updateRewards();
    },
    set_orderline_options(line, options) {
        super.set_orderline_options(...arguments);
        if (options && options.is_reward_line) {
            line.is_reward_line = options.is_reward_line;
            line.reward_id = options.reward_id;
            line.reward_product_id = options.reward_product_id;
            line.coupon_id = options.coupon_id;
            line.reward_identifier_code = options.reward_identifier_code;
            line.points_cost = options.points_cost;
            line.price_type = "automatic";
        }
        line.giftBarcode = options.giftBarcode;
        line.giftCardId = options.giftCardId;
        line.eWalletGiftCardProgram = options.eWalletGiftCardProgram;
    },
    _initializePrograms() {
        // When deleting a reward line, a popup will be displayed if the reward was automatic,
        //  if confirmed the reward is added to this list and will not be claimed automatically again.
        if (!this.disabledRewards) {
            this.disabledRewards = new Set();
        }
        // List of programs that require a code that are activated.
        if (!this.codeActivatedProgramRules) {
            this.codeActivatedProgramRules = [];
        }
        // List of coupons activated manually
        if (!this.codeActivatedCoupons) {
            this.codeActivatedCoupons = [];
        }
        // This field will hold the added points for each coupon.
        // Points lost are directly linked to the order lines.
        if (!this.couponPointChanges) {
            this.couponPointChanges = {};
        }
    },
    _resetPrograms() {
        this.disabledRewards = new Set();
        this.codeActivatedProgramRules = [];
        this.codeActivatedCoupons = [];
        this.couponPointChanges = {};
        this.orderlines.remove(this._get_reward_lines());
        this._updateRewards();
    },
    _updateRewards() {
        // Calls are not expected to take some time besides on the first load + when loyalty programs are made applicable
        if (this.pos.programs.length === 0) {
            return;
        }

        updateRewardsMutex.exec(() => {
            return this._updateLoyaltyPrograms().then(async () => {
                // Try auto claiming rewards
                const claimableRewards = this.getClaimableRewards(false, false, true);
                let changed = false;
                for (const { coupon_id, reward } of claimableRewards) {
                    if (
                        reward.program_id.rewards.length === 1 &&
                        !reward.program_id.is_nominative &&
                        (reward.reward_type !== "product" ||
                            (reward.reward_type == "product" && !reward.multi_product))
                    ) {
                        this._applyReward(reward, coupon_id);
                        changed = true;
                    }
                }
                // Rewards may impact the number of points gained
                if (changed) {
                    await this._updateLoyaltyPrograms();
                }
                this._updateRewardLines();
            });
        });
    },
    async _updateLoyaltyPrograms() {
        await this._checkMissingCoupons();
        await this._updatePrograms();
    },
    /**
     * Checks that all 'existing' coupons are in our cache, and if not load/update them.
     */
    async _checkMissingCoupons() {
        // This function must stay sequential to avoid potential concurrency errors.
        await mutex.exec(async () => {
            if (!this.invalidCoupons) {
                return;
            }
            this.invalidCoupons = false;
            const allCoupons = [];
            for (const pe of Object.values(this.couponPointChanges)) {
                if (pe.coupon_id > 0) {
                    allCoupons.push(pe.coupon_id);
                }
            }
            allCoupons.push(...this.codeActivatedCoupons.map((coupon) => coupon.id));
            const couponsToFetch = allCoupons.filter((elem) => !this.pos.couponCache[elem]);
            if (couponsToFetch.length) {
                await this.pos.fetchCoupons([["id", "in", couponsToFetch]], couponsToFetch.length);
                // Remove coupons that could not be loaded from the db
                this.codeActivatedCoupons = this.codeActivatedCoupons.filter(
                    (coupon) => this.pos.couponCache[coupon.id]
                );
                this.couponPointChanges = Object.fromEntries(
                    Object.entries(this.couponPointChanges).filter(
                        ([k, pe]) => this.pos.couponCache[pe.coupon_id]
                    )
                );
            }
        });
    },
    /**
     * Refreshes the currently applied rewards, if they are not applicable anymore they are removed.
     */
    _updateRewardLines() {
        if (!this.orderlines.length) {
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
                reward: this.pos.reward_by_id[line.reward_id],
                coupon_id: line.coupon_id,
                args: {
                    product: line.reward_product_id,
                    price: line.price,
                    quantity: line.quantity,
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
            this.orderlines.remove(line);
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
                !this.codeActivatedCoupons.find(
                    (coupon) => coupon.id === claimedReward.coupon_id
                ) &&
                !this.couponPointChanges[claimedReward.coupon_id]
            ) {
                continue;
            }
            this._applyReward(claimedReward.reward, claimedReward.coupon_id, claimedReward.args);
        }
    },
    /**
     * Update our couponPointChanges, meaning the points/coupons each program give etc.
     */
    async _updatePrograms() {
        const changesPerProgram = {};
        const programsToCheck = new Set();
        // By default include all programs that are considered 'applicable'
        for (const program of this.pos.programs) {
            if (this._programIsApplicable(program)) {
                programsToCheck.add(program.id);
            }
        }
        for (const pe of Object.values(this.couponPointChanges)) {
            if (!changesPerProgram[pe.program_id]) {
                changesPerProgram[pe.program_id] = [];
                programsToCheck.add(pe.program_id);
            }
            changesPerProgram[pe.program_id].push(pe);
        }
        for (const coupon of this.codeActivatedCoupons) {
            programsToCheck.add(coupon.program_id);
        }
        const programs = [...programsToCheck].map((programId) => this.pos.program_by_id[programId]);
        const pointsAddedPerProgram = this.pointsForPrograms(programs);
        for (const program of this.pos.programs) {
            // Future programs may split their points per unit paid (gift cards for example), consider a non applicable program to give no points
            const pointsAdded = this._programIsApplicable(program)
                ? pointsAddedPerProgram[program.id]
                : [];
            // For programs that apply to both (loyalty) we always add a change of 0 points, if there is none, since it makes it easier to
            //  track for claimable rewards, and makes sure to load the partner's loyalty card.
            if (program.is_nominative && !pointsAdded.length && this.get_partner()) {
                pointsAdded.push({ points: 0 });
            }
            const oldChanges = changesPerProgram[program.id] || [];
            // Update point changes for those that exist
            for (let idx = 0; idx < Math.min(pointsAdded.length, oldChanges.length); idx++) {
                Object.assign(oldChanges[idx], pointsAdded[idx]);
            }
            if (pointsAdded.length < oldChanges.length) {
                const removedIds = oldChanges.map((pe) => pe.coupon_id);
                this.couponPointChanges = Object.fromEntries(
                    Object.entries(this.couponPointChanges).filter(([k, pe]) => {
                        return !removedIds.includes(pe.coupon_id);
                    })
                );
            } else if (pointsAdded.length > oldChanges.length) {
                for (const pa of pointsAdded.splice(oldChanges.length)) {
                    const coupon = await this._couponForProgram(program);
                    this.couponPointChanges[coupon.id] = {
                        points: pa.points,
                        program_id: program.id,
                        coupon_id: coupon.id,
                        barcode: pa.barcode,
                        appliedRules: pointsForProgramsCountedRules[program.id],
                        giftCardId: pa.giftCardId
                    };
                }
            }
        }
        // Also remove coupons from codeActivatedCoupons if their program applies_on current orders and the program does not give any points
        this.codeActivatedCoupons = this.codeActivatedCoupons.filter((coupon) => {
            const program = this.pos.program_by_id[coupon.program_id];
            if (
                program.applies_on === "current" &&
                pointsAddedPerProgram[program.id].length === 0
            ) {
                return false;
            }
            return true;
        });
    },
    /**
     * @typedef {{ won: number, spend: number, total: number, balance: number, name: string}} LoyaltyPoints
     * @typedef {{ couponId: number, program: object, points: LoyaltyPoints}} LoyaltyStat
     * @returns {Array<LoyaltyStat>}
     */
    getLoyaltyPoints() {
        // map: couponId -> LoyaltyPoints
        const loyaltyPoints = {};
        for (const pointChange of Object.values(this.couponPointChanges)) {
            const { coupon_id, points, program_id } = pointChange;
            const program = this.pos.program_by_id[program_id];
            if (program.program_type !== "loyalty") {
                // Not a loyalty program, skip
                continue;
            }
            const loyaltyCard = this.pos.couponCache[coupon_id] || /* or new card */ {
                id: coupon_id,
                balance: 0,
            };
            let [won, spent, total] = [0, 0, 0];
            const balance = loyaltyCard.balance;
            won += points - this._getPointsCorrection(program);
            if (coupon_id !== 0) {
                for (const line of this._get_reward_lines()) {
                    if (line.coupon_id === coupon_id) {
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
        const rewardLines = this.orderlines.filter((line) => line.is_reward_line);
        let res = 0;
        for (const rule of program.rules) {
            for (const line of rewardLines) {
                const reward = this.pos.reward_by_id[line.reward_id];
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
        if (rule.reward_point_mode === 'order') {
            return false;
        }

        // Check if the reward line is part of the rule
        if (!(rule.any_product || rule.valid_product_ids.has(line.reward_product_id))) {
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
    _getRealCouponPoints(coupon_id) {
        let points = 0;
        const dbCoupon = this.pos.couponCache[coupon_id];
        if (dbCoupon) {
            points += dbCoupon.balance;
        }
        Object.values(this.couponPointChanges).some((pe) => {
            if (pe.coupon_id === coupon_id) {
                if (this.pos.program_by_id[pe.program_id].applies_on !== "future") {
                    points += pe.points;
                }
                // couponPointChanges is not supposed to have a coupon multiple times
                return true;
            }
            return false;
        });
        for (const line of this.get_orderlines()) {
            if (line.is_reward_line && line.coupon_id === coupon_id) {
                points -= line.points_cost;
            }
        }
        return points;
    },
    /**
     * Depending on the program type returns a new (local) instance of coupon or tries to retrieve the coupon in case of loyalty cards.
     * Existing coupons are put in a cache which is also used to fetch the coupons.
     *
     * @param {object} program
     */
    async _couponForProgram(program) {
        if (program.is_nominative) {
            return await this.pos.fetchLoyaltyCard(program.id, this.get_partner().id);
        }
        // This type of coupons don't need to really exist up until validating the order, so no need to cache
        return new PosLoyaltyCard(null, null, program.id, (this.get_partner() || { id: -1 }).id, 0);
    },
    _programIsApplicable(program) {
        if (
            program.trigger === "auto" &&
            !program.rules.find(
                (rule) => rule.mode === "auto" || this.codeActivatedProgramRules.includes(rule.id)
            )
        ) {
            return false;
        }
        if (
            program.trigger === "with_code" &&
            !program.rules.find((rule) => this.codeActivatedProgramRules.includes(rule.id))
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
            (!this.pricelist || !program.pricelist_ids.includes(this.pricelist.id))
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
        const orderLines = this.get_orderlines().filter((line) => !line.refunded_orderline_id);
        const linesPerRule = {};
        for (const line of orderLines) {
            const reward = line.reward_id ? this.pos.reward_by_id[line.reward_id] : undefined;
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
                for (const rule of program.rules) {
                    // Skip lines to which the rule doesn't apply.
                    if (rule.any_product || rule.valid_product_ids.has(line.get_product().id)) {
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
            for (const rule of program.rules) {
                if (
                    rule.mode === "with_code" &&
                    !this.codeActivatedProgramRules.includes(rule.id)
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
                            (rule.any_product ||
                                rule.valid_product_ids.has(line.get_product().id))) ||
                            (line.reward_product_id &&
                                (rule.any_product ||
                                    rule.valid_product_ids.has(line.reward_product_id)))) &&
                        !line.ignoreLoyaltyPoints({ program })
                    ) {
                        // We only count reward products from the same program to avoid unwanted feedback loops
                        if (line.is_reward_line) {
                            const reward = this.pos.reward_by_id[line.reward_id];
                            if ((program.id === reward.program_id.id) || ['gift_card', 'ewallet'].includes(reward.program_id.program_type)) {
                                continue;
                            }
                        }
                        const lineQty = line.reward_product_id
                            ? -line.get_quantity()
                            : line.get_quantity();
                        if (qtyPerProduct[line.reward_product_id || line.get_product().id]) {
                            qtyPerProduct[line.reward_product_id || line.get_product().id] +=
                                lineQty;
                        } else {
                            qtyPerProduct[line.reward_product_id || line.get_product().id] =
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
                            ...Array.apply(null, Array(totalProductQty)).map((_) => {
                                return { points: rule.reward_point_amount };
                            })
                        );
                    } else if (rule.reward_point_mode === "money") {
                        for (const line of orderLines) {
                            if (
                                line.is_reward_line ||
                                !rule.valid_product_ids.has(line.get_product().id) ||
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
                                        if (line.giftBarcode && line.get_quantity() == 1) {
                                            return {
                                                points: pointsPerUnit,
                                                barcode: line.giftBarcode,
                                                giftCardId: line.giftCardId,
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
            (line) => line.reward_id && this.pos.reward_by_id[line.reward_id].is_global_discount
        );
    },
    /**
     * Returns the number of product items in the order based on the given rule.
     * @param {*} rule
     */
    _computeNItems(rule) {
        return this._get_regular_order_lines().reduce((nItems, line) => {
            let increment = 0;
            if (rule.any_product || rule.valid_product_ids.has(line.product.id)) {
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
        for (const rule of couponProgram.rules) {
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
        const allCouponPrograms = Object.values(this.couponPointChanges)
            .map((pe) => {
                return {
                    program_id: pe.program_id,
                    coupon_id: pe.coupon_id,
                };
            })
            .concat(
                this.codeActivatedCoupons.map((coupon) => {
                    return {
                        program_id: coupon.program_id,
                        coupon_id: coupon.id,
                    };
                })
            );
        const result = [];
        const totalWithTax = this.get_total_with_tax();
        const totalWithoutTax = this.get_total_without_tax();
        const totalIsZero = totalWithTax === 0;
        const globalDiscountLines = this._getGlobalDiscountLines();
        const globalDiscountPercent = globalDiscountLines.length
            ? this.pos.reward_by_id[globalDiscountLines[0].reward_id].discount
            : 0;
        for (const couponProgram of allCouponPrograms) {
            const program = this.pos.program_by_id[couponProgram.program_id];
            if (
                program.pricelist_ids.length > 0 &&
                (!this.pricelist || !program.pricelist_ids.includes(this.pricelist.id))
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
            for (const reward of program.rewards) {
                if (points < reward.required_points) {
                    continue;
                }
                // Skip if the reward program is of type 'coupons' and there is already an reward orderline linked to the current reward to avoid multiple reward apply
                if ((reward.program_id.program_type === 'coupons' && this.orderlines.find(((rewardline) => rewardline.reward_id === reward.id)))) {
                    continue;
                }
                if (auto && this.disabledRewards.has(reward.id)) {
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
                        const product = this.pos.db.get_product_by_id(reward.reward_product_ids[0]);
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
     * Applies a reward to the order, `_updateRewards` is expected to be called right after.
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
                if (
                    rewardId != reward.id &&
                    this.pos.reward_by_id[rewardId].discount >= reward.discount
                ) {
                    return _t("A better global discount is already applied.");
                } else if (rewardId != rewardId.id) {
                    for (const line of globalDiscountLines) {
                        this.orderlines.remove(line);
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
            this.orderlines.add(this._createLineFromVals(rewardLine));
        }
        return true;
    },
    _createLineFromVals(vals) {
        vals["lst_price"] = vals["price"];
        const line = new Orderline(
            { env: this.env },
            { pos: this.pos, order: this, product: vals["product"] }
        );
        this.fix_tax_included_price(line);
        this.set_orderline_options(line, vals);
        return line;
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
            const taxKey = ['ewallet', 'gift_card'].includes(reward.program_id.program_type)
                ? line.get_taxes().map((t) => t.id)
                : line.get_taxes().filter((t) => t.amount_type !== 'fixed').map((t) => t.id);
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
        const filtered_lines = this.get_orderlines().filter((line) => !line.comboParent && !line.reward_id && line.get_quantity);
        return filtered_lines.toSorted((lineA, lineB) => 
            lineA.getComboTotalPrice() - lineB.getComboTotalPrice()
       )[0]
    },
    /**
     * @returns the discountable and discountable per tax for this discount on cheapest reward.
     */
    _getDiscountableOnCheapest(reward) {
        const cheapestLine = this._getCheapestLine();
        if (!cheapestLine) {
            return { discountable: 0, discountablePerTax: {} };
        }
        const taxKey = cheapestLine.get_taxes().map((t) => t.id);
        return {
            discountable: cheapestLine.getComboTotalPriceWithoutTax(),
            discountablePerTax: Object.fromEntries([[taxKey, cheapestLine.getComboTotalPriceWithoutTax()]]),
        };
    },
    /**
     * @param {loyalty.reward} reward
     * @returns all lines to which the reward applies.
     */
    _getSpecificDiscountableLines(reward) {
        const discountableLines = [];
        const applicableProducts = reward.all_discount_product_ids;
        for (const line of this.get_orderlines()) {
            if (!line.get_quantity()) {
                continue;
            }
            if (
                applicableProducts.has(line.get_product().id) ||
                applicableProducts.has(line.reward_product_id)
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
        const applicableProducts = reward.all_discount_product_ids;
        const linesToDiscount = [];
        const discountLinesPerReward = {};
        const orderLines = this.get_orderlines();
        const orderProducts = orderLines.map((line) => line.product.id);
        const remainingAmountPerLine = {};
        for (const line of orderLines) {
            if (!line.get_quantity() || !line.price) {
                continue;
            }
            const product_id = line.comboParent?.product.id || line.get_product().id;
            remainingAmountPerLine[line.cid] = line.get_price_with_tax();
            if (
                applicableProducts.has(product_id) ||
                (line.reward_product_id && applicableProducts.has(line.reward_product_id))
            ) {
                linesToDiscount.push(line);
            } else if (line.reward_id) {
                const lineReward = this.pos.reward_by_id[line.reward_id];
                if (lineReward.id === reward.id ||
                    (
                        orderProducts.some(product =>
                            lineReward.all_discount_product_ids.has(product) &&
                            applicableProducts.has(product)
                        ) &&
                        lineReward.reward_type === 'discount' &&
                        lineReward.discount_mode != 'percent'
                    )
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
            const lineReward = this.pos.reward_by_id[lines[0].reward_id];
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
                        remainingAmountPerLine[line.cid] *= 1 - discount / line.get_quantity();
                    } else {
                        remainingAmountPerLine[line.cid] *= 1 - discount;
                    }
                }
            }
        }

        let discountable = 0;
        const discountablePerTax = {};
        for (const line of linesToDiscount) {
            discountable += remainingAmountPerLine[line.cid];
            const taxKey = line.get_taxes().map((t) => t.id);
            if (!discountablePerTax[taxKey]) {
                discountablePerTax[taxKey] = 0;
            }
            discountablePerTax[taxKey] +=
                line.get_base_price() *
                (remainingAmountPerLine[line.cid] / line.get_price_with_tax());
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
            const points = (["ewallet", "gift_card"].includes(reward.program_id.program_type)) ?
                this._getRealCouponPoints(coupon_id) :
                Math.floor(this._getRealCouponPoints(coupon_id) / reward.required_points) * reward.required_points;
            maxDiscount = Math.min(
                maxDiscount,
                reward.discount * points
            );
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
            const taxes_to_apply = discountProduct.taxes_id.map((id) => {
                return { ...this.pos.taxes_by_id[id], price_include: true };
            });
            const tax_res = this.pos.compute_all(
                taxes_to_apply,
                -Math.min(maxDiscount, discountable),
                1,
                this.pos.currency.rounding
            );
            let new_price = tax_res["total_excluded"];
            new_price += tax_res.taxes
                .filter((tax) => this.pos.taxes_by_id[tax.id].price_include)
                .reduce((sum, tax) => (sum += tax.amount), 0);
            return [
                {
                    product: discountProduct,
                    price: new_price,
                    quantity: 1,
                    reward_id: reward.id,
                    is_reward_line: true,
                    coupon_id: coupon_id,
                    points_cost: pointCost,
                    reward_identifier_code: rewardCode,
                    merge: false,
                    taxIds: discountProduct.taxes_id,
                },
            ];
        }
        const discountFactor = discountable ? Math.min(1, maxDiscount / discountable) : 1;
        const result = Object.entries(discountablePerTax).reduce((lst, entry) => {
            // Ignore 0 price lines
            if (!entry[1]) {
                return lst;
            }
            const taxIds = entry[0] === "" ? [] : entry[0].split(",").map((str) => parseInt(str));
            lst.push({
                product: discountProduct,
                price: -(entry[1] * discountFactor),
                quantity: 1,
                reward_id: reward.id,
                is_reward_line: true,
                coupon_id: coupon_id,
                points_cost: 0,
                reward_identifier_code: rewardCode,
                tax_ids: taxIds,
                merge: false,
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
            reward.program_id.rules.filter(
                (rule) => rule.any_product || rule.valid_product_ids.has(product.id)
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
            if (reward.reward_product_ids.includes(product.id) && reward.reward_product_ids.includes(line.product.id)) {
                if (this._get_reward_lines() == 0) {
                    if (line.get_product().id === product.id) {
                        available += line.get_quantity();
                    }
                } else {
                    available += line.get_quantity();
                }
            } else if (reward.reward_product_ids.includes(line.reward_product_id)) {
                if (line.reward_id == reward.id) {
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
                const appliedRulesIds = this.couponPointChanges[coupon_id].appliedRules;
                const appliedRules =
                    appliedRulesIds !== undefined
                        ? reward.program_id.rules.filter((rule) =>
                              appliedRulesIds.includes(rule.id)
                          )
                        : reward.program_id.rules;
                let factor = 0;
                let orderPoints = 0;
                for (const rule of appliedRules) {
                    if (rule.any_product || rule.valid_product_ids.has(product.id)) {
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
                    (line) => line.reward_product_id === product.id
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
        const product = this.pos.db.get_product_by_id(
            args["product"] || reward.reward_product_ids[0]
        );
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
                product: reward.discount_line_product_id,
                price: -roundDecimals(
                    product.get_price(this.pricelist, freeQuantity),
                    this.pos.currency.decimal_places
                ),
                tax_ids: product.taxes_id,
                quantity: freeQuantity,
                reward_id: reward.id,
                is_reward_line: true,
                reward_product_id: product.id,
                coupon_id: args["coupon_id"],
                points_cost: cost,
                reward_identifier_code: _newRandomRewardCode(),
                merge: false,
            },
        ];
    },
    /**
     * Full routine for activating a code for the order.
     * If only one more reward is claimable after activating the code, that reward is claimed
     *  directly, to avoid more steps than necessary.
     * If more rewards are claimable, the employee will have to manually select the reward
     *  in the reward selection menu.
     *
     * @param {String} code
     * @returns true if everything went right, error message if not.
     */
    async _activateCode(code) {
        const rule = this.pos.rules.find((rule) => {
            return rule.mode === "with_code" && (rule.promo_barcode === code || rule.code === code);
        });
        let claimableRewards = null;
        let coupon = null;
        if (rule) {
            if (
                rule.program_id.date_from &&
                this.date_order < rule.program_id.date_from.startOf("day")
            ) {
                return _t("That promo code program is not yet valid.");
            }
            if (rule.program_id.date_to && this.date_order > rule.program_id.date_to.endOf("day")) {
                return _t("That promo code program is expired.");
            }
            const program_pricelists = rule.program_id.pricelist_ids;
            if (
                program_pricelists.length > 0 &&
                (!this.pricelist || !program_pricelists.includes(this.pricelist.id))
            ) {
                return _t("That promo code program requires a specific pricelist.");
            }
            if (this.codeActivatedProgramRules.includes(rule.id)) {
                return _t("That promo code program has already been activated.");
            }
            this.codeActivatedProgramRules.push(rule.id);
            await this._updateLoyaltyPrograms();
            claimableRewards = this.getClaimableRewards(false, rule.program_id.id);
        } else {
            if (this.codeActivatedCoupons.find((coupon) => coupon.code === code)) {
                return _t("That coupon code has already been scanned and activated.");
            }
            const customerId = this.get_partner() ? this.get_partner().id : false;
            const { successful, payload } = await this.env.services.orm.call(
                "pos.config",
                "use_coupon_code",
                [
                    [this.pos.config.id],
                    code,
                    this.date_order,
                    customerId,
                    this.pricelist ? this.pricelist.id : false,
                ]
            );
            if (successful) {
                // Allow rejecting a gift card that is not yet paid.
                const program = this.pos.program_by_id[payload.program_id];
                if (program && program.program_type === "gift_card" && !payload.has_source_order) {
                    const { confirmed } = await this.env.services.popup.add(ConfirmPopup, {
                        title: _t("Unpaid gift card"),
                        body: _t(
                            "This gift card is not linked to any order. Do you really want to apply its reward?"
                        ),
                    });
                    if (!confirmed) {
                        return _t("Unpaid gift card rejected.");
                    }
                }
                coupon = new PosLoyaltyCard(
                    code,
                    payload.coupon_id,
                    payload.program_id,
                    payload.partner_id,
                    payload.points,
                    payload.expiration_date
                );
                this.pos.couponCache[coupon.id] = coupon;
                this.codeActivatedCoupons.push(coupon);
                await this._updateLoyaltyPrograms();
                claimableRewards = this.getClaimableRewards(coupon.id);
            } else {
                return payload.error_message;
            }
        }
        if (claimableRewards && claimableRewards.length === 1) {
            if (
                claimableRewards[0].reward.reward_type !== "product" ||
                !claimableRewards[0].reward.multi_product
            ) {
                this._applyReward(claimableRewards[0].reward, claimableRewards[0].coupon_id);
                this._updateRewards();
            }
        }
        if (!rule && this.orderlines.length === 0 && coupon) {
            return _t(
                "Gift Card: %s\nBalance: %s",
                code,
                this.env.utils.formatCurrency(coupon.balance)
            );
        }
        return true;
    },
    async activateCode(code) {
        const res = await this._activateCode(code);
        if (res !== true) {
            this.env.services.pos_notification.add(res, 5000);
        }
    },
    isProgramsResettable() {
        const array = [
            this.disabledRewards,
            this.codeActivatedProgramRules,
            this.codeActivatedCoupons,
            this.couponPointChanges,
            this._get_reward_lines(),
        ];
        return array.some((elem) => elem.length > 0);
    },
    removeOrderline(lineToRemove) {
        if (lineToRemove.is_reward_line) {
            // Remove any line that is part of that same reward aswell.
            const linesToRemove = this.get_orderlines().filter((line) => {
                return (
                    line.reward_id === lineToRemove.reward_id &&
                    line.coupon_id === lineToRemove.coupon_id &&
                    line.reward_identifier_code === lineToRemove.reward_identifier_code
                );
            });
            for (const line of linesToRemove) {
                this._unlinkOrderline(line);
            }
            return true;
        } else {
            return super.removeOrderline(lineToRemove);
        }
    },
});

```

## File: static\src\overrides\models\orderline.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { Orderline } from "@point_of_sale/app/store/models";

patch(Orderline.prototype, {
    /**
     * @returns {boolean}
     */
    isGiftCardOrEWalletReward() {
        const coupon = this.pos.couponCache[this.coupon_id];
        if (!coupon || !this.is_reward_line) {
            return false;
        }
        const program = this.pos.program_by_id[coupon.program_id];
        return ["ewallet", "gift_card"].includes(program.program_type);
    },
    /**
     * @returns {string}
     */
    getGiftCardOrEWalletBalance() {
        const coupon = this.pos.couponCache[this.coupon_id];
        return this.env.utils.formatCurrency(coupon?.balance || 0);
    },
    /**
     * override
     */
    getDisplayClasses() {
        return {
            ...super.getDisplayClasses(),
            "fst-italic":
                this.pos.mainScreen.component.name === "ProductScreen" && this.is_reward_line,
        };
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { _t } from "@web/core/l10n/translation";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { TextInputPopup } from "@point_of_sale/app/utils/input_popups/text_input_popup";
import { Domain, InvalidDomainError } from "@web/core/domain";
import { PosLoyaltyCard } from "@pos_loyalty/overrides/models/loyalty";

const { DateTime } = luxon;
const COUPON_CACHE_MAX_SIZE = 4096; // Maximum coupon cache size, prevents long run memory issues and (to some extent) invalid data

patch(PosStore.prototype, {
    async addProductFromUi(product, options) {
        const order = this.get_order();
        const linkedProgramIds = this.productId2ProgramIds[product.id] || [];
        const linkedPrograms = linkedProgramIds.map((id) => this.program_by_id[id]);
        let selectedProgram = null;
        if (linkedPrograms.length > 1) {
            const { confirmed, payload: program } = await this.popup.add(SelectionPopup, {
                title: _t("Select program"),
                list: linkedPrograms.map((program) => ({
                    id: program.id,
                    item: program,
                    label: program.name,
                })),
            });
            if (confirmed) {
                selectedProgram = program;
            } else {
                // Do nothing here if the selection is cancelled.
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
            options.price = -orderTotal;
        }
        if (selectedProgram && selectedProgram.program_type == "gift_card") {
            const shouldProceed = await this._setupGiftCardOptions(selectedProgram, options);
            if (!shouldProceed) {
                return;
            }
        } else if (selectedProgram && selectedProgram.program_type == "ewallet") {
            const shouldProceed = await this.setupEWalletOptions(selectedProgram, options);
            if (!shouldProceed) {
                return;
            }
        }
        const potentialRewards = this.getPotentialFreeProductRewards();
        const rewardsToApply = [];
        for (const reward of potentialRewards) {
            for (const reward_product_id of reward.reward.reward_product_ids) {
                if (reward_product_id == product.id) {
                    rewardsToApply.push(reward);
                }
            }
        }
        await super.addProductFromUi(product, options);
        await order._updatePrograms();
        if (rewardsToApply.length == 1) {
            const reward = rewardsToApply[0];
            order._applyReward(reward.reward, reward.coupon_id, { product: product.id });
        }

        order._updateRewards();
        return options;
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

        // If gift card program setting is 'scan_use', ask for the code.
        if (this.config.gift_card_settings == "scan_use") {
            const { confirmed, payload: code } = await this.env.services.popup.add(TextInputPopup, {
                title: _t("Generate a Gift Card"),
                startingValue: "",
                placeholder: _t("Enter the gift card code"),
            });
            if (!confirmed) {
                return false;
            }
            const trimmedCode = code.trim();
            let nomenclatureRules = this.barcodeReader.parser.nomenclature.rules;
            if (this.barcodeReader.fallbackParser) {
                nomenclatureRules = nomenclatureRules.concat(
                    this.barcodeReader.fallbackParser.nomenclature.rules
                );
            }
            const couponRules = nomenclatureRules.filter((rule) => rule.type === "coupon");
            const isValidCoupon = couponRules.some((rule) => {
                const patterns = rule.pattern.split("|");
                return patterns.some((pattern) => trimmedCode.startsWith(pattern));
            });
            if (isValidCoupon) {
                // check if the code exist in the database
                // if so, use its balance, otherwise, use the unit price of the gift card product
                const fetchedGiftCard = await this.orm.searchRead(
                    "loyalty.card",
                    [
                        ["code", "=", trimmedCode],
                        ["program_id", "=", program.id],
                    ],
                    ["points", "source_pos_order_id"]
                );
                // There should be maximum one gift card for a given code.
                const giftCard = fetchedGiftCard[0];
                if (giftCard && giftCard.source_pos_order_id) {
                    this.popup.add(ErrorPopup, {
                        title: _t("This gift card has already been sold"),
                        body: _t("You cannot sell a gift card that has already been sold."),
                    });
                    return false;
                }
                options.giftBarcode = trimmedCode;
                if (giftCard) {
                    // Use the balance of the gift card as the price of the orderline.
                    // NOTE: No need to convert the points to price because when opening a session,
                    // the gift card programs are made sure to have 1 point = 1 currency unit.
                    options.price = giftCard.points;
                    options.giftCardId = giftCard.id;
                }
            } else {
                this.env.services.pos_notification.add("Please enter a valid gift card code.");
                return false;
            }
        }
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
    getPotentialFreeProductRewards() {
        const order = this.get_order();
        const allCouponPrograms = Object.values(order.couponPointChanges)
            .map((pe) => {
                return {
                    program_id: pe.program_id,
                    coupon_id: pe.coupon_id,
                };
            })
            .concat(
                order.codeActivatedCoupons.map((coupon) => {
                    return {
                        program_id: coupon.program_id,
                        coupon_id: coupon.id,
                    };
                })
            );
        const result = [];
        for (const couponProgram of allCouponPrograms) {
            const program = this.program_by_id[couponProgram.program_id];
            if (
                program.pricelist_ids.length > 0 &&
                (!order.pricelist || !program.pricelist_ids.includes(order.pricelist.id))
            ) {
                continue;
            }

            const points = order._getRealCouponPoints(couponProgram.coupon_id);
            const hasLine = order.orderlines.filter((line) => !line.is_reward_line).length > 0;
            for (const reward of program.rewards.filter(
                (reward) => reward.reward_type == "product" && reward.reward_product_ids.length > 0
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
                    for (const productId of reward.reward_product_ids) {
                        const product = this.db.get_product_by_id(productId);
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
    async _processData(loadedData) {
        this.couponCache = {};
        this.partnerId2CouponIds = {};
        this.rewards = loadedData["loyalty.reward"] || [];

        for (const reward of this.rewards) {
            reward.all_discount_product_ids = new Set(reward.all_discount_product_ids);
        }

        this.fieldTypes = loadedData["field_types"];
        await super._processData(loadedData);
        this.productId2ProgramIds = loadedData["product_id_to_program_ids"];
        this.programs = loadedData["loyalty.program"] || []; //TODO: rename to `loyaltyPrograms` etc
        this.rules = loadedData["loyalty.rule"] || [];
        this._loadLoyaltyData();
    },

    _loadProductProduct(products) {
        super._loadProductProduct(...arguments);

        for (const reward of this.rewards) {
            this.compute_discount_product_ids(reward, products);
        }

        this.rewards = this.rewards.filter(Boolean);
    },

    compute_discount_product_ids(reward, products) {
        const reward_product_domain = JSON.parse(reward.reward_product_domain);
        if (!reward_product_domain) {
            return;
        }

        const domain = new Domain(reward_product_domain);

        try {
            products
                .filter((product) => domain.contains(product))
                .forEach((product) => reward.all_discount_product_ids.add(product.id));
        } catch (error) {
            if (!(error instanceof InvalidDomainError || error instanceof TypeError)) {
                throw error;
            }
            const index = this.rewards.indexOf(reward);
            if (index != -1) {
                this.env.services.popup.add(ErrorPopup, {
                    title: _t("A reward could not be loaded"),
                    body: _t(
                        'The reward "%s" contain an error in its domain, your domain must be compatible with the PoS client',
                        this.rewards[index].description
                    ),
                });
                this.rewards[index] = null;
            }
        }
    },

    _loadLoyaltyData() {
        this.program_by_id = {};
        this.reward_by_id = {};

        for (const program of this.programs) {
            this.program_by_id[program.id] = program;
            if (program.date_from) {
                program.date_from = DateTime.fromISO(program.date_from);
            }
            if (program.date_to) {
                program.date_to = DateTime.fromISO(program.date_to);
            }
            program.rules = [];
            program.rewards = [];
        }
        for (const rule of this.rules) {
            rule.valid_product_ids = new Set(rule.valid_product_ids);
            rule.program_id = this.program_by_id[rule.program_id[0]];
            rule.program_id.rules.push(rule);
        }
        for (const reward of this.rewards) {
            this.reward_by_id[reward.id] = reward;
            reward.program_id = this.program_by_id[reward.program_id[0]];
            reward.discount_line_product_id = this.db.get_product_by_id(
                reward.discount_line_product_id[0]
            );
            reward.all_discount_product_ids = new Set(reward.all_discount_product_ids);
            reward.program_id.rewards.push(reward);
        }
    },
    async load_server_data() {
        await super.load_server_data(...arguments);
        if (this.selectedOrder) {
            this.selectedOrder._updateRewards();
        }
    },
    set_order(order) {
        const result = super.set_order(...arguments);
        // FIXME - JCB: This is a temporary fix.
        // When an order is selected, it doesn't always contain the reward lines.
        // And the list of active programs are not always correct. This is because
        // of the use of DropPrevious in _updateRewards.
        if (order && !order.finalized) {
            order._updateRewards();
        }
        return result;
    },
    /**
     * Fetches `loyalty.card` records from the server and adds/updates them in our cache.
     *
     * @param {domain} domain For the search
     * @param {int} limit Default to 1
     */
    async fetchCoupons(domain, limit = 1) {
        const result = await this.env.services.orm.searchRead(
            "loyalty.card",
            domain,
            ["id", "points", "code", "partner_id", "program_id", "expiration_date"],
            { limit }
        );
        if (Object.keys(this.couponCache).length + result.length > COUPON_CACHE_MAX_SIZE) {
            this.couponCache = {};
            this.partnerId2CouponIds = {};
            // Make sure that the current order has no invalid data.
            if (this.selectedOrder) {
                this.selectedOrder.invalidCoupons = true;
            }
        }
        const couponList = [];
        for (const dbCoupon of result) {
            const coupon = new PosLoyaltyCard(
                dbCoupon.code,
                dbCoupon.id,
                dbCoupon.program_id[0],
                dbCoupon.partner_id[0],
                dbCoupon.points,
                dbCoupon.expiration_date
            );
            this.couponCache[coupon.id] = coupon;
            this.partnerId2CouponIds[coupon.partner_id] =
                this.partnerId2CouponIds[coupon.partner_id] || new Set();
            this.partnerId2CouponIds[coupon.partner_id].add(coupon.id);
            couponList.push(coupon);
        }
        return couponList;
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
        for (const coupon of Object.values(this.couponCache)) {
            if (coupon.partner_id === partnerId && coupon.program_id === programId) {
                return coupon;
            }
        }
        const fetchedCoupons = await this.fetchCoupons([
            ["partner_id", "=", partnerId],
            ["program_id", "=", programId],
        ]);
        const dbCoupon = fetchedCoupons.length > 0 ? fetchedCoupons[0] : null;
        return dbCoupon || new PosLoyaltyCard(null, null, programId, partnerId, 0);
    },
    getLoyaltyCards(partner) {
        const loyaltyCards = [];
        if (this.partnerId2CouponIds[partner.id]) {
            this.partnerId2CouponIds[partner.id].forEach((couponId) =>
                loyaltyCards.push(this.couponCache[couponId])
            );
        }
        return loyaltyCards;
    },
    addPartners(partners) {
        const result = super.addPartners(partners);
        // cache the loyalty cards of the partners
        for (const partner of partners) {
            for (const [couponId, { code, program_id, points }] of Object.entries(
                partner.loyalty_cards || {}
            )) {
                this.couponCache[couponId] = new PosLoyaltyCard(
                    code,
                    parseInt(couponId, 10),
                    program_id,
                    partner.id,
                    points
                );
                this.partnerId2CouponIds[partner.id] =
                    this.partnerId2CouponIds[partner.id] || new Set();
                this.partnerId2CouponIds[partner.id].add(couponId);
            }
        }
        return result;
    },
});

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
        <field name="name">loyalty.mail.view.tree.inherit.pos.loyalty</field>
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
        <field name="name">loyalty.program.view.tree.inherit.pos.loyalty</field>
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
                    <div class="mt16 o_light_label" id="pos_gift_card_settings" invisible="is_kiosk_mode">
                        <field name="pos_gift_card_settings" colspan="4" nolabel="1" widget="radio"/>
                    </div>
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

