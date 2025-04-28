# Odoo Module: pos_coupon

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
    "name": "Point of Sale Coupons",
    "version": "1.0",
    "category": "Sales/Point Of Sale",
    "sequence": 6,
    "summary": "Use coupons in Point of Sale",
    "description": "",
    "depends": ["coupon", "point_of_sale"],
    "data": [
        "data/mail_template_data.xml",
        'data/default_barcode_patterns.xml',
        "security/ir.model.access.csv",
        "views/coupon_views.xml",
        "views/coupon_program_views.xml",
        "views/pos_config_views.xml",
        "views/res_config_settings_views.xml",
        ],
    "demo": [
        "demo/pos_coupon_demo.xml",
    ],
    "installable": True,
    'assets': {
        'point_of_sale.assets': [
            'pos_coupon/static/src/css/coupon.css',
            'pos_coupon/static/src/js/coupon.js',
            'pos_coupon/static/src/js/Orderline.js',
            'pos_coupon/static/src/js/PaymentScreen.js',
            'pos_coupon/static/src/js/ProductScreen.js',
            'pos_coupon/static/src/js/ActivePrograms.js',
            'pos_coupon/static/src/js/ControlButtons/PromoCodeButton.js',
            'pos_coupon/static/src/js/ControlButtons/ResetProgramsButton.js',
        ],
        'web.assets_tests': [
            'pos_coupon/static/src/js/tours/**/*',
        ],
        'web.assets_qweb': [
            'pos_coupon/static/src/xml/**/*',
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
        <field name="name">Coupon Barcodes</field>
        <field name="barcode_nomenclature_id" ref="barcodes.default_barcode_nomenclature"/>
        <field name="sequence">50</field>
        <field name="type">coupon</field>
        <field name="encoding">any</field>
        <field name="pattern">043</field>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
   <data noupdate="1">
      <record id="pos_coupon.mail_coupon_template" model="mail.template">
         <field name="name">[POS] Coupon: Send by Email</field>
         <field name="model_id" ref="coupon.model_coupon_coupon"/>
         <field name="subject">Your reward coupon from {{ object.program_id.company_id.name }} </field>
         <field name="email_from">{{ object.program_id.company_id.email }}</field>
         <field name="partner_to">{{ object.source_pos_order_id.partner_id.id or object.partner_id.id }}</field>
         <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="width:100%; margin:0px auto;"><tbody>
    <tr><td valign="top" style="text-align: center; font-size: 14px;">
        <t t-if="object.partner_id.name">
            Congratulations <t t-out="object.partner_id.name or ''">Brandon Freeman</t>,<br />
        </t>

        Here is your reward from <t t-out="object.program_id.company_id.name or ''">YourCompany</t>.<br />

        <t t-if="object.program_id.reward_type == 'discount'">
            <t t-if="object.program_id.discount_type == 'fixed_amount'">
                <span style="font-size: 50px; color: #875A7B; font-weight: bold;" t-out="'%s' % format_amount(object.program_id.discount_fixed_amount, object.program_id.currency_id) or ''">$ 10.0</span><br />
                <strong style="font-size: 24px;">off on your next order</strong><br />
            </t>
            <t t-else="">
                <span style="font-size: 50px; color: #875A7B; font-weight: bold;">
                    <t t-out="object.program_id.discount_percentage or ''">10.0</t> %
                </span>
                <t t-if="object.program_id.discount_apply_on == 'specific_products'">
                    <br />
                    <t t-if="len(object.program_id.discount_specific_product_ids) != 1">
                        <t t-set="display_specific_products" t-value="True"/>
                        <strong style="font-size: 24px;">
                            on some products*
                        </strong>
                    </t>
                    <t t-else="">
                        <strong style="font-size: 24px;" t-out="'on %s' % object.program_id.discount_specific_product_ids.name or ''">Chair floor protection</strong>
                    </t>
                </t>
                <t t-elif="object.program_id.discount_apply_on == 'cheapest_product'">
                    <br /><strong style="font-size: 24px;">
                        off on the cheapest product
                    </strong>
                </t>
                <t t-else="">
                    <br /><strong style="font-size: 24px;">
                        off on your next order
                    </strong>
                </t>
                <br />
            </t>
        </t>
        <t t-elif="object.program_id.reward_type == 'product'">
            <span style="font-size: 36px; color: #875A7B; font-weight: bold;" t-out="'get %s free %s' % (object.program_id.reward_product_quantity, object.program_id.reward_product_id.name) or ''">Chair floor protection</span><br />
            <strong style="font-size: 24px;">on your next order</strong><br />
        </t>
        <t t-elif="object.program_id.reward_type == 'free_shipping'">
            <span style="font-size: 36px; color: #875A7B; font-weight: bold;">
                get free shipping
            </span><br />
            <strong style="font-size: 24px;">on your next order</strong><br />
        </t>
    </td></tr>
    <tr style="margin-top: 16px"><td valign="top" style="text-align: center; font-size: 14px;">
        Use this promo code
        <t t-if="object.expiration_date">
            before <t t-out="object.expiration_date or ''">2021-06-05</t>
        </t>
        <p style="margin-top: 16px;">
            <strong style="padding: 16px 8px 16px 8px; border-radius: 3px; background-color: #F1F1F1;" t-out="object.code or ''">13996301932606901095</strong>
        </p>
        <t t-if="object.program_id.rule_min_quantity not in [0, 1]">
            <span style="font-size: 14px;">
                Minimum purchase of <t t-out="object.program_id.rule_min_quantity or ''">10</t> products
            </span><br />
        </t>
        <t t-if="object.program_id.rule_minimum_amount != 0.00">
            <span style="font-size: 14px;">
                Valid for purchase above <t t-out="object.program_id.company_id.currency_id.symbol or ''">$</t><t t-out="'%0.2f' % float(object.program_id.rule_minimum_amount) or ''">10.00</t>
            </span><br />
        </t>
        <t t-if="display_specific_products">
            <span>
                *Valid for following products: <t t-out="', '.join(object.program_id.discount_specific_product_ids.mapped('name')) or ''">Office Chair Black</t>
            </span><br />
        </t>
        <br/>
        Thank you,
        <t t-if="object.source_pos_order_id.user_id.signature">
            <br />
            <t t-out="object.source_pos_order_id.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
    </td></tr>
</tbody></table>
            </field>
            <field name="report_template" ref="coupon.report_coupon_code"/>
            <field name="report_name">Your Coupon Code</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
      </record>
   </data>
</odoo>

```

## File: models\barcode_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields
from odoo.tools.translate import _


class BarcodeRule(models.Model):
    _inherit = 'barcode.rule'

    type = fields.Selection(selection_add=[
        ('coupon', 'Coupon'),
    ], ondelete={
        'coupon': 'set default',
    })

```

## File: models\coupon.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# NOTE Use black to automatically format this code.

from odoo import api, fields, models, _


class Coupon(models.Model):
    _inherit = "coupon.coupon"

    source_pos_order_id = fields.Many2one(
        "pos.order",
        string="PoS Order Reference",
        help="PoS order where this coupon is generated.",
    )
    pos_order_id = fields.Many2one(
        "pos.order",
        string="Applied on PoS Order",
        help="PoS order where this coupon is consumed/booked.",
    )

    def _check_coupon_code(self, order_date, partner_id, **kwargs):
        if self.program_id.id in kwargs.get("reserved_program_ids", []):
            return {
                "error": _("A coupon from the same program has already been reserved for this order.")
            }
        return super()._check_coupon_code(order_date, partner_id, **kwargs)

    def _get_default_template(self):
        if self.source_pos_order_id:
            return self.env.ref('pos_coupon.mail_coupon_template', False)
        return super()._get_default_template()

    @api.model
    def _generate_code(self):
        """
        Modify the generated barcode to be compatible with the default
        barcode rule in this module. See `data/default_barcode_patterns.xml`.
        """
        code = super()._generate_code()
        return '043' + code[3:]

```

## File: models\coupon_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# NOTE Use black to automatically format this code.

from odoo import api, fields, models, _

import ast

class CouponProgram(models.Model):
    _inherit = "coupon.program"

    pos_config_ids = fields.Many2many(
        "pos.config",
        string="Point of Sales",
        readonly=True,
    )
    pos_order_line_ids = fields.One2many(
        "pos.order.line",
        "program_id",
        string="PoS Order Lines",
        help="Order lines where this program is applied.",
    )
    promo_barcode = fields.Char(
        "Barcode",
        default=lambda self: self.env["coupon.coupon"]._generate_code(),
        help="A technical field used as an alternative to the promo_code. "
        "This is automatically generated when promo_code is changed.",
    )
    pos_order_ids = fields.Many2many(
        "pos.order", help="The PoS orders where this program is applied.", copy=False
    )
    pos_order_count = fields.Integer(
        "PoS Order Count", compute="_compute_pos_order_count"
    )
    valid_product_ids = fields.Many2many(
        "product.product",
        "Valid Products",
        compute="_compute_valid_product_ids",
        help="These are the products that are valid in this program.",
    )
    valid_partner_ids = fields.Many2many(
        "res.partner",
        "Valid Partners",
        compute="_compute_valid_partner_ids",
        help="These are the partners that can avail this program.",
    )

    @api.depends("pos_order_ids")
    def _compute_pos_order_count(self):
        for program in self:
            program.pos_order_count = len(program.pos_order_ids)

    def write(self, vals):
        if "promo_code" in vals:
            vals.update({"promo_barcode": self.env["coupon.coupon"]._generate_code()})
        return super(CouponProgram, self).write(vals)

    def action_view_pos_orders(self):
        self.ensure_one()
        return {
            "name": _("PoS Orders"),
            "view_mode": "tree,form",
            "res_model": "pos.order",
            "type": "ir.actions.act_window",
            "domain": [("id", "in", self.pos_order_ids.ids)],
            "context": dict(self._context, create=False),
        }

    @api.depends("rule_products_domain")
    def _compute_valid_product_ids(self):
        domain_products = {}
        for program in self:
            product_ids = domain_products.get(program.rule_products_domain)
            if product_ids is None:
                domain = ast.literal_eval(program.rule_products_domain) if program.rule_products_domain else []
                product_ids = self.env["product.product"].search(domain, order="id").ids
                domain_products[program.rule_products_domain] = product_ids
            program.valid_product_ids = product_ids

    @api.depends("rule_partners_domain")
    def _compute_valid_partner_ids(self):
        domain_partners = {}
        for program in self:
            partner_ids = []
            if program.rule_partners_domain and program.rule_partners_domain != "[]":
                partner_ids = domain_partners.get(program.rule_partners_domain)
                if partner_ids is None:
                    domain = ast.literal_eval(program.rule_partners_domain)
                    partner_ids = self.env["res.partner"].search(domain, order="id").ids
                    domain_partners[program.rule_partners_domain] = partner_ids
            program.valid_partner_ids = partner_ids

    @api.depends('pos_order_ids')
    def _compute_total_order_count(self):
        super(CouponProgram, self)._compute_total_order_count()
        for program in self:
            program.total_order_count += len(program.pos_order_ids)

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# NOTE Use black to automatically format this code.

from datetime import datetime

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class PosConfig(models.Model):
    _inherit = "pos.config"

    use_coupon_programs = fields.Boolean(
        "Coupons & Promotions",
        help="Use coupon and promotion programs in this PoS configuration.",
    )
    coupon_program_ids = fields.Many2many(
        "coupon.program",
        string="Coupon Programs",
        compute="_filter_programs",
        inverse="_set_programs",
    )
    promo_program_ids = fields.Many2many(
        "coupon.program",
        string="Promotion Programs",
        compute="_filter_programs",
        inverse="_set_programs",
    )
    program_ids = fields.Many2many("coupon.program", string="Coupons and Promotions")

    @api.depends("program_ids")
    def _filter_programs(self):
        for config in self:
            config.coupon_program_ids = config.program_ids.filtered(
                lambda program: program.program_type == "coupon_program"
            )
            config.promo_program_ids = config.program_ids.filtered(
                lambda program: program.program_type == "promotion_program"
            )

    def _set_programs(self):
        for config in self:
            config.program_ids = config.coupon_program_ids | config.promo_program_ids

    def open_session_cb(self, check_coa=True):
        # Check validity of programs before opening a new session
        invalid_reward_products_msg = ""
        for program in self.program_ids:
            if (
                program.reward_product_id
                and not program.reward_product_id.available_in_pos
            ):
                reward_product = program.reward_product_id
                invalid_reward_products_msg += "\n\t"
                invalid_reward_products_msg += _(
                    "Program: %(name)s (%(type)s), Reward Product: `%(reward_product)s`",
                    name=program.name,
                    type=program.program_type,
                    reward_product=reward_product.name,
                )

        if invalid_reward_products_msg:
            intro = _(
                "To continue, make the following reward products to be available in Point of Sale."
            )
            raise UserError(f"{intro}\n{invalid_reward_products_msg}")

        return super(PosConfig, self).open_session_cb()

    def use_coupon_code(self, code, creation_date, partner_id, reserved_program_ids):
        coupon_to_check = self.env["coupon.coupon"].search(
            [("code", "=", code), ("program_id", "in", self.program_ids.ids)]
        )
        # If not unique, we only check the first coupon.
        coupon_to_check = coupon_to_check[:1]
        if not coupon_to_check:
            return {
                "successful": False,
                "payload": {
                    "error_message": _("This coupon is invalid (%s).") % (code)
                },
            }
        message = coupon_to_check._check_coupon_code(
            fields.Date.from_string(creation_date[:11]),
            partner_id,
            reserved_program_ids=reserved_program_ids,
        )
        error_message = message.get("error", False)
        if error_message:
            return {
                "successful": False,
                "payload": {"error_message": error_message},
            }

        coupon_to_check.sudo().write({"state": "used"})
        return {
            "successful": True,
            "payload": {
                "program_id": coupon_to_check.program_id.id,
                "coupon_id": coupon_to_check.id,
            },
        }

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# NOTE Use black to automatically format this code.

from collections import defaultdict

from odoo import api, fields, models, _


class PosOrder(models.Model):
    _inherit = "pos.order"

    applied_program_ids = fields.Many2many(
        "coupon.program",
        string="Applied Programs",
        help="Technical field. This is set when the order is validated. "
        "We normally get this value thru the `program_id` of the reward lines.",
    )
    used_coupon_ids = fields.One2many(
        "coupon.coupon", "pos_order_id", string="Consumed Coupons"
    )
    generated_coupon_ids = fields.One2many(
        "coupon.coupon", "source_pos_order_id", string="Generated Coupons"
    )

    def validate_coupon_programs(
        self, program_ids_to_generate_coupons, unused_coupon_ids
    ):
        """This is called after create_from_ui is called. We set here fields
        that are used to link programs and coupons to the order.

        We also return the generated coupons that can be used in the frontend
        to print the generated codes in the receipt.
        """
        self.ensure_one()

        program_ids_to_generate_coupons = program_ids_to_generate_coupons or []
        unused_coupon_ids = unused_coupon_ids or []

        self.env["coupon.coupon"].browse(unused_coupon_ids).write({"state": "new"})
        self.sudo().write(
            {
                "applied_program_ids": [(4, i) for i in self.lines.program_id.ids],
                "used_coupon_ids": [(4, i) for i in self.lines.coupon_id.ids],
                "generated_coupon_ids": [
                    (4, i)
                    for i in (
                        self.env["coupon.program"]
                        .browse(program_ids_to_generate_coupons)
                        .sudo()._generate_coupons(self.partner_id.id)
                    ).ids
                ],
            }
        )
        return [
            {
                "code": coupon.code,
                "expiration_date": coupon.expiration_date,
                "program_name": coupon.program_id.name,
            }
            for coupon in self.generated_coupon_ids
        ]

    def _get_fields_for_order_line(self):
        fields = super(PosOrder, self)._get_fields_for_order_line()
        fields.extend({
            'is_program_reward',
            'coupon_id',
            'program_id',
        })
        return fields

    def _prepare_order_line(self, order_line):
        order_line = super(PosOrder, self)._prepare_order_line(order_line)
        if order_line['program_id']:
            order_line['program_id'] = order_line['program_id'][0]
        return order_line

class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    is_program_reward = fields.Boolean(
        "Is reward line",
        help="Flag indicating that this order line is a result of coupon/promo program.",
    )
    program_id = fields.Many2one(
        "coupon.program",
        string="Program",
        help="Promotion/Coupon Program where this reward line is based.",
    )
    coupon_id = fields.Many2one(
        "coupon.coupon", string="Coupon", help="Coupon that generated this reward.",
    )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import coupon
from . import coupon_program
from . import barcode_rule

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_program_pos_user,Coupon Program (PoS User),coupon.model_coupon_program,point_of_sale.group_pos_user,1,0,0,0
access_program_pos_manager,Coupon Program (PoS Manager),coupon.model_coupon_program,point_of_sale.group_pos_manager,1,1,1,1
access_rule_pos_user,Coupon Rule (PoS User),coupon.model_coupon_rule,point_of_sale.group_pos_user,1,0,0,0
access_rule_pos_manager,Coupon Rule (PoS Manager),coupon.model_coupon_rule,point_of_sale.group_pos_manager,1,1,1,0
access_coupon_pos_user,Coupon (PoS User),coupon.model_coupon_coupon,point_of_sale.group_pos_user,1,0,0,0
access_coupon_pos_manager,Coupon (PoS Manager),coupon.model_coupon_coupon,point_of_sale.group_pos_manager,1,1,1,0
access_reward_pos_user,Coupon Reward (PoS User),coupon.model_coupon_reward,point_of_sale.group_pos_user,1,0,0,0
access_reward_pos_manager,Coupon Reward (PoS Manager),coupon.model_coupon_reward,point_of_sale.group_pos_manager,1,1,1,0
access_coupon_generate_wizard,Coupon Generation,coupon.model_coupon_generate_wizard,point_of_sale.group_pos_user,1,1,1,0

```

## File: static\src\js\ActivePrograms.js

```javascript
odoo.define('pos_coupon.ActivePrograms', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const { onChangeOrder } = require('point_of_sale.custom_hooks');

    class ActivePrograms extends PosComponent {
        constructor() {
            super(...arguments);
            onChangeOrder(this._onPrevOrder, this._onNewOrder);
            this.renderParams = {};
        }
        _onPrevOrder(prevOrder) {
            if (prevOrder) {
                prevOrder.off('change', null, this);
                prevOrder.off('rewards-updated', null, this);
            }
        }
        _onNewOrder(newOrder) {
            if (newOrder) {
                newOrder.on('change', this.render, this);
                newOrder.on('rewards-updated', this.render, this);
                newOrder.trigger('update-rewards');
            }
        }
        async render() {
            this._setRenderParams();
            await super.render();
        }
        get currentOrder() {
            return this.env.pos.get_order();
        }
        /**
         * This is used to set the render parameters before eventually rendering this component.
         */
        _setRenderParams() {
            const order = this.currentOrder;
            const unRewardedArray = order.rewardsContainer ? order.rewardsContainer.getUnawarded() : [];
            const nonGeneratingProgramIds = new Set(unRewardedArray.map(({ program }) => program.id));
            const nonGeneratingCouponIds = new Set(
                unRewardedArray.map(({ coupon_id }) => coupon_id).filter((coupon_id) => coupon_id)
            );
            const onNextOrderPromoPrograms = order.activePromoProgramIds
                .filter((program_id) => {
                    const program = order.pos.coupon_programs_by_id[program_id];
                    return program.promo_applicability === 'on_next_order' && order.programIdsToGenerateCoupons.includes(program_id);
                })
                .map((program_id) => order.pos.coupon_programs_by_id[program_id]);
            const onCurrentOrderPromoProgramIds = order.activePromoProgramIds.filter((program_id) => {
                const program = order.pos.coupon_programs_by_id[program_id];
                return program.promo_applicability === 'on_current_order';
            });
            const withRewardsPromoPrograms = onCurrentOrderPromoProgramIds
                .filter((program_id) => !nonGeneratingProgramIds.has(program_id))
                .map((program_id) => {
                    const program = order.pos.coupon_programs_by_id[program_id];
                    return {
                        name: program.name,
                        promo_code: program.promo_code,
                    };
                });
            const withRewardsBookedCoupons = Object.values(order.bookedCouponCodes)
                .filter((couponCode) => !nonGeneratingCouponIds.has(couponCode.coupon_id))
                .map((couponCode) => {
                    let program = order.pos.coupon_programs_by_id[couponCode.program_id];
                    return {
                        program_name: program.name,
                        coupon_code: couponCode.code,
                    };
                });
            Object.assign(this.renderParams, {
                withRewardsPromoPrograms,
                withRewardsBookedCoupons,
                onNextOrderPromoPrograms,
                show:
                    withRewardsPromoPrograms.length !== 0 ||
                    withRewardsBookedCoupons.length !== 0 ||
                    onNextOrderPromoPrograms.length !== 0,
            });
        }
    }
    ActivePrograms.template = 'ActivePrograms';

    Registries.Component.add(ActivePrograms);

    return ActivePrograms;
});

```

## File: static\src\js\coupon.js

```javascript
odoo.define('pos_coupon.pos', function (require) {
    'use strict';

    /**
     * When pos_coupon is active (`use_coupon_programs == true`), reward lines
     * are generated for each order. Everytime an order is updated ('update-rewards'
     * event is triggered), the reward lines are recalculated. Generated reward lines
     * are computed based on `bookedCouponCodes` and `activePromoProgramIds` which are
     * initialized in `_initializePrograms`. `activePromoProgramIds` start with the
     * automatic promo programs - promo programs with `promo_code_usage == 'no_code_needed'`.
     * `bookedCouponCodes` and `activePromoProgramIds` containers are then updated when
     * scanning codes (see `activateCode`).
     *
     * In short, `bookedCouponCodes` and `activePromoProgramIds` are populated via
     * `activateCode`, then whenever 'update-rewards' is triggered, its callback updates
     * the reward lines based on the values stored in `bookedCouponCodes` and
     * `activePromoProgramIds`
     */

    const models = require('point_of_sale.models');
    const rpc = require('web.rpc');
    const session = require('web.session');
    const concurrency = require('web.concurrency');
    const { Gui } = require('point_of_sale.Gui');
    const { float_is_zero,round_decimals } = require('web.utils');

    const dp = new concurrency.DropPrevious();

    class CouponCode {
        /**
         * @param {string} code coupon code
         * @param {number} coupon_id id of coupon.coupon
         * @param {numnber} program_id id of coupon.program
         */
        constructor(code, coupon_id, program_id) {
            this.code = code;
            this.coupon_id = coupon_id;
            this.program_id = program_id;
        }
    }

    class Reward {
        static createKey(program_id, coupon_id) {
            return coupon_id ? `${program_id}-${coupon_id}` : `${program_id}`;
        }
        /**
         * Generated reward lines are based on the values stored in this object.
         *
         * @param {product.product} product product used in creating the reward line
         * @param {number} unit_price unit price of the reward
         * @param {number} quantity
         * @param {coupon.program} program
         * @param {number[]} tax_ids tax ids
         * @param {number?} coupon_id id of the coupon.coupon the generates this reward
         * @param {boolean=true} awarded identifies this object as 'awarded' or not.
         * @param {string?} reason reason why this reward is 'unawarded'.
         */
        constructor({
            product,
            unit_price,
            quantity,
            program,
            tax_ids,
            coupon_id = undefined,
            awarded = true,
            reason = undefined,
        }) {
            this.product = product;
            this.unit_price = unit_price;
            this.quantity = quantity;
            this.program = program;
            this.tax_ids = tax_ids;
            this.coupon_id = coupon_id;
            this._discountAmount = Math.abs(unit_price * quantity);
            this.status = {
                awarded,
                reason,
            };
            this._key = Reward.createKey(program.id, coupon_id);
        }
        /**
         * If the program's reward_type is 'product', return the product.product id of the
         * reward product.
         */
        get rewardedProductId() {
            return (
                this.program.reward_type == 'product' &&
                this.program.reward_product_id &&
                this.program.reward_product_id[0]
            );
        }
        get discountAmount() {
            return this._discountAmount;
        }
        get key() {
            return this._key;
        }
    }

    class RewardsContainer {
        /**
         * The idea here is to have a data structure that will contain the awarded
         * and unawarded rewards based on the program and coupon combination.
         * If the program-coupon combination does not generate rewards because of
         * rules or because it is not the highest global discount, we create
         * an 'unawarded' `Reward` that corresponds to it, then we `add` it to
         * this data structure. Otherwise, we create an 'awarded' `Reward` object
         * then `add` it as well.
         *
         * We can then get the 'awarded' and 'unawarded' rewards via `getAwarded`
         * and `getUnawarded` methods, respectively.
         */
        constructor() {
            /**
             * @type {Record<string, Reward[]>} key is based on `program_id` and `coupon_id`.
             */
            this._rewards = {};
        }
        clear() {
            this._rewards = {};
        }
        /**
         * @param {Reward[]} rewards
         */
        add(rewards) {
            for (const reward of rewards) {
                if (reward.key in this._rewards) {
                    this._rewards[reward.key].push(reward);
                } else {
                    this._rewards[reward.key] = [reward];
                }
            }
        }
        getUnawarded() {
            return this._getFlattenRewards().filter((reward) => !reward.status.awarded);
        }
        getAwarded() {
            return this._getFlattenRewards().filter((reward) => reward.status.awarded);
        }
        _getFlattenRewards() {
            return Object.values(this._rewards).reduce((flatArr, arr) => [...flatArr, ...arr], []);
        }
    }

    // Some utility functions

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
        return free + adjustment;
    }

    // Load the products used for creating program reward lines.
    var existing_models = models.PosModel.prototype.models;
    var product_index = _.findIndex(existing_models, function (model) {
        return model.model === 'product.product';
    });
    var product_model = existing_models[product_index];
    models.load_models([
        {
            model: 'coupon.program',
            fields: [],
            domain: function (self) {
                return [['id', 'in', self.config.program_ids]];
            },
            loaded: function (self, programs) {
                self.programs = programs;
                self.coupon_programs_by_id = {};
                self.coupon_programs = [];
                self.promo_programs = [];
                for (let program of self.programs) {
                    // index by id
                    self.coupon_programs_by_id[program.id] = program;
                    // separate coupon programs from promo programs
                    if (program.program_type === 'coupon_program') {
                        self.coupon_programs.push(program);
                    } else {
                        self.promo_programs.push(program);
                    }
                    // cast some arrays to Set for faster membership checking
                    program.valid_product_ids = new Set(program.valid_product_ids);
                    program.valid_partner_ids = new Set(program.valid_partner_ids);
                    program.discount_specific_product_ids = new Set(program.discount_specific_product_ids);
                }
            },
        },
        {
            model: product_model.model,
            fields: product_model.fields,
            order: product_model.order,
            domain: function (self) {
                const discountLineProductIds = self.programs.map((program) => program.discount_line_product_id[0]);
                const rewardProductIds = self.programs.map((program) => program.reward_product_id[0]);
                return [['id', 'in', discountLineProductIds.concat(rewardProductIds)]];
            },
            context: product_model.context,
            loaded: product_model.loaded,
        },
    ]);

    var _posmodel_super = models.PosModel.prototype;
    models.PosModel = models.PosModel.extend({
        initialize: function () {
            _posmodel_super.initialize.apply(this, arguments);
            this.ready.then(() => {
                if (this.get('selectedOrder')) {
                    this.get('selectedOrder').trigger('update-rewards');
                }
            });
        },
    });

    /**
     * @listens 'update-rewards'
     * @listens 'reset-coupons' calls resetCoupons when triggered.
     * @emits 'rewards-updated' emitted after 'update-rewards' callback.
     */
    var _order_super = models.Order.prototype;
    models.Order = models.Order.extend({
        // OVERIDDEN METHODS

        initialize: function () {
            _order_super.initialize.apply(this, arguments);
            this.on(
                'update-rewards',
                () => {
                    if (!this.pos.config.use_coupon_programs) return;
                    dp.add(this._getNewRewardLines()).then(([newRewardLines, rewardsContainer]) => {
                        newRewardLines.forEach(line => this.add_orderline(line));
                        // We need this for the rendering of ActivePrograms component.
                        this.rewardsContainer = rewardsContainer;
                        // Send a signal that the rewardsContainer are updated.
                        this.trigger('rewards-updated');
                    }).catch(() => { /* catch the reject of dp when calling `add` to avoid unhandledrejection */ });
                },
                this
            );
            this.on('reset-coupons', this.resetCoupons, this);
            this._initializePrograms();
            return this;
        },
        init_from_JSON: function (json) {
            this.bookedCouponCodes = json.bookedCouponCodes ? json.bookedCouponCodes : {};
            this.activePromoProgramIds = json.activePromoProgramIds ? json.activePromoProgramIds : [];
            _order_super.init_from_JSON.apply(this, arguments);
        },
        export_as_JSON: function () {
            let json = _order_super.export_as_JSON.apply(this, arguments);
            return Object.assign(json, {
                bookedCouponCodes: this.bookedCouponCodes,
                activePromoProgramIds: this.activePromoProgramIds,
            });
        },
        set_orderline_options: function (orderline, options) {
            _order_super.set_orderline_options.apply(this, [orderline, options]);
            if (options && options.is_program_reward) {
                orderline.is_program_reward = true;
                orderline.program_id = options.program_id;
                orderline.coupon_id = options.coupon_id;
            }
        },
        /**
         * This function's behavior is modified so that the reward lines are
         * rendered at the bottom of the orderlines list.
         */
        get_orderlines: function () {
            const orderlines = _order_super.get_orderlines.apply(this, arguments);
            const rewardLines = [];
            const nonRewardLines = [];
            for (const line of orderlines) {
                if (line.is_program_reward) {
                    rewardLines.push(line);
                } else {
                    nonRewardLines.push(line);
                }
            }
            return [...nonRewardLines, ...rewardLines];
        },
        _getRegularOrderlines: function () {
            const orderlines = _order_super.get_orderlines.apply(this, arguments);
            const is_gift_card_product = (line) => this.pos.config.gift_card_product_id && line.product.id === this.pos.config.gift_card_product_id[0];
            const is_tips_product = (line) => this.pos.config.tip_product_id && line.product.id === this.pos.config.tip_product_id[0];
            //reward_id is always false unless the line is a reward from pos_loyalty
            return orderlines.filter((line) => !line.is_program_reward && !line.reward_id && !line.refunded_orderline_id && !is_gift_card_product(line) && !is_tips_product(line));
        },
        _getRewardLines: function () {
            const orderlines = _order_super.get_orderlines.apply(this, arguments);
            return orderlines.filter((line) => line.is_program_reward);
        },
        wait_for_push_order: function () {
            return (
                (this.programIdsToGenerateCoupons && this.programIdsToGenerateCoupons.length) ||
                this.get_orderlines().filter((line) => line.is_program_reward).length ||
                _order_super.wait_for_push_order.apply(this, arguments)
            );
        },
        export_for_printing: function () {
            let result = _order_super.export_for_printing.apply(this, arguments);
            result.generated_coupons = this.generated_coupons;
            return result;
        },
        add_product: async function (product, options) {
            await _order_super.add_product.apply(this, [product, options]);
            this.trigger('update-rewards');
        },
        get_last_orderline: function () {
            const regularLines = _order_super.get_orderlines
                .apply(this, arguments)
                .filter((line) => !line.is_program_reward);
            return regularLines[regularLines.length - 1];
        },
        selectLastOrderline: function(line){
            if(!line.is_program_reward) {
                _order_super.selectLastOrderline.apply(this, arguments);
            }
        },
        set_pricelist: function (pricelist) {
            _order_super.set_pricelist.apply(this, arguments);
            this.trigger('update-rewards');
        },

        // NEW METHODS

        _initializePrograms: async function () {
            if (!this.bookedCouponCodes) {
                /**
                 * This field contains the activated coupons.
                 * @type {Record<string, CouponCode>} key is the coupon code.
                 */
                this.bookedCouponCodes = {};
            }
            if (!(this.activePromoProgramIds && this.activePromoProgramIds.length)) {
                /**
                 * This field contains the ids of automatically/manually activated
                 * promo programs.
                 * @type {number[]} array of program ids.
                 */
                this.activePromoProgramIds = this._getAutomaticPromoProgramIds();
            }
        },
        resetPrograms: function () {
            let deactivatedCount = 0;
            if (this.bookedCouponCodes) {
                const couponIds = Object.values(this.bookedCouponCodes).map((couponCode) => couponCode.coupon_id);
                if (couponIds.length > 0) {
                    this.trigger('reset-coupons', couponIds);
                }
                this.bookedCouponCodes = {};
                deactivatedCount += couponIds.length;
            }
            if (this.activePromoProgramIds) {
                const codeNeededPromoProgramIds = this.activePromoProgramIds.filter((program_id) => {
                    return this.pos.coupon_programs_by_id[program_id].promo_code_usage === 'code_needed';
                });
                this.activePromoProgramIds = this._getAutomaticPromoProgramIds();
                deactivatedCount += codeNeededPromoProgramIds.length;
            }
            if (deactivatedCount > 0) Gui.showNotification('Active coupons and promo codes were deactivated.');
            this.trigger('update-rewards');
        },
        /**
         * Updates `bookedCouponCodes` or `activePromoProgramIds` depending on which code
         * is scanned.
         *
         * @param {string} code
         */
        activateCode: async function (code) {
            const promoProgram = this.pos.promo_programs.find(
                (program) => program.promo_barcode == code || program.promo_code == code
            );
            if (promoProgram && this.activePromoProgramIds.includes(promoProgram.id)) {
                Gui.showNotification('That promo code program has already been activated.');
            } else if (promoProgram) {
                // TODO these two operations should be atomic
                this.activePromoProgramIds.push(promoProgram.id);
                this.trigger('update-rewards');
            } else if (code in this.bookedCouponCodes) {
                Gui.showNotification('That coupon code has already been scanned and activated.');
            } else {
                const programIdsWithScannedCoupon = Object.values(this.bookedCouponCodes).map(
                    (couponCode) => couponCode.program_id
                );
                const customer = this.get_client();
                const { successful, payload } = await rpc.query({
                    model: 'pos.config',
                    method: 'use_coupon_code',
                    args: [
                        [this.pos.config.id],
                        code,
                        this.creation_date,
                        customer ? customer.id : false,
                        programIdsWithScannedCoupon,
                    ],
                    kwargs: { context: session.user_context },
                });
                if (successful) {
                    // TODO these two operations should be atomic
                    this.bookedCouponCodes[code] = new CouponCode(code, payload.coupon_id, payload.program_id);
                    this.trigger('update-rewards');
                } else {
                    Gui.showNotification(payload.error_message);
                }
            }
        },
        /**
         * @returns {[models.Orderline[], RewardsContainer]}
         */
        _getNewRewardLines: async function () {
            // Remove the reward lines before recalculation of rewards.
            this.orderlines.remove(this._getRewardLines());
            const rewardsContainer = await this._calculateRewards();
            // We set the programs that will generate coupons after validation of this order.
            // See `_postPushOrderResolve` in the `PaymentScreen`.
            await this._setProgramIdsToGenerateCoupons(rewardsContainer);
            // Create reward orderlines here based on the content of `rewardsContainer` field.
            return [this._getLinesToAdd(rewardsContainer), rewardsContainer];
        },
        /**
         * @param {number[]} couponIds ids of the coupon.coupon records to reset
         */
        resetCoupons: async function (couponIds) {
            await rpc.query(
                {
                    model: 'coupon.coupon',
                    method: 'write',
                    args: [couponIds, { state: 'new' }],
                    kwargs: { context: session.user_context },
                },
                {}
            );
        },
        /**
         * Create orderline rewards based on the `awarded` rewards from `rewardsContainer`.
         *
         * @param {RewardsContainer} rewardsContainer
         * @returns {models.Orderline[]}
         */
        _getLinesToAdd: function (rewardsContainer) {
            this.assert_editable();
            return rewardsContainer
                .getAwarded()
                .map(({ product, unit_price, quantity, program, tax_ids, coupon_id }) => {
                    let description;
                    /**
                     * Improved description only aplicable for rewards of type discount, and the discount is a percentage
                     * of the price, those are:
                     * - % discount on specific products.
                     * - % discount on the whole order.
                     * - % discount on the cheapest product.
                     */
                    if (tax_ids && program.reward_type === "discount" && program.discount_type === "percentage") {
                        description =
                            tax_ids.length > 0
                                ? _.str.sprintf(
                                    this.pos.env._t("Tax: %s"),
                                    tax_ids.map((tax_id) => `%${this.pos.taxes_by_id[tax_id].amount}`).join(", ")
                                )
                                : this.pos.env._t("No tax");
                    }
                    const options = {
                        description,
                        quantity: quantity,
                        price: unit_price,
                        lst_price: unit_price,
                        is_program_reward: true,
                        program_id: program.id,
                        tax_ids: tax_ids,
                        coupon_id: coupon_id,
                    };
                    const line = new models.Orderline({}, { pos: this.pos, order: this, product });
                    this.fix_tax_included_price(line);
                    this.set_orderline_options(line, options);
                    return line;
                });
        },
        /**
         * Sets the programs ids that will generate coupons based on the `rewardsContainer`.
         * If a program do not pass the rules-check, we add an 'unawarded' `Reward` in the
         * `rewardsContainer`.
         *
         * @param {RewardsContainer} rewardsContainer
         */
        _setProgramIdsToGenerateCoupons: async function (rewardsContainer) {
            const programIdsToGenerateCoupons = [];
            for (let [program] of this._getActiveOnNextPromoPrograms()) {
                const { successful, reason } = await this._checkProgramRules(program);
                if (successful) {
                    programIdsToGenerateCoupons.push(program.id);
                } else {
                    const notAwarded = new Reward({ program, reason, awarded: false });
                    rewardsContainer.add([notAwarded]);
                }
            }
            this.programIdsToGenerateCoupons = programIdsToGenerateCoupons;
        },
        /**
         * In this method, we compute all the rewards as a result of `bookedCouponCodes`
         * and `activePromoProgramIds`. If a program do not generate a reward, we create
         * a `Reward` object with `awarded = false` then we `add` this object to the
         * `rewardsContainer`. Else, we create a `Reward` object with `awarded = true` then
         * we also `add` this to `rewardsContainer`. (See `collectRewards`.)
         *
         * The procedure in calculating the rewards is as follows:
         * - Compute the free product rewards. We will need its result in the calculation
         *   of discount rewards.
         * - Compute the fixed amount discount. This is independent of the free product
         *   rewards.
         * - Compute discount on specific products. Requires the free product rewards.
         * - Compute discount on cheapest product. Does not need the free product. We
         *   only discount the first item of the cheapest product.
         * - Compute discount on the whole order. This requires the free product rewards.
         * - We consider results of on order and cheapest product discounts as global
         *   discounts. We only choose one global discount, whichever gives the highest discount.
         * - We add the free product rewards, fixed amount discount, discount on specific
         *   products and the sole global discount to `rewardsContainer` then return it.
         *
         * @returns {RewardsContainer}
         */
        _calculateRewards: async function () {
            const rewardsContainer = new RewardsContainer();

            if (this._getRegularOrderlines().length === 0) {
                return rewardsContainer;
            }

            const {
                freeProductPrograms,
                fixedAmountDiscountPrograms,
                onSpecificPrograms,
                onCheapestPrograms,
                onOrderPrograms,
            } = await this._getValidActivePrograms(rewardsContainer);

            const collectRewards = (validPrograms, rewardGetter) => {
                const allRewards = [];
                for (let [program, coupon_id] of validPrograms) {
                    const [rewards, reason] = rewardGetter(program, coupon_id);
                    if (reason) {
                        const notAwarded = new Reward({ awarded: false, reason, program, coupon_id });
                        rewardsContainer.add([notAwarded]);
                    }
                    allRewards.push(...rewards);
                }
                return allRewards;
            };

            // - Gather the product rewards
            const freeProducts = collectRewards(freeProductPrograms, this._getProductRewards.bind(this));

            // - Gather the fixed amount discounts
            const fixedAmountDiscounts = collectRewards(fixedAmountDiscountPrograms, this._getFixedDiscount.bind(this));

            // - Gather the specific discounts
            const specificDiscountGetter = (program, coupon_id) => {
                return this._getSpecificDiscount(program, coupon_id, freeProducts);
            };
            const specificDiscounts = collectRewards(onSpecificPrograms, specificDiscountGetter);

            // - Collect the discounts from on order and on cheapest discount programs.
            const globalDiscounts = [];
            const onOrderDiscountGetter = (program, coupon_id) => {
                return this._getOnOrderDiscountRewards(program, coupon_id, freeProducts);
            };
            globalDiscounts.push(...collectRewards(onOrderPrograms, onOrderDiscountGetter));
            globalDiscounts.push(...collectRewards(onCheapestPrograms, (program, coupon_id) => this._getOnCheapestProductDiscount(program, coupon_id, freeProducts)));

            // - Group the discounts by program id.
            const groupedGlobalDiscounts = {};
            for (let discount of globalDiscounts) {
                const key = [discount.program.id, discount.coupon_id].join(',');
                if (!(key in groupedGlobalDiscounts)) {
                    groupedGlobalDiscounts[key] = [discount];
                } else {
                    groupedGlobalDiscounts[key].push(discount);
                }
            }

            // - We select the group of discounts with highest total amount.
            // Note that the result is an Array that might contain more than one
            // discount lines. This is because discounts are grouped by tax.
            let currentMaxTotal = 0;
            let currentMaxKey = null;
            for (let key in groupedGlobalDiscounts) {
                const discountRewards = groupedGlobalDiscounts[key];
                const newTotal = discountRewards.reduce((sum, discReward) => sum + discReward.discountAmount, 0);
                if (newTotal > currentMaxTotal) {
                    currentMaxTotal = newTotal;
                    currentMaxKey = key;
                }
            }
            const theOnlyGlobalDiscount = currentMaxKey
                ? groupedGlobalDiscounts[currentMaxKey].filter((discountReward) => discountReward.discountAmount !== 0)
                : [];

            // - Get the messages for the discarded global_discounts
            if (theOnlyGlobalDiscount.length > 0) {
                const theOnlyGlobalDiscountKey = [
                    theOnlyGlobalDiscount[0].program.id,
                    theOnlyGlobalDiscount[0].coupon_id,
                ].join(',');
                for (let [key, discounts] of Object.entries(groupedGlobalDiscounts)) {
                    if (key !== theOnlyGlobalDiscountKey) {
                        const notAwarded = new Reward({
                            program: discounts[0].program,
                            coupon_id: discounts[0].coupon_id,
                            reason: 'Not the greatest global discount.',
                            awarded: false,
                        });
                        rewardsContainer.add([notAwarded]);
                    }
                }
            }

            // - Add the calculated rewards.
            rewardsContainer.add([
                ...freeProducts,
                ...fixedAmountDiscounts,
                ...specificDiscounts,
                ...theOnlyGlobalDiscount,
            ]);

            return rewardsContainer;
        },
        /**
         * This method returns the segregated programs based on the types of rewards:
         * reward_type === 'product'
         *   1. freeProductPrograms
         * reward_type === 'discount'
         *   discount_type === 'fixed_amount'
         *     2. fixedAmountDiscountPrograms
         *   discount_type === 'percentage'
         *     discount_apply_on === 'specific_products'
         *       3. onSpecificPrograms
         *     discount_apply_on === 'cheapest_product'
         *       4. onCheapestPrograms
         *     discount_apply_on === 'on_order'
         *       5. onOrderPrograms
         *
         * It only includes the valid programs, those that passes the program rules.
         * This has side-effect of `add`ing unawarded rewards to the given rewardsContainer.
         */
        _getValidActivePrograms: async function (rewardsContainer) {
            const freeProductPrograms = [],
                fixedAmountDiscountPrograms = [],
                onSpecificPrograms = [],
                onCheapestPrograms = [],
                onOrderPrograms = [];

            function updateProgramLists(program, coupon_id) {
                if (program.reward_type === 'product') {
                    freeProductPrograms.push([program, coupon_id]);
                } else {
                    if (program.discount_type === 'fixed_amount') {
                        fixedAmountDiscountPrograms.push([program, coupon_id]);
                    } else if (program.discount_apply_on === 'specific_products') {
                        onSpecificPrograms.push([program, coupon_id]);
                    } else if (program.discount_apply_on === 'cheapest_product') {
                        onCheapestPrograms.push([program, coupon_id]);
                    } else {
                        onOrderPrograms.push([program, coupon_id]);
                    }
                }
            }

            for (let [program, coupon_id] of this._getBookedPromoPrograms()) {
                // Booked coupons from on next order promo programs do not need
                // checking of rules because checks are done before generating
                // coupons.
                updateProgramLists(program, coupon_id);
            }

            for (let [program, coupon_id] of [
                ...this._getBookedCouponPrograms(),
                ...this._getActiveOnCurrentPromoPrograms(),
            ]) {
                const { successful, reason } = await this._checkProgramRules(program);
                if (successful) {
                    updateProgramLists(program, coupon_id);
                } else {
                    // side-effect
                    const notAwarded = new Reward({ program, coupon_id, reason, awarded: false });
                    rewardsContainer.add([notAwarded]);
                }
            }

            return {
                freeProductPrograms,
                fixedAmountDiscountPrograms,
                onSpecificPrograms,
                onCheapestPrograms,
                onOrderPrograms,
            };
        },
        _getAutomaticPromoProgramIds: function () {
            return this.pos.promo_programs
                .filter((program) => {
                    return program.promo_code_usage == 'no_code_needed';
                })
                .map((program) => program.id);
        },
        /**
         * These are the coupon programs that are activated
         * via coupon codes. RewardsContainer can only be generated if the coupon
         * program rules are satisfied.
         *
         * @returns {[coupon.program, number][]}
         */
        _getBookedCouponPrograms: function () {
            return Object.values(this.bookedCouponCodes)
                .map((couponCode) => [
                    this.pos.coupon_programs_by_id[couponCode.program_id],
                    parseInt(couponCode.coupon_id, 10),
                ])
                .filter(([program]) => {
                    return program.program_type === 'coupon_program';
                });
        },
        /**
         * These are the on_next_order promo programs that are activated
         * via coupon codes. RewardsContainer can be generated from this program
         * without checking the constraints.
         *
         * @returns {[coupon.program, number][]}
         */
        _getBookedPromoPrograms: function () {
            return Object.values(this.bookedCouponCodes)
                .map((couponCode) => [
                    this.pos.coupon_programs_by_id[couponCode.program_id],
                    parseInt(couponCode.coupon_id, 10),
                ])
                .filter(([program]) => {
                    return program.program_type === 'promotion_program';
                });
        },
        /**
         * These are the active on_current_order promo programs that will generate
         * rewards if the program constraints are fully-satisfied.
         *
         * @returns {[coupon.program, null][]}
         */
        _getActiveOnCurrentPromoPrograms: function () {
            return this.activePromoProgramIds
                .map((program_id) => [this.pos.coupon_programs_by_id[program_id], null])
                .filter(([program]) => {
                    return program.promo_applicability === 'on_current_order';
                });
        },
        /**
         * These are the active on_next_order promo programs that will generate
         * coupon codes if the program constraints are fully-satisfied.
         *
         * @returns {[coupon.program, null][]}
         */
        _getActiveOnNextPromoPrograms: function () {
            return this.activePromoProgramIds
                .map((program_id) => [this.pos.coupon_programs_by_id[program_id], null])
                .filter(([program]) => {
                    return program.promo_applicability === 'on_next_order';
                });
        },
        _convertToDate: function (stringDate) {
            return new Date(stringDate.replace(/ /g, 'T').concat('Z'))
        },
        /**
         * @param {coupon.program} program
         * @returns {{ successful: boolean, reason: string | undefined }}
         */
        _checkProgramRules: async function (program) {
            // Check minimum amount
            const amountToCheck =
                program.rule_minimum_amount_tax_inclusion === 'tax_included'
                    ? this.get_total_with_tax()
                    : this.get_total_without_tax();
            // TODO jcb rule_minimum_amount has to be converted.
            if (
                !(
                    amountToCheck > program.rule_minimum_amount ||
                    float_is_zero(amountToCheck - program.rule_minimum_amount, this.pos.currency.decimals)
                )
            ) {
                return {
                    successful: false,
                    reason: 'Minimum amount for this program is not satisfied.',
                };
            }

            // Check minimum quantity
            const validQuantity = this._getRegularOrderlines()
                .filter((line) => {
                    return program.valid_product_ids.has(line.product.id);
                })
                .reduce((total, line) => total + line.quantity, 0);
            if (!(validQuantity >= program.rule_min_quantity)) {
                return {
                    successful: false,
                    reason: "Program's minimum quantity is not satisfied.",
                };
            }

            // Bypass other rules if program is coupon_program
            if (program.program_type === 'coupon_program') {
                return {
                    successful: true,
                };
            }

            // Check if valid customer
            const customer = this.get_client();
            const partnersDomain = program.rule_partners_domain || '[]';
            if (partnersDomain !== '[]' && !program.valid_partner_ids.has(customer ? customer.id : 0)) {
                return {
                    successful: false,
                    reason: "Current customer can't avail this program.",
                };
            }

            // Check rule date
            const ruleFrom = program.rule_date_from ? this._convertToDate(program.rule_date_from) : new Date(-8640000000000000);
            const ruleTo = program.rule_date_to ? this._convertToDate(program.rule_date_to) : new Date(8640000000000000);
            const orderDate = new Date();
            if (!(orderDate >= ruleFrom && orderDate <= ruleTo)) {
                return {
                    successful: false,
                    reason: 'Program already expired.',
                };
            }

            // Check max number usage
            if (program.maximum_use_number !== 0) {
                const [result] = await rpc
                    .query({
                        model: 'coupon.program',
                        method: 'read',
                        args: [program.id, ['total_order_count']],
                        kwargs: { context: session.user_context },
                    })
                    .catch(() => Promise.resolve([false])); // may happen because offline
                if (!result) {
                    return {
                        successful: false,
                        reason: 'Unable to get the number of usage of the program.',
                    };
                } else if (!(result.total_order_count < program.maximum_use_number)) {
                    return {
                        successful: false,
                        reason: "Program's maximum number of usage has been reached.",
                    };
                }
            }

            return {
                successful: true,
            };
        },
        /**
         * This method is called via `collectRewards` inside `_calculateRewards`.
         * The purpose of this method is to create `Reward` objects based on the given
         * `program` and `coupon_id`.
         * It returns a tuple of rewards and reason if no rewards are created.
         *
         * @param {coupon.program} program
         * @param {number} coupon_id
         * @returns {[Reward[], string | null]}
         */
        _getProductRewards: function (program, coupon_id) {
            const rewardProduct = this.pos.db.get_product_by_id(program.reward_product_id[0]);
            const countedOrderlines = this._getRegularOrderlines().filter((line) =>
                program.valid_product_ids.has(line.product.id)
            );
            const totalQuantity = countedOrderlines.reduce((quantity, line) => quantity + line.quantity, 0);
            const totalAmount = countedOrderlines.reduce((amount, line) => {
                const { priceWithTax, priceWithoutTax } = line.get_all_prices();
                if (program.rule_minimum_amount_tax_inclusion == 'tax_included') {
                    amount += priceWithTax;
                } else {
                    amount += priceWithoutTax;
                }
                return amount;
            }, 0);

            // Compute the free quantities based on rule_min_amount and rule_min_quantity.
            let freeQuantityFromMinAmount = Math.Infinity;
            let freeQuantityFromMinQuantity;
            const existingRewardQty = this._getRegularOrderlines()
                .filter((line) => line.product.id == rewardProduct.id)
                .reduce((total, line) => total + line.quantity, 0);
            if (program.valid_product_ids.has(rewardProduct.id)) {
                if (existingRewardQty) {
                    freeQuantityFromMinQuantity = Math.min(
                        computeFreeQuantity(totalQuantity, program.rule_min_quantity, program.reward_product_quantity),
                        existingRewardQty
                    );
                    if (program.rule_minimum_amount !== 0) {
                        // Normalize the values based on amount to be able to utilize computeFreeQuantity.
                        const rewardProductAmount = program.reward_product_quantity * rewardProduct.lst_price;
                        const freeAmount = computeFreeQuantity(
                            totalAmount,
                            program.rule_minimum_amount,
                            rewardProductAmount
                        );
                        freeQuantityFromMinAmount = Math.min(
                            Math.trunc(freeAmount / rewardProduct.lst_price),
                            existingRewardQty
                        );
                    }
                } else {
                    // No free quantity if the reward product is not among the orderlines.
                    freeQuantityFromMinQuantity = 0;
                    freeQuantityFromMinAmount = 0;
                }
            } else {
                freeQuantityFromMinQuantity = Math.min(
                    Math.trunc((totalQuantity * program.reward_product_quantity) / program.rule_min_quantity),
                    existingRewardQty
                );
                if (program.rule_minimum_amount !== 0) {
                    freeQuantityFromMinAmount = Math.min(
                        Math.trunc((totalAmount * program.reward_product_quantity) / program.rule_minimum_amount),
                        existingRewardQty
                    );
                }
            }

            // Based on freeQuantityFromMinAmount and freeQuantityFromMinQuantity, compute the actual free quantity.
            let freeQuantity = 0;
            if (freeQuantityFromMinAmount < freeQuantityFromMinQuantity) {
                freeQuantity = freeQuantityFromMinAmount;
            } else {
                freeQuantity = freeQuantityFromMinQuantity;
            }

            if (freeQuantity === 0) {
                return [[], 'Zero free product quantity.'];
            } else {
                const discountLineProduct = this.pos.db.get_product_by_id(program.discount_line_product_id[0]);
                return [
                    [
                        new Reward({
                            product: discountLineProduct,
                            unit_price: -round_decimals(rewardProduct.get_price(this.pricelist, freeQuantity), this.pos.currency.decimals),
                            quantity: freeQuantity,
                            program: program,
                            tax_ids: rewardProduct.taxes_id,
                            coupon_id: coupon_id,
                        }),
                    ],
                    null,
                ];
            }
        },
        /**
         * This method is called via `collectRewards` inside `_calculateRewards`.
         * It returns fixed discount reward based on the given program.
         *
         * @param {coupon.program} program
         * @param {number} coupon_id
         * @returns {[Reward[], string | null]}
         */
        _getFixedDiscount: function (program, coupon_id) {
            const discountAmount = Math.min(program.discount_fixed_amount, program.discount_max_amount || Infinity);
            return [
                [
                    new Reward({
                        product: this.pos.db.get_product_by_id(program.discount_line_product_id[0]),
                        unit_price: -discountAmount,
                        quantity: 1,
                        program: program,
                        coupon_id: coupon_id,
                    }),
                ],
                null,
            ];
        },
        /**
         * This method is called via `collectRewards` inside `_calculateRewards`.
         * This returns discount rewards based on the program's specific products.
         * Amounts are grouped based on products tax ids (see `_getGroupKey`).
         * We also adjust the `amountsToDiscount` based on the rewarded products.
         *
         * @param {coupon.program} program
         * @param {number} coupon_id
         * @param {Reward[]} productRewards
         * @returns {[Reward[], string | null]}
         */
        _getSpecificDiscount: function (program, coupon_id, productRewards) {
            const productIdsToAccount = new Set();
            const amountsToDiscount = {};
            for (let line of this._getRegularOrderlines()) {
                if (program.discount_specific_product_ids.has(line.get_product().id)) {
                    const key = this._getGroupKey(line);
                    if (!(key in amountsToDiscount)) {
                        amountsToDiscount[key] = line.get_base_price();
                    } else {
                        amountsToDiscount[key] += line.get_base_price();
                    }
                    productIdsToAccount.add(line.get_product().id);
                }
            }
            this._considerProductRewards(amountsToDiscount, productIdsToAccount, productRewards);
            return this._createDiscountRewards(program, coupon_id, amountsToDiscount);
        },
        /**
         * This method is called via `collectRewards` inside `_calculateRewards`.
         * It returns a discount reward for the cheapest item in the order. Cheapest
         * item is a single quantity with lowest price.
         *
         * @param {coupon.program} program
         * @param {number} coupon_id
         * @returns {[Reward[], string | null]}
         */
        _getOnCheapestProductDiscount: function (program, coupon_id, productRewards) {
            const amountsToDiscount = {};
            const orderlines = this._getRegularOrderlines();
            if (orderlines.length > 0) {
                const cheapestLine = this._findCheapestLine(orderlines, productRewards);
                if (cheapestLine) {
                    const key = this._getGroupKey(cheapestLine);
                    amountsToDiscount[key] = cheapestLine.price;
                }
            }
            return this._createDiscountRewards(program, coupon_id, amountsToDiscount);
        },
        /**
         * Returns the cheapest line from the given orderlines considering the rewarded products.
         * @param {models.Orderline[]} orderlines
         * @param {Reward[]} productRewards
         * @returns {models.Orderline}
         */
        _findCheapestLine: function (orderlines, productRewards) {
            // Compute free quantity per product.
            const freeQuantityPerProduct = {};
            for (const productReward of productRewards) {
                const productId = productReward.rewardedProductId;
                if (!(productId in freeQuantityPerProduct)) {
                    freeQuantityPerProduct[productId] = 0;
                }
                freeQuantityPerProduct[productId] += productReward.quantity;
            }
            // Map each line to its remaining free quantity.
            // Important to loop over the lines in decreasing price.
            const remainingQtyOfLine = new Map();
            for (const line of [...orderlines].sort((a, b) => b.price - a.price)) {
                const productId = line.product.id;
                let freeQuantity = freeQuantityPerProduct[productId] || 0;
                remainingQtyOfLine.set(line, line.get_quantity());
                if (float_is_zero(freeQuantity, this.pos.dp['Product Unit of Measure'])) {
                    continue;
                }
                const lineQty = remainingQtyOfLine.get(line);
                if (lineQty < freeQuantity) {
                    remainingQtyOfLine.set(line, 0);
                    freeQuantity -= lineQty;
                } else {
                    remainingQtyOfLine.set(line, lineQty - freeQuantity);
                    freeQuantity = 0;
                }
                freeQuantityPerProduct[productId] = freeQuantity;
            }
            // Among the lines with remaining quantity, return the one with the lowest price.
            const linesWithoutRewards = [...remainingQtyOfLine.entries()]
                .filter(([_, remainingQty]) => !float_is_zero(remainingQty, this.pos.currency.decimals))
                .map(([line, _]) => line)
                .sort((a, b) => a.price - b.price);
            return linesWithoutRewards[0];
        },
        /**
         * This method is called via `collectRewards` inside `_calculateRewards`.
         * This returns discount rewards based on all the orderlines. Amounts are grouped based
         * on products tax ids (see `_getGroupKey`). `amountsToDiscount` is adjusted
         * based on the rewarded products.
         *
         * @param {coupon.program} program
         * @param {number} coupon_id
         * @param {Reward[]} productRewards
         * @returns {[Reward[], string | null]}
         */
        _getOnOrderDiscountRewards: function (program, coupon_id, productRewards) {
            const productIdsToAccount = new Set();
            const amountsToDiscount = {};
            for (let line of this._getRegularOrderlines()) {
                const key = this._getGroupKey(line);
                if (!(key in amountsToDiscount)) {
                    amountsToDiscount[key] = line.get_base_price();
                } else {
                    amountsToDiscount[key] += line.get_base_price();
                }
                productIdsToAccount.add(line.get_product().id);
            }
            this._considerProductRewards(amountsToDiscount, productIdsToAccount, productRewards);
            return this._createDiscountRewards(program, coupon_id, amountsToDiscount);
        },
        /**
         * Mutates `amountsToDiscount` to take into account the product rewards.
         *
         * @param {Record<string, number>} amountsToDiscount
         * @param {Set<number>} productIdsToAccount
         * @param {Reward[]} productRewards
         */
        _considerProductRewards: function (amountsToDiscount, productIdsToAccount, productRewards) {
            for (let reward of productRewards) {
                if (reward.rewardedProductId && productIdsToAccount.has(reward.rewardedProductId)) {
                    const key = reward.tax_ids.join(',');
                    amountsToDiscount[key] += reward.quantity * reward.unit_price;
                }
            }
            //Remove entries from amountsToDiscount that are 0
            for (let key in amountsToDiscount) {
                if (amountsToDiscount[key] === 0) {
                    delete amountsToDiscount[key];
                }
            }
        },
        _getGroupKey: function (line) {
            return line
                .get_taxes()
                .map((tax) => tax.id)
                .join(',');
        },
        _createDiscountRewards: function (program, coupon_id, amountsToDiscount) {
            const rewards = [];
            const totalAmountsToDiscount = Object.values(amountsToDiscount).reduce((a, b) => a + b, 0);
            for (let [tax_keys, amount] of Object.entries(amountsToDiscount)) {
                let discountAmount = (amount * program.discount_percentage) / 100.0;
                let maxDiscount = amount / totalAmountsToDiscount * (program.discount_max_amount || Infinity);
                discountAmount = Math.min(discountAmount, maxDiscount);
                rewards.push(new Reward({
                    product: this.pos.db.get_product_by_id(program.discount_line_product_id[0]),
                    unit_price: -discountAmount,
                    quantity: 1,
                    program: program,
                    tax_ids: tax_keys !== '' ? tax_keys.split(',').map((val) => parseInt(val, 10)) : [],
                    coupon_id: coupon_id,
                }));
            }
            return [rewards, rewards.length > 0 ? null : 'No items to discount.'];
        }
    });

    var _orderline_super = models.Orderline.prototype;
    models.Orderline = models.Orderline.extend({
        export_as_JSON: function () {
            var result = _orderline_super.export_as_JSON.apply(this);
            result.is_program_reward = this.is_program_reward;
            result.program_id = this.program_id;
            result.coupon_id = this.coupon_id;
            return result;
        },
        init_from_JSON: function (json) {
            if (json.is_program_reward) {
                this.is_program_reward = json.is_program_reward;
                this.program_id = json.program_id;
                this.coupon_id = json.coupon_id;
                if (this.coupon_id && this.coupon_id[1]) {
                    this.order.bookedCouponCodes[this.coupon_id[1]] = new CouponCode(this.coupon_id[1], this.coupon_id[0], this.program_id);
                    this.coupon_id = json.coupon_id[0];
                } else if (json.program_id && this.order.activePromoProgramIds.indexOf(json.program_id) === -1) {
                    this.order.activePromoProgramIds.push(json.program_id);
                }
            }
            _orderline_super.init_from_JSON.apply(this, [json]);
        },
        set_quantity: function (quantity, keep_price) {
            const result = _orderline_super.set_quantity.apply(this, [quantity, keep_price]);
            // This function removes an order line if we set the quantity to 'remove'
            // We extend its functionality so that if a reward line is removed,
            // other reward lines from the same program are also deleted.
            if (quantity === 'remove' && this.is_program_reward) {
                let related_rewards = this.order.orderlines.filter(
                    (line) => line.is_program_reward && line.program_id === this.program_id
                );
                for (let line of related_rewards) {
                    line.order.remove_orderline(line);
                }
                if (related_rewards.length !== 0) {
                    Gui.showNotification('Other reward lines from the same program were also removed.');
                }
            }
            return result;
        },
    });

    return {CouponCode, RewardsContainer, Reward};
});

```

## File: static\src\js\Orderline.js

```javascript
odoo.define('pos_coupon.Orderline', function (require) {
    'use strict';

    const Orderline = require('point_of_sale.Orderline');
    const Registries = require('point_of_sale.Registries');

    const PosCouponOrderline = (Orderline) =>
        class extends Orderline {
            get addedClasses() {
                return Object.assign({ 'program-reward': this.props.line.is_program_reward }, super.addedClasses);
            }
        };

    Registries.Component.extend(Orderline, PosCouponOrderline);

    return Orderline;
});

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('pos_coupon.PaymentScreen', function (require) {
    'use strict';

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const session = require('web.session');

    const PosCouponPaymentScreen = (PaymentScreen) =>
        class extends PaymentScreen {
            async _postPushOrderResolve(order, server_ids) {
                const bookedCouponIds = new Set(
                    Object.values(order.bookedCouponCodes)
                        .map((couponCode) => couponCode.coupon_id)
                        .filter((coupon_id) => coupon_id)
                );
                const usedCouponIds = order.orderlines.models
                    .map((line) => line.coupon_id)
                    .filter((coupon_id) => coupon_id);
                for (let coupon_id of usedCouponIds) {
                    bookedCouponIds.delete(coupon_id);
                }
                const unusedCouponIds = [...bookedCouponIds.values()];
                order.generated_coupons = await this.rpc(
                    {
                        model: 'pos.order',
                        method: 'validate_coupon_programs',
                        args: [server_ids, order.programIdsToGenerateCoupons || [], unusedCouponIds],
                        kwargs: { context: session.user_context },
                    },
                    {}
                );
                return super._postPushOrderResolve(order, server_ids);
            }

            mounted() {
                super.mounted();
                this.currentOrder.on('rewards-updated', () => this.render(), this);
            }

            willUnmount() {
                super.willUnmount();
                this.currentOrder.off('rewards-updated', null, this);
            }

        };

    Registries.Component.extend(PaymentScreen, PosCouponPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\ProductScreen.js

```javascript
odoo.define('pos_coupon.ProductScreen', function (require) {
    'use strict';

    const ProductScreen = require('point_of_sale.ProductScreen');
    const Registries = require('point_of_sale.Registries');
    const { useBarcodeReader } = require('point_of_sale.custom_hooks');

    const PosCouponProductScreen = (ProductScreen) =>
        class extends ProductScreen {
            constructor() {
                super(...arguments);
                useBarcodeReader({
                    coupon: this._onCouponScan,
                });
            }
            _onCouponScan(code) {
                this.currentOrder.activateCode(code.base_code);
            }
            async _updateSelectedOrderline(event) {
                const selectedLine = this.currentOrder.get_selected_orderline();
                if (selectedLine && selectedLine.is_program_reward && event.detail.key === 'Backspace') {
                    const program = this.env.pos.coupon_programs_by_id[selectedLine.program_id]
                    const { confirmed } = await this.showPopup('ConfirmPopup', {
                        title: this.env._t('Deactivating program'),
                        body: _.str.sprintf(
                            this.env._t('Are you sure you want to deactivate %s in this order?'),
                            program.name
                        ),
                        cancelText: this.env._t('No'),
                        confirmText: this.env._t('Yes'),
                    });
                    if (confirmed) {
                        event.detail.buffer = null;
                    } else {
                        return; // do nothing on the line
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
                    !selectedLine.is_program_reward ||
                    (selectedLine.is_program_reward && ['', 'remove'].includes(val))
                ) {
                    super._setValue(val);
                }
                if (!selectedLine) return;
                if (selectedLine.is_program_reward && val === 'remove') {
                    if (selectedLine.coupon_id) {
                        const coupon_code = Object.values(selectedLine.order.bookedCouponCodes).find(
                            (couponCode) => couponCode.coupon_id === selectedLine.coupon_id
                        ).code;
                        delete selectedLine.order.bookedCouponCodes[coupon_code];
                        selectedLine.order.trigger('reset-coupons', [selectedLine.coupon_id]);
                        this.showNotification(`Coupon (${coupon_code}) has been deactivated.`);
                    } else if (selectedLine.program_id) {
                        // remove program from active programs
                        const index = selectedLine.order.activePromoProgramIds.indexOf(selectedLine.program_id);
                        selectedLine.order.activePromoProgramIds.splice(index, 1);
                        this.showNotification(
                            `'${
                                this.env.pos.coupon_programs_by_id[selectedLine.program_id].name
                            }' program has been deactivated.`
                        );
                    }
                }
                if (!selectedLine.is_program_reward || (selectedLine.is_program_reward && val === 'remove')) {
                    selectedLine.order.trigger('update-rewards');
                }
            }
        };

    Registries.Component.extend(ProductScreen, PosCouponProductScreen);

    return ProductScreen;
});

```

## File: static\src\js\ControlButtons\PromoCodeButton.js

```javascript
odoo.define('pos_coupon.PromoCodeButton', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class PromoCodeButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        async onClick() {
            const { confirmed, payload: code } = await this.showPopup('TextInputPopup', {
                title: this.env._t('Enter Promotion or Coupon Code'),
                startingValue: '',
            });
            if (confirmed && code !== '') {
                const order = this.env.pos.get_order();
                order.activateCode(code);
            }
        }
    }
    PromoCodeButton.template = 'PromoCodeButton';

    ProductScreen.addControlButton({
        component: PromoCodeButton,
        condition: function () {
            return this.env.pos.config.use_coupon_programs;
        },
    });

    Registries.Component.add(PromoCodeButton);

    return PromoCodeButton;
});

```

## File: static\src\js\ControlButtons\ResetProgramsButton.js

```javascript
odoo.define('pos_coupon.ResetProgramsButton', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const ProductScreen = require('point_of_sale.ProductScreen');
    const { useListener } = require('web.custom_hooks');
    const Registries = require('point_of_sale.Registries');

    class ResetProgramsButton extends PosComponent {
        constructor() {
            super(...arguments);
            useListener('click', this.onClick);
        }
        async onClick() {
            const order = this.env.pos.get_order();
            order.resetPrograms();
            this.trigger('close-popup');
        }
    }
    ResetProgramsButton.template = 'ResetProgramsButton';

    ProductScreen.addControlButton({
        component: ResetProgramsButton,
        condition: function () {
            return this.env.pos.config.use_coupon_programs;
        },
    });

    Registries.Component.add(ResetProgramsButton);

    return ResetProgramsButton;
});

```

## File: static\src\js\tours\PosCoupon1.tour.js

```javascript
odoo.define('pos_coupon.tour.pos_coupon1', function (require) {
    'use strict';

    // --- PoS Coupon Tour Basic Part 1 ---
    // Generate coupons for PosCouponTour2.

    const { PosCoupon } = require('pos_coupon.tour.PosCouponTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.confirmOpeningPopup();
    ProductScreen.do.clickHomeCategory();

    // basic order
    // just accept the automatically applied promo program
    // applied programs:
    //   - on cheapest product
    ProductScreen.exec.addOrderline('Whiteboard Pen', '5');
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.88');
    PosCoupon.do.selectRewardLine('on cheapest product');
    PosCoupon.check.orderTotalIs('13.12');
    PosCoupon.exec.finalizeOrder('Cash', '20');

    // remove the reward from auto promo program
    // no applied programs
    ProductScreen.exec.addOrderline('Whiteboard Pen', '6');
    PosCoupon.check.hasRewardLine('on cheapest product', '-2.88');
    PosCoupon.check.orderTotalIs('16.32');
    PosCoupon.exec.removeRewardLine('90.0% discount on cheapest product');
    PosCoupon.check.orderTotalIs('19.2');
    PosCoupon.exec.finalizeOrder('Cash', '20');

    // order with coupon code from coupon program
    // applied programs:
    //   - coupon program
    ProductScreen.exec.addOrderline('Desk Organizer', '9');
    PosCoupon.check.hasRewardLine('on cheapest product', '-4.59');
    PosCoupon.exec.removeRewardLine('90.0% discount on cheapest product');
    PosCoupon.check.orderTotalIs('45.90');
    PosCoupon.do.enterCode('invalid_code');
    PosCoupon.do.enterCode('1234');
    PosCoupon.check.hasRewardLine('Free Product - Desk Organizer', '-15.30');
    PosCoupon.exec.finalizeOrder('Cash', '50');

    // Use coupon but eventually remove the reward
    // applied programs:
    //   - on cheapest product
    ProductScreen.exec.addOrderline('Letter Tray', '4');
    ProductScreen.exec.addOrderline('Desk Organizer', '9');
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-4.32');
    PosCoupon.check.orderTotalIs('62.27');
    PosCoupon.do.enterCode('5678');
    PosCoupon.check.hasRewardLine('Free Product - Desk Organizer', '-15.30');
    PosCoupon.check.orderTotalIs('46.97');
    PosCoupon.exec.removeRewardLine('Free Product - Desk Organizer');
    PosCoupon.check.orderTotalIs('62.27');
    PosCoupon.exec.finalizeOrder('Cash', '90');

    // specific product discount
    // applied programs:
    //   - on cheapest product
    //   - on specific products
    ProductScreen.exec.addOrderline('Magnetic Board', '10') // 1.98
    ProductScreen.exec.addOrderline('Desk Organizer', '3') // 5.1
    ProductScreen.exec.addOrderline('Letter Tray', '4') // 4.8 tax 10%
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-1.78')
    PosCoupon.check.orderTotalIs('54.44')
    PosCoupon.do.enterCode('promocode')
    PosCoupon.check.hasRewardLine('50.0% discount on products', '-17.55')
    PosCoupon.check.orderTotalIs('36.89')
    PosCoupon.exec.finalizeOrder('Cash', '50')

    // code_promo_program_free_product
    // applied programs:
    //   - on cheapest product
    //   - free product different from criterion product
    //      (Buy 3 Whiteboard Pen, Take 1 Magnetic Board)
    PosCoupon.do.enterCode('board')
    ProductScreen.exec.addOrderline('Whiteboard Pen', '5') // 3.20 each
    // User should manually add the free product to get the reward.
    ProductScreen.exec.addOrderline('Magnetic Board', '1') // 1.98
    PosCoupon.check.hasRewardLine('Free Product - Magnetic Board', '-1.98') // meaning 1 item
    // cheapest product should point to Whiteboard Pen and not the added Magnetic Board
    // even though Whiteboard Pen ($3.20) costs more than Magnetic Board ($1.98).
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.88')
    PosCoupon.check.orderTotalIs('13.12')
    PosCoupon.exec.finalizeOrder('Cash', '20')

    Tour.register('PosCouponTour1', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosCoupon2.tour.js

```javascript
odoo.define('pos_coupon.tour.pos_coupon2', function (require) {
    'use strict';

    // --- PoS Coupon Tour Basic Part 2 ---
    // Using the coupons generated from PosCouponTour1.

    const { PosCoupon } = require('pos_coupon.tour.PosCouponTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.clickHomeCategory();

    // Cheapest product discount should be replaced by the global discount
    // because it's amount is lower.
    // Applied programs:
    //   - global discount
    ProductScreen.exec.addOrderline('Desk Organizer', '10'); // 5.1
    PosCoupon.check.hasRewardLine('on cheapest product', '-4.59');
    ProductScreen.exec.addOrderline('Letter Tray', '4'); // 4.8 tax 10%
    PosCoupon.check.hasRewardLine('on cheapest product', '-4.32');
    PosCoupon.do.enterCode('123456');
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-5.10');
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-1.92');
    PosCoupon.check.orderTotalIs('64.91');
    PosCoupon.exec.finalizeOrder('Cash', '70');

    // Use coupon from global discount but on cheapest discount prevails.
    // The global discount coupon should be consumed during the order as it is
    // activated in the order. But upon validation, the coupon should return
    // to new state.
    // Applied programs:
    //   - on cheapest discount
    ProductScreen.exec.addOrderline('Small Shelf', '3'); // 2.83 per item
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.55');
    PosCoupon.do.enterCode('345678');
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.55');
    ProductScreen.exec.addOrderline('Desk Organizer', '9'); // 4.80 per item
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-5.44');
    ProductScreen.do.pressNumpad('Backspace Backspace')
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.55');
    ProductScreen.exec.addOrderline('Desk Pad', '1'); // 1.98
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-1.78');
    PosCoupon.check.orderTotalIs('8.69');
    PosCoupon.exec.finalizeOrder('Cash', '10');

    // Scanning coupon twice.
    // Also apply global discount on top of free product to check if the
    // calculated discount is correct.
    // Applied programs:
    //  - coupon program (free product)
    //  - global discount
    ProductScreen.exec.addOrderline('Desk Organizer', '11'); // 5.1 per item
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-4.59');
    PosCoupon.check.orderTotalIs('51.51');
    // add global discount and the discount will be replaced
    PosCoupon.do.enterCode('345678');
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-5.61');
    // add free product coupon (for qty=11, free=4)
    // the discount should change after having free products
    // it should go back to cheapest discount as it is higher
    PosCoupon.do.enterCode('5678');
    PosCoupon.check.hasRewardLine('Free Product - Desk Organizer', '-20.40');
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-4.59');
    // set quantity to 18
    // should result to 'charged qty'=12, 'free qty'=6
    ProductScreen.do.pressNumpad('Backspace 8')
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-6.12');
    PosCoupon.check.hasRewardLine('Free Product - Desk Organizer', '-30.60');
    // scan the code again and check notification
    PosCoupon.do.enterCode('5678');
    PosCoupon.check.orderTotalIs('55.08');
    PosCoupon.exec.finalizeOrder('Cash', '60');

    // Specific products discount (with promocode) and free product (1357)
    // Applied programs:
    //   - discount on specific products
    //   - free product
    ProductScreen.exec.addOrderline('Desk Organizer', '6'); // 5.1 per item
    PosCoupon.check.hasRewardLine('on cheapest product', '-4.59');
    PosCoupon.exec.removeRewardLine('90.0% discount on cheapest product');
    PosCoupon.do.enterCode('promocode');
    PosCoupon.check.hasRewardLine('50.0% discount on products', '-15.30');
    PosCoupon.do.enterCode('1357');
    PosCoupon.check.hasRewardLine('Free Product - Desk Organizer', '-10.20');
    PosCoupon.check.hasRewardLine('50.0% discount on products', '-10.20');
    PosCoupon.check.orderTotalIs('10.20');
    PosCoupon.exec.finalizeOrder('Cash', '20');

    // Check reset program
    // Enter two codes and reset the programs.
    // The codes should be checked afterwards. They should return to new.
    // Applied programs:
    //   - cheapest product
    PosCoupon.do.enterCode('2468');
    PosCoupon.do.enterCode('098765');
    ProductScreen.exec.addOrderline('Monitor Stand', '6'); // 3.19 per item
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.87');
    PosCoupon.check.orderTotalIs('16.27');
    PosCoupon.exec.removeRewardLine('90.0% discount on cheapest product');
    PosCoupon.check.hasRewardLine('10.0% discount on total amount', '-1.91');
    PosCoupon.do.resetActivePrograms();
    PosCoupon.check.hasRewardLine('90.0% discount on cheapest product', '-2.87');
    PosCoupon.check.orderTotalIs('16.27');
    PosCoupon.exec.finalizeOrder('Cash', '20');

    Tour.register('PosCouponTour2', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosCoupon3tour.js

```javascript
odoo.define('pos_coupon.tour.pos_coupon3', function (require) {
    'use strict';

    // --- PoS Coupon Tour Basic Part 3 ---

    const { PosCoupon } = require('pos_coupon.tour.PosCouponTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.confirmOpeningPopup();
    ProductScreen.do.clickHomeCategory();

    ProductScreen.do.clickDisplayedProduct('Promo Product');
    PosCoupon.check.orderTotalIs('34.50');
    ProductScreen.do.clickDisplayedProduct('Product B');
    PosCoupon.check.hasRewardLine('100.0% discount on products', '25.00');
    ProductScreen.do.clickDisplayedProduct('Product A');
    PosCoupon.check.hasRewardLine('100.0% discount on products', '15.00');
    PosCoupon.check.orderTotalIs('34.50');
    ProductScreen.do.clickDisplayedProduct('Product A');
    PosCoupon.check.hasRewardLine('100.0% discount on products', '21.82');
    PosCoupon.check.hasRewardLine('100.0% discount on products', '18.18');
    PosCoupon.check.orderTotalIs('49.50');


    Tour.register('PosCouponTour3', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosCoupon4.tour.js

```javascript
odoo.define('pos_coupon.tour.pos_coupon4', function (require) {
    'use strict';

    // --- PoS Coupon Tour Basic Part 2 ---
    // Using the coupons generated from PosCouponTour1.

    const { PosCoupon } = require('pos_coupon.tour.PosCouponTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { PaymentScreen } = require('point_of_sale.tour.PaymentScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.clickHomeCategory();

    ProductScreen.exec.addOrderline('Test Product 1', '1');
    ProductScreen.exec.addOrderline('Test Product 2', '1');
    ProductScreen.do.clickPricelistButton();
    ProductScreen.do.selectPriceList('Public Pricelist');
    PosCoupon.do.enterCode('abcda');
    PosCoupon.check.orderTotalIs('0.00');
    ProductScreen.do.clickPricelistButton();
    ProductScreen.do.selectPriceList('Test multi-currency');
    PosCoupon.check.orderTotalIs('0.00');


    Tour.register('PosCouponTour4', { test: true, url: '/pos/web' }, getSteps());

    startSteps();

    ProductScreen.do.clickHomeCategory();

    ProductScreen.do.clickDisplayedProduct('Test Product 1');
    ProductScreen.do.clickDisplayedProduct('Test Product 2');
    ProductScreen.do.clickPricelistButton();
    ProductScreen.do.selectPriceList('Public Pricelist');
    PosCoupon.check.orderTotalIs('53.75');
    PosCoupon.do.enterCode('abcdb');
    PosCoupon.check.orderTotalIs('0.00');
    ProductScreen.do.clickPayButton();
    PaymentScreen.check.dueIs('0.00');
    PaymentScreen.do.clickCustomerButton();
    PaymentScreen.do.clickCustomer('Colleen Diaz');
    PaymentScreen.do.clickValidateCustomer();
    PaymentScreen.check.dueIs('0.00');

    Tour.register('PosCouponTour4.1', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosCoupon5.tour.js

```javascript
odoo.define('pos_coupon.tour.pos_coupon5', function (require) {
    'use strict';

    // A tour that add a product, add a coupon, add a global discount, and check the lines content.

    const { PosCoupon } = require('pos_coupon.tour.PosCouponTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.clickHomeCategory();

    ProductScreen.exec.addOrderline('Test Product 1', '1.00', '100');
    PosCoupon.do.clickDiscountButton();
    PosCoupon.do.clickConfirmButton();
    ProductScreen.check.totalAmountIs('93.15');

    Tour.register('PosCouponTour5', { test: true, url: '/pos/web' }, getSteps());

    startSteps();

    ProductScreen.do.clickHomeCategory();
    ProductScreen.do.confirmOpeningPopup();

    ProductScreen.exec.addOrderline('Test Product 1', '1.00', '100');
    ProductScreen.do.clickCustomerButton();
    ProductScreen.do.clickCustomer('Test Partner');
    ProductScreen.do.clickSetCustomer();
    PosCoupon.do.clickRewardButton();
    ProductScreen.check.totalAmountIs('93.50');

    Tour.register('PosCouponTour5.1', { test: true, url: '/pos/web' }, getSteps());


    startSteps();

    ProductScreen.do.clickHomeCategory();
    ProductScreen.do.confirmOpeningPopup();

    ProductScreen.do.clickDisplayedProduct('Product B');
    ProductScreen.do.clickDisplayedProduct('Product A');
    ProductScreen.check.totalAmountIs('50.00');

    Tour.register('PosCouponTour5.2', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosCouponTourMethods.js

```javascript
odoo.define('pos_coupon.tour.PosCouponTourMethods', function (require) {
    'use strict';

    const { createTourMethods } = require('point_of_sale.tour.utils');
    const { Do: ProductScreenDo } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { Do: PaymentScreenDo } = require('point_of_sale.tour.PaymentScreenTourMethods');
    const { Do: ReceiptScreenDo } = require('point_of_sale.tour.ReceiptScreenTourMethods');
    const { Do: ChromeDo } = require('point_of_sale.tour.ChromeTourMethods');

    const ProductScreen = { do: new ProductScreenDo() };
    const PaymentScreen = { do: new PaymentScreenDo() };
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
            return [
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
            ];
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
                    content: 'open code input dialog',
                    trigger: '.control-button:contains("Reward")',
                },
            ];
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
        hasRewardLine(rewardName, amount) {
            return [
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
    }

    class Execute {
        constructor() {
            this.do = new Do();
            this.check = new Check();
        }
        finalizeOrder(paymentMethod, amount) {
            return [
                ...ProductScreen.do.clickPayButton(),
                ...PaymentScreen.do.clickPaymentMethod(paymentMethod),
                ...PaymentScreen.do.pressNumpad([...amount].join(' ')),
                ...PaymentScreen.do.clickValidate(),
                ...ReceiptScreen.do.clickNextOrder(),
            ];
        }
        removeRewardLine(name) {
            return [
                ...this.do.selectRewardLine(name),
                ...ProductScreen.do.pressNumpad('Backspace'),
                ...Chrome.do.confirmPopup(),
            ];
        }
    }

    return createTourMethods('PosCoupon', Do, Check, Execute);
});

```

## File: static\src\xml\ActivePrograms.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ActivePrograms" owl="1">
        <div class="active-programs">
            <div t-if="currentOrder and renderParams.show">
                <div class="title">Active Programs</div>
                <t t-foreach="renderParams.withRewardsPromoPrograms" t-as="program" t-key="program.id">
                    <div>
                        <t t-esc="program.name"/>
                        <span t-if="program.promo_code !== false">
                            (<t t-esc="program.promo_code"/>)
                        </span>
                    </div>
                </t>
                <t t-foreach="renderParams.withRewardsBookedCoupons" t-as="coupon" t-key="coupon.coupon_code">
                    <div>
                        <t t-esc="coupon.program_name"/> (<t t-esc="coupon.coupon_code"/>)
                    </div>
                </t>
                <t t-foreach="renderParams.onNextOrderPromoPrograms" t-as="program" t-key="program.id">
                    <div style="font-style: italic;">
                        <t t-esc="program.name"/>
                        <span t-if="program.promo_code !== false">
                            (<t t-esc="program.promo_code"/>)
                        </span>
                    </div>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_coupon.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('before-footer')]" position="inside">
            <t t-if="receipt.generated_coupons and receipt.generated_coupons.length !== 0">
                <div class="pos-coupon-rewards">
                    <div>------------------------</div>
                    <br/>
                    <div>
                        Coupon Codes
                    </div>
                    <t t-foreach="receipt.generated_coupons" t-as="coupon_info" t-key="coupon_info.code">
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

## File: static\src\xml\OrderWidget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_coupon.OrderWidget" t-inherit="point_of_sale.OrderWidget" t-inherit-mode="extension" owl="1">
        <xpath expr="//OrderSummary" position="after">
            <ActivePrograms />
        </xpath>
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

## File: views\coupon_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Menu Items -->
    <menuitem
        id="menu_coupon_type_config"
        action="coupon.coupon_program_action_coupon_program"
        parent="point_of_sale.pos_config_menu_catalog"
        name="Coupon Programs"
        groups="point_of_sale.group_pos_manager"
        sequence="91"
    />

    <menuitem
        id="menu_promotion_type_config"
        action="coupon.coupon_program_action_promo_program"
        parent="point_of_sale.pos_config_menu_catalog"
        name="Promotion Programs"
        groups="point_of_sale.group_pos_manager"
        sequence="90"
    />

    <!-- Form Views -->
    <record id="pos_coupon_program_view_coupon_program_form" model="ir.ui.view">
        <field name="name">coupon.program.form</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon.coupon_program_view_coupon_program_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='coupon_count']/.." position="before">
                <button class="oe_stat_button" type="object" icon="fa-usd" name="action_view_pos_orders">
                    <field name="pos_order_count" string="PoS Sales" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="pos_coupon_program_view_promo_program_form" model="ir.ui.view">
        <field name="name">coupon.program.form</field>
        <field name="model">coupon.program</field>
        <field name="inherit_id" ref="coupon.coupon_program_view_promo_program_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='promo_code']" position="after">
                <field name="promo_barcode" attrs="{'invisible': [('promo_code_usage', '=', 'no_code_needed')]}" readonly="1" groups="base.group_no_one"/>
            </xpath>
            <xpath expr="//field[@name='coupon_count']/.." position="before">
                <button class="oe_stat_button" type="object" icon="fa-usd" name="action_view_pos_orders">
                    <field name="pos_order_count" string="PoS Sales" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\coupon_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="pos_coupon_view_form" model="ir.ui.view">
        <field name="name">coupon.coupon.form</field>
        <field name="model">coupon.coupon</field>
        <field name="inherit_id" ref="coupon.coupon_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="pos_order_id" invisible="1"/>
                <field name="source_pos_order_id" invisible="1"/>
                <field name="pos_order_id" readonly="1" attrs="{'invisible': [('pos_order_id', '=', False)]}" />
                <field name="source_pos_order_id" readonly="1" attrs="{'invisible': [('source_pos_order_id', '=', False)]}" />
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_coupon_pos_config_view_form" model="ir.ui.view">
        <field name="name">pos.config.form</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='pos-loyalty']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="pos-coupon">
                    <div class="o_setting_left_pane">
                        <field name="use_coupon_programs" nolabel="1"/>
                    </div>
                    <div class="o_setting_right_pane" title="Define the coupon and promotion programs you can use in this PoS.">
                        <label for="use_coupon_programs"/>
                        <div class="text-muted">
                            Define the coupon and promotion programs you can use in this PoS.
                        </div>
                        <div attrs="{'invisible': [('use_coupon_programs', '=', False)]}" title="Promotion &amp; coupon programs to use.">
                            <div class="content-group">
                                <div class="row mt16">
                                    <label for="promo_program_ids" class="col-lg-3 o_light_label"/>
                                    <field name="promo_program_ids"
                                        widget="many2many_tags"
                                        context="{'form_view_ref': 'coupon.coupon_program_view_promo_program_form'}"
                                        domain="[('program_type', '=', 'promotion_program'), ('active', '=', True)]" />
                                </div>
                            </div>
                            <div class="content-group">
                                <div class="row mt16">
                                    <label for="coupon_program_ids" class="col-lg-3 o_light_label"/>
                                    <field name="coupon_program_ids"
                                        widget="many2many_tags"
                                        context="{'form_view_ref': 'coupon.coupon_program_view_coupon_program_form'}"
                                        domain="[('program_type', '=', 'coupon_program'), ('active', '=', True)]"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_view_form_inherit_pos_coupon" model="ir.ui.view">
        <field name="name">res.config.form.inherit.pos.coupon</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='pos-coupon']/div[last()]" position="after">
                <div class="mt8" attrs="{'invisible': [('module_pos_coupon', '=', False)]}">
                    <button name="%(coupon.coupon_program_action_promo_program)d" icon="fa-arrow-right" type="action" string="Promotion Programs" class="btn-link"/>
                </div>
                <div class="mt8" attrs="{'invisible': [('module_pos_coupon', '=', False)]}">
                    <button name="%(coupon.coupon_program_action_coupon_program)d" icon="fa-arrow-right" type="action" string="Coupon Programs" class="btn-link"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

