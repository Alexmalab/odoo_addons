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
    'name': "POS - HR",
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
        'point_of_sale._assets_pos': [
            'pos_hr/static/src/**/*',
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
        configs_with_all_employees = configs_with_employees.filtered(lambda c: not c.basic_employee_ids and not c.advanced_employee_ids)
        configs_with_specific_employees = configs_with_employees.filtered(lambda c: (c.basic_employee_ids or c.advanced_employee_ids) & self)
        if configs_with_all_employees or configs_with_specific_employees:
            error_msg = _("You cannot delete an employee that may be used in an active PoS session, close the session(s) first: \n")
            for employee in self:
                config_ids = configs_with_all_employees | configs_with_specific_employees.filtered(lambda c: employee in c.basic_employee_ids)
                if config_ids:
                    error_msg += _("Employee: %s - PoS Config(s): %s \n", employee.name, ', '.join(config.name for config in config_ids))

            raise UserError(error_msg)

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api
from odoo.osv.expression import AND


class PosConfig(models.Model):
    _inherit = 'pos.config'

    basic_employee_ids = fields.Many2many(
        'hr.employee', 'pos_hr_basic_employee_hr_employee', string="Employees with basic access",
        help='If left empty, all employees can log in to PoS')
    advanced_employee_ids = fields.Many2many(
        'hr.employee', 'pos_hr_advanced_employee_hr_employee', string="Employees with manager access",
        help='If left empty, only Odoo users have extended rights in PoS')

    @api.onchange('basic_employee_ids')
    def _onchange_basic_employee_ids(self):
        for employee in self.basic_employee_ids:
            if employee in self.advanced_employee_ids:
                self.advanced_employee_ids -= employee

    @api.onchange('advanced_employee_ids')
    def _onchange_advanced_employee_ids(self):
        for employee in self.advanced_employee_ids:
            if employee in self.basic_employee_ids:
                self.basic_employee_ids -= employee

    def _employee_domain(self, user_id):
        domain = self._check_company_domain(self.company_id)
        if len(self.basic_employee_ids) > 0:
            domain = AND([
                domain,
                ['|', ('user_id', '=', user_id), ('id', 'in', self.basic_employee_ids.ids + self.advanced_employee_ids.ids)]
            ])
        return domain

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api


class PosOrder(models.Model):
    _inherit = "pos.order"

    employee_id = fields.Many2one('hr.employee', help="Person who uses the cash register. It can be a reliever, a student or an interim employee.")
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
        domain = self.config_id._employee_domain(self.user_id.id)
        return {'search_params': {'domain': domain, 'fields': ['name', 'id', 'user_id', 'work_contact_id'], 'load': False}}

    def _get_pos_ui_hr_employee(self, params):
        employees = self.env['hr.employee'].search_read(**params['search_params'])
        employee_ids = [employee['id'] for employee in employees]
        user_ids = [employee['user_id'] for employee in employees if employee['user_id']]
        manager_ids = self.env['res.users'].browse(user_ids).filtered(lambda user: self.config_id.group_pos_manager_id in user.groups_id).mapped('id')

        employees_barcode_pin = self.env['hr.employee'].browse(employee_ids).get_barcodes_and_pin_hashed()
        bp_per_employee_id = {bp_e['id']: bp_e for bp_e in employees_barcode_pin}
        for employee in employees:
            if employee['user_id'] and employee['user_id'] in manager_ids or employee['id'] in self.config_id.advanced_employee_ids.ids:
                employee['role'] = 'manager'
            else:
                employee['role'] = 'cashier'
            employee['barcode'] = bp_per_employee_id[employee['id']]['barcode']
            employee['pin'] = bp_per_employee_id[employee['id']]['pin']

        return employees

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    # pos.config fields
    pos_basic_employee_ids = fields.Many2many(related='pos_config_id.basic_employee_ids', readonly=False,
        help='If left empty, all employees can log in to PoS')
    pos_advanced_employee_ids = fields.Many2many(related='pos_config_id.advanced_employee_ids', readonly=False,
        help='If left empty, only Odoo users have extended rights in PoS')

    @api.onchange('pos_basic_employee_ids')
    def _onchange_basic_employee_ids(self):
        for employee in self.pos_basic_employee_ids:
            if employee in self.pos_advanced_employee_ids:
                self.pos_advanced_employee_ids -= employee

    @api.onchange('pos_advanced_employee_ids')
    def _onchange_advanced_employee_ids(self):
        for employee in self.pos_advanced_employee_ids:
            if employee in self.pos_basic_employee_ids:
                self.pos_basic_employee_ids -= employee

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_config
from . import pos_order
from . import hr_employee
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

## File: static\src\app\select_cashier_mixin.js

```javascript
/** @odoo-module */
/* global Sha1 */

import { _t } from "@web/core/l10n/translation";

import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { useService } from "@web/core/utils/hooks";
import { useBarcodeReader } from "@point_of_sale/app/barcode/barcode_reader_hook";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export function useCashierSelector(
    { onCashierChanged, exclusive } = { onCashierChanged: () => {}, exclusive: false }
) {
    const popup = useService("popup");
    const pos = usePos();
    useBarcodeReader(
        {
            async cashier(code) {
                const employee = pos.employees.find((emp) => emp.barcode === Sha1.hash(code.code));
                if (
                    employee &&
                    employee !== pos.get_cashier() &&
                    (!employee.pin || (await checkPin(employee)))
                ) {
                    pos.set_cashier(employee);
                    if (onCashierChanged) {
                        onCashierChanged();
                    }
                }
                return employee;
            },
        },
        exclusive
    );

    async function checkPin(employee) {
        const { confirmed, payload: inputPin } = await popup.add(NumberPopup, {
            isPassword: true,
            title: _t("Password?"),
        });

        if (!confirmed) {
            return false;
        }

        if (employee.pin !== Sha1.hash(inputPin)) {
            await popup.add(ErrorPopup, {
                title: _t("Incorrect Password"),
                body: _t("Please try again."),
            });
            return false;
        }
        return true;
    }

    /**
     * Select a cashier, the returning value will either be an object or nothing (undefined)
     */
    return async function selectCashier() {
        if (pos.config.module_pos_hr) {
            const employeesList = pos.employees
                .filter((employee) => employee.id !== pos.get_cashier().id)
                .map((employee) => {
                    return {
                        id: employee.id,
                        item: employee,
                        label: employee.name,
                        isSelected: false,
                    };
                });
            if (!employeesList.length) {
                if (!pos.get_cashier().id) {
                    await popup.add(ErrorPopup, {
                        title: _t("No Cashiers"),
                        body: _t("There are no employees to select as cashier. Please create one."),
                    });
                    this.pos.redirectToBackend();
                }
                return
            }
            const { confirmed, payload: employee } = await popup.add(SelectionPopup, {
                title: _t("Change Cashier"),
                list: employeesList,
            });

            if (!confirmed || !employee || (employee.pin && !(await checkPin(employee)))) {
                return;
            }

            pos.set_cashier(employee);
            if (onCashierChanged) {
                onCashierChanged();
            }
        }
    };
}

```

## File: static\src\app\login_screen\login_screen.js

```javascript
/** @odoo-module */

import { useCashierSelector } from "@pos_hr/app/select_cashier_mixin";
import { registry } from "@web/core/registry";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component } from "@odoo/owl";

export class LoginScreen extends Component {
    static template = "pos_hr.LoginScreen";
    setup() {
        super.setup(...arguments);
        this.cashierSelector = useCashierSelector({
            onCashierChanged: () => this.back(),
            exclusive: true, // takes exclusive control on the barcode reader
        });
        this.pos = usePos();
    }

    back() {
        this.props.resolve({ confirmed: false, payload: false });
        this.pos.closeTempScreen();
        this.pos.hasLoggedIn = true;
        this.pos.openCashControl();
    }

    get shopName() {
        return this.pos.config.name;
    }

    async selectCashier() {
        return await this.cashierSelector();
    }
}

registry.category("pos_screens").add("LoginScreen", LoginScreen);

```

## File: static\src\app\login_screen\login_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_hr.LoginScreen">
        <div class="login-overlay fixed-top w-100 h-100 bg-view">
            <div class="screen-login position-absolute top-0 start-0 bottom-0 end-0 d-flex flex-column py-4 m-auto w-100 rounded bg-view text-center fw-bolder">
                <div class="login-title fs-2 mb-3 mb-lg-0">Log in to
                    <span class="text-primary" t-esc="shopName" />
                </div>
                <div class="login-body d-flex d-flex flex-column flex-sm-row align-items-center justify-content-around px-3 py-4">
                    <span class="login-element border rounded">
                        <img class="login-barcode-img img-fluid"
                             src="/point_of_sale/static/img/barcode.png" />
                        <div class="login-barcode-text mt-2">Scan your badge</div>
                    </span>
                    <span class="login-or m-2 fs-2 text-muted">or</span>
                    <span class="login-element">
                        <button class="login-button select-cashier btn btn-lg btn-secondary"
                                t-on-click="() => this.selectCashier()">Select Cashier</button>
                    </span>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\overrides\components\cashier_name\cashier_name.js

```javascript
/** @odoo-module */

import { CashierName } from "@point_of_sale/app/navbar/cashier_name/cashier_name";
import { patch } from "@web/core/utils/patch";
import { useCashierSelector } from "@pos_hr/app/select_cashier_mixin";

patch(CashierName.prototype, {
    setup() {
        super.setup(...arguments);
        this.cashierSelector = useCashierSelector();
    },
    //@Override
    get avatar() {
        if (this.pos.config.module_pos_hr) {
            const cashier = this.pos.get_cashier();
            if (!(cashier && cashier.id)) {
                return "";
            }
            return `/web/image/hr.employee.public/${cashier.id}/avatar_128`;
        }
        return super.avatar;
    },
    //@Override
    get cssClass() {
        if (this.pos.config.module_pos_hr) {
            return { oe_status: true };
        }
        return super.cssClass;
    },
    async selectCashier() {
        return await this.cashierSelector();
    },
});

```

## File: static\src\overrides\components\cashier_name\cashier_name.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_hr.CashierName" t-inherit="point_of_sale.CashierName" t-inherit-mode="extension">
        <xpath expr="//div" position="attributes">
            <attribute name="t-on-click">() => this.selectCashier()</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\navbar\navbar.js

```javascript
/** @odoo-module */

import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";

patch(Navbar.prototype, {
    get showCashMoveButton() {
        const { cashier } = this.pos;
        return super.showCashMoveButton && (!cashier || cashier.role == "manager");
    },
    get showCloseSessionButton() {
        return (
            !this.pos.config.module_pos_hr ||
            (this.pos.get_cashier().role === "manager" && this.pos.get_cashier().user_id) ||
            this.pos.get_cashier_user_id() === this.pos.user.id
        );
    },
    get showBackendButton() {
        return (
            !this.pos.config.module_pos_hr ||
            (this.pos.get_cashier().role === "manager" && this.pos.get_cashier().user_id) ||
            this.pos.get_cashier_user_id() === this.pos.user.id
        );
    },
    async showLoginScreen() {
        this.pos.reset_cashier();
        await this.pos.showTempScreen("LoginScreen");
    },
});

```

## File: static\src\overrides\components\navbar\navbar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_hr.Navbar" t-inherit="point_of_sale.Navbar" t-inherit-mode="extension">
        <xpath expr="//li[hasclass('backend-button')]" position="attributes">
            <attribute name="t-if">
                !pos.config.module_pos_hr or pos.get_cashier().role === 'manager' or pos.get_cashier_user_id() === pos.pos_session.user_id[0]
            </attribute>
        </xpath>

        <xpath expr="//li[hasclass('close-button')]" position="attributes">
            <attribute name="t-if">
                !pos.config.module_pos_hr or pos.get_cashier().role === 'manager' or pos.get_cashier_user_id() === pos.pos_session.user_id[0]
            </attribute>
        </xpath>

        <xpath expr="//li[hasclass('backend-button')]" position="after">
            <li t-if="pos.config.module_pos_hr" class="menu-item navbar-button lock-button" t-on-click="showLoginScreen">
                <a class="dropdown-item py-2">
                Lock
                </a>
            </li>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\navbar\closing_popup\close_pos_popup.js

```javascript
/** @odoo-module */
import { ClosePosPopup } from "@point_of_sale/app/navbar/closing_popup/closing_popup";
import { patch } from "@web/core/utils/patch";

patch(ClosePosPopup.prototype, {
    async closeSession() {
        this.pos._resetConnectedCashier();
        super.closeSession();
    },
});

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
/** @odoo-module */

import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    async validateOrder(isForceValidate) {
        this.currentOrder.cashier = this.pos.get_cashier();
        await super.validateOrder(...arguments);
    },
});

```

## File: static\src\overrides\models\Chrome.js

```javascript
/** @odoo-module */

import { Chrome } from "@point_of_sale/app/pos_app";
import { patch } from "@web/core/utils/patch";

patch(Chrome.prototype, {
    get showCashMoveButton() {
        const { cashier } = this.pos;
        return super.showCashMoveButton && (!cashier || cashier.role == "manager");
    },
});

```

## File: static\src\overrides\models\models.js

```javascript
/** @odoo-module */

import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    setup(_defaultObj, options) {
        super.setup(...arguments);
        if (!options.json && this.pos.config.module_pos_hr) {
            this.cashier = this.pos.get_cashier();
        }
    },
    init_from_JSON(json) {
        super.init_from_JSON(...arguments);
        if (this.pos.config.module_pos_hr && json.employee_id) {
            this.cashier = this.pos.employee_by_id[json.employee_id];
        }
    },
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        if (this.pos.config.module_pos_hr) {
            json.employee_id = this.cashier ? this.cashier.id : false;
        }
        return json;
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    async setup() {
        await super.setup(...arguments);
        if (this.config.module_pos_hr) {
            if (!this.hasLoggedIn) {
                this.showTempScreen("LoginScreen");
            }
        }
    },
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.config.module_pos_hr) {
            this.employees = loadedData["hr.employee"];
            this.employee_by_id = loadedData["employee_by_id"];
            const savedCashier = this._getConnectedCashier();
            if (savedCashier) {
                this.set_cashier(savedCashier);
            } else {
                this.reset_cashier();
            }
        }
    },
    async after_load_server_data() {
        await super.after_load_server_data(...arguments);
        if (this.config.module_pos_hr) {
            const saved_cashier = this._getConnectedCashier();
            this.hasLoggedIn = saved_cashier ? true : false;
        }
    },
    reset_cashier() {
        this.cashier = {
            name: null,
            id: null,
            barcode: null,
            user_id: null,
            pin: null,
            role: null,
        };
        this._resetConnectedCashier();
    },
    set_cashier(employee) {
        this.cashier = employee;
        this._storeConnectedCashier(employee);
        const selectedOrder = this.get_order();
        if (selectedOrder && !selectedOrder.get_orderlines().length) {
            // Order without lines can be considered to be un-owned by any employee.
            // We set the cashier on that order to the currently set employee.
            selectedOrder.cashier = employee;
        }
        if (!this.cashierHasPriceControlRights() && this.numpadMode === "price") {
            this.numpadMode = "quantity";
        }
    },
    /**{name: null, id: null, barcode: null, user_id:null, pin:null}
     * If pos_hr is activated, return {name: string, id: int, barcode: string, pin: string, user_id: int}
     * @returns {null|*}
     */
    get_cashier() {
        if (this.config.module_pos_hr) {
            return this.cashier;
        }
        return super.get_cashier(...arguments);
    },
    get_cashier_user_id() {
        if (this.config.module_pos_hr) {
            return this.cashier.user_id ? this.cashier.user_id : null;
        }
        return super.get_cashier_user_id(...arguments);
    },
    async logEmployeeMessage(action, message) {
        if (!this.config.module_pos_hr) {
            super.logEmployeeMessage(...arguments);
            return;
        }
        await this.orm.call("pos.session", "log_partner_message", [
            this.pos_session.id,
            this.cashier.work_contact_id,
            action,
            message,
        ]);
    },
    _getConnectedCashier() {
        const cashier_id = sessionStorage.getItem(`connected_cashier_${this.config.id}`);
        if (cashier_id && this.employee_by_id[cashier_id]) {
            return this.employee_by_id[cashier_id];
        }
        return false;
    },
    _storeConnectedCashier(employee) {
        sessionStorage.setItem(`connected_cashier_${this.config.id}`, employee.id);
    },
    _resetConnectedCashier() {
        sessionStorage.removeItem(`connected_cashier_${this.config.id}`);
    },

    /**
     * @override
     */
    shouldShowCashControl() {
        if (this.config.module_pos_hr) {
            return super.shouldShowCashControl(...arguments) && this.hasLoggedIn;
        }
        return super.shouldShowCashControl(...arguments);
    },
    closePos() {
        if (this.config.module_pos_hr) {
            this._resetConnectedCashier();
        }
        return super.closePos(...arguments);
    },
    getPhoneSearchFields() {
        return ["phone_mobile_search"];
    },
});

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
                    <label for="basic_employee_ids" string="Basic rights" class="col-lg-4 o_light_label" />
                    <field name="basic_employee_ids" widget="many2many_tags" domain="[('company_id', '=', company_id)]" />
                </div>
                <div class="row">
                    <label for="advanced_employee_ids" string="Advanced rights" class="col-lg-4 o_light_label" />
                    <field name="advanced_employee_ids" widget="many2many_tags" domain="[('company_id', '=', company_id)]" />
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
                <field string='Cashier' name="employee_id" readonly="1" invisible="not employee_id"/>
                <field string='Cashier' name="user_id" readonly="1" invisible="employee_id"/>
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
                <field name="employee_id" widget="many2one_avatar_employee" readonly="state in ['done', 'invoiced']"/>
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
                    <label for="pos_basic_employee_ids" string="Basic rights" class="col-lg-4 o_light_label" />
                    <field name="pos_basic_employee_ids" widget="many2many_tags" placeholder="All employees" domain="[('company_id', '=', company_id)]" />
                </div>
                <div class="row">
                    <label for="pos_advanced_employee_ids" string="Advanced rights" class="col-lg-4 o_light_label" />
                    <field name="pos_advanced_employee_ids" widget="many2many_tags" placeholder="No employee" domain="[('company_id', '=', company_id)]" />
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

