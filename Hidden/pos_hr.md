# Odoo Module: pos_hr

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import report
from . import wizard

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
        'views/pos_payment_view.xml',
        'views/pos_order_report_view.xml',
        'views/single_employee_sales_report.xml',
        'views/multi_employee_sales_report.xml',
        'views/res_config_settings_views.xml',
        'wizard/pos_daily_sales_reports.xml',
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
        'web.assets_backend': [
            'pos_hr/static/src/app/print_report_button/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\account_bank_statement.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountBankStatementLine(models.Model):
    _inherit = 'account.bank.statement.line'

    employee_id = fields.Many2one('hr.employee', string="Employee", help="The employee who made the cash move.")

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib

from odoo import api, models, _
from odoo.exceptions import UserError
from odoo.tools import format_list

class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    @api.model
    def _load_pos_data_domain(self, data):
        config_id = self.env['pos.config'].browse(data['pos.config']['data'][0]['id'])
        return config_id._employee_domain(config_id.current_user_id.id)

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name', 'user_id', 'work_contact_id']

    def _load_pos_data(self, data):
        domain = self._load_pos_data_domain(data)
        fields = self._load_pos_data_fields(data['pos.config']['data'][0]['id'])

        employees = self.search(domain)
        manager_ids = employees.filtered(lambda emp: data['pos.config']['data'][0]['group_pos_manager_id'] in emp.user_id.groups_id.ids).mapped('id')

        employees_barcode_pin = employees.get_barcodes_and_pin_hashed()
        bp_per_employee_id = {bp_e['id']: bp_e for bp_e in employees_barcode_pin}

        employees = employees.read(fields, load=False)
        for employee in employees:
            if employee['id'] in manager_ids or employee['id'] in data['pos.config']['data'][0]['advanced_employee_ids']:
                role = 'manager'
            else:
                role = 'cashier'

            employee['_role'] = role
            employee['_barcode'] = bp_per_employee_id[employee['id']]['barcode']
            employee['_pin'] = bp_per_employee_id[employee['id']]['pin']

        return {
            'data': employees,
            'fields': fields,
        }

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
        configs_with_employees = self.env['pos.config'].sudo().search([('module_pos_hr', '=', True)]).filtered(lambda c: c.current_session_id)
        configs_with_all_employees = configs_with_employees.filtered(lambda c: not c.basic_employee_ids and not c.advanced_employee_ids)
        configs_with_specific_employees = configs_with_employees.filtered(lambda c: (c.basic_employee_ids or c.advanced_employee_ids) & self)
        if configs_with_all_employees or configs_with_specific_employees:
            error_msg = _("You cannot delete an employee that may be used in an active PoS session, close the session(s) first: \n")
            for employee in self:
                config_ids = configs_with_all_employees | configs_with_specific_employees.filtered(lambda c: employee in c.basic_employee_ids)
                if config_ids:
                    error_msg += _("Employee: %(employee)s - PoS Config(s): %(config_list)s \n", employee=employee.name, config_list=format_list(self.env, config_ids.mapped("name")))

            raise UserError(error_msg)

```

## File: models\multi_employee_sales_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class MultiEmployeeSalesReport(models.AbstractModel):
    _name = 'report.pos_hr.multi_employee_sales_report'
    _description = 'A collection of single session reports. One for each employee'

    @api.model
    def _get_report_values(self, docids, data=None):
        data = dict(data or {})
        data.update({
            'session_ids': data.get('session_ids'),
            'employee_ids': data.get('employee_ids'),
            'config_ids': data.get('config_ids'),
            'date_start': data.get('date_start'),
            'date_stop': data.get('date_stop'),
        })
        return data

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

    def write(self, vals):
        if 'advanced_employee_ids' not in vals:
            vals['advanced_employee_ids'] = []
        vals['advanced_employee_ids'] += [(4, emp_id) for emp_id in self._get_group_pos_manager().users.employee_id.ids]
        return super().write(vals)

    @api.onchange('basic_employee_ids')
    def _onchange_basic_employee_ids(self):
        for employee in self.basic_employee_ids:
            if employee in self.advanced_employee_ids:
                if employee.user_id._has_group('point_of_sale.group_pos_manager'):
                    self.basic_employee_ids -= employee
                else:
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
from odoo import models, fields, api, _
from markupsafe import Markup


class PosOrder(models.Model):
    _inherit = "pos.order"

    employee_id = fields.Many2one('hr.employee', string="Cashier", help="The employee who uses the cash register.")
    cashier = fields.Char(string="Cashier name", compute="_compute_cashier", store=True)

    @api.depends('employee_id', 'user_id')
    def _compute_cashier(self):
        for order in self:
            if order.employee_id:
                order.cashier = order.employee_id.name
            else:
                order.cashier = order.user_id.name

    def _post_chatter_message(self, body):
        body += Markup("<br/>")
        body += _("Cashier %s", self.cashier)
        self.message_post(body=body)

```

## File: models\pos_payment.py

```python
from odoo import models, fields, api


class PosPayment(models.Model):
    _inherit = "pos.payment"

    employee_id = fields.Many2one('hr.employee', string='Cashier', related='pos_order_id.employee_id', store=True, index=True)

    @api.depends('employee_id', 'user_id')
    def _compute_cashier(self):
        for order in self:
            if order.employee_id:
                order.cashier = order.employee_id.name
            else:
                order.cashier = order.user_id.name

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _
from odoo.tools import plaintext2html


class PosSession(models.Model):
    _inherit = 'pos.session'
    employee_id = fields.Many2one(
        "hr.employee",
        string="Cashier",
        help="The employee who currently uses the cash register",
        tracking=True,
    )

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        config_id = self.env['pos.config'].browse(config_id)
        if config_id.module_pos_hr:
            data += ['hr.employee']
        return data

    def set_opening_control(self, cashbox_value: int, notes: str):
        super().set_opening_control(cashbox_value, notes)
        if author_id := self._get_message_author():
            self.message_post(body=plaintext2html(_('Opened register')), author_id=author_id.id)

    def post_close_register_message(self):
        if author_id := self._get_message_author():
            self.message_post(body=plaintext2html(_('Closed Register')), author_id=author_id.id)
        else:
            return super().post_close_register_message()

    def _get_message_author(self):
        if not self.employee_id:
            return None
        
        if related_partners := self.employee_id._get_related_partners():
            return related_partners[0]
        
        return self.user_id.partner_id

    def _aggregate_payments_amounts_by_employee(self, payments):
        payments_by_employee = []

        for employee, payments_group in payments.grouped('employee_id').items():
            payments_by_employee.append({
                'id': employee.id if employee else 'others',
                'name': employee.name if employee else _('Others'),
                'amount': sum(payments_group.mapped('amount')),
            })

        # Sort such that "Others" is always the last item
        return sorted(
            payments_by_employee,
            key=lambda p: (p['id'] == 'others', p['name'])
        )

    def _aggregate_moves_by_employee(self):
        moves_per_employee = {}
        for employee, moves in self.sudo().statement_line_ids.grouped('employee_id').items():
            moves_per_employee[employee.id] = {
                'id': employee.id,
                'name': employee.name,
                'amount': sum(moves.mapped('amount')),
            }

        return sorted(moves_per_employee.values(), key=lambda p: -p['amount'])

    def get_closing_control_data(self):
        data = super().get_closing_control_data()

        orders = self._get_closed_orders()
        payments = orders.payment_ids.filtered(lambda p: p.payment_method_id.type != "pay_later")
        cash_payment_method_ids = self.payment_method_ids.filtered(lambda pm: pm.type == 'cash')
        default_cash_payment_method_id = cash_payment_method_ids[0] if cash_payment_method_ids else None
        default_cash_payments = payments.filtered(lambda p: p.payment_method_id == default_cash_payment_method_id) if default_cash_payment_method_id else self.env['pos.payment']
        non_cash_payment_method_ids = self.payment_method_ids - default_cash_payment_method_id if default_cash_payment_method_id else self.payment_method_ids
        non_cash_payments_grouped_by_method_id = {pm.id: orders.payment_ids.filtered(lambda p: p.payment_method_id == pm) for pm in non_cash_payment_method_ids}

        data['default_cash_details']['amount_per_employee'] = self._aggregate_payments_amounts_by_employee(default_cash_payments)
        for payment_method in data['non_cash_payment_methods']:
            payment_method['amount_per_employee'] = self._aggregate_payments_amounts_by_employee(non_cash_payments_grouped_by_method_id[payment_method['id']])

        data['default_cash_details']['moves_per_employee'] = self._aggregate_moves_by_employee()

        return data

    def _prepare_account_bank_statement_line_vals(self, session, sign, amount, reason, extras):
        vals = super()._prepare_account_bank_statement_line_vals(session, sign, amount, reason, extras)
        if extras.get('employee_id'):
            vals['employee_id'] = extras['employee_id']
        return vals

```

## File: models\product_product.py

```python
from odoo import api, models


class ProductProduct(models.Model):
    _inherit = 'product.product'

    @api.model
    def _load_pos_data_fields(self, config_id):
        result = super()._load_pos_data_fields(config_id)
        result.append('all_product_tag_ids')
        return result

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

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            pos_config_id = vals.get('pos_config_id')
            if pos_config_id:
                vals['pos_advanced_employee_ids'] = vals.get('pos_advanced_employee_ids', []) + [[4, emp_id] for emp_id in self.env['pos.config'].browse(pos_config_id)._get_group_pos_manager().users.employee_id.ids]
        return super().create(vals_list)

    @api.onchange('pos_basic_employee_ids')
    def _onchange_basic_employee_ids(self):
        for employee in self.pos_basic_employee_ids:
            if employee in self.pos_advanced_employee_ids:
                if employee.user_id._has_group('point_of_sale.group_pos_manager'):
                    self.pos_basic_employee_ids -= employee
                else:
                    self.pos_advanced_employee_ids -= employee

    @api.onchange('pos_advanced_employee_ids')
    def _onchange_advanced_employee_ids(self):
        for employee in self.pos_advanced_employee_ids:
            if employee in self.pos_basic_employee_ids:
                self.pos_basic_employee_ids -= employee

```

## File: models\single_employee_sales_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models
from odoo.osv.expression import AND


class SingleEmployeeSalesReport(models.AbstractModel):
    _name = 'report.pos_hr.single_employee_sales_report'
    _inherit = 'report.point_of_sale.report_saledetails'
    _description = 'Session sales details for a single employee'

    def _get_domain(self, date_start=False, date_stop=False, config_ids=False, session_ids=False, employee_id=False):
        domain = super()._get_domain(config_ids=config_ids, session_ids=session_ids)

        if (employee_id):
            domain = AND([domain, [('employee_id', '=', employee_id)]])

        return domain

    def _prepare_get_sale_details_args_kwargs(self, data):
        args, kwargs = super()._prepare_get_sale_details_args_kwargs(data)
        kwargs['employee_id'] = data.get('employee_id')
        return args, kwargs

    @api.model
    def get_sale_details(self, date_start=False, date_stop=False, config_ids=False, session_ids=False, employee_id=False):
        data = super().get_sale_details(config_ids=config_ids, session_ids=session_ids, employee_id=employee_id)

        if (employee_id):
            employee = self.env['hr.employee'].search([('id', '=', employee_id)])
            data['employee_name'] = employee.name

        return data

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_config
from . import pos_order
from . import hr_employee
from . import pos_session
from . import res_config_settings
from . import product_product
from . import pos_payment
from . import account_bank_statement
from . import single_employee_sales_report
from . import multi_employee_sales_report

```

## File: report\pos_order_report.py

```python
# -*- coding: utf-8 -*-

from functools import partial

from odoo import models, fields


class PosOrderReport(models.Model):
    _inherit = "report.pos.order"
    employee_id = fields.Many2one('hr.employee', string='Employee', readonly=True)

    def _select(self):
        return super()._select() + ',s.employee_id AS employee_id'

    def _group_by(self):
        return super()._group_by() + ',s.employee_id'

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-

from . import pos_order_report

```

## File: static\src\app\select_cashier_mixin.js

```javascript
/* global Sha1 */

import { _t } from "@web/core/l10n/translation";

import { NumberPopup } from "@point_of_sale/app/utils/input_popups/number_popup";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { useBarcodeReader } from "@point_of_sale/app/barcode/barcode_reader_hook";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { useService } from "@web/core/utils/hooks";
import { makeAwaitable, ask } from "@point_of_sale/app/store/make_awaitable_dialog";

export function useCashierSelector({ exclusive, onScan } = { onScan: () => {}, exclusive: false }) {
    const pos = usePos();
    const dialog = useService("dialog");
    const notification = useService("notification");
    useBarcodeReader(
        {
            async cashier(code) {
                const employee = pos.models["hr.employee"].find(
                    (emp) => emp._barcode === Sha1.hash(code.code)
                );
                if (
                    employee &&
                    employee !== pos.get_cashier() &&
                    (!employee._pin || (await checkPin(employee)))
                ) {
                    onScan && onScan(employee);
                }
                return employee;
            },
        },
        exclusive
    );

    async function checkPin(employee, pin = false) {
        let inputPin = pin;
        if (!pin) {
            inputPin = await makeAwaitable(dialog, NumberPopup, {
                formatDisplayedValue: (x) => x.replace(/./g, "•"),
                title: _t("Password?"),
            });
        } else {
            if (employee._pin !== Sha1.hash(inputPin)) {
                inputPin = await makeAwaitable(dialog, NumberPopup, {
                    formatDisplayedValue: (x) => x.replace(/./g, "•"),
                    title: _t("Password?"),
                });
            }
        }
        if (!inputPin || employee._pin !== Sha1.hash(inputPin)) {
            notification.add(_t("PIN not found"), {
                type: "warning",
                title: _t(`Wrong PIN`),
            });
            return false;
        }
        return true;
    }

    /**
     * Select a cashier, the returning value will either be an object or nothing (undefined)
     */
    return async function selectCashier(pin = false, login = false, list = false) {
        if (!pos.config.module_pos_hr) {
            return;
        }

        const prepareList = (employees) => {
            return employees.map((employee) => {
                return {
                    id: employee.id,
                    item: employee,
                    label: employee.name,
                    isSelected: false,
                };
            });
        };

        const wrongPinNotification = () => {
            notification.add(_t("PIN not found"), {
                type: "warning",
                title: _t(`Wrong PIN`),
            });
        };

        let employee = false;
        const allEmployees = pos.models["hr.employee"].filter(
            (employee) => employee.id !== pos.get_cashier()?.id
        );
        const pinMatchEmployees = allEmployees.filter(
            (employee) => !pin || Sha1.hash(pin) === employee._pin
        );

        if (!pinMatchEmployees.length && !pin) {
            await ask(dialog, {
                title: _t("No Cashiers"),
                body: _t("There is no cashier available."),
            });
            return;
        } else if (pin && !pinMatchEmployees.length) {
            wrongPinNotification();
            return;
        }

        if (pinMatchEmployees.length > 1 || list) {
            employee = await makeAwaitable(dialog, SelectionPopup, {
                title: _t("Change Cashier"),
                list: prepareList(allEmployees),
            });

            if (!employee) {
                return;
            }

            if (pin && Sha1.hash(pin) !== employee._pin) {
                wrongPinNotification();
                return;
            }
        } else if (pinMatchEmployees.length === 1) {
            employee = pinMatchEmployees[0];
        }

        if (!pin && employee && employee._pin) {
            const result = await checkPin(employee);

            if (!result) {
                return false;
            }
        }

        if (login && employee) {
            pos.hasLoggedIn = true;
            pos.set_cashier(employee);
        }

        const currentScreen = pos.mainScreen.component.name;
        if (currentScreen === "LoginScreen" && login && employee) {
            const isRestaurant = pos.config.module_pos_restaurant;
            let selectedScreen =
                pos.previousScreen && pos.previousScreen !== "LoginScreen"
                    ? pos.previousScreen
                    : isRestaurant
                    ? "FloorScreen"
                    : "ProductScreen";

            const props = {};
            if (selectedScreen === "PaymentScreen") {
                if (!pos.selectedOrderUuid) {
                    selectedScreen = isRestaurant ? "FloorScreen" : "ProductScreen";
                } else {
                    props.orderUuid = pos.selectedOrderUuid;
                }
            }
            pos.showScreen(selectedScreen, props);
        }

        return employee;
    };
}

```

## File: static\src\app\print_report_button\print_report_button.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { Component } from "@odoo/owl";
import { makeContext } from "@web/core/context";

class PrintReportButton extends Component {
    static template = "pos_hr.PrintReportButton";
    static props = {
        ...standardWidgetProps,
    };

    setup() {
        this.action = useService("action");
        // Why not `this.orm = useService("orm");` ?
        // Because in the `onClick` method, the first action call will potentially open the `new layout action wizard`
        // closing the current dialog. Once closed, the succeeding actions will be terminated because of the protection
        // made by calling `useService`. We want to continue with the full logic of the `onClick` method even if the
        // dialog is closed.
        this.orm = this.env.services.orm;
    }

    async onClick() {
        const context = makeContext([this.props.record.evalContext || {}]);

        const single_report_action = await this.orm.call(
            "pos.daily.sales.reports.wizard",
            "get_single_report_print_action",
            [[]],
            {
                pos_session_id: context.pos_session_id,
            }
        );
        await this.action.doAction({
            ...single_report_action,
            close_on_report_download: !context.add_report_per_employee,
        });

        if (context.add_report_per_employee) {
            const multi_report_action = await this.orm.call(
                "pos.daily.sales.reports.wizard",
                "get_multi_report_print_action",
                [[]],
                {
                    pos_session_id: context.pos_session_id,
                    employee_ids: context.employee_ids,
                }
            );
            await this.action.doAction({ ...multi_report_action, close_on_report_download: true });
        }
    }
}

export const printReportButton = {
    component: PrintReportButton,
};
registry.category("view_widgets").add("print_report_button", printReportButton);

```

## File: static\src\app\print_report_button\print_report_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="pos_hr.PrintReportButton">
        <button class="btn btn-primary" t-on-click.stop="onClick">Print</button>
    </t>

</templates>

```

## File: static\src\overrides\components\cashier_name\cashier_name.js

```javascript
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
    async selectCashier(pin = false, login = false, list = false) {
        return await this.cashierSelector(...arguments);
    },
});

```

## File: static\src\overrides\components\cashier_name\cashier_name.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_hr.CashierName" t-inherit="point_of_sale.CashierName" t-inherit-mode="extension">
        <xpath expr="//button" position="attributes">
            <attribute name="t-on-click">() => this.selectCashier(false, true, true)</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\closing_popup\closing_popup.js

```javascript
import { ClosePosPopup } from "@point_of_sale/app/navbar/closing_popup/closing_popup";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { AccordionItem } from "@point_of_sale/app/generic_components/accordion_item/accordion_item";

patch(ClosePosPopup, {
    components: { ...ClosePosPopup.components, AccordionItem },
    get paymentMethodBreakdownTitle() {
        return _t("Payments in %(paymentMethod)s", {
            paymentMethod: this.props.default_cash_details.name,
        });
    },
});

```

## File: static\src\overrides\components\closing_popup\closing_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_hr.ClosePosPopup" t-inherit="point_of_sale.ClosePosPopup" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('payment-methods-overview')]" position="attributes">
            <attribute name="t-if">!pos.config.module_pos_hr</attribute>
        </xpath>
        <xpath expr="//div[hasclass('payment-methods-overview')]" position="after">
            <t t-if="pos.config.module_pos_hr">
                <div t-if="pos.config.cash_control and pos.config.module_pos_hr" class="w-100 mb-3">
                    <t t-set="diff" t-value="getDifference(props.default_cash_details.id)" />
                    <t t-set="counted" t-value="state.payments[props.default_cash_details.id]?.counted || '0'" />
                    <div class="d-flex align-items-center justify-content-between fs-3">
                        <span t-esc="props.default_cash_details.name" />
                        <span t-esc="env.utils.formatCurrency(props.default_cash_details.amount)" />
                    </div>
                    <div class="d-flex align-items-center justify-content-between text-muted border-start ps-2">
                        <span>Opening</span>
                        <span t-esc="env.utils.formatCurrency(props.default_cash_details.opening)" />
                    </div>
                    <t t-set="amountByEmployee" t-value="props.default_cash_details.amount_per_employee" />
                    <PaymentMethodBreakdown title.translate="Payments" total_amount="props.default_cash_details.payment_amount" transactions="props.default_cash_details.amount_per_employee"/>
                    <PaymentMethodBreakdown title.translate="Cash in/out" total_amount="getMovesTotalAmount()" transactions="props.default_cash_details.moves_per_employee"/>
                    <div class="d-flex align-items-center justify-content-between text-muted border-start ps-2">
                        <span>Counted</span>
                        <span t-esc="env.utils.formatCurrency(env.utils.parseValidFloat(counted))" />
                    </div>
                    <div class="d-flex align-items-center justify-content-between text-muted border-start ps-2" t-att-class="{'text-danger fw-bold': diff}">
                        <span>Difference</span>
                        <span class="cash-difference" t-esc="env.utils.formatCurrency(diff)" />
                    </div>
                </div>
                <div class="w-100 mb-3" t-foreach="props.non_cash_payment_methods" t-as="pm" t-key="pm.id">
                    <t t-set="_showDiff" t-value="pm.type === 'bank' and pm.number !== 0" />
                    <t t-set="diff" t-value="_showDiff ? getDifference(pm.id) : 0" />
                    <t t-set="counted" t-value="_showDiff ? env.utils.parseValidFloat(state.payments[pm.id].counted) : 0" />
                    <div class="d-flex align-items-center justify-content-between fs-3">
                        <span t-esc="pm.name" />
                        <span t-esc="env.utils.formatCurrency(pm.amount)" />
                    </div>
                    <PaymentMethodBreakdown title.translate="Payments" total_amount="pm.amount" transactions="pm.amount_per_employee"/>
                    <div class="d-flex align-items-center justify-content-between text-muted border-start ps-2" t-att-class="{'text-danger fw-bold': diff}">
                        <span>Difference</span>
                        <span t-esc="env.utils.formatCurrency(diff)" />
                    </div>
            </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\navbar\navbar.js

```javascript
import { Navbar } from "@point_of_sale/app/navbar/navbar";
import { patch } from "@web/core/utils/patch";

patch(Navbar.prototype, {
    get showCreateProductButton() {
        if (!this.pos.config.module_pos_hr || this.pos.employeeIsAdmin) {
            return super.showCreateProductButton;
        } else {
            return false;
        }
    },
});

```

## File: static\src\overrides\components\navbar\navbar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="pos_hr.Navbar" t-inherit="point_of_sale.Navbar" t-inherit-mode="extension">
        <xpath expr="//DropdownItem[contains(text(), 'Backend')]" position="attributes">
            <attribute name="t-if">
                !pos.config.module_pos_hr or pos.employeeIsAdmin or pos.get_cashier_user_id() === pos.session.user_id?.id
            </attribute>
        </xpath>
        <xpath expr="//DropdownItem[contains(text(), 'Close Register')]" position="attributes">
            <attribute name="t-if">
                !pos.config.module_pos_hr or pos.employeeIsAdmin or pos.get_cashier_user_id() === pos.session.user_id?.id
            </attribute>
        </xpath>
        <xpath expr="//CashierName" position="after">
            <button t-if="pos.config.module_pos_hr and !ui.isSmall" class="lock-screen btn btn-light btn-lg" title="Lock" t-on-click="() => this.pos.showLoginScreen()">
                <i class="fa fa-fw fa-unlock"/>
            </button>
        </xpath>
        <xpath expr="//Dropdown//div[contains(@class, 'pos-burger-menu-items')]" position="inside">
            <DropdownItem t-if="pos.config.module_pos_hr and ui.isSmall" onSelected="() => this.pos.showLoginScreen()">
                Lock
            </DropdownItem>
        </xpath>
    </t>

</templates>

```

## File: static\src\overrides\components\navbar\cash_move_popup\cash_move_popup.js

```javascript
import { CashMovePopup } from "@point_of_sale/app/navbar/cash_move_popup/cash_move_popup";
import { patch } from "@web/core/utils/patch";

patch(CashMovePopup.prototype, {
    _prepare_try_cash_in_out_payload() {
        const result = super._prepare_try_cash_in_out_payload(...arguments);
        if (this.pos.config.module_pos_hr) {
            const employee_id = this.pos.get_cashier().id;
            result[result.length - 1] = { ...result[result.length - 1], employee_id };
        }
        return result;
    },
});

```

## File: static\src\overrides\components\opening_control_popup\opening_control_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_hr.OpeningControlPopup" t-inherit="point_of_sale.OpeningControlPopup" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('backend-button')]" position="attributes">
            <attribute name="t-if">!this.pos.config.module_pos_hr</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    async validateOrder(isForceValidate) {
        if (this.pos.config.module_pos_hr && this.pos.get_cashier() === null) {
            this.currentOrder.update({ employee_id: this.pos.get_cashier() });
        }

        await super.validateOrder(...arguments);
    },
});

```

## File: static\src\overrides\models\Chrome.js

```javascript
import { Chrome } from "@point_of_sale/app/pos_app";
import { patch } from "@web/core/utils/patch";

patch(Chrome.prototype, {
    get showCashMoveButton() {
        const { cashier } = this.pos;
        return super.showCashMoveButton && (!cashier || cashier._role == "manager");
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    // @Override
    getCashierName() {
        return this.employee_id?.name || super.getCashierName(...arguments);
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { browser } from "@web/core/browser/browser";

patch(PosStore.prototype, {
    async setup() {
        await super.setup(...arguments);
        if (this.config.module_pos_hr) {
            this.login = Boolean(odoo.from_backend) && !this.config.module_pos_hr;
            if (!this.hasLoggedIn) {
                this.showScreen("LoginScreen");
            }
        }
        this.employeeBuffer = [];
        browser.addEventListener("online", () => {
            this.employeeBuffer.forEach((employee) =>
                this.data.write("pos.session", [this.config.current_session_id.id], {
                    employee_id: employee.id,
                })
            );
            this.employeeBuffer = [];
        });
    },
    get employeeIsAdmin() {
        const cashier = this.get_cashier();
        return cashier._role === "manager";
    },
    checkPreviousLoggedCashier() {
        if (this.config.module_pos_hr) {
            const savedCashier = this._getConnectedCashier();
            if (savedCashier) {
                this.set_cashier(savedCashier);
            } else {
                this.reset_cashier();
            }
        } else {
            super.checkPreviousLoggedCashier(...arguments);
        }
    },
    async afterProcessServerData() {
        await super.afterProcessServerData(...arguments);
        if (this.config.module_pos_hr) {
            const saved_cashier = this._getConnectedCashier();
            this.hasLoggedIn = saved_cashier ? true : false;
        }
    },
    createNewOrder() {
        const order = super.createNewOrder(...arguments);

        if (this.config.module_pos_hr) {
            order.update({ employee_id: this.get_cashier() });
        }

        return order;
    },
    set_cashier(employee) {
        super.set_cashier(employee);

        if (this.config.module_pos_hr) {
            if (navigator.onLine) {
                this.data.write("pos.session", [this.config.current_session_id.id], {
                    employee_id: employee.id,
                });
            } else {
                this.employeeBuffer.push(employee);
            }
            const o = this.get_order();
            if (o && !o.get_orderlines().length) {
                // Order without lines can be considered to be un-owned by any employee.
                // We set the cashier on that order to the currently set employee.
                o.update({ employee_id: employee });
            }
            if (!this.cashierHasPriceControlRights() && this.numpadMode === "price") {
                this.numpadMode = "quantity";
            }
        }
    },
    addLineToCurrentOrder(vals, opt = {}, configure = true) {
        vals.employee_id = false;

        if (this.config.module_pos_hr) {
            const cashier = this.get_cashier();

            if (cashier && cashier.model.modelName === "hr.employee") {
                const order = this.get_order();
                order.update({ employee_id: this.get_cashier() });
            }
        }

        return super.addLineToCurrentOrder(vals, opt, configure);
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
        await this.data.call("pos.session", "log_partner_message", [
            this.session.id,
            this.cashier.work_contact_id?.id,
            action,
            message,
        ]);
    },
    _getConnectedCashier() {
        if (!this.config.module_pos_hr) {
            return super._getConnectedCashier(...arguments);
        }
        const cashier_id = Number(sessionStorage.getItem(`connected_cashier_${this.config.id}`));
        if (cashier_id && this.models["hr.employee"].get(cashier_id)) {
            return this.models["hr.employee"].get(cashier_id);
        }
        return false;
    },

    /**
     * @override
     */
    shouldShowOpeningControl() {
        if (this.config.module_pos_hr) {
            return super.shouldShowOpeningControl(...arguments) && this.hasLoggedIn;
        }
        return super.shouldShowOpeningControl(...arguments);
    },
    async allowProductCreation() {
        if (this.config.module_pos_hr) {
            return this.employeeIsAdmin;
        }
        return await super.allowProductCreation();
    },
});

```

## File: static\src\overrides\screens\login_screen\login_screen.js

```javascript
import { useCashierSelector } from "@pos_hr/app/select_cashier_mixin";
import { _t } from "@web/core/l10n/translation";
import { LoginScreen } from "@point_of_sale/app/screens/login_screen/login_screen";
import { patch } from "@web/core/utils/patch";
import { useAutofocus } from "@web/core/utils/hooks";
import { onWillUnmount, useExternalListener, useState } from "@odoo/owl";

patch(LoginScreen.prototype, {
    setup() {
        super.setup(...arguments);

        this.state = useState({
            pin: "",
        });

        if (this.pos.config.module_pos_hr) {
            this.cashierSelector = useCashierSelector({
                onScan: (employee) => employee && this.selectOneCashier(employee),
                exclusive: true,
            });

            useAutofocus();
            useExternalListener(window, "keypress", async (ev) => {
                if (this.pos.login && ev.key === "Enter" && this.state.pin) {
                    await this.selectCashier(this.state.pin, true);
                }
            });
        }

        onWillUnmount(() => {
            this.state.pin = "";
            this.pos.login = false;
        });
    },
    async selectCashier(pin = false, login = false, list = false) {
        return await this.cashierSelector(pin, login, list);
    },
    unlockRegister() {
        this.pos.login = true;
    },
    openRegister() {
        if (this.pos.config.module_pos_hr) {
            this.pos.login = true;
        } else {
            super.openRegister();
        }
    },
    async clickBack() {
        if (!this.pos.config.module_pos_hr) {
            super.clickBack();
            return;
        }

        if (this.pos.login) {
            this.state.pin = "";
            this.pos.login = false;
        } else {
            const employee = await this.selectCashier();
            if (
                employee &&
                (employee._role === "manager" || employee.user_id?.id === this.pos.user.id)
            ) {
                super.clickBack();
                return;
            }
        }
    },
    get backBtnName() {
        return this.pos.login && this.pos.config.module_pos_hr ? _t("Discard") : super.backBtnName;
    },
});

```

## File: static\src\overrides\screens\login_screen\login_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_hr.LoginScreen" t-inherit="point_of_sale.LoginScreen" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('screen-login')]" position="attributes">
            <attribute name="t-if">!this.pos.config.module_pos_hr || !this.pos.login</attribute>
        </xpath>
        <xpath expr="//div[hasclass('screen-login')]" position="after">
            <div t-if="this.pos.config.module_pos_hr and this.pos.login" class="screen-login flex-grow-1 d-flex align-items-center justify-content-center">
                <div class="d-flex bg-white p-3 gap-2 rounded">
                    <input t-model="this.state.pin"
                        t-ref="autofocus"
                        type="password"
                        class="form-control form-control-lg rounded flex-grow-1"
                        placeholder="Enter your PIN" />
                    <button class="select-cashier btn btn-secondary px-4" t-on-click="() => this.selectCashier(false, true)">
                        <i class="fa fa-users" aria-hidden="true"></i>
                    </button>
                    <button class="btn btn-secondary mobile-scanner px-4" t-if="this.pos.config.module_pos_hr">
                        <i class="fa fa-barcode" aria-hidden="true"></i>
                    </button>
                </div>
            </div>
        </xpath>
        <xpath expr="//button[hasclass('open-register-btn')]/span" position="replace">
            <span t-if="this.pos.session.state !== 'opened'" class="d-flex flex-grow-1 align-items-center">Open Register</span>
            <span t-else="" class="d-flex flex-grow-1 align-items-center">Unlock Register</span>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\screens\product_screen\order_summary\order_summary.js

```javascript
import { OrderSummary } from "@point_of_sale/app/screens/product_screen/order_summary/order_summary";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(OrderSummary.prototype, {
    async setLinePrice(line, price) {
        if (this.pos.cashierHasPriceControlRights()) {
            await super.setLinePrice(line, price);
            return;
        }

        this.dialog.add(AlertDialog, {
            title: _t("Access Denied"),
            body: _t("You are not allowed to change the price of a product."),
        });
    },
});

```

## File: static\src\overrides\screens\product_screen\product_info_popup\product_info_popup.js

```javascript
import { ProductInfoPopup } from "@point_of_sale/app/screens/product_screen/product_info_popup/product_info_popup";
import { patch } from "@web/core/utils/patch";

patch(ProductInfoPopup.prototype, {
    get allowProductEdition() {
        return !this.pos.config.module_pos_hr || this.pos.employeeIsAdmin;
    },
});

```

## File: static\src\overrides\screens\receipt_screen\receipt_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ReceiptScreen" t-inherit="point_of_sale.ReceiptScreen" t-inherit-mode="extension">
        <xpath expr="//span[hasclass('edit-order-payment')]" position="attributes">
            <attribute name="t-if">(!this.pos.config.module_pos_hr || this.pos.employeeIsAdmin) and this.currentOrder.nb_print === 0</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\screens\screens\ticket_screen\ticket_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_event.TicketScreen" t-inherit="point_of_sale.TicketScreen" t-inherit-mode="extension">
        <xpath expr="//button[hasclass('edit-order-payment')]" position="attributes">
            <attribute name="t-if">!this.pos.config.module_pos_hr || this.pos.employeeIsAdmin</attribute>
        </xpath>
    </t>
</templates>

```

## File: views\multi_employee_sales_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="multi_employee_sales_report_action" model="ir.actions.report">
        <field name="name">Employee Sales Details</field>
        <field name="model">pos.session</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">pos_hr.multi_employee_sales_report</field>
    </record>

    <template id="multi_employee_sales_report">
        <t t-set="company" t-value="env.company"/>
        <t t-call="point_of_sale.pos_daily_sales_html_container">
            <t t-call="web.basic_layout">
                <t t-foreach="employee_ids" t-as="employee_id" t-key="employee_id">
                    <t t-call="pos_hr.single_employee_sales_report">
                        <t t-set="data" t-value="env['report.pos_hr.single_employee_sales_report'].get_sale_details(date_start, date_stop, config_ids, session_ids, employee_id)"/>
                        <t t-set="opening_note" t-value="data['opening_note']"/>
                        <t t-set="closing_note" t-value="data['closing_note']"/>
                        <t t-set="state" t-value="data['state']"/>
                        <t t-set="currency" t-value="data['currency']"/>
                        <t t-set="nbr_orders" t-value="data['nbr_orders']"/>
                        <t t-set="date_start" t-value="data['date_start']"/>
                        <t t-set="date_stop" t-value="data['date_stop']"/>
                        <t t-set="session_name" t-value="data['session_name']"/>
                        <t t-set="employee_name" t-value="data['employee_name']"/>
                        <t t-set="config_names" t-value="data['config_names']"/>
                        <t t-set="payments" t-value="data['payments']"/>
                        <t t-set="company_name" t-value="data['company_name']"/>
                        <t t-set="taxes" t-value="data['taxes']"/>
                        <t t-set="taxes_info" t-value="data['taxes_info']"/>
                        <t t-set="products" t-value="data['products']"/>
                        <t t-set="products_info" t-value="data['products_info']"/>
                        <t t-set="refund_taxes" t-value="data['refund_taxes']"/>
                        <t t-set="refund_taxes_info" t-value="data['refund_taxes_info']"/>
                        <t t-set="refund_info" t-value="data['refund_info']"/>
                        <t t-set="refund_products" t-value="data['refund_products']"/>
                        <t t-set="discount_number" t-value="data['discount_number']"/>
                        <t t-set="discount_amount" t-value="data['discount_amount']"/>
                        <t t-set="invoiceList" t-value="data['invoiceList']"/>
                        <t t-set="invoiceTotal" t-value="data['invoiceTotal']"/>
                        <t t-set="total_paid" t-value="data['total_paid']"/>
                    </t>
                </t>    
            </t>
        </t>
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
            <xpath expr="//div[@id='warning_text_employees']" position='replace'>
                <field name="company_id" invisible="1" />
                <div class="row">
                    <label for="basic_employee_ids" string="Basic rights" class="col-lg-5 o_light_label" />
                    <field name="basic_employee_ids" widget="many2many_tags" placeholder="All Employees" domain="[('company_id', '=', company_id)]" />
                </div>
                <div class="row">
                    <label for="advanced_employee_ids" string="Advanced rights" class="col-lg-5 o_light_label" />
                    <field name="advanced_employee_ids" widget="many2many_tags" placeholder="Select Employee(s)" domain="[('company_id', '=', company_id)]" />
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
        <field name="name">report.pos.order.view.list.inherit.pos.hr</field>
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
                <field name="employee_id" readonly="1" invisible="not employee_id"/>
                <field name="user_id" readonly="1" invisible="employee_id"/>
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
        <field name="name">pos.order.list.inherit</field>
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

## File: views\pos_payment_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_payment_tree_inherit" model="ir.ui.view">
        <field name="name">pos.payment.list.inherit</field>
        <field name="model">pos.payment</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_payment_tree"/>
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
                    <label for="pos_basic_employee_ids" string="Basic rights" class="col-lg-4 o_light_label" />
                    <field name="pos_basic_employee_ids" widget="many2many_tags" placeholder="All Employees" domain="[('company_id', '=', company_id)]" />
                </div>
                <div class="row">
                    <label for="pos_advanced_employee_ids" string="Advanced rights" class="col-lg-4 o_light_label" />
                    <field name="pos_advanced_employee_ids" widget="many2many_tags" placeholder="Select Employee(s)" domain="[('company_id', '=', company_id)]" />
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\single_employee_sales_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="pos_hr.single_employee_sales_report" inherit_id="point_of_sale.pos_session_sales_details" primary="True">
        <xpath expr="//h2[@id='daily_report_title']" position="after">
            <t t-if="employee_name">
                <h5>Employee: <span t-out="employee_name">Abigal Peterson</span></h5>
            </t>
        </xpath>

        <xpath expr="//h2[@id='daily_report_title']" position="replace">
            <h2>Employee Sales Report</h2>
        </xpath>

        <xpath expr="//t[@id='closing_session']" position="replace"/>
    </template>
</odoo>
```

## File: wizard\pos_daily_sales_reports.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class PosDailyReportPerEmployee(models.TransientModel):
    _inherit = 'pos.daily.sales.reports.wizard'

    add_report_per_employee = fields.Boolean(string='Add a report per each employee', default=True)
    employee_ids = fields.Many2many('hr.employee', compute='_compute_employee_ids')

    def _get_report_data(self, pos_session_id):
        pos_session = self.env['pos.session'].browse(pos_session_id)
        return {'date_start': False, 'date_stop': False, 'config_ids': pos_session.config_id.ids, 'session_ids': pos_session.ids}

    def get_single_report_print_action(self, pos_session_id):
        data = self._get_report_data(pos_session_id)
        return self.env.ref('point_of_sale.sale_details_report').report_action([], data=data)

    def get_multi_report_print_action(self, pos_session_id, employee_ids):
        data = self._get_report_data(pos_session_id)
        data['employee_ids'] = employee_ids
        return self.env.ref('pos_hr.multi_employee_sales_report_action').report_action([], data=data)

    @api.depends('pos_session_id')
    def _compute_employee_ids(self):
        for wizard in self:
            domain = [('session_id', '=', self.pos_session_id.id)]
            orders = self.env['pos.order'].search(domain)
            wizard.employee_ids = orders.mapped('employee_id')

    @api.onchange('pos_session_id')
    def _onchange_pos_session_id(self):
        self.ensure_one()
        if self.pos_session_id and not self.employee_ids:
            self.add_report_per_employee = False

```

## File: wizard\pos_daily_sales_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
   <record id="view_pos_daily_sales_reports_wizard" model="ir.ui.view">
        <field name="name">pos.daily.sales.reports.wizard.form.inherit</field>
        <field name="model">pos.daily.sales.reports.wizard</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_daily_sales_reports_wizard"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='pos_session_group']" position="before">
                <div class="alert alert-warning oe_button_box" role="alert" invisible="not pos_session_id or employee_ids">
                    Can't generate a report per employee! as selected session has no orders associated with any employee.
                </div>
            </xpath>
            <xpath expr="//group[@name='pos_session_group']" position="after">
                <div>
                    <field name="add_report_per_employee" readonly="pos_session_id and not employee_ids"/>
                    <label for="add_report_per_employee"/>
                </div>
                <field name="employee_ids" invisible="True"/>
            </xpath>
            <xpath expr="//button[@name='generate_report']" position="replace">
                <widget name="print_report_button"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_daily_sales_reports

```

