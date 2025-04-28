# Odoo Module: pos_gift_card

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
    'name': "Gift Card for point of sales module",
    'summary': "Use gift card in your sales orders",
    'description': """Integrate gift card mechanism in sales orders.""",
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['gift_card', 'point_of_sale'],
    'auto_install': True,
    'data': [
        'data/gift_card_data.xml',
        'views/gift_card_views.xml',
        'views/res_config_settings_views.xml',
        'views/pos_config_views.xml',
        'security/ir.model.access.csv',
    ],
    'assets': {
        'point_of_sale.assets': [
            'pos_gift_card/static/src/css/giftCard.css',
            'pos_gift_card/static/src/js/models.js',
            'pos_gift_card/static/src/js/GiftCardButton.js',
            'pos_gift_card/static/src/js/GiftCardPopup.js',
            'pos_gift_card/static/src/js/PaymentScreen.js',
        ],
        'web.assets_qweb': [
            'pos_gift_card/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'pos_gift_card/static/src/js/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\gift_card_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- the product used to deduce on sale order -->
        <record id="gift_card.pay_with_gift_card_product" model="product.product">
            <field name="available_in_pos">True</field>
        </record>

        <record id="gift_card_report_pdf" model="ir.actions.report">
            <field name="name">Gift Card</field>
            <field name="model">gift.card</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">pos_gift_card.gift_card_template</field>
            <field name="binding_type">report</field>
            <field name="binding_model_id" ref="model_gift_card"/>
        </record>

        <template id="gift_card_template">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="o">
                    <t t-call="web.external_layout">
                        <div style="margin:0px; font-size:24px; font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:36px; color:#333333; text-align: center">
                            Here is your gift card!
                        </div>
                        <div style="padding-top:20px; padding-bottom:20px">
                            <img src="/gift_card/static/img/gift_card.png" style="display:block; border:0; outline:none; text-decoration:none; margin:auto;" width="300"/>
                        </div>
                        <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; text-align:center;">
                            <h3 style="margin:0px; line-height:48px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:40px; font-style:normal; font-weight:normal; color:#333333; text-align:center">
                                <strong><span t-field="o.initial_amount"/></strong>
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
                        <div style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                            <h3 style="margin:0px; line-height:17px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:14px; font-style:normal; font-weight:normal; color:#A9A9A9; text-align:center">
                                Card expires <span t-field="o.expired_date"/>
                            </h3>
                        </div>
                        <div style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                            <img t-att-src="'/report/barcode/Code128/'+o.code" style="width:400px;height:75px" alt="Barcode"/>
                        </div>
                    </t>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: models\barcode_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class BarcodeRule(models.Model):
    _inherit = 'barcode.rule'

    type = fields.Selection(selection_add=[
        ('gift_card', 'Gift Card'),
    ], ondelete={
        'gift_card': 'set default',
    })

```

## File: models\gift_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class GiftCard(models.Model):
    _inherit = "gift.card"

    buy_pos_order_line_id = fields.Many2one(
        "pos.order.line",
        copy=False,
        readonly=True,
        help="Pos Order line where this gift card has been bought.",
    )
    redeem_pos_order_line_ids = fields.One2many(
        "pos.order.line", "gift_card_id", string="Pos Redeems"
    )

    def _get_confirmed_redeem_pos_order_lines(self):
        self.ensure_one()
        return self.redeem_pos_order_line_ids.sudo().filtered(
                lambda l: l.order_id.state in ('paid', 'done', 'invoiced')
            )

    @api.depends("redeem_pos_order_line_ids")
    def _compute_balance(self):
        super()._compute_balance()
        for record in self:
            confirmed_line = record._get_confirmed_redeem_pos_order_lines()
            balance = record.balance
            if confirmed_line:
                balance -= sum(
                    confirmed_line.mapped(
                        lambda line: line.currency_id._convert(
                            line.price_unit,
                            record.currency_id,
                            record.env.company,
                            line.create_date,
                        )
                        * -1
                    )
                )
            record.balance = balance

    def can_be_used_in_pos(self, sale_order_origin_id=False):
        # expired state are computed once a day, so can be not synchro
        return self.state == 'valid' and self.balance > 0 and (not self.expired_date or self.expired_date >= fields.Date.today())

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosConfig(models.Model):
    _inherit = "pos.config"

    use_gift_card = fields.Boolean(string="Gift Card")

    gift_card_product_id = fields.Many2one(
        "product.product",
        string="Gift Card Product",
        help="This product is used as reference on customer receipts.",
    )

    gift_card_settings = fields.Selection(
        [
            ("create_set", "Generate a new barcode and set a price"),
            ("scan_set", "Scan an existing barcode and set a price"),
            ("scan_use", "Scan an existing barcode with an existing price"),
        ],
        string="Gift Cards settings",
        default="create_set",
        help="Defines the way you want to set your gift cards.",
    )

    @api.onchange("use_gift_card")
    def _onchange_giftproduct(self):
        if self.use_gift_card:
            self.gift_card_product_id = self.env.ref(
                "gift_card.pay_with_gift_card_product", False
            )
        else:
            self.gift_card_product_id = False

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
import base64


class PosOrder(models.Model):
    _inherit = "pos.order"

    gift_card_count = fields.Integer(compute="_compute_gift_card_count")

    @api.depends("lines.generated_gift_card_ids")
    def _compute_gift_card_count(self):
        for record in self:
            record.gift_card_count = len(record.lines.mapped("generated_gift_card_ids"))

    @api.model
    def create_from_ui(self, orders, draft=False):
        order_ids = super(PosOrder, self).create_from_ui(orders, draft)
        for order in self.sudo().browse([o["id"] for o in order_ids]):
            gift_card_config = order.config_id.gift_card_settings
            for line in order.lines:
                if line.product_id.id == order.config_id.gift_card_product_id.id:
                    if not line.gift_card_id:
                        if gift_card_config == "create_set":
                            new_card = line._create_gift_cards()
                            new_card.partner_id = order.partner_id or False
                            line.generated_gift_card_ids = new_card
                        else:
                            gift_card = self.env["gift.card"].search(
                                [("id", "=", line.generated_gift_card_ids.id)]
                            )
                            gift_card.buy_pos_order_line_id = line.id
                            gift_card.expired_date = fields.Date.add(
                                fields.Date.today(), years=1
                            )
                            gift_card.partner_id = order.partner_id or False

                            if gift_card_config == "scan_set":
                                gift_card.initial_amount = line.price_unit

        return order_ids

    def get_new_card_ids(self):
        return self.lines.mapped("generated_gift_card_ids").ids

    def _add_mail_attachment(self, name, ticket):
        attachment = super()._add_mail_attachment(name, ticket)
        if self.config_id.use_gift_card and len(self.get_new_card_ids()) > 0:
            report = self.env.ref('pos_gift_card.gift_card_report_pdf')._render_qweb_pdf(self.get_new_card_ids())
            filename = name + '.pdf'
            gift_card = self.env['ir.attachment'].create({
                'name': filename,
                'type': 'binary',
                'datas': base64.b64encode(report[0]),
                'store_fname': filename,
                'res_model': 'pos.order',
                'res_id': self.ids[0],
                'mimetype': 'application/x-pdf'
            })
            attachment += [(4, gift_card.id)]

        return attachment

```

## File: models\pos_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    generated_gift_card_ids = fields.One2many(
        "gift.card", "buy_pos_order_line_id", string="Bought Gift Card"
    )
    gift_card_id = fields.Many2one(
        "gift.card", help="Deducted from this Gift Card", copy=False
    )

    def _is_not_sellable_line(self):
        return self.gift_card_id or super()._is_not_sellable_line()

    def _create_gift_cards(self):
        return self.env["gift.card"].create(
            [self._build_gift_card() for _ in range(int(self.qty))]
        )

    def _build_gift_card(self):
        return {
            "initial_amount": self.order_id.currency_id._convert(
                self.price_unit,
                self.company_id.currency_id,
                self.company_id,
                fields.Date.today(),
            ),
            "buy_pos_order_line_id": self.id,
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import barcode_rule
from . import gift_card
from . import pos_config
from . import pos_order
from . import pos_order_line

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_gift_card_sales,Gift Card Program Salesman,model_gift_card,point_of_sale.group_pos_manager,1,0,0,0
access_pos_gift_card_manager,Gift Card Program Sale Manager,model_gift_card,point_of_sale.group_pos_manager,1,1,1,1

```

## File: static\src\js\GiftCardButton.js

```javascript
odoo.define("pos_gift_card.GiftCardButton", function (require) {
  "use strict";

  const PosComponent = require("point_of_sale.PosComponent");
  const ProductScreen = require("point_of_sale.ProductScreen");
  const { useListener } = require("web.custom_hooks");
  const Registries = require("point_of_sale.Registries");
  const { Gui } = require("point_of_sale.Gui");

  class GiftCardButton extends PosComponent {
    constructor() {
      super(...arguments);
      useListener("click", this.onClick);
    }
    async onClick() {
      this.showPopup("GiftCardPopup", {});
    }
  }
  GiftCardButton.template = "GiftCardButton";

  ProductScreen.addControlButton({
    component: GiftCardButton,
    condition: function () {
      return this.env.pos.config.use_gift_card;
    },
  });

  Registries.Component.add(GiftCardButton);

  return GiftCardButton;
});

```

## File: static\src\js\GiftCardPopup.js

```javascript
odoo.define("pos_gift_card.GiftCardPopup", function (require) {
  "use strict";

  const { useState, useRef, onPatched, useComponent} = owl.hooks;
  const AbstractAwaitablePopup = require("point_of_sale.AbstractAwaitablePopup");
  const Registries = require("point_of_sale.Registries");

  class GiftCardPopup extends AbstractAwaitablePopup {
    constructor() {
      super(...arguments);
      this.state = useState({
        giftCardConfig: this.env.pos.config.gift_card_settings,
        showBarcodeGeneration: false,
        showNewGiftCardMenu: false,
        showUseGiftCardMenu: false,
        showGiftCardDetails: false,
        amountToSet: 0,
        giftCardBarcode: "",
      });
      this.useAutoFocus(this.state);
    }

    useAutoFocus(state) {
      const component = useComponent();
      let hasFocused = false;
      function autofocus() {
          if (state.showBarcodeGeneration) {
              // Should autofocus here but only if it hasn't autofocus yet.
              if (!hasFocused) {
                  const elem = component.el.querySelector(`.giftCardPopupInput`);
                  if (elem) {
                      elem.focus();
                      hasFocused = true;
                  }
              }
          } else {
              // When changing showBarcodeGeneration to false, we reset hasFocused.
              hasFocused = false;
          }
      }
      onPatched(autofocus);
  }

    switchBarcodeView() {
      this.state.showBarcodeGeneration = !this.state.showBarcodeGeneration;
      if (this.state.showUseGiftCardMenu)
        this.state.showUseGiftCardMenu = false;
      if (this.state.showGiftCardDetails)
        this.state.showGiftCardDetails = false;
    }

    switchToUseGiftCardMenu() {
      this.switchBarcodeView();
      this.state.showUseGiftCardMenu = true;
    }

    switchToShowGiftCardDetails() {
      this.switchBarcodeView();
      this.state.showGiftCardDetails = true;
    }

    async addGiftCardProduct(giftCard) {
      let gift =
        this.env.pos.db.product_by_id[
          this.env.pos.config.gift_card_product_id[0]
        ];

      let can_be_sold = true;
      if (giftCard) {
        can_be_sold = !(giftCard.buy_pos_order_line_id || giftCard.buy_line_id);
      }

      if (can_be_sold) {
        await this.env.pos.get_order().add_product(gift, {
          price: this.state.amountToSet,
          quantity: 1,
          merge: false,
          generated_gift_card_ids: giftCard ? giftCard.id : false,
          extras: { price_automatically_set: true },
        });
      } else {
        await this.showPopup('ErrorPopup', {
          'title': this.env._t('This gift card has already been sold'),
          'body': this.env._t('You cannot sell a gift card that has already been sold'),
        });
      }
    }

    async getGiftCard() {
      if (this.state.giftCardBarcode == "") return;

      let giftCard = await this.rpc({
          model: "gift.card",
          method: "search_read",
          args: [[["code", "=", this.state.giftCardBarcode]]],
        });
        if (giftCard.length) {
          giftCard = giftCard[0];
        } else {
          return false;
        }

      return giftCard;
    }

    async scanAndUseGiftCard() {
      let giftCard = await this.getGiftCard();
      if (!giftCard) return;

      if (this.state.giftCardConfig === "scan_use")
        this.state.amountToSet = giftCard.initial_amount;

      await this.addGiftCardProduct(giftCard);
      this.cancel();
    }

    async generateBarcode() {
      await this.addGiftCardProduct(false);
      this.confirm();
    }

    async isGiftCardAlreadyUsed() {
      let order = this.env.pos.get_order();
      let giftProduct =
        this.env.pos.db.product_by_id[
          this.env.pos.config.gift_card_product_id[0]
        ];

      for (let line of order.orderlines.models) {
        if (line.product.id === giftProduct.id && line.price < 0) {
          if (line.gift_card_id === (await this.getGiftCard()).id) return line;
        }
      }
      return false;
    }

    getPriceToRemove(giftCard) {
      let currentOrder = this.env.pos.get_order();
      return currentOrder.get_total_with_tax() > giftCard.balance
        ? -giftCard.balance
        : -currentOrder.get_total_with_tax();
    }

    async payWithGiftCard() {
      let giftCard = await this.getGiftCard();
      if (!giftCard) return;

      let gift =
        this.env.pos.db.product_by_id[
          this.env.pos.config.gift_card_product_id[0]
        ];

      let currentOrder = this.env.pos.get_order();
      let lineUsed = await this.isGiftCardAlreadyUsed()
      if (lineUsed) currentOrder.remove_orderline(lineUsed);

      await currentOrder.add_product(gift, {
        price: this.getPriceToRemove(giftCard),
        quantity: 1,
        merge: false,
        gift_card_id: giftCard.id,
        extras: { price_automatically_set: true },
      });

      this.cancel();
    }

    async ShowRemainingAmount() {
      let giftCard = await this.getGiftCard();
      if (!giftCard) return;

      this.state.amountToSet = giftCard.balance;
    }
  }
  GiftCardPopup.template = "GiftCardPopup";

  Registries.Component.add(GiftCardPopup);

  return GiftCardPopup;
});

```

## File: static\src\js\models.js

```javascript
odoo.define("pos_gift_card.gift_card", function (require) {
  "use strict";

  const models = require("point_of_sale.models");
  const core = require('web.core');
  const _t = core._t;

  models.load_fields("pos.order.line", "generated_gift_card_ids");

    // Load the products used for creating program reward lines.
    var existing_models = models.PosModel.prototype.models;
    var product_index = _.findIndex(existing_models, function (model) {
        return model.model === 'product.product';
    });
    var product_model = existing_models[product_index];

  models.load_models([
    {
        model: product_model.model,
        fields: product_model.fields,
        order: product_model.order,
        domain: function (self) {
            return [['id', '=', self.config.gift_card_product_id[0]]];
        },
        context: product_model.context,
        loaded: product_model.loaded,
    },
  ]);

  var _order_super = models.Order.prototype;
  models.Order = models.Order.extend({
    //@override
    set_orderline_options: function (orderline, options) {
      _order_super.set_orderline_options.apply(this, [orderline, options]);
      if (options && options.generated_gift_card_ids) {
        orderline.generated_gift_card_ids = [options.generated_gift_card_ids];
      }
      if (options && options.gift_card_id) {
        if (orderline.order.orderlines.find((line) => line.gift_card_id === options.gift_card_id)) {
            throw new Error(_t('This gift card is already applied'));
        }
        orderline.gift_card_id = options.gift_card_id;
      }
    },
    //@override
    wait_for_push_order: function () {
        if(this.pos.config.use_gift_card) {
            let giftProduct = this.pos.db.product_by_id[this.pos.config.gift_card_product_id[0]];
            for (let line of this.orderlines.models) {
                if(line.product.id === giftProduct.id)
                    return true;
            }
        }
        return _order_super.wait_for_push_order.apply(this, arguments);
    },
    //@override
    _reduce_total_discount_callback: function(sum, orderLine) {
        if (this.pos.config.gift_card_product_id[0] === orderLine.product.id) {
            return sum;
        }
        return _order_super._reduce_total_discount_callback.apply(this, arguments);
    },

  });

  var _super_orderline = models.Orderline;
  models.Orderline = models.Orderline.extend({
    export_as_JSON: function () {
      var json = _super_orderline.prototype.export_as_JSON.apply(
        this,
        arguments
      );
      json.generated_gift_card_ids = this.generated_gift_card_ids;
      json.gift_card_id = this.gift_card_id;
      return json;
    },
    init_from_JSON: function (json) {
      _super_orderline.prototype.init_from_JSON.apply(this, arguments);
      this.generated_gift_card_ids = json.generated_gift_card_ids;
      this.gift_card_id = json.gift_card_id;
    },
  });

  var _posmodel_super = models.PosModel.prototype;
    models.PosModel = models.PosModel.extend({
        print_gift_pdf: function (giftCardIds) {
            this.do_action('pos_gift_card.gift_card_report_pdf', {
                additional_context: {
                    active_ids: [giftCardIds],
                },
            })
        }
    });
});

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('pos_gift_card.PaymentScreen', function(require) {
    "use strict";

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    var core = require('web.core');
    var _t = core._t;


    const PosGiftCardPaymentScreen = PaymentScreen => class extends PaymentScreen {
        //@Override
        async validateOrder(isForceValidate) {
            if(this.env.pos.config.use_gift_card) {
                if (await this._isOrderValid(isForceValidate)) {
                    try {
                        let giftProduct = this.env.pos.db.product_by_id[this.env.pos.config.gift_card_product_id[0]];

                        for (let line of this.currentOrder.orderlines.models) {
                            if(line.product.id === giftProduct.id && line.price <= 0) {
                                let is_valid = await this.isGiftCardValid(line);
                                if(!is_valid) {
                                    await this.showPopup('ErrorPopup', {
                                        'title': _t("Gift Card Error"),
                                        'body': _t("Gift card is not valid."),
                                    });
                                    return;
                                }

                                let gift_card = await this.rpc({
                                    model: "gift.card",
                                    method: 'search_read',
                                    domain: [['id', '=', line.gift_card_id]],
                                    fields: ['balance'],
                                  });

                                if(Math.abs(line.get_unit_price()) > gift_card[0].balance) {
                                    await this.showPopup('ErrorPopup', {
                                        'title': _t("Gift Card Error"),
                                        'body': _t("Gift card balance is too low."),
                                    });
                                    return;
                                }
                            }
                        }
                    } catch (e) {
                        // do nothing with the error
                    }
                } else {
                    return; // do nothing if the order is not valid
                }
            }
            await super.validateOrder(...arguments);
        }

        async _postPushOrderResolve(order, server_ids) {
            if(this.env.pos.config.use_gift_card) {
                let ids = await this.rpc({
                    model: 'pos.order',
                    method: 'get_new_card_ids',
                    args: [server_ids]
                });
                if(ids.length > 0)
                    this.env.pos.print_gift_pdf(ids);
            }
            return super._postPushOrderResolve(order, server_ids);
        }

        async isGiftCardValid(line) {
            let is_valid = await this.rpc({
                model: "gift.card",
                method: 'can_be_used_in_pos',
                args: [line.gift_card_id],
              });
            return is_valid;
        }
    };

    Registries.Component.extend(PaymentScreen, PosGiftCardPaymentScreen);

    return PosGiftCardPaymentScreen;
});

```

## File: static\src\js\tours\PosGiftCardstour.js

```javascript
odoo.define('pos_gift_card.tour.pos_gift_card1', function (require) {
    'use strict';

    // A tour that add a product, add a coupon, add a global discount, and check the lines content.

    const { PosGiftCards } = require('pos_giftcard.tour.PosGiftCardsTourMethods');
    const { ProductScreen } = require('point_of_sale.tour.ProductScreenTourMethods');
    const { getSteps, startSteps } = require('point_of_sale.tour.utils');
    var Tour = require('web_tour.tour');

    startSteps();

    ProductScreen.do.clickHomeCategory();

    ProductScreen.exec.addOrderline('product1', '1.00', '10');
    PosGiftCards.do.useGiftCard('1234');
    ProductScreen.check.totalAmountIs('0.00');

    Tour.register('PosGiftCardTour', { test: true, url: '/pos/web' }, getSteps());
});

```

## File: static\src\js\tours\PosGiftCardsTourMethods.js

```javascript
odoo.define('pos_giftcard.tour.PosGiftCardsTourMethods', function (require) {
    'use strict';

    const { createTourMethods } = require('point_of_sale.tour.utils');

    class Do {
        useGiftCard(giftCardCode) {
            return [
                {
                    content: 'open gift card popup',
                    trigger: `.control-button:contains("Gift Card")`,
                },
                {
                    content: 'click the "Use a gift card" button',
                    trigger: `.giftCardPopupConfirmButton:contains("Use a gift card")`,
                },
                {
                    content: 'click the "Use a gift card" button',
                    trigger: `.giftCardPopupInput`,
                    run: `text ${giftCardCode}`,
                },
                {
                    content: 'click the "Use a gift card" button',
                    trigger: '.confirm'
                },
            ];
        }
    }
    return createTourMethods('PosGiftCards', Do);
});

```

## File: static\src\xml\GiftCardButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="GiftCardButton" owl="1">
        <span class="control-button">
            <i class="fa fa-gift"></i>
            <span> </span>
            <span>Gift Card</span>
        </span>
    </t>

</templates>

```

## File: static\src\xml\GiftCardPopup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="GiftCardPopup" owl="1">
        <div role="dialog" class="modal-dialog">
            <Draggable>
                <div class="popup popup-textarea">
                    <header class="title drag-handle">
                        Gift Card
                    </header>

                    <main class="giftCardPopupMain">
                        <div class="giftCardPopupContainer" t-if="!state.showBarcodeGeneration">
                            <span t-if="state.giftCardConfig == 'create_set'"
                                  class="giftCardPopupButton button"
                                  t-on-click="switchBarcodeView">Generate barcode</span>
                            <span t-if="state.giftCardConfig == 'scan_set'"
                                  class="giftCardPopupButton button"
                                  t-on-click="switchBarcodeView">Scan and set price on gift card</span>
                            <span t-if="state.giftCardConfig == 'scan_use'"
                                  class="giftCardPopupButton button"
                                  t-on-click="switchBarcodeView">Scan gift card</span>
                        </div>

                        <div t-if="!state.showUseGiftCardMenu &amp;&amp; !state.showGiftCardDetails">
                            <div class="giftCardPopupContainer"
                                 t-if="state.showBarcodeGeneration &amp;&amp; state.giftCardConfig == 'create_set'">
                                <div>
                                    <span>Amount of the gift card:</span><br/>
                                    <div>
                                        <input t-model.number="state.amountToSet" class="giftCardPopupInput" type="text"/>
                                        <span class="currency">
                                            <t t-esc="env.pos.getCurrencySymbol()" />
                                        </span>
                                    </div>
                                </div>
                                <div>
                                    <div class="button confirm" t-on-click="generateBarcode">
                                        Confirm
                                    </div>
                                        <div class="button confirm" t-on-click="switchBarcodeView">
                                        Cancel
                                    </div>
                                </div>
                            </div>

                            <div class="giftCardPopupContainer"
                                 t-if="state.showBarcodeGeneration &amp;&amp; state.giftCardConfig == 'scan_set'">
                                <div>
                                    Gift Card Barcode: <br/>
                                    <input t-model="state.giftCardBarcode" class="giftCardPopupInput" type="text"/>
                                </div>
                                <div>
                                    Amount of the gift card: <br/>
                                    <input t-model.number="state.amountToSet" class="giftCardPopupInput" type="text"/>
                                </div>
                                <div>
                                    <span class="button confirm" t-on-click="scanAndUseGiftCard">
                                        Confirm
                                    </span>
                                    <span class="button cancel" t-on-click="switchBarcodeView">
                                        Discard
                                    </span>
                                </div>
                            </div>

                            <div class="giftCardPopupContainer"
                                 t-if="state.showBarcodeGeneration &amp;&amp; state.giftCardConfig == 'scan_use'">
                                <div>
                                    Gift Card Barcode:
                                    <input t-model="state.giftCardBarcode" class="giftCardPopupInput" type="text"/>
                                </div>
                                <div>
                                    <span class="button confirm" t-on-click="scanAndUseGiftCard">
                                        Confirm
                                    </span>
                                    <span class="button cancel" t-on-click="switchBarcodeView">
                                        Discard
                                    </span>
                                </div>
                            </div>
                        </div>

                        <div class="giftCardPopupContainer"
                             t-if="state.showBarcodeGeneration &amp;&amp; state.showUseGiftCardMenu">
                            <div>
                                Gift Card Barcode:
                                <input t-model="state.giftCardBarcode" class="giftCardPopupInput" type="text"/>
                            </div>
                            <div>
                                <span class="button confirm" t-on-click="payWithGiftCard">
                                    Confirm
                                </span>
                                <span class="button cancel" t-on-click="switchBarcodeView">
                                    Discard
                                </span>
                            </div>
                        </div>

                        <div class="giftCardPopupContainer"
                             t-if="state.showBarcodeGeneration &amp;&amp; state.showGiftCardDetails">
                            <div>
                                Gift Card Barcode:
                                <input t-model="state.giftCardBarcode" class="giftCardPopupInput" type="text"/>
                            </div>
                            <div>
                                Remaining amount of the gift card:
                                <t t-esc="state.amountToSet"/>
                            </div>
                            <div>
                                <span class="button confirm" t-on-click="ShowRemainingAmount">
                                    Confirm
                                </span>
                                <span class="button cancel" t-on-click="switchBarcodeView">
                                    Discard
                                </span>
                            </div>
                        </div>

                        <div class="giftCardPopupContainer" t-if="!state.showBarcodeGeneration">
                            <div class="button giftCardPopupConfirmButton" t-on-click="switchToUseGiftCardMenu">
                                Use a gift card
                            </div>
                                <div class="button giftCardPopupConfirmButton" t-on-click="switchToShowGiftCardDetails">
                                Check a gift card
                            </div>
                        </div>
                    </main>

                    <footer class="footer giftCardPopupFooter">
                        <div class="button cancel" t-on-click="cancel">
                            Cancel
                        </div>
                    </footer>
                </div>
            </Draggable>
        </div>
    </t>

</templates>

```

## File: views\gift_card_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        id="pos_gift_card_menu"
        action="gift_card.gift_card_action"
        parent="point_of_sale.pos_config_menu_catalog"
        name="Gift Cards"
        groups="point_of_sale.group_pos_manager"
        sequence="100"
    />

    <record id="pos_gift_card_view_form" model="ir.ui.view">
        <field name="name">gift.card.form Website</field>
        <field name="model">gift.card</field>
        <field name="inherit_id" ref="gift_card.gift_card_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='code']" position="after">
                <field name="buy_pos_order_line_id" options="{'no_create': True}"/>
            </xpath>
            <xpath expr="//group[@name='gift_card']" position="after">
                <group>
                    <field name="redeem_pos_order_line_ids" options="{'no_create': True}" readonly="1">
                        <tree>
                            <field name="id"/>
                        </tree>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>



```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_gift_card_config_view_form" model="ir.ui.view">
        <field name="name">pos.config.form</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='pos-loyalty']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="pos-coupon">
                    <div class="o_setting_left_pane">
                        <field name="use_gift_card"/>
                    </div>
                    <div class="o_setting_right_pane" title="Gift Card">
                        <label for="use_gift_card"/>
                        <div class="content-group" attrs="{'invisible': [('use_gift_card', '=', False)]}">
                            <div class="mt16" id="gift_card_product">
                                <label string="Gift Card Product" for="gift_card_product_id" class="o_light_label"/>
                                <field name="gift_card_product_id"/><br/>
                                <button name="%(gift_card.gift_card_action)d" icon="fa-arrow-right" type="action" string="Gift Card" class="btn-link"/><br/>
                                <label for="gift_card_settings" string="Gift card settings"/>
                                <field name="gift_card_settings" widget="radio"/>
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
            <xpath expr="//div[@id='pos-gift-card']/div[last()]" position="after">
                <div class="mt8" attrs="{'invisible': [('module_pos_gift_card', '=', False)]}">
                    <button name="%(gift_card.gift_card_action)d" icon="fa-arrow-right" type="action" string="Gift Card" class="btn-link"/><br/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

