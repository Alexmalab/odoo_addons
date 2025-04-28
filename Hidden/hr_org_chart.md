# Odoo Module: hr_org_chart

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'HR Org Chart',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
Org Chart Widget for HR
=======================

This module extend the employee form with a organizational chart.
(N+1, N+2, direct subordinates)
        """,
    'depends': ['hr', 'web_hierarchy'],
    'auto_install': ['hr'],
    'data': [
        'views/hr_department_views.xml',
        'views/hr_views.xml',
        'views/hr_employee_public_views.xml',
        'views/hr_org_chart_menus.xml',
    ],
    'assets': {
        'web._assets_primary_variables': [
            'hr_org_chart/static/src/scss/variables.scss',
        ],
        'web.assets_backend': [
            'hr_org_chart/static/src/fields/*',
        ],
        'web.assets_backend_lazy': [
            'hr_org_chart/static/src/views/**/*',
        ],
        'web.assets_unit_tests': [
            'hr_org_chart/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\hr_org_chart.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.exceptions import AccessError
from odoo.http import request


class HrOrgChartController(http.Controller):
    _managers_level = 5  # FP request

    def _check_employee(self, employee_id, **kw):
        if not employee_id:  # to check
            return None
        employee_id = int(employee_id)

        context = kw.get('context', request.env.context)
        if 'allowed_company_ids' in context:
            cids = context['allowed_company_ids']
        else:
            cids = [request.env.company.id]

        Employee = request.env['hr.employee.public'].with_context(allowed_company_ids=cids)
        employee = Employee.browse(employee_id)
        # check and raise
        return employee if employee.has_access('read') else None

    def _prepare_employee_data(self, employee):
        job = employee.sudo().job_id
        return dict(
            id=employee.id,
            name=employee.name,
            link='/mail/view?model=%s&res_id=%s' % ('hr.employee.public', employee.id,),
            job_id=job.id,
            job_name=job.name or '',
            job_title=employee.job_title or '',
            direct_sub_count=len(employee.child_ids - employee),
            indirect_sub_count=employee.child_all_count,
        )

    @http.route('/hr/get_redirect_model', type='json', auth='user')
    def get_redirect_model(self):
        if request.env['hr.employee'].has_access('read'):
            return 'hr.employee'
        return 'hr.employee.public'

    @http.route('/hr/get_org_chart', type='json', auth='user')
    def get_org_chart(self, employee_id, **kw):

        employee = self._check_employee(employee_id, **kw)
        if not employee:  # to check
            return {
                'managers': [],
                'children': [],
            }

        # compute employee data for org chart
        ancestors, current = request.env['hr.employee.public'].sudo(), employee.sudo()
        while current.parent_id and len(ancestors) < self._managers_level+1 and current != current.parent_id:
            ancestors += current.parent_id
            current = current.parent_id

        values = dict(
            self=self._prepare_employee_data(employee),
            managers=[
                self._prepare_employee_data(ancestor)
                for idx, ancestor in enumerate(ancestors)
                if idx < self._managers_level
            ],
            managers_more=len(ancestors) > self._managers_level,
            children=[self._prepare_employee_data(child) for child in employee.child_ids if child != employee],
        )
        values['managers'].reverse()
        return values

    @http.route('/hr/get_subordinates', type='json', auth='user')
    def get_subordinates(self, employee_id, subordinates_type=None, **kw):
        """
        Get employee subordinates.
        Possible values for 'subordinates_type':
            - 'indirect'
            - 'direct'
        """
        employee = self._check_employee(employee_id, **kw)
        if not employee:  # to check
            return {}

        if subordinates_type == 'direct':
            res = (employee.child_ids - employee).ids
        elif subordinates_type == 'indirect':
            res = (employee.subordinate_ids - employee.child_ids).ids
        else:
            res = employee.subordinate_ids.ids

        return res

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*
from . import hr_org_chart

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Employee(models.Model):
    _inherit = ["hr.employee"]

    subordinate_ids = fields.One2many('hr.employee', string='Subordinates', compute='_compute_subordinates', help="Direct and indirect subordinates",
                                      compute_sudo=True)
    is_subordinate = fields.Boolean(compute="_compute_is_subordinate", search="_search_is_subordinate")


class HrEmployeePublic(models.Model):
    _inherit = ["hr.employee.public"]

    subordinate_ids = fields.One2many('hr.employee.public', string='Subordinates', compute='_compute_subordinates', help="Direct and indirect subordinates",
                                      compute_sudo=True)
    is_subordinate = fields.Boolean(compute="_compute_is_subordinate", search="_search_is_subordinate")

```

## File: models\hr_org_chart_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    child_all_count = fields.Integer(
        'Indirect Subordinates Count',
        compute='_compute_subordinates', recursive=True, store=False,
        compute_sudo=True)
    department_color = fields.Integer("Department Color", related="department_id.color")
    child_count = fields.Integer(
        'Direct Subordinates Count',
        compute='_compute_child_count', recursive=True,
        compute_sudo=True,
    )

    def _get_subordinates(self, parents=None):
        """
        Helper function to compute subordinates_ids.
        Get all subordinates (direct and indirect) of an employee.
        An employee can be a manager of his own manager (recursive hierarchy; e.g. the CEO is manager of everyone but is also
        member of the RD department, managed by the CTO itself managed by the CEO).
        In that case, the manager in not counted as a subordinate if it's in the 'parents' set.
        """
        if not parents:
            parents = self.env[self._name]

        indirect_subordinates = self.env[self._name]
        parents |= self
        direct_subordinates = self.child_ids - parents
        child_subordinates = direct_subordinates._get_subordinates(parents=parents) if direct_subordinates else self.browse()
        indirect_subordinates |= child_subordinates
        return indirect_subordinates | direct_subordinates

    @api.depends('child_ids', 'child_ids.child_all_count')
    def _compute_subordinates(self):
        for employee in self:
            employee.subordinate_ids = employee._get_subordinates()
            employee.child_all_count = len(employee.subordinate_ids)

    @api.depends_context('uid', 'company')
    @api.depends('parent_id')
    def _compute_is_subordinate(self):
        subordinates = self.env.user.employee_id.subordinate_ids
        if not subordinates:
            self.is_subordinate = False
        else:
            for employee in self:
                employee.is_subordinate = employee in subordinates

    def _search_is_subordinate(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise UserError(_('Operation not supported'))
        # Double negation
        if not value:
            operator = '!=' if operator == '=' else '='
        if not self.env.user.employee_id.subordinate_ids:
            return [('id', operator, self.env.user.employee_id.id)]
        return (['!'] if operator == '!=' else []) + [('id', 'in', self.env.user.employee_id.subordinate_ids.ids)]

    def _compute_child_count(self):
        employee_read_group = self._read_group(
            [('parent_id', 'in', self.ids)],
            ['parent_id'],
            ['id:count'],
        )
        child_count_per_parent_id = dict(employee_read_group)
        for employee in self:
            employee.child_count = child_count_per_parent_id.get(employee._origin, 0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_org_chart_mixin
from . import hr_employee

```

## File: static\src\fields\hooks.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { user } from "@web/core/user";
import { rpc } from "@web/core/network/rpc";
import { useService } from "@web/core/utils/hooks";

/**
 * Redirect to the sub employee kanban view.
 *
 * @private
 * @param {MouseEvent} event
 * @returns {Promise} action loaded
 *
 */
export function onEmployeeSubRedirect() {
    const actionService = useService('action');
    const orm = useService('orm');

    return async (event) => {
        const employeeId = parseInt(event.currentTarget.dataset.employeeId);
        if (!employeeId) {
            return {};
        }
        const type = event.currentTarget.dataset.type || 'direct';
        // Get subordonates of an employee through a rpc call.
        const subordinateIds = await rpc('/hr/get_subordinates', {
            employee_id: employeeId,
            subordinates_type: type,
            context: user.context
        });
        let action = await orm.call('hr.employee', 'get_formview_action', [employeeId]);
        action = {...action,
            name: _t('Team'),
            view_mode: 'kanban,list,form',
            views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
            domain: [['id', 'in', subordinateIds]],
            res_id: false,
            context: {
                default_parent_id: employeeId,
            }
        };
        actionService.doAction(action);
    };
}

```

## File: static\src\fields\hr_org_chart.js

```javascript
/** @odoo-module */

import { rpc } from "@web/core/network/rpc";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { usePopover } from "@web/core/popover/popover_hook";
import { user } from "@web/core/user";
import { onEmployeeSubRedirect } from './hooks';
import { Component, onWillStart, onWillRender, useState } from "@odoo/owl";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

class HrOrgChartPopover extends Component {
    static template = "hr_org_chart.hr_orgchart_emp_popover";
    static props = {
        employee: Object,
        close: Function,
    };
    async setup() {
        super.setup();

        this.orm = useService('orm');
        this.actionService = useService("action");
        this._onEmployeeSubRedirect = onEmployeeSubRedirect();
    }

    /**
     * Redirect to the employee form view.
     *
     * @private
     * @param {MouseEvent} event
     * @returns {Promise} action loaded
     */
    async _onEmployeeRedirect(employeeId) {
        const action = await this.orm.call('hr.employee', 'get_formview_action', [employeeId]);
        this.actionService.doAction(action); 
    }
}

export class HrOrgChart extends Component {
    static template = "hr_org_chart.hr_org_chart";
    static props = {...standardFieldProps};
    async setup() {
        super.setup();

        this.orm = useService('orm');
        this.actionService = useService("action");
        this.popover = usePopover(HrOrgChartPopover);

        this.state = useState({'employee_id': null});
        this.lastParent = null;
        this._onEmployeeSubRedirect = onEmployeeSubRedirect();

        onWillStart(this.handleComponentUpdate.bind(this));
        onWillRender(this.handleComponentUpdate.bind(this));
    }

    /**
     * Called on start and on render
     */
    async handleComponentUpdate() {
        this.employee = this.props.record.data;
        // the widget is either dispayed in the context of a hr.employee form or a res.users form
        this.state.employee_id = this.employee.employee_ids !== undefined ? this.employee.employee_ids.resIds[0] : this.props.record.resId;
        const manager = this.employee.parent_id || this.employee.employee_parent_id;
        const forceReload = this.lastRecord !== this.props.record || this.lastParent != manager;
        this.lastParent = manager;
        this.lastRecord = this.props.record;
        await this.fetchEmployeeData(this.state.employee_id, forceReload);
    }

    async fetchEmployeeData(employeeId, force = false) {
        if (!employeeId) {
            this.managers = [];
            this.children = [];
            if (this.view_employee_id) {
                this.render(true);
            }
            this.view_employee_id = null;
        } else if (employeeId !== this.view_employee_id || force) {
            this.view_employee_id = employeeId;
            let orgData = await rpc(
                '/hr/get_org_chart',
                {
                    employee_id: employeeId,
                    context: user.context,
                }
            );
            if (Object.keys(orgData).length === 0) {
                orgData = {
                    managers: [],
                    children: [],
                }
            }
            this.managers = orgData.managers;
            this.children = orgData.children;
            this.managers_more = orgData.managers_more;
            this.self = orgData.self;
            this.render(true);
        }
    }

    _onOpenPopover(event, employee) {
        this.popover.open(event.currentTarget, { employee });
    }

    /**
     * Redirect to the employee form view.
     *
     * @private
     * @param {MouseEvent} event
     * @returns {Promise} action loaded
     */
    async _onEmployeeRedirect(employeeId) {
        const action = await this.orm.call('hr.employee', 'get_formview_action', [employeeId]);
        this.actionService.doAction(action); 
    }

    async _onEmployeeMoreManager(managerId) {
        await this.fetchEmployeeData(managerId);
        this.state.employee_id = managerId;
    }
}

export const hrOrgChart = {
    component: HrOrgChart,
};

registry.category("fields").add("hr_org_chart", hrOrgChart);

```

## File: static\src\fields\hr_org_chart.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="hr_org_chart.hr_org_chart_employee">
    <t t-set="is_self" t-value="employee.id == view_employee_id"/>

    <section t-if="employee_type == 'self'" t-attf-class="o_org_chart_entry_self_container #{managers.length &gt; 0 ? 'o_org_chart_has_managers' : ''}">
        <div t-attf-class="o_org_chart_entry o_org_chart_entry_#{employee_type} d-flex position-relative py-2 overflow-visible #{managers.length &gt; 0 ? 'o_treeEntry' : ''}">
            <t t-call="hr_org_chart.hr_org_chart_employee_content">
                <t t-set="is_self" t-value="is_self"/>
            </t>
        </div>
    </section>

    <div t-else="" t-attf-class="o_org_chart_entry o_org_chart_entry_#{employee_type} o_treeEntry d-flex position-relative py-2 overflow-visible">
        <t t-call="hr_org_chart.hr_org_chart_employee_content">
            <t t-set="is_self" t-value="is_self"/>
        </t>
    </div>
</t>

<t t-name="hr_org_chart.hr_org_chart_employee_content">
    <div class="o_media_left position-relative">
        <!-- NOTE: Since by the default on not squared images odoo add white borders,
            use bg-images to get a clean and centred images -->
        <a t-if="! is_self"
            class="o_media_object d-block rounded o_employee_redirect"
            t-att-style="'background-image:url(\'/web/image/hr.employee.public/' + employee.id + '/avatar_1024/\')'"
            t-att-alt="employee.name"
            t-att-data-employee-id="employee.id"
            t-att-href="employee.link"
            t-on-click.prevent="() => this._onEmployeeRedirect(employee.id)"/>
        <div t-if="is_self"
            class="o_media_object d-block rounded border border-info"
            t-att-style="'background-image:url(\'/web/image/hr.employee.public/' + employee.id + '/avatar_1024/\')'"/>
    </div>

    <div class="d-flex flex-grow-1 align-items-center justify-content-between position-relative px-3">
        <a t-if="!is_self" t-att-href="employee.link" class="o_employee_redirect d-flex flex-column" t-att-data-employee-id="employee.id" t-on-click.prevent="() => this._onEmployeeRedirect(employee.id)">
            <b class="o_media_heading m-0 fs-6" t-esc="employee.name"/>
            <small class="text-muted fw-bold" t-esc="employee.job_title"/>
        </a>
        <div t-if="is_self" class="d-flex flex-column">
            <h5 class="o_media_heading m-0" t-esc="employee.name"/>
            <small class="text-muted fw-bold" t-esc="employee.job_title"/>
        </div>
        <button t-if="employee.indirect_sub_count &gt; 0"
                class="btn p-0 fs-3"
                tabindex="0"
                t-att-data-emp-name="employee.name"
                t-att-data-emp-id="employee.id"
                t-att-data-emp-dir-subs="employee.direct_sub_count"
                t-att-data-emp-ind-subs="employee.indirect_sub_count"
                data-bs-trigger="focus"
                data-bs-toggle="popover"
                t-on-click="(event) => this._onOpenPopover(event, employee)">
            <a href="#"
                t-attf-class="badge rounded-pill bg-300 border {{employee.indirect_sub_count &lt; 10 ? 'px-2' : 'px-1' }}"
                t-esc="employee.indirect_sub_count"
                />
        </button>

    </div>
</t>

<t t-name="hr_org_chart.hr_org_chart">
    <!-- NOTE: Desidered behaviour:
            The maximun number of people is always 7 (including 'self'). Managers have priority over suburdinates
            Eg. 1 Manager + 1 self = show just 5 subordinates (if availables)
            Eg. 0 Manager + 1 self = show 6 subordinates (if available)

        -->
    <t t-set="emp_count" t-value="0"/>
    <div t-if='managers.length &gt; 0' class="o_org_chart_group_up position-relative">
        <div t-if='managers_more' class="o_org_chart_more pe-3">
            <a href="#" t-att-data-employee-id="managers[0].id" class="o_employee_more_managers d-block bg-100 px-3" t-on-click.prevent="() => this._onEmployeeMoreManager(managers[0].id)">
                <i class="fa fa-angle-double-up" role="img" aria-label="More managers" title="More managers"/>
            </a>
        </div>

        <t t-foreach="managers" t-as="employee" t-key="employee_index">
            <t t-set="emp_count" t-value="emp_count + 1"/>
            <t t-call="hr_org_chart.hr_org_chart_employee">
                <t t-set="employee_type" t-value="'manager'"/>
            </t>
        </t>
    </div>

    <t t-if="children.length || managers.length" t-call="hr_org_chart.hr_org_chart_employee">
        <t t-set="employee_type" t-value="'self'"/>
        <t t-set="employee" t-value="self"/>
    </t>

    <t t-if="!children.length &amp;&amp; !managers.length">
        <div class="alert alert-info" role="alert">
            <p><b>No hierarchy position.</b></p>
            <p>This employee has no manager or subordinate.</p>
            <p>In order to get an organigram, set a manager and save the record.</p>
        </div>
    </t>

    <div t-if="children.length" t-attf-class="o_org_chart_group_down position-relative #{managers.length &gt; 0 ? 'o_org_chart_has_managers' : ''}">
        <t t-foreach="children" t-as="employee" t-key="employee_index">
            <t t-set="emp_count" t-value="emp_count + 1"/>
            <t t-if="emp_count &lt; 20">
                <t t-call="hr_org_chart.hr_org_chart_employee">
                    <t t-set="employee_type" t-value="'sub'"/>
                </t>
            </t>
        </t>

        <t t-if="(children.length + managers.length) &gt; 19">
            <div class="o_org_chart_entry o_org_chart_more d-flex overflow-visible">
                <div class="o_media_left position-relative">
                    <a href="#"
                        t-att-data-employee-id="self.id"
                        t-att-data-employee-name="self.name"
                        class="o_org_chart_show_more o_employee_sub_redirect btn btn-link ps-2"
                        t-on-click.prevent="_onEmployeeSubRedirect">See All</a>
                </div>
            </div>
        </t>
    </div>
</t>

<t t-name="hr_org_chart.hr_orgchart_emp_popover">
    <div class="popover o_org_chart_popup" role="tooltip">
        <div class="tooltip-arrow">
            
        </div>
        <h3 class="popover-header">
            <div class="d-flex align-items-center">
                <span class="flex-shrink-0" t-att-style='"background-image:url(\"/web/image/hr.employee.public/" + props.employee.id + "/avatar_1024/\")"'/>
                <b class="flew-grow-1"><t t-esc="props.employee.name"/></b>
                <a href="#" class="ms-auto o_employee_redirect" t-att-data-employee-id="props.employee.id" t-on-click.prevent="() => this._onEmployeeRedirect(props.employee.id)"><i class="fa fa-external-link" role="img" aria-label='Redirect' title="Redirect"></i></a>
            </div>
        </h3>
        <div class="popover-body">
            <table class="table table-sm table-borderless mb-0">
                <tbody>
                    <tr>
                        <td class="text-end"><b t-esc="props.employee.direct_sub_count"/></td>
                        <td>
                            <a href="#" class="o_employee_sub_redirect" data-type='direct'
                                    t-att-data-employee-name="props.employee.name" t-att-data-employee-id="props.employee.id"
                                    t-on-click.prevent="_onEmployeeSubRedirect">
                                <b>Direct subordinates</b></a>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-end">
                            <b t-esc="props.employee.indirect_sub_count - props.employee.direct_sub_count"/>
                        </td>
                        <td>
                            <a href="#" class="o_employee_sub_redirect" data-type='indirect'
                                    t-att-data-employee-name="props.employee.name" t-att-data-employee-id="props.employee.id"
                                    t-on-click.prevent="_onEmployeeSubRedirect">
                                Indirect subordinates</a>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-end"><b t-esc="props.employee.indirect_sub_count"/></td>
                        <td>
                            <a href="#" class="o_employee_sub_redirect" data-type='total'
                                    t-att-data-employee-name="props.employee.name" t-att-data-employee-id="props.employee.id"
                                    t-on-click.prevent="_onEmployeeSubRedirect">
                                Total</a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</t>

</templates>

```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_card.js

```javascript
/** @odoo-module */

import { HierarchyCard } from "@web_hierarchy/hierarchy_card";

export class HrEmployeeHierarchyCard extends HierarchyCard {
    static template = "hr_org_chart.HrEmployeeHierarchyCard";
}

```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_card.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates>

    <t t-name="hr_org_chart.HrEmployeeHierarchyCard" t-inherit="web_hierarchy.HierarchyCard">
        <xpath expr="//button[@name='hierarchy_search_subsidiaries']" position="attributes">
            <attribute name="class" separator=" " remove="d-grid"/>
            <attribute name="class" separator=" " remove="rounded-0"/>
        </xpath>
        <xpath expr="//button[@name='hierarchy_search_subsidiaries']" position="inside">
            <t t-out="props.node.childResIds.length"/> people
        </xpath>
        <xpath expr="//button[@name='hierarchy_search_subsidiaries']/t[@t-if]" position="replace">
            <t t-if="!props.node.nodes.length">
                <i class="fa fa-fw fa-caret-right"/>
            </t>
        </xpath>
        <xpath expr="//button[@name='hierarchy_search_subsidiaries']/t[@t-else]" position="replace">
            <t t-else="">
                <i class="fa fa-fw fa-caret-down"/>
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="hr_org_chart.HierarchyView" t-inherit="web_hierarchy.HierarchyView" t-inherit-mode="primary">
        <t t-call="web.ActionHelper" position="replace">
            <t t-if="props.info.noContentHelp">
                <HrActionHelper
                    noContentTitle.translate="No Data"
                    noContentParagraph.translate="In the Organigram you will have a clear overview of the hierarchy of employees."
                />
            </t>
        </t>
    </t>
</odoo>
```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_renderer.js

```javascript
/** @odoo-module **/

import { Avatar } from "@mail/views/web/fields/avatar/avatar";

import { HierarchyRenderer } from "@web_hierarchy/hierarchy_renderer";
import { HrEmployeeHierarchyCard } from "./hr_employee_hierarchy_card";

export class HrEmployeeHierarchyRenderer extends HierarchyRenderer {
    static template = "hr_org_chart.HrEmployeeHierarchyRenderer";
    static components = {
        ...HierarchyRenderer.components,
        HierarchyCard: HrEmployeeHierarchyCard,
        Avatar,
    };
}

```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_renderer.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates>

    <t t-name="hr_org_chart.HrEmployeeHierarchyRenderer" t-inherit="web_hierarchy.HierarchyRenderer">
        <xpath expr="//div[hasclass('o_hierarchy_parent_node_container')]/span" position="replace">
            <Avatar
                resModel="row.parentNode.model.resModel"
                resId="row.parentNode.resId"
                displayName="row.parentNode.data.display_name || row.parentNode.data.name"
            />
        </xpath>
    </t>

</templates>

```

## File: static\src\views\hr_employee_hierarchy\hr_employee_hierarchy_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { hierarchyView } from "@web_hierarchy/hierarchy_view";
import { HrEmployeeHierarchyRenderer } from "./hr_employee_hierarchy_renderer";
import { HierarchyController } from "@web_hierarchy/hierarchy_controller";
import { HrActionHelper } from "@hr/views/hr_action_helper";

export class HrEmployeeHierarchyController extends HierarchyController {
    static template = "hr_org_chart.HierarchyView";
    static components = { ...HierarchyController.components, HrActionHelper };
}

export const hrEmployeeHierarchyView = {
    ...hierarchyView,
    Controller: HrEmployeeHierarchyController,
    Renderer: HrEmployeeHierarchyRenderer,
};

registry.category("views").add("hr_employee_hierarchy", hrEmployeeHierarchyView);

```

## File: views\hr_department_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="hr_department_hierarchy_view" model="ir.ui.view">
        <field name="name">hr.departmnent.view.hierarchy</field>
        <field name="model">hr.department</field>
        <field name="arch" type="xml">
            <hierarchy child_field="child_ids" draggable="1">
                <field name="name" />
                <field name="color" />
                <field name="total_employee" />
                <templates>
                    <t t-name="hierarchy-box">
                        <div t-attf-class="o_hierarchy_node_header d-flex justify-content-center align-items-center o_hierarchy_node_color_{{ record.color.raw_value }}">
                            <field name="name" />
                        </div>
                        <div class="o_hierarchy_node_body d-flex flex-column">
                            <field name="manager_id" class="pt-1 ps-1" widget="many2one_avatar_employee" />
                            <div>
                                <button class="btn btn-link" name="action_employee_from_department" type="object">
                                    <t t-out="record.total_employee.raw_value"/> Employees
                                </button>
                            </div>
                        </div>
                    </t>
                </templates>
            </hierarchy>
        </field>
    </record>

    <record id="act_hr_department_hierarchy_view_kanban_action" model="ir.actions.act_window.view">
        <field name="sequence" eval="25"/>
        <field name="view_mode">hierarchy</field>
        <field name="act_window_id" ref="hr.hr_department_kanban_action"/>
    </record>

    <record id="act_hr_department_hierarchy_view_tree_action" model="ir.actions.act_window.view">
        <field name="sequence" eval="25"/>
        <field name="view_mode">hierarchy</field>
        <field name="act_window_id" ref="hr.hr_department_tree_action"/>
    </record>

</odoo>

```

## File: views\hr_employee_public_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_public_hierarchy_view" model="ir.ui.view">
        <field name="name">hr.employee.public.hierarchy.view</field>
        <field name="model">hr.employee.public</field>
        <field name="arch" type="xml">
            <hierarchy child_field="child_ids" js_class="hr_employee_hierarchy" icon="fa-users" draggable="0">
                <field name="name" />
                <field name="job_id" />
                <field name="department_color" />
                <field name="hr_icon_display" />
                <field name="department_id" />
                <templates>
                    <t t-name="hierarchy-box">
                        <div t-attf-class="o_hierarchy_node_header d-flex justify-content-center pb-4 o_hierarchy_node_color_{{ record.department_color.raw_value }}"
                             t-att-title="record.department_id.value"
                        >
                            <field name="image_1024" preview_image="image_128" options="{'zoom': true, 'zoom_delay': 1000}" widget="background_image" />
                        </div>
                        <div class="d-flex flex-column text-center">
                            <div class="d-flex">
                                <field class="fw-bold w-100" name="name" />
                                <field name="hr_icon_display" class="flex-shrink-0" widget="hr_presence_status" />
                            </div>
                            <field name="job_title"/>
                        </div>
                    </t>
                </templates>
            </hierarchy>
        </field>
    </record>

    <record id="act_hr_employee_public_hierarchy_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="25"/>
        <field name="view_mode">hierarchy</field>
        <field name="act_window_id" ref="hr.hr_employee_public_action"/>
    </record>
</odoo>

```

## File: views\hr_org_chart_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem
        id="menu_hr_employee_org_chart"
        name="Org Chart"
        action="action_hr_employee_org_chart"
        groups="hr.group_hr_user"
        parent="hr.menu_hr_employee_payroll"
        sequence="5"
    />

</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_hr_employee_org_chart" model="ir.actions.act_window">
        <field name="name">Org Chart</field>
        <field name="res_model">hr.employee</field>
        <field name="path">org-chart</field>
        <field name="view_mode">hierarchy,kanban,list,form,activity,graph,pivot</field>
        <field name="domain">[]</field>
        <field name="context">{'chat_icon': True}</field>
        <field name="view_id" eval="False"/>
        <field name="search_view_id" ref="hr.view_employee_filter"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a new employee
            </p><p>
                Quickly find all the information you need for your employees such as contact data, job position, availability, etc.
            </p>
        </field>
    </record>

    <record id="hr_employee_view_form_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.org_chart</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="%(hr_org_chart.action_hr_employee_org_chart)d"
                    icon="fa-users"
                    type="action"
                    context="{'hierarchy_res_id': id}"
                    invisible="(not parent_id and not child_ids) or parent_id == id">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Org Chart</span>
                    </div>
                </button>
            </xpath>
            <div id="o_work_employee_main" position="after">
                <div id="o_employee_right" class="col-lg-4 px-0 ps-lg-5 pe-lg-0">
                    <separator string="Organization Chart"/>
                    <field name="child_ids" class="position-relative" widget="hr_org_chart" readonly="1" nolabel="1"/>
                </div>
            </div>
        </field>
    </record>
    
    <record id="hr_employee_view_pivot_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.view.pivot.inherit.org_chart</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.hr_employee_view_pivot"/>
        <field name="arch" type="xml">
            <xpath expr="//pivot" position="inside">
                <field name="department_color" type="measure" invisible="1"/>
            </xpath>
        </field>
    </record>
    
    <record id="hr_employee_view_graph_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.view.graph.inherit.org_chart</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.hr_employee_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//graph" position="inside">
                <field name="department_color" type="measure" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_hierarchy_view" model="ir.ui.view">
        <field name="name">hr.employee.view.hierarchy</field>
        <field name="model">hr.employee</field>
        <field name="arch" type="xml">
            <hierarchy child_field="child_ids" js_class="hr_employee_hierarchy" icon="fa-users" draggable="1">
                <field name="name" />
                <field name="job_id" />
                <field name="department_color" />
                <field name="hr_icon_display" />
                <field name="department_id" />
                <templates>
                    <t t-name="hierarchy-box">
                        <div t-attf-class="o_hierarchy_node_header d-flex justify-content-center pb-4 o_hierarchy_node_color_{{ record.department_color.raw_value }}"
                             t-att-title="record.department_id.value"
                        >
                            <field name="image_1024" preview_image="image_128" options="{'zoom': true, 'zoom_delay': 1000}" widget="background_image" />
                        </div>
                        <div class="d-flex flex-column text-center">
                            <div class="d-flex">
                                <field class="fw-bold w-100" name="name" />
                                <field name="hr_icon_display" class="flex-shrink-0" widget="hr_presence_status" />
                            </div>
                            <field name="job_title"/>
                        </div>
                    </t>
                </templates>
            </hierarchy>
        </field>
    </record>

    <record id="hr_employee_public_view_form_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.public.view.form.inherit.org_chart</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='o_work_employee_main']" position="after">
                <div id="o_employee_right" class="col-lg-4 px-0 ps-lg-5">
                    <separator string="Organization Chart"/>
                    <field name="child_ids" class="position-relative" widget="hr_org_chart" readonly="1" nolabel="1"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">res.users.preferences.view.form.inherit.org_chart</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <div id="o_work_employee_main" position="after">
                <div id="o_employee_right" class="col-lg-4 px-0 ps-lg-5">
                    <separator string="Organization Chart"/>
                    <field name="child_ids" class="position-relative" widget="hr_org_chart" readonly="1" nolabel="1"/>
                </div>
            </div>
        </field>
    </record>

    <record id="act_open_view_employee_list_my_hierarchy_view" model="ir.actions.act_window.view">
        <field name="sequence" eval="40"/>
        <field name="view_mode">hierarchy</field>
        <field name="act_window_id" ref="hr.open_view_employee_list_my"/>
    </record>
</odoo>

```

