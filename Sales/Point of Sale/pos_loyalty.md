# Odoo Module: pos_loyalty

Category: Sales/Point Of Sale

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
    ],
    'demo': [
        'data/pos_loyalty_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_loyalty/static/src/css/Loyalty.scss',
            'pos_loyalty/static/src/js/**/*',
            'pos_loyalty/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'pos_loyalty/static/src/tours/**/*',
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

     <!-- Gift Cards -->
    <record id="loyalty.gift_card_program" model="loyalty.program">
        <field name="pos_report_print_id" ref="loyalty.report_gift_card"/>
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
            [('coupon_id', 'in', self.ids)], ['id'], ['coupon_id'])
        count_per_coupon = {r['coupon_id'][0]: r['coupon_id_count'] for r in read_group_res}
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

    def action_view_pos_orders(self):
        self.ensure_one()
        pos_order_ids = list(unique(r['order_id'] for r in\
                self.env['pos.order.line'].search_read([('reward_id', 'in', self.reward_ids.ids)], fields=['order_id'])))
        return {
            'name': _("PoS Orders"),
            'view_mode': 'tree,form',
            'res_model': 'pos.order',
            'type': 'ir.actions.act_window',
            'domain': [('id', 'in', pos_order_ids)],
            'context': dict(self._context, create=False),
        }

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

    def use_coupon_code(self, code, creation_date, partner_id):
        self.ensure_one()
        # Points desc so that in coupon mode one could use a coupon multiple times
        coupon = self.env['loyalty.card'].search(
            [('program_id', 'in', self._get_program_ids().ids),
            '|', ('partner_id', 'in', (False, partner_id)), ('program_type', '=', 'gift_card'),
            ('code', '=', code)],
            order='points desc', limit=1)
        if not coupon or not coupon.program_id.active:
            return {
                'successful': False,
                'payload': {
                    'error_message': _('This coupon is invalid (%s).', code),
                },
            }
        check_date = fields.Date.from_string(creation_date[:11])
        if (coupon.expiration_date and coupon.expiration_date < check_date) or\
            (coupon.program_id.date_to and coupon.program_id.date_to < fields.Date.context_today(self)) or\
            (coupon.program_id.limit_usage and coupon.program_id.total_order_count >= coupon.program_id.max_usage):
            return {
                'successful': False,
                'payload': {
                    'error_message': _('This coupon is expired (%s).', code),
                },
            }
        if not coupon.program_id.reward_ids or not any(reward.required_points <= coupon.points for reward in coupon.program_id.reward_ids):
            return {
                'successful': False,
                'payload': {
                    'error_message': _('No reward can be claimed with this coupon.'),
                },
            }
        return {
            'successful': True,
            'payload': {
                'program_id': coupon.program_id.id,
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
        # Keys are stringified when using rpc
        coupon_data = {int(k): v for k, v in coupon_data.items()}

        self._check_existing_loyalty_cards(coupon_data)
        # Map negative id to newly created ids.
        coupon_new_id_map = {k: k for k in coupon_data.keys() if k > 0}

        # Create the coupons that were awarded by the order.
        coupons_to_create = {k: v for k, v in coupon_data.items() if k < 0 and not v.get('giftCardId')}
        coupon_create_vals = [{
            'program_id': p['program_id'],
            'partner_id': p.get('partner_id', False),
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
                'partner_id': coupon_vals.get('partner_id', False),
            })
            updated_gift_cards |= gift_card

        # Map the newly created coupons
        for old_id, new_id in zip(coupons_to_create.keys(), new_coupons):
            coupon_new_id_map[new_id.id] = old_id

        all_coupons = self.env['loyalty.card'].browse(coupon_new_id_map.keys()).exists()
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

    def _prepare_order_line(self, order_line):
        order_line = super()._prepare_order_line(order_line)
        for f in ['reward_id', 'coupon_id']:
            if order_line.get(f):
                order_line[f] = order_line[f][0]
        return order_line

    def _add_activated_coupon_to_draft_orders(self, table_orders):
        table_orders = super()._add_activated_coupon_to_draft_orders(table_orders)

        for order in table_orders:
            activated_coupon = []

            rewards_list = [{
                'reward_id': orderline[2]['reward_id'],
                'coupon_id': orderline[2]['coupon_id']
                } for orderline in order['lines'] if orderline[2]['is_reward_line'] and orderline[2]['reward_id']
            ]

            order_reward_ids = self.env['loyalty.reward'].browse(set([reward_id['reward_id'] for reward_id in rewards_list]))

            for reward in rewards_list:
                order_reward_id = order_reward_ids.filtered(lambda order_reward: order_reward.id == reward['reward_id'])

                if order_reward_id:
                    if order_reward_id.program_type in ['gift_card', 'ewallet']:
                        coupon_id = self.env['loyalty.card'].search([('id', '=', reward['coupon_id'])])

                        activated_coupon.append({
                            'balance': coupon_id.points,
                            'code': coupon_id.code,
                            'id': coupon_id.id,
                            'program_id': coupon_id.program_id.id,
                        })

            order['codeActivatedCoupons'] = activated_coupon

        return table_orders

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
        return result

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import AND
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
                'fields': ['name', 'trigger', 'applies_on', 'program_type', 'date_to', 'total_order_count',
                    'limit_usage', 'max_usage', 'is_nominative', 'portal_visible', 'portal_point_name', 'trigger_product_ids'],
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
        domain_products = self.env['loyalty.reward']._get_active_products_domain()
        return {
            'search_params': {
                'domain': AND([[('program_id', 'in', self.config_id._get_program_ids().ids)], domain_products]),
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

        domain = ast.literal_eval(domain_str)

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
        for group in self.env['loyalty.card'].read_group(
            domain=[('partner_id', 'in', [p['id'] for p in partners]), ('program_id', 'in', loyalty_programs.ids)],
            fields=[f"{field_name}:array_agg" for field_name in loyalty_card_fields] + ["ids:array_agg(id)"],
            groupby=['partner_id']
        ):
            loyalty_cards = {}
            for i in range(group['partner_id_count']):
                loyalty_cards[group['ids'][i]] = {field_name: group[field_name][i] for field_name in loyalty_card_fields}
            partner_id_to_loyalty_card[group['partner_id'][0]] = loyalty_cards

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

## File: static\src\js\Loyalty.js

```javascript
/** @odoo-module **/

import { Order, Orderline, PosGlobalState} from 'point_of_sale.models';
import Registries from 'point_of_sale.Registries';
import session from 'web.session';
import concurrency from 'web.concurrency';
import { Gui } from 'point_of_sale.Gui';
import { round_decimals,round_precision } from 'web.utils';
import core from 'web.core';
import { Domain, InvalidDomainError } from '@web/core/domain';
import { sprintf } from '@web/core/utils/strings';

const _t = core._t;
const dropPrevious = new concurrency.MutexedDropPrevious(); // Used for queuing reward updates
const mutex = new concurrency.Mutex(); // Used for sequential cache updates

const COUPON_CACHE_MAX_SIZE = 4096 // Maximum coupon cache size, prevents long run memory issues and (to some extent) invalid data

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
    let factor = Math.trunc(numberItems / (n + m));
    let free = factor * m;
    let charged = numberItems - free;
    // adjust the calculated free quantities
    let x = (factor + 1) * n;
    let y = x + (factor + 1) * m;
    let adjustment = x <= charged && charged < y ? charged - x : 0;
    return Math.floor(free + adjustment);
}

const PosLoyaltyGlobalState = (PosGlobalState) => class PosLoyaltyGlobalState extends PosGlobalState {
    //@override
    async _processData(loadedData) {
        this.couponCache = {};
        this.partnerId2CouponIds = {};
        this.rewards = loadedData['loyalty.reward'] || [];

        for (const reward of this.rewards) {
            reward.all_discount_product_ids = new Set(reward.all_discount_product_ids);
        }

        this.fieldTypes = loadedData['field_types'];
        await super._processData(loadedData);
        this.productId2ProgramIds = loadedData['product_id_to_program_ids'];
        this.programs = loadedData['loyalty.program'] || []; //TODO: rename to `loyaltyPrograms` etc
        this.rules = loadedData['loyalty.rule'] || [];
        this._loadLoyaltyData();
    }

    _loadProductProduct(products) {
        super._loadProductProduct(...arguments);

        for (const reward of this.rewards) {
            this.compute_discount_product_ids(reward, products);
        }

        this.rewards = this.rewards.filter(Boolean)
    }

    compute_discount_product_ids(reward, products) {
        const reward_product_domain = JSON.parse(reward.reward_product_domain);
        if (!reward_product_domain) {
            return;
        }

        const domain = new Domain(reward_product_domain);

        try {
            products
                .filter((product) => domain.contains(product))
                .forEach(product => reward.all_discount_product_ids.add(product.id));
        } catch (error) {
            if (!(error instanceof InvalidDomainError)) {
                throw error
            }
            const index = this.rewards.indexOf(reward);
            if (index != -1) {
                Gui.showPopup('ErrorPopup', {
                    title: _t('A reward could not be loaded'),
                    body:  sprintf(
                        _t('The reward "%s" contain an error in its domain, your domain must be compatible with the PoS client'),
                        this.rewards[index].description)
                    });
                this.rewards[index] = null;
            }
        }
    }

    async _getTableOrdersFromServer(tableIds) {
        const oldOrders = this.orders;
        const orders = await super._getTableOrdersFromServer(tableIds);

        const oldOrderlinesWithCoupons = [].concat(...oldOrders.map(oldOrder =>
            oldOrder.orderlines.filter(orderline => orderline.is_reward_line && orderline.coupon_id < 1)
        ));

        // Remapping of coupon_id for both couponPointChanges and Orderline.coupon_id
        if (oldOrderlinesWithCoupons.length) {
            for (const oldOrderline of oldOrderlinesWithCoupons) {
                const matchingOrderline = orders
                    .flatMap((order) => order.lines.map((line) => line[2]))
                    .find(line => line.reward_id === oldOrderline.reward_id);

                if (matchingOrderline) {
                    matchingOrderline.coupon_id = nextId;
                }
            }

            for (const order of orders) {
                const oldOrder = oldOrders.find(oldOrder => oldOrder.uid === order.uid);

                if (oldOrder) {
                    if (oldOrder.partner && oldOrder.partner.id === order.partner_id) {
                        order.partner = oldOrder.partner;
                    }

                    order.couponPointChanges = oldOrder.couponPointChanges;

                    Object.keys(order.couponPointChanges).forEach(index => {
                        order.couponPointChanges[nextId] = {...order.couponPointChanges[index]};
                        order.couponPointChanges[nextId].coupon_id = nextId;
                        delete order.couponPointChanges[index];
                    });
                }
            }
        }

        return orders;
    }

    _loadLoyaltyData() {
        this.program_by_id = {};
        this.reward_by_id = {};

        for (const program of this.programs) {
            this.program_by_id[program.id] = program;
            if (program.date_to) {
                program.date_to = new Date(program.date_to);
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
            this.reward_by_id[reward.id] = reward
            reward.program_id = this.program_by_id[reward.program_id[0]];;
            reward.discount_line_product_id = this.db.get_product_by_id(reward.discount_line_product_id[0]);
            reward.all_discount_product_ids = new Set(reward.all_discount_product_ids);
            reward.program_id.rewards.push(reward);
        }
    }
    async load_server_data() {
        await super.load_server_data(...arguments);
        if (this.selectedOrder) {
            this.selectedOrder._updateRewards();
        }
    }
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
    }
    /**
     * Fetches `loyalty.card` records from the server and adds/updates them in our cache.
     *
     * @param {domain} domain For the search
     * @param {int} limit Default to 1
     */
    async fetchCoupons(domain, limit=1) {
        const result = await this.env.services.rpc({
            model: 'loyalty.card',
            method: 'search_read',
            kwargs: {
                domain: domain,
                fields: ['id', 'points', 'code', 'partner_id', 'program_id', 'expiration_date'],
                limit: limit,
                context: session.user_context,
            }
        });
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
            const coupon = new PosLoyaltyCard(dbCoupon.code, dbCoupon.id, dbCoupon.program_id[0], dbCoupon.partner_id[0], dbCoupon.points, dbCoupon.expiration_date);
            this.couponCache[coupon.id] = coupon;
            this.partnerId2CouponIds[coupon.partner_id] = this.partnerId2CouponIds[coupon.partner_id] || new Set();
            this.partnerId2CouponIds[coupon.partner_id].add(coupon.id);
            couponList.push(coupon);
        }
        return couponList;
    }
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
        const fetchedCoupons = await this.fetchCoupons([['partner_id', '=', partnerId], ['program_id', '=', programId]]);
        const dbCoupon = fetchedCoupons.length > 0 ? fetchedCoupons[0] : null;
        return dbCoupon || new PosLoyaltyCard(null, null, programId, partnerId, 0);
    }
    getLoyaltyCards(partner) {
        const loyaltyCards = [];
        if (this.partnerId2CouponIds[partner.id]) {
            this.partnerId2CouponIds[partner.id].forEach(couponId => loyaltyCards.push(this.couponCache[couponId]));
        }
        return loyaltyCards;
    }
    addPartners(partners) {
        const result = super.addPartners(partners);
        // cache the loyalty cards of the partners
        for (const partner of partners) {
            for (const [couponId, { code, program_id, points }] of Object.entries(partner.loyalty_cards || {})) {
                this.couponCache[couponId] = new PosLoyaltyCard(code, parseInt(couponId, 10), program_id, partner.id, points);
                this.partnerId2CouponIds[partner.id] = this.partnerId2CouponIds[partner.id] || new Set();
                this.partnerId2CouponIds[partner.id].add(couponId);
            }
        }
        return result;
    }
}
Registries.Model.extend(PosGlobalState, PosLoyaltyGlobalState);

const PosLoyaltyOrderline = (Orderline) => class PosLoyaltyOrderline extends Orderline {
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
        result.eWalletGiftCardProgramId = this.eWalletGiftCardProgram ? this.eWalletGiftCardProgram.id : null;
        return result;
    }
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
    }
    set_quantity(quantity, keep_price) {
        if (quantity === 'remove' && this.is_reward_line) {
            // Remove any line that is part of that same reward aswell.
            const linesToRemove = []
            this.order.get_orderlines().forEach((line) => {
                if (line != this &&
                        line.reward_id === this.reward_id &&
                        line.coupon_id === this.coupon_id &&
                        line.reward_identifier_code === this.reward_identifier_code) {
                    linesToRemove.push(line);
                }
            });
            for (const line of linesToRemove) {
                this.order.orderlines.remove(line);
            }
        }
        return super.set_quantity(...arguments);
    }
    getEWalletGiftCardProgramType() {
        return this.eWalletGiftCardProgram && this.eWalletGiftCardProgram.program_type
    }
    ignoreLoyaltyPoints({ program }) {
        return (
            ['gift_card', 'ewallet'].includes(program.program_type) && this.eWalletGiftCardProgram &&
            this.eWalletGiftCardProgram.id !== program.id
        );
    }
}
Registries.Model.extend(Orderline, PosLoyaltyOrderline);

const PosLoyaltyOrder = (Order) => class PosLoyaltyOrder extends Order {
    constructor() {
        super(...arguments);
        this._initializePrograms({});
        // Always start with invalid coupons so that coupon for this
        // order is properly assigned. @see _checkMissingCoupons
        this.invalidCoupons = true;
    }
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        json.disabledRewards = [...this.disabledRewards];
        json.codeActivatedProgramRules = this.codeActivatedProgramRules;
        json.codeActivatedCoupons = this.codeActivatedCoupons;
        json.couponPointChanges = this.couponPointChanges;
        return json;
    }
    init_from_JSON(json) {
        this.couponPointChanges = json.couponPointChanges;
        this.partner = json.partner;
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
    }
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
            const loyaltyProgramIds = new Set(this.pos.programs.filter(program => program.is_nominative).map(program => program.id));
            for (const [key, pointChange] of Object.entries(this.couponPointChanges)) {
                if (loyaltyProgramIds.has(pointChange.program_id)) {
                    delete this.couponPointChanges[key];
                }
            }
            this._updateRewards();
        }
    }
    wait_for_push_order() {
        return (!_.isEmpty(this.couponPointChanges) || this._get_reward_lines().length || super.wait_for_push_order(...arguments));
    }
    /**
     * Add additional information for our ticket, such as new coupons and loyalty point gains.
     *
     * @override
     */
    export_for_printing() {
        const result = super.export_for_printing(...arguments);
        if (this.get_partner()) {
            result.loyaltyStats = this.getLoyaltyPoints();
        }
        result.new_coupon_info = this.new_coupon_info;
        return result;
    }
    //@override
    _get_ignored_product_ids_total_discount() {
        const productIds = super._get_ignored_product_ids_total_discount(...arguments);
        const giftCardPrograms = this.pos.programs.filter(p => p.program_type === 'gift_card');
        for (const program of giftCardPrograms) {
            const giftCardProductId = [...program.rules[0].valid_product_ids][0];
            if (giftCardProductId) {
                productIds.push(giftCardProductId);
            }
        }
        return productIds;
    }
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
    }
    _get_reward_lines() {
        const orderLines = super.get_orderlines(...arguments);
        if (orderLines) {
            return orderLines.filter((line) => line.is_reward_line);
        }
        return orderLines;
    }
    _get_regular_order_lines() {
        const orderLines = super.get_orderlines(...arguments);
        if (orderLines) {
            return orderLines.filter((line) => !line.is_reward_line && !line.refunded_orderline_id);
        }
        return orderLines;
    }
    get_last_orderline() {
        const orderLines = super.get_orderlines(...arguments).filter((line) => !line.is_reward_line);
        return orderLines[orderLines.length - 1];
    }
    set_pricelist(pricelist) {
        super.set_pricelist(...arguments);
        this._updateRewards();
    }
    set_orderline_options(line, options) {
        super.set_orderline_options(...arguments);
        if (options && options.is_reward_line) {
            line.is_reward_line = options.is_reward_line;
            line.reward_id = options.reward_id;
            line.reward_product_id = options.reward_product_id;
            line.coupon_id = options.coupon_id;
            line.reward_identifier_code = options.reward_identifier_code;
            line.points_cost = options.points_cost;
            line.price_automatically_set = true;
        }
        line.giftBarcode = options.giftBarcode;
        line.giftCardId = options.giftCardId;
        line.eWalletGiftCardProgram = options.eWalletGiftCardProgram;
    }
    add_product(product, options) {
        super.add_product(...arguments);
        this._updateRewards();
    }

    async _initializePrograms() {
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
    }
    _resetPrograms() {
        this.disabledRewards = new Set();
        this.codeActivatedProgramRules = [];
        this.codeActivatedCoupons = [];
        this.couponPointChanges = {};
        this.orderlines.remove(this._get_reward_lines());
        this._updateRewards();
    }
    _updateRewards() {
        // Calls are not expected to take some time besides on the first load + when loyalty programs are made applicable
        if (this.pos.programs.length === 0) {
            return;
        }
        dropPrevious.exec(() => {return this._updateLoyaltyPrograms().then(async () => {
            // Try auto claiming rewards
            const claimableRewards = this.getClaimableRewards(false, false, true);
            let changed = false;
            for (const {coupon_id, reward} of claimableRewards) {
                if (reward.program_id.rewards.length === 1 && !reward.program_id.is_nominative &&
                    (reward.reward_type !== 'product' || (reward.reward_type == 'product' && !reward.multi_product))) {
                    this._applyReward(reward, coupon_id);
                    changed = true;
                }
            }
            // Rewards may impact the number of points gained
            if (changed) {
                await this._updateLoyaltyPrograms();
            }
            this._updateRewardLines();
        })}).catch(() => {/* catch the reject of dp when calling `add` to avoid unhandledrejection */});
    }
    async _updateLoyaltyPrograms() {
        await this._checkMissingCoupons();
        await this._updatePrograms();
    }
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
                await this.pos.fetchCoupons([['id', 'in', couponsToFetch]], couponsToFetch.length);
                // Remove coupons that could not be loaded from the db
                this.codeActivatedCoupons = this.codeActivatedCoupons.filter((coupon) => this.pos.couponCache[coupon.id]);
                this.couponPointChanges = Object.fromEntries(Object.entries(this.couponPointChanges).filter(([k, pe]) => this.pos.couponCache[pe.coupon_id]));
            }
        });
    }
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
        const productRewards = []
        const otherRewards = [];
        const paymentRewards = []; // Gift card and ewallet rewards are considered payments and must stay at the end

        for (const line of rewardLines) {
            const claimedReward = {
                reward: this.pos.reward_by_id[line.reward_id],
                coupon_id: line.coupon_id,
                args: {
                    product: line.reward_product_id,
                },
                reward_identifier_code: line.reward_identifier_code,
            }
            if (claimedReward.reward.program_id.program_type === 'gift_card' || claimedReward.reward.program_id.program_type === 'ewallet') {
                paymentRewards.push(claimedReward);
            } else if (claimedReward.reward.reward_type === 'product') {
                productRewards.push(claimedReward);
            } else if (!otherRewards.some(reward => reward.reward_identifier_code === claimedReward.reward_identifier_code)) {
                otherRewards.push(claimedReward);
            }
            this.orderlines.remove(line);
        }
        for (const claimedReward of productRewards.concat(otherRewards).concat(paymentRewards)) {
            // For existing coupons check that they are still claimed, they can exist in either `couponPointChanges` or `codeActivatedCoupons`
            if (!this.codeActivatedCoupons.find((coupon) => coupon.id === claimedReward.coupon_id) &&
                !this.couponPointChanges[claimedReward.coupon_id]) {
                continue;
            }
            this._applyReward(claimedReward.reward, claimedReward.coupon_id, claimedReward.args);
        }
    }
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
        const newPointChanges = Object.assign({}, JSON.parse(JSON.stringify(this.couponPointChanges)));
        for (const pe of Object.values(newPointChanges)) {
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
            const pointsAdded = this._programIsApplicable(program) ? pointsAddedPerProgram[program.id] : [];
            // For programs that apply to both (loyalty) we always add a change of 0 points, if there is none, since it makes it easier to
            //  track for claimable rewards, and makes sure to load the partner's loyalty card.
            if (program.is_nominative && !pointsAdded.length && this.get_partner()) {
                pointsAdded.push({points: 0});
            }
            const oldChanges = changesPerProgram[program.id] || [];
            // Update point changes for those that exist
            for (let idx = 0; idx < Math.min(pointsAdded.length, oldChanges.length); idx++) {
                Object.assign(oldChanges[idx], pointsAdded[idx]);
            }
            if (pointsAdded.length < oldChanges.length) {
                const removedIds = oldChanges.map((pe) => pe.coupon_id);
                removedIds.forEach(id => delete newPointChanges[id]);
            } else if (pointsAdded.length > oldChanges.length) {
                for (const pa of pointsAdded.splice(oldChanges.length)) {
                    const coupon = await this._couponForProgram(program);
                    newPointChanges[coupon.id] = {
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
            if (program.applies_on === 'current' && pointsAddedPerProgram[program.id].length === 0) {
                return false;
            }
            return true;
        });
        this.couponPointChanges = newPointChanges;
    }
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
            if (program.program_type !== 'loyalty') {
                // Not a loyalty program, skip
                continue;
            }
            const loyaltyCard = this.pos.couponCache[coupon_id] || /* or new card */ { id: coupon_id, balance: 0 };
            let [won, spent, total] = [0, 0, 0];
            let balance = loyaltyCard.balance;
            won += points - this._getPointsCorrection(program);
            if (coupon_id !== 0) {
                for (const line of this._get_reward_lines()) {
                    if (line.coupon_id === coupon_id) {
                        spent += line.points_cost;
                    }
                }
            }
            total = balance + won - spent;
            const name = program.portal_visible ? program.portal_point_name : _t('Points');
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
        return Object.entries(loyaltyPoints).map(([couponId, points]) => ({ couponId, points, program: points.program }));
    }
    /**
     * The points in the couponPointChanges for free product reward is not correct.
     * It doesn't take into account the points from the `free` product. Use this method
     * to compute the necessary correction.
     * @param {*} program
     * @returns {number}
     */
    _getPointsCorrection(program) {
        const rewardLines = this.orderlines.filter(line => line.is_reward_line);
        let res = 0;
        for (const rule of program.rules) {
            for (const line of rewardLines) {
                const reward = this.pos.reward_by_id[line.reward_id]
                if (this._validForPointsCorrection(reward, line, rule)) {
                    if (rule.reward_point_mode === 'money') {
                        res -= round_precision(rule.reward_point_amount * line.get_price_with_tax(), 0.01);
                    } else if (rule.reward_point_mode === 'unit') {
                        res += rule.reward_point_amount * line.get_quantity();
                    }
                }
            }
        }
        return res;
    }
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
        if (reward.reward_type !== 'product') {
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
    }
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
                if (this.pos.program_by_id[pe.program_id].applies_on !== 'future') {
                    points += pe.points;
                }
                // couponPointChanges is not supposed to have a coupon multiple times
                return true;
            }
            return false
        });
        for (const line of this.get_orderlines()) {
            if (line.is_reward_line && line.coupon_id === coupon_id) {
                points -= line.points_cost;
            }
        }
        return points
    }
    /**
     * Depending on the program type returns a new (local) instance of coupon or tries to retrieve the coupon in case of loyalty cards.
     * Existing coupons are put in a cache which is also used to fetch the coupons.
     *
     * @param {object} program
     */
    async _couponForProgram(program) {
        if (program.is_nominative) {
            return this.pos.fetchLoyaltyCard(program.id, this.get_partner().id);
        }
        // This type of coupons don't need to really exist up until validating the order, so no need to cache
        return new PosLoyaltyCard(null, null, program.id, (this.get_partner() || {id: -1}).id, 0);
    }
    _programIsApplicable(program) {
        if (program.trigger === 'auto' && !program.rules.find((rule) => rule.mode === 'auto' || this.codeActivatedProgramRules.includes(rule.id))) {
            return false;
        }
        if (program.trigger === 'with_code' && !program.rules.find((rule) => this.codeActivatedProgramRules.includes(rule.id))) {
            return false;
        }
        if (program.is_nominative && !this.get_partner()) {
            return false;
        }
        if (program.date_to && program.date_to <= new Date()) {
            return false;
        }
        if (program.limit_usage && program.total_order_count >= program.max_usage) {
            return false;
        }
        return true;
    }
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
            const reward = line.reward_id
              ? this.pos.reward_by_id[line.reward_id]
              : undefined;
            const isDiscount = reward && reward.reward_type === "discount";
            const rewardProgram = reward && reward.program_id;
            // Skip lines for automatic discounts.
            if (isDiscount && rewardProgram.trigger === 'auto') {
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
        const result = {}
        for (const program of programs) {
            let points = 0;
            const splitPoints = [];
            for (const rule of program.rules) {
                if (rule.mode === 'with_code' && !this.codeActivatedProgramRules.includes(rule.id)) {
                    continue;
                }
                const linesForRule = linesPerRule[rule.id] ? linesPerRule[rule.id] : [];
                const amountWithTax = linesForRule.reduce((sum, line) => sum + line.get_price_with_tax(), 0);
                const amountWithoutTax = linesForRule.reduce((sum, line) => sum + line.get_price_without_tax(), 0);
                const amountCheck = rule.minimum_amount_tax_mode === 'incl' && amountWithTax || amountWithoutTax;
                if (rule.minimum_amount > amountCheck) {
                    continue;
                }
                let totalProductQty = 0;
                // Only count points for paid lines.
                const qtyPerProduct = {};
                let orderedProductPaid = 0;
                for (const line of orderLines) {
                    if (((!line.reward_product_id && (rule.any_product || rule.valid_product_ids.has(line.get_product().id))) ||
                        (line.reward_product_id && (rule.any_product || rule.valid_product_ids.has(line.reward_product_id)))) &&
                        !line.ignoreLoyaltyPoints({ program })){
                        if (line.is_reward_line) {
                            const reward = this.pos.reward_by_id[line.reward_id];
                            if ((program.id === reward.program_id.id) || ['gift_card', 'ewallet'].includes(reward.program_id.program_type)) {
                                continue;
                            }
                        }
                        const lineQty = (line.reward_product_id ? -line.get_quantity() : line.get_quantity());
                        if (qtyPerProduct[line.reward_product_id || line.get_product().id]) {
                            qtyPerProduct[line.reward_product_id || line.get_product().id] += lineQty;
                        } else {
                            qtyPerProduct[line.reward_product_id || line.get_product().id] = lineQty;
                        }
                        orderedProductPaid += line.get_price_with_tax();
                        if(!line.is_reward_line){
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
                pointsForProgramsCountedRules[program.id].push(rule.id)
                if (program.applies_on === 'future' && rule.reward_point_split && rule.reward_point_mode !== 'order') {
                    // In this case we count the points per rule
                    if (rule.reward_point_mode === 'unit') {
                        splitPoints.push(...Array.apply(null, Array(totalProductQty)).map((_) => {return {points: rule.reward_point_amount}}));
                    } else if (rule.reward_point_mode === 'money') {
                        for (const line of orderLines) {
                            if (line.is_reward_line || !(rule.valid_product_ids.has(line.get_product().id)) || line.get_quantity() <= 0
                                || line.ignoreLoyaltyPoints({ program })) {
                                continue;
                            }
                            const pointsPerUnit = round_precision(rule.reward_point_amount * line.get_price_with_tax() / line.get_quantity(), 0.01);
                            if (pointsPerUnit > 0) {
                                splitPoints.push(...Array.apply(null, Array(line.get_quantity())).map(() => {
                                    if (line.giftBarcode && line.get_quantity() == 1) {
                                        return {points: pointsPerUnit, barcode: line.giftBarcode, giftCardId: line.giftCardId };
                                    }
                                    return {points: pointsPerUnit}
                                }));
                            }
                        }
                    }
                } else {
                    // In this case we add on to the global point count
                    if (rule.reward_point_mode === 'order') {
                        points += rule.reward_point_amount;
                    } else if (rule.reward_point_mode === 'money') {
                        // NOTE: unlike in sale_loyalty this performs a round half-up instead of round down
                        points += round_precision(rule.reward_point_amount * orderedProductPaid, 0.01);
                    } else if (rule.reward_point_mode === 'unit') {
                        points += rule.reward_point_amount * totalProductQty;
                    }
                }
            }
            const res = (points || program.program_type === 'coupons') ? [{points}] : [];
            if (splitPoints.length) {
                res.push(...splitPoints);
            }
            result[program.id] = res;
        }
        return result;
    }
    /**
     * @returns {Array} List of lines composing the global discount
     */
    _getGlobalDiscountLines() {
        return this.get_orderlines().filter((line) => line.reward_id && this.pos.reward_by_id[line.reward_id].is_global_discount);
    }
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
    }
    /**
     * Checks whether this order is allowed to generate rewards
     * from the given coupon program.
     * @param {*} couponProgram
     */
    _canGenerateRewards(couponProgram, orderTotalWithTax, orderTotalWithoutTax) {
        for (const rule of couponProgram.rules) {
            const amountToCompare = rule.minimum_amount_tax_mode == 'incl' ? orderTotalWithTax : orderTotalWithoutTax
            if (rule.minimum_amount > amountToCompare) {
                return false;
            }
            const nItems = this._computeNItems(rule);
            if (rule.minimum_qty > nItems) {
                return false;
            }
        }
        return true;
    }
    /**
     * @param {Integer} coupon_id (optional) Coupon id
     * @param {Integer} program_id (optional) Program id
     * @returns {Array} List of {Object} containing the coupon_id and reward keys
     */
    getClaimableRewards(coupon_id=false, program_id=false, auto=false) {
        const allCouponPrograms = Object.values(this.couponPointChanges).map((pe) => {
            return {
                program_id: pe.program_id,
                coupon_id: pe.coupon_id,
            };
        }).concat(this.codeActivatedCoupons.map((coupon) => {
            return {
                program_id: coupon.program_id,
                coupon_id: coupon.id,
            };
        }));
        const result = [];
        const totalWithTax = this.get_total_with_tax();
        const totalWithoutTax = this.get_total_without_tax();
        const totalIsZero = totalWithTax === 0;
        const globalDiscountLines = this._getGlobalDiscountLines();
        const globalDiscountPercent = globalDiscountLines.length ?
            this.pos.reward_by_id[globalDiscountLines[0].reward_id].discount : 0;
        for (const couponProgram of allCouponPrograms) {
            const program = this.pos.program_by_id[couponProgram.program_id];
            if (program.trigger == 'with_code') {
                // For coupon programs, the rules become conditions.
                // Points to purchase rewards will only come from the scanned coupon.
                if (!this._canGenerateRewards(program, totalWithTax, totalWithoutTax)) {
                    continue;
                };
            }
            if ((coupon_id && couponProgram.coupon_id !== coupon_id) ||
                (program_id && couponProgram.program_id !== program_id)) {
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
                if (reward.reward_type === 'discount' && totalIsZero) {
                    continue;
                }
                let potentialQty;
                if (reward.reward_type === 'product') {
                    if(!reward.multi_product){
                        const product = this.pos.db.get_product_by_id(reward.reward_product_ids[0]);
                        potentialQty = this._computeUnclaimedFreeProductQty(reward, couponProgram.coupon_id, product, points);
                    }
                    if (!potentialQty || potentialQty <= 0) {
                        continue;
                    }
                }
                result.push({
                    coupon_id: couponProgram.coupon_id,
                    reward: reward,
                    potentialQty
                });
            }
        }
        return result;
    }
    /**
     * Returns the reward such that when its reward product is added
     * in the order, it will be added as free. That is, when added,
     * it comes with the corresponding reward product line.
     */
    getPotentialFreeProductRewards() {
        const allCouponPrograms = Object.values(this.couponPointChanges).map((pe) => {
            return {
                program_id: pe.program_id,
                coupon_id: pe.coupon_id,
            };
        }).concat(this.codeActivatedCoupons.map((coupon) => {
            return {
                program_id: coupon.program_id,
                coupon_id: coupon.id,
            };
        }));
        const result = [];
        for (const couponProgram of allCouponPrograms) {
            const program = this.pos.program_by_id[couponProgram.program_id];
            const points = this._getRealCouponPoints(couponProgram.coupon_id);
            const hasLine = this.orderlines.filter(line => !line.is_reward_line).length > 0;
            for (const reward of program.rewards.filter(reward => reward.reward_type == 'product')) {
                if (points < reward.required_points) {
                    continue;
                }
                // Loyalty program (applies_on == 'both') should needs an orderline before it can apply a reward.
                const considerTheReward = program.applies_on !== 'both' || (program.applies_on == 'both' && hasLine);
                if (reward.reward_type === 'product' && considerTheReward) {
                    let hasPotentialQty = true;
                    let potentialQty;
                    for (const productId of reward.reward_product_ids) {
                        const product = this.pos.db.get_product_by_id(productId);
                        potentialQty = this._computePotentialFreeProductQty(reward, product, points);
                        if (potentialQty <= 0) {
                            hasPotentialQty = false;
                        }
                    }
                    if (hasPotentialQty) {
                        result.push({
                            coupon_id: couponProgram.coupon_id,
                            reward: reward,
                            potentialQty
                        });
                    }
                }
            }
        }
        return result;
    }
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
                if (rewardId != reward.id && this.pos.reward_by_id[rewardId].discount >= reward.discount) {
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
            product: args['product'] || null,
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
    }
    _createLineFromVals(vals) {
        vals['lst_price'] = vals['price'];
        const line = Orderline.create({}, {pos: this.pos, order: this, product: vals['product']});
        this.fix_tax_included_price(line);
        this.set_orderline_options(line, vals);
        return line;
    }
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
        return {discountable, discountablePerTax};
    }
    /**
     * @returns the order's cheapest line
     */
    _getCheapestLine() {
        let cheapestLine;
        for (const line of this.get_orderlines()) {
            if (line.reward_id || !line.get_quantity()) {
                continue;
            }
            if (!cheapestLine || cheapestLine.price > line.price) {
                cheapestLine = line;
            }
        }
        return cheapestLine;
    }
    /**
     * @param {loyalty.reward} reward
     * @returns the discountable and discountable per tax for this discount on cheapest reward.
     */
    _getDiscountableOnCheapest(reward) {
        const cheapestLine = this._getCheapestLine();
        if (!cheapestLine) {
            return {discountable: 0, discountablePerTax: {}};
        }
        const taxKey = cheapestLine.get_taxes().map((t) => t.id);
        return {discountable: cheapestLine.price, discountablePerTax: Object.fromEntries([[taxKey, cheapestLine.price]])};
    }
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
            if (applicableProducts.has(line.get_product().id) ||
                applicableProducts.has(line.reward_product_id)) {
                discountableLines.push(line);
            }
        }
        return discountableLines;
    }
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
            remainingAmountPerLine[line.cid] = line.get_price_with_tax();
            if (applicableProducts.has(line.get_product().id) ||
                (line.reward_product_id && applicableProducts.has(line.reward_product_id))) {
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
            if (lineReward.reward_type !== 'discount') {
                continue;
            }
            let discountedLines = orderLines;
            if (lineReward.discount_applicability === 'cheapest') {
                cheapestLine = cheapestLine || this._getCheapestLine();
                discountedLines = [cheapestLine];
            } else if (lineReward.discount_applicability === 'specific') {
                discountedLines = this._getSpecificDiscountableLines(lineReward);
            }
            if (!discountedLines.length) {
                continue;
            }
            if (lineReward.discount_mode === 'percent') {
                const discount = lineReward.discount / 100;
                for (const line of discountedLines) {
                    if (line.reward_id) {
                        continue;
                    }
                    if (lineReward.discount_applicability === 'cheapest') {
                        remainingAmountPerLine[line.cid] *= (1 - (discount / line.get_quantity()))
                    } else {
                        remainingAmountPerLine[line.cid] *= (1 - discount);
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
            discountablePerTax[taxKey] += (line.get_base_price()) * (remainingAmountPerLine[line.cid] / line.get_price_with_tax());
        }
        return {discountable, discountablePerTax};
    }
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the reward lines
     */
    _getRewardLineValues(args) {
        const reward = args['reward'];
        if (reward.reward_type === 'discount') {
            return this._getRewardLineValuesDiscount(args);
        } else if (reward.reward_type === 'product') {
            return this._getRewardLineValuesProduct(args);
        }
        // NOTE: we may reach this step if for some reason there is a free shipping reward
        return [];
    }
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the discount lines
     */
    _getRewardLineValuesDiscount(args) {
        const reward = args['reward'];
        const coupon_id = args['coupon_id'];
        const rewardAppliesTo = reward.discount_applicability;
        let getDiscountable;
        if (rewardAppliesTo === 'order') {
            getDiscountable = this._getDiscountableOnOrder.bind(this);
        } else if (rewardAppliesTo === 'cheapest') {
            getDiscountable = this._getDiscountableOnCheapest.bind(this);
        } else if (rewardAppliesTo === 'specific') {
            getDiscountable = this._getDiscountableOnSpecific.bind(this);
        }
        if (!getDiscountable) {
            return _t("Unknown discount type");
        }
        let {discountable, discountablePerTax} = getDiscountable(reward);
        discountable = Math.min(this.get_total_with_tax(), discountable);
        if (!discountable) {
            return [];
        }
        let maxDiscount = reward.discount_max_amount || Infinity;
        if (reward.discount_mode === 'per_point') {
            let points = (["ewallet", "gift_card"].includes(reward.program_id.program_type)) ?
                this._getRealCouponPoints(coupon_id) :
                Math.floor(this._getRealCouponPoints(coupon_id) / reward.required_points) * reward.required_points;
            maxDiscount = Math.min(maxDiscount, reward.discount * points);
        } else if (reward.discount_mode === 'per_order') {
            maxDiscount = Math.min(maxDiscount, reward.discount);
        } else if (reward.discount_mode === 'percent') {
            maxDiscount = Math.min(maxDiscount, discountable * (reward.discount / 100));
        }
        const rewardCode = _newRandomRewardCode();
        let pointCost = reward.clear_wallet ? this._getRealCouponPoints(coupon_id) : reward.required_points;
        if (reward.discount_mode === 'per_point' && !reward.clear_wallet) {
            pointCost = Math.min(maxDiscount, discountable) / reward.discount;
        }
        // These are considered payments and do not require to be either taxed or split by tax
        const discountProduct = reward.discount_line_product_id;
        if (['ewallet', 'gift_card'].includes(reward.program_id.program_type)) {
            const taxes_to_apply = discountProduct.taxes_id.map(id => { return { ...this.pos.taxes_by_id[id], price_include:true } })
            const tax_res = this.pos.compute_all(taxes_to_apply, -Math.min(maxDiscount, discountable), 1, this.pos.currency.rounding)
            let new_price = tax_res['total_excluded']
            new_price += tax_res.taxes.filter(tax => this.pos.taxes_by_id[tax.id].price_include).reduce((sum,tax) => sum += tax.amount,0)
            return [{
                product: discountProduct,
                price: new_price,
                quantity: 1,
                reward_id: reward.id,
                is_reward_line: true,
                coupon_id: coupon_id,
                points_cost: pointCost,
                reward_identifier_code: rewardCode,
                merge: false,
                taxIds: discountProduct.taxes_id
            }];
        }
        const discountFactor = discountable ? Math.min(1, (maxDiscount / discountable)) : 1;
        const result = Object.entries(discountablePerTax).reduce((lst, entry) => {
            // Ignore 0 price lines
            if (!entry[1]) {
                return lst;
            }
            const taxIds = entry[0] === '' ? [] : entry[0].split(',').map((str) => parseInt(str));
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
            result[0]['points_cost'] = pointCost;
        }
        return result;
    }
    _isRewardProductPartOfRules(reward, product) {
        return (
            reward.program_id.rules.filter((rule) => rule.any_product || rule.valid_product_ids.has(product.id))
                .length > 0
        );
    }
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
                if (line.reward_id == reward.id ) {
                    remainingPoints += line.points_cost;
                    claimed += line.get_quantity();
                } else {
                    shouldCorrectRemainingPoints = true;
                }
            }
        }
        let freeQty;
        if (reward.program_id.trigger == 'auto') {
            if (this._isRewardProductPartOfRules(reward, product) && reward.program_id.applies_on !== 'future') {
                // OPTIMIZATION: Pre-calculate the factors for each reward-product combination during the loading.
                // For points not based on quantity, need to normalize the points to compute free quantity.
                const appliedRulesIds = this.couponPointChanges[coupon_id].appliedRules;
                const appliedRules = appliedRulesIds !== undefined
                    ? reward.program_id.rules.filter(rule => appliedRulesIds.includes(rule.id))
                    : reward.program_id.rules;
                let factor = 0;
                let orderPoints = 0;
                for (const rule of appliedRules) {
                    if (rule.any_product || rule.valid_product_ids.has(product.id)) {
                        if (rule.reward_point_mode === 'order') {
                            orderPoints += rule.reward_point_amount;
                        } else if (rule.reward_point_mode === 'money') {
                            factor += round_precision(rule.reward_point_amount * product.lst_price, 0.01);
                        } else if (rule.reward_point_mode === 'unit') {
                            factor += rule.reward_point_amount;
                        }
                    }
                }
                if (factor === 0) {
                    freeQty = Math.floor((remainingPoints / reward.required_points) * reward.reward_product_qty);
                } else {
                    const correction = shouldCorrectRemainingPoints ? this._getPointsCorrection(reward.program_id) : 0
                    freeQty = computeFreeQuantity((remainingPoints - correction - orderPoints) / factor, reward.required_points / factor, reward.reward_product_qty);
                    freeQty += Math.floor((orderPoints / reward.required_points) * reward.reward_product_qty);
                }
            } else {
                freeQty = Math.floor((remainingPoints / reward.required_points) * reward.reward_product_qty);
            }
        } else if (reward.program_id.trigger == 'with_code') {
            freeQty = Math.floor((remainingPoints / reward.required_points) * reward.reward_product_qty);
        }
        return Math.min(available, freeQty) - claimed;
    }
    _computePotentialFreeProductQty(reward, product, remainingPoints) {
        if (reward.program_id.trigger == 'auto') {
            if (this._isRewardProductPartOfRules(reward, product) && reward.program_id.applies_on !== 'future') {
                const line = this.get_orderlines().find(line => line.reward_product_id === product.id);
                // Compute the correction points once even if there are multiple reward lines.
                // This is because _getPointsCorrection is taking into account all the lines already.
                const claimedPoints = line ? this._getPointsCorrection(reward.program_id) : 0;
                return Math.floor((remainingPoints - claimedPoints) / reward.required_points) > 0
                    ? reward.reward_product_qty
                    : 0;
            } else {
                return Math.floor((remainingPoints / reward.required_points) * reward.reward_product_qty);
            }
        } else if (reward.program_id.trigger == 'with_code') {
            return Math.floor((remainingPoints / reward.required_points) * reward.reward_product_qty);
        }
    }
    /**
     * @param {Object} args See `_applyReward`
     * @returns {Array} List of values to create the reward lines
     */
    _getRewardLineValuesProduct(args) {
        const reward = args['reward'];
        const product = this.pos.db.get_product_by_id(args['product'] || reward.reward_product_ids[0]);
        const points = this._getRealCouponPoints(args['coupon_id']);
        const unclaimedQty = this._computeUnclaimedFreeProductQty(reward, args['coupon_id'], product, points);
        if (unclaimedQty <= 0) {
            return _t("There are not enough products in the basket to claim this reward.");
        }
        const claimable_count = reward.clear_wallet ? 1 : Math.min(Math.ceil(unclaimedQty / reward.reward_product_qty), Math.floor(points / reward.required_points));
        const cost = reward.clear_wallet ? points : claimable_count * reward.required_points;
        // In case the reward is the product multiple times, give it as many times as possible
        const freeQuantity = Math.min(unclaimedQty, reward.reward_product_qty * claimable_count);
        return [{
            product: reward.discount_line_product_id,
            price: -round_decimals(product.get_price(this.pricelist, freeQuantity), this.pos.currency.decimal_places),
            tax_ids: product.taxes_id,
            quantity: freeQuantity,
            reward_id: reward.id,
            is_reward_line: true,
            reward_product_id: product.id,
            coupon_id: args['coupon_id'],
            points_cost: cost,
            reward_identifier_code: _newRandomRewardCode(),
            merge: false,
        }]
    }
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
            return rule.mode === 'with_code' && (rule.promo_barcode === code || rule.code === code)
        });
        let claimableRewards = null;
        if (rule) {
            if (this.codeActivatedProgramRules.includes(rule.id)) {
                return _t('That promo code program has already been activated.');
            }
            this.codeActivatedProgramRules.push(rule.id);
            await this._updateLoyaltyPrograms();
            claimableRewards = this.getClaimableRewards(false, rule.program_id.id);
        } else {
            if (this.codeActivatedCoupons.find((coupon) => coupon.code === code)) {
                return _t('That coupon code has already been scanned and activated.');
            }
            const customer = this.get_partner();
            const { successful, payload } = await this.pos.env.services.rpc({
                model: 'pos.config',
                method: 'use_coupon_code',
                args: [
                    [this.pos.config.id],
                    code,
                    this.creation_date,
                    customer ? customer.id : false,
                ],
                kwargs: { context: session.user_context },
            });
            if (successful) {
                // Allow rejecting a gift card that is not yet paid.
                const program = this.pos.program_by_id[payload.program_id];
                if (program && program.program_type === 'gift_card' && !payload.has_source_order) {
                    const { confirmed } = await Gui.showPopup('ConfirmPopup', {
                        title: _t('Unpaid gift card'),
                        body: _t('This gift card is not linked to any order. Do you really want to apply its reward?'),
                    });
                    if (!confirmed) {
                        return _t('Unpaid gift card rejected.');
                    }
                }
                const coupon = new PosLoyaltyCard(code, payload.coupon_id, payload.program_id, payload.partner_id, payload.points, payload.expiration_date);
                this.pos.couponCache[coupon.id] = coupon;
                this.codeActivatedCoupons.push(coupon);
                await this._updateLoyaltyPrograms();
                claimableRewards = this.getClaimableRewards(coupon.id);
            } else {
                return payload.error_message;
            }
        }
        if (claimableRewards && claimableRewards.length === 1) {
            if (claimableRewards[0].reward.reward_type !== 'product' || !claimableRewards[0].reward.multi_product) {
                this._applyReward(claimableRewards[0].reward, claimableRewards[0].coupon_id);
                this._updateRewards();
            }
        }
        return true;
    }
    async activateCode(code) {
        const res = await this._activateCode(code);
        if (res !== true) {
            Gui.showNotification(res);
        }
    }
}
Registries.Model.extend(Order, PosLoyaltyOrder);

```

## File: static\src\js\Orderline.js

```javascript
/** @odoo-module **/

import Orderline from 'point_of_sale.Orderline';
import Registries from 'point_of_sale.Registries';

export const PosLoyaltyOrderline = (Orderline) =>
    class extends Orderline{
        get addedClasses() {
            return Object.assign({'program-reward': this.props.line.is_reward_line}, super.addedClasses);
        }
        _isGiftCardOrEWalletReward() {
            const coupon = this.env.pos.couponCache[this.props.line.coupon_id];
            if (coupon) {
                const program = this.env.pos.program_by_id[coupon.program_id]
                return ['ewallet', 'gift_card'].includes(program.program_type) && this.props.line.is_reward_line;
            }
            return false;
        }
        _getGiftCardOrEWalletBalance() {
            const coupon = this.env.pos.couponCache[this.props.line.coupon_id];
            if (coupon) {
                return this.env.pos.format_currency(coupon.balance);
            }
            return this.env.pos.format_currency(0);
        }
    };

Registries.Component.extend(Orderline, PosLoyaltyOrderline);

```

## File: static\src\js\OrderSummary.js

```javascript
/** @odoo-module **/

import OrderSummary from 'point_of_sale.OrderSummary';
import Registries from 'point_of_sale.Registries';

export const PosLoyaltyOrderSummary = (OrderSummary) => 
    class PosLoyaltyOrderSummary extends OrderSummary {
        getLoyaltyPoints() {
            const order = this.env.pos.get_order();
            return order.getLoyaltyPoints();
        }
    };

Registries.Component.extend(OrderSummary, PosLoyaltyOrderSummary)

```

## File: static\src\js\PartnerLine.js

```javascript
odoo.define('pos_loyalty.PartnerLine', function (require) {
    'use strict';

    const PartnerLine = require('point_of_sale.PartnerLine');
    const Registries = require('point_of_sale.Registries');

    const PosLoyaltyPartnerLine = (PartnerLine) =>
        class extends PartnerLine {
            _getLoyaltyPointsRepr(loyaltyCard) {
                const program = this.env.pos.program_by_id[loyaltyCard.program_id];
                if (program.program_type === 'ewallet') {
                    return `${program.name}: ${this.env.pos.format_currency(loyaltyCard.balance)}`;
                }
                const balanceRepr = this.env.pos.format_pr(loyaltyCard.balance, 0.01);
                if (program.portal_visible) {
                    return `${balanceRepr} ${program.portal_point_name}`;
                }
                return _.str.sprintf(this.env._t('%s Points'), balanceRepr);
            }
        };

    Registries.Component.extend(PartnerLine, PosLoyaltyPartnerLine);

    return PartnerLine;
});

```

## File: static\src\js\PartnerListScreen.js

```javascript
odoo.define('pos_loyalty.PartnerListScreen', function (require) {
    'use strict';

    const PartnerListScreen = require('point_of_sale.PartnerListScreen');
    const Registries = require('point_of_sale.Registries');

    const PosLoyaltyPartnerListScreen = (PartnerListScreen) =>
        class extends PartnerListScreen {
            /**
             * Needs to be set to true to show the loyalty points in the partner list.
             * @override
             */
            get isBalanceDisplayed() {
                return true;
            }
        };

    Registries.Component.extend(PartnerListScreen, PosLoyaltyPartnerListScreen);

    return PartnerListScreen;
});

```

## File: static\src\js\PaymentScreen.js

```javascript
/** @odoo-module **/

import PaymentScreen from 'point_of_sale.PaymentScreen';
import Registries from 'point_of_sale.Registries';
import session from 'web.session';
import { PosLoyaltyCard } from '@pos_loyalty/js/Loyalty';

export const PosLoyaltyPaymentScreen = (PaymentScreen) =>
    class extends PaymentScreen {
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
            if (!await this._isOrderValid(isForceValidate)) {
                return;
            }
            // No need to do an rpc if no existing coupon is being used.
            if (!_.isEmpty(pointChanges) || newCodes.length) {
                try {
                    const {successful, payload} = await this.rpc({
                        model: 'pos.order',
                        method: 'validate_coupon_programs',
                        args: [[], pointChanges, newCodes],
                        kwargs: { context: session.user_context },
                    });
                    // Payload may contain the points of the concerned coupons to be updated in case of error. (So that rewards can be corrected)
                    if (payload && payload.updated_points) {
                        for (const pointChange of Object.entries(payload.updated_points)) {
                            if (this.env.pos.couponCache[pointChange[0]]) {
                                this.env.pos.couponCache[pointChange[0]].balance = pointChange[1];
                            }
                        }
                    }
                    if (payload && payload.removed_coupons) {
                        for (const couponId of payload.removed_coupons) {
                            if (this.env.pos.couponCache[couponId]) {
                                delete this.env.pos.couponCache[couponId];
                            }
                        }
                        this.currentOrder.codeActivatedCoupons = this.currentOrder.codeActivatedCoupons.filter((coupon) => !payload.removed_coupons.includes(coupon.id));
                    }
                    if (!successful) {
                        this.showPopup('ErrorPopup', {
                            title: this.env._t('Error validating rewards'),
                            body: payload.message,
                        });
                        return;
                    }
                } catch (_e) {
                    // Do nothing with error, while this validation step is nice for error messages
                    // it should not be blocking.
                }
            }
            await super.validateOrder(...arguments);
        }

        /**
         * @override
         */
        async _postPushOrderResolve(order, server_ids) {
            // Compile data for our function
            const rewardLines = order._get_reward_lines();
            const partner = order.get_partner();
            let couponData = Object.values(order.couponPointChanges).reduce((agg, pe) => {
                agg[pe.coupon_id] = Object.assign({}, pe, {
                    points: pe.points - order._getPointsCorrection(this.env.pos.program_by_id[pe.program_id]),
                });
                const program = this.env.pos.program_by_id[pe.program_id];
                if (program.is_nominative && partner) {
                    agg[pe.coupon_id].partner_id = partner.id;
                }
                if (program.program_type != 'loyalty') {
                    agg[pe.coupon_id].date_to = program.date_to;
                }
                return agg;
            }, {});
            for (const line of rewardLines) {
                const reward = this.env.pos.reward_by_id[line.reward_id];
                if (!couponData[line.coupon_id]) {
                    couponData[line.coupon_id] = {
                        points: 0,
                        program_id: reward.program_id.id,
                        coupon_id: line.coupon_id,
                        barcode: false,
                    }
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
            couponData = Object.fromEntries(Object.entries(couponData).filter(([key, value]) => {
                const program = this.env.pos.program_by_id[value.program_id];
                if (program.applies_on === 'current') {
                    return value.line_codes && value.line_codes.length;
                }
                return true;
            }));
            if (!_.isEmpty(couponData)) {
                const payload = await this.rpc({
                    model: 'pos.order',
                    method: 'confirm_coupon_programs',
                    args: [server_ids, couponData],
                    kwargs: { context: session.user_context },
                });
                if (payload.coupon_updates) {
                    for (const couponUpdate of payload.coupon_updates) {
                        let dbCoupon = this.env.pos.couponCache[couponUpdate.old_id];
                        if (dbCoupon) {
                            dbCoupon.id = couponUpdate.id;
                            dbCoupon.balance = couponUpdate.points;
                            dbCoupon.code = couponUpdate.code;
                        } else {
                            dbCoupon = new PosLoyaltyCard(
                                couponUpdate.code, couponUpdate.id, couponUpdate.program_id, couponUpdate.partner_id, couponUpdate.points);
                            this.env.pos.partnerId2CouponIds[partner.id] = this.env.pos.partnerId2CouponIds[partner.id] || new Set();
                            this.env.pos.partnerId2CouponIds[partner.id].add(couponUpdate.id);
                        }
                        delete this.env.pos.couponCache[couponUpdate.old_id];
                        this.env.pos.couponCache[couponUpdate.id] = dbCoupon;
                    }
                }
                // Update the usage count since it is checked based on local data
                if (payload.program_updates) {
                    for (const programUpdate of payload.program_updates) {
                        const program = this.env.pos.program_by_id[programUpdate.program_id];
                        if (program) {
                            program.total_order_count = programUpdate.usages;
                        }
                    }
                }
                if (payload.coupon_report) {
                    for (const report_entry of Object.entries(payload.coupon_report)) {
                        await this.env.legacyActionManager.do_action(report_entry[0], {
                            additional_context: {
                                active_ids: report_entry[1],
                            }
                        });
                    }
                }
                order.new_coupon_info = payload.new_coupon_info;
            }
            return super._postPushOrderResolve(order, server_ids);
        }
    };

Registries.Component.extend(PaymentScreen, PosLoyaltyPaymentScreen);

```

## File: static\src\js\ProductScreen.js

```javascript
/** @odoo-module **/

import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';
import { useBarcodeReader } from 'point_of_sale.custom_hooks';

export const PosLoyaltyProductScreen = (ProductScreen) =>
    class extends ProductScreen {
        setup() {
            super.setup();
            useBarcodeReader({
                coupon: this._onCouponScan,
            });
        }
        async _onClickPay() {
            const order = this.env.pos.get_order();
            const eWalletLine = order.get_orderlines().find(line => line.getEWalletGiftCardProgramType() === 'ewallet');
            if (eWalletLine && !order.get_partner()) {
                const {confirmed} = await this.showPopup('ConfirmPopup', {
                    title: this.env._t('Customer needed'),
                    body: this.env._t('eWallet requires a customer to be selected'),
                });
                if (confirmed) {
                    const { confirmed, payload: newPartner } = await this.showTempScreen(
                        'PartnerListScreen',
                        { partner: null }
                    );
                    if (confirmed) {
                        order.set_partner(newPartner);
                        order.updatePricelist(newPartner);
                    }
                }
            } else {
                return super._onClickPay(...arguments);
            }
        }
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
            if (this.env.pos.config.gift_card_settings == 'scan_use') {
                const { confirmed, payload: code } = await this.showPopup('TextInputPopup', {
                    title: this.env._t('Generate a Gift Card'),
                    startingValue: '',
                    placeholder: this.env._t('Enter the gift card code'),
                });
                if (!confirmed) {
                    return false;
                }
                const trimmedCode = code.trim();
                let nomenclatureRules = this.env.barcode_reader.barcode_parser.nomenclature.rules;
                if (this.env.barcode_reader.fallbackBarcodeParser) {
                    nomenclatureRules.push(...this.env.barcode_reader.fallbackBarcodeParser.nomenclature.rules);
                }
                const couponNomenclatureRules = _.filter(nomenclatureRules, function(rule) {
                    return rule.type == "coupon";
                });
                let nomenclatureCodePatterns = [];
                _.each(_.pluck(couponNomenclatureRules, "pattern"), function(pattern){
                    nomenclatureCodePatterns.push(...pattern.split("|"));
                });
                const trimmedCodeValid = _.find(nomenclatureCodePatterns, function(pattern) {
                    return trimmedCode.startsWith(pattern);
                });
                if (trimmedCode && trimmedCodeValid) {
                    // check if the code exist in the database
                    // if so, use its balance, otherwise, use the unit price of the gift card product
                    const fetchedGiftCard = await this.rpc({
                        model: 'loyalty.card',
                        method: 'search_read',
                        args: [
                            [['code', '=', trimmedCode], ['program_id', '=', program.id]],
                            ['points', 'source_pos_order_id'],
                        ],
                    });
                    // There should be maximum one gift card for a given code.
                    const giftCard = fetchedGiftCard[0];
                    if (giftCard && giftCard.source_pos_order_id) {
                        this.showPopup('ErrorPopup', {
                            title: this.env._t('This gift card has already been sold'),
                            body: this.env._t('You cannot sell a gift card that has already been sold.'),
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
                    this.showNotification('Please enter a valid gift card code.');
                    return false;
                }
            }
            return true;
        }
        async setupEWalletOptions(program, options) {
            options.quantity = 1;
            options.merge = false;
            options.eWalletGiftCardProgram = program;
            return true;
        }
        /**
         * If the product is a potential reward, also apply the reward.
         * @override
         */
        async _addProduct(product, options) {
            const linkedProgramIds = this.env.pos.productId2ProgramIds[product.id] || [];
            const linkedPrograms = linkedProgramIds.map(id => this.env.pos.program_by_id[id]);
            let selectedProgram = null;
            if (linkedPrograms.length > 1) {
                const { confirmed, payload: program } = await this.showPopup('SelectionPopup', {
                    title: this.env._t('Select program'),
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
            if (selectedProgram && selectedProgram.program_type == 'gift_card') {
                const shouldProceed = await this._setupGiftCardOptions(selectedProgram, options);
                if (!shouldProceed) {
                    return;
                }
            } else if (selectedProgram && selectedProgram.program_type == 'ewallet') {
                const shouldProceed = await this.setupEWalletOptions(selectedProgram, options);
                if (!shouldProceed) {
                    return;
                }
            }
            const order = this.env.pos.get_order();
            const potentialRewards = order.getPotentialFreeProductRewards();
            let rewardsToApply = [];
            for (const reward of potentialRewards) {
                for (const reward_product_id of reward.reward.reward_product_ids) {
                    if (reward_product_id == product.id) {
                        rewardsToApply.push(reward);
                    }
                }
            }
            await super._addProduct(product, options);
            await order._updatePrograms();
            if (rewardsToApply.length == 1) {
                const reward = rewardsToApply[0];
                order._applyReward(reward.reward, reward.coupon_id, { product: product.id });
            }
        }

        _onCouponScan(code) {
            // IMPROVEMENT: Ability to understand if the scanned code is to be paid or to be redeemed.
            this.currentOrder.activateCode(code.base_code);
        }

        async _updateSelectedOrderline(event) {
            const selectedLine = this.currentOrder.get_selected_orderline();
            if (event.detail.key === '-') {
                if (selectedLine && selectedLine.eWalletGiftCardProgram) {
                    // Do not allow negative quantity or price in a gift card or ewallet orderline.
                    // Refunding gift card or ewallet is not supported.
                    this.showNotification(this.env._t('You cannot set negative quantity or price to gift card or ewallet.'), 4000);
                    return;
                }
            }
            if (selectedLine && selectedLine.is_reward_line && !selectedLine.manual_reward &&
                    (event.detail.key === 'Backspace' || event.detail.key === 'Delete')) {
                const reward = this.env.pos.reward_by_id[selectedLine.reward_id];
                const { confirmed } = await this.showPopup('ConfirmPopup', {
                    title: this.env._t('Deactivating reward'),
                    body: _.str.sprintf(
                        this.env._t('Are you sure you want to remove %s from this order?\n You will still be able to claim it through the reward button.'),
                        reward.description
                    ),
                    cancelText: this.env._t('No'),
                    confirmText: this.env._t('Yes'),
                });
                if (confirmed) {
                    event.detail.buffer = null;
                } else {
                    // Cancel backspace
                    return;
                }
            }
            return super._updateSelectedOrderline(...arguments);
        }


        /**
         * 1/ Perform the usual set value operation (super._setValue) if the line being modified
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
                (selectedLine.is_reward_line && ['', 'remove'].includes(val))
            ) {
                super._setValue(val);
            }
            if (!selectedLine) return;
            if (selectedLine.is_reward_line && val === 'remove') {
                this.currentOrder.disabledRewards.add(selectedLine.reward_id);
                const coupon = this.env.pos.couponCache[selectedLine.coupon_id];
                if (coupon && coupon.id > 0 && this.currentOrder.codeActivatedCoupons.find((c) => c.code === coupon.code)) {
                    delete this.env.pos.couponCache[selectedLine.coupon_id];
                    this.currentOrder.codeActivatedCoupons.splice(this.currentOrder.codeActivatedCoupons.findIndex((coupon) => {
                        return coupon.id === selectedLine.coupon_id;
                    }), 1);
                }
            }
            if (!selectedLine.is_reward_line || (selectedLine.is_reward_line && val === 'remove')) {
                selectedLine.order._updateRewards();
            }
        }
        async _showDecreaseQuantityPopup() {
            const result = await super._showDecreaseQuantityPopup();
            if (result){
                this.env.pos.get_order()._updateRewards();
            }
        }
    };

Registries.Component.extend(ProductScreen, PosLoyaltyProductScreen);

```

## File: static\src\js\TicketScreen.js

```javascript
/** @odoo-module **/

import TicketScreen from 'point_of_sale.TicketScreen';
import Registries from 'point_of_sale.Registries';
import NumberBuffer from 'point_of_sale.NumberBuffer';

/**
 * Prevent refunding ewallet/gift card lines.
 */
export const PosLoyaltyTicketScreen = (TicketScreen) =>
    class PosLoyaltyTicketScreen extends TicketScreen {
        _onUpdateSelectedOrderline() {
            const order = this.getSelectedSyncedOrder();
            if (!order) return NumberBuffer.reset();
            const selectedOrderlineId = this.getSelectedOrderlineId();
            const orderline = order.orderlines.find((line) => line.id == selectedOrderlineId);
            if (orderline && this._isEWalletGiftCard(orderline)) {
                this._showNotAllowedRefundNotification();
                return NumberBuffer.reset();
            }
            return super._onUpdateSelectedOrderline(...arguments);
        }
        _prepareAutoRefundOnOrder(order) {
            const selectedOrderlineId = this.getSelectedOrderlineId();
            const orderline = order.orderlines.find((line) => line.id == selectedOrderlineId);
            if (this._isEWalletGiftCard(orderline)) {
                this._showNotAllowedRefundNotification();
                return false;
            }
            return super._prepareAutoRefundOnOrder(...arguments);
        }
        _showNotAllowedRefundNotification() {
            this.showNotification(this.env._t("Refunding a top up or reward product for an eWallet or gift card program is not allowed."), 5000);
        }
        _isEWalletGiftCard(orderline) {
            const linkedProgramIds = this.env.pos.productId2ProgramIds[orderline.product.id];
            if (linkedProgramIds) {
                return linkedProgramIds.length > 0;
            }
            if (orderline.is_reward_line) {
                const reward = this.env.pos.reward_by_id[orderline.reward_id];
                const program = reward && reward.program_id;
                if (program && ['gift_card', 'ewallet'].includes(program.program_type)) {
                    return true;
                }
            }
            return false;
        }
    };

Registries.Component.extend(TicketScreen, PosLoyaltyTicketScreen);

```

## File: static\src\js\ControlButtons\eWalletButton.js

```javascript
/** @odoo-module **/

import PosComponent from 'point_of_sale.PosComponent';
import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';

export class eWalletButton extends PosComponent {
    _getEWalletRewards(order) {
        const claimableRewards = order.getClaimableRewards();
        return claimableRewards.filter((reward_line) => {
            const coupon = this.env.pos.couponCache[reward_line.coupon_id];
            return coupon && reward_line.reward.program_id.program_type == 'ewallet' && !coupon.isExpired();
        });
    }
    _getEWalletPrograms() {
        return this.env.pos.programs.filter((p) => p.program_type == 'ewallet');
    }
    async _onClickWalletButton() {
        const order = this.env.pos.get_order();
        const eWalletPrograms = this.env.pos.programs.filter((p) => p.program_type == 'ewallet');
        const orderTotal = order.get_total_with_tax();
        const eWalletRewards = this._getEWalletRewards(order);
        if (eWalletRewards.length === 0 && orderTotal >= 0) {
            this.showPopup('ErrorPopup', {
                title: this.env._t('No valid eWallet found'),
                body: this.env._t('You either have not created an eWallet or all your eWallets have expired.'),
            });
            return;
        }
        if (orderTotal < 0 && eWalletPrograms.length >= 1) {
            let selectedProgram = null;
            if (eWalletPrograms.length == 1) {
                selectedProgram = eWalletPrograms[0];
            } else {
                const { confirmed, payload } = await this.showPopup('SelectionPopup', {
                    title: this.env._t('Refund with eWallet'),
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
                const eWalletProduct = this.env.pos.db.get_product_by_id(selectedProgram.trigger_product_ids[0]);
                order.add_product(eWalletProduct, {
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
                const { confirmed, payload } = await this.showPopup('SelectionPopup', {
                    title: this.env._t('Use eWallet to pay'),
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
                const result = order._applyReward(eWalletReward.reward, eWalletReward.coupon_id, {});
                if (result !== true) {
                    // Returned an error
                    this.showPopup('ErrorPopup', {
                        title: this.env._t('Error'),
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
            return this.env._t('eWallet Refund');
        } else if (eWalletRewards.length >= 1) {
            return this.env._t('eWallet Pay');
        } else {
            return this.env._t('eWallet');
        }
    }
}
eWalletButton.template = 'point_of_sale.eWalletButton';

ProductScreen.addControlButton({
    component: eWalletButton,
    condition: function () {
        return this.env.pos.programs.filter((p) => p.program_type == 'ewallet').length > 0;
    },
});

Registries.Component.add(eWalletButton);

```

## File: static\src\js\ControlButtons\PromoCodeButton.js

```javascript
/** @odoo-module **/

import PosComponent from 'point_of_sale.PosComponent';
import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';
import { useListener } from "@web/core/utils/hooks";

export class PromoCodeButton extends PosComponent {
    setup() {
        super.setup();
        useListener('click', this.onClick);
    }

    async onClick() {
        let { confirmed, payload: code } = await this.showPopup('TextInputPopup', {
            title: this.env._t('Enter Code'),
            startingValue: '',
            placeholder: this.env._t('Gift card or Discount code'),
        });
        if (confirmed) {
            code = code.trim();
            if (code !== '') {
                this.env.pos.get_order().activateCode(code);
            }
        }
    }
}

PromoCodeButton.template = 'PromoCodeButton';

ProductScreen.addControlButton({
    component: PromoCodeButton,
    condition: function () {
        return this.env.pos.programs.some(p => ['coupons', 'promotion', 'gift_card', 'promo_code', 'next_order_coupons'].includes(p.program_type));
    }
});

Registries.Component.add(PromoCodeButton);

```

## File: static\src\js\ControlButtons\ResetProgramsButton.js

```javascript
/** @odoo-module **/

import PosComponent from 'point_of_sale.PosComponent';
import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';
import { useListener } from "@web/core/utils/hooks";

export class ResetProgramsButton extends PosComponent {
    setup() {
        super.setup();
        useListener('click', this.onClick);
    }

    async onClick() {
        this.env.pos.get_order()._resetPrograms();
    }
}

ResetProgramsButton.template = 'ResetProgramsButton';

ProductScreen.addControlButton({
    component: ResetProgramsButton,
    condition: function () {
        return this.env.pos.programs.some(p => ['coupons', 'promotion'].includes(p.program_type));
    }
});

Registries.Component.add(ResetProgramsButton);

```

## File: static\src\js\ControlButtons\RewardButton.js

```javascript
/** @odoo-module **/

import { Gui } from 'point_of_sale.Gui';
import PosComponent from 'point_of_sale.PosComponent';
import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';
import { useListener } from "@web/core/utils/hooks";

export class RewardButton extends PosComponent {
    setup() {
        super.setup()
        useListener('click', this.onClick);
    }

    /**
     * If rewards are the same, prioritize the one from freeProductRewards.
     * Make sure that the reward is claimable first.
     */
    _mergeFreeProductRewards(freeProductRewards, potentialFreeProductRewards) {
        const result = []
        for (const reward of potentialFreeProductRewards) {
            if (!freeProductRewards.find(item => item.reward.id === reward.reward.id)) {
                result.push(reward);
            }
        }
        return freeProductRewards.concat(result);
    }

    _getPotentialRewards() {
        const order = this.env.pos.get_order();
        // Claimable rewards excluding those from eWallet programs.
        // eWallet rewards are handled in the eWalletButton.
        let rewards = [];
        if (order) {
            const claimableRewards = order.getClaimableRewards();
            rewards = claimableRewards.filter(({ reward }) => reward.program_id.program_type !== 'ewallet');
        }
        const discountRewards = rewards.filter(({ reward }) => reward.reward_type == 'discount');
        const freeProductRewards = rewards.filter(({ reward }) => reward.reward_type == 'product');
        const potentialFreeProductRewards = order.getPotentialFreeProductRewards();
        return discountRewards.concat(this._mergeFreeProductRewards(freeProductRewards, potentialFreeProductRewards));
    }

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
        const order = this.env.pos.get_order();
        order.disabledRewards.delete(reward.id);

        const args = {};
        if (reward.reward_type === 'product' && reward.multi_product) {
            const productsList = reward.reward_product_ids.map((product_id) => ({
                id: product_id,
                label: this.env.pos.db.get_product_by_id(product_id).display_name,
                item: product_id,
            }));
            const { confirmed, payload: selectedProduct } = await this.showPopup('SelectionPopup', {
                title: this.env._t('Please select a product for this reward'),
                list: productsList,
            });
            if (!confirmed) {
                return false;
            }
            args['product'] = selectedProduct;
        }
        if (
            (reward.reward_type == 'product' && reward.program_id.applies_on !== 'both') ||
            (reward.program_id.applies_on == 'both' && potentialQty)
        ) {
            const product = this.env.pos.db.get_product_by_id(args['product'] || reward.reward_product_ids[0]);
            this.trigger(
                'click-product',
                { product, quantity: potentialQty }
            );
            return true;
        } else {
            const result = order._applyReward(reward, coupon_id, args);
            if (result !== true) {
                // Returned an error
                Gui.showNotification(result);
            }
            order._updateRewards();
            return result;
        }
    }

    async onClick() {
        const rewards = this._getPotentialRewards();
        if (rewards.length === 0) {
            await this.showPopup('ErrorPopup', {
                title: this.env._t('No rewards available.'),
                body: this.env._t('There are no rewards claimable for this customer.')
            });
            return false;
        } else if (rewards.length === 1) {
            return this._applyReward(rewards[0].reward, rewards[0].coupon_id, rewards[0].potentialQty);
        } else {
            const rewardsList = rewards.map((reward) => ({
                id: reward.reward.id,
                label: reward.reward.description,
                item: reward,
            }));
            const { confirmed, payload: selectedReward } = await this.showPopup('SelectionPopup', {
                title: this.env._t('Please select a reward'),
                list: rewardsList,
            });
            if (confirmed) {
                return this._applyReward(selectedReward.reward, selectedReward.coupon_id, selectedReward.potentialQty);
            }
        }
        return false;
    }
}

RewardButton.template = 'RewardButton';

ProductScreen.addControlButton({
    component: RewardButton,
    condition: function() {
        return this.env.pos.programs.length > 0;
    }
});

Registries.Component.add(RewardButton);

```

## File: static\src\tours\EWalletProgramTours.js

```javascript
/** @odoo-module **/

import { ErrorPopup } from 'point_of_sale.tour.ErrorPopupTourMethods';
import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { TicketScreen } from 'point_of_sale.tour.TicketScreenTourMethods';
import { Chrome } from 'point_of_sale.tour.ChromeTourMethods';
import { PartnerListScreen } from 'point_of_sale.tour.PartnerListScreenTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

//#region EWalletProgramTour1

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// Topup 50$ for partner_aaa
ProductScreen.do.clickDisplayedProduct('Top-up eWallet');
PosLoyalty.check.orderTotalIs('50.00');
ProductScreen.do.clickPayButton(false);
// If there's no partner, we asked to redirect to the partner list screen.
Chrome.do.confirmPopup();
PartnerListScreen.check.isShown();
PartnerListScreen.do.clickPartner('AAAAAAA');
PosLoyalty.exec.finalizeOrder('Cash');

// Topup 10$ for partner_bbb
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
ProductScreen.exec.addOrderline('Top-up eWallet', '1', '10');
PosLoyalty.check.orderTotalIs('10.00');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('EWalletProgramTour1', { test: true, url: '/pos/web' }, getSteps());

//#endregion

//#region EWalletProgramTour2

const getEWalletText = (suffix) => 'eWallet' + (suffix !== '' ? ` ${suffix}` : '');

startSteps();
ProductScreen.do.clickHomeCategory();
ProductScreen.exec.addOrderline('Whiteboard Pen', '2', '6', '12.00');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Pay') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Pay'));
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');

// Consume partner_bbb's full eWallet.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
ProductScreen.exec.addOrderline('Desk Pad', '6', '6', '36.00');
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Pay') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Pay'));
PosLoyalty.check.orderTotalIs('26.00');
PosLoyalty.exec.finalizeOrder('Cash');

// Switching partners should work.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
ProductScreen.exec.addOrderline('Desk Pad', '2', '19', '38.00');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Pay') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Pay'));
PosLoyalty.check.orderTotalIs('0.00');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
PosLoyalty.check.orderTotalIs('38.00');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Pay') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Pay'));
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');

// Refund with eWallet.
// - Make an order to refund.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
ProductScreen.exec.addOrderline('Whiteboard Pen', '1', '20', '20.00');
PosLoyalty.check.orderTotalIs('20.00');
PosLoyalty.exec.finalizeOrder('Cash');
// - Refund order.
ProductScreen.do.clickRefund();
TicketScreen.check.filterIs('Paid');
TicketScreen.do.selectOrder('-0004');
TicketScreen.check.partnerIs('BBBBBBB');
TicketScreen.do.confirmRefund();
ProductScreen.check.isShown();
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Refund') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Refund'));
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('EWalletProgramTour2', { test: true, url: '/pos/web' }, getSteps());

//#endregion

//#region ExpiredEWalletProgramTour

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAA');
ProductScreen.exec.addOrderline('Whiteboard Pen', '2', '6', '12.00');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
PosLoyalty.do.clickEWalletButton();
ErrorPopup.check.isShown();
ErrorPopup.do.clickConfirm();

Tour.register('ExpiredEWalletProgramTour', { test: true, url: '/pos/web' }, getSteps());

//#endregion

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer("partner_a");
PosLoyalty.check.eWalletButtonState({ highlighted: false });
ProductScreen.exec.addOrderline("product_a", "1");
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText("Pay") });
PosLoyalty.do.clickEWalletButton(getEWalletText("Pay"));
PosLoyalty.check.pointsAwardedAre("100"),
PosLoyalty.exec.finalizeOrder("Cash", "90");
Tour.register("PosLoyaltyPointsEwallet", { test: true, url: "/pos/web" }, getSteps());

```

## File: static\src\tours\GiftCardProgramTours.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { TextInputPopup } from 'point_of_sale.tour.TextInputPopupTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

//#region GiftCardProgramCreateSetTour1
startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Gift Card');
PosLoyalty.check.orderTotalIs('50.00');
PosLoyalty.exec.finalizeOrder('Cash');
Tour.register('GiftCardProgramCreateSetTour1', { test: true, url: '/pos/web' }, getSteps());
//#endregion

//#region GiftCardProgramCreateSetTour2
startSteps();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.do.enterCode('044123456');
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');
Tour.register('GiftCardProgramCreateSetTour2', { test: true, url: '/pos/web' }, getSteps());
//#endregion

//#region GiftCardProgramScanUseTour
startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
// Pay the 5$ gift card.
ProductScreen.do.clickDisplayedProduct('Gift Card');
TextInputPopup.check.isShown();
TextInputPopup.do.inputText('044123456');
TextInputPopup.do.clickConfirm();
PosLoyalty.check.orderTotalIs('5.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Partially use the gift card. (4$)
ProductScreen.exec.addOrderline('Desk Pad', '2', '2', '4.0');
PosLoyalty.do.enterCode('044123456');
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Use the remaining of the gift card. (5$ - 4$ = 1$)
ProductScreen.exec.addOrderline('Whiteboard Pen', '6', '6', '36.0');
PosLoyalty.do.enterCode('044123456');
PosLoyalty.check.orderTotalIs('35.00');
PosLoyalty.exec.finalizeOrder('Cash');
Tour.register('GiftCardProgramScanUseTour', { test: true, url: '/pos/web' }, getSteps());
//#endregion

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Gift Card');
TextInputPopup.check.isShown();
TextInputPopup.do.inputText('044123456');
TextInputPopup.do.clickConfirm();
PosLoyalty.check.orderTotalIs('50.00');
PosLoyalty.exec.finalizeOrder('Cash');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer("partner_a");
ProductScreen.exec.addOrderline("product_a", "1");
PosLoyalty.do.enterCode("044123456");
PosLoyalty.check.orderTotalIs("50.00");
PosLoyalty.check.pointsAwardedAre("100"),
PosLoyalty.exec.finalizeOrder("Cash", "50");
Tour.register("PosLoyaltyPointsGiftcard", { test: true, url: "/pos/web" }, getSteps());

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Gift Card');
TextInputPopup.check.isShown();
TextInputPopup.do.inputText('044123456');
TextInputPopup.do.clickConfirm();
PosLoyalty.check.orderTotalIs('50.00');
PosLoyalty.exec.finalizeOrder('Cash');
ProductScreen.do.clickDisplayedProduct("Test Product A");
PosLoyalty.do.enterCode("044123456");
PosLoyalty.check.orderTotalIs("50.00");
ProductScreen.check.checkTaxAmount("-6.52");
Tour.register("PosLoyaltyGiftCardTaxes", { test: true }, getSteps());

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Gift Card');
TextInputPopup.check.isShown();
TextInputPopup.do.inputText('044123456');
TextInputPopup.do.clickConfirm();
PosLoyalty.check.orderTotalIs('0.00');
ProductScreen.do.pressNumpad("Price 5");
PosLoyalty.check.orderTotalIs('5.00');
PosLoyalty.exec.finalizeOrder('Cash');
Tour.register("PosLoyaltyGiftCardNoPoints", { test: true }, getSteps());

```

## File: static\src\tours\MultipleGiftWalletProgramsTour.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { SelectionPopup } from 'point_of_sale.tour.SelectionPopupTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

const getEWalletText = (suffix) => 'eWallet' + (suffix !== '' ? ` ${suffix}` : '');

startSteps();
// One card for gift_card_1.
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Gift Card');
SelectionPopup.check.hasSelectionItem('gift_card_1');
SelectionPopup.check.hasSelectionItem('gift_card_2');
SelectionPopup.do.clickItem('gift_card_1');
ProductScreen.do.pressNumpad('Price');
ProductScreen.do.pressNumpad('1 0');
PosLoyalty.check.orderTotalIs('10.00');
PosLoyalty.exec.finalizeOrder('Cash');
// One card for gift_card_1.
ProductScreen.do.clickDisplayedProduct('Gift Card');
SelectionPopup.do.clickItem('gift_card_2');
ProductScreen.do.pressNumpad('Price');
ProductScreen.do.pressNumpad('2 0');
PosLoyalty.check.orderTotalIs('20.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Top up ewallet_1 for AAAAAAA.
ProductScreen.do.clickDisplayedProduct('Top-up eWallet');
SelectionPopup.check.hasSelectionItem('ewallet_1');
SelectionPopup.check.hasSelectionItem('ewallet_2');
SelectionPopup.do.clickItem('ewallet_1');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
ProductScreen.do.pressNumpad('Price');
ProductScreen.do.pressNumpad('3 0');
PosLoyalty.check.orderTotalIs('30.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Top up ewallet_2 for AAAAAAA.
ProductScreen.do.clickDisplayedProduct('Top-up eWallet');
SelectionPopup.do.clickItem('ewallet_2');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
ProductScreen.do.pressNumpad('Price');
ProductScreen.do.pressNumpad('4 0');
PosLoyalty.check.orderTotalIs('40.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Top up ewallet_1 for BBBBBBB.
ProductScreen.do.clickDisplayedProduct('Top-up eWallet');
SelectionPopup.do.clickItem('ewallet_1');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('BBBBBBB');
PosLoyalty.check.orderTotalIs('50.00');
PosLoyalty.exec.finalizeOrder('Cash');
// Consume 12$ from ewallet_1 of AAAAAAA.
ProductScreen.exec.addOrderline('Whiteboard Pen', '2', '6', '12.00');
PosLoyalty.check.eWalletButtonState({ highlighted: false });
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAAAAAA');
PosLoyalty.check.eWalletButtonState({ highlighted: true, text: getEWalletText('Pay') });
PosLoyalty.do.clickEWalletButton(getEWalletText('Pay'));
SelectionPopup.check.hasSelectionItem('ewallet_1');
SelectionPopup.check.hasSelectionItem('ewallet_2');
SelectionPopup.do.clickItem('ewallet_1');
PosLoyalty.check.orderTotalIs('0.00');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('MultipleGiftWalletProgramsTour', { test: true, url: '/pos/web' }, getSteps());

```

## File: static\src\tours\PosLoyaltyLoyaltyProgramTour.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// Order1: Generates 2 points.
ProductScreen.exec.addOrderline('Whiteboard Pen', '2');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner AAA');
PosLoyalty.check.orderTotalIs('6.40');
PosLoyalty.exec.finalizeOrder('Cash');

// Order2: Consumes points to get free product.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner AAA');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '1.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '2.00');
// At this point, Test Partner AAA has 4 points.
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '3.00');
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.orderTotalIs('6.40');
PosLoyalty.exec.finalizeOrder('Cash');

// Order3: Generate 4 points.
// - Initially gets free product, but was removed automatically by changing the
//   number of items to zero.
// - It's impossible to checked here if the free product reward is really removed
//   so we check in the backend the number of orders that consumed the reward.
ProductScreen.exec.addOrderline('Whiteboard Pen', '4');
PosLoyalty.check.orderTotalIs('12.80');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner AAA');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.pressNumpad('Backspace');
// At this point, the reward line should have been automatically removed
// because there is not enough points to purchase it. Unfortunately, we
// can't check that here.
PosLoyalty.check.orderTotalIs('0.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '1.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '2.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '3.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '4.00');
PosLoyalty.check.isRewardButtonHighlighted(true);

PosLoyalty.check.orderTotalIs('12.80');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyLoyaltyProgram1', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();

// Order1: Immediately set the customer to Test Partner AAA which has 4 points.
// - He has enough points to purchase a free product but since there is still
//   no product in the order, reward button should not yet be highlighted.
// - Furthermore, clicking the reward product should not add it as reward product.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner AAA');
// No item in the order, so reward button is off.
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.orderTotalIs('3.20');
PosLoyalty.exec.finalizeOrder('Cash');

// Order2: Generate 4 points for Test Partner CCC.
// - Reference: Order2_CCC
// - But set Test Partner BBB first as the customer.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner BBB');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '1.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '2.00');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '3.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '4.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner CCC');
PosLoyalty.check.customerIs('Test Partner CCC');
PosLoyalty.check.orderTotalIs('12.80');
PosLoyalty.exec.finalizeOrder('Cash');

// Order3: Generate 3 points for Test Partner BBB.
// - Reference: Order3_BBB
// - But set Test Partner CCC first as the customer.
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner CCC');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.exec.addOrderline('Whiteboard Pen', '3');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner BBB');
PosLoyalty.check.customerIs('Test Partner BBB');
PosLoyalty.check.orderTotalIs('9.60');
PosLoyalty.exec.finalizeOrder('Cash');

// Order4: Should not have reward because the customer will be removed.
// - Reference: Order4_no_reward
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
ProductScreen.check.selectedOrderlineHas('Whiteboard Pen', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner CCC');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
ProductScreen.do.clickPartnerButton();
// This deselects the customer.
PosLoyalty.do.unselectPartner();
PosLoyalty.check.customerIs('Customer');
PosLoyalty.check.orderTotalIs('6.40');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyLoyaltyProgram2', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// Generates 10.2 points and use points to get the reward product with zero sale price
ProductScreen.exec.addOrderline('Desk Organizer', '2');
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner AAA');

// At this point, the free_product program is triggered.
// The reward button should be highlighted.
PosLoyalty.check.isRewardButtonHighlighted(true);

PosLoyalty.do.clickRewardButton();
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '0.0', '1.00');

PosLoyalty.check.orderTotalIs('10.2');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyLoyaltyProgram3', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
ProductScreen.exec.addOrderline('Test Product 1', '1.00', '100');
ProductScreen.check.totalAmountIs('80.00');

Tour.register('PosLoyaltyPromotion', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// Generates 10.2 points and use points to get the reward product with zero sale price
ProductScreen.exec.addOrderline('Desk Organizer', '3');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyNextOrderCouponExpirationDate', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('Test Partner');

ProductScreen.exec.addOrderline('Desk Organizer', '1');
ProductScreen.exec.addOrderline('Whiteboard Pen', '1');

PosLoyalty.do.clickRewardButton();

PosLoyalty.check.orderTotalIs('5.10');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyDontGrantPointsForRewardOrderLines', { test: true, url: '/pos/web' }, getSteps());

```

## File: static\src\tours\PosLoyaltyRewardButtonTour.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { SelectionPopup } from 'point_of_sale.tour.SelectionPopupTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.exec.addOrderline('Desk Organizer', '2');

// At this point, the free_product program is triggered.
// The reward button should be highlighted.
PosLoyalty.check.isRewardButtonHighlighted(true);
// Since the reward button is highlighted, clicking the reward product should be added as reward.
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '3.00');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-5.10', '1.00');
// In the succeeding 2 clicks on the product, it is considered as a regular product.
// In the third click, the product will be added as reward.
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '6.00');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-10.20', '2.00');


ProductScreen.do.clickDisplayedProduct('Desk Organizer');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.orderTotalIs('25.50');
// Finalize order that consumed a reward.
PosLoyalty.exec.finalizeOrder('Cash');

ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '1.00');
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '2.00');
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-5.10', '1.00');
ProductScreen.do.pressNumpad('Backspace');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '0.00');
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '1.00');
ProductScreen.do.clickDisplayedProduct('Desk Organizer');
ProductScreen.check.selectedOrderlineHas('Desk Organizer', '2.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
// Finalize order but without the reward.
// This step is important. When syncing the order, no reward should be synced.
PosLoyalty.check.orderTotalIs('10.20');
PosLoyalty.exec.finalizeOrder('Cash');


ProductScreen.exec.addOrderline('Magnetic Board', '2');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickDisplayedProduct('Magnetic Board');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
ProductScreen.do.clickOrderline('Magnetic Board', '3.00');
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '3.00');
ProductScreen.do.pressNumpad('6');
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '6.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-6.40', '2.00');
// Finalize order that consumed a reward.
PosLoyalty.check.orderTotalIs('11.88');
PosLoyalty.exec.finalizeOrder('Cash');

ProductScreen.exec.addOrderline('Magnetic Board', '6');
ProductScreen.do.clickDisplayedProduct('Whiteboard Pen');
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(true);

ProductScreen.do.clickOrderline('Magnetic Board', '6.00');
ProductScreen.do.pressNumpad('Backspace');
// At this point, the reward should have been removed.
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '0.00');
ProductScreen.do.clickDisplayedProduct('Magnetic Board');
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '1.00');
ProductScreen.do.clickDisplayedProduct('Magnetic Board');
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '2.00');
ProductScreen.do.clickDisplayedProduct('Magnetic Board');
ProductScreen.check.selectedOrderlineHas('Magnetic Board', '3.00');
PosLoyalty.check.hasRewardLine('Free Product - Whiteboard Pen', '-3.20', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);

PosLoyalty.check.orderTotalIs('5.94');
PosLoyalty.exec.finalizeOrder('Cash');

// Promotion: 2 items of shelves, get desk_pad/monitor_stand free
// This is the 5th order.
ProductScreen.do.clickDisplayedProduct('Wall Shelf Unit');
ProductScreen.check.selectedOrderlineHas('Wall Shelf Unit', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.do.clickDisplayedProduct('Small Shelf');
ProductScreen.check.selectedOrderlineHas('Small Shelf', '1.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
// Click reward product. Should be automatically added as reward.
ProductScreen.do.clickDisplayedProduct('Desk Pad');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.hasRewardLine('Free Product', '-1.98', '1.00');
// Remove the reward line. The next steps will check if cashier
// can select from the different reward products.
ProductScreen.do.pressNumpad('Backspace');
ProductScreen.do.pressNumpad('Backspace');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
SelectionPopup.check.hasSelectionItem('Monitor Stand');
SelectionPopup.check.hasSelectionItem('Desk Pad');
SelectionPopup.do.clickItem('Desk Pad');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.check.hasRewardLine('Free Product', '-1.98', '1.00');
ProductScreen.do.pressNumpad('Backspace');
ProductScreen.do.pressNumpad('Backspace');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.claimReward('Monitor Stand');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.check.selectedOrderlineHas('Monitor Stand', '1.00', '3.19');
PosLoyalty.check.hasRewardLine('Free Product', '-3.19', '1.00');
PosLoyalty.check.orderTotalIs('4.81');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyFreeProductTour', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
ProductScreen.exec.addOrderline('Test Product A', '1');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
PosLoyalty.check.hasRewardLine('Free Product - Test Product A', '-11.50', '1.00');

Tour.register('PosLoyaltyFreeProductTour2', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct('Test Product A');
ProductScreen.check.selectedOrderlineHas('Test Product A', '1.00', '40.00');
ProductScreen.do.clickDisplayedProduct('Test Product B');
ProductScreen.check.selectedOrderlineHas('Test Product B', '1.00', '40.00');
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("$ 10 per order on specific products");
PosLoyalty.check.hasRewardLine('$ 10 per order on specific products', '-10.00', '1.00');
PosLoyalty.check.orderTotalIs('70.00');
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("$ 10 per order on specific products");
PosLoyalty.check.orderTotalIs('60.00');
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("$ 30 per order on specific products");
PosLoyalty.check.hasRewardLine('$ 30 per order on specific products', '-30.00', '1.00');
PosLoyalty.check.orderTotalIs('30.00');

Tour.register('PosLoyaltySpecificDiscountTour', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct('Test Product A');
ProductScreen.do.clickDisplayedProduct('Test Product C');
PosLoyalty.check.orderTotalIs('130.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
PosLoyalty.check.orderTotalIs('130.00');

Tour.register('PosLoyaltySpecificDiscountWithFreeProductTour', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.selectedOrderlineHas('Product A', '1.00', '15.00');
PosLoyalty.check.orderTotalIs('15.00');

ProductScreen.do.clickDisplayedProduct('Product B');
ProductScreen.check.selectedOrderlineHas('Product B', '1.00', '50.00');
PosLoyalty.check.orderTotalIs('40.00');

Tour.register('PosLoyaltySpecificDiscountWithRewardProductDomainTour', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.selectedOrderlineHas('Product A', '1.00', '15.00');
PosLoyalty.check.orderTotalIs('15.00');

ProductScreen.do.clickDisplayedProduct('Product B');
ProductScreen.check.selectedOrderlineHas('Product B', '1.00', '50.00');
PosLoyalty.check.orderTotalIs('40.00');

Tour.register('PosLoyaltySpecificDiscountCategoryTour', { test: true, url: '/pos/web' }, getSteps());

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct("Desk Organizer");
ProductScreen.do.clickDisplayedProduct("Desk Organizer");
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("product_a");
PosLoyalty.check.hasRewardLine("Free Product", "-2", "1.00");
PosLoyalty.check.isRewardButtonHighlighted(false);

ProductScreen.do.clickDisplayedProduct("Desk Organizer");
ProductScreen.do.clickDisplayedProduct("Desk Organizer");
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("product_b");
PosLoyalty.check.hasRewardLine("Free Product", "-5", "1.00");
PosLoyalty.check.isRewardButtonHighlighted(false);

ProductScreen.do.clickDisplayedProduct("Desk Organizer");
ProductScreen.do.clickDisplayedProduct("Desk Organizer");
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
SelectionPopup.do.clickItem("product_b");
PosLoyalty.check.hasRewardLine("Free Product", "-10", "2.00");
PosLoyalty.check.isRewardButtonHighlighted(false);

Tour.register("PosLoyaltyRewardProductTag", { test: true, url: "/pos/web" }, getSteps());

startSteps();
ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();
ProductScreen.do.clickDisplayedProduct('Product A');
PosLoyalty.do.enterCode('563412');
PosLoyalty.check.hasRewardLine('10% on your order', '-1.50');

Tour.register("test_loyalty_on_order_with_fixed_tax", { test: true, url: "/pos/web" }, getSteps());

```

## File: static\src\tours\PosLoyaltyTour.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

// --- PoS Loyalty Tour Basic Part 1 ---
// Generate coupons for PosLoyaltyTour2.
startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// basic order
// just accept the automatically applied promo program
// applied programs:
//   - on cheapest product
ProductScreen.exec.addOrderline('Whiteboard Pen', '5');
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-2.88');
PosLoyalty.do.selectRewardLine('on the cheapest product');
PosLoyalty.check.orderTotalIs('13.12');
PosLoyalty.exec.finalizeOrder('Cash');

// remove the reward from auto promo program
// no applied programs
ProductScreen.exec.addOrderline('Whiteboard Pen', '6');
PosLoyalty.check.hasRewardLine('on the cheapest product', '-2.88');
PosLoyalty.check.orderTotalIs('16.32');
PosLoyalty.exec.removeRewardLine('90% on the cheapest product');
PosLoyalty.check.orderTotalIs('19.2');
PosLoyalty.exec.finalizeOrder('Cash');

// order with coupon code from coupon program
// applied programs:
//   - coupon program
ProductScreen.exec.addOrderline('Desk Organizer', '9');
PosLoyalty.check.hasRewardLine('on the cheapest product', '-4.59');
PosLoyalty.exec.removeRewardLine('90% on the cheapest product');
PosLoyalty.check.orderTotalIs('45.90');
PosLoyalty.do.enterCode('invalid_code');
PosLoyalty.do.enterCode('1234');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-15.30');
PosLoyalty.exec.finalizeOrder('Cash');

// Use coupon but eventually remove the reward
// applied programs:
//   - on cheapest product
ProductScreen.exec.addOrderline('Letter Tray', '4');
ProductScreen.exec.addOrderline('Desk Organizer', '9');
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-4.75');
PosLoyalty.check.orderTotalIs('62.27');
PosLoyalty.do.enterCode('5678');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-15.30');
PosLoyalty.check.orderTotalIs('46.97');
PosLoyalty.exec.removeRewardLine('Free Product');
PosLoyalty.check.orderTotalIs('62.27');
PosLoyalty.exec.finalizeOrder('Cash');

// specific product discount
// applied programs:
//   - on cheapest product
//   - on specific products
ProductScreen.exec.addOrderline('Magnetic Board', '10') // 1.98
ProductScreen.exec.addOrderline('Desk Organizer', '3') // 5.1
ProductScreen.exec.addOrderline('Letter Tray', '4') // 4.8 tax 10%
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-1.78')
PosLoyalty.check.orderTotalIs('54.44')
PosLoyalty.do.enterCode('promocode')
PosLoyalty.check.hasRewardLine('50% on specific products', '-16.66') // 17.55 - 1.78*0.5
PosLoyalty.check.orderTotalIs('37.78')
PosLoyalty.exec.finalizeOrder('Cash')

Tour.register('PosLoyaltyTour1', { test: true, url: '/pos/web' }, getSteps());

// --- PoS Loyalty Tour Basic Part 2 ---
// Using the coupons generated from PosLoyaltyTour1.
startSteps();

ProductScreen.do.clickHomeCategory();

// Test that global discount and cheapest product discounts can be accumulated.
// Applied programs:
//   - global discount
//   - on cheapest discount
ProductScreen.exec.addOrderline('Desk Organizer', '10'); // 5.1
PosLoyalty.check.hasRewardLine('on the cheapest product', '-4.59');
ProductScreen.exec.addOrderline('Letter Tray', '4'); // 4.8 tax 10%
PosLoyalty.check.hasRewardLine('on the cheapest product', '-4.75');
PosLoyalty.do.enterCode('123456');
PosLoyalty.check.hasRewardLine('10% on your order', '-5.10');
PosLoyalty.check.hasRewardLine('10% on your order', '-1.64');
PosLoyalty.check.orderTotalIs('60.63'); //SUBTOTAL
PosLoyalty.exec.finalizeOrder('Cash');

// Scanning coupon twice.
// Also apply global discount on top of free product to check if the
// calculated discount is correct.
// Applied programs:
//  - coupon program (free product)
//  - global discount
//  - on cheapest discount
ProductScreen.exec.addOrderline('Desk Organizer', '11'); // 5.1 per item
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-4.59');
PosLoyalty.check.orderTotalIs('51.51');
// add global discount and the discount will be replaced
PosLoyalty.do.enterCode('345678');
PosLoyalty.check.hasRewardLine('10% on your order', '-5.15');
// add free product coupon (for qty=11, free=4)
// the discount should change after having free products
// it should go back to cheapest discount as it is higher
PosLoyalty.do.enterCode('5678');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-20.40');
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-4.59');
// set quantity to 18
// free qty stays the same since the amount of points on the card only allows for 4 free products
ProductScreen.do.pressNumpad('Backspace 8')
PosLoyalty.check.hasRewardLine('10% on your order', '-6.68');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-20.40');
// scan the code again and check notification
PosLoyalty.do.enterCode('5678');
PosLoyalty.check.orderTotalIs('60.13');
PosLoyalty.exec.finalizeOrder('Cash');

// Specific products discount (with promocode) and free product (1357)
// Applied programs:
//   - discount on specific products
//   - free product
ProductScreen.exec.addOrderline('Desk Organizer', '6'); // 5.1 per item
PosLoyalty.check.hasRewardLine('on the cheapest product', '-4.59');
PosLoyalty.exec.removeRewardLine('90% on the cheapest product');
PosLoyalty.do.enterCode('promocode');
PosLoyalty.check.hasRewardLine('50% on specific products', '-15.30');
PosLoyalty.do.enterCode('1357');
PosLoyalty.check.hasRewardLine('Free Product - Desk Organizer', '-10.20');
PosLoyalty.check.hasRewardLine('50% on specific products', '-10.20');
PosLoyalty.check.orderTotalIs('10.20');
PosLoyalty.exec.finalizeOrder('Cash');

// Check reset program
// Enter two codes and reset the programs.
// The codes should be checked afterwards. They should return to new.
// Applied programs:
//   - cheapest product
ProductScreen.exec.addOrderline('Monitor Stand', '6'); // 3.19 per item
PosLoyalty.do.enterCode('098765');
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-2.87');
PosLoyalty.check.hasRewardLine('10% on your order', '-1.63');
PosLoyalty.check.orderTotalIs('14.64');
PosLoyalty.exec.removeRewardLine('90% on the cheapest product');
PosLoyalty.check.hasRewardLine('10% on your order', '-1.91');
PosLoyalty.check.orderTotalIs('17.23');
PosLoyalty.do.resetActivePrograms();
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-2.87');
PosLoyalty.check.orderTotalIs('16.27');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyTour2', { test: true, url: '/pos/web' }, getSteps());

// --- PoS Loyalty Tour Basic Part 3 ---

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickDisplayedProduct('Promo Product');
PosLoyalty.check.orderTotalIs('34.50');
ProductScreen.do.clickDisplayedProduct('Product B');
PosLoyalty.check.hasRewardLine('100% on specific products', '25.00');
ProductScreen.do.clickDisplayedProduct('Product A');
PosLoyalty.check.hasRewardLine('100% on specific products', '15.00');
PosLoyalty.check.orderTotalIs('34.50');
ProductScreen.do.clickDisplayedProduct('Product A');
PosLoyalty.check.hasRewardLine('100% on specific products', '21.82');
PosLoyalty.check.hasRewardLine('100% on specific products', '18.18');
PosLoyalty.check.orderTotalIs('49.50');


Tour.register('PosLoyaltyTour3', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.exec.addOrderline('Test Product 1', '1');
ProductScreen.exec.addOrderline('Test Product 2', '1');
ProductScreen.do.clickPricelistButton();
ProductScreen.do.selectPriceList('Public Pricelist');
PosLoyalty.do.enterCode('abcda');
PosLoyalty.check.orderTotalIs('0.00');
ProductScreen.do.clickPricelistButton();
ProductScreen.do.selectPriceList('Test multi-currency');
PosLoyalty.check.orderTotalIs('0.00');

Tour.register('PosLoyaltyTour4', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();

ProductScreen.exec.addOrderline('Test Product 1', '1.00', '100');
PosLoyalty.do.clickDiscountButton();
PosLoyalty.do.clickConfirmButton();
ProductScreen.check.totalAmountIs('92.00');

Tour.register('PosLoyaltyTour5', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
ProductScreen.do.clickDisplayedProduct('Test Product A');
PosLoyalty.do.clickRewardButton();
ProductScreen.check.totalAmountIs('139');

Tour.register('PosLoyaltyTour6', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.exec.addOrderline('Test Product', '1');
PosLoyalty.check.orderTotalIs('100');
PosLoyalty.do.enterCode('abcda');
PosLoyalty.check.orderTotalIs('90');

Tour.register('PosLoyaltyTour7', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickDisplayedProduct('Product B');
ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.totalAmountIs('50.00');

Tour.register('PosLoyaltyTour8', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
ProductScreen.do.clickDisplayedProduct('Product B');
ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.totalAmountIs('210.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
ProductScreen.check.totalAmountIs('205.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
ProductScreen.check.totalAmountIs('200.00');

Tour.register('PosLoyaltyTour9', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
ProductScreen.do.clickDisplayedProduct('Product Test');
ProductScreen.check.totalAmountIs('1.00');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.claimReward('Free Product B');
PosLoyalty.check.hasRewardLine('Free Product B', '-1.00');
ProductScreen.check.totalAmountIs('1.00');
PosLoyalty.check.isRewardButtonHighlighted(false);

Tour.register('PosLoyaltyTour10', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
PosLoyalty.check.customerIs('AAA Partner');
ProductScreen.exec.addOrderline('Product Test', '3');
ProductScreen.check.totalAmountIs('150.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyTour11.1', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer('AAA Partner');
PosLoyalty.check.customerIs('AAA Partner');
ProductScreen.do.clickDisplayedProduct('Product Test');
ProductScreen.check.totalAmountIs('50.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
PosLoyalty.do.enterCode('123456');
PosLoyalty.check.isRewardButtonHighlighted(true);
PosLoyalty.do.clickRewardButton();
PosLoyalty.check.hasRewardLine('Free Product', '-3.00');
PosLoyalty.check.isRewardButtonHighlighted(false);
ProductScreen.check.totalAmountIs('50.00');
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyTour11.2', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

ProductScreen.exec.addOrderline('Free Product A', '2');
ProductScreen.do.clickDisplayedProduct('Free Product A');
ProductScreen.check.totalAmountIs('2.00');
PosLoyalty.check.hasRewardLine('Free Product', '-1.00');

ProductScreen.exec.addOrderline('Free Product B', '2');
ProductScreen.do.clickDisplayedProduct('Free Product B');
ProductScreen.check.totalAmountIs('4.00');
PosLoyalty.check.hasRewardLine('Free Product', '-2.00');

ProductScreen.exec.addOrderline('Free Product B', '5');
ProductScreen.do.clickDisplayedProduct('Free Product B');
ProductScreen.check.totalAmountIs('6.00');
PosLoyalty.check.hasRewardLine('Free Product', '-3.00');

Tour.register('PosLoyaltyTour12', { test: true, url: '/pos/web' }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.selectedOrderlineHas('Product A', '1.00', '20.00');
PosLoyalty.check.orderTotalIs('20.00');

ProductScreen.do.clickDisplayedProduct('Product B');
ProductScreen.check.selectedOrderlineHas('Product B', '1.00', '30.00');
PosLoyalty.check.orderTotalIs('50.00');

ProductScreen.do.clickDisplayedProduct('Product A');
ProductScreen.check.selectedOrderlineHas('Product A', '2.00', '40.00');
PosLoyalty.check.orderTotalIs('66.00');

Tour.register('PosLoyaltyMinAmountAndSpecificProductTour', {test: true, url: '/pos/web'}, getSteps());

function createOrderCoupon(totalAmount, couponName, couponAmount, loyaltyPoints) {
    return [
        ProductScreen.do.confirmOpeningPopup(),
        ProductScreen.do.clickHomeCategory(),
        ProductScreen.do.clickPartnerButton(),
        ProductScreen.do.clickCustomer("partner_a"),
        ProductScreen.exec.addOrderline("product_a", "1"),
        ProductScreen.exec.addOrderline("product_b", "1"),
        PosLoyalty.do.enterCode("promocode"),
        PosLoyalty.check.hasRewardLine(`${couponName}`, `${couponAmount}`),
        PosLoyalty.check.orderTotalIs(`${totalAmount}`),
        PosLoyalty.check.pointsAwardedAre(`${loyaltyPoints}`),
        PosLoyalty.exec.finalizeOrder("Cash"),
    ];
}

startSteps();
createOrderCoupon("135.00", "10% on your order", "-15.00", "135");
Tour.register("PosLoyaltyPointsDiscountNoDomainProgramNoDomain", { test: true, url: "/pos/web" }, getSteps());

startSteps();
createOrderCoupon("135.00", "10% on your order", "-15.00", "100");
Tour.register("PosLoyaltyPointsDiscountNoDomainProgramDomain", { test: true, url: "/pos/web" }, getSteps());

startSteps();
createOrderCoupon("140.00", "10% on food", "-10.00", "90");
Tour.register("PosLoyaltyPointsDiscountWithDomainProgramDomain", { test: true, url: "/pos/web" }, getSteps());

startSteps();
ProductScreen.do.confirmOpeningPopup(),
ProductScreen.do.clickHomeCategory(),
ProductScreen.do.clickPartnerButton(),
ProductScreen.do.clickCustomer("partner_a"),
ProductScreen.exec.addOrderline("product_a", "1"),
PosLoyalty.check.hasRewardLine('10% on your order', '-10.00');
PosLoyalty.check.orderTotalIs('90'),
PosLoyalty.check.pointsAwardedAre("90"),
PosLoyalty.exec.finalizeOrder("Cash", "90"),
Tour.register("PosLoyaltyPointsGlobalDiscountProgramNoDomain", { test: true, url: "/pos/web" }, getSteps());

startSteps();

ProductScreen.do.clickHomeCategory();
ProductScreen.do.confirmOpeningPopup();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer("partner_a");

ProductScreen.do.clickDisplayedProduct('Test Product A');
PosLoyalty.check.checkNoClaimableRewards();
ProductScreen.check.selectedOrderlineHas('Test Product A', '1.00', '100.00');
PosLoyalty.exec.finalizeOrder("Cash");

Tour.register('PosLoyaltyArchivedRewardProductsInactive', {test: true, url: '/pos/web'}, getSteps());

startSteps();

ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer("partner_a");

ProductScreen.do.clickDisplayedProduct('Test Product A');
PosLoyalty.check.isRewardButtonHighlighted(true);
ProductScreen.check.selectedOrderlineHas('Test Product A', '1.00', '100.00');
PosLoyalty.exec.finalizeOrder("Cash");

Tour.register('PosLoyaltyArchivedRewardProductsActive', {test: true, url: '/pos/web'}, getSteps());

startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickPartnerButton();
ProductScreen.do.clickCustomer("partner_a");

ProductScreen.exec.addOrderline("Test Product A", "5"),
ProductScreen.do.clickDisplayedProduct('Test Product B');
PosLoyalty.check.hasRewardLine('10% on your order', '-3.00');
PosLoyalty.check.hasRewardLine('10% on Test Product B', '-0.45');
PosLoyalty.exec.finalizeOrder("Cash");

Tour.register('PosLoyalty2DiscountsSpecificGlobal', {test: true, url: '/pos/web'}, getSteps());

```

## File: static\src\tours\PosLoyaltyTourMethods.js

```javascript
odoo.define('pos_loyalty.tour.PosCouponTourMethods', function (require) {
    'use strict';

    const { createTourMethods } = require('point_of_sale.tour.utils');
    const { Do: ProductScreenDo } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { Do: PaymentScreenDo, Check: PaymentScreenCheck } = require('point_of_sale.tour.PaymentScreenTourMethods');
    const { Do: ReceiptScreenDo } = require('point_of_sale.tour.ReceiptScreenTourMethods');
    const { Do: ChromeDo } = require('point_of_sale.tour.ChromeTourMethods');

    const ProductScreen = { do: new ProductScreenDo() };
    const PaymentScreen = { do: new PaymentScreenDo(), check: new PaymentScreenCheck() };
    const ReceiptScreen = { do: new ReceiptScreenDo() };
    const Chrome = { do: new ChromeDo() };

    class Do {
        selectRewardLine(rewardName) {
            return [
                {
                    content: 'select reward line',
                    trigger: `.orderline.program-reward .product-name:contains("${rewardName}")`,
                },
                {
                    content: 'check reward line if selected',
                    trigger: `.orderline.selected.program-reward .product-name:contains("${rewardName}")`,
                    run: function () {}, // it's a check
                },
            ];
        }
        enterCode(code) {
            const steps = [
                {
                    content: 'open code input dialog',
                    trigger: '.control-button:contains("Enter Code")',
                },
                {
                    content: `enter code value: ${code}`,
                    trigger: '.popup-textinput input[type="text"]',
                    run: `text ${code}`,
                },
                {
                    content: 'confirm inputted code',
                    trigger: '.popup-textinput .button.confirm',
                },
                {
                    content: 'verify popup is closed',
                    trigger: 'body:not(:has(.popup-textinput))',
                    run: function () {}, // it's a check
                },
            ];
            return steps;
        }
        resetActivePrograms() {
            return [
                {
                    content: 'open code input dialog',
                    trigger: '.control-button:contains("Reset Programs")',
                },
            ];
        }
        clickRewardButton() {
            return [
                {
                    content: 'open reward dialog',
                    trigger: '.control-button:contains("Reward")',
                },
            ];
        }
        clickEWalletButton(text = 'eWallet') {
            return [{ trigger: `.control-button:contains("${text}")` }];
        }
        claimReward(rewardName) {
            return [
                {
                    content: 'open reward dialog',
                    trigger: '.control-button:contains("Reward")',
                },
                {
                    content: 'select reward',
                    trigger: `.selection-item:contains("${rewardName}")`,
                }
            ];
        }
        unselectPartner() {
            return [{ trigger: '.unselect-tag' }];
        }
        clickDiscountButton() {
            return [
                {
                    content: 'click discount button',
                    trigger: '.js_discount',
                },
            ];
        }
        clickConfirmButton() {
            return [
                {
                    content: 'click confirm button',
                    trigger: '.button.confirm',
                },
            ];
        }
    }

    class Check {
        hasRewardLine(rewardName, amount, qty) {
            const steps = [
                {
                    content: 'check if reward line is there',
                    trigger: `.orderline.program-reward span.product-name:contains("${rewardName}")`,
                    run: function () {},
                },
                {
                    content: 'check if the reward price is correct',
                    trigger: `.orderline.program-reward span.price:contains("${amount}")`,
                    run: function () {},
                },
            ];
            if (qty) {
                steps.push({
                    content: 'check if the reward qty is correct',
                    trigger: `.order .orderline.program-reward .product-name:contains("${rewardName}") ~ .info-list em:contains("${qty}")`,
                    run: function () {},
                });
            }
            return steps;
        }
        orderTotalIs(total_str) {
            return [
                {
                    content: 'order total contains ' + total_str,
                    trigger: '.order .total .value:contains("' + total_str + '")',
                    run: function () {}, // it's a check
                },
            ];
        }
        checkNoClaimableRewards() {
            return [
                {
                    content: 'check that no reward can be claimed',
                    trigger: ".control-button:contains('Reward'):not(.highlight)",
                    run: function () {}, // it's a check
                }
            ]
        }
        isRewardButtonHighlighted(isHighlighted) {
            return [
                {
                    trigger: isHighlighted
                        ? '.control-button.highlight:contains("Reward")'
                        : '.control-button:contains("Reward"):not(:has(.highlight))',
                    run: function () {}, // it's a check
                },
            ];
        }
        eWalletButtonState({ highlighted, text = 'eWallet' }) {
            return [
                {
                    trigger: highlighted
                        ? `.control-button.highlight:contains("${text}")`
                        : `.control-button:contains("${text}"):not(:has(.highlight))`,
                    run: function () {}, // it's a check
                },
            ];
        }
        customerIs(name) {
            return [
                {
                    trigger: `.actionpad button.set-partner:contains("${name}")`,
                    run: function () {},
                }
            ]
        }
        pointsAwardedAre(points_str) {
            return [
                {
                    content: 'loyalty points awarded ' + points_str,
                    trigger: '.loyalty-points-won.value:contains("' + points_str + '")',
                    run: function () {}, // it's a check
                },
            ];
        }
    }

    class Execute {
        constructor() {
            this.do = new Do();
            this.check = new Check();
        }
        finalizeOrder(paymentMethod, amount) {
            const actions = [
                ...ProductScreen.do.clickPayButton(),
                ...PaymentScreen.do.clickPaymentMethod(paymentMethod),
            ];
            if (amount) {
                actions.push(...PaymentScreen.do.pressNumpad([...amount].join(' ')));
            } else {
                actions.push(
                    ...PaymentScreen.check.remainingIs('0.0'),
                    ...PaymentScreen.check.changeIs('0.0'),
                )
            }
            actions.push(
                ...PaymentScreen.do.clickValidate(),
                ...ReceiptScreen.do.clickNextOrder(),
            );
            return actions;
        }
        removeRewardLine(name) {
            return [
                ...this.do.selectRewardLine(name),
                ...ProductScreen.do.pressNumpad('Backspace'),
                ...Chrome.do.confirmPopup(),
            ];
        }
    }

    return createTourMethods('PosLoyalty', Do, Check, Execute);
});

```

## File: static\src\tours\PosLoyaltyValidityTour.js

```javascript
/** @odoo-module **/

import { PosLoyalty } from 'pos_loyalty.tour.PosCouponTourMethods';
import { ProductScreen } from 'point_of_sale.tour.ProductScreenTourMethods';
import { getSteps, startSteps } from 'point_of_sale.tour.utils';
import Tour from 'web_tour.tour';

// First tour should not get any automatic rewards
startSteps();

ProductScreen.do.confirmOpeningPopup();
ProductScreen.do.clickHomeCategory();

// Not valid -> date
ProductScreen.exec.addOrderline('Whiteboard Pen', '5');
PosLoyalty.check.checkNoClaimableRewards();
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyValidity1', { test: true, url: '/pos/web' }, getSteps());

// Second tour
startSteps();

ProductScreen.do.clickHomeCategory();

// Valid
ProductScreen.exec.addOrderline('Whiteboard Pen', '5');
PosLoyalty.check.hasRewardLine('90% on the cheapest product', '-2.88');
PosLoyalty.exec.finalizeOrder('Cash');

// Not valid -> usage
ProductScreen.exec.addOrderline('Whiteboard Pen', '5');
PosLoyalty.check.checkNoClaimableRewards();
PosLoyalty.exec.finalizeOrder('Cash');

Tour.register('PosLoyaltyValidity2', { test: true, url: '/pos/web' }, getSteps());

```

## File: static\src\xml\Orderline.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_loyalty.Orderline" t-inherit="point_of_sale.Orderline" t-inherit-mode="extension" owl="1">
        <xpath expr="//ul[hasclass('info-list')]" position="attributes">
            <attribute name="t-if">!_isGiftCardOrEWalletReward()</attribute>
        </xpath>
        <xpath expr="//ul[hasclass('info-list')]" position="after">
            <ul t-else="" class="info-list">
                Current Balance: <span t-esc="_getGiftCardOrEWalletBalance()"/>
            </ul>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_coupon.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('pos-receipt')]//div[hasclass('before-footer')]" position="inside">
            <t t-if='receipt.loyaltyStats'>
                <t t-foreach="receipt.loyaltyStats" t-as="_loyaltyStat" t-key="_loyaltyStat.couponId">
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
                <br/>
                <div>Customer <span t-esc='receipt.partner.name' class='pos-receipt-right-align'/></div>
            </t>
            <t t-if="receipt.new_coupon_info and receipt.new_coupon_info.length !== 0">
                <div class="pos-coupon-rewards">
                    <div>------------------------</div>
                    <br/>
                    <div>
                        Coupon Codes
                    </div>
                    <t t-foreach="receipt.new_coupon_info" t-as="coupon_info" t-key="coupon_info.code">
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

## File: static\src\xml\OrderSummary.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="OrderSummary" t-inherit="point_of_sale.OrderSummary" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('summary')]" position="after">
            <div class="summary clearfix">
                <t t-set="_loyaltyStats" t-value="getLoyaltyPoints()"/>
                <t t-foreach="_loyaltyStats" t-as="_loyaltyStat" t-key="_loyaltyStat.couponId">
                    <t t-if="_loyaltyStat.points.won || _loyaltyStat.points.spent">
                        <div class='loyalty-points'>
                            <div class='loyalty-points-title'>
                                <t t-esc="_loyaltyStat.points.name"/>
                            </div>
                            <t t-if='_loyaltyStat.points.balance'>
                                <div class="loyalty-points-balance">
                                    <span class='value'><t t-esc='_loyaltyStat.points.balance'/></span>
                                </div>
                            </t>
                            <div>
                                <t t-if='_loyaltyStat.points.won'>
                                    <span class="value loyalty-points-won">+<t t-esc='_loyaltyStat.points.won'/></span>
                                </t>
                                <t t-if='_loyaltyStat.points.spent'>
                                    <span class="value loyalty-points-spent">-<t t-esc='_loyaltyStat.points.spent'/></span>
                                </t>
                            </div>
                            <div class='loyalty-points-total'>
                                <span class='value'><t t-esc='_loyaltyStat.points.total'/></span>
                            </div>
                        </div>
                    </t>
                    <t t-else="">
                        <div></div>
                    </t>
                </t>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\PartnerLine.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="pos_loyalty.PartnerLine" t-inherit="point_of_sale.PartnerLine" t-inherit-mode="extension" owl="1">
        <xpath expr="//td[hasclass('partner-line-balance')]" position="inside">
            <t t-set="_loyaltyCards" t-value="env.pos.getLoyaltyCards(props.partner)" />
            <t t-foreach="_loyaltyCards" t-as="_loyaltyCard" t-key="_loyaltyCard.id">
                <div class="pos-right-align">
                    <t t-esc="_getLoyaltyPointsRepr(_loyaltyCard)"/>
                </div>
            </t>
        </xpath>
    </t>

 </templates>

```

## File: static\src\xml\ControlButtons\eWalletButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="point_of_sale.eWalletButton" owl="1">
        <t t-set="_order" t-value="env.pos.get_order()" />
        <t t-set="_orderTotal" t-value="_order.get_total_with_tax()" />
        <t t-set="_eWalletPrograms" t-value="_getEWalletPrograms()" />
        <t t-set="_eWalletRewards" t-value="_getEWalletRewards(_order)" />
        <span class="control-button" t-att-class="_shouldBeHighlighted(_orderTotal, _eWalletPrograms, _eWalletRewards) ? 'highlight' : ''" t-on-click="_onClickWalletButton">
            <i class="fa fa-credit-card"></i>
            <span> </span>
            <span><t t-esc="_getText(_orderTotal, _eWalletPrograms, _eWalletRewards)" /></span>
        </span>
    </t>

 </templates>

```

## File: static\src\xml\ControlButtons\PromoCodeButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PromoCodeButton" owl="1">
        <span class="control-button">
            <i class="fa fa-barcode"></i>
            <span> </span>
            <span>Enter Code</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\ControlButtons\ResetProgramsButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="ResetProgramsButton" owl="1">
        <span class="control-button">
            <i class="fa fa-star"></i>
            <span> </span>
            <span>Reset Programs</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\ControlButtons\RewardButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

     <t t-name="RewardButton" owl="1">
        <span class="control-button" t-att-class="hasClaimableRewards() ? 'highlight' : ''">
            <i class="fa fa-star"></i>
            <span> </span>
            <span>Reward</span>
        </span>
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
                <field name="source_pos_order_id" readonly="1" attrs="{'invisible': [('source_pos_order_id', '=', False)]}"/>
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
                <field name="pos_report_print_id" attrs="{'invisible': [('trigger', '!=', 'create')]}"/>
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
                <field name="pos_report_print_id" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}" />
            </field>
            <xpath expr="//label[@for='available_on']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="inside">
                <span class="d-inline-block">
                    <field name="pos_ok" class="w-auto me-0"/>
                    <label for="pos_ok" class="me-3"/>
                </span>
            </xpath>
            <xpath expr="//div[@id='o_loyalty_program_availabilities']" position="after">
                <field name="pos_config_ids" string="Point of Sale" widget="many2many_tags" attrs="{'invisible': [('pos_ok', '=', False)]}" options="{'create': False}" placeholder="All PoS"/>
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
            <xpath expr="//div[@id='loyalty_program_text']" position="after">
                <div class="content-group" attrs="{'invisible': [('module_loyalty', '=', False)]}">
                    <div class="mt16 o_light_label">
                        <field name="pos_gift_card_settings" colspan="4" nolabel="1" widget="radio"/>
                    </div>
                    <div class="mt8">
                        <button name="%(loyalty.loyalty_program_discount_loyalty_action)d" icon="fa-arrow-right" type="action" string="Discount &amp; Loyalty" class="btn-link"/>
                    </div>
                    <div class="mt8">
                        <button name="%(loyalty.loyalty_program_gift_ewallet_action)d" icon="fa-arrow-right" type="action" string="Gift cards &amp; eWallet" class="btn-link"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

