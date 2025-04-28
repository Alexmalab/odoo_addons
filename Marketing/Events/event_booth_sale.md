# Odoo Module: event_booth_sale

Category: Marketing/Events

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
    'name': "Events Booths Sales",
    'category': 'Marketing/Events',
    'version': '1.1',
    'summary': "Manage event booths sale",
    'description': """
Sell your event booths and track payments on sale orders.
    """,
    'depends': ['event_booth', 'event_sale'],
    'data': [
        'security/ir.model.access.csv',
        'data/product_data.xml',
        'data/event_booth_category_data.xml',
        'views/sale_order_views.xml',
        'views/event_type_booth_views.xml',
        'views/event_booth_category_views.xml',
        'views/event_booth_registration_views.xml',
        'views/event_booth_views.xml',
        'wizard/event_booth_configurator_views.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'event_booth_sale/static/src/**/*',
        ]
    },
    'license': 'LGPL-3',
}

```

## File: data\event_booth_category_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth.event_booth_category_standard" model="event.booth.category">
        <field name="product_id" ref="event_booth_sale.product_product_event_booth"/>
        <field name="price">100</field>
    </record>

    <record id="event_booth.event_booth_category_premium" model="event.booth.category">
        <field name="product_id" ref="event_booth_sale.product_product_event_booth"/>
        <field name="price">500</field>
    </record>

    <record id="event_booth.event_booth_category_vip" model="event.booth.category">
        <field name="product_id" ref="event_booth_sale.product_product_event_booth"/>
        <field name="price">1000</field>
    </record>

</data></odoo>

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <record id="product_product_event_booth" model="product.product">
        <field name="name">Event Booth</field>
        <field name="purchase_ok" eval="False"/>
        <field name="list_price">100.0</field>
        <field name="description_sale" eval="False"/>
        <field name="categ_id" ref="event_sale.product_category_events"/>
        <field name="invoice_policy">order</field>
        <field name="detailed_type">event_booth</field>
    </record>

</data></odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _invoice_paid_hook(self):
        """ When an invoice linked to a sales order selling registrations is
        paid, update booths accordingly as they are booked when invoice is paid.
        """
        res = super(AccountMove, self)._invoice_paid_hook()
        self.mapped('line_ids.sale_line_ids')._update_event_booths(set_paid=True)
        return res

```

## File: models\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class EventBooth(models.Model):
    _inherit = 'event.booth'

    # registrations
    event_booth_registration_ids = fields.One2many('event.booth.registration', 'event_booth_id')
    # sale information
    sale_order_line_registration_ids = fields.Many2many(
        'sale.order.line', 'event_booth_registration',
        'event_booth_id', 'sale_order_line_id', string='SO Lines with reservations',
        groups='sales_team.group_sale_salesman', copy=False)
    sale_order_line_id = fields.Many2one(
        'sale.order.line', string='Final Sale Order Line', ondelete='set null',
        readonly=True, states={'available': [('readonly', False)]},
        groups='sales_team.group_sale_salesman', copy=False)
    sale_order_id = fields.Many2one(
        related='sale_order_line_id.order_id', store='True', readonly=True,
        groups='sales_team.group_sale_salesman')
    is_paid = fields.Boolean('Is Paid', copy=False)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_sale_order(self):
        booth_with_so = self.sudo().filtered('sale_order_id')
        if booth_with_so:
            raise UserError(_(
                'You can\'t delete the following booths as they are linked to sales orders: '
                '%(booths)s', booths=', '.join(booth_with_so.mapped('name'))))

    def action_set_paid(self):
        self.write({'is_paid': True})

    def action_view_sale_order(self):
        self.sale_order_id.ensure_one()
        action = self.env['ir.actions.actions']._for_xml_id('sale.action_orders')
        action['views'] = [(False, 'form')]
        action['res_id'] = self.sale_order_id.id
        return action

    def _get_booth_multiline_description(self):
        return '%s : \n%s' % (
            self.event_id.display_name,
            '\n'.join(['- %s' % booth.name for booth in self])
        )

```

## File: models\event_booth_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class EventBoothCategory(models.Model):
    _inherit = 'event.booth.category'

    def _default_product_id(self):
        return self.env.ref('event_booth_sale.product_product_event_booth', raise_if_not_found=False)

    product_id = fields.Many2one(
        'product.product', string='Product', required=True,
        domain=[('detailed_type', '=', 'event_booth')], default=_default_product_id,
        groups="event.group_event_registration_desk")
    price = fields.Float(
        string='Price', compute='_compute_price', digits='Product Price', readonly=False,
        store=True, groups="event.group_event_registration_desk")
    currency_id = fields.Many2one(related='product_id.currency_id', groups="event.group_event_registration_desk")
    price_reduce = fields.Float(
        string='Price Reduce', compute='_compute_price_reduce',
        compute_sudo=True, digits='Product Price', groups="event.group_event_registration_desk")
    price_reduce_taxinc = fields.Float(
        string='Price Reduce Tax inc', compute='_compute_price_reduce_taxinc',
        compute_sudo=True
    )
    image_1920 = fields.Image(compute='_compute_image_1920', readonly=False, store=True)

    @api.depends('product_id')
    def _compute_image_1920(self):
        for category in self:
            category.image_1920 = category.image_1920 if category.image_1920 else category.product_id.image_1920

    @api.depends('product_id')
    def _compute_price(self):
        """ By default price comes from category but can be changed by event
        people as product may be shared across various categories. """
        for category in self:
            if category.product_id and category.product_id.list_price:
                category.price = category.product_id.list_price + category.product_id.price_extra

    @api.depends_context('pricelist', 'quantity')
    @api.depends('product_id', 'price')
    def _compute_price_reduce(self):
        for category in self:
            product = category.product_id
            pricelist = product.product_tmpl_id._get_contextual_pricelist()
            lst_price = product.currency_id._convert(
                product.lst_price,
                pricelist.currency_id,
                self.env.company,
                fields.Datetime.now(),
                round=False,
            )
            discount = (lst_price - product._get_contextual_price()) / lst_price if lst_price else 0.0
            category.price_reduce = (1.0 - discount) * category.price

    @api.depends_context('pricelist', 'quantity')
    @api.depends('product_id', 'price_reduce')
    def _compute_price_reduce_taxinc(self):
        for category in self:
            tax_ids = category.product_id.taxes_id
            taxes = tax_ids.compute_all(category.price_reduce, category.currency_id, 1.0, product=category.product_id)
            category.price_reduce_taxinc = taxes['total_included']

    def _init_column(self, column_name):
        """ Initialize product_id for existing columns when installing sale
        bridge, to ensure required attribute is fulfilled. """
        if column_name != "product_id":
            return super(EventBoothCategory, self)._init_column(column_name)

        # fetch void columns
        self.env.cr.execute("SELECT id FROM %s WHERE product_id IS NULL" % self._table)
        booth_category_ids = self.env.cr.fetchall()
        if not booth_category_ids:
            return

        # update existing columns
        _logger.debug("Table '%s': setting default value of new column %s to unique values for each row",
                      self._table, column_name)
        default_booth_product = self._default_product_id()
        if default_booth_product:
            product_id = default_booth_product.id
        else:
            product_id = self.env['product.product'].create({
                'name': 'Generic Event Booth Product',
                'categ_id': self.env.ref('event_sale.product_category_events').id,
                'list_price': 100,
                'standard_price': 0,
                'detailed_type': 'event_booth',
                'invoice_policy': 'order',
            }).id
            self.env['ir.model.data'].create({
                'name': 'product_product_event_booth',
                'module': 'event_booth_sale',
                'model': 'product.product',
                'res_id': product_id,
            })
        self.env.cr._obj.execute(
            f'UPDATE {self._table} SET product_id = %s WHERE id IN %s;',
            (product_id, tuple(booth_category_ids))
        )

```

## File: models\event_booth_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class EventBoothRegistration(models.Model):
    """event.booth.registrations are used to allow multiple partners to book the same booth.
    Whenever a partner has paid their registration all the others linked to the booth will be deleted."""

    _name = 'event.booth.registration'
    _description = 'Event Booth Registration'

    sale_order_line_id = fields.Many2one('sale.order.line', string='Sale Order Line', required=True, ondelete='cascade')
    event_booth_id = fields.Many2one('event.booth', string='Booth', required=True)
    partner_id = fields.Many2one(
        'res.partner', related='sale_order_line_id.order_partner_id', store=True)
    contact_name = fields.Char(string='Contact Name', compute='_compute_contact_name', readonly=False, store=True)
    contact_email = fields.Char(string='Contact Email', compute='_compute_contact_email', readonly=False, store=True)
    contact_phone = fields.Char(string='Contact Phone', compute='_compute_contact_phone', readonly=False, store=True)
    contact_mobile = fields.Char(string='Contact Mobile', compute='_compute_contact_mobile', readonly=False, store=True)

    _sql_constraints = [('unique_registration', 'unique(sale_order_line_id, event_booth_id)',
                         'There can be only one registration for a booth by sale order line')]

    @api.depends('partner_id')
    def _compute_contact_name(self):
        for registration in self:
            if not registration.contact_name:
                registration.contact_name = registration.partner_id.name or False

    @api.depends('partner_id')
    def _compute_contact_email(self):
        for registration in self:
            if not registration.contact_email:
                registration.contact_email = registration.partner_id.email or False

    @api.depends('partner_id')
    def _compute_contact_phone(self):
        for registration in self:
            if not registration.contact_phone:
                registration.contact_phone = registration.partner_id.phone or False

    @api.depends('partner_id')
    def _compute_contact_mobile(self):
        for registration in self:
            if not registration.contact_mobile:
                registration.contact_mobile = registration.partner_id.mobile or False

    @api.model
    def _get_fields_for_booth_confirmation(self):
        return ['sale_order_line_id', 'partner_id', 'contact_name', 'contact_email', 'contact_phone', 'contact_mobile']

    def action_confirm(self):
        for registration in self:
            values = {
                field: registration[field].id if isinstance(registration[field], models.BaseModel) else registration[field]
                for field in self._get_fields_for_booth_confirmation()
            }
            registration.event_booth_id.action_confirm(values)
        self._cancel_pending_registrations()

    def _cancel_pending_registrations(self):
        body = '<p>%(message)s: <ul>%(booth_names)s</ul></p>' % {
            'message': _('Your order has been cancelled because the following booths have been reserved'),
            'booth_names': ''.join('<li>%s</li>' % booth.display_name for booth in self.event_booth_id)
        }
        other_registrations = self.search([
            ('event_booth_id', 'in', self.event_booth_id.ids),
            ('id', 'not in', self.ids)
        ])
        for order in other_registrations.sale_order_line_id.order_id:
            order.sudo().message_post(
                body=body,
                partner_ids=order.user_id.partner_id.ids,
            )
            order.sudo()._action_cancel()
        other_registrations.unlink()

```

## File: models\event_type_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventTypeBooth(models.Model):
    _inherit = 'event.type.booth'

    product_id = fields.Many2one(related='booth_category_id.product_id')
    price = fields.Float(related='booth_category_id.price', store=True)
    currency_id = fields.Many2one(related='booth_category_id.currency_id')

    @api.model
    def _get_event_booth_fields_whitelist(self):
        res = super(EventTypeBooth, self)._get_event_booth_fields_whitelist()
        return res + ['product_id', 'price']

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    detailed_type = fields.Selection(selection_add=[
        ('event_booth', 'Event Booth'),
    ], ondelete={'event_booth': 'set service'})

    @api.onchange('detailed_type')
    def _onchange_type_event_booth(self):
        if self.detailed_type == 'event_booth':
            self.invoice_policy = 'order'

    def _detailed_type_mapping(self):
        type_mapping = super()._detailed_type_mapping()
        type_mapping['event_booth'] = 'service'
        return type_mapping


class Product(models.Model):
    _inherit = 'product.product'

    @api.onchange('detailed_type')
    def _onchange_type_event_booth(self):
        if self.detailed_type == 'event_booth':
            self.invoice_policy = 'order'

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    event_booth_ids = fields.One2many('event.booth', 'sale_order_id', string='Booths')
    event_booth_count = fields.Integer(string='Booth Count', compute='_compute_event_booth_count')

    @api.depends('event_booth_ids')
    def _compute_event_booth_count(self):
        if self.ids:
            slot_data = self.env['event.booth']._read_group(
                [('sale_order_id', 'in', self.ids)],
                ['sale_order_id'], ['sale_order_id']
            )
            slot_mapped = dict((data['sale_order_id'][0], data['sale_order_id_count']) for data in slot_data)
        else:
            slot_mapped = dict()
        for so in self:
            so.event_booth_count = slot_mapped.get(so.id, 0)

    def action_confirm(self):
        res = super(SaleOrder, self).action_confirm()
        for so in self:
            so.order_line._update_event_booths()
        return res

    def action_view_booth_list(self):
        action = self.env['ir.actions.act_window']._for_xml_id('event_booth.event_booth_action')
        action['domain'] = [('sale_order_id', 'in', self.ids)]
        return action

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    event_booth_category_id = fields.Many2one('event.booth.category', string='Booths Category', ondelete='set null')
    event_booth_pending_ids = fields.Many2many(
        'event.booth', string='Pending Booths', search='_search_event_booth_pending_ids',
        compute='_compute_event_booth_pending_ids', inverse='_inverse_event_booth_pending_ids',
        help='Used to create registration when providing the desired event booth.'
    )
    event_booth_registration_ids = fields.One2many(
        'event.booth.registration', 'sale_order_line_id', string='Confirmed Registration')
    event_booth_ids = fields.One2many('event.booth', 'sale_order_line_id', string='Confirmed Booths')
    is_event_booth = fields.Boolean(compute='_compute_is_event_booth')

    @api.depends('product_id.type')
    def _compute_is_event_booth(self):
        for record in self:
            record.is_event_booth = record.product_id.detailed_type == 'event_booth'

    @api.depends('event_booth_ids')
    def _compute_name_short(self):
        wbooth = self.filtered(lambda line: line.event_booth_pending_ids)
        for record in wbooth:
            record.name_short = record.event_booth_pending_ids.event_id.name
        super(SaleOrderLine, self - wbooth)._compute_name_short()

    @api.depends('event_booth_registration_ids')
    def _compute_event_booth_pending_ids(self):
        for so_line in self:
            so_line.event_booth_pending_ids = so_line.event_booth_registration_ids.event_booth_id

    def _inverse_event_booth_pending_ids(self):
        """ This method will take care of creating the event.booth.registrations based on selected booths.
        It will also unlink ones that are de-selected. """

        for so_line in self:
            existing_booths = so_line.event_booth_registration_ids.event_booth_id or self.env[
                'event.booth']
            selected_booths = so_line.event_booth_pending_ids

            so_line.event_booth_registration_ids.filtered(
                lambda reg: reg.event_booth_id not in selected_booths).unlink()

            self.env['event.booth.registration'].create([{
                'event_booth_id': booth.id,
                'sale_order_line_id': so_line.id,
                'partner_id': so_line.order_id.partner_id.id
            } for booth in selected_booths - existing_booths])

    def _search_event_booth_pending_ids(self, operator, value):
        return [('event_booth_registration_ids.event_booth_id', operator, value)]

    @api.constrains('event_booth_registration_ids')
    def _check_event_booth_registration_ids(self):
        if len(self.event_booth_registration_ids.event_booth_id.event_id) > 1:
            raise ValidationError(_('Registrations from the same Order Line must belong to a single event.'))

    @api.onchange('product_id')
    def _onchange_product_id_booth(self):
        """We reset the event when the selected product doesn't belong to any pending booths."""
        if self.event_id and (not self.product_id or self.product_id not in self.event_booth_pending_ids.product_id):
            self.event_id = None

    @api.onchange('event_id')
    def _onchange_event_id_booth(self):
        """We reset the pending booths when the event changes to avoid inconsistent state."""
        if self.event_booth_pending_ids and (not self.event_id or self.event_id != self.event_booth_pending_ids.event_id):
            self.event_booth_pending_ids = None

    @api.depends('event_booth_registration_ids.event_booth_id')
    def _compute_name(self):
        """Override to add the compute dependency.

        The custom name logic can be found below in _get_sale_order_line_multiline_description_sale.
        """
        super()._compute_name()

    def _update_event_booths(self, set_paid=False):
        for so_line in self.filtered('is_event_booth'):
            if so_line.event_booth_pending_ids and not so_line.event_booth_ids:
                unavailable = so_line.event_booth_pending_ids.filtered(lambda booth: not booth.is_available)
                if unavailable:
                    raise ValidationError(
                        _('The following booths are unavailable, please remove them to continue : %(booth_names)s',
                          booth_names=''.join('\n\t- %s' % booth.display_name for booth in unavailable)))
                so_line.event_booth_registration_ids.sudo().action_confirm()
            if so_line.event_booth_ids and set_paid:
                so_line.event_booth_ids.sudo().action_set_paid()
        return True

    def _get_sale_order_line_multiline_description_sale(self):
        if self.event_booth_pending_ids:
            return self.event_booth_pending_ids._get_booth_multiline_description()
        return super()._get_sale_order_line_multiline_description_sale()

    def _get_display_price(self):
        if self.event_booth_pending_ids and self.event_id:
            company = self.event_id.company_id or self.env.company
            currency = company.currency_id
            pricelist = self.order_id.pricelist_id
            if pricelist.discount_policy == "with_discount":
                total_price = sum([booth.booth_category_id.with_context(pricelist=pricelist.id).price_reduce for booth in self.event_booth_pending_ids])
            else:
                total_price = sum([booth.price for booth in self.event_booth_pending_ids])
            return currency._convert(
                total_price, self.order_id.currency_id,
                self.order_id.company_id or self.env.company.id,
                self.order_id.date_order or fields.Date.today())
        return super()._get_display_price()

```

## File: models\sale_order_template_line.py

```python
from odoo import fields, models

class SaleOrderTemplateLine(models.Model):
    _inherit = "sale.order.template.line"

    product_id = fields.Many2one(domain="[('sale_ok', '=', True), ('detailed_type', 'not in', ['event', 'event_booth']), ('company_id', 'in', [company_id, False])]")

```

## File: models\sale_order_template_option.py

```python
from odoo import fields, models

class SaleOrderTemplateOption(models.Model):
    _inherit = "sale.order.template.option"

    product_id = fields.Many2one(domain="[('sale_ok', '=', True), ('detailed_type', 'not in', ['event', 'event_booth']), ('company_id', 'in', [company_id, False])]")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import product
from . import event_booth_registration
from . import event_booth
from . import event_booth_category
from . import event_type_booth
from . import sale_order
from . import sale_order_line
from . import sale_order_template_line
from . import sale_order_template_option

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_booth_registration_salesman,access.event.booth.registration.salesman,model_event_booth_registration,sales_team.group_sale_salesman,1,1,1,1
access_event_booth_registration_event_desk,access.event.booth.registration.event.desk,model_event_booth_registration,event.group_event_registration_desk,1,0,0,0
access_event_booth_registration_event_user,access.event.booth.registration.event.user,model_event_booth_registration,event.group_event_user,1,1,1,1
access_event_booth_configurator_salesman,access.event.booth.configurator.salesman,model_event_booth_configurator,sales_team.group_sale_salesman,1,1,1,0

```

## File: static\src\js\event_booth_configurator_model.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { Record, RelationalModel } from "@web/views/relational_model";

/**
 * This model is overridden to allow configuring sale_order_lines through a popup
 * window when a product with 'detailed_type' == 'event_booth' is selected.
 *
 * This allows keeping an editable list view for sales order and remove the noise of
 * those 3 fields ('event_id', 'event_booth_category_id' and 'event_booth_ids')
 */
class EventBoothConfiguratorRelationalModel extends RelationalModel {}

class EventBoothConfiguratorRecord extends Record {
    //--------------------------------------------------------------------------
    // Overrides
    //--------------------------------------------------------------------------
    async save() {
        const isSaved = await super.save(...arguments);
        if (!isSaved) {
            return false;
        }
        this.model.action.doAction({type: 'ir.actions.act_window_close', infos: {
            eventBoothConfiguration: {
                event_id: this.data.event_id,
                event_booth_category_id: this.data.event_booth_category_id,
                event_booth_pending_ids: {
                    operation: 'MULTI',
                    commands: [{
                        operation: 'REPLACE_WITH',
                        ids: this.data.event_booth_ids.currentIds,
                    }],
                }
            }
        }});
        return true;
    }
}

EventBoothConfiguratorRelationalModel.Record = EventBoothConfiguratorRecord;

registry.category("views").add("event_booth_configurator_form", {
    ...formView,
    Model: EventBoothConfiguratorRelationalModel,
});

```

## File: static\src\js\sale_product_field.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { SaleOrderLineProductField } from '@sale/js/sale_product_field';


patch(SaleOrderLineProductField.prototype, 'event_booth_sale', {

    async _onProductUpdate() {
        this._super(...arguments);
        if (this.props.record.data.product_type === 'event_booth') {
            this._openEventBoothConfigurator(false);
        }
    },

    _editLineConfiguration() {
        this._super(...arguments);
        if (this.props.record.data.product_type === 'event_booth') {
            this._openEventBoothConfigurator(true);
        }
    },

    get isConfigurableLine() {
        return this._super(...arguments) || Boolean(this.props.record.data.event_booth_category_id);
    },

    async _openEventBoothConfigurator(edit) {
        let actionContext = {
            default_product_id: this.props.record.data.product_id[0],
        };
        if (edit) {
            const recordData = this.props.record.data;
            if (recordData.event_id) {
                actionContext.default_event_id = recordData.event_id[0];
            }
            if (recordData.event_booth_category_id) {
                actionContext.default_event_booth_category_id = recordData.event_booth_category_id[0];
            }
            if (recordData.event_booth_pending_ids) {
                actionContext.default_event_booth_ids = recordData.event_booth_pending_ids.records.map(
                    record => {
                        return [4, record.data.id];
                    }
                );
            }
        }
        this.action.doAction(
            'event_booth_sale.event_booth_configurator_action',
            {
                additionalContext: actionContext,
                onClose: async (closeInfo) => {
                    if (!closeInfo || closeInfo.special) {
                        // wizard popup closed or 'Cancel' button triggered
                        if (!this.props.record.data.event_ticket_id) {
                            // remove product if event configuration was cancelled.
                            this.props.record.update({
                                [this.props.name]: undefined,
                            });
                        }
                    } else {
                        const eventBoothConfiguration = closeInfo.eventBoothConfiguration;
                        this.props.record.update({
                            event_id: eventBoothConfiguration.event_id,
                            event_booth_category_id: eventBoothConfiguration.event_booth_category_id,
                            event_booth_pending_ids: eventBoothConfiguration.event_booth_pending_ids,
                        });
                    }
                }
            }
        );
    },
});

```

## File: views\event_booth_category_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_category_view_form" model="ir.ui.view">
        <field name="name">event.booth.category.view.form.inherit.sale</field>
        <field name="model">event.booth.category</field>
        <field name="inherit_id" ref="event_booth.event_booth_category_view_form"/>
        <field name="priority" eval="1"/>
        <field name="arch" type="xml">
            <group name="main" position="inside">
                <group string="Booth Details">
                    <field name="currency_id" invisible="1"/>
                    <field name="product_id" context="{'default_detailed_type': 'event_booth', 'default_detailed_type': 'service'}"/>
                    <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                </group>
            </group>
        </field>
    </record>

    <record id="event_booth_category_view_tree" model="ir.ui.view">
        <field name="name">event.booth.category.view.tree.inherit.sale</field>
        <field name="model">event.booth.category</field>
        <field name="inherit_id" ref="event_booth.event_booth_category_view_tree"/>
        <field name="priority">3</field>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="product_id"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </field>
        </field>
    </record>

</data></odoo>

```

## File: views\event_booth_registration_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_registration_view_form" model="ir.ui.view">
        <field name="name">event.booth.registration.view.form</field>
        <field name="model">event.booth.registration</field>
        <field name="arch" type="xml">
            <form string="Booth Registration">
                <sheet>
                    <div class="oe_title">
                        <h1><field name="partner_id"/></h1>
                    </div>
                    <group>
                        <group string="Details">
                            <field name="event_booth_id"/>
                            <field name="sale_order_line_id"/>
                        </group>
                        <group string="Contact">
                            <field name="contact_name"/>
                            <field name="contact_email"/>
                            <field name="contact_phone"/>
                            <field name="contact_mobile"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_booth_registration_view_tree" model="ir.ui.view">
        <field name="name">event.booth.registration.view.tree</field>
        <field name="model">event.booth.registration</field>
        <field name="arch" type="xml">
            <tree string="Booth Registration">
                <field name="partner_id"/>
                <field name="event_booth_id"/>
                <field name="sale_order_line_id"/>
                <field name="contact_name" optional="hide"/>
                <field name="contact_email" optional="hide"/>
                <field name="contact_phone" optional="hide"/>
                <field name="contact_mobile" optional="hide"/>
            </tree>
        </field>
    </record>

</data></odoo>

```

## File: views\event_booth_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_view_form_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.form.inherit.sale</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_form_from_event"/>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="action_view_sale_order" type="object" class="oe_stat_button"
                        icon="fa-usd" groups="sales_team.group_sale_salesman"
                        string="Sale Order" attrs="{'invisible': [('sale_order_id', '=', False)]}">
                </button>
            </div>
            <div name="button_box" position="after">
                <field name="is_paid" invisible="1"/>
                <widget name="web_ribbon" title="Paid" attrs="{'invisible': [('is_paid', '=', False)]}"/>
            </div>
            <field name="booth_category_id" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="product_id" attrs="{'invisible': [('booth_category_id', '=', False)]}"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"
                       attrs="{'invisible': [('booth_category_id', '=', False)]}"/>
            </field>
            <group name="renter" position="after">
                <group name="sales" groups="base.group_no_one"
                       attrs="{'invisible': [('sale_order_line_id', '=', False)]}">
                    <field name="sale_order_id"/>
                    <field name="sale_order_line_id"/>
                </group>
            </group>
            <xpath expr="//sheet" position="inside">
                <notebook groups="base.group_no_one">
                    <page string="Registrations">
                        <field name="event_booth_registration_ids" readonly="1"/>
                    </page>
                </notebook>
            </xpath>
        </field>
    </record>

    <record id="event_booth_view_tree_from_event" model="ir.ui.view">
        <field name="name">event.booth.view.tree.from.event.inherit.sale</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_tree_from_event"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}" sum="Total"/>
            </field>
        </field>
    </record>

    <record id="event_booth_view_search" model="ir.ui.view">
        <field name="name">event.booth.view.search.inherit.sale</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="sale_order_id"/>
            </xpath>
            <xpath expr="//search/group/filter[@name='group_by_booth_category_id']" position="after">
                <filter name="is_paid" context="{'group_by': 'is_paid'}"/>
            </xpath>
        </field>
    </record>

    <record id="event_booth_view_graph" model="ir.ui.view">
        <field name="name">event.booth.event.booth.view.graph.inherit.sale</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="price" type="measure"/>
            </xpath>
        </field>
    </record>

    <record id="event_booth_view_pivot" model="ir.ui.view">
        <field name="name">event.booth.event.booth.view.pivot.inherit.sale</field>
        <field name="model">event.booth</field>
        <field name="inherit_id" ref="event_booth.event_booth_view_pivot"/>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="inside">
                <field name="price" type="measure"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\event_type_booth_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_type_booth_view_form_from_type" model="ir.ui.view">
        <field name="name">event.booth.view.form.from.type.inherit.sale</field>
        <field name="model">event.type.booth</field>
        <field name="inherit_id" ref="event_booth.event_type_booth_view_form_from_type"/>
        <field name="arch" type="xml">
            <field name="booth_category_id" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="product_id"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </field>
        </field>
    </record>

    <record id="event_type_booth_view_tree_from_type" model="ir.ui.view">
        <field name="name">event.booth.view.tree.from.type.inherit.sale</field>
        <field name="model">event.type.booth</field>
        <field name="inherit_id" ref="event_booth.event_type_booth_view_tree_from_type"/>
        <field name="arch" type="xml">
            <field name="booth_category_id" position="after">
                <field name="currency_id" invisible="1"/>
                <field name="product_id"/>
                <field name="price" widget="monetary" options="{'currency_field': 'currency_id'}"/>
            </field>
        </field>
    </record>

</data></odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="sale_order_view_form" model="ir.ui.view">
        <field name="name">sale.order.view.form.inherit.event.booth.sale</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_attendee_list']" position="after">
                <field name="event_booth_count" invisible="1"/>
                <button name="action_view_booth_list" type="object" class="oe_stat_button" icon="fa-university"
                        attrs="{'invisible': [('event_booth_count', '=', 0)]}">
                    <field name="event_booth_count" widget="statinfo" string="Booths"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='order_line']//tree//field[@name='event_ticket_id']" position="after">
                <field name="is_event_booth" optional="hide"/>
                <field name="event_booth_category_id" optional="hide"/>
                <field name="event_booth_pending_ids" optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='order_line']//tree//field[@name='product_uom_qty']" position="attributes">
                <attribute name="attrs">{'readonly': [('is_event_booth', '=', True)]}</attribute>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: wizard\event_booth_configurator.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _
from odoo.exceptions import ValidationError


class EventBoothConfigurator(models.TransientModel):
    _name = 'event.booth.configurator'
    _description = 'Event Booth Configurator'

    product_id = fields.Many2one('product.product', string='Product', readonly=True)
    sale_order_line_id = fields.Many2one('sale.order.line', string='Sale Order Line', readonly=True)
    event_id = fields.Many2one('event.event', string='Event', required=True)
    event_booth_category_available_ids = fields.Many2many(related='event_id.event_booth_category_available_ids', readonly=True)
    event_booth_category_id = fields.Many2one(
        'event.booth.category', string='Booth Category', required=True,
        compute='_compute_event_booth_category_id', readonly=False, store=True)
    event_booth_ids = fields.Many2many(
        'event.booth', string='Booth', required=True,
        compute='_compute_event_booth_ids', readonly=False, store=True)

    @api.depends('event_id')
    def _compute_event_booth_category_id(self):
        self.event_booth_category_id = False

    @api.depends('event_id', 'event_booth_category_id')
    def _compute_event_booth_ids(self):
        self.event_booth_ids = False

    @api.constrains('event_booth_ids')
    def _check_if_no_booth_ids(self):
        if any(not wizard.event_booth_ids for wizard in self):
            raise ValidationError(_('You have to select at least one booth.'))

```

## File: wizard\event_booth_configurator_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_booth_configurator_view_form" model="ir.ui.view">
        <field name="name">event.booth.configurator.view.form</field>
        <field name="model">event.booth.configurator</field>
        <field name="arch" type="xml">
            <form js_class="event_booth_configurator_form">
                <group>
                    <field name="product_id" invisible="1"/>
                    <field name="event_id" options="{'no_open': True, 'no_create': True}"
                           domain="[
                                ('event_booth_ids.product_id', '=', product_id),
                                ('is_finished', '=', False)
                           ]"/>
                    <field name="event_booth_category_available_ids" invisible="1"/>
                    <field name="event_booth_category_id" options="{'no_open': True, 'no_create': True}"
                           attrs="{'invisible': [('event_id', '=', False)]}"
                           domain="[('id', 'in', event_booth_category_available_ids)]"/>
                    <field name="event_booth_ids" options="{'no_open': True, 'no_create': True}"
                           widget="many2many_checkboxes" attrs="{'invisible': [('event_booth_category_id', '=', False)]}"
                           domain="[
                                ('event_id', '=', event_id),
                                ('booth_category_id', '=', event_booth_category_id),
                                ('product_id', '=', product_id),
                                ('state', '=', 'available')
                           ]"/>
                </group>
                <footer>
                    <button string="Ok" class="btn-primary" special="save"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="event_booth_configurator_action" model="ir.actions.act_window">
        <field name="name">Select an event booth</field>
        <field name="res_model">event.booth.configurator</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

</data></odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth_configurator

```

