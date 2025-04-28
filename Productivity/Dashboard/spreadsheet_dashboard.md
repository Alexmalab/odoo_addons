# Odoo Module: spreadsheet_dashboard

Category: Productivity/Dashboard

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Spreadsheet dashboard",
    "version": "1.0",
    "category": "Productivity/Dashboard",
    "summary": "Spreadsheet",
    "description": "Spreadsheet",
    "depends": ["spreadsheet"],
    "installable": True,
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
        'spreadsheet.assets_print': [
            'spreadsheet_dashboard/static/src/print_assets/**/*',
        ],
        "web.assets_backend": [
            "spreadsheet_dashboard/static/src/assets/**/*.js",
            "spreadsheet_dashboard/static/src/**/*.scss",
        ],
        'web.assets_unit_tests': [
            "spreadsheet_dashboard/static/tests/**/*",
        ],
    },
}

```

## File: controllers\share.py

```python
from odoo import http
from odoo.http import request

class DashboardShareRoute(http.Controller):
    @http.route(['/dashboard/share/<int:share_id>/<token>'], type='http', auth='public')
    def share_portal(self, share_id=None, token=None):
        share = request.env["spreadsheet.dashboard.share"].sudo().browse(share_id).exists()
        if not share:
            raise request.not_found()
        share._check_dashboard_access(token)
        return request.render(
            "spreadsheet.public_spreadsheet_layout",
            {
                "spreadsheet_name": share.dashboard_id.name,
                "share": share,
                "is_frozen": True,
                "session_info": request.env["ir.http"].session_info(),
                "props": {
                    "dataUrl": f"/dashboard/data/{share.id}/{token}",
                    "downloadExcelUrl": f"/dashboard/download/{share.id}/{token}",
                    "mode": "dashboard",
                },
            },
        )

    @http.route(["/dashboard/download/<int:share_id>/<token>"],
                type='http', auth='public')
    def download(self, token=None, share_id=None):
        share = request.env["spreadsheet.dashboard.share"].sudo().browse(share_id)
        share._check_dashboard_access(token)
        stream = request.env["ir.binary"]._get_stream_from(
            share, "excel_export", filename=share.name
        )
        return stream.get_response()

    @http.route(
        ["/dashboard/data/<int:share_id>/<token>"],
        type="http",
        auth="public",
        methods=["GET"],
    )
    def get_shared_dashboard_data(self, share_id, token):
        share = (
            request.env["spreadsheet.dashboard.share"]
            .sudo()
            .browse(share_id)
            .exists()
        )
        if not share:
            raise request.not_found()

        share._check_dashboard_access(token)
        stream = request.env["ir.binary"]._get_stream_from(
            share, "spreadsheet_binary_data"
        )
        return stream.get_response()

```

## File: controllers\__init__.py

```python
from . import share

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
        <field name="name">Services</field>
        <field name="sequence">500</field>
    </record>

    <record id="spreadsheet_dashboard_group_logistics" model="spreadsheet.dashboard.group">
        <field name="name">Logistics</field>
        <field name="sequence">400</field>
    </record>

    <record id="spreadsheet_dashboard_group_marketing" model="spreadsheet.dashboard.group">
        <field name="name">Marketing</field>
        <field name="sequence">600</field>
    </record>

</odoo>

```

## File: models\spreadsheet_dashboard.py

```python
import json

from odoo import _, fields, models
from odoo.tools import file_open


class SpreadsheetDashboard(models.Model):
    _name = 'spreadsheet.dashboard'
    _description = 'Spreadsheet Dashboard'
    _inherit = "spreadsheet.mixin"
    _order = 'sequence'

    name = fields.Char(required=True, translate=True)
    dashboard_group_id = fields.Many2one('spreadsheet.dashboard.group', required=True)
    sequence = fields.Integer()
    sample_dashboard_file_path = fields.Char(export_string_translation=False)
    is_published = fields.Boolean(default=True)
    company_id = fields.Many2one('res.company')
    group_ids = fields.Many2many('res.groups', default=lambda self: self.env.ref('base.group_user'))
    main_data_model_ids = fields.Many2many('ir.model')

    def get_readonly_dashboard(self):
        self.ensure_one()
        snapshot = json.loads(self.spreadsheet_data)
        if self._dashboard_is_empty() and self.sample_dashboard_file_path:
            sample_data = self._get_sample_dashboard()
            if sample_data:
                return {
                    "snapshot": sample_data,
                    "is_sample": True,
                }
        user_locale = self.env['res.lang']._get_user_spreadsheet_locale()
        snapshot.setdefault('settings', {})['locale'] = user_locale
        default_currency = self.env['res.currency'].get_company_currency_for_spreadsheet()
        return {
            'snapshot': snapshot,
            'revisions': [],
            'default_currency': default_currency,
        }

    def _get_sample_dashboard(self):
        try:
            with file_open(self.sample_dashboard_file_path) as f:
                return json.load(f)
        except FileNotFoundError:
            return

    def _dashboard_is_empty(self):
        return any(self.env[model].search_count([], limit=1) == 0 for model in self.main_data_model_ids.sudo().mapped("model"))

    def copy_data(self, default=None):
        default = dict(default or {})
        vals_list = super().copy_data(default=default)
        if 'name' not in default:
            for dashboard, vals in zip(self, vals_list):
                vals['name'] = _("%s (copy)", dashboard.name)
        return vals_list

```

## File: models\spreadsheet_dashboard_group.py

```python
from odoo import fields, models, api, _
from odoo.exceptions import UserError


class SpreadsheetDashboardGroup(models.Model):
    _name = 'spreadsheet.dashboard.group'
    _description = 'Group of dashboards'
    _order = 'sequence'

    name = fields.Char(required=True, translate=True)
    dashboard_ids = fields.One2many('spreadsheet.dashboard', 'dashboard_group_id')
    published_dashboard_ids = fields.One2many('spreadsheet.dashboard', 'dashboard_group_id', domain=[('is_published', '=', True)])
    sequence = fields.Integer()

    @api.ondelete(at_uninstall=False)
    def _unlink_except_spreadsheet_data(self):
        external_ids = self.get_external_id()
        for group in self:
            external_id = external_ids[group.id]
            if external_id and not external_id.startswith('__export__'):
                raise UserError(_("You cannot delete %s as it is used in another module.", group.name))

```

## File: models\spreadsheet_dashboard_share.py

```python
import base64
import uuid
from werkzeug.exceptions import Forbidden

from odoo import models, fields, api, _
from odoo.tools import consteq

class SpreadsheetDashboardShare(models.Model):
    _name = 'spreadsheet.dashboard.share'
    _inherit = 'spreadsheet.mixin'
    _description = 'Copy of a shared dashboard'

    dashboard_id = fields.Many2one('spreadsheet.dashboard', required=True, ondelete='cascade')
    excel_export = fields.Binary()
    access_token = fields.Char(required=True, default=lambda _x: str(uuid.uuid4()))
    full_url = fields.Char(string="URL", compute='_compute_full_url')
    name = fields.Char(related='dashboard_id.name')

    @api.depends('access_token')
    def _compute_full_url(self):
        for share in self:
            share.full_url = "%s/dashboard/share/%s/%s" % (share.get_base_url(), share.id, share.access_token)

    @api.model
    def action_get_share_url(self, vals):
        if "excel_files" in vals:
            excel_zip = self._zip_xslx_files(
                vals["excel_files"]
            )
            del vals["excel_files"]
            vals["excel_export"] = base64.b64encode(excel_zip)
        return self.create(vals).full_url

    def _check_token(self, access_token):
        if not access_token:
            return False
        return consteq(access_token, self.access_token)

    def _check_dashboard_access(self, access_token):
        self.ensure_one()
        token_access = self._check_token(access_token)
        dashboard = self.dashboard_id.with_user(self.create_uid)
        user_access = dashboard.has_access("read")
        if not (token_access and user_access):
            raise Forbidden(_("You don't have access to this dashboard. "))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import spreadsheet_dashboard_group
from . import spreadsheet_dashboard
from . import spreadsheet_dashboard_share

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_spreadsheet_dashboard_group_user","access_spreadsheet_dashboard_group_user","model_spreadsheet_dashboard_group","base.group_user",1,0,0,0
spreadsheet_dashboard_user","spreadsheet_dashboard_user","model_spreadsheet_dashboard","base.group_user",1,0,0,0
"access_spreadsheet_dashboard_group","access_spreadsheet_dashboard_group","model_spreadsheet_dashboard_group","spreadsheet_dashboard.group_dashboard_manager",1,1,1,1
"spreadsheet_dashboard","spreadsheet_dashboard","model_spreadsheet_dashboard","spreadsheet_dashboard.group_dashboard_manager",1,1,1,1
access_spreadsheet_dashboard_share,access_spreadsheet_dashboard_share,model_spreadsheet_dashboard_share,"base.group_user",1,1,1,1

```

## File: security\security.xml

```xml
<odoo>
    <!-- duplicated for spreadsheetCellThreads in rule spreadsheet_dashboard_edition.ir_rule_spreadsheet_dashboard_threads
        Please update aforemetioned the rule accordingly -->
    <record id="ir_rule_spreadsheet_dashboard" model="ir.rule">
        <field name="name">Spreadsheet dashboard: groups</field>
        <field name="model_id" ref="model_spreadsheet_dashboard" />
        <field name="groups" eval="[(4, ref('base.group_user'))]" />
        <field name="domain_force">[('group_ids', 'in', user.groups_id.ids)]</field>
    </record>

    <record id="spreadsheet_dashboard_rule_company" model="ir.rule">
        <field name="name">Dashboard multi-company</field>
        <field name="model_id" ref="model_spreadsheet_dashboard"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="base.module_category_productivity_dashboard" model="ir.module.category">
        <field name="name">Dashboard</field>
        <field name="description">User access level for Dashboard module</field>
        <field name="sequence">30</field>
    </record>

    <record id="spreadsheet_dashboard.group_dashboard_manager" model="res.groups">
        <field name="name">Admin</field>
        <field name="category_id" ref="base.module_category_productivity_dashboard" />
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]" />
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="spreadsheet_dashboard_share_create_uid_rule" model="ir.rule">
        <field name="name">spreadsheet.dashboard.share: create uid</field>
        <field name="model_id" ref="model_spreadsheet_dashboard_share" />
        <field name="groups" eval="[(4, ref('base.group_user'))]" />
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M4 8a4 4 0 0 1 4-4h17v17a4 4 0 0 1-4 4H4V8Z" fill="#985184"/><path d="M4 35a4 4 0 0 1 4-4h11v11a4 4 0 0 1-4 4H4V35Z" fill="#088BF5"/><path d="M25 46h4a4 4 0 0 0 4-4V31h-4a4 4 0 0 0-4 4v11Z" fill="#1AD3BB"/><path d="M31 17v4a4 4 0 0 0 4 4h11v-4a4 4 0 0 0-4-4H31Z" fill="#F9464C"/><path d="M31 4v4a4 4 0 0 0 4 4h11V8a4 4 0 0 0-4-4H31Z" fill="#FC868B"/><path d="M38 46h4a4 4 0 0 0 4-4V31h-4a4 4 0 0 0-4 4v11Z" fill="#03AF89"/></svg>

```

## File: static\src\assets\dashboard_action_loader.js

```javascript
import { addSpreadsheetActionLazyLoader } from "@spreadsheet/assets_backend/spreadsheet_action_loader";

addSpreadsheetActionLazyLoader("action_spreadsheet_dashboard");

```

## File: static\src\bundle\dashboard_action\dashboard_action.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { ControlPanel } from "@web/search/control_panel/control_panel";
import { DashboardLoader, Status } from "./dashboard_loader";
import { SpreadsheetComponent } from "@spreadsheet/actions/spreadsheet_component";
import { useSetupAction } from "@web/search/action_hook";
import { DashboardMobileSearchPanel } from "./mobile_search_panel/mobile_search_panel";
import { MobileFigureContainer } from "./mobile_figure_container/mobile_figure_container";
import { FilterValue } from "@spreadsheet/global_filters/components/filter_value/filter_value";
import { useService } from "@web/core/utils/hooks";
import { standardActionServiceProps } from "@web/webclient/actions/action_service";
import { SpreadsheetShareButton } from "@spreadsheet/components/share_button/share_button";
import { useSpreadsheetPrint } from "@spreadsheet/hooks";
import { Registry } from "@odoo/o-spreadsheet";
import { router } from "@web/core/browser/router";

import { Component, onWillStart, useState, useEffect } from "@odoo/owl";

export const dashboardActionRegistry = new Registry();

export class SpreadsheetDashboardAction extends Component {
    static template = "spreadsheet_dashboard.DashboardAction";
    static components = {
        ControlPanel,
        SpreadsheetComponent,
        FilterValue,
        DashboardMobileSearchPanel,
        MobileFigureContainer,
        SpreadsheetShareButton,
    };
    static props = { ...standardActionServiceProps };

    setup() {
        this.Status = Status;
        this.controlPanelDisplay = {};
        this.orm = useService("orm");
        this.actionService = useService("action");
        // Use the non-protected orm service (`this.env.services.orm` instead of `useService("orm")`)
        // because spreadsheets models are preserved across multiple components when navigating
        // with the breadcrumb
        // TODO write a test
        /** @type {DashboardLoader}*/
        this.loader = useState(new DashboardLoader(this.env, this.env.services.orm));
        onWillStart(async () => {
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
            () => router.pushState({ dashboard_id: this.activeDashboardId }),
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
                return [dashboard?.model, dashboard?.status];
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
        useSpreadsheetPrint(() => this.state.activeDashboard?.model);
        /** @type {{ activeDashboard: import("./dashboard_loader").Dashboard}} */
        this.state = useState({ activeDashboard: undefined, sidebarExpanded: true });
    }

    get dashboardButton() {
        return dashboardActionRegistry.getAll()[0];
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
     * @param {number} id - The ID of the dashboard to be edited.
     * @returns {Promise<void>}
     */
    async editDashboard(id) {
        const action = await this.env.services.orm.call(
            "spreadsheet.dashboard",
            "action_edit_dashboard",
            [id]
        );
        this.actionService.doAction(action);
    }

    async shareSpreadsheet(data, excelExport) {
        const url = await this.orm.call("spreadsheet.dashboard.share", "action_get_share_url", [
            {
                dashboard_id: this.activeDashboardId,
                spreadsheet_data: JSON.stringify(data),
                excel_files: excelExport.files,
            },
        ]);
        return url;
    }

    toggleSidebar() {
        this.state.sidebarExpanded = !this.state.sidebarExpanded;
    }

    get activeDashboardGroupName() {
        return this.getDashboardGroups().find((group) =>
            group.dashboards.some((d) => d.id === this.activeDashboardId)
        )?.name;
    }
}

registry
    .category("actions")
    .add("action_spreadsheet_dashboard", SpreadsheetDashboardAction, { force: true });

```

## File: static\src\bundle\dashboard_action\dashboard_action.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="spreadsheet_dashboard.DashboardAction" class="o_action o_spreadsheet_dashboard_action o_field_highlight">
        <ControlPanel display="controlPanelDisplay">
            <t t-set-slot="layout-actions" t-if="!state.activeDashboard?.isSample">
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
            <t t-set-slot="control-panel-navigation-additional" t-if="!state.activeDashboard?.isSample">
                <SpreadsheetShareButton t-key="activeDashboardId" model="state.activeDashboard?.model" onSpreadsheetShared.bind="shareSpreadsheet" togglerClass="'btn-light'"/>
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
                <t t-if="state.sidebarExpanded" t-call="spreadsheet_dashboard.DashboardAction.Expanded"/>
                <t t-else="" t-call="spreadsheet_dashboard.DashboardAction.Collapsed"/>
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
                    <div t-else="" class="o_renderer" t-att-class="{'o-sample-dashboard': dashboard.isSample}">
                        <SpreadsheetComponent
                            model="dashboard.model"
                            t-key="dashboard.id"/>
                    </div>
                </t>
            </t>
        </div>
    </div>

    <t t-name="spreadsheet_dashboard.DashboardAction.Collapsed">
        <div class="bg-view h-100 o_search_panel_sidebar cursor-pointer" t-on-click="toggleSidebar">
            <div class="d-flex">
                <button class="btn btn-light btn-sm m-1 mb-2 p-2">
                    <i class="fa fa-angle-double-right"/>
                </button>
                <div class="mx-auto" t-if="state.activeDashboard">
                    <span class="fw-bolder" t-esc="activeDashboardGroupName"/>
                    /
                    <t t-esc="state.activeDashboard.displayName"/>
                </div>
            </div>
        </div>
    </t>

    <t t-name="spreadsheet_dashboard.DashboardAction.Expanded">
        <div class="o_spreadsheet_dashboard_search_panel o_search_panel flex-grow-0 border-end flex-shrink-0 pe-2 pb-5 ps-4 h-100 bg-view overflow-auto position-relative">
            <button t-if="!env.isSmall and state.activeDashboard" class="btn btn-light btn-sm end-0 m-1 mb-2 position-absolute px-2 py-1 top-0 z-1" t-on-click="toggleSidebar">
                <i class="fa fa-fw fa-angle-double-left"/>
            </button>
            <div class="mt-2"/>
            <section t-foreach="getDashboardGroups()" t-as="group" t-key="group.id" class="o_search_panel_section o_search_panel_category">
                <header class="o_search_panel_section_header pt-4 pb-2 text-uppercase o_cursor_default user-select-none">
                    <b t-esc="group.name"/>
                </header>
                <ul class="list-group d-block o_search_panel_field">
                    <li t-foreach="group.dashboards" t-as="dashboard" t-key="dashboard.id"
                        t-on-click="() => this.openDashboard(dashboard.id)"
                        t-att-data-name="dashboard.displayName"
                        class="o_search_panel_category_value list-group-item cursor-pointer border-0 d-flex justify-content-between align-items-center"
                        t-att-class="{'active': dashboard.id === state.activeDashboard.id}">
                        <div class="o_dashboard_name">
                            <t t-esc="dashboard.displayName" />
                        </div>
                        <t t-set="comp" t-value="dashboardButton"/>
                        <t t-if="comp">
                            <t t-component="comp" t-props="{ dashboardId: dashboard.id, onClick: this.editDashboard.bind(this) }"/>
                        </t>
                    </li>
                </ul>
            </section>
        </div>
    </t>
</templates>

```

## File: static\src\bundle\dashboard_action\dashboard_loader.js

```javascript
/** @odoo-module */

import { Model } from "@odoo/o-spreadsheet";
import { OdooDataProvider } from "@spreadsheet/data_sources/odoo_data_provider";
import { createDefaultCurrency } from "@spreadsheet/currency/helpers";

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
 * @property {Array<{id: number, name: string}>} dashboards
 *
 * @typedef DashboardGroup
 * @property {number} id
 * @property {string} name
 * @property {Array<Dashboard>} dashboards
 *
 * @typedef {import("@web/env").OdooEnv} OdooEnv
 *
 * @typedef {import("@web/core/orm_service").ORM} ORM
 */

export class DashboardLoader {
    /**
     * @param {OdooEnv} env
     * @param {ORM} orm
     */
    constructor(env, orm) {
        /** @private */
        this.env = env;
        /** @private */
        this.orm = orm;
        /** @private @type {Array<DashboardGroupData>} */
        this.groups = [];
        /** @private @type {Object<number, Dashboard>} */
        this.dashboards = {};
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
            .filter((group) => group.published_dashboard_ids.length)
            .map((group) => ({
                id: group.id,
                name: group.name,
                dashboards: group.published_dashboard_ids,
            }));
        const dashboards = this.groups.map((group) => group.dashboards).flat();
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
            dashboards: section.dashboards.map((dashboard) => ({
                id: dashboard.id,
                displayName: dashboard.name,
                status: this._getDashboard(dashboard.id).status,
            })),
        }));
    }

    /**
     * @private
     * @returns {Promise<{id: number, name: string, published_dashboard_ids: number[]}[]>}
     */
    async _fetchGroups() {
        const groups = await this.orm.webSearchRead(
            "spreadsheet.dashboard.group",
            [["published_dashboard_ids", "!=", false]],
            {
                specification: {
                    name: {},
                    published_dashboard_ids: { fields: { name: {} } },
                },
            }
        );
        return groups.records;
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
            const { snapshot, revisions, default_currency, is_sample } = await this.orm.call(
                "spreadsheet.dashboard",
                "get_readonly_dashboard",
                [dashboardId]
            );
            dashboard.model = this._createSpreadsheetModel(snapshot, revisions, default_currency);
            dashboard.status = Status.Loaded;
            dashboard.isSample = is_sample;
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
     * @param {object} snapshot
     * @param {object[]} revisions
     * @param {object} [defaultCurrency]
     * @returns {Model}
     */
    _createSpreadsheetModel(snapshot, revisions = [], currency) {
        const odooDataProvider = new OdooDataProvider(this.env);
        const model = new Model(
            snapshot,
            {
                custom: { env: this.env, orm: this.orm, odooDataProvider },
                mode: "dashboard",
                defaultCurrency: createDefaultCurrency(currency),
            },
            revisions
        );
        this._activateFirstSheet(model);
        odooDataProvider.addEventListener("data-source-updated", () =>
            model.dispatch("EVALUATE_CELLS")
        );
        return model;
    }
}

```

## File: static\src\bundle\dashboard_action\mobile_figure_container\mobile_figure_container.js

```javascript
/** @odoo-module */

import * as spreadsheet from "@odoo/o-spreadsheet";

import { Component, useSubEnv } from "@odoo/owl";
const { registries } = spreadsheet;
const { figureRegistry } = registries;

export class MobileFigureContainer extends Component {
    static template = "documents_spreadsheet.MobileFigureContainer";
    static props = {
        spreadsheetModel: Object,
    };

    setup() {
        useSubEnv({
            model: this.props.spreadsheetModel,
            isDashboard: () => this.props.spreadsheetModel.getters.isDashboard(),
            openSidePanel: () => {},
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

```

## File: static\src\bundle\dashboard_action\mobile_figure_container\mobile_figure_container.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <t t-name="documents_spreadsheet.MobileFigureContainer">
        <t t-if="!figures.length">
            Only chart figures are displayed in small screens but this dashboard doesn't contain any
        </t>
        <div
            t-foreach="figures" t-as="figure"
            t-key="figure.id"
            t-attf-style="min-height: #{figure.height}px;"
        >
            <t t-component="getFigureComponent(figure)" figure="figure" onFigureDeleted="() => {}"/>
        </div>
    </t>
</templates>

```

## File: static\src\bundle\dashboard_action\mobile_search_panel\mobile_search_panel.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";

import { Component, useState } from "@odoo/owl";

export class DashboardMobileSearchPanel extends Component {
    static template = "spreadsheet_dashboard.DashboardMobileSearchPanel";
    static props = {
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

```

## File: static\src\bundle\dashboard_action\mobile_search_panel\mobile_search_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="spreadsheet_dashboard.DashboardMobileSearchPanel" class="o_search_panel o_search_panel_summary btn w-100 overflow-visible">
        <t t-if="state.isOpen">
            <t t-portal="'body'">
                <div class="o_spreadsheet_dashboard_search_panel o_search_panel o_searchview o_mobile_search">
                    <div class="o_mobile_search_header">
                        <button type="button" class="o_mobile_search_button btn" t-on-click="() => this.state.isOpen = false">
                            <i class="oi oi-arrow-left" />
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

## File: static\src\bundle\list\clickable_cell.js

```javascript
/** @odoo-module */

import { SEE_RECORD_LIST, SEE_RECORD_LIST_VISIBLE } from "@spreadsheet/list/list_actions";
import * as spreadsheet from "@odoo/o-spreadsheet";

const { clickableCellRegistry } = spreadsheet.registries;

clickableCellRegistry.add("list", {
    condition: SEE_RECORD_LIST_VISIBLE,
    execute: SEE_RECORD_LIST,
    sequence: 10,
});

```

## File: static\src\bundle\pivot\clickable_cell.js

```javascript
/** @odoo-module */

import * as spreadsheet from "@odoo/o-spreadsheet";
import {
    SEE_RECORDS_PIVOT,
    SEE_RECORDS_PIVOT_VISIBLE,
    SET_FILTER_MATCHING,
    SET_FILTER_MATCHING_CONDITION,
} from "@spreadsheet/pivot/pivot_actions";

const { clickableCellRegistry } = spreadsheet.registries;

clickableCellRegistry.add("pivot", {
    condition: SEE_RECORDS_PIVOT_VISIBLE,
    execute: SEE_RECORDS_PIVOT,
    sequence: 3,
});

clickableCellRegistry.add("pivot_set_filter_matching", {
    condition: SET_FILTER_MATCHING_CONDITION,
    execute: SET_FILTER_MATCHING,
    sequence: 2,
});

```

## File: views\menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_actions_dashboard_action" model="ir.actions.client">
        <field name="name">Dashboards</field>
        <field name="path">dashboards</field>
        <field name="tag">action_spreadsheet_dashboard</field>
    </record>

    <menuitem
        id="spreadsheet_dashboard_menu_root"
        name="Dashboards"
        action="ir_actions_dashboard_action"
        web_icon="spreadsheet_dashboard,static/description/icon.png"
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
        <field name="view_mode">list,form</field>
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
            <list create="false" editable="bottom">
                <field name="sequence" widget="handle" groups="base.group_system"/>
                <field name="name"/>
                <field name="group_ids" widget="many2many_tags" required="1"/>
                <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                <field name="spreadsheet_binary_data" groups="base.group_no_one" widget="binary_spreadsheet" filename="spreadsheet_file_name" string="Data" />
                <field name="is_published" widget="boolean_toggle"/>
                <field name="dashboard_group_id" optional="hidden"/>
            </list>
        </field>
    </record>

    <record id="spreadsheet_dashboard_container_view_list" model="ir.ui.view">
        <field name="name">spreadsheet.dashboard.group.view.list</field>
        <field name="model">spreadsheet.dashboard.group</field>
        <field name="arch" type="xml">
            <list string="Dashboards">
                <field name="sequence" widget="handle" groups="base.group_system"/>
                <field name="name"/>
            </list>
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
                            <field name="dashboard_ids" context="{'list_view_ref': 'spreadsheet_dashboard.spreadsheet_dashboard_view_list'}"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="spreadsheet_dashboard_view_form" model="ir.ui.view">
        <field name="name">spreadsheet.dashboard.view.form</field>
        <field name="model">spreadsheet.dashboard</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name"/>
                    <field name="dashboard_group_id"/>
                    <field name="spreadsheet_binary_data"/>
                    <field name="thumbnail"/>
                    <field name="group_ids" widget="many2many_tags"/>
                </group>
            </form>
        </field>
    </record>
</odoo>

```

