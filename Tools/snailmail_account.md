# Odoo Module: snailmail_account

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "snailmail_account",
    'description': """
Allows users to send invoices by post
=====================================================
        """,
    'category': 'Tools',
    'version': '0.1',
    'depends': ['account', 'snailmail'],
    'data': [
        'views/res_config_settings_views.xml',
        'views/assets.xml',
        'wizard/account_invoice_send_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = "res.company"

    invoice_is_snailmail = fields.Boolean(string='Send by Post', default=False)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-	
# Part of Odoo. See LICENSE file for full copyright and licensing details.	

from odoo import fields, models	


class ResConfigSettings(models.TransientModel):	
    _inherit = 'res.config.settings'	

    invoice_is_snailmail = fields.Boolean(string='Send by Post', related='company_id.invoice_is_snailmail', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_company
from . import res_config_settings

```

## File: static\src\js\snailmail_account_notification_manager.js

```javascript
odoo.define('snailmail_account.NotificationManager', function (require) {
"use strict";

var AbstractService = require('web.AbstractService');
var core = require("web.core");

var SnailmailAccountNotificationManager =  AbstractService.extend({
    dependencies: ['bus_service'],

    /**
     * @override
     */
    start: function () {
        this._super.apply(this, arguments);
        this.call('bus_service', 'onNotification', this, this._onNotification);
    },

    _onNotification: function (notifs) {
        var self = this;
        _.each(notifs, function (notif) {
            var model = notif[0][1];
            var type = notif[1].type;
            if (model === 'res.partner' && type === 'snailmail_invalid_address') {
                self.do_warn(notif[1].title,  notif[1].message);
            }
        });
    }

});

core.serviceRegistry.add('snailmail_account_notification_service', SnailmailAccountNotificationManager);

return SnailmailAccountNotificationManager;

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="iap assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/snailmail_account/static/src/js/snailmail_account_notification_manager.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.snailmail.account</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="100"/>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='send_default']" position="inside">
                <div class="row" attrs="{'invisible': [('module_snailmail_account', '=', False)]}">
                    <field name="invoice_is_snailmail" class="col-lg-1 ml16"/>
                    <label for="invoice_is_snailmail"/>
                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                </div>
            </xpath>

            <div id="snailmail_settings" position="inside">
                <div class="mt16" attrs="{'invisible': [('module_snailmail_account', '=', False)]}">
                    <div class="content-group">
                        <div class="row">
                            <field name="snailmail_color" class="col-lg-1 ml16"/>
                            <label for="snailmail_color"/>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                        </div>
                        <div class="row">
                            <field name="snailmail_duplex" class="col-lg-1 ml16"/>
                            <label for="snailmail_duplex"/>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                        </div>
                        <div class="row">
                            <field name="snailmail_cover" class="col-lg-1 ml16"/>
                            <label for="snailmail_cover"/>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                        </div>
                    </div>
                </div>
                <widget name="iap_buy_more_credits" service_name="snailmail"/>
            </div>
        </field>
    </record>
</odoo>

```

## File: wizard\account_invoice_send.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class AccountInvoiceSend(models.TransientModel):
    _name = 'account.invoice.send'
    _inherit = 'account.invoice.send'
    _description = 'Account Invoice Send'

    partner_id = fields.Many2one('res.partner', compute='_get_partner', string='Partner')
    snailmail_is_letter = fields.Boolean('Send by Post', help='Allows to send the document by Snailmail (coventional posting delivery service)', default=lambda self: self.env.company.invoice_is_snailmail)
    snailmail_cost = fields.Float(string='Stamp(s)', compute='_compute_snailmail_cost', readonly=True)
    invalid_addresses = fields.Integer('Invalid Addresses Count', compute='_compute_invalid_addresses')
    invalid_invoice_ids = fields.Many2many('account.move', string='Invalid Addresses', compute='_compute_invalid_addresses')

    @api.depends('invoice_ids')
    def _compute_invalid_addresses(self):
        for wizard in self:
            invalid_invoices = wizard.invoice_ids.filtered(lambda i: not i.partner_id or not self.env['snailmail.letter']._is_valid_address(i.partner_id))
            wizard.invalid_invoice_ids = invalid_invoices
            wizard.invalid_addresses = len(invalid_invoices)

    @api.depends('invoice_ids')
    def _get_partner(self):
        self.partner_id = self.env['res.partner']
        for wizard in self:
            if wizard.invoice_ids and len(wizard.invoice_ids) == 1:
                wizard.partner_id = wizard.invoice_ids.partner_id.id

    @api.depends('snailmail_is_letter')
    def _compute_snailmail_cost(self):
        for wizard in self:
            wizard.snailmail_cost = len(wizard.invoice_ids.ids)

    def snailmail_print_action(self):
        self.ensure_one()
        letters = self.env['snailmail.letter']
        for invoice in self.invoice_ids:
            letter = self.env['snailmail.letter'].create({
                'partner_id': invoice.partner_id.id,
                'model': 'account.move',
                'res_id': invoice.id,
                'user_id': self.env.user.id,
                'company_id': invoice.company_id.id,
                'report_template': self.env.ref('account.account_invoices').id
            })
            letters |= letter

        self.invoice_ids.filtered(lambda inv: not inv.invoice_sent).write({'invoice_sent': True})
        if len(self.invoice_ids) == 1:
            letters._snailmail_print()
        else:
            letters._snailmail_print(immediate=False)

    def send_and_print_action(self):
        if self.snailmail_is_letter:
            if self.invalid_addresses and self.composition_mode == "mass_mail":
                self.notify_invalid_addresses()
            self.snailmail_print_action()
        res = super(AccountInvoiceSend, self).send_and_print_action()
        return res

    def notify_invalid_addresses(self):
        self.ensure_one()
        self.env['bus.bus'].sendone(
            (self._cr.dbname, 'res.partner', self.env.user.partner_id.id),
            {'type': 'snailmail_invalid_address', 'title': _("Invalid Addresses"),
            'message': _("%s of the selected invoice(s) had an invalid address and were not sent") % self.invalid_addresses}
        )

    def invalid_addresses_action(self):
        return {
            'name': _('Invalid Addresses'),
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,tree,form',
            'res_model': 'account.move',
            'domain': [('id', 'in', self.mapped('invalid_invoice_ids').ids)],
        }

```

## File: wizard\account_invoice_send_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="account_invoice_send_inherit_account_wizard_form">
            <field name="name">account.invoice.send.form.inherited.snailmail</field>
            <field name="model">account.invoice.send</field>
            <field name="inherit_id" ref="account.account_invoice_send_wizard_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='option_email']" position='before'>
                    <div name="option_letter">
                        <field name="invalid_addresses" invisible="1"/>
                        <div name="option" class="text-left d-inline-block">
                            <field name="snailmail_is_letter" />
                            <b><label for="snailmail_is_letter"/></b>
                        </div>
                        <span attrs="{'invisible': [('snailmail_is_letter','=', False)]}">
                            <span class="mr4" attrs="{'invisible': [('snailmail_cost', '=', 0)]}">
                                <b>(
                                    <span>
                                        <field name="snailmail_cost" options="{'digits':[0,0]}" class="mr4"/>
                                        <label for="snailmail_cost" class="mr4"/>
                                    </span>
                                    <i class="fa fa-info-circle" role="img" aria-label="Warning" title="Make sure you have enough Stamps on your account."/>
                                )</b>
                            </span>
                            <span attrs="{'invisible': [('composition_mode', '=', 'mass_mail')]}">
                                <span attrs="{'invisible': [('invalid_addresses', '!=', 0)]}">
                                    <div class="text-right text-muted d-inline-block" name="address">
                                        <span> to: </span>
                                        <field name="partner_id" readonly="1" force_save="1" context="{'show_address': 1, 'address_inline': 1}" options="{'always_reload': True, 'no_quick_create': True}"/>
                                    </div>
                                </span>
                                <span attrs="{'invisible': [('invalid_addresses', '=', 0)]}">
                                    <div class="text-right d-inline-block" attrs="{'invisible': [('composition_mode', '=', 'mass_mail')]}">
                                        <span class="text-danger">
                                            The customer address is not complete.
                                        </span>
                                    </div>
                                </span>
                            </span>
                            <span attrs="{'invisible': [('composition_mode', '!=', 'mass_mail')]}">
                                <span attrs="{'invisible': [('invalid_addresses', '=', 0)]}">
                                    <span class="text-danger">
                                        Some customer addresses are not complete.
                                    </span>
                                    <button type="object" name="invalid_addresses_action" class="btn btn-link" role="button"><field name="invalid_addresses" readonly="1" options="{'digits':[0,0]}"/> invoices</button>
                                </span>
                            </span>
                        </span>
                    </div>
                </xpath>
                <xpath expr="//footer/button[hasclass('send_and_print')]" position='attributes'>
                    <attribute name="attrs">{'invisible': ['|', ('is_print', '=', False), '&amp;', '&amp;', ('is_print', '=', True), ('snailmail_is_letter', '=', False), ('is_email', '=', False)]}</attribute>
                </xpath>
                <xpath expr="//footer/button[hasclass('send')]" position='attributes'>
                    <attribute name="attrs">{'invisible': ['|', ('is_print', '=', True), '&amp;', '&amp;', ('is_print', '=', False), ('snailmail_is_letter', '=', False), ('is_email', '=', False)]}</attribute>
                </xpath>
                <xpath expr="//footer/button[hasclass('print')]" position='attributes'>
                    <attribute name="attrs">{'invisible': ['|', '|', ('is_print', '=', False), ('snailmail_is_letter', '=', True), ('is_email', '=', True)]}</attribute>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import account_invoice_send

```

