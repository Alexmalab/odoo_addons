# Odoo Module: pos_event

Category: Technical

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "POS - Event",
    'category': "Technical",
    'summary': 'Link module between Point of Sale and Event',
    'depends': ['point_of_sale', 'event_product'],
    'data': [
        'security/ir.model.access.csv',
        'data/point_of_sale_data.xml',
        'data/event_product_data.xml',
        'views/event_event_views.xml',
        'views/pos_order_views.xml',
    ],
    'demo': [
        'data/event_product_demo.xml',
        'data/point_of_sale_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_event/static/src/**/*',
        ],
          'web.assets_tests': [
            'pos_event/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\event_product_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="event_product.product_product_event" model="product.product" forcecreate="False">
            <field name="available_in_pos">True</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('pos_event.pos_category_event')])]"/>
        </record>
    </data>
</odoo>

```

## File: data\event_product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="event_product.product_product_event_standard" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('pos_event.pos_category_event')])]"/>
        </record>
        <record id="event_product.product_product_event_vip" model="product.product">
            <field name="available_in_pos">True</field>
            <field name="pos_categ_ids" eval="[(6, 0, [ref('pos_event.pos_category_event')])]"/>
        </record>
    </data>
</odoo>

```

## File: data\point_of_sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="pos_category_event" model="pos.category">
            <field name="name">Events</field>
            <field name="image_128" type="base64" file="pos_event/static/img/event_category.png" />
        </record>
    </data>
</odoo>

```

## File: data\point_of_sale_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
         <record model="pos.config" id="point_of_sale.pos_config_main">
            <field name="iface_available_categ_ids" eval="[(4, ref('pos_event.pos_category_event'))]" />
        </record>
    </data>
</odoo>

```

## File: models\event_event.py

```python
from odoo import api, fields, models


class Event(models.Model):
    _name = 'event.event'
    _inherit = ['event.event', 'pos.load.mixin']

    image_1024 = fields.Image("PoS Image", max_width=1024, max_height=1024)

    @api.model
    def _load_pos_data_domain(self, data):
        return [('event_ticket_ids', 'in', [ticket['id'] for ticket in data['event.event.ticket']['data']])]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['id', 'name', 'seats_available', 'event_ticket_ids', 'registration_ids', 'seats_limited', 'write_date',
                'question_ids', 'general_question_ids', 'specific_question_ids', 'badge_format']

```

## File: models\event_question.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class EventQuestion(models.Model):
    _name = 'event.question'
    _inherit = ['event.question', 'pos.load.mixin']

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['title', 'question_type', 'event_type_id', 'event_id', 'sequence', 'once_per_order', 'is_mandatory_answer', 'answer_ids']

    @api.model
    def _load_pos_data_domain(self, data):
        return [('event_id', 'in', [event['id'] for event in data['event.event']['data']])]

```

## File: models\event_question_answer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class EventQuestionAnswer(models.Model):
    _name = 'event.question.answer'
    _inherit = ['event.question.answer', 'pos.load.mixin']

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['question_id', 'name', 'sequence']

    @api.model
    def _load_pos_data_domain(self, data):
        return [('question_id', 'in', [quest['id'] for quest in data['event.question']['data']])]

```

## File: models\event_registration.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class EventRegistration(models.Model):
    _name = 'event.registration'
    _inherit = ['event.registration', 'pos.load.mixin']

    pos_order_id = fields.Many2one(related='pos_order_line_id.order_id', string='PoS Order')
    pos_order_line_id = fields.Many2one('pos.order.line', string='PoS Order Line', ondelete='cascade', copy=False)

    @api.model
    def _load_pos_data_domain(self, data):
        return False

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['id', 'event_id', 'event_ticket_id', 'pos_order_line_id', 'pos_order_id', 'phone', 'email', 'name', 'registration_answer_ids', 'registration_answer_choice_ids']

    @api.model_create_multi
    def create(self, vals_list):
        result = super().create(vals_list)
        result._update_available_seat()
        return result

    def write(self, vals):
        result = super().write(vals)
        self._update_available_seat()
        return result

    def _update_available_seat(self):
        # Here sudo is used in order for pos_event to update the available seats to all open pos session when a ticket is sold in website for example
        session_ids = self.env['pos.session'].sudo().search([("state", "!=", "closed")])
        if len(session_ids) > 0:
            session_ids.config_id._update_events_seats(self.event_id)

```

## File: models\event_registration_answer.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class EventRegistrationAnswer(models.Model):
    _name = 'event.registration.answer'
    _inherit = ['event.registration.answer', 'pos.load.mixin']

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['question_id', 'registration_id', 'value_answer_id', 'value_text_box', 'partner_id', 'event_id']

    @api.model
    def _load_pos_data_domain(self, data):
        return False

```

## File: models\event_ticket.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api, fields


class EventTicket(models.Model):
    _name = 'event.event.ticket'
    _inherit = ['event.event.ticket', 'pos.load.mixin']

    @api.model
    def _load_pos_data_domain(self, data):
        return [
            ('event_id.is_finished', '=', False),
            ('event_id.company_id', '=', data['pos.config']['data'][0]['company_id']),
            ('product_id', 'in', [product['id'] for product in data['product.product']['data']]),
            '|', ('end_sale_datetime', '>=', fields.Datetime.now()), ('end_sale_datetime', '=', False),
            '|', ('start_sale_datetime', '<=', fields.Datetime.now()), ('start_sale_datetime', '=', False)
        ]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['id', 'name', 'event_id', 'seats_used', 'seats_available', 'price', 'product_id', 'seats_max', 'start_sale_datetime', 'end_sale_datetime']

```

## File: models\pos_config.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    def _update_events_seats(self, events):
        data = []
        for event in events:
            data.append({
                'event_id': event.id,
                'seats_available': event.seats_available,
                'event_ticket_ids': [{
                    'ticket_id': ticket.id,
                    'seats_available': ticket.seats_available
                } for ticket in event.event_ticket_ids]
            })

        for record in self:
            record._notify('UPDATE_AVAILABLE_SEATS', data)

```

## File: models\pos_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class PosOrder(models.Model):
    _inherit = 'pos.order'

    attendee_count = fields.Integer('Attendee Count', compute='_compute_attendee_count')

    @api.depends('lines.event_registration_ids')
    def _compute_attendee_count(self):
        for order in self:
            order.attendee_count = len(order.lines.mapped('event_registration_ids'))

    def action_view_attendee_list(self):
        action = self.env["ir.actions.actions"]._for_xml_id("event.event_registration_action_tree")
        action['domain'] = [('pos_order_id', 'in', self.ids)]
        return action

    def read_pos_data(self, data, config_id):
        results = super().read_pos_data(data, config_id)
        paid_orders = self.filtered_domain([('state', 'in', ['paid', 'done', 'invoiced'])])

        if not paid_orders:
            return results

        lines_with_event = paid_orders.mapped('lines').filtered(lambda line: line.event_ticket_id)
        event_event_fields = self.env['event.event']._load_pos_data_fields(paid_orders[0].config_id.id)
        event_ticket_fields = self.env['event.event.ticket']._load_pos_data_fields(paid_orders[0].config_id.id)
        event_registrations_fields = self.env['event.registration']._load_pos_data_fields(paid_orders[0].config_id.id)
        event_registrations_answer_fields = self.env['event.registration.answer']._load_pos_data_fields(paid_orders[0].config_id.id)
        results['event.registration'] = lines_with_event.event_registration_ids.read(event_registrations_fields, load=False)
        results['event.event'] = lines_with_event.event_registration_ids.mapped('event_id').read(event_event_fields, load=False)
        results['event.event.ticket'] = lines_with_event.event_registration_ids.mapped('event_ticket_id').read(event_ticket_fields, load=False)
        results['event.registration.answer'] = lines_with_event.event_registration_ids.mapped('registration_answer_ids').read(event_registrations_answer_fields, load=False)

        for registration in lines_with_event.event_registration_ids:
            if registration.email:
                registration.action_send_badge_email()

        return results

    @api.model
    def _process_order(self, order, existing_order):
        res = super()._process_order(order, existing_order)
        refunded_line_ids = [line[2].get('refunded_orderline_id') for line in order.get('lines') if line[0] in [0, 1] and line[2].get('refunded_orderline_id')]
        refunded_orderlines = self.env['pos.order.line'].browse(refunded_line_ids)
        event_to_cancel = []

        for refunded_orderline in refunded_orderlines:
            if refunded_orderline.event_registration_ids:
                refund_qty = abs(sum(refunded_orderline.refund_orderline_ids.mapped('qty')))
                already_cancelled_qty = len(refunded_orderline.event_registration_ids.filtered(lambda r: r.state == 'cancel'))
                to_cancel_qty = refund_qty - already_cancelled_qty
                if to_cancel_qty > 0:
                    event_to_cancel += refunded_orderline.event_registration_ids.filtered(lambda registration: registration.state != 'cancel').ids[:int(to_cancel_qty)]

        if event_to_cancel:
            self.env['event.registration'].browse(event_to_cancel).write({'state': 'cancel'})

        return res

    def print_event_tickets(self):
        return self.env.ref('event.action_report_event_registration_full_page_ticket').report_action(self.lines.event_registration_ids)

    def print_event_badges(self):
        return self.env.ref('event.action_report_event_registration_badge').report_action(self.lines.event_registration_ids)

```

## File: models\pos_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class PosOrderLine(models.Model):
    _inherit = 'pos.order.line'

    event_ticket_id = fields.Many2one('event.event.ticket', string='Event Ticket')
    event_registration_ids = fields.One2many('event.registration', 'pos_order_line_id', string='Event Registrations')

    @api.model
    def _load_pos_data_fields(self, config_id):
        fields = super()._load_pos_data_fields(config_id)
        fields += ['event_ticket_id', 'event_registration_ids']
        return fields

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class PosSession(models.Model):
    _inherit = 'pos.session'

    @api.model
    def _load_pos_data_models(self, config_id):
        models = super()._load_pos_data_models(config_id)
        models += ['event.event.ticket', 'event.event', 'event.registration', 'event.question', 'event.question.answer', 'event.registration.answer']
        return models

    @api.model
    def _load_pos_data_relations(self, model, response):
        super()._load_pos_data_relations(model, response)
        if model == 'event.registration':
            # Force compute to False otherwise the frontend will not send the data
            response['event.registration']['relations']['email']['compute'] = False
            response['event.registration']['relations']['phone']['compute'] = False
            response['event.registration']['relations']['name']['compute'] = False

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import pos_config
from . import event_ticket
from . import pos_session
from . import event_event
from . import pos_order
from . import pos_order_line
from . import event_registration
from . import event_question
from . import event_question_answer
from . import event_registration_answer

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_registration,event.registration,model_event_registration,point_of_sale.group_pos_user,1,1,1,0
access_event_event_ticket,event.event.ticket,model_event_event_ticket,point_of_sale.group_pos_user,1,0,0,0
access_event_event,event.event,model_event_event,point_of_sale.group_pos_user,1,0,0,0

```

## File: static\src\app\generic_components\product_card\product_card.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { ProductCard } from "@point_of_sale/app/generic_components/product_card/product_card";
import { patch } from "@web/core/utils/patch";

patch(ProductCard.prototype, {
    get displayRemainingSeats() {
        return Boolean(this.props.product.event_id) && this.props.product.event_id.seats_limited;
    },
});

```

## File: static\src\app\generic_components\product_card\product_card.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.ProductCard" t-inherit="point_of_sale.ProductCard" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('product-information-tag')]" position="attributes">
            <attribute name="t-if" add="!this.props.product.event_id" separator=" and " />
        </xpath>
        <xpath expr="//div[hasclass('product-information-tag')]" position="after">
            <div t-if="displayRemainingSeats"
                class="shadow-sm m-1 py-1 px-2 rounded position-absolute top-0 end-0"
                t-attf-class="{{ this.props.product.event_id.seats_available === 0 ? 'bg-danger text-white' : 'bg-white'}}">
                <span t-esc="this.props.product.event_id.seats_available" /> left
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\models\data_service_options.js

```javascript
import { DataServiceOptions } from "@point_of_sale/app/models/data_service_options";
import { patch } from "@web/core/utils/patch";

patch(DataServiceOptions.prototype, {
    get databaseTable() {
        return {
            ...super.databaseTable,
            "event.registration": {
                key: "id",
                condition: (record) => {
                    return (
                        !record.pos_order_line_id || record.pos_order_line_id?.order_id?.finalized
                    );
                },
            },
            "event.registration.answer": {
                key: "id",
                condition: (record) => {
                    return (
                        !record.registration_id ||
                        record.registration_id?.pos_order_line_id?.order_id?.finalized
                    );
                },
            },
        };
    },
    get dynamicModels() {
        return [...super.dynamicModels, "event.registration", "event.registration.answer"];
    },
});

```

## File: static\src\app\models\pos_order.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    get eventRegistrations() {
        return this.lines.flatMap((line) => line.event_registration_ids);
    },
});

```

## File: static\src\app\models\pos_order_line.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { PosOrderline } from "@point_of_sale/app/models/pos_order_line";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(PosOrderline.prototype, {
    can_be_merged_with(orderline) {
        return (
            this.event_ticket_id?.id === orderline.event_ticket_id?.id &&
            super.can_be_merged_with(...arguments)
        );
    },
    set_quantity(quantity, keep_price) {
        if (this.event_ticket_id && quantity !== "") {
            return {
                title: _t("Ticket error"),
                body: _t("You cannot change quantity for a line linked with an event registration"),
            };
        } else if (this.event_ticket_id) {
            for (const registration of this.event_registration_ids) {
                registration.delete({ silent: true });
            }
        }

        return super.set_quantity(quantity, keep_price);
    },
});

```

## File: static\src\app\models\product_product.js

```javascript
import { ProductProduct } from "@point_of_sale/app/models/product_product";
import { patch } from "@web/core/utils/patch";

patch(ProductProduct.prototype, {
    get event_id() {
        if (!this._event_id) {
            return false;
        }

        return this.models["event.event"].get(this._event_id);
    },
    get canBeDisplayed() {
        if (this.event_id) {
            return true;
        }
        return super.canBeDisplayed;
    },
});

```

## File: static\src\app\popup\event_configurator_popup\event_configurator_popup.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { Dialog } from "@web/core/dialog/dialog";
import { Component, useState } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { ProductCard } from "@point_of_sale/app/generic_components/product_card/product_card";
import { NumericInput } from "@point_of_sale/app/generic_components/inputs/numeric_input/numeric_input";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { deserializeDateTime } from "@web/core/l10n/dates";

const { DateTime } = luxon;

export class EventConfiguratorPopup extends Component {
    static template = "pos_event.EventConfiguratorPopup";
    static props = ["tickets", "getPayload", "close"];
    static components = {
        Dialog,
        ProductCard,
        NumericInput,
    };
    setup() {
        this.pos = usePos();
        this.dialog = useService("dialog");
        this.state = useState({});

        for (const ticket of this.props.tickets) {
            this.state[ticket.id] = {
                qty: 0,
            };
        }
    }
    get dialogTitle() {
        const event = this.props.tickets[0].event_id;
        let title = _t("Select tickets for %s", [event.name]);

        if (event.seats_limited) {
            title += _t(" (%s seats available)", [event.seats_available]);
        }

        return title;
    }
    getTicketMaxQty(ticket) {
        const event = ticket.event_id;
        const maxTicket = ticket.seats_available - this.getOrderAlreadyBooked(ticket);

        if ((event.seats_limited && event.seats_available < maxTicket) || ticket.seats_max === 0) {
            return event.seats_available;
        }

        return maxTicket;
    }
    getProductProxy(productId) {
        return this.pos.models["product.product"].get(productId);
    }
    confirm() {
        const data = [];
        for (const [ticketId, { qty }] of Object.entries(this.state)) {
            if (qty > 0) {
                const ticket = this.pos.models["event.event.ticket"].get(parseInt(ticketId));
                const available = this.ticketIsAvailable(ticket);

                if (!available) {
                    this.dialog.add(AlertDialog, {
                        title: _t("Error"),
                        body: _t(
                            "The selected ticket (%s) is not available. Please select a different ticket.",
                            [ticket.name]
                        ),
                    });
                    this.props.close();
                    return;
                }

                data.push({
                    product_id: ticket.product_id,
                    ticket_id: ticket,
                    qty,
                });
            }
        }

        this.props.getPayload(data);
        this.props.close();
    }
    cancel() {
        this.props.close();
    }
    getOrderAlreadyBooked(ticket) {
        return this.pos
            .get_order()
            .lines.filter((l) => l.event_ticket_id?.id === ticket.id)
            .reduce((acc, l) => (acc += l.qty), 0);
    }
    ticketIsAvailable(ticket) {
        const dateTimeNow = DateTime.now();
        const bookedTicket = this.getOrderAlreadyBooked(ticket) + this.state[ticket.id].qty;
        const eventAvailable =
            !ticket.event_id?.seats_limited || bookedTicket <= ticket.event_id.seats_available;

        const eventSaleEnd =
            !ticket.end_sale_datetime ||
            deserializeDateTime(ticket.end_sale_datetime).ts > dateTimeNow.ts;
        const eventSaleStart =
            !ticket.start_sale_datetime ||
            deserializeDateTime(ticket.start_sale_datetime).ts < dateTimeNow.ts;

        if (!eventSaleStart || !eventSaleEnd) {
            return false;
        }

        if (ticket.seats_max === 0 && eventAvailable) {
            return true;
        }

        return ticket.seats_available >= bookedTicket && eventAvailable;
    }
}

```

## File: static\src\app\popup\event_configurator_popup\event_configurator_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.EventConfiguratorPopup">
        <Dialog title="dialogTitle">
            <div class="o_event_configurator_popup">
                <t t-foreach="this.props.tickets" t-as="ticket" t-key="ticket.id">
                    <div class="d-flex justify-content-between">
                        <div class="d-flex flex-column">
                            <div class="fs-5">
                                <span t-esc="ticket.name"/>
                             </div>
                            <span t-if="ticket.seats_max === 0 and !ticket.event_id.seats_limited" class="fs-6 text-success">
                                Unlimited
                            </span>
                            <span t-else="" t-attf-class="{{ this.ticketIsAvailable(ticket) ? 'text-success' : 'text-danger'}} fs-6">
                                <t t-esc="this.getTicketMaxQty(ticket)" /> left
                            </span>
                        </div>
                        <div class="d-flex align-items-center gap-3">
                            <span t-esc="env.utils.formatCurrency(ticket.price)" />
                            <NumericInput class="'w-100'" tModel="[this.state[ticket.id], 'qty']" min="0"/>
                        </div>
                    </div>
                    <hr t-if="ticket_index !== this.props.tickets.length - 1" class="hr" />
                </t>
            </div>
            <t t-set-slot="footer">
                <button class="btn btn-secondary o-default-button" t-on-click="cancel">Cancel</button>
                <button class="btn btn-primary o-default-button" t-on-click="confirm">Confirm</button>
            </t>
        </Dialog>
    </t>
</templates>

```

## File: static\src\app\popup\event_registration_popup\event_registration_popup.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { Dialog } from "@web/core/dialog/dialog";
import { Component, useState } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { ProductCard } from "@point_of_sale/app/generic_components/product_card/product_card";
import { NumericInput } from "@point_of_sale/app/generic_components/inputs/numeric_input/numeric_input";
import { useService } from "@web/core/utils/hooks";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

export class EventRegistrationPopup extends Component {
    static template = "pos_event.EventRegistrationPopup";
    static props = ["data", "getPayload", "close", "event"];
    static components = {
        Dialog,
        ProductCard,
        NumericInput,
    };
    setup() {
        this.pos = usePos();
        this.dialog = useService("dialog");
        this.state = useState({
            byRegistration: [],
            byOrder: {},
        });
        this.dataInQty = this.props.data.reduce((acc, data) => {
            for (let i = 0; i < data.qty; i++) {
                acc.push(data);
            }
            return acc;
        }, []);

        for (const [idx, data] of Object.entries(this.dataInQty)) {
            this.state.byRegistration[idx] = {
                ticket_id: data.ticket_id,
                product_id: data.product_id,
                questions: {},
            };

            for (const question of this.questionsByRegistration) {
                this.state.byRegistration[idx].questions[question.id] = "";
            }
        }

        for (const question of this.questionsOncePerOrder) {
            this.state.byOrder[question.id] = "";
        }

        if (this.props.event.question_ids.length === 0) {
            this.confirm();
        }
    }

    get questionsByRegistration() {
        return this.props.event.question_ids.filter((question) => !question.once_per_order);
    }

    get questionsOncePerOrder() {
        return this.props.event.question_ids.filter((question) => question.once_per_order);
    }

    confirm() {
        const required = Object.values(this.state.byRegistration).some((data) => {
            for (const [id, value] of Object.entries(data.questions)) {
                const question = this.pos.models["event.question"].get(id);

                if (question && question.is_mandatory_answer && !value) {
                    return true;
                }
            }
        });

        if (required) {
            this.dialog.add(AlertDialog, {
                title: "Error",
                body: "Please fill in all required fields",
            });
            return;
        }

        const registrationByTickets = this.state.byRegistration.reduce((acc, data) => {
            if (!acc[data.ticket_id.id]) {
                acc[data.ticket_id.id] = [];
            }

            acc[data.ticket_id.id].push(data.questions);
            return acc;
        }, {});

        this.props.getPayload({
            byRegistration: registrationByTickets,
            byOrder: this.state.byOrder,
        });
        this.props.close();
    }
    close() {
        this.props.close();
    }
}

```

## File: static\src\app\popup\event_registration_popup\event_registration_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.QuestionInputs">
        <div t-foreach="questions" t-as="question" t-key="question.id" class="global_question input-group mt-3">
            <span class="input-group-text">
                <t t-esc="question.title"/>
                <span t-if="question.is_mandatory_answer" class="text-danger">*</span>
            </span>
            <select t-if="question.question_type === 'simple_choice'" t-model="stateObject[question.id]" class="form-select">
                <option value="">--- Select ---</option>
                <option t-foreach="question.answer_ids" t-as="choice" t-key="choice.id" t-att-value="choice.id">
                    <t t-esc="choice.name"/>
                </option>
            </select>
            <input t-else="" t-model="stateObject[question.id]" class="form-control flex-grow-1"/>
        </div>
    </t>
    <t t-name="pos_event.EventRegistrationPopup">
        <Dialog title.translate="Tickets">
            <div class="o_event_registration_popup">
                <div t-if="this.questionsOncePerOrder.length" class="mb-4">
                    <div>
                        <span class="fs-3">Global questions</span>
                        <hr class="hr mt-1" />
                    </div>
                    <t t-set="questions" t-value="this.questionsOncePerOrder"/>
                    <t t-set="stateObject" t-value="this.state.byOrder"/>
                    <t t-call="pos_event.QuestionInputs" />
                </div>
                <t t-if="this.questionsByRegistration.length">
                    <div t-foreach="this.dataInQty" t-as="data" t-key="data_index" class="ticket_question mb-4">
                        <div>
                            <span class="fs-3">
                                Ticket #<t t-esc="data_index + 1" /> for <t t-esc="this.state.byRegistration[data_index].ticket_id.name" />
                            </span>
                            <hr class="hr mt-1" />
                        </div>
                        <t t-set="questions" t-value="this.questionsByRegistration"/>
                        <t t-set="stateObject" t-value="this.state.byRegistration[data_index].questions"/>
                        <t t-call="pos_event.QuestionInputs" />
                    </div>
                </t>
            </div>
            <t t-set-slot="footer">
                <button class="btn btn-secondary o-default-button" t-on-click="close">Cancel</button>
                <button class="btn btn-primary o-default-button" t-on-click="confirm">Confirm</button>
            </t>
        </Dialog>
    </t>
</templates>

```

## File: static\src\app\screens\product_screen\product_screen.js

```javascript
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { makeAwaitable } from "@point_of_sale/app/store/make_awaitable_dialog";
import { patch } from "@web/core/utils/patch";
import { EventConfiguratorPopup } from "@pos_event/app/popup/event_configurator_popup/event_configurator_popup";
import { _t } from "@web/core/l10n/translation";
import { EventRegistrationPopup } from "../../popup/event_registration_popup/event_registration_popup";

patch(ProductScreen.prototype, {
    get products() {
        const products = super.products;
        return [...products].filter((p) => p.service_tracking !== "event");
    },
    getProductPrice(product) {
        if (!product.event_id) {
            return super.getProductPrice(product);
        }

        return _t("From %s", this.env.utils.formatCurrency(this.pos.getProductPrice(product)));
    },
    getProductImage(product) {
        if (!product.event_id) {
            return super.getProductImage(product);
        }

        return `/web/image?model=event.event&id=${product.event_id.id}&field=image_1024&unique=${product.event_id.write_date}`;
    },
    async addProductToOrder(product) {
        if (!product.event_id) {
            return await super.addProductToOrder(product);
        }

        if (product.event_id.seats_available === 0 && product.event_id.seats_limited) {
            this.notification.add("No more seats available for this event", {
                type: "danger",
            });
            return;
        }

        const event = product.event_id;
        const tickets = event.event_ticket_ids.filter(
            (ticket) => ticket.product_id && ticket.product_id.service_tracking === "event"
        );

        const ticketResult = await makeAwaitable(this.dialog, EventConfiguratorPopup, {
            tickets: tickets,
        });

        if (!ticketResult || !ticketResult.length) {
            return;
        }

        const result = await makeAwaitable(this.dialog, EventRegistrationPopup, {
            event: event,
            data: ticketResult,
        });

        if (!result || !result.byRegistration || !Object.keys(result.byRegistration).length) {
            return;
        }

        const { globalSimpleChoice, globalTextAnswer } = Object.entries(result.byOrder).reduce(
            (acc, [questionId, answer]) => {
                const question = this.pos.models["event.question"].get(parseInt(questionId));
                if (
                    question.question_type === "simple_choice" &&
                    this.pos.models["event.question.answer"].get(parseInt(answer))
                ) {
                    acc.globalSimpleChoice[questionId] = answer;
                } else if (answer) {
                    acc.globalTextAnswer[questionId] = answer;
                }

                return acc;
            },
            { globalSimpleChoice: {}, globalTextAnswer: {} }
        );

        for (const [ticketId, data] of Object.entries(result.byRegistration)) {
            const ticket = this.pos.models["event.event.ticket"].get(parseInt(ticketId));
            const line = await this.pos.addLineToCurrentOrder({
                product_id: ticket.product_id,
                price_unit: ticket.price,
                qty: data.length,
                event_ticket_id: ticket,
            });

            for (const registration of data) {
                const userData = {};
                for (const [questionId, answer] of Object.entries(registration)) {
                    const question = this.pos.models["event.question"].get(parseInt(questionId));

                    if (!question) {
                        continue;
                    }

                    if (question.question_type === "email") {
                        userData.email = answer;
                    } else if (question.question_type === "phone") {
                        userData.phone = answer;
                    } else if (question.question_type === "name") {
                        userData.name = answer;
                    } else if (question.question_type === "company") {
                        userData.company = answer;
                    }
                }

                const { simpleChoice, textAnswer } = Object.entries(registration).reduce(
                    (acc, [questionId, answer]) => {
                        const question = this.pos.models["event.question"].get(
                            parseInt(questionId)
                        );
                        if (
                            question.question_type === "simple_choice" &&
                            this.pos.models["event.question.answer"].get(parseInt(answer))
                        ) {
                            acc.simpleChoice[questionId] = answer;
                        } else if (answer) {
                            acc.textAnswer[questionId] = answer;
                        }

                        return acc;
                    },
                    { simpleChoice: {}, textAnswer: {} }
                );

                this.pos.models["event.registration"].create({
                    ...userData,
                    event_id: event,
                    event_ticket_id: ticket,
                    pos_order_line_id: line,
                    partner_id: this.pos.get_order().partner_id,
                    registration_answer_ids: Object.entries({
                        ...textAnswer,
                        ...globalTextAnswer,
                    }).map(([questionId, answer]) => [
                        "create",
                        {
                            question_id: this.pos.models["event.question"].get(
                                parseInt(questionId)
                            ),
                            value_text_box: answer,
                        },
                    ]),
                    registration_answer_choice_ids: Object.entries({
                        ...simpleChoice,
                        ...globalSimpleChoice,
                    }).map(([questionId, answer]) => [
                        "create",
                        {
                            question_id: this.pos.models["event.question"].get(
                                parseInt(questionId)
                            ),
                            value_answer_id: this.pos.models["event.question.answer"].get(
                                parseInt(answer)
                            ),
                        },
                    ]),
                });
            }
        }
    },
});

```

## File: static\src\app\screens\product_screen\order_summary\order_summary.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.OrderSummary" t-inherit="point_of_sale.OrderSummary" t-inherit-mode="extension">
		<xpath expr="//Orderline" position="inside" >
            <t t-if="line.event_ticket_id">
                <li class="info ms-2">
                    <i class="fa fa-ticket me-1" role="img" aria-label="Event name" title="Event name"/>
                    <t t-esc="line.event_ticket_id.event_id.name" />
                </li>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\screens\receipt_screen\receipt_screen.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { ReceiptScreen } from "@point_of_sale/app/screens/receipt_screen/receipt_screen";
import { patch } from "@web/core/utils/patch";
import { useTrackedAsync } from "@point_of_sale/app/utils/hooks";
import { useService } from "@web/core/utils/hooks";

patch(ReceiptScreen.prototype, {
    setup() {
        super.setup(...arguments);

        this.report = useService("report");
        this.orm = useService("orm");
        this.doPrintEventFull = useTrackedAsync(() => this.printEventFull());
        this.doPrintEventBadge = useTrackedAsync(() => this.printEventBadge());
    },
    async printEventFull() {
        const registrations = this.pos.get_order().eventRegistrations.map((reg) => reg.id);
        await this.report.doAction("event.action_report_event_registration_full_page_ticket", [
            registrations,
        ]);
    },
    async printEventBadge() {
        const registrations = this.pos.get_order().eventRegistrations;

        const smallBadgeRegistrations = registrations.filter(
            (reg) => reg.event_id.badge_format === "96x82"
        );
        const largeBadgeRegistrations = registrations.filter(
            (reg) => reg.event_id.badge_format === "96x134"
        );
        const nonBadgePrinterRegistrations = registrations.filter(
            (reg) => !["96x82", "96x134"].includes(reg.event_id.badge_format)
        );

        if (nonBadgePrinterRegistrations.length > 0) {
            await this.report.doAction(
                "event.action_report_event_registration_badge",
                nonBadgePrinterRegistrations.map((reg) => reg.id)
            );
        }
        if (largeBadgeRegistrations.length > 0) {
            await this.report.doAction(
                "event.action_report_event_registration_badge_96x134",
                largeBadgeRegistrations.map((reg) => reg.id)
            );
        }
        if (smallBadgeRegistrations.length > 0) {
            await this.report.doAction(
                "event.action_report_event_registration_badge_96x82",
                smallBadgeRegistrations.map((reg) => reg.id)
            );
        }

        // Update the status to "attended" if we print the attendee badge
        if (registrations.length > 0) {
            const registrationIds = registrations.map((registration) => registration.id);
            await this.orm.write("event.registration", registrationIds, { state: "done" });
        }
    },
});

```

## File: static\src\app\screens\receipt_screen\receipt_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.ReceiptScreen" t-inherit="point_of_sale.ReceiptScreen" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_payment_successful')]" position="inside">
            <t t-if="this.pos.get_order().eventRegistrations.length > 0">
                <h2 class="mt-3">Event Registrations</h2>
                <div class="o-event-button buttons mb-3 d-flex gap-1">
                    <button class="o-event-full button print btn btn-md btn-secondary w-50 py-3" t-on-click="() => doPrintEventFull.call()">
                        <i t-attf-class="fa {{doPrintEventFull.status === 'loading' ? 'fa-fw fa-spin fa-circle-o-notch' : 'fa-print'}} me-1" />Print Full Page Ticket
                    </button>
                    <button class="o-event-badge button print btn btn-md btn-secondary w-50 py-3" t-on-click="() => doPrintEventBadge.call()">
                        <i t-attf-class="fa {{doPrintEventBadge.status === 'loading' ? 'fa-fw fa-spin fa-circle-o-notch' : 'fa-print'}} me-1" />Print Badge
                    </button>
                </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\app\store\pos_store.js

```javascript
// Part of Odoo. See LICENSE file for full copyright and licensing details.
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    async setup() {
        await super.setup(...arguments);
        this.data.connectWebSocket("UPDATE_AVAILABLE_SEATS", (data) => {
            for (const ev of data) {
                const event = this.models["event.event"].get(ev.event_id);
                if (event) {
                    event.seats_available = ev.seats_available;
                } else {
                    continue;
                }

                for (const ticket of ev.event_ticket_ids) {
                    const eventTicket = this.models["event.event.ticket"].get(ticket.ticket_id);
                    if (eventTicket) {
                        eventTicket.seats_available = ticket.seats_available;
                    }
                }
            }
        });

        this.createDummyProductForEvents();
    },

    createDummyProductForEvents() {
        for (const event of this.models["event.event"].getAll()) {
            const eventTicketWithProduct = event.event_ticket_ids.filter(
                (ticket) => ticket.product_id
            );

            if (!eventTicketWithProduct.length) {
                continue;
            }

            const lowestPrice = eventTicketWithProduct.sort((a, b) => a.price - b.price)[0];
            const categIds = eventTicketWithProduct.flatMap(
                (ticket) => ticket.product_id.pos_categ_ids
            );
            const taxeIds = eventTicketWithProduct.flatMap((ticket) => ticket.product_id.taxes_id);
            this.models["product.product"].create({
                id: `dummy_${event.id}`,
                available_in_pos: true,
                lst_price: lowestPrice.price,
                display_name: event.name,
                pos_categ_ids: categIds.map((categ) => ["link", categ]),
                taxes_id: taxeIds.map((tax) => ["link", tax]),
                _event_id: event.id,
            });
        }
    },
});

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_event_view_form_inherit_pos_event" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.pos.event</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]" position="before">
                <field name="image_1024" widget="image" class="oe_avatar me-4"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_order_form_view_inherit" model="ir.ui.view">
        <field name="name">pos.order.form.view.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="print_event_tickets" invisible="attendee_count == 0" type="object"><i class="fa fa-file-pdf-o me-1" aria-hidden="true" />Print Event Tickets</button>
                <button name="print_event_badges" invisible="attendee_count == 0" type="object"><i class="fa fa-file-pdf-o me-1" aria-hidden="true" />Print Event Badges</button>
            </xpath>
            <xpath expr="//sheet/div[hasclass('oe_button_box')]" position="inside">
                <button name="action_view_attendee_list" type="object"
                    class="oe_stat_button" icon="fa-users" invisible="attendee_count == 0">
                    <field name="attendee_count" widget="statinfo" string="Attendees"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

