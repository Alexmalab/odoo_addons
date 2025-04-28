# Odoo Module: product_expiry

Category: Inventory/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

from odoo import api, SUPERUSER_ID, Command

def _enable_tracking_numbers(env):
    """ This hook ensures the tracking numbers are enabled when the module is installed since the
    user can install `product_expiry` manually without enable `group_production_lot`.
    """
    group_production_lot = env.ref('stock.group_production_lot')
    groups = env.ref('base.group_user') + env.ref('base.group_portal')
    groups.write({'implied_ids': [Command.link(group_production_lot.id)]})

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Products Expiration Date',
    'category': 'Inventory/Inventory',
    'depends': ['stock'],
    'description': """
Track different dates on products and production lots.
======================================================

Following dates can be tracked:
-------------------------------
    - end of life
    - best before date
    - removal date
    - alert date

Also implements the removal strategy First Expiry First Out (FEFO) widely used, for example, in food industries.
""",
    'data': ['security/ir.model.access.csv',
             'security/stock_security.xml',
             'views/production_lot_views.xml',
             'views/product_template_views.xml',
             'views/res_config_settings_views.xml',
             'views/stock_move_views.xml',
             'views/stock_quant_views.xml',
             'wizard/confirm_expiry_view.xml',
             'report/report_deliveryslip.xml',
             'report/report_lot_barcode.xml',
             'data/product_expiry_data.xml',
             'data/mail_activity_type_data.xml',
            ],
    'post_init_hook': '_enable_tracking_numbers',
    'license': 'LGPL-3',
}

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_activity_type_alert_date_reached" model="mail.activity.type">
        <field name="name">Alert Date Reached</field>
        <field name="category">default</field>
        <field name="res_model">stock.lot</field>
        <field name="icon">fa-tasks</field>
        <field name="delay_count">0</field>
    </record>
</odoo>

```

## File: data\product_expiry_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="removal_fefo" model="product.removal">
            <field name="name">First Expiry First Out (FEFO)</field>
            <field name="method">fefo</field>
        </record>
</odoo>

```

## File: models\production_lot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import datetime
from odoo import api, fields, models, SUPERUSER_ID, _


class StockLot(models.Model):
    _inherit = 'stock.lot'

    use_expiration_date = fields.Boolean(
        string='Use Expiration Date', related='product_id.use_expiration_date')
    expiration_date = fields.Datetime(
        string='Expiration Date', compute='_compute_expiration_date', store=True, readonly=False,
        help='This is the date on which the goods with this Serial Number may become dangerous and must not be consumed.')
    use_date = fields.Datetime(string='Best before Date', compute='_compute_dates', store=True, readonly=False,
        help='This is the date on which the goods with this Serial Number start deteriorating, without being dangerous yet.')
    removal_date = fields.Datetime(string='Removal Date', compute='_compute_dates', store=True, readonly=False,
        help='This is the date on which the goods with this Serial Number should be removed from the stock. This date will be used in FEFO removal strategy.')
    alert_date = fields.Datetime(string='Alert Date', compute='_compute_dates', store=True, readonly=False,
        help='Date to determine the expired lots and serial numbers using the filter "Expiration Alerts".')
    product_expiry_alert = fields.Boolean(compute='_compute_product_expiry_alert', help="The Expiration Date has been reached.")
    product_expiry_reminded = fields.Boolean(string="Expiry has been reminded")

    @api.depends('expiration_date')
    def _compute_product_expiry_alert(self):
        current_date = fields.Datetime.now()
        for lot in self:
            if lot.expiration_date:
                lot.product_expiry_alert = lot.expiration_date <= current_date
            else:
                lot.product_expiry_alert = False

    @api.depends('product_id')
    def _compute_expiration_date(self):
        self.expiration_date = False
        for lot in self:
            if lot.product_id.use_expiration_date and not lot.expiration_date:
                duration = lot.product_id.product_tmpl_id.expiration_time
                lot.expiration_date = datetime.datetime.now() + datetime.timedelta(days=duration)

    @api.depends('product_id', 'expiration_date')
    def _compute_dates(self):
        for lot in self:
            if not lot.product_id.use_expiration_date:
                lot.use_date = False
                lot.removal_date = False
                lot.alert_date = False
            elif lot.expiration_date:
                # when create
                if lot.product_id != lot._origin.product_id or \
                   (not lot.use_date and not lot.removal_date and not lot.alert_date) or \
                   (lot.expiration_date and not lot._origin.expiration_date):
                    product_tmpl = lot.product_id.product_tmpl_id
                    lot.use_date = lot.expiration_date - datetime.timedelta(days=product_tmpl.use_time)
                    lot.removal_date = lot.expiration_date - datetime.timedelta(days=product_tmpl.removal_time)
                    lot.alert_date = lot.expiration_date - datetime.timedelta(days=product_tmpl.alert_time)
                # when change
                elif lot._origin.expiration_date:
                    time_delta = lot.expiration_date - lot._origin.expiration_date
                    lot.use_date = lot._origin.use_date and lot._origin.use_date + time_delta
                    lot.removal_date = lot._origin.removal_date and lot._origin.removal_date + time_delta
                    lot.alert_date = lot._origin.alert_date and lot._origin.alert_date + time_delta

    @api.model
    def _alert_date_exceeded(self):
        """Log an activity on internally stored lots whose alert_date has been reached.

        No further activity will be generated on lots whose alert_date
        has already been reached (even if the alert_date is changed).
        """
        alert_lots = self.env['stock.lot'].search([
            ('alert_date', '<=', fields.Date.today()),
            ('product_expiry_reminded', '=', False)])

        lot_stock_quants = self.env['stock.quant'].search([
            ('lot_id', 'in', alert_lots.ids),
            ('quantity', '>', 0),
            ('location_id.usage', '=', 'internal')])
        alert_lots = lot_stock_quants.mapped('lot_id')

        for lot in alert_lots:
            lot.activity_schedule(
                'product_expiry.mail_activity_type_alert_date_reached',
                user_id=lot.product_id.with_company(lot.company_id).responsible_id.id or lot.product_id.responsible_id.id or SUPERUSER_ID,
                note=_("The alert date has been reached for this lot/serial number")
            )
        alert_lots.write({
            'product_expiry_reminded': True
        })


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    @api.model
    def _run_scheduler_tasks(self, use_new_cursor=False, company_id=False):
        super(ProcurementGroup, self)._run_scheduler_tasks(use_new_cursor=use_new_cursor, company_id=company_id)
        self.env['stock.lot']._alert_date_exceeded()
        if 'scheduler_task_done' in self._context:
            task_done = self._context.get('scheduler_task_done', {'task_done': 0})['task_done'] + 1
            self._context['scheduler_task_done']['task_done'] = task_done
        else:
            task_done = self._get_scheduler_tasks_to_do()

        if use_new_cursor:
            self.env['ir.cron']._notify_progress(done=task_done, remaining=self._get_scheduler_tasks_to_do() - task_done)
            self.env.cr.commit()

    @api.model
    def _get_scheduler_tasks_to_do(self):
        return super()._get_scheduler_tasks_to_do() + 1

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Product(models.Model):
    _inherit = "product.product"

    def action_open_quants(self):
        # Override to hide the `removal_date` column if not needed.
        if not any(product.use_expiration_date for product in self):
            self = self.with_context(hide_removal_date=True)
        return super().action_open_quants()


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    use_expiration_date = fields.Boolean(string='Use Expiration Date',
        help='When this box is ticked, you have the possibility to specify dates to manage'
        ' product expiration, on the product and on the corresponding lot/serial numbers')
    expiration_time = fields.Integer(string='Expiration Date',
        help='Number of days after the receipt of the products (from the vendor'
        ' or in stock after production) after which the goods may become dangerous'
        ' and must not be consumed. It will be computed on the lot/serial number.')
    use_time = fields.Integer(string='Best Before Date',
        help='Number of days before the Expiration Date after which the goods starts'
        ' deteriorating, without being dangerous yet. It will be computed on the lot/serial number.')
    removal_time = fields.Integer(string='Removal Date',
        help='Number of days before the Expiration Date after which the goods'
        ' should be removed from the stock. It will be computed on the lot/serial number.')
    alert_time = fields.Integer(string='Alert Date',
        help='Number of days before the Expiration Date after which an alert should be'
        ' raised on the lot/serial number. It will be computed on the lot/serial number.')

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_expiry_date_on_delivery_slip = fields.Boolean("Display Expiration Dates on Delivery Slips",
        implied_group='product_expiry.group_expiry_date_on_delivery_slip')

    @api.onchange('group_lot_on_delivery_slip')
    def _onchange_group_lot_on_delivery_slip(self):
        if not self.group_lot_on_delivery_slip:
            self.group_expiry_date_on_delivery_slip = False

    @api.onchange('group_stock_production_lot')
    def _onchange_group_stock_production_lot(self):
        super()._onchange_group_stock_production_lot()
        if self.group_stock_production_lot:
            self.module_product_expiry = True

    @api.onchange('module_product_expiry')
    def _onchange_module_product_expiry(self):
        if not self.module_product_expiry:
            self.group_expiry_date_on_delivery_slip = False

```

## File: models\stock_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import dateutil.parser as dparser
from re import findall as re_findall

from odoo import fields, models
from odoo.tools import get_lang


class StockMove(models.Model):
    _inherit = "stock.move"

    use_expiration_date = fields.Boolean(
        string='Use Expiration Date', related='product_id.use_expiration_date')

    def _generate_serial_move_line_commands(self, field_data, location_dest_id=False, origin_move_line=None):
        """Override to add a default `expiration_date` into the move lines values."""
        move_lines_commands = super()._generate_serial_move_line_commands(field_data, location_dest_id, origin_move_line)
        if self.product_id.use_expiration_date:
            date = fields.Datetime.today() + datetime.timedelta(days=self.product_id.expiration_time)
            for move_line_command in move_lines_commands:
                move_line_vals = move_line_command[2]
                if 'expiration_date' not in move_line_vals:
                    move_line_vals['expiration_date'] = date
        return move_lines_commands

    def _convert_string_into_field_data(self, string, options):
        res = super()._convert_string_into_field_data(string, options)
        if not res:
            try:
                datetime = dparser.parse(string, **options)
                if self and not self.use_expiration_date:
                    # The datetime was correctly parsed but this move's product doesn't use expiration date.
                    return "ignore"
                return {'expiration_date': datetime}
            except ValueError:
                pass
        return res

    def _get_formating_options(self, strings):
        options = super()._get_formating_options(strings)
        separators = "-/ "
        date_regex = f'[^{separators}]+'
        for string in strings:
            # Searches for a date.
            date_data = re_findall(date_regex, string)
            if len(date_data) < 2:  # Not enough data.
                continue
            value_1, value_2 = date_data[:2]
            if re_findall('[a-zA-Z]', value_1):
                # Assumes the first value is the mounth (written in letters). Don't add any option
                # as mounth as the first date's value is the default behavior for `dateutil.parse`.
                break
            # Try to guess if the first data is the day or the year.
            if int(value_1) > 31:
                options['yearfirst'] = True
                break
            elif int(value_1) > 12 and (re_findall('[a-zA-Z]', value_2) or int(value_2) <= 12):
                options['dayfirst'] = True
                break
            else:  # Too ambiguous, gets the option from the user's lang's date setting.
                user_lang_format = get_lang(self.env).date_format
                if re_findall('^%[mbB]', user_lang_format):  # First parameter is for month.
                    return options
                elif re_findall('^%[djaA]', user_lang_format):  # First parameter is for day.
                    options['dayfirst'] = True
                    break
                elif re_findall('^%[yY]', user_lang_format):  # First parameter is for year.
                    options['yearfirst'] = True
                    break
        return options

    def _update_reserved_quantity(self, need, location_id, lot_id=None, package_id=None, owner_id=None, strict=True):
        if self.product_id.use_expiration_date:
            return super(StockMove, self.with_context(with_expiration=self.date))._update_reserved_quantity(need, location_id, lot_id, package_id, owner_id, strict)
        return super()._update_reserved_quantity(need, location_id, lot_id, package_id, owner_id, strict)

    def _get_available_quantity(self, location_id, lot_id=None, package_id=None, owner_id=None, strict=False, allow_negative=False):
        if self.product_id.use_expiration_date:
            return super(StockMove, self.with_context(with_expiration=self.date))._get_available_quantity(location_id, lot_id, package_id, owner_id, strict, allow_negative)
        return super()._get_available_quantity(location_id, lot_id, package_id, owner_id, strict, allow_negative)

```

## File: models\stock_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class StockMoveLine(models.Model):
    _inherit = "stock.move.line"

    expiration_date = fields.Datetime(
        string='Expiration Date', compute='_compute_expiration_date', store=True,
        help='This is the date on which the goods with this Serial Number may'
        ' become dangerous and must not be consumed.')
    is_expired = fields.Boolean(related='lot_id.product_expiry_alert')
    use_expiration_date = fields.Boolean(
        string='Use Expiration Date', related='product_id.use_expiration_date')

    def _auto_init(self):
        """ Create column for 'expiration_date' here to avoid MemoryError when letting
        the ORM compute it after module installation. Since both 'lot_id.expiration_date'
        and 'product_id.use_expiration_date' are new fields introduced in this module,
        there is no need for an UPDATE statement here.
        """
        if not column_exists(self._cr, "stock_move_line", "expiration_date"):
            create_column(self._cr, "stock_move_line", "expiration_date", "timestamp")
        return super()._auto_init()

    @api.depends('product_id', 'lot_id.expiration_date', 'picking_id.scheduled_date')
    def _compute_expiration_date(self):
        for move_line in self:
            if move_line.lot_id.expiration_date:
                move_line.expiration_date = move_line.lot_id.expiration_date
            elif move_line.picking_type_use_create_lots:
                if move_line.product_id.use_expiration_date:
                    if not move_line.expiration_date:
                        from_date = move_line.picking_id.scheduled_date or fields.Datetime.today()
                        move_line.expiration_date = from_date + datetime.timedelta(days=move_line.product_id.expiration_time)
                else:
                    move_line.expiration_date = False

    @api.onchange('product_id', 'product_uom_id', 'picking_id')
    def _onchange_product_id(self):
        res = super()._onchange_product_id()
        if self.picking_type_use_create_lots:
            if self.product_id.use_expiration_date:
                from_date = self.picking_id.scheduled_date or fields.Datetime.today()
                self.expiration_date = from_date + datetime.timedelta(days=self.product_id.expiration_time)
            else:
                self.expiration_date = False
        return res

    def _prepare_new_lot_vals(self):
        vals = super()._prepare_new_lot_vals()
        if self.expiration_date:
            vals['expiration_date'] = self.expiration_date
        return vals

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class StockPicking(models.Model):
    _inherit = "stock.picking"

    def _pre_action_done_hook(self):
        res = super()._pre_action_done_hook()
        # We use the 'skip_expired' context key to avoid to make the check when
        # user did already confirmed the wizard about expired lots.
        if res is True and not self.env.context.get('skip_expired'):
            pickings_to_warn_expired = self._check_expired_lots()
            if pickings_to_warn_expired:
                return pickings_to_warn_expired._action_generate_expired_wizard()
        return res

    def _check_expired_lots(self):
        expired_pickings = self.move_line_ids.filtered(lambda ml: ml.lot_id.product_expiry_alert).picking_id
        return expired_pickings

    def _action_generate_expired_wizard(self):
        expired_lot_ids = self.move_line_ids.filtered(lambda ml: ml.lot_id.product_expiry_alert).lot_id.ids
        view_id = self.env.ref('product_expiry.confirm_expiry_view').id
        context = dict(self.env.context)

        context.update({
            'default_picking_ids': [(6, 0, self.ids)],
            'default_lot_ids': [(6, 0, expired_lot_ids)],
        })
        return {
            'name': _('Confirmation'),
            'type': 'ir.actions.act_window',
            'res_model': 'expiry.picking.confirmation',
            'view_mode': 'form',
            'views': [(view_id, 'form')],
            'view_id': view_id,
            'target': 'new',
            'context': context,
        }

```

## File: models\stock_quant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockQuant(models.Model):
    _inherit = 'stock.quant'

    expiration_date = fields.Datetime(related='lot_id.expiration_date', store=True)
    removal_date = fields.Datetime(related='lot_id.removal_date', store=True)
    use_expiration_date = fields.Boolean(related='product_id.use_expiration_date')

    def _get_gs1_barcode(self, gs1_quantity_rules_ai_by_uom):
        barcode = super()._get_gs1_barcode(gs1_quantity_rules_ai_by_uom)
        if self.use_expiration_date:
            if self.lot_id.expiration_date:
                barcode = '17' + self.lot_id.expiration_date.strftime('%y%m%d') + barcode
            if self.lot_id.use_date:
                barcode = '15' + self.lot_id.use_date.strftime('%y%m%d') + barcode
        return barcode

    @api.model
    def _get_removal_strategy_order(self, removal_strategy):
        if removal_strategy == 'fefo':
            return 'removal_date, in_date, id'
        return super()._get_removal_strategy_order(removal_strategy)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import production_lot
from . import product_product
from . import res_config_settings
from . import stock_move_line
from . import stock_move
from . import stock_picking
from . import stock_quant

```

## File: report\report_deliveryslip.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="stock_report_delivery_document_inherit_product_expiry" inherit_id="stock.report_delivery_document">
        <xpath expr="//th[@name='lot_serial']" position="after">
            <t t-set="has_expiry_date" t-value="False"/>
            <t t-set="has_expiry_date"
               t-value="o.move_line_ids.filtered(lambda ml: ml.lot_id.expiration_date)"
               groups="product_expiry.group_expiry_date_on_delivery_slip"/>
            <t name="expiry_date" t-if="has_expiry_date">
                <th>Expiration Date</th>
            </t>
        </xpath>
    </template>

    <template id="stock_report_delivery_has_serial_move_line_inherit_product_expiry" inherit_id="stock.stock_report_delivery_has_serial_move_line">
        <xpath expr="//t[@name='move_line_lot']" position="after">
            <t t-if="has_expiry_date">
                <td><span t-field="move_line.lot_id.expiration_date"/></td>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: report\report_lot_barcode.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="report_lot_label_expiry" inherit_id="stock.report_lot_label">
    <xpath expr="//div[@name='lot_name']" position="after">
        <t t-if="o.use_expiration_date">
            <div class="o_label_4x12" t-if="o.use_date">
                B.b. <t t-out="o.use_date" t-options='{"widget": "date"}'/>
            </div>
            <div class="o_label_4x12" t-if="o.expiration_date">
                Exp. <t t-out="o.expiration_date" t-options='{"widget": "date"}'/>
            </div>
        </t>
    </xpath>
    <xpath expr="//t[@name='gs1_datamatrix_lot']" position="before">
        <t t-if="o.use_expiration_date">
            <t t-if="o.use_date" t-set="final_barcode" t-value="(final_barcode or '') + '15' + o.use_date.strftime('%y%m%d')"/>
            <t t-if="o.expiration_date" t-set="final_barcode" t-value="(final_barcode or '') + '17' + o.expiration_date.strftime('%y%m%d')"/>
        </t>
    </xpath>
</template>

<template id="label_lot_template_view_expiry" inherit_id="stock.label_lot_template_view">
    <xpath expr="//t[@name='datamatrix_lot']" position="before">
        <t t-if="lot['lot_record'].use_expiration_date">
            <t t-if="lot['lot_record'].use_date">
                <t t-set="final_barcode" t-value="(final_barcode or '') + '15' + lot['lot_record'].use_date.strftime('%y%m%d')"/>
^FO100,150
^A0N,44,33^FDBest before: <t t-out="lot['lot_record'].use_date" t-options='{"widget": "date"}'/>^FS
            </t>
            <t t-if="lot['lot_record'].expiration_date">
                <t t-set="final_barcode" t-value="(final_barcode or '') + '17' + lot['lot_record'].expiration_date.strftime('%y%m%d')"/>
                    <t t-if="not lot['lot_record'].use_date">
^FO100,150
                    </t>
                    <t t-else="">
^FO100,200
                    </t>
^A0N,44,33^FDUse by: <t t-out="lot['lot_record'].expiration_date" t-options='{"widget": "date"}'/>^FS
            </t>
        </t>
    </xpath>
    <xpath expr="//t[@name='code128_barcode']" position="before">
        <t t-elif="lot['lot_record'].use_expiration_date and (lot['lot_record'].use_date or lot['lot_record'].expiration_date)">
^FO100,50
^A0N,44,33^FD<t t-out="lot['display_name_markup']"/>^FS
^FO100,100
^A0N,44,33^FDLN/SN: <t t-out="lot['name']"/>^FS
            <t t-if="lot['lot_record'].use_date">
^FO100,150
^A0N,44,33^FDBest before: <t t-out="lot['lot_record'].use_date" t-options='{"widget": "date"}'/>^FS
                <t t-if="lot['lot_record'].expiration_date">
^FO100,200
^A0N,44,33^FDUse by: <t t-out="lot['lot_record'].expiration_date" t-options='{"widget": "date"}'/>^FS
^FO100,250^BY3
                </t>
                <t t-else="">
^FO100,200^BY3
                </t>
^BCN,100,Y,N,N
^FD<t t-out="lot['name']"/>^FS
            </t>
            <t t-elif="lot['lot_record'].expiration_date">
^FO100,150
^A0N,44,33^FDUse by: <t t-out="lot['lot_record'].expiration_date" t-options='{"widget": "date"}'/>^FS
^FO100,200^BY3
^BCN,100,Y,N,N
^FD<t t-out="lot['name']"/>^FS
            </t>
        </t>
    </xpath>
</template>
</odoo>

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_expiry_picking_confirmation","access.expiry.picking.confirmation","model_expiry_picking_confirmation","stock.group_stock_user",1,1,1,0

```

## File: security\stock_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">
    <record id="group_expiry_date_on_delivery_slip" model="res.groups">
        <field name="name">Include expiration dates on delivery slip</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>
</data>
</odoo>

```

## File: views\production_lot_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="view_move_form_expiry" model="ir.ui.view">
        <field name="name">stock.production.lot.inherit.form</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form" />
        <field name="arch" type="xml">
        <xpath expr="//page[@name='description']" position="before">
            <page string="Dates" name="expiration_dates" invisible="not use_expiration_date">
                <field name="use_expiration_date" invisible="1"/>
                <group>
                    <group>
                        <field name="expiration_date" />
                        <field name="removal_date" />
                    </group>
                    <group>
                        <field name="use_date" />
                        <field name="alert_date" />
                    </group>
                </group>
            </page>
        </xpath>
        <xpath expr="//div[hasclass('oe_title')]" position="inside">
            <field name="product_expiry_alert" invisible="1"/>
            <span class="badge text-bg-danger" invisible="not product_expiry_alert">Expiration Alert</span>
        </xpath>
        </field>
    </record>

    <record id="search_product_lot_filter_inherit_product_expiry" model="ir.ui.view">
        <field name="name">stock.production.lot.search.inherit</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.search_product_lot_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="after">
                <filter string="Expiration Alerts" name="expiration_alerts" domain="[('alert_date', '&lt;=', time.strftime('%Y-%m-%d %H:%M:%S'))]"/>
            </xpath>
        </field>
     </record>

    <record id="view_production_lot_view_tree" model="ir.ui.view">
        <field name="name">stock.production.lot.list.inherit.product.expiry</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='create_date']" position="after">
                <field name="product_qty" column_invisible="True"/>
                <field name="alert_date" optional="hide" widget='remaining_days' invisible="product_qty &lt;= 0"/>
                <field name="use_date" optional="hide"/>
                <field name="removal_date" optional="hide"/>
                <field name="expiration_date" optional="hide"/>
            </xpath>
        </field>
    </record>

    <record id="view_production_lot_view_kanban" model="ir.ui.view">
        <field name="name">stock.production.lot.kanban.inherit.product.expiry</field>
        <field name="model">stock.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//templates" position="before">
                <field name="product_qty"/>
                <field name="product_expiry_alert"/>
            </xpath>
            <xpath expr="//field[@name='name']" position="after">
                <div>
                    <span title="Alert" class="fa fa-exclamation-triangle text-danger me-2"
                          invisible="product_qty &lt;= 0 or not product_expiry_alert or not expiration_date"/>
                    <field name="alert_date" widget='remaining_days' class="d-inline-block"/>
                </div>
            </xpath>
            <xpath expr="//field[@name='product_id']" position="after">
                <div invisible="not expiration_date">
                    <div>
                        <label for="expiration_date">Expiration: </label>
                        <field class="ms-2" name="expiration_date"/>
                    </div>
                    <div>
                        <label for="removal_date">Removal: </label>
                        <field class="ms-2" name="removal_date"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="view_product_form_expiry" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="stock.view_template_property_form" />
            <field name="arch" type="xml">
                <group name="traceability" position="attributes">
                    <attribute name="invisible">tracking == "none"</attribute>
                </group>
                <group name="traceability" position="inside">
                    <field name="use_expiration_date" string="Expiration Date" invisible="tracking == 'none'"/>
                </group>
                <group name="stock_property" position="after">
                    <group string="Dates" name="expiry_and_lots" groups="stock.group_production_lot"
                        invisible="tracking == 'none' or not use_expiration_date">
                        <label for="expiration_time"/>
                        <div>
                            <field name="expiration_time" class="oe_inline"/>
                            <span> days after receipt</span>
                        </div>
                        <label for="use_time"/>
                        <div>
                            <field name="use_time" class="oe_inline"/>
                            <span> days before expiration date</span>
                        </div>
                        <label for="removal_time"/>
                        <div>
                            <field name="removal_time" class="oe_inline"/>
                            <span> days before expiration date</span>
                        </div>
                        <label for="alert_time"/>
                        <div>
                            <field name="alert_time" class="oe_inline"/>
                            <span> days before expiration date</span>
                        </div>
                    </group>
                </group>
            </field>
        </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_stock" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.product.expiry.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='group_lot_on_delivery_slip']" position="after">
                <setting help="Expiration dates will appear on the delivery slip" invisible="not group_lot_on_delivery_slip or not module_product_expiry" id="group_expiry_date_on_delivery_slip">
                    <field name="group_expiry_date_on_delivery_slip"/>
                </setting>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_move_operations_expiry" model="ir.ui.view">
        <field name="name">stock.move.operations.inherit.form</field>
        <field name="model">stock.move</field>
        <field name="inherit_id" ref="stock.view_stock_move_operations"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="after" >
                <field name="picking_code" invisible="1"/>
                <field name="use_expiration_date" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="view_stock_move_line_operation_tree_expiry" model="ir.ui.view">
        <field name="name">stock.move.line.inherit.list</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_stock_move_line_operation_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_id']" position="after">
                <field name="is_expired" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='lot_name']" position="after" >
                <field name="picking_type_use_existing_lots" column_invisible="True"/>
                <field name="expiration_date" force_save="1" column_invisible="not parent.use_expiration_date" readonly="picking_type_use_existing_lots"
                 decoration-danger="is_expired if state == 'done' else expiration_date &lt; context_today().strftime('%Y-%m-%d')"
                 decoration-bf="is_expired if state == 'done' else expiration_date &lt; context_today().strftime('%Y-%m-%d')"/>
            </xpath>
        </field>
    </record>

    <record id="view_stock_move_line_detailed_operation_tree_expiry" model="ir.ui.view">
        <field name="name">stock.move.line.operations.inherit.list</field>
        <field name="model">stock.move.line</field>
        <field name="inherit_id" ref="stock.view_stock_move_line_detailed_operation_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_name']" position="after">
                <field name="picking_type_use_existing_lots" column_invisible="True"/>
                <field name="tracking" column_invisible="True"/>
                <field name="expiration_date" decoration-danger="is_expired if state == 'done' else expiration_date &lt; context_today().strftime('%Y-%m-%d')"
                 decoration-bf="is_expired if state == 'done' else expiration_date &lt; context_today().strftime('%Y-%m-%d')"
                 force_save="1" readonly="picking_type_use_existing_lots" invisible="tracking == 'none'"/>
            </xpath>
            <xpath expr="//field[@name='lot_id']" position="after">
                <field name="is_expired" column_invisible="True"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_quant_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="view_stock_quant_tree" model="ir.ui.view">
        <field name="name">stock.quant.list.inherit.expiry_date</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.view_stock_quant_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="decoration-danger">
                    removal_date &lt; current_date or quantity &lt; 0
                </attribute>
            </xpath>
            <xpath expr="//field[@name='quantity']" position="after" >
                <field name="removal_date"/>
            </xpath>
        </field>
    </record>

    <record id="view_stock_quant_tree_editable" model="ir.ui.view">
        <field name="name">stock.quant.list.editable.inherit.expiry_date</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.view_stock_quant_tree_editable"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_id']" position="after">
                <field name="use_expiration_date" column_invisible="True"/>
                <field name="expiration_date" optional="hide" column_invisible="context.get('hide_removal_date')"/>
                <field name="removal_date" optional="hide" column_invisible="context.get('hide_removal_date')"/>
            </xpath>
        </field>
    </record>

    <record id="view_stock_quant_tree_inventory_editable" model="ir.ui.view">
        <field name="name">stock.quant.inventory.list.editable.inherit.expiry_date</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.view_stock_quant_tree_inventory_editable"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lot_id']" position="after">
                <field name="use_expiration_date" column_invisible="True"/>
                <field name="expiration_date" groups="stock.group_production_lot"
                    optional="hide" column_invisible="context.get('hide_removal_date')"/>
            </xpath>
        </field>
    </record>

    <record id="quant_search_view_inherit_product_expiry" model="ir.ui.view">
        <field name="name">stock.quant.search.inherit</field>
        <field name="model">stock.quant</field>
        <field name="inherit_id" ref="stock.quant_search_view"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='reserved']" position="after">
                <separator/>
                <filter string="Expiration Alerts" name="expiration_alerts"
                    domain="[('lot_id.alert_date', '&lt;=', context_today().strftime('%Y-%m-%d'))]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\confirm_expiry.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ConfirmExpiry(models.TransientModel):
    _name = 'expiry.picking.confirmation'
    _description = 'Confirm Expiry'

    lot_ids = fields.Many2many('stock.lot', readonly=True, required=True)
    picking_ids = fields.Many2many('stock.picking', readonly=True)
    description = fields.Char('Description', compute='_compute_descriptive_fields')
    show_lots = fields.Boolean('Show Lots', compute='_compute_descriptive_fields')

    @api.depends('lot_ids')
    def _compute_descriptive_fields(self):
        # Shows expired lots only if we are more than one expired lot.
        self.show_lots = len(self.lot_ids) > 1
        if self.show_lots:
            # For multiple expired lots, they are listed in the wizard view.
            self.description = _(
                "You are going to deliver some product expired lots."
                "\nDo you confirm you want to proceed?"
            )
        else:
            # For one expired lot, its name is written in the wizard message.
            self.description = _(
                "You are going to deliver the product %(product_name)s, %(lot_name)s which is expired."
                "\nDo you confirm you want to proceed?",
                product_name=self.lot_ids.product_id.display_name,
                lot_name=self.lot_ids.name
            )

    def process(self):
        picking_to_validate = self.env.context.get('button_validate_picking_ids')
        if picking_to_validate:
            picking_to_validate = self.env['stock.picking'].browse(picking_to_validate)
            ctx = dict(self.env.context, skip_expired=True)
            ctx.pop('default_lot_ids')
            return picking_to_validate.with_context(ctx).button_validate()
        return True

    def process_no_expired(self):
        """ Don't process for concerned pickings (ones with expired lots), but
        process for all other pickings (in case of multi). """
        # Remove `self.pick_ids` from `button_validate_picking_ids` and call
        # `button_validate` with the subset (if any).
        pickings_to_validate = self.env['stock.picking'].browse(self.env.context.get('button_validate_picking_ids'))
        pickings_to_validate = pickings_to_validate - self.picking_ids
        if pickings_to_validate:
            return pickings_to_validate.with_context(skip_expired=True).button_validate()
        return True

```

## File: wizard\confirm_expiry_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="confirm_expiry_view" model="ir.ui.view">
        <field name="name">Confirm</field>
        <field name="model">expiry.picking.confirmation</field>
        <field name="arch" type="xml">
            <form string="Confirmation">
                <p>
                    <field name="description"/>
                </p>
                <field name="show_lots" invisible="1"/>
                <field name="lot_ids" invisible="not show_lots">
                    <list string="Expired Lot(s)">
                        <field name="product_id"/>
                        <field name="name"/>
                    </list>
                </field>
                <footer>
                    <button name="process"
                        string="Confirm"
                        type="object"
                        data-hotkey="q"
                        class="btn-primary"/>
                    <button name="process_no_expired"
                        string="Proceed except for expired one"
                        type="object"
                        data-hotkey="w"
                        class="btn-secondary"/>
                    <button string="Discard"
                        class="btn-secondary"
                        special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import confirm_expiry

```

