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
    'depends': ['hr'],
    'auto_install': True,
    'data': [
        'views/hr_templates.xml',
        'views/hr_views.xml'
    ],
    'qweb': [
        'static/src/xml/hr_org_chart.xml',
    ],
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

        if 'allowed_company_ids' in request.env.context:
            cids = request.env.context['allowed_company_ids']
        else:
            cids = [request.env.company.id]

        Employee = request.env['hr.employee.public'].with_context(allowed_company_ids=cids)
        # check and raise
        if not Employee.check_access_rights('read', raise_exception=False):
            return None
        try:
            Employee.browse(employee_id).check_access_rule('read')
        except AccessError:
            return None
        else:
            return Employee.browse(employee_id)

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
        if request.env['hr.employee'].check_access_rights('read', raise_exception=False):
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


class HrEmployeePublic(models.Model):
    _inherit = ["hr.employee.public"]

    subordinate_ids = fields.One2many('hr.employee.public', string='Subordinates', compute='_compute_subordinates', help="Direct and indirect subordinates",
                                      compute_sudo=True)

```

## File: models\hr_org_chart_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    child_all_count = fields.Integer(
        'Indirect Subordinates Count',
        compute='_compute_subordinates', store=False,
        compute_sudo=True)

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
        for child in direct_subordinates:
            child_subordinate = child._get_subordinates(parents=parents)
            indirect_subordinates |= child_subordinate
        return indirect_subordinates | direct_subordinates


    @api.depends('child_ids', 'child_ids.child_all_count')
    def _compute_subordinates(self):
        for employee in self:
            employee.subordinate_ids = employee._get_subordinates()
            employee.child_all_count = len(employee.subordinate_ids)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_org_chart_mixin
from . import hr_employee

```

## File: static\src\js\hr_org_chart.js

```javascript
odoo.define('web.OrgChart', function (require) {
"use strict";

var AbstractField = require('web.AbstractField');
var concurrency = require('web.concurrency');
var core = require('web.core');
var field_registry = require('web.field_registry');
var session = require('web.session');

var QWeb = core.qweb;
var _t = core._t;

var FieldOrgChart = AbstractField.extend({

    events: {
        "click .o_employee_redirect": "_onEmployeeRedirect",
        "click .o_employee_sub_redirect": "_onEmployeeSubRedirect",
        "click .o_employee_more_managers": "_onEmployeeMoreManager"
    },
    /**
     * @constructor
     * @override
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.dm = new concurrency.DropMisordered();
        this.employee = null;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * Get the chart data through a rpc call.
     *
     * @private
     * @param {integer} employee_id
     * @returns {Promise}
     */
    _getOrgData: function () {
        var self = this;
        return this.dm.add(this._rpc({
            route: '/hr/get_org_chart',
            params: {
                employee_id: this.employee,
                context: session.user_context,
            },
        })).then(function (data) {
            return data;
        });
    },
    /**
     * Get subordonates of an employee through a rpc call.
     *
     * @private
     * @param {integer} employee_id
     * @returns {Promise}
     */
    _getSubordinatesData: function (employee_id, type) {
        return this.dm.add(this._rpc({
            route: '/hr/get_subordinates',
            params: {
                employee_id: employee_id,
                subordinates_type: type,
                context: session.user_context,
            },
        }));
    },
    /**
     * @override
     * @private
     */
    _render: function () {
        if (!this.recordData.id) {
            return this.$el.html(QWeb.render("hr_org_chart", {
                managers: [],
                children: [],
            }));
        }
        else if (!this.employee) {
            // the widget is either dispayed in the context of a hr.employee form or a res.users form
            this.employee = this.recordData.employee_ids !== undefined ? this.recordData.employee_ids.res_ids[0] : this.recordData.id;
        }

        var self = this;
        return this._getOrgData().then(function (orgData) {
            if (_.isEmpty(orgData)) {
                orgData = {
                    managers: [],
                    children: [],
                }
            }
            orgData.view_employee_id = self.recordData.id;
            self.$el.html(QWeb.render("hr_org_chart", orgData));
            self.$('[data-toggle="popover"]').each(function () {
                $(this).popover({
                    html: true,
                    title: function () {
                        var $title = $(QWeb.render('hr_orgchart_emp_popover_title', {
                            employee: {
                                name: $(this).data('emp-name'),
                                id: $(this).data('emp-id'),
                            },
                        }));
                        $title.on('click',
                            '.o_employee_redirect', _.bind(self._onEmployeeRedirect, self));
                        return $title;
                    },
                    container: this,
                    placement: 'left',
                    trigger: 'focus',
                    content: function () {
                        var $content = $(QWeb.render('hr_orgchart_emp_popover_content', {
                            employee: {
                                id: $(this).data('emp-id'),
                                name: $(this).data('emp-name'),
                                direct_sub_count: parseInt($(this).data('emp-dir-subs')),
                                indirect_sub_count: parseInt($(this).data('emp-ind-subs')),
                            },
                        }));
                        $content.on('click',
                            '.o_employee_sub_redirect', _.bind(self._onEmployeeSubRedirect, self));
                        return $content;
                    },
                    template: QWeb.render('hr_orgchart_emp_popover', {}),
                });
            });
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onEmployeeMoreManager: function(event) {
        event.preventDefault();
        this.employee = parseInt($(event.currentTarget).data('employee-id'));
        this._render();
    },
    /**
     * Redirect to the employee form view.
     *
     * @private
     * @param {MouseEvent} event
     * @returns {Promise} action loaded
     */
    _onEmployeeRedirect: function (event) {
        var self = this;
        event.preventDefault();
        var employee_id = parseInt($(event.currentTarget).data('employee-id'));
        return this._rpc({
            model: 'hr.employee',
            method: 'get_formview_action',
            args: [employee_id],
        }).then(function(action) {
            return self.do_action(action); 
        });
    },
    /**
     * Redirect to the sub employee form view.
     *
     * @private
     * @param {MouseEvent} event
     * @returns {Promise} action loaded
     */
    _onEmployeeSubRedirect: function (event) {
        event.preventDefault();
        var employee_id = parseInt($(event.currentTarget).data('employee-id'));
        var employee_name = $(event.currentTarget).data('employee-name');
        var type = $(event.currentTarget).data('type') || 'direct';
        var self = this;
        if (employee_id) {
            this._getSubordinatesData(employee_id, type).then(function(data) {
                var domain = [['id', 'in', data]];
                return self._rpc({
                    model: 'hr.employee',
                    method: 'get_formview_action',
                    args: [employee_id],
                }).then(function(action) {
                    action = _.extend(action, {
                        'name': _t('Team'),
                        'view_mode': 'kanban,list,form',
                        'views':  [[false, 'kanban'], [false, 'list'], [false, 'form']],
                        'domain': domain,
                        'context': {
                            'default_parent_id': employee_id,
                        }
                    });
                    delete action['res_id'];
                    return self.do_action(action); 
                });
            });
        }
    },
});

field_registry.add('hr_org_chart', FieldOrgChart);

return FieldOrgChart;

});

```

## File: static\src\xml\hr_org_chart.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="hr_org_chart_employee">
    <div t-attf-class="o_org_chart_entry o_org_chart_entry_#{employee_type} media">
        <t t-set="is_self" t-value="employee.id == view_employee_id"/>

        <div class="o_media_left">
            <!-- NOTE: Since by the default on not squared images odoo add white borders,
                use bg-images to get a clean and centred images -->
            <a t-if="! is_self"
                class="o_media_object rounded-circle o_employee_redirect"
                t-att-style="'background-image:url(\'/web/image/hr.employee.public/' + employee.id + '/image_1024/\')'"
                t-att-alt="employee.name"
                t-att-data-employee-id="employee.id"
                t-att-href="employee.link"/>
            <div t-if="is_self"
                class="o_media_object rounded-circle"
                t-att-style="'background-image:url(\'/web/image/hr.employee.public/' + employee.id + '/image_1024/\')'"/>
        </div>

        <div class="media-body">
            <span
                    t-if="employee.indirect_sub_count &gt; 0"
                    class="badge badge-pill"
                    tabindex="0"
                    data-trigger="focus"
                    t-att-data-emp-name="employee.name"
                    t-att-data-emp-id="employee.id"
                    t-att-data-emp-dir-subs="employee.direct_sub_count"
                    t-att-data-emp-ind-subs="employee.indirect_sub_count"
                    data-toggle="popover">
                <t t-esc="employee.indirect_sub_count"/>
            </span>

            <t t-if="!is_self">
                <a t-att-href="employee.link" class="o_employee_redirect" t-att-data-employee-id="employee.id">
                    <h5 class="o_media_heading"><b><t t-esc="employee.name"/></b></h5>
                    <strong><t t-esc="employee.job_title"/></strong>
                </a>
            </t>
            <t t-if="is_self">
                <h5 class="o_media_heading"><b><t t-esc="employee.name"/></b></h5>
                <strong><t t-esc="employee.job_title"/></strong>
            </t>
        </div>
    </div>
</t>

<t t-name="hr_org_chart">
    <!-- NOTE: Desidered behaviour:
            The maximun number of people is always 7 (including 'self'). Managers have priority over suburdinates
            Eg. 1 Manager + 1 self = show just 5 subordinates (if availables)
            Eg. 0 Manager + 1 self = show 6 subordinates (if available)

        -->
    <t t-set="emp_count" t-value="0"/>

    <div t-if='managers.length &gt; 0' class="o_org_chart_group_up">
        <t t-if='managers_more'>
            <div class="o_org_chart_entry o_org_chart_more media">
                <div class="o_media_left">
                    <a class="text-center o_employee_more_managers"
                            t-att-data-employee-id="managers[0].id">
                        <i t-attf-class="fa fa-angle-double-up" role="img" aria-label="More managers" title="More managers"/>
                    </a>
                </div>
            </div>
        </t>

        <t t-foreach="managers" t-as="employee">
            <t t-set="emp_count" t-value="emp_count + 1"/>
            <t t-call="hr_org_chart_employee">
                <t t-set="employee_type" t-value="'manager'"/>
            </t>
        </t>
    </div>

    <t t-if="children.length || managers.length" t-call="hr_org_chart_employee">
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

    <div t-if="children.length" class="o_org_chart_group_down">
        <t t-foreach="children" t-as="employee">
            <t t-set="emp_count" t-value="emp_count + 1"/>
            <t t-if="emp_count &lt; 20">
                <t t-call="hr_org_chart_employee">
                    <t t-set="employee_type" t-value="'sub'"/>
                </t>
            </t>
        </t>

        <t t-if="(children.length + managers.length) &gt; 19">
            <div class="o_org_chart_entry o_org_chart_more media">
                <div class="o_media_left">
                    <a href="#"
                        t-att-data-employee-id="self.id"
                        t-att-data-employee-name="self.name"
                        class="o_org_chart_show_more text-center o_employee_sub_redirect">See All</a>
                </div>
            </div>
        </t>
    </div>
</t>

<t t-name="hr_orgchart_emp_popover">
    <div class="popover o_org_chart_popup" role="tooltip"><div class="arrow"></div><h3 class="popover-header"></h3><div class="popover-body"></div></div>
</t>

<t t-name="hr_orgchart_emp_popover_content">
    <table class="table table-sm">
        <thead>
            <td class="text-right"><t t-esc="employee.direct_sub_count"/></td>
            <td>
                <a href="#" class="o_employee_sub_redirect" data-type='direct'
                        t-att-data-employee-name="employee.name" t-att-data-employee-id="employee.id">
                    <b>Direct subordinates</b></a>
            </td>
        </thead>
        <tbody>
            <tr>
                <td class="text-right">
                    <t t-esc="employee.indirect_sub_count - employee.direct_sub_count"/>
                </td>
                <td>
                    <a href="#" class="o_employee_sub_redirect" data-type='indirect'
                            t-att-data-employee-name="employee.name" t-att-data-employee-id="employee.id">
                        Indirect subordinates</a>
                </td>
            </tr>
            <tr>
                <td class="text-right"><t t-esc="employee.indirect_sub_count"/></td>
                <td>
                    <a href="#" class="o_employee_sub_redirect" data-type='total'
                            t-att-data-employee-name="employee.name" t-att-data-employee-id="employee.id">
                        Total</a>
                </td>
            </tr>
        </tbody>
    </table>
</t>

<t t-name="hr_orgchart_emp_popover_title">
    <div>
        <span t-att-style='"background-image:url(\"/web/image/hr.employee.public/" + employee.id + "/image_1024/\")"'/>
        <a href="#" class="float-right o_employee_redirect" t-att-data-employee-id="employee.id"><i class="fa fa-external-link" role="img" aria-label='Redirect' title="Redirect"></i></a>
        <b><t t-esc="employee.name"/></b>
    </div>
</t>

</templates>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="_assets_primary_variables" inherit_id="web._assets_primary_variables">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/hr_org_chart/static/src/scss/variables.scss"/>
        </xpath>
    </template>

    <template id="assets_backend" name="hr_org_chart_assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr_org_chart/static/src/scss/hr_org_chart.scss"/>
            <script type="text/javascript" src="/hr_org_chart/static/src/js/hr_org_chart.js"></script>
        </xpath>
    </template>

    <template id="qunit_suite" inherit_id="web.qunit_suite_tests">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/hr_org_chart/static/tests/hr_org_chart_tests.js"/>
        </xpath>
    </template>
</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_form_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.org_chart</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <div id="o_work_employee_main" position="after">
                <div id="o_employee_right">
                    <h4 class="o_org_chart_title mb16 mt0">Organization Chart</h4>
                    <field name="child_ids" widget="hr_org_chart" readonly="1"/>
                </div>
            </div>
        </field>
    </record>

    <record id="hr_employee_public_view_form_inherit_org_chart" model="ir.ui.view">
        <field name="name">hr.employee.public.view.form.inherit.org_chart</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='o_work_employee_main']" position="after">
                <div id="o_employee_right">
                    <h4 class="o_org_chart_title mb16 mt0">Organization Chart</h4>
                    <field name="child_ids" widget="hr_org_chart" readonly="1"/>
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
                <div id="o_employee_right">
                    <field name="employee_ids" invisible="1"/>
                    <h4 class="o_org_chart_title mb16 mt0">Organization Chart</h4>
                    <field name="child_ids" widget="hr_org_chart"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

