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
    'application': False,
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
                    request.env['ir.ui.view.custom'].create({
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

        custom_view = self.env['ir.ui.view.custom'].search([('user_id', '=', self.env.uid), ('ref_id', '=', view_id)], limit=1)
        if custom_view:
            res.update({'custom_view_id': custom_view.id,
                        'arch': custom_view.arch})
        res['arch'] = self._arch_preprocessing(res['arch'])
        return res

    @api.model
    def get_views(self, views, options=None):
        res = super().get_views(views, options)
        for view in res['views'].values():
            view['toolbar'] = {'print': [], 'action': [], 'relate': []}
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
access_board_board_all,board.board,model_board_board,,1,0,0,0
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

## File: static\src\board_action.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { View } from "@web/views/view";
import { makeContext } from "@web/core/context";

const { Component, onWillStart } = owl;

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

    <t t-name="board.BoardAction" owl="1">
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

import { browser } from "@web/core/browser/browser";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { useService } from "@web/core/utils/hooks";
import { renderToString } from "@web/core/utils/render";
import { useSortable } from "@web/core/utils/sortable";
import { standardViewProps } from "@web/views/standard_view_props";
import { BoardAction } from "./board_action";

const { blockDom, Component, useState, useRef } = owl;

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
            body: this.env._t("Are you sure that you want to remove this item?"),
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

    <t t-name="board.BoardView" owl="1">
        <div class="o_dashboard d-flex flex-column">
            <t t-if="board.isEmpty">
                <t t-call="board.NoContent"/>
            </t>
            <t t-else="">
                <t t-call="board.Content"/>
            </t>
        </div>
    </t>

    <t t-name="board.Content" owl="1">
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

    <t t-name="board.NoContent" owl="1">
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

    <t t-name="board.arch" owl="1">
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

import { _lt } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { BoardController } from "./board_controller";
import { XMLParser } from "@web/core/utils/xml";
import { Domain } from "@web/core/domain";

export class BoardArchParser extends XMLParser {
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

        this.visitXML(arch, (node) => {
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
                    let action = {
                        id: nextId++,
                        title: node.getAttribute("string"),
                        actionId: parseInt(node.getAttribute("name"), 10),
                        viewMode: node.getAttribute("view_mode"),
                        context: node.getAttribute("context"),
                        isFolded,
                    };
                    if (node.hasAttribute("domain")) {
                        let domain = node.getAttribute("domain");
                        // Since bfadb8e491fe2acda63a79f9577eaaec8a1c8d9c some databases might have
                        // double-encoded domains in the db, so we need to unescape them before use.
                        // TODO: remove unescape in saas-16.3
                        domain = _.unescape(domain);
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
    display_name: _lt("Board"),
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

import { Dropdown } from "@web/core/dropdown/dropdown";
import { registry } from "@web/core/registry";
import { useAutofocus, useService } from "@web/core/utils/hooks";
import { sprintf } from "@web/core/utils/strings";

const { Component, useState } = owl;
const favoriteMenuRegistry = registry.category("favoriteMenu");

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
                this.env._t("Please refresh your browser for the changes to take effect."),
                {
                    title: sprintf(this.env._t(`"%s" added to dashboard`), this.state.name),
                    type: "warning",
                }
            );
            this.state.name = this.env.config.getDisplayName();
        } else {
            this.notification.add(this.env._t("Could not add filter to dashboard"), {
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
    groupNumber: 4,
    isDisplayed: ({ config }) => config.actionType === "ir.actions.act_window" && config.actionId,
};

favoriteMenuRegistry.add("add-to-board", addToBoardItem, { sequence: 10 });

```

## File: static\src\add_to_board\add_to_board.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="board.AddToBoard" owl="1">
        <Dropdown class="'o_add_to_board'">
            <t t-set-slot="toggler">Add to my dashboard</t>
            <div class="px-3 py-2">
                <input type="text" class="o_input" t-ref="autofocus" t-model.trim="state.name" t-on-keydown="onInputKeydown" />
            </div>
            <div class="px-3 py-2">
                <button type="button" class="btn btn-primary" t-on-click="addToBoard">Add</button>
            </div>
        </Dropdown>
    </t>

</templates>

```

## File: static\src\add_to_board\legacy_add_to_board.js

```javascript
odoo.define("board.AddToBoardMenu", function (require) {
    "use strict";

    const Context = require("web.Context");
    const Domain = require("web.Domain");
    const { Dropdown } = require("@web/core/dropdown/dropdown");
    const FavoriteMenu = require("web.FavoriteMenu");
    const { sprintf } = require("web.utils");
    const { useAutofocus } = require("@web/core/utils/hooks");

    const { Component, useState } = owl;

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
     */
    class AddToBoardMenu extends Component {
        setup() {
            this.interactive = true;
            this.state = useState({
                name: this.env.action.name || "",
                open: false,
            });

            useAutofocus();
        }

        //---------------------------------------------------------------------
        // Private
        //---------------------------------------------------------------------

        /**
         * This is the main function for actually saving the dashboard.  This method
         * is supposed to call the route /board/add_to_dashboard with proper
         * information.
         * @private
         */
        async addToBoard() {
            const searchQuery = this.env.searchModel.get("query");
            const context = new Context(this.env.action.context);
            context.add(searchQuery.context);
            context.add({
                group_by: searchQuery.groupBy,
                orderedBy: searchQuery.orderedBy,
            });
            if (
                searchQuery.timeRanges &&
                Object.prototype.hasOwnProperty.call(searchQuery.timeRanges, "fieldName")
            ) {
                context.add({
                    comparison: searchQuery.timeRanges,
                });
            }
            let controllerQueryParams;
            this.env.searchModel.trigger("get-controller-query-params", (params) => {
                controllerQueryParams = params || {};
            });
            controllerQueryParams.context = controllerQueryParams.context || {};
            const queryContext = controllerQueryParams.context;
            delete controllerQueryParams.context;
            context.add(Object.assign(controllerQueryParams, queryContext));

            const domainArray = new Domain(this.env.action.domain || []);
            const domain = Domain.prototype.normalizeArray(
                domainArray.toArray().concat(searchQuery.domain)
            );

            const evalutatedContext = context.eval();
            for (const key in evalutatedContext) {
                if (
                    Object.prototype.hasOwnProperty.call(evalutatedContext, key) &&
                    /^search_default_/.test(key)
                ) {
                    delete evalutatedContext[key];
                }
            }
            evalutatedContext.dashboard_merge_domains_contexts = false;

            Object.assign(this.state, {
                name: $(".o_input").val() || "",
                open: false,
            });

            const result = await this.env.services.rpc({
                route: "/board/add_to_dashboard",
                params: {
                    action_id: this.env.action.id || false,
                    context_to_save: evalutatedContext,
                    domain: domain,
                    view_mode: this.env.view.type,
                    name: this.state.name,
                },
            });
            if (result) {
                this.env.services.notification.notify({
                    title: sprintf(this.env._t("'%s' added to dashboard"), this.state.name),
                    message: this.env._t(
                        "Please refresh your browser for the changes to take effect."
                    ),
                    type: "warning",
                });
            } else {
                this.env.services.notification.notify({
                    message: this.env._t("Could not add filter to dashboard"),
                    type: "danger",
                });
            }
        }

        //---------------------------------------------------------------------
        // Handlers
        //---------------------------------------------------------------------

        /**
         * @private
         * @param {KeyboardEvent} ev
         */
        onInputKeydown(ev) {
            switch (ev.key) {
                case "Enter":
                    ev.preventDefault();
                    this.addToBoard();
                    break;
                case "Escape":
                    // Gives the focus back to the component.
                    ev.preventDefault();
                    ev.target.blur();
                    break;
            }
        }

        //---------------------------------------------------------------------
        // Static
        //---------------------------------------------------------------------

        /**
         * @param {Object} env
         * @returns {boolean}
         */
        static shouldBeDisplayed(env) {
            return env.action.type === "ir.actions.act_window";
        }
    }

    AddToBoardMenu.props = {};
    AddToBoardMenu.template = "board.AddToBoard";
    AddToBoardMenu.components = { Dropdown };

    FavoriteMenu.registry.add("add-to-board-menu", AddToBoardMenu, 10);

    return AddToBoardMenu;
});

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

