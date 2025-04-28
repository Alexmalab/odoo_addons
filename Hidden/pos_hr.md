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
        'views/pos_order_view.xml',
        'views/pos_order_report_view.xml',
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_hr/static/src/css/pos.css',
            'pos_hr/static/src/js/models.js',
            'pos_hr/static/src/js/SelectCashierMixin.js',
            'pos_hr/static/src/js/Chrome.js',
            'pos_hr/static/src/js/HeaderLockButton.js',
            'pos_hr/static/src/js/CashierName.js',
            'pos_hr/static/src/js/LoginScreen.js',
            'pos_hr/static/src/js/PaymentScreen.js',
            'pos_hr/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'pos_hr/static/tests/**/*',
        ],
    },
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

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active_pos_session(self):
        configs_with_employees = self.env['pos.config'].sudo().search([('module_pos_hr', '=', 'True')]).filtered(lambda c: c.current_session_id)
        configs_with_all_employees = configs_with_employees.filtered(lambda c: not c.employee_ids)
        configs_with_specific_employees = configs_with_employees.filtered(lambda c: c.employee_ids & self)
        if configs_with_all_employees or configs_with_specific_employees:
            error_msg = _("You cannot delete an employee that may be used in an active PoS session, close the session(s) first: \n")
            for employee in self:
                config_ids = configs_with_all_employees | configs_with_specific_employees.filtered(lambda c: employee in c.employee_ids)
                if config_ids:
                    error_msg += _("Employee: %s - PoS Config(s): %s \n") % (employee.name, ', '.join(config.name for config in config_ids))

            raise UserError(error_msg)

```

## File: models\hr_employee_public.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class HrEmployeePublic(models.Model):
    _inherit = "hr.employee.public"

    def read(self, fields=None, load='_classic_read'):
        # as `pos_blackbox_be` is a certified module, it's hard to make fixes in it
        # so this is a workaround to remove `insz_or_bis_number` field from the fields list
        # as the parent hr.employee model will attempt to read it from hr.employee.public
        # where it doesn't exist
        if fields and 'insz_or_bis_number' in fields:
            pos_blackbox_be_installed = self.env['ir.module.module'].sudo().search_count([('name', '=', 'pos_blackbox_be'), ('state', '=', 'installed')])
            has_hr_user_group = self.env.user.has_group('hr.group_hr_user')
            if pos_blackbox_be_installed and not has_hr_user_group:
                fields.remove('insz_or_bis_number')

        return super().read(fields=fields, load=load)

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

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _pos_data_process(self, loaded_data):
        super()._pos_data_process(loaded_data)
        if self.config_id.module_pos_hr:
            loaded_data['employee_by_id'] = {employee['id']: employee for employee in loaded_data['hr.employee']}

    def _pos_ui_models_to_load(self):
        result = super()._pos_ui_models_to_load()
        if self.config_id.module_pos_hr:
            new_model = 'hr.employee'
            if new_model not in result:
                result.append(new_model)
        return result

    def _loader_params_hr_employee(self):
        if len(self.config_id.employee_ids) > 0:
            domain = ['&', ('company_id', '=', self.config_id.company_id.id), '|', ('user_id', '=', self.user_id.id), ('id', 'in', self.config_id.employee_ids.ids)]
        else:
            domain = [('company_id', '=', self.config_id.company_id.id)]
        return {'search_params': {'domain': domain, 'fields': ['name', 'id', 'user_id'], 'load': False}}

    def _get_pos_ui_hr_employee(self, params):
        employees = self.env['hr.employee'].search_read(**params['search_params'])
        employee_ids = [employee['id'] for employee in employees]
        user_ids = [employee['user_id'] for employee in employees if employee['user_id']]
        manager_ids = self.env['res.users'].browse(user_ids).filtered(lambda user: self.config_id.group_pos_manager_id in user.groups_id).mapped('id')

        employees_barcode_pin = self.env['hr.employee'].browse(employee_ids).get_barcodes_and_pin_hashed()
        bp_per_employee_id = {bp_e['id']: bp_e for bp_e in employees_barcode_pin}
        for employee in employees:
            employee['role'] = 'manager' if employee['user_id'] and employee['user_id'] in manager_ids else 'cashier'
            employee['barcode'] = bp_per_employee_id[employee['id']]['barcode']
            employee['pin'] = bp_per_employee_id[employee['id']]['pin']

        return employees

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # pos.config fields
    pos_employee_ids = fields.Many2many(related='pos_config_id.employee_ids', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_config
from . import pos_order
from . import hr_employee
from . import hr_employee_public
from . import pos_session
from . import res_config_settings

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
    const SelectCashierMixin = require('pos_hr.SelectCashierMixin');
    const { useBarcodeReader } = require('point_of_sale.custom_hooks');

    const PosHrCashierName = (CashierName) =>
        class extends SelectCashierMixin(CashierName) {
            setup() {
                super.setup();
                useBarcodeReader({ cashier: this.barcodeCashierAction });
            }
            //@Override
            get avatar() {
                if (this.env.pos.config.module_pos_hr) {
                    const cashier = this.env.pos.get_cashier();
                    if (!(cashier && cashier.id)) {
                        return '';
                    }
                    return `/web/image/hr.employee.public/${cashier.id}/avatar_128`;
                }
                return super.avatar;
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
                if (this.env.pos.config.module_pos_hr) this.showTempScreen('LoginScreen');
            }
            get headerButtonIsShown() {
                return !this.env.pos.config.module_pos_hr || this.env.pos.get_cashier().role == 'manager' || this.env.pos.get_cashier_user_id() === this.env.pos.user.id;
            }
            showCashMoveButton() {
                return super.showCashMoveButton() && (!this.env.pos.cashier || this.env.pos.cashier.role == 'manager');
            }
            shouldShowCashControl() {
                if (this.env.pos.config.module_pos_hr){
                    return super.shouldShowCashControl() && this.env.pos.hasLoggedIn;
                }
                return super.shouldShowCashControl();
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
        setup() {
            super.setup();
            this.state = useState({ isUnlockIcon: true, title: 'Unlocked' });
        }
        async showLoginScreen() {
            this.env.pos.reset_cashier();
            await this.showTempScreen('LoginScreen');
        }
        onMouseOver(isMouseOver) {
            this.state.isUnlockIcon = !isMouseOver;
            this.state.title = isMouseOver ? 'Lock' : 'Unlocked';
        }
    }
    HeaderLockButton.template = "HeaderLockButton";

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
    const SelectCashierMixin = require('pos_hr.SelectCashierMixin');
    const { useBarcodeReader } = require('point_of_sale.custom_hooks');

    class LoginScreen extends SelectCashierMixin(PosComponent) {
        setup() {
            super.setup();
            useBarcodeReader({cashier: this.barcodeCashierAction}, true);
        }
        async selectCashier() {
            if (await super.selectCashier()) {
                this.back();
            }
        }
        async barcodeCashierAction(code) {
            if (await super.barcodeCashierAction(code) && this.env.pos.get_cashier().id) {
                this.back();
            }
        }
        back() {
            this.props.resolve({ confirmed: false, payload: false });
            this.trigger('close-temp-screen');
            this.env.pos.hasLoggedIn = true;
            this.env.posbus.trigger('start-cash-control');
        }
        confirm() {
            this.props.resolve({ confirmed: true, payload: true });
            this.trigger('close-temp-screen');
        }
        get shopName() {
            return this.env.pos.config.name;
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

var { PosGlobalState, Order } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');


const PosHrPosGlobalState = (PosGlobalState) => class PosHrPosGlobalState extends PosGlobalState {
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.config.module_pos_hr) {
            this.employees = loadedData['hr.employee'];
            this.employee_by_id = loadedData['employee_by_id'];
            this.reset_cashier();
        }
    }
    async after_load_server_data() {
        await super.after_load_server_data(...arguments);
        if (this.config.module_pos_hr) {
            this.hasLoggedIn = !this.config.module_pos_hr;
        }
    }
    reset_cashier() {
        this.cashier = {name: null, id: null, barcode: null, user_id: null, pin: null, role: null};
    }
    set_cashier(employee) {
        this.cashier = employee;
        const selectedOrder = this.get_order();
        if (selectedOrder && !selectedOrder.get_orderlines().length) {
            // Order without lines can be considered to be un-owned by any employee.
            // We set the cashier on that order to the currently set employee.
            selectedOrder.cashier = employee;
        }
        if (!this.cashierHasPriceControlRights() && this.numpadMode === 'price') {
            this.numpadMode = 'quantity';
        }
    }

    /**{name: null, id: null, barcode: null, user_id:null, pin:null}
     * If pos_hr is activated, return {name: string, id: int, barcode: string, pin: string, user_id: int}
     * @returns {null|*}
     */
    get_cashier() {
        if (this.config.module_pos_hr) {
            return this.cashier;
        }
        return super.get_cashier();
    }
    get_cashier_user_id() {
        if (this.config.module_pos_hr) {
            return this.cashier.user_id ? this.cashier.user_id : null;
        }
        return super.get_cashier_user_id();
    }
}
Registries.Model.extend(PosGlobalState, PosHrPosGlobalState);


const PosHrOrder = (Order) => class PosHrOrder extends Order {
    constructor(obj, options) {
        super(...arguments);
        if (!options.json && this.pos.config.module_pos_hr) {
            this.cashier = this.pos.get_cashier();
        }
    }
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        if (this.pos.config.module_pos_hr && json.employee_id) {
            this.cashier = this.pos.employee_by_id[json.employee_id];
        }
    }
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        if (this.pos.config.module_pos_hr) {
            json.employee_id = this.cashier ? this.cashier.id : false;
        }
        return json;
    }
}
Registries.Model.extend(Order, PosHrOrder);

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
                  this.currentOrder.cashier = this.env.pos.get_cashier();
                  await super._finalizeValidation();
              }
          };

    Registries.Component.extend(PaymentScreen, PosHrPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\SelectCashierMixin.js

```javascript
/* global Sha1 */
odoo.define('pos_hr.SelectCashierMixin', function (require) {
    'use strict';

    const SelectCashierMixin = (PosComponent) => class ComponentWithSelectCashierMixin extends PosComponent {
        async askPin(employee) {
            const { confirmed, payload: inputPin } = await this.showPopup('NumberPopup', {
                isPassword: true,
                title: this.env._t('Password ?'),
                startingValue: null,
            });

            if (!confirmed) return;

            if (employee.pin === Sha1.hash(inputPin)) {
                return employee;
            } else {
                await this.showPopup('ErrorPopup', {
                    title: this.env._t('Incorrect Password'),
                });
                return;
            }
        }

        /**
         * Select a cashier, the returning value will either be an object or nothing (undefined)
         */
        async selectCashier() {
            if (this.env.pos.config.module_pos_hr) {
                const employeesList = this.env.pos.employees
                    .filter((employee) => employee.id !== this.env.pos.get_cashier().id)
                    .map((employee) => {
                        return {
                            id: employee.id,
                            item: employee,
                            label: employee.name,
                            isSelected: false,
                        };
                    });
                let {confirmed, payload: employee} = await this.showPopup('SelectionPopup', {
                    title: this.env._t('Change Cashier'),
                    list: employeesList,
                });

                if (!confirmed) {
                    return;
                }

                if (employee && employee.pin) {
                    employee = await this.askPin(employee);
                }
                if (employee) {
                    this.env.pos.set_cashier(employee);
                }
                return employee;
            }
        }

        async barcodeCashierAction(code) {
            const employee = this.env.pos.employees.find(
                (emp) => emp.barcode === Sha1.hash(code.code)
            );
            if (employee && employee !== this.env.pos.get_cashier() && (!employee.pin || (await this.askPin(employee)))) {
                this.env.pos.set_cashier(employee);
            }
            return employee;
        }
    }

    return SelectCashierMixin;
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
        <div class="header-button lock-button" t-on-mouseover="() => this.onMouseOver(true)"
             t-on-click="showLoginScreen" t-on-mouseout="() => this.onMouseOver(false)">
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
                        <button class="login-button select-cashier"
                                t-on-click="selectCashier">Select Cashier</button>
                    </span>
                </div>
            </div>
        </div>
    </t>
</templates>

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
            <xpath expr="//div[@id='warning_text_employees']" position='replace'>
                <field name="company_id" invisible="1" />
                <div class="row">
                    <label for="employee_ids" string="Allowed Employees" class="col-lg-3 o_light_label" />
                    <field name="employee_ids" widget="many2many_tags" domain="[('company_id', '=', company_id)]" />
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

   <record id="report_pos_order_view_tree" model="ir.ui.view">
        <field name="name">report.pos.order.view.tree.inherit.pos.hr</field>
        <field name="model">report.pos.order</field>
        <field name="inherit_id" ref="point_of_sale.report_pos_order_view_tree"/>
        <field name="arch" type="xml">
            <field name="product_categ_id" position="after">
                <field name="employee_id" widget="many2one_avatar_employee"/>
            </field>
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

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_hr</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='warning_text_employees']" position='replace'>
                <div class="row">
                    <label for="pos_employee_ids" string="Allowed Employees" class="col-lg-3 o_light_label" />
                    <field name="pos_employee_ids" widget="many2many_tags" domain="[('company_id', '=', company_id)]" />
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

