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
    'qweb': [
        'static/src/xml/HeaderLockButton.xml',
        'static/src/xml/Chrome.xml',
        'static/src/xml/CashierName.xml',
        'static/src/xml/LoginScreen.xml',
    ],
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

    def _export_for_ui(self, order):
        result = super(PosOrder, self)._export_for_ui(order)
        result.update({
            'employee_id': order.employee_id.id,
        })
        return result

    def _get_fields_for_draft_order(self):
        fields = super(PosOrder, self)._get_fields_for_draft_order()
        fields.append('employee_id')
        return fields

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

## File: static\src\js\CashierName.js

```javascript
odoo.define('pos_hr.CashierName', function (require) {
    'use strict';

    const CashierName = require('point_of_sale.CashierName');
    const Registries = require('point_of_sale.Registries');
    const useSelectEmployee = require('pos_hr.useSelectEmployee');
    const { useBarcodeReader } = require('point_of_sale.custom_hooks');

    const PosHrCashierName = (CashierName) =>
        class extends CashierName {
            constructor() {
                super(...arguments);
                const { selectEmployee, askPin } = useSelectEmployee();
                this.askPin = askPin;
                this.selectEmployee = selectEmployee;
                useBarcodeReader({ cashier: this._onCashierScan });
            }
            mounted() {
                this.env.pos.on('change:cashier', this.render, this);
            }
            willUnmount() {
                this.env.pos.off('change:cashier', null, this);
            }
            async selectCashier() {
                if (!this.env.pos.config.module_pos_hr) return;

                const list = this.env.pos.employees
                    .filter((employee) => employee.id !== this.env.pos.get_cashier().id)
                    .map((employee) => {
                        return {
                            id: employee.id,
                            item: employee,
                            label: employee.name,
                            isSelected: false,
                        };
                    });

                const employee = await this.selectEmployee(list);
                if (employee) {
                    this.env.pos.set_cashier(employee);
                }
            }
            async _onCashierScan(code) {
                const employee = this.env.pos.employees.find(
                    (emp) => emp.barcode === Sha1.hash(code.code)
                );

                if (!employee || employee === this.env.pos.get_cashier()) return;

                if (!employee.pin || (await this.askPin(employee))) {
                    this.env.pos.set_cashier(employee);
                }
            }
        };

    Registries.Component.extend(CashierName, PosHrCashierName);

    return CashierName;
});

```

## File: static\src\js\Chrome.js

```javascript
odoo.define('pos_hr.chrome', function (require) {
    'use strict';

    const Chrome = require('point_of_sale.Chrome');
    const Registries = require('point_of_sale.Registries');

    const PosHrChrome = (Chrome) =>
        class extends Chrome {
            async start() {
                await super.start();
                this.env.pos.on('change:cashier', this.render, this);
                if (this.env.pos.config.module_pos_hr) this.showTempScreen('LoginScreen');
            }
            get headerButtonIsShown() {
                return !this.env.pos.config.module_pos_hr || this.env.pos.get('cashier').role == 'manager';
            }
        };

    Registries.Component.extend(Chrome, PosHrChrome);

    return Chrome;
});

```

## File: static\src\js\HeaderLockButton.js

```javascript
odoo.define('point_of_sale.HeaderLockButton', function(require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const { useState } = owl;

    class HeaderLockButton extends PosComponent {
        constructor() {
            super(...arguments);
            this.state = useState({ isUnlockIcon: true, title: 'Unlocked' });
        }
        async showLoginScreen() {
            await this.showTempScreen('LoginScreen');
        }
        onMouseOver(isMouseOver) {
            this.state.isUnlockIcon = !isMouseOver;
            this.state.title = isMouseOver ? 'Lock' : 'Unlocked';
        }
    }

    Registries.Component.add(HeaderLockButton);

    return HeaderLockButton;
});

```

## File: static\src\js\LoginScreen.js

```javascript
odoo.define('pos_hr.LoginScreen', function (require) {
    'use strict';

    const PosComponent = require('point_of_sale.PosComponent');
    const Registries = require('point_of_sale.Registries');
    const useSelectEmployee = require('pos_hr.useSelectEmployee');
    const { useBarcodeReader } = require('point_of_sale.custom_hooks');

    class LoginScreen extends PosComponent {
        constructor() {
            super(...arguments);
            const { selectEmployee, askPin } = useSelectEmployee();
            this.selectEmployee = selectEmployee;
            this.askPin = askPin;
            useBarcodeReader(
                {
                    cashier: this._barcodeCashierAction,
                },
                true
            );
        }
        back() {
            this.props.resolve({ confirmed: false, payload: false });
            this.trigger('close-temp-screen');
        }
        confirm() {
            this.props.resolve({ confirmed: true, payload: true });
            this.trigger('close-temp-screen');
        }
        get shopName() {
            return this.env.pos.config.name;
        }
        closeSession() {
            this.trigger('close-pos');
        }
        async selectCashier() {
            const list = this.env.pos.employees.map((employee) => {
                return {
                    id: employee.id,
                    item: employee,
                    label: employee.name,
                    isSelected: false,
                };
            });

            const employee = await this.selectEmployee(list);
            if (employee) {
                this.env.pos.set_cashier(employee);
                this.back();
            }
        }
        async _barcodeCashierAction(code) {
            let theEmployee;
            for (let employee of this.env.pos.employees) {
                if (employee.barcode === Sha1.hash(code.code)) {
                    theEmployee = employee;
                    break;
                }
            }

            if (!theEmployee) return;

            if (!theEmployee.pin || (await this.askPin(theEmployee))) {
                this.env.pos.set_cashier(theEmployee);
                this.back();
            }
        }
    }
    LoginScreen.template = 'LoginScreen';

    Registries.Component.add(LoginScreen);

    return LoginScreen;
});

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_hr.employees', function (require) {
    "use strict";

var models = require('point_of_sale.models');

models.load_models([{
    model:  'hr.employee',
    fields: ['name', 'id', 'user_id'],
    domain: function(self){
        return self.config.employee_ids.length > 0
            ? [
                  '&',
                  ['company_id', '=', self.config.company_id[0]],
                  '|',
                  ['user_id', '=', self.user.id],
                  ['id', 'in', self.config.employee_ids],
              ]
            : [['company_id', '=', self.config.company_id[0]]];
    },
    loaded: function(self, employees) {
        if (self.config.module_pos_hr) {
            self.employees = employees;
            self.employee_by_id = {};
            self.employees.forEach(function(employee) {
                self.employee_by_id[employee.id] = employee;
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
            var records = self.rpc({
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
    set_cashier: function(employee) {
        posmodel_super.set_cashier.apply(this, arguments);
        const selectedOrder = this.get_order();
        if (selectedOrder && !selectedOrder.get_orderlines().length) {
            // Order without lines can be considered to be un-owned by any employee.
            // We set the employee on that order to the currently set employee.
            selectedOrder.employee = employee;
        }
    }
});

var super_order_model = models.Order.prototype;
models.Order = models.Order.extend({
    initialize: function (attributes, options) {
        super_order_model.initialize.apply(this, arguments);
        if (!options.json) {
            this.employee = this.pos.get_cashier();
        }
    },
    init_from_JSON: function (json) {
        super_order_model.init_from_JSON.apply(this, arguments);
        if (this.pos.config.module_pos_hr && json.employee_id) {
            this.employee = this.pos.employee_by_id[json.employee_id];
        }
    },
    export_as_JSON: function () {
        const json = super_order_model.export_as_JSON.apply(this, arguments);
        if (this.pos.config.module_pos_hr) {
            json.employee_id = this.employee ? this.employee.id : false;
        }
        return json;
    },
});

});

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('pos_hr.PaymentScreen', function (require) {
    'use strict';

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');

    const PosHrPaymentScreen = (PaymentScreen_) =>
          class extends PaymentScreen_ {
              async _finalizeValidation() {
                  this.currentOrder.employee = this.env.pos.get_cashier();
                  await super._finalizeValidation();
              }
          };

    Registries.Component.extend(PaymentScreen, PosHrPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\useSelectEmployee.js

```javascript
odoo.define('pos_hr.useSelectEmployee', function (require) {
    'use strict';

    const { Component } = owl;

    function useSelectEmployee() {
        const current = Component.current;

        async function askPin(employee) {
            const { confirmed, payload: inputPin } = await this.showPopup('NumberPopup', {
                isPassword: true,
                title: this.env._t('Password ?'),
                startingValue: null,
            });

            if (!confirmed) return false;

            if (employee.pin === Sha1.hash(inputPin)) {
                return employee;
            } else {
                await this.showPopup('ErrorPopup', {
                    title: this.env._t('Incorrect Password'),
                });
                return false;
            }
        }

        async function selectEmployee(selectionList) {
            const { confirmed, payload: employee } = await this.showPopup('SelectionPopup', {
                title: this.env._t('Change Cashier'),
                list: selectionList,
            });

            if (!confirmed) return false;

            if (!employee.pin) {
                return employee;
            }

            return await askPin.call(current, employee);
        }
        return { askPin: askPin.bind(current), selectEmployee: selectEmployee.bind(current) };
    }

    return useSelectEmployee;
});

```

## File: static\src\xml\CashierName.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="CashierName" t-inherit="point_of_sale.CashierName" t-inherit-mode="extension" owl="1">
        <xpath expr="//div" position="attributes">
            <attribute name="t-on-click">selectCashier</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\Chrome.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="Chrome" t-inherit="point_of_sale.Chrome" t-inherit-mode="extension" owl="1">
        <xpath expr="//HeaderButton" position="replace">
            <HeaderLockButton t-if="env.pos.config.module_pos_hr" />
            <HeaderButton t-if="headerButtonIsShown" />
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\HeaderLockButton.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="HeaderLockButton" owl="1">
        <div class="header-button lock-button" t-on-mouseover="onMouseOver(true)"
             t-on-click="showLoginScreen" t-on-mouseout="onMouseOver(false)">
            <span class="lock-button">
                <i class="fa"
                   t-att-class="{ 'fa-unlock': state.isUnlockIcon, 'fa-lock': !state.isUnlockIcon }"
                   role="img" t-att-aria-label="state.title" t-att-title="state.title"></i>
            </span>
        </div>
    </t>

</templates>

```

## File: static\src\xml\LoginScreen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="LoginScreen" owl="1">
        <div class="login-overlay">
            <div class="screen-login">
                <div class="login-title"><small>Log in to </small>
                    <t t-esc="shopName" />
                </div>
                <div class="login-body">
                    <span class="login-element">
                        <img class="login-barcode-img"
                             src="/point_of_sale/static/img/barcode.png" />
                        <div class="login-barcode-text">Scan your badge</div>
                    </span>
                    <span class="login-or">or</span>
                    <span class="login-element">
                        <button class="login-button select-employee"
                                t-on-click="selectCashier">Select Cashier</button>
                    </span>
                </div>
                <div class="login-footer">
                    <small>
                        <button class="login-button close-session" t-on-click="closeSession">Close session</button>
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
            <script type="text/javascript" src="/pos_hr/static/src/js/useSelectEmployee.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/Chrome.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/HeaderLockButton.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/CashierName.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/LoginScreen.js"></script>
            <script type="text/javascript" src="/pos_hr/static/src/js/PaymentScreen.js"></script>
        </xpath>
    </template>

    <template id="assets_tests" name="Pos Hr Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/pos_hr/static/tests/tours/PosHrTourMethods.js"></script>
            <script type="text/javascript" src="/pos_hr/static/tests/tours/PosHrTour.js"></script>
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
                <field name="employee_id" widget="many2one_avatar_employee"/>
            </xpath>
        </field>
    </record>
</odoo>

```

