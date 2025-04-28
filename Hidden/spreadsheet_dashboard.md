# Odoo Module: spreadsheet_dashboard

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Spreadsheet dashboard",
    "version": "1.0",
    "category": "Hidden",
    "summary": "Spreadsheet",
    "description": "Spreadsheet",
    "depends": ["spreadsheet"],
    "demo": [],
    "installable": True,
    "auto_install": False,
    "license": "LGPL-3",
    "data": [
        "security/security.xml",
        "security/ir.model.access.csv",
        "views/spreadsheet_dashboard_views.xml",
        "views/menu_views.xml",
        "data/dashboard.xml",
    ],
    "assets": {
        "spreadsheet.o_spreadsheet": [
            "spreadsheet_dashboard/static/src/bundle/**/*.js",
            "spreadsheet_dashboard/static/src/bundle/**/*.xml",
        ],
        "web.assets_backend": [
            "spreadsheet_dashboard/static/src/assets/**/*.js",
            "spreadsheet_dashboard/static/src/**/*.scss",
        ],
        "web.qunit_suite_tests": [
            "spreadsheet_dashboard/static/tests/**/*",
            ("include", "spreadsheet.o_spreadsheet"),
            ("remove", "spreadsheet_dashboard/static/tests/mobile/**/*.js"),
        ],
        "web.qunit_mobile_suite_tests": [
            "spreadsheet_dashboard/static/tests/mobile/**/*.js",
            "spreadsheet_dashboard/static/tests/utils/**/*.js",
            ("include", "spreadsheet.o_spreadsheet"),
        ],
    },
}

```

## File: data\dashboard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="spreadsheet_dashboard_group_finance" model="spreadsheet.dashboard.group">
        <field name="name">Finance</field>
        <field name="sequence">300</field>
    </record>

    <record id="spreadsheet_dashboard_group_sales" model="spreadsheet.dashboard.group">
        <field name="name">Sales</field>
        <field name="sequence">100</field>
    </record>

    <record id="spreadsheet_dashboard_group_hr" model="spreadsheet.dashboard.group">
        <field name="name">Human Resources</field>
        <field name="sequence">800</field>
    </record>

    <record id="spreadsheet_dashboard_group_website" model="spreadsheet.dashboard.group">
        <field name="name">Website</field>
        <field name="sequence">700</field>
    </record>

    <record id="spreadsheet_dashboard_group_project" model="spreadsheet.dashboard.group">
        <field name="name">Project</field>
        <field name="sequence">500</field>
    </record>

    <record id="spreadsheet_dashboard_group_logistics" model="spreadsheet.dashboard.group">
        <field name="name">Logistics</field>
        <field name="sequence">400</field>
    </record>

</odoo>

```

## File: models\spreadsheet_dashboard.py

```python
import base64
import json

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

class SpreadsheetDashboard(models.Model):
    _name = 'spreadsheet.dashboard'
    _description = 'Spreadsheet Dashboard'
    _order = 'sequence'

    name = fields.Char(required=True)
    dashboard_group_id = fields.Many2one('spreadsheet.dashboard.group', required=True)
    data = fields.Binary(required=True, default=lambda self: self._default_data())
    raw = fields.Binary(compute='_compute_raw', inverse='_inverse_raw')
    thumbnail = fields.Binary()
    sequence = fields.Integer()
    group_ids = fields.Many2many('res.groups', default=lambda self: self.env.ref('base.group_user'))

    def _default_data(self):
        data = json.dumps(self._empty_workbook_data())
        return base64.b64encode(data.encode())

    def _empty_workbook_data(self):
        """Create an empty spreadsheet workbook.
        The sheet name should be the same for all users to allow consistent references
        in formulas. It is translated for the user creating the spreadsheet.
        """
        return {
            "version": 1,
            "sheets": [
                {
                    "id": "sheet1",
                    "name": _("Sheet1"),
                }
            ]
        }

    @api.depends('data')
    def _compute_raw(self):
        for dashboard in self:
            dashboard.raw = base64.decodebytes(dashboard.data)

    def _inverse_raw(self):
        for dashboard in self:
            dashboard.data = base64.encodebytes(dashboard.raw)

    @api.onchange('data')
    def _onchange_data_(self):
        if self.data:
            try:
                data_str = base64.b64decode(self.data).decode('utf-8')
                json.loads(data_str)
            except:
                raise ValidationError(_('Invalid JSON Data'))

    def copy(self, default=None):
        self.ensure_one()
        if default is None:
            default = {}
        if 'name' not in default:
            default['name'] = _("%s (copy)") % self.name
        return super().copy(default=default)

```

## File: models\spreadsheet_dashboard_group.py

```python
from odoo import fields, models, api, _
from odoo.exceptions import UserError


class SpreadsheetDashboardGroup(models.Model):
    _name = 'spreadsheet.dashboard.group'
    _description = 'Group of dashboards'
    _order = 'sequence'

    name = fields.Char(required=True)
    dashboard_ids = fields.One2many('spreadsheet.dashboard', 'dashboard_group_id')
    sequence = fields.Integer()

    @api.ondelete(at_uninstall=False)
    def _unlink_except_spreadsheet_data(self):
        external_ids = self.get_external_id()
        for group in self:
            external_id = external_ids[group.id]
            if external_id and not external_id.startswith('__export__'):
                raise UserError(_("You cannot delete %s as it is used in another module.", group.name))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import spreadsheet_dashboard_group
from . import spreadsheet_dashboard

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_spreadsheet_dashboard_group_user","access_spreadsheet_dashboard_group_user","model_spreadsheet_dashboard_group","base.group_user",1,0,0,0
spreadsheet_dashboard_user","spreadsheet_dashboard_user","model_spreadsheet_dashboard","base.group_user",1,0,0,0
"access_spreadsheet_dashboard_group","access_spreadsheet_dashboard_group","model_spreadsheet_dashboard_group","base.group_system",1,1,1,1
"spreadsheet_dashboard","spreadsheet_dashboard","model_spreadsheet_dashboard","base.group_system",1,1,1,1

```

## File: security\security.xml

```xml
<odoo>
    <record id="ir_rule_spreadsheet_dashboard" model="ir.rule">
        <field name="name">Spreadsheet dashboard: groups</field>
        <field name="model_id" ref="model_spreadsheet_dashboard"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        <field name="domain_force">[('group_ids', 'in', user.groups_id.ids)]</field>
    </record>
</odoo>


```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#CD7690"/>
            <stop offset="100%" stop-color="#CA5377"/>
        </linearGradient>
        <path id="icon-d" d="M18.0450069,57.225 C16.6239398,57.2249541 15.319401,56.4292666 14.6550625,55.1573449 C12.9601701,51.9125391 12,48.2137078 12,44.2875 C12,31.4261695 22.2974514,21 35,21 C47.7025486,21 58,31.4261695 58,44.2875 C58,48.2137078 57.0398299,51.9125391 55.3449375,55.1573449 C54.6806259,56.4292924 53.3760701,57.2249902 51.9549931,57.225 L18.0450069,57.225 Z M52.8888889,41.7 C51.4775035,41.7 50.3333333,42.8584723 50.3333333,44.2875 C50.3333333,45.7165277 51.4775035,46.875 52.8888889,46.875 C54.3002743,46.875 55.4444444,45.7165277 55.4444444,44.2875 C55.4444444,42.8584723 54.3002743,41.7 52.8888889,41.7 Z M35,28.7625 C36.4113854,28.7625 37.5555556,27.6040277 37.5555556,26.175 C37.5555556,24.7459723 36.4113854,23.5875 35,23.5875 C33.5886146,23.5875 32.4444444,24.7459723 32.4444444,26.175 C32.4444444,27.6040277 33.5886146,28.7625 35,28.7625 Z M17.1111111,41.7 C15.6997257,41.7 14.5555556,42.8584723 14.5555556,44.2875 C14.5555556,45.7165277 15.6997257,46.875 17.1111111,46.875 C18.5224965,46.875 19.6666667,45.7165277 19.6666667,44.2875 C19.6666667,42.8584723 18.5224965,41.7 17.1111111,41.7 Z M22.3506389,28.8925219 C20.9392535,28.8925219 19.7950833,30.0509941 19.7950833,31.4800219 C19.7950833,32.9090496 20.9392535,34.0675219 22.3506389,34.0675219 C23.7620243,34.0675219 24.9061944,32.9090496 24.9061944,31.4800219 C24.9061944,30.0509941 23.7620243,28.8925219 22.3506389,28.8925219 Z M47.6493611,28.8925219 C46.2379757,28.8925219 45.0938056,30.0509941 45.0938056,31.4800219 C45.0938056,32.9090496 46.2379757,34.0675219 47.6493611,34.0675219 C49.0607465,34.0675219 50.2049167,32.9090496 50.2049167,31.4800219 C50.2049167,30.0509941 49.0607465,28.8925219 47.6493611,28.8925219 Z M40.6952153,31.4423414 C39.686809,31.1156695 38.6082049,31.6784508 38.285566,32.6992195 L34.6181042,44.3034293 C31.9739028,44.501373 29.8888889,46.7346281 29.8888889,49.4625 C29.8888889,52.3205555 32.1772292,54.6375 35,54.6375 C37.8227708,54.6375 40.1111111,52.3205555 40.1111111,49.4625 C40.1111111,47.8636676 39.3946771,46.434559 38.269434,45.4852699 L41.9365764,33.8821113 C42.2591354,32.8612617 41.7033819,31.7690133 40.6952153,31.4423414 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M4,50 C2,50 -7.10542736e-15,49.851312 0,45.8367347 L0,26.3942795 L16.3536575,8.86200565 C29.4512192,-0.488174988 39.6666667,-2.3877551 47,3.16326531 C54.3333333,8.71428571 58,14.9591837 58,21.8979592 C55.8677728,29.7827578 54.7719047,33.7755585 54.7123959,33.8763613 C54.6528871,33.9771642 49.9857922,39.3517104 40.7111111,50 L4,50 Z" opacity=".324" transform="translate(0 20)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <path fill="#FFF" fill-rule="nonzero" d="M18.0450069,55.225 C16.6239398,55.2249541 15.319401,54.4292666 14.6550625,53.1573449 C12.9601701,49.9125391 12,46.2137078 12,42.2875 C12,29.4261695 22.2974514,19 35,19 C47.7025486,19 58,29.4261695 58,42.2875 C58,46.2137078 57.0398299,49.9125391 55.3449375,53.1573449 C54.6806259,54.4292924 53.3760701,55.2249902 51.9549931,55.225 L18.0450069,55.225 Z M52.8888889,39.7 C51.4775035,39.7 50.3333333,40.8584723 50.3333333,42.2875 C50.3333333,43.7165277 51.4775035,44.875 52.8888889,44.875 C54.3002743,44.875 55.4444444,43.7165277 55.4444444,42.2875 C55.4444444,40.8584723 54.3002743,39.7 52.8888889,39.7 Z M35,26.7625 C36.4113854,26.7625 37.5555556,25.6040277 37.5555556,24.175 C37.5555556,22.7459723 36.4113854,21.5875 35,21.5875 C33.5886146,21.5875 32.4444444,22.7459723 32.4444444,24.175 C32.4444444,25.6040277 33.5886146,26.7625 35,26.7625 Z M17.1111111,39.7 C15.6997257,39.7 14.5555556,40.8584723 14.5555556,42.2875 C14.5555556,43.7165277 15.6997257,44.875 17.1111111,44.875 C18.5224965,44.875 19.6666667,43.7165277 19.6666667,42.2875 C19.6666667,40.8584723 18.5224965,39.7 17.1111111,39.7 Z M22.3506389,26.8925219 C20.9392535,26.8925219 19.7950833,28.0509941 19.7950833,29.4800219 C19.7950833,30.9090496 20.9392535,32.0675219 22.3506389,32.0675219 C23.7620243,32.0675219 24.9061944,30.9090496 24.9061944,29.4800219 C24.9061944,28.0509941 23.7620243,26.8925219 22.3506389,26.8925219 Z M47.6493611,26.8925219 C46.2379757,26.8925219 45.0938056,28.0509941 45.0938056,29.4800219 C45.0938056,30.9090496 46.2379757,32.0675219 47.6493611,32.0675219 C49.0607465,32.0675219 50.2049167,30.9090496 50.2049167,29.4800219 C50.2049167,28.0509941 49.0607465,26.8925219 47.6493611,26.8925219 Z M40.6952153,29.4423414 C39.686809,29.1156695 38.6082049,29.6784508 38.285566,30.6992195 L34.6181042,42.3034293 C31.9739028,42.501373 29.8888889,44.7346281 29.8888889,47.4625 C29.8888889,50.3205555 32.1772292,52.6375 35,52.6375 C37.8227708,52.6375 40.1111111,50.3205555 40.1111111,47.4625 C40.1111111,45.8636676 39.3946771,44.434559 38.269434,43.4852699 L41.9365764,31.8821113 C42.2591354,30.8612617 41.7033819,29.7690133 40.6952153,29.4423414 Z"/>
        </g>
    </g>
</svg>

```

## File: static\src\assets\dashboard_action_loader.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { loadSpreadsheetAction } from "@spreadsheet/assets_backend/spreadsheet_action_loader";

const actionRegistry = registry.category("actions");

const loadDashboardAction = async (env, context) => {
    await loadSpreadsheetAction(env, "action_spreadsheet_dashboard", loadDashboardAction);
    return {
        ...context,
        target: "current",
        tag: "action_spreadsheet_dashboard",
        type: "ir.actions.client",
    };
};

actionRegistry.add("action_spreadsheet_dashboard", loadDashboardAction);

```

## File: static\src\bundle\dashboard_action\dashboard_action.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { ControlPanel } from "@web/search/control_panel/control_panel";
import { DashboardLoader, Status } from "./dashboard_loader";
import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";
import { useSetupAction } from "@web/webclient/actions/action_hook";
import { DashboardMobileSearchPanel } from "./mobile_search_panel/mobile_search_panel";
import { MobileFigureContainer } from "./mobile_figure_container/mobile_figure_container";
import { FilterValue } from "@spreadsheet/global_filters/components/filter_value/filter_value";
import { loadSpreadsheetDependencies } from "@spreadsheet/helpers/helpers";
import { useService } from "@web/core/utils/hooks";

const { Spreadsheet } = spreadsheet;
const { Component, onWillStart, useState, useEffect } = owl;

export class SpreadsheetDashboardAction extends Component {
    setup() {
        this.Status = Status;
        this.controlPanelDisplay = {
            "top-left": true,
            "top-right": true,
            "bottom-left": false,
            "bottom-right": false,
        };
        this.orm = useService("orm");
        this.router = useService("router");
        // Use the non-protected orm service (`this.env.services.orm` instead of `useService("orm")`)
        // because spreadsheets models are preserved across multiple components when navigating
        // with the breadcrumb
        // TODO write a test
        /** @type {DashboardLoader}*/
        this.loader = useState(
            new DashboardLoader(this.env, this.env.services.orm, this._fetchDashboardData)
        );
        onWillStart(async () => {
            await loadSpreadsheetDependencies();
            if (this.props.state && this.props.state.dashboardLoader) {
                const { groups, dashboards } = this.props.state.dashboardLoader;
                this.loader.restoreFromState(groups, dashboards);
            } else {
                await this.loader.load();
            }
            const activeDashboardId = this.getInitialActiveDashboard();
            if (activeDashboardId) {
                this.openDashboard(activeDashboardId);
            }
        });
        useEffect(
            () => this.router.pushState({ dashboard_id: this.activeDashboardId }),
            () => [this.activeDashboardId]
        );
        useEffect(
            () => {
                const dashboard = this.state.activeDashboard;
                if (dashboard && dashboard.status === Status.Loaded) {
                    const render = () => this.render(true);
                    dashboard.model.on("update", this, render);
                    return () => dashboard.model.off("update", this, render);
                }
            },
            () => {
                const dashboard = this.state.activeDashboard;
                return [dashboard && dashboard.model, dashboard && dashboard.status];
            }
        );
        useSetupAction({
            getLocalState: () => {
                return {
                    activeDashboardId: this.activeDashboardId,
                    dashboardLoader: this.loader.getState(),
                };
            },
        });
        /** @type {{ activeDashboard: import("./dashboard_loader").Dashboard}} */
        this.state = useState({ activeDashboard: undefined });
    }

    /**
     * @returns {number | undefined}
     */
    get activeDashboardId() {
        return this.state.activeDashboard ? this.state.activeDashboard.id : undefined;
    }

    /**
     * @returns {object[]}
     */
    get filters() {
        const dashboard = this.state.activeDashboard;
        if (!dashboard || dashboard.status !== Status.Loaded) {
            return [];
        }
        return dashboard.model.getters.getGlobalFilters();
    }

    /**
     * @private
     * @returns {number | undefined}
     */
    getInitialActiveDashboard() {
        if (this.props.state && this.props.state.activeDashboardId) {
            return this.props.state.activeDashboardId;
        }
        const params = this.props.action.params || this.props.action.context.params;
        if (params && params.dashboard_id) {
            return params.dashboard_id;
        }
        const [firstSection] = this.getDashboardGroups();
        if (firstSection && firstSection.dashboards.length) {
            return firstSection.dashboards[0].id;
        }
    }

    getDashboardGroups() {
        return this.loader.getDashboardGroups();
    }

    /**
     * @param {number} dashboardId
     */
    openDashboard(dashboardId) {
        this.state.activeDashboard = this.loader.getDashboard(dashboardId);
    }

    /**
     * @private
     * @param {number} dashboardId
     * @returns {Promise<{ data: string, revisions: object[] }>}
     */
    async _fetchDashboardData(dashboardId) {
        const [record] = await this.orm.read("spreadsheet.dashboard", [dashboardId], ["raw"]);
        return { data: record.raw, revisions: [] };
    }
}
SpreadsheetDashboardAction.template = "spreadsheet_dashboard.DashboardAction";
SpreadsheetDashboardAction.components = {
    ControlPanel,
    Spreadsheet,
    FilterValue,
    DashboardMobileSearchPanel,
    MobileFigureContainer,
};

registry
    .category("actions")
    .add("action_spreadsheet_dashboard", SpreadsheetDashboardAction, { force: true });

```

## File: static\src\bundle\dashboard_action\dashboard_action.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="spreadsheet_dashboard.DashboardAction" owl="1" class="o_action o_spreadsheet_dashboard_action o_field_highlight">
        <ControlPanel display="controlPanelDisplay">
            <t t-set-slot="control-panel-top-right">
                <t t-set="status" t-value="state.activeDashboard and state.activeDashboard.status"/>
                <div t-if="status === Status.Loaded"
                    class="o_filter_value_container"
                    t-foreach="filters"
                    t-key="activeDashboardId + '_' + filter.id"
                    t-as="filter">
                    <FilterValue
                        filter="filter"
                        model="state.activeDashboard.model"
                        showTitle="true"
                    />
                </div>
            </t>
        </ControlPanel>
        <t t-set="dashboard" t-value="state.activeDashboard"/>
        <div class="o_content o_component_with_search_panel" t-att-class="{ o_mobile_dashboard: env.isSmall }">
            <!-- Dashboard selection -->
            <t t-if="env.isSmall">
                <DashboardMobileSearchPanel
                    onDashboardSelected="(dashboardId) => this.openDashboard(dashboardId)"
                    activeDashboard="dashboard"
                    groups="getDashboardGroups()"/>
            </t>
            <t t-else="">
                <div class="o_spreadsheet_dashboard_search_panel o_search_panel flex-grow-0 border-end flex-shrink-0 pe-2 pb-5 ps-4 h-100 bg-view overflow-auto">
                    <section t-foreach="getDashboardGroups()" t-as="group" t-key="group.id" class="o_search_panel_section o_search_panel_category">
                        <header class="o_search_panel_section_header pt-4 pb-2 text-uppercase o_cursor_default user-select-none">
                            <b t-esc="group.name"/>
                        </header>
                        <ul class="list-group d-block o_search_panel_field">
                            <li t-foreach="group.dashboards" t-as="dashboard" t-key="dashboard.id"
                                t-on-click="() => this.openDashboard(dashboard.id)"
                                t-esc="dashboard.displayName"
                                t-att-data-name="dashboard.displayName"
                                class="o_search_panel_category_value list-group-item cursor-pointer border-0"
                                t-att-class="{'active': dashboard.id === state.activeDashboard.id}"/>
                        </ul>
                    </section>
                </div>
            </t>
            <!-- Main content -->
            <h3 t-if="!dashboard" class="dashboard-loading-status">No available dashboard</h3>
            <t t-else="">
                <t t-set="status" t-value="dashboard.status"/>
                <h3 t-if="status === Status.Loading" class="dashboard-loading-status">Loading...</h3>
                <div t-elif="status === Status.Error" class="dashboard-loading-status error">
                    An error occured while loading the dashboard
                </div>
                <t t-else="">
                    <MobileFigureContainer t-if="env.isSmall" spreadsheetModel="dashboard.model" t-key="dashboard.id"/>
                    <div t-else="" class="o_renderer">
                        <Spreadsheet
                            model="dashboard.model"
                            t-key="dashboard.id"/>
                    </div>
                </t>
            </t>
        </div>
    </div>
</templates>

```

## File: static\src\bundle\dashboard_action\dashboard_loader.js

```javascript
/** @odoo-module */

import { DataSources } from "@spreadsheet/data_sources/data_sources";
import { migrate } from "@spreadsheet/o_spreadsheet/migration";
import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";

const { Model } = spreadsheet;

/**
 * @type {{
 *  NotLoaded: "NotLoaded",
 *  Loading: "Loading",
 *  Loaded: "Loaded",
 *  Error: "Error",
 * }}
 */
export const Status = {
    NotLoaded: "NotLoaded",
    Loading: "Loading",
    Loaded: "Loaded",
    Error: "Error",
};

/**
 * @typedef Dashboard
 * @property {number} id
 * @property {string} displayName
 * @property {string} status
 * @property {Model} [model]
 * @property {Error} [error]
 *
 * @typedef DashboardGroupData
 * @property {number} id
 * @property {string} name
 * @property {Array<number>} dashboardIds
 *
 * @typedef DashboardGroup
 * @property {number} id
 * @property {string} name
 * @property {Array<Dashboard>} dashboards
 *
 * @typedef {(dashboardId: number) => Promise<{ data: string, revisions: object[] }>} FetchDashboardData
 *
 * @typedef {import("@web/env").OdooEnv} OdooEnv
 *
 * @typedef {import("@web/core/orm_service").ORM} ORM
 */

export class DashboardLoader {
    /**
     * @param {OdooEnv} env
     * @param {ORM} orm
     * @param {FetchDashboardData} fetchDashboardData
     */
    constructor(env, orm, fetchDashboardData) {
        /** @private */
        this.env = env;
        /** @private */
        this.orm = orm;
        /** @private @type {Array<DashboardGroupData>} */
        this.groups = [];
        /** @private @type {Object<number, Dashboard>} */
        this.dashboards = {};
        /** @private */
        this.fetchDashboardData = fetchDashboardData;
    }

    /**
     * @param {Array<DashboardGroupData>} groups
     * @param {Object<number, Dashboard>} dashboards
     */
    restoreFromState(groups, dashboards) {
        this.groups = groups;
        this.dashboards = dashboards;
    }

    /**
     * Return data needed to restore a dashboard loader
     */
    getState() {
        return {
            groups: this.groups,
            dashboards: this.dashboards,
        };
    }

    async load() {
        const groups = await this._fetchGroups();
        this.groups = groups
            .filter((group) => group.dashboard_ids.length)
            .map((group) => ({
                id: group.id,
                name: group.name,
                dashboardIds: group.dashboard_ids,
            }));
        const dashboards = await this._fetchDashboardNames(this.groups);
        for (const dashboard of dashboards) {
            this.dashboards[dashboard.id] = {
                id: dashboard.id,
                displayName: dashboard.name,
                status: Status.NotLoaded,
            };
        }
    }

    /**
     * @param {number} dashboardId
     * @returns {Dashboard}
     */
    getDashboard(dashboardId) {
        const dashboard = this._getDashboard(dashboardId);
        if (dashboard.status === Status.NotLoaded) {
            dashboard.promise = this._loadDashboardData(dashboardId);
        }
        return dashboard;
    }

    /**
     * @returns {Array<DashboardGroup>}
     */
    getDashboardGroups() {
        return this.groups.map((section) => ({
            id: section.id,
            name: section.name,
            dashboards: section.dashboardIds.map((dashboardId) => ({
                id: dashboardId,
                displayName: this._getDashboard(dashboardId).displayName,
                status: this._getDashboard(dashboardId).status,
            })),
        }));
    }

    /**
     * @private
     * @returns {Promise<{id: number, name: string, dashboard_ids: number[]}[]>}
     */
    _fetchGroups() {
        return this.orm.searchRead(
            "spreadsheet.dashboard.group",
            [["dashboard_ids", "!=", false]],
            ["id", "name", "dashboard_ids"]
        );
    }

    /**
     * @private
     * @param {Array<DashboardGroupData>} groups
     * @returns {Promise}
     */
    _fetchDashboardNames(groups) {
        return this.orm.read(
            "spreadsheet.dashboard",
            groups.map((group) => group.dashboardIds).flat(),
            ["name"]
        );
    }

    /**
     * @private
     * @param {number} id
     * @returns {Dashboard}
     */
    _getDashboard(id) {
        if (!this.dashboards[id]) {
            this.dashboards[id] = { status: Status.NotLoaded, id, displayName: "" };
        }
        return this.dashboards[id];
    }

    /**
     * @private
     * @param {number} dashboardId
     */
    async _loadDashboardData(dashboardId) {
        const dashboard = this._getDashboard(dashboardId);
        dashboard.status = Status.Loading;
        try {
            const { data, revisions } = await this.fetchDashboardData(dashboardId);
            dashboard.model = this._createSpreadsheetModel(data, revisions);
            dashboard.status = Status.Loaded;
        } catch (error) {
            dashboard.error = error;
            dashboard.status = Status.Error;
            throw error;
        }
    }

    /**
     * Activate the first sheet of a model
     *
     * @param {Model} model
     */
    _activateFirstSheet(model) {
        const sheetId = model.getters.getActiveSheetId();
        const firstSheetId = model.getters.getSheetIds()[0];
        if (firstSheetId !== sheetId) {
            model.dispatch("ACTIVATE_SHEET", {
                sheetIdFrom: sheetId,
                sheetIdTo: firstSheetId,
            });
        }
    }

    /**
     * @private
     * @param {string} data
     * @param {object[]} revisions
     * @returns {Model}
     */
    _createSpreadsheetModel(data, revisions = []) {
        const dataSources = new DataSources(this.orm);
        const model = new Model(
            migrate(JSON.parse(data)),
            {
                evalContext: { env: this.env, orm: this.orm },
                mode: "dashboard",
                dataSources,
            },
            revisions
        );
        this._activateFirstSheet(model);
        dataSources.addEventListener("data-source-updated", () => model.dispatch("EVALUATE_CELLS"));
        return model;
    }
}

```

## File: static\src\bundle\dashboard_action\mobile_figure_container\mobile_figure_container.js

```javascript
/** @odoo-module */

import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";

const { Component, useSubEnv } = owl;
const { registries } = spreadsheet;
const { figureRegistry } = registries;

export class MobileFigureContainer extends Component {
    setup() {
        useSubEnv({
            model: this.props.spreadsheetModel,
            isDashboard: () => this.props.spreadsheetModel.getters.isDashboard(),
        });
    }

    get figures() {
        const sheetId = this.props.spreadsheetModel.getters.getActiveSheetId();
        return this.props.spreadsheetModel.getters
            .getFigures(sheetId)
            .sort((f1, f2) => (this.isBefore(f1, f2) ? -1 : 1))
            .map((figure) => ({
                ...figure,
                width: window.innerWidth,
            }));
    }

    getFigureComponent(figure) {
        return figureRegistry.get(figure.tag).Component;
    }

    isBefore(f1, f2) {
        // TODO be smarter
        return f1.x < f2.x ? f1.y < f2.y : f1.y < f2.y;
    }
}

MobileFigureContainer.template = "documents_spreadsheet.MobileFigureContainer";

```

## File: static\src\bundle\dashboard_action\mobile_figure_container\mobile_figure_container.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <t t-name="documents_spreadsheet.MobileFigureContainer" owl="1">
        <t t-if="!figures.length">
            Only chart figures are displayed in small screens but this dashboard doesn't contain any
        </t>
        <div
            t-foreach="figures" t-as="figure"
            t-key="figure.id"
            t-attf-style="min-height: #{figure.height}px;"
        >
            <t t-component="getFigureComponent(figure)" figure="figure"/>
        </div>
    </t>
</templates>


```

## File: static\src\bundle\dashboard_action\mobile_search_panel\mobile_search_panel.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";

const { Component, useState } = owl;

export class DashboardMobileSearchPanel extends Component {
    setup() {
        this.state = useState({ isOpen: false });
    }

    get searchBarText() {
        return this.props.activeDashboard
            ? this.props.activeDashboard.displayName
            : _t("Choose a dashboard....");
    }

    onDashboardSelected(dashboardId) {
        this.props.onDashboardSelected(dashboardId);
        this.state.isOpen = false;
    }

    openDashboardSelection() {
        const dashboards = this.props.groups.map((group) => group.dashboards).flat();
        if (dashboards.length > 1) {
            this.state.isOpen = true;
        }
    }
}

DashboardMobileSearchPanel.template = "documents_spreadsheet.DashboardMobileSearchPanel";
DashboardMobileSearchPanel.props = {
    /**
     * (dashboardId: number) => void
     */
    onDashboardSelected: Function,
    groups: Object,
    activeDashboard: {
        type: Object,
        optional: true,
    },
};

```

## File: static\src\bundle\dashboard_action\mobile_search_panel\mobile_search_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="documents_spreadsheet.DashboardMobileSearchPanel" class="o_search_panel o_search_panel_summary btn w-100 overflow-visible" owl="1">
        <t t-if="state.isOpen">
            <t t-portal="'body'">
                <div class="o_spreadsheet_dashboard_search_panel o_search_panel o_searchview o_mobile_search">
                    <div class="o_mobile_search_header">
                        <button type="button" class="o_mobile_search_button btn" t-on-click="() => this.state.isOpen = false">
                            <i class="fa fa-arrow-left" />
                            <strong class="ml8">BACK</strong>
                        </button>
                    </div>
                    <div class="o_mobile_search_content">
                        <div class="o_search_panel flex-grow-0 flex-shrink-0 border-end pe-2 pb-5 ps-4 h-100 bg-view overflow-auto">
                            <section t-foreach="props.groups" t-as="group" t-key="group.id" class="o_search_panel_section o_search_panel_category">
                                <header class="o_search_panel_section_header pt-4 pb-2 text-uppercase cursor-default">
                                    <b t-esc="group.name"/>
                                </header>
                                <ul class="list-group d-block o_search_panel_field">
                                    <li t-foreach="group.dashboards" t-as="dashboard" t-key="dashboard.id" t-on-click="() => this.onDashboardSelected(dashboard.id)" class="o_search_panel_category_value list-group-item py-1 o_cursor_pointer border-0 ps-0 pe-2">
                                        <header class="list-group-item list-group-item-action d-flex align-items-center p-0 border-0" t-att-class="{'active text-900 fw-bold': dashboard.id === props.activeDashboard.id}">
                                            <div class="o_search_panel_label d-flex align-items-center overflow-hidden w-100 o_cursor_pointer mb-0">
                                                <!-- empty button to mimick the standard search panel -->
                                                <button class="o_toggle_fold btn p-0 flex-shrink-0 text-center"/>
                                                <span t-esc="dashboard.displayName" class="o_search_panel_label_title text-truncate"/>
                                            </div>

                                        </header>
                                    </li>
                                </ul>
                            </section>
                        </div>
                    </div>
                </div>
            </t>
        </t>
        <div t-elif="props.groups.length" t-on-click="openDashboardSelection" class="d-flex align-items-center">
            <i class="fa fa-fw fa-filter"/>
            <div t-esc="searchBarText" class="o_search_panel_current_selection text-truncate ms-2 me-auto"/>
        </div>
    </div>
</templates>


```

## File: static\src\bundle\links\dashboard_link_plugin.js

```javascript
/** @odoo-module */

import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";

export default class DashboardLinkPlugin extends spreadsheet.UIPlugin {
    constructor(getters, state, dispatch, config, selection) {
        super(...arguments);
        this.env = config.evalContext.env;
        this.selection.observe(this, {
            handleEvent: this.handleEvent.bind(this),
        });
    }

    /**
     * @private
     */
    handleEvent(event) {
        if (!this.getters.isDashboard()) {
            return;
        }
        switch (event.type) {
            case "ZonesSelected": {
                const sheetId = this.getters.getActiveSheetId();
                const { col, row } = event.anchor.cell;
                const cell = this.getters.getCell(sheetId, col, row);
                if (cell !== undefined && cell.isLink()) {
                    cell.action(this.env);
                }
            }
        }
    }
}

```

## File: static\src\bundle\links\index.js

```javascript
/** @odoo-module */

import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";
import DashboardLinkPlugin from "./dashboard_link_plugin";

const { uiPluginRegistry } = spreadsheet.registries;

uiPluginRegistry.add("odooDashboardClickLink", DashboardLinkPlugin);

```

## File: static\src\bundle\list\clickable_cell.js

```javascript
/** @odoo-module */

import { SEE_RECORD_LIST, SEE_RECORD_LIST_VISIBLE } from "@spreadsheet/list/list_actions";
import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";

const { clickableCellRegistry } = spreadsheet.registries;

clickableCellRegistry.add("list", {
    condition: SEE_RECORD_LIST_VISIBLE,
    action: SEE_RECORD_LIST,
    sequence: 10,
});

```

## File: static\src\bundle\pivot\clickable_cell.js

```javascript
/** @odoo-module */

import spreadsheet from "@spreadsheet/o_spreadsheet/o_spreadsheet_extended";
import { SEE_RECORDS_PIVOT, SEE_RECORDS_PIVOT_VISIBLE } from "@spreadsheet/pivot/pivot_actions";
import { getFirstPivotFunction } from "@spreadsheet/pivot/pivot_helpers";

const { clickableCellRegistry } = spreadsheet.registries;

clickableCellRegistry.add("pivot", {
    condition: SEE_RECORDS_PIVOT_VISIBLE,
    action: SEE_RECORDS_PIVOT,
    sequence: 3,
});

clickableCellRegistry.add("pivot_set_filter_matching", {
    condition: (cell, env) => {
        return (
            SEE_RECORDS_PIVOT_VISIBLE(cell, env) &&
            getFirstPivotFunction(cell.content).functionName === "ODOO.PIVOT.HEADER" &&
            env.model.getters.getFiltersMatchingPivot(cell.content).length > 0
        );
    },
    action: (cell, env) => {
        const filters = env.model.getters.getFiltersMatchingPivot(cell.content);
        env.model.dispatch("SET_MANY_GLOBAL_FILTER_VALUE", { filters });
    },
    sequence: 2,
});

```

## File: views\menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_actions_dashboard_action" model="ir.actions.client">
        <field name="name">Dashboards</field>
        <field name="tag">action_spreadsheet_dashboard</field>
    </record>

    <menuitem
        id="spreadsheet_dashboard_menu_root"
        name="Dashboards"
        action="ir_actions_dashboard_action"
        web_icon="spreadsheet_dashboard,static/description/icon.svg"
        sequence="37"/>

    <menuitem
        id="spreadsheet_dashboard_menu_dashboard"
        name="Dashboards"
        action="ir_actions_dashboard_action"
        parent="spreadsheet_dashboard_menu_root"
        sequence="1"/>

    <menuitem
        id="spreadsheet_dashboard_menu_configuration"
        name="Configuration"
        parent="spreadsheet_dashboard_menu_root"
        sequence="150"/>

    <record id="spreadsheet_dashboard_action_configuration_dashboards" model="ir.actions.act_window">
        <field name="name">Dashboards</field>
        <field name="res_model">spreadsheet.dashboard.group</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem
        id="spreadsheet_dashboard_menu_configuration_dashboards"
        name="Dashboards"
        parent="spreadsheet_dashboard_menu_configuration"
        action="spreadsheet_dashboard_action_configuration_dashboards"
        sequence="10"/>



</odoo>

```

## File: views\spreadsheet_dashboard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="spreadsheet_dashboard_view_list" model="ir.ui.view">
        <field name="name">spreadsheet.dashboard.view.list</field>
        <field name="model">spreadsheet.dashboard</field>
        <field name="arch" type="xml">
            <tree create="false" editable="bottom">
                <field name="sequence" widget="handle" groups="base.group_system"/>
                <field name="name"/>
                <field name="group_ids" widget="many2many_tags"/>
                <field name="dashboard_group_id" optional="hidden"/>
            </tree>
        </field>
    </record>

    <record id="spreadsheet_dashboard_container_view_list" model="ir.ui.view">
        <field name="name">spreadsheet.dashboard.group.view.list</field>
        <field name="model">spreadsheet.dashboard.group</field>
        <field name="arch" type="xml">
            <tree string="Dashboards">
                <field name="sequence" widget="handle" groups="base.group_system"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="spreadsheet_dashboard_container_view_form" model="ir.ui.view">
        <field name="name">spreadsheet.dashboard.group.view.form</field>
        <field name="model">spreadsheet.dashboard.group</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <notebook>
                        <page string="Spreadsheets" name="spreadsheets">
                            <field name="dashboard_ids" context="{'tree_view_ref': 'spreadsheet_dashboard.spreadsheet_dashboard_view_list'}"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

