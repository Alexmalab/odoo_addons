# Odoo Module: board

Category: Productivity

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
    'name': 'Dashboards',
    'version': '1.0',
    'category': 'Productivity',
    'sequence': 225,
    'summary': 'Build your own dashboards',
    'description': """
Lets the user create a custom dashboard.
========================================

Allows users to create custom dashboard.
    """,
    'depends': ['spreadsheet_dashboard'],
    'data': [
        'security/ir.model.access.csv',
        'views/board_views.xml',
        ],
    'assets': {
        'web.assets_backend': [
            'board/static/src/**/*.scss',
            'board/static/src/**/*.js',
            'board/static/src/**/*.xml',
        ],
        'web.qunit_suite_tests': [
            'board/static/tests/**/*',
            ('remove', 'board/static/tests/mobile/**/*'), # mobile test
        ],
        'web.qunit_mobile_suite_tests': [
            'board/static/tests/mobile/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from lxml import etree as ElementTree

from odoo.http import Controller, route, request


class Board(Controller):

    @route('/board/add_to_dashboard', type='json', auth='user')
    def add_to_dashboard(self, action_id, context_to_save, domain, view_mode, name=''):
        # Retrieve the 'My Dashboard' action from its xmlid
        action = request.env.ref('board.open_board_my_dash_action').sudo()

        if action and action['res_model'] == 'board.board' and action['views'][0][1] == 'form' and action_id:
            # Maybe should check the content instead of model board.board ?
            view_id = action['views'][0][0]
            board_view = request.env['board.board'].get_view(view_id, 'form')
            if board_view and 'arch' in board_view:
                board_arch = ElementTree.fromstring(board_view['arch'])
                column = board_arch.find('./board/column')
                if column is not None:
                    # We don't want to save allowed_company_ids
                    # Otherwise on dashboard, the multi-company widget does not filter the records
                    if 'allowed_company_ids' in context_to_save:
                        context_to_save.pop('allowed_company_ids')
                    new_action = ElementTree.Element('action', {
                        'name': str(action_id),
                        'string': name,
                        'view_mode': view_mode,
                        'context': str(context_to_save),
                        'domain': str(domain)
                    })
                    column.insert(0, new_action)
                    arch = ElementTree.tostring(board_arch, encoding='unicode')
                    request.env['ir.ui.view.custom'].sudo().create({
                        'user_id': request.session.uid,
                        'ref_id': view_id,
                        'arch': arch
                    })
                    return True

        return False

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\board.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Board(models.AbstractModel):
    _name = 'board.board'
    _description = "Board"
    _auto = False

    # This is necessary for when the web client opens a dashboard. Technically
    # speaking, the dashboard is a form view, and opening it makes the client
    # initialize a dummy record by invoking onchange(). And the latter requires
    # an 'id' field to work properly...
    id = fields.Id()

    @api.model_create_multi
    def create(self, vals_list):
        return self

    @api.model
    def get_view(self, view_id=None, view_type='form', **options):
        """
        Overrides orm field_view_get.
        @return: Dictionary of Fields, arch and toolbar.
        """

        res = super().get_view(view_id, view_type, **options)

        custom_view = self.env['ir.ui.view.custom'].sudo().search([('user_id', '=', self.env.uid), ('ref_id', '=', view_id)], limit=1)
        if custom_view:
            res.update({'custom_view_id': custom_view.id,
                        'arch': custom_view.arch})
        res['arch'] = self._arch_preprocessing(res['arch'])
        return res

    @api.model
    def _arch_preprocessing(self, arch):
        from lxml import etree

        def remove_unauthorized_children(node):
            for child in node.iterchildren():
                if child.tag == 'action' and child.get('invisible'):
                    node.remove(child)
                else:
                    remove_unauthorized_children(child)
            return node

        archnode = etree.fromstring(arch)
        # add the js_class 'board' on the fly to force the webclient to
        # instantiate a BoardView instead of FormView
        archnode.set('js_class', 'board')
        return etree.tostring(remove_unauthorized_children(archnode), pretty_print=True, encoding='unicode')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import board

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_board_board_all,board.board,model_board_board,base.group_user,1,0,0,0
```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M4 8a4 4 0 0 1 4-4h17v17a4 4 0 0 1-4 4H4V8Z" fill="#FC868B"/><path d="M4 35a4 4 0 0 1 4-4h11v11a4 4 0 0 1-4 4H4V35Z" fill="#F9464C"/><path d="M25 46h4a4 4 0 0 0 4-4V31h-4a4 4 0 0 0-4 4v11Z" fill="#FBB945"/><path d="M31 17v4a4 4 0 0 0 4 4h11v-4a4 4 0 0 0-4-4H31Z" fill="#088BF5"/><path d="M31 4v4a4 4 0 0 0 4 4h11V8a4 4 0 0 0-4-4H31Z" fill="#2EBCFA"/><path d="M38 46h4a4 4 0 0 0 4-4V31h-4a4 4 0 0 0-4 4v11Z" fill="#F78613"/></svg>

```

## File: static\src\board_action.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { View } from "@web/views/view";
import { makeContext } from "@web/core/context";
import { Component, onWillStart } from "@odoo/owl";

export class BoardAction extends Component {
    setup() {
        const rpc = useService("rpc");
        const userService = useService("user");
        this.actionService = useService("action");
        const action = this.props.action;
        this.formViewId = false;
        this.isValid = true;
        onWillStart(async () => {
            let result = BoardAction.cache[action.actionId];
            if (!result) {
                result = await rpc("/web/action/load", { action_id: action.actionId });
                BoardAction.cache[action.actionId] = result;
            }
            if (!result) {
                // action does not exist
                this.isValid = false;
                return;
            }
            const viewMode = action.viewMode || result.views[0][1];
            const formView = result.views.find((v) => v[1] === "form");
            if (formView) {
                this.formViewId = formView[0];
            }
            this.viewProps = {
                resModel: result.res_model,
                type: viewMode,
                display: { controlPanel: false },
                selectRecord: (resId) => this.selectRecord(result.res_model, resId),
            };
            const view = result.views.find((v) => v[1] === viewMode);
            if (view) {
                this.viewProps.viewId = view[0];
            }
            const searchView = result.views.find((v) => v[1] === "search");
            this.viewProps.views = [
                [this.viewProps.viewId || false, viewMode],
                [(searchView && searchView[0]) || false, "search"],
            ];

            if (action.context) {
                this.viewProps.context = makeContext([
                    action.context,
                    { lang: userService.context.lang },
                ]);
                if ("group_by" in this.viewProps.context) {
                    const groupBy = this.viewProps.context.group_by;
                    this.viewProps.groupBy = typeof groupBy === "string" ? [groupBy] : groupBy;
                }
                if ("comparison" in this.viewProps.context) {
                    const comparison = this.viewProps.context.comparison;
                    if (
                        comparison !== null &&
                        typeof comparison === "object" &&
                        "domains" in comparison &&
                        "fieldName" in comparison
                    ) {
                        // Some comparison object with the wrong form might have been stored in db.
                        // This is why we make the checks on the keys domains and fieldName
                        this.viewProps.comparison = comparison;
                    }
                }
            }
            if (action.domain) {
                this.viewProps.domain = action.domain;
            }
            if (viewMode === "list") {
                this.viewProps.allowSelectors = false;
            }
        });
    }
    selectRecord(resModel, resId) {
        this.actionService.doAction({
            type: "ir.actions.act_window",
            res_model: resModel,
            views: [[this.formViewId, "form"]],
            res_id: resId,
        });
    }
}
BoardAction.template = "board.BoardAction";
BoardAction.components = { View };
BoardAction.cache = {};

```

## File: static\src\board_action.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="board.BoardAction">
        <t t-if="isValid">
            <View t-props="viewProps"/>
        </t>
        <t t-else="">
            <div class="text-center text-warning m-1">Invalid action</div>
        </t>
    </t>
</templates>

```

## File: static\src\board_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { browser } from "@web/core/browser/browser";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { useService } from "@web/core/utils/hooks";
import { renderToString } from "@web/core/utils/render";
import { useSortable } from "@web/core/utils/sortable_owl";
import { standardViewProps } from "@web/views/standard_view_props";
import { BoardAction } from "./board_action";
import { blockDom, Component, useState, useRef } from "@odoo/owl";

export class BoardController extends Component {
    setup() {
        this.board = useState(this.props.board);
        this.rpc = useService("rpc");
        this.dialogService = useService("dialog");
        if (this.env.isSmall) {
            this.selectLayout("1", false);
        } else {
            const mainRef = useRef("main");
            useSortable({
                ref: mainRef,
                elements: ".o-dashboard-action",
                handle: ".o-dashboard-action-header",
                cursor: "move",
                groups: ".o-dashboard-column",
                connectGroups: true,
                onDrop: ({ element, previous, parent }) => {
                    const fromColIdx = parseInt(element.parentElement.dataset.idx, 10);
                    const fromActionIdx = parseInt(element.dataset.idx, 10);
                    const toColIdx = parseInt(parent.dataset.idx, 10);
                    const toActionIdx = previous ? parseInt(previous.dataset.idx, 10) + 1 : 0;
                    if (fromColIdx !== toColIdx) {
                        // to reduce visual flickering
                        element.classList.add("d-none");
                    }
                    this.moveAction(fromColIdx, fromActionIdx, toColIdx, toActionIdx);
                },
            });
        }
    }

    moveAction(fromColIdx, fromActionIdx, toColIdx, toActionIdx) {
        const action = this.board.columns[fromColIdx].actions[fromActionIdx];
        if (fromColIdx !== toColIdx) {
            // action moving from a column to another
            this.board.columns[fromColIdx].actions.splice(fromActionIdx, 1);
            this.board.columns[toColIdx].actions.splice(toActionIdx, 0, action);
        } else {
            // move inside a column
            if (fromActionIdx === toActionIdx) {
                return;
            }
            const actions = this.board.columns[fromColIdx].actions;
            if (fromActionIdx < toActionIdx) {
                actions.splice(toActionIdx + 1, 0, action);
                actions.splice(fromActionIdx, 1);
            } else {
                actions.splice(fromActionIdx, 1);
                actions.splice(toActionIdx, 0, action);
            }
        }
        this.saveBoard();
    }

    selectLayout(layout, save = true) {
        const currentColNbr = this.board.colNumber;
        const nextColNbr = layout.split("-").length;
        if (nextColNbr < currentColNbr) {
            // need to move all actions in last cols in the last visible col
            const cols = this.board.columns;
            const lastVisibleCol = cols[nextColNbr - 1];
            for (let i = nextColNbr; i < currentColNbr; i++) {
                lastVisibleCol.actions.push(...cols[i].actions);
                cols[i].actions = [];
            }
        }
        this.board.layout = layout;
        this.board.colNumber = nextColNbr;
        if (save) {
            this.saveBoard();
        }
        if (document.querySelector("canvas")) {
            // horrible hack to force charts to be recreated so they pick up the
            // proper size. also, no idea why raf is needed :(
            browser.requestAnimationFrame(() => this.render(true));
        }
    }

    closeAction(column, action) {
        this.dialogService.add(ConfirmationDialog, {
            body: _t("Are you sure that you want to remove this item?"),
            confirm: () => {
                const index = column.actions.indexOf(action);
                column.actions.splice(index, 1);
                this.saveBoard();
            },
            cancel: () => {},
        });
    }

    toggleAction(action, save = true) {
        action.isFolded = !action.isFolded;
        if (save) {
            this.saveBoard();
        }
    }

    saveBoard() {
        const templateFn = renderToString.app.getTemplate("board.arch");
        const bdom = templateFn(this.board, {});
        const root = document.createElement("rendertostring");
        blockDom.mount(bdom, root);
        const result = xmlSerializer.serializeToString(root);
        const arch = result.slice(result.indexOf("<", 1), result.indexOf("</rendertostring>"));

        this.rpc("/web/view/edit_custom", {
            custom_id: this.board.customViewId,
            arch,
        });
        this.env.bus.trigger("CLEAR-CACHES");
    }
}

BoardController.template = "board.BoardView";
BoardController.components = { BoardAction, Dropdown, DropdownItem };
BoardController.props = {
    ...standardViewProps,
    board: Object,
};

const xmlSerializer = new XMLSerializer();

```

## File: static\src\board_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="board.BoardView">
        <div class="o_dashboard d-flex flex-column">
            <t t-if="board.isEmpty">
                <t t-call="board.NoContent"/>
            </t>
            <t t-else="">
                <t t-call="board.Content"/>
            </t>
        </div>
    </t>

    <t t-name="board.Content">
        <div t-if="!env.isSmall" class="o-dashboard-header d-flex justify-content-end">
            <Dropdown togglerClass="'btn btn-secondary m-2 p-2'">
                <t t-set-slot="toggler">
                    <img t-attf-src="/board/static/img/layout_{{board.layout}}.png" width="16" height="16" alt=""/>
                    Change Layout
                </t>
                <t t-foreach="'1 1-1 1-1-1 1-2 2-1'.split(' ')" t-as="layout" t-key="layout">
                    <DropdownItem onSelected="() => this.selectLayout(layout)">
                        <img t-attf-src="/board/static/img/layout_{{layout}}.png" alt=""/>
                        <i t-if="layout == board.layout" class="fa fa-check fa-lg text-success" aria-label='Layout' role="img" title="Layout"/>
                    </DropdownItem>
                </t>
            </Dropdown>
        </div>
        <div class="o-dashboard-layout flex-grow-1" t-attf-class="o-dashboard-layout-{{board.layout}}" t-ref="main">
            <t t-foreach="board.columns" t-as="column" t-key="column_index">
                <div t-if="column_index lt board.colNumber" class="o-dashboard-column h-100" t-att-data-idx="column_index">
                    <t t-foreach="column.actions" t-as="action" t-key="action.id">
                        <div class="o-dashboard-action mx-3 my-1 bg-view border" t-att-data-idx="action_index">
                            <h3 t-attf-class="o-dashboard-action-header {{action.title ? '' : 'oe_header_empty'}} p-2 m-2">
                                <span> <t t-esc="action.title"/> </span>
                                <span t-if="!env.isSmall" class="btn float-end p-1 text-muted" t-on-click="() => this.closeAction(column, action)"><i class="fa fa-close"/></span>
                                <span class="btn float-end p-1 text-muted" t-if="!action.isFolded" t-on-click="() => this.toggleAction(action, !env.isSmall)"><i class="fa fa-window-minimize"/></span>
                                <span class="btn float-end p-1 text-muted" t-if="action.isFolded"  t-on-click="() => this.toggleAction(action, !env.isSmall)"><i class="fa fa-window-maximize"/></span>
                            </h3>
                            <BoardAction t-if="!action.isFolded" action="action" />
                        </div>
                    </t>
                </div>
            </t>
        </div>
    </t>

    <t t-name="board.NoContent">
        <div class="o_view_nocontent">
            <div class="o_nocontent_help">
                <p class="o_view_nocontent_neutral_face">
                  Your personal dashboard is empty
                </p>
                <p>
                  To add your first report into this dashboard, go to any
                  menu, switch to list or graph view, and click <i>"Add to
                  Dashboard"</i> in the extended search options.
                </p>
                <p>
                  You can filter and group data before inserting into the
                  dashboard using the search options.
                </p>
            </div>
        </div>
    </t>

    <t t-name="board.arch">
        <form t-att-string="title">
            <board t-att-style="layout">
                <column t-foreach="columns" t-as="column" t-key="column_index">
                    <t t-foreach="column.actions" t-as="action" t-key="action_index">
                        <action
                            t-att-name="action.actionId"
                            t-att-string="action.title"
                            t-att-view_mode="action.viewMode"
                            t-att-context="action.context"
                            t-att-domain="action.domain"
                            t-att-fold="action.isFolded ? 1 : 0"/>
                    </t>
                </column>
            </board>
        </form>
    </t>


</templates>

```

## File: static\src\board_view.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { BoardController } from "./board_controller";
import { visitXML } from "@web/core/utils/xml";
import { Domain } from "@web/core/domain";
export class BoardArchParser {
    parse(arch, customViewId) {
        let nextId = 1;
        const archInfo = {
            title: null,
            layout: null,
            colNumber: 0,
            isEmpty: true,
            columns: [{ actions: [] }, { actions: [] }, { actions: [] }],
            customViewId,
        };
        let currentIndex = -1;

        visitXML(arch, (node) => {
            switch (node.tagName) {
                case "form":
                    archInfo.title = node.getAttribute("string");
                    break;
                case "board":
                    archInfo.layout = node.getAttribute("style");
                    archInfo.colNumber = archInfo.layout.split("-").length;
                    break;
                case "column":
                    currentIndex++;
                    break;
                case "action": {
                    archInfo.isEmpty = false;
                    const isFolded = Boolean(
                        node.hasAttribute("fold") ? parseInt(node.getAttribute("fold"), 10) : 0
                    );
                    const action = {
                        id: nextId++,
                        title: node.getAttribute("string"),
                        actionId: parseInt(node.getAttribute("name"), 10),
                        viewMode: node.getAttribute("view_mode"),
                        context: node.getAttribute("context"),
                        isFolded,
                    };
                    if (node.hasAttribute("domain")) {
                        const domain = node.getAttribute("domain");
                        action.domain = new Domain(domain).toList();
                        // so it can be serialized when reexporting board xml
                        action.domain.toString = () => node.getAttribute("domain");
                    }
                    archInfo.columns[currentIndex].actions.push(action);
                    break;
                }
            }
        });
        return archInfo;
    }
}

export const boardView = {
    type: "form",
    display_name: _t("Board"),
    Controller: BoardController,

    props: (genericProps, view) => {
        const { arch, info } = genericProps;
        const board = new BoardArchParser().parse(arch, info.customViewId);
        return {
            ...genericProps,
            className: "o_dashboard",
            board,
        };
    },
};

registry.category("views").add("board", boardView);

```

## File: static\src\add_to_board\add_to_board.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { registry } from "@web/core/registry";
import { useAutofocus, useService } from "@web/core/utils/hooks";
import { Component, useState } from "@odoo/owl";

const cogMenuRegistry = registry.category("cogMenu");

/**
 * 'Add to board' menu
 *
 * Component consisiting of a toggle button, a text input and an 'Add' button.
 * The first button is simply used to toggle the component and will determine
 * whether the other elements should be rendered.
 * The input will be given the name (or title) of the view that will be added.
 * Finally, the last button will send the name as well as some of the action
 * properties to the server to add the current view (and its context) to the
 * user's dashboard.
 * This component is only available in actions of type 'ir.actions.act_window'.
 * @extends Component
 */
export class AddToBoard extends Component {
    setup() {
        this.notification = useService("notification");
        this.rpc = useService("rpc");
        this.state = useState({ name: this.env.config.getDisplayName() });

        useAutofocus();
    }

    //---------------------------------------------------------------------
    // Protected
    //---------------------------------------------------------------------

    async addToBoard() {
        const { domain, globalContext } = this.env.searchModel;
        const { context, groupBys, orderBy } = this.env.searchModel.getPreFavoriteValues();
        const comparison = this.env.searchModel.comparison;
        const contextToSave = {
            ...Object.fromEntries(
                Object.entries(globalContext).filter(
                    (entry) => !entry[0].startsWith("search_default_")
                )
            ),
            ...context,
            orderedBy: orderBy,
            group_by: groupBys,
            dashboard_merge_domains_contexts: false,
        };
        if (comparison) {
            contextToSave.comparison = comparison;
        }

        const result = await this.rpc("/board/add_to_dashboard", {
            action_id: this.env.config.actionId || false,
            context_to_save: contextToSave,
            domain,
            name: this.state.name,
            view_mode: this.env.config.viewType,
        });

        if (result) {
            this.notification.add(
                _t("Please refresh your browser for the changes to take effect."),
                {
                    title: _t("“%s” added to dashboard", this.state.name),
                    type: "warning",
                }
            );
            this.state.name = this.env.config.getDisplayName();
        } else {
            this.notification.add(_t("Could not add filter to dashboard"), {
                type: "danger",
            });
        }
    }

    //---------------------------------------------------------------------
    // Handlers
    //---------------------------------------------------------------------

    /**
     * @param {KeyboardEvent} ev
     */
    onInputKeydown(ev) {
        if (ev.key === "Enter") {
            ev.preventDefault();
            this.addToBoard();
        }
    }
}

AddToBoard.template = "board.AddToBoard";
AddToBoard.components = { Dropdown };

export const addToBoardItem = {
    Component: AddToBoard,
    groupNumber: 20,
    isDisplayed: ({ config }) => {
        const { actionType, actionId, viewType } = config;
        return actionType === "ir.actions.act_window" && actionId && viewType !== "form";
    },
};

cogMenuRegistry.add("add-to-board", addToBoardItem, { sequence: 10 });

```

## File: static\src\add_to_board\add_to_board.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="board.AddToBoard">
        <Dropdown class="'o_add_to_board'">
            <t t-set-slot="toggler">
                <t t-call="Dashboard.DashboardIcon"/>
                Dashboard
            </t>
            <div class="px-3 pb-2">
                <label class="mb-2">Add to my dashboard</label>
                <input type="text" class="o_input mb-3" t-ref="autofocus" t-model.trim="state.name" t-on-keydown="onInputKeydown" />
                <button type="button" class="btn btn-primary" t-on-click="addToBoard">Add</button>
            </div>
        </Dropdown>
    </t>

    <t t-name="Dashboard.DashboardIcon">
        <svg viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg" class="oi-large">
            <path d="M4 8C4 5.79086 5.79086 4 8 4H25V21C25 23.2091 23.2091 25 21 25H4V8Z" fill="var(--oi-color, #FC868B)"/>
            <path d="M4 35C4 32.7909 5.79086 31 8 31H19V42C19 44.2091 17.2091 46 15 46H4V35Z" fill="var(--oi-color, #F9464C)"/>
            <path d="M25 46H29C31.2091 46 33 44.2091 33 42V31H29C26.7909 31 25 32.7909 25 35V46Z" fill="var(--oi-color, #FBB945)"/>
            <path d="M31 17L31 21C31 23.2091 32.7909 25 35 25L46 25V21C46 18.7909 44.2091 17 42 17L31 17Z" fill="var(--oi-color, #088BF5)"/>
            <path d="M31 4L31 8C31 10.2091 32.7909 12 35 12L46 12V8C46 5.79086 44.2091 4 42 4L31 4Z" fill="var(--oi-color, #2EBCFA)"/>
            <path d="M38 46H42C44.2091 46 46 44.2091 46 42V31H42C39.7909 31 38 32.7909 38 35V46Z" fill="var(--oi-color, #F78613)"/>
        </svg>
    </t>
</templates>

```

## File: views\board_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!--My Dashboard-->
        <record model="ir.ui.view" id="board_my_dash_view">
            <field name="name">My Dashboard</field>
            <field name="model">board.board</field>
            <field name="arch" type="xml">
                <form string="My Dashboard">
                    <board style="2-1">
                        <column>
                        </column>
                    </board>
                </form>
            </field>
        </record>

        <!--My Dashboard Action-->
        <record model="ir.actions.act_window" id="open_board_my_dash_action">
            <field name="name">My Dashboard</field>
            <field name="res_model">board.board</field>
            <field name="view_mode">form</field>
            <field name="context">{'disable_toolbar': True}</field>
            <field name="usage">menu</field>
            <field name="view_id" ref="board_my_dash_view"/>
        </record> 
    </data>
    <data>

        <!--My Dashboard Menu-->
        <menuitem
            id="menu_board_my_dash"
            name="My Dashboard"
            parent="spreadsheet_dashboard.spreadsheet_dashboard_menu_root"
            action="open_board_my_dash_action"
            sequence="100"/>
    </data>
</odoo>

```

