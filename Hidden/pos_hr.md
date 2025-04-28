# Odoo Module: pos_hr

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "pos_hr",
    'category': "Hidden",
    'summary': 'Link module between Point of Sale and HR',

    'description': """
This module allows Employees (and not users) to log in to the Point of Sale application using a barcode, a PIN number or both.
The actual till still requires one user but an unlimited number of employees can log on to that till and process sales.
    """,

    'depends': ['point_of_sale', 'hr'],

    'data': [
        'views/pos_config.xml',
        'views/point_of_sale.xml',
        'views/pos_order_view.xml',
        'views/pos_order_report_view.xml',
    ],
    'installable': True,
    'auto_install': True,
    'qweb': ['static/src/xml/pos.xml'],
    'license': 'LGPL-3',
}

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib

from odoo import api, models, _
from odoo.exceptions import UserError

class HrEmployee(models.Model):

    _inherit = 'hr.employee'

    def get_barcodes_and_pin_hashed(self):
        if not self.env.user.has_group('point_of_sale.group_pos_user'):
            return []
        # Apply visibility filters (record rules)
        visible_emp_ids = self.search([('id', 'in', self.ids)])
        employees_data = self.sudo().search_read([('id', 'in', visible_emp_ids.ids)], ['barcode', 'pin'])

        for e in employees_data:
            e['barcode'] = hashlib.sha1(e['barcode'].encode('utf8')).hexdigest() if e['barcode'] else False
            e['pin'] = hashlib.sha1(e['pin'].encode('utf8')).hexdigest() if e['pin'] else False
        return employees_data

    def unlink(self):
        configs_with_employees = self.env['pos.config'].search([('module_pos_hr', '=', 'True')]).filtered(lambda c: c.current_session_id)
        configs_with_all_employees = configs_with_employees.filtered(lambda c: not c.employee_ids)
        configs_with_specific_employees = configs_with_employees.filtered(lambda c: c.employee_ids & self)
        if configs_with_all_employees or configs_with_specific_employees:
            error_msg = _("You cannot delete an employee that may be used in an active PoS session, close the session(s) first: \n")
            for employee in self:
                config_ids = configs_with_all_employees | configs_with_specific_employees.filtered(lambda c: employee in c.employee_ids)
                if config_ids:
                    error_msg += _("Employee: %s - PoS Config(s): %s \n") % (employee.name, ', '.join(config.name for config in config_ids))

            raise UserError(error_msg)
        return super(HrEmployee, self).unlink()

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-

from functools import partial

from odoo import models, fields


class PosConfig(models.Model):
    _inherit = 'pos.config'

    employee_ids = fields.Many2many(
        'hr.employee', string="Employees with access",
        help='If left empty, all employees can log in to the PoS session')

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api


class PosOrder(models.Model):
    _inherit = "pos.order"

    employee_id = fields.Many2one('hr.employee', help="Person who uses the cash register. It can be a reliever, a student or an interim employee.", states={'done': [('readonly', True)], 'invoiced': [('readonly', True)]})
    cashier = fields.Char(string="Cashier", compute="_compute_cashier", store=True)

    @api.model
    def _order_fields(self, ui_order):
        order_fields = super(PosOrder, self)._order_fields(ui_order)
        order_fields['employee_id'] = ui_order.get('employee_id')
        return order_fields

    @api.depends('employee_id', 'user_id')
    def _compute_cashier(self):
        for order in self:
            if order.employee_id:
                order.cashier = order.employee_id.name
            else:
                order.cashier = order.user_id.name

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_config
from . import pos_order
from . import hr_employee

```

## File: report\pos_order_report.py

```python
# -*- coding: utf-8 -*-

from functools import partial

from odoo import models, fields


class PosOrderReport(models.Model):
    _inherit = "report.pos.order"
    employee_id = fields.Many2one(
                'hr.employee', string='Employee', readonly=True)

    def _select(self):
        return super(PosOrderReport, self)._select() + ',s.employee_id AS employee_id'

    def _group_by(self):
        return super(PosOrderReport, self)._group_by() + ',s.employee_id'

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_order_report

```

## File: static\img\login-bg-overlay.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="2000" height="1128" viewBox="0 0 2000 1128">
    <polygon fill-opacity=".03" points="0 1077.844 392.627 778.443 1504.99 1127.745 0 1127.745"/>
    <polygon fill-opacity=".02" points="392.216 778.443 283.294 0 0 0 0 666.504"/>
    <polygon fill-opacity=".03" points="1000 0 2000 1009.98 2000 439.94 1749.817 0"/>
</svg>

```

## File: static\src\js\chrome.js

```javascript
odoo.define('pos_hr.chrome', function (require) {
    "use strict";

    var core = require('web.core');
    var chrome = require('point_of_sale.chrome');
    var _t = core._t;
    var _lt = core._lt;
    var QWeb = core.qweb;


    /* ------- The User Name Widget ------- */

    // Displays the current cashier's name and allows
    // to switch between cashiers.

    chrome.UsernameWidget.include({
        renderElement: function(){
            var self = this;
            this._super();
            this.$el.click(function(){
                self.click_username();
            });
        },
        click_username: function(){
            if(!this.pos.config.module_pos_hr) { return; }
            var self = this;
            this.gui.select_employee({
                'security':     true,
                'current_employee': this.pos.get_cashier(),
                'title':      _t('Change Cashier'),
            }).then(function(employee){
                self.pos.set_cashier(employee);
                self.chrome.widget.username.renderElement();
                self.renderElement();
            });
        },
    });

    var HeaderCloseButtonWidget = chrome.HeaderButtonWidget.extend({
        start: function(){
            if (this.pos.config.module_pos_hr) {
                var self = this;
                this.pos.bind('change:cashier', this._show_hide_close_button , this);
            }
            return this._super();
        },
        _show_hide_close_button: function(){
            if (this.pos.get('cashier').role == 'manager') {
                this.show();
            } else {
                this.hide();
            }
        }
    });

    var HeaderLockButtonWidget = chrome.HeaderButtonWidget.extend({
        init: function(parent, options) {
        this._super(parent, options);
        this.icon = 'fa-unlock';
        this.icon_color = 'green';
        this.icon_mouseover = 'fa-lock';
        this.icon_color_mouseover = 'red';
        },

        start: function(){
            if (this.pos.config.module_pos_hr) {
                this.show();
            } else {
                this.hide();
            }
            return this._super();
        },

        renderElement: function() {
            var self = this;
            this._super();
            this.iconElement = this.$el.find("i");
            this.$el.css('font-size','20px');
            this.iconElement.addClass(this.icon);
            this.$el.css('color',this.icon_color);
            this.$el.mouseover(function(){
                self.iconElement.addClass(self.icon_mouseover).removeClass(self.icon);
                $(this).css('color', self.icon_color_mouseover);
            }).mouseleave(function(){
                self.iconElement.addClass(self.icon).removeClass(self.icon_mouseover);
                $(this).css('color', self.icon_color);
            });
        },
    });
    chrome.Chrome.include({
        lock_button_widget: {
            'name':   'lock_button',
            'widget': HeaderLockButtonWidget,
            'append':  '.pos-rightheader',
            'args': {
                label: _lt('Lock'),
                action: function() {
                    this.chrome.return_to_login_screen();
                }
            }
        },
        build_widgets: function() {
            var self = this;
                this.widgets.some(function(widget, index){
                    if (widget.name === 'close_button'){
                        widget.widget = HeaderCloseButtonWidget;
                        self.widgets.splice(index, 0, self.lock_button_widget);
                            return true;
                    }
                    return false;
                });
            this._super();
        },
        return_to_login_screen: function() {
            this.gui.show_screen('login');
        },

    });

    return {
        HeaderLockButtonWidget: HeaderLockButtonWidget,
        HeaderCloseButtonWidget: HeaderCloseButtonWidget,
    };
});

```

## File: static\src\js\gui.js

```javascript
odoo.define('pos_hr.gui', function (require) {
    "use strict";

var core = require('web.core');
var gui = require('point_of_sale.gui');
var _t = core._t;

gui.Gui.include({
    _show_first_screen: function () {
        if (this.pos.config.module_pos_hr) {
            this.show_screen('login');
        } else {
            this._super();
        }
    },
    select_employee: function(options) {
        options = options || {};
        var self = this;

        var list = [];
        this.pos.employees.forEach(function(employee) {
            if (!options.only_managers || employee.role === 'manager') {
                list.push({
                'label': employee.name,
                'item':  employee,
                });
            }
        });

        var prom = new Promise(function (resolve, reject) {
            self.show_popup('selection', {
                title: options.title || _t('Select User'),
                list: list,
                confirm: resolve,
                cancel: reject,
                is_selected: function (employee) {
                    return employee === self.pos.get_cashier();
                },
            });
        });

        return prom.then(function (employee) {
            return self.ask_password(employee.pin).then(function(){
                return employee;
            });
        });
    },
    // Ask for a password, and checks if it this
    // the same as specified by the function call.
    // Returns a promise that resolves on success,
    // fails on failure.
    ask_password: function(password) {
        var self = this;
        var prom = new Promise(function (resolve, reject) {
            if (password) {
                self.show_popup('password',{
                    'title': _t('Password ?'),
                    confirm: function (pw) {
                        if (Sha1.hash(pw) !== password) {
                            self.show_popup('error', _t('Incorrect Password'));
                            reject();
                        } else {
                            resolve();
                        }
                    },
                });
            } else {
                resolve();
            }
        });
        return prom;
    },
});
});
```

## File: static\src\js\models.js

```javascript
odoo.define('pos_hr.employees', function (require) {
    "use strict";

var models = require('point_of_sale.models');
var rpc = require('web.rpc');

models.load_models([{
    model:  'hr.employee',
    fields: ['name', 'id', 'user_id'],
    domain: function(self){ return [['company_id', '=', self.config.company_id[0]]]; },
    loaded: function(self, employees) {
        if (self.config.module_pos_hr) {
            if (self.config.employee_ids.length > 0) {
                self.employees = employees.filter(function(employee) {
                    return self.config.employee_ids.includes(employee.id) || employee.user_id[0] === self.user.id;
                });
            } else {
                self.employees = employees;
            }
            self.employees.forEach(function(employee) {
                var hasUser = self.users.some(function(user) {
                    if (user.id === employee.user_id[0]) {
                        employee.role = user.role;
                        return true;
                    }
                    return false;
                });
                if (!hasUser) {
                    employee.role = 'cashier';
                }
            });
        }
    }
}]);

var posmodel_super = models.PosModel.prototype;
models.PosModel = models.PosModel.extend({
    load_server_data: function () {
        var self = this;
        return posmodel_super.load_server_data.apply(this, arguments).then(function () {
            var employee_ids = _.map(self.employees, function(employee){return employee.id;});
            var records = rpc.query({
                model: 'hr.employee',
                method: 'get_barcodes_and_pin_hashed',
                args: [employee_ids],
            });
            return records.then(function (employee_data) {
                self.employees.forEach(function (employee) {
                    var data = _.findWhere(employee_data, {'id': employee.id});
                    if (data !== undefined){
                        employee.barcode = data.barcode;
                        employee.pin = data.pin;
                    }
                });
            });
        });
    },
});

});

```

## File: static\src\js\screens.js

```javascript
odoo.define('pos_hr.screens', function (require) {
    "use strict";

var core = require('web.core');
var gui = require('point_of_sale.gui');
var ScreenWidget = require('point_of_sale.screens').ScreenWidget;

var _t = core._t;

ScreenWidget.include({

    // what happens when a cashier id barcode is scanned.
    // the default behavior is the following : 
    // - if there's an employee with a matching barcode, put it as the active 'cashier', go to cashier mode, and return true
    // - else : do nothing and return false. You probably want to extend this to show and appropriate error popup... 
    barcode_cashier_action: function(code){
        var self = this;
        var employees = this.pos.employees;
        var prom;
        for(var i = 0, len = employees.length; i < len; i++){
            if(employees[i].barcode === Sha1.hash(code.code)){
                if (employees[i].id !== this.pos.get_cashier().id && employees[i].pin) {
                    prom =  this.gui.ask_password(employees[i].pin).then(function(){
                        self.pos.set_cashier(employees[i]);
                        self.chrome.widget.username.renderElement();
                        return true;
                    });
                } else {
                    this.pos.set_cashier(employees[i]);
                    this.chrome.widget.username.renderElement();
                    prom = Promise.resolve(true);
                }
                break;
            }
        }
        if (!prom){
            this.barcode_error_action(code);
            return Promise.resolve(false);
        }
        else {
            return prom
        }
    },
    show: function() {
        this._super();
        if (this.gui.get_current_screen() == 'login'){
            this.pos.barcode_reader.save_callbacks();
            this.pos.barcode_reader.reset_action_callbacks();
            this.pos.barcode_reader.set_action_callback('cashier', _.bind(this.barcode_cashier_action, this));
        }
    },
});

/*--------------------------------------*\
 |         THE LOGIN SCREEN           |
\*======================================*/

// The login screen enables employees to log in to the PoS
// at startup or after it was locked, with either barcode, pin, or both.

var LoginScreenWidget = ScreenWidget.extend({
    template: 'LoginScreenWidget',

    /**
     * @override
     */
    show: function() {
        var self = this;
        this.$('.select-employee').click(function() {
            self.gui.select_employee({
                'security': true,
                'current_employee': self.pos.get_cashier(),
                'title':_t('Change Cashier'),})
            .then(function(employee){
                self.pos.set_cashier(employee);
                self.chrome.widget.username.renderElement();
                self.unlock_screen();
            });
        });

        this.$('.close-session').click(function() {
            self.gui.close();
        });

        this._super();
    },

    /**
     * @override
     */
    barcode_cashier_action: function(code) {
        var self = this;
        return this._super(code).then(function (unlock) {
            if (unlock) {
                self.unlock_screen();
            }
        });
    },

    unlock_screen: function() {
        this.pos.barcode_reader.restore_callbacks();
        var screen = (this.gui.pos.get_order() ? this.gui.pos.get_order().get_screen_data('previous-screen') : this.gui.startup_screen) || this.gui.startup_screen;
        this.gui.show_screen(screen);
    }
});

gui.define_screen({name:'login', widget: LoginScreenWidget});

return {
    LoginScreenWidget: LoginScreenWidget
};
});

```

## File: static\src\xml\pos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="LoginScreenWidget">
        <div class="login-overlay">
            <div class="screen-login">
                <div class="login-title"><small>Log in to </small><t t-esc="widget.pos.config.name"/></div>
                <div class="login-body">
                    <span class="login-element">
                        <img class="login-barcode-img" src="/point_of_sale/static/img/barcode.png"/>
                        <div class="login-barcode-text">Scan your badge</div>
                    </span>
                    <span class="login-or">or</span>
                    <span class="login-element">
                        <button class="login-button select-employee">Select Cashier</button>
                    </span>
                </div>
                 <div class="login-footer">
                     <small>
                         <button class="login-button close-session">Close session</button>
                     </small>
                 </div>
            </div>
        </div>
    </t>
</templates>

```

## File: views\point_of_sale.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/pos_hr/static/src/css/pos.css"/>

            <script type="text/javascript" src="/pos_hr/static/src/js/models.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/screens.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/gui.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/chrome.js"></script>
        </xpath>
    </template>
</odoo>


```

## File: views\pos_config.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_config_form_view_inherit" model="ir.ui.view">
        <field name="name">pos.config.form.view.inherit</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='btn_use_employees']" position='replace'>
                <div attrs="{'invisible': [('module_pos_hr', '=', False)]}">
                    <span class="o_form_label oe_edit_only">Allowed Employees </span>
                    <field name="employee_ids" widget="many2many_tags" 
                    domain="[('company_id', '=', company_id)]"
                    />
                </div>
            </xpath>
            <xpath expr="//div[@id='pricing']" position='inside'>
                <div class="col-12 col-lg-6 o_setting_box price_control" title="Only users with Manager access rights for PoS app can modify the product prices on orders.">
                    <div class="o_setting_left_pane">
                        <field name="restrict_price_control"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="restrict_price_control" string="Price Control"/>
                        <div class="text-muted">
                            Restrict price modification to managers
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_order_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_report_pos_order_search_inherit" model="ir.ui.view">
        <field name="name">report.pos.order.search.inherit</field>
        <field name="model">report.pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_report_pos_order_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='User']" position='after'>
                <filter string="Employee" name="employee_id" domain="[]" context="{'group_by':'employee_id'}"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\pos_order_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_order_form_inherit" model="ir.ui.view">
        <field name="name">pos.order.form.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position='replace'>
                <field string='Cashier' name="employee_id" readonly="1" attrs="{'invisible': [('employee_id','=',False)]}"/>
                <field string='Cashier' name="user_id" readonly="1" attrs="{'invisible': [('employee_id','!=',False)]}"/>
            </xpath>
        </field>
    </record>

    <record id="pos_order_list_select_inherit" model="ir.ui.view">
        <field name="name">pos.order.list.select.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_order_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="replace">
                <field name="cashier"/>
            </xpath>
            <xpath expr="//filter[@name='user_id']" position="replace">
                <filter string="Cashier" name="by_cashier" domain="[]" context="{'group_by': 'cashier'}"/>
            </xpath>
        </field>
    </record>

    <record id="view_pos_order_tree_inherit" model="ir.ui.view">
        <field name="name">pos.order.tree.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_order_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="replace">
                <field name="cashier"/>
            </xpath>
        </field>
    </record>
</odoo>

```

