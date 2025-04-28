# Odoo Module: board

Category: Tools

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
    'category': 'Tools',
    'summary': 'Build your own dashboards',
    'description': """
Lets the user create a custom dashboard.
========================================

Allows users to create custom dashboard.
    """,
    'depends': ['base', 'web'],
    'data': [
        'security/ir.model.access.csv',
        'views/board_views.xml',
        'views/board_templates.xml',
    ],
    'qweb': ['static/src/xml/board.xml'],
    'application': True,
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
        action = request.env.ref('board.open_board_my_dash_action')

        if action and action['res_model'] == 'board.board' and action['views'][0][1] == 'form' and action_id:
            # Maybe should check the content instead of model board.board ?
            view_id = action['views'][0][0]
            board = request.env['board.board'].fields_view_get(view_id, 'form')
            if board and 'arch' in board:
                xml = ElementTree.fromstring(board['arch'])
                column = xml.find('./board/column')
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
                    arch = ElementTree.tostring(xml, encoding='unicode')
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

from odoo import api, models


class Board(models.AbstractModel):
    _name = 'board.board'
    _description = "Board"
    _auto = False

    @api.model
    def create(self, vals):
        return self

    @api.model
    def fields_view_get(self, view_id=None, view_type='form', toolbar=False, submenu=False):
        """
        Overrides orm field_view_get.
        @return: Dictionary of Fields, arch and toolbar.
        """

        res = super(Board, self).fields_view_get(view_id=view_id, view_type=view_type, toolbar=toolbar, submenu=submenu)

        custom_view = self.env['ir.ui.view.custom'].search([('user_id', '=', self.env.uid), ('ref_id', '=', view_id)], limit=1)
        if custom_view:
            res.update({'custom_view_id': custom_view.id,
                        'arch': custom_view.arch})
        res.update({
            'arch': self._arch_preprocessing(res['arch']),
            'toolbar': {'print': [], 'action': [], 'relate': []}
        })
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
access_board_board all,board.board,model_board_board,,1,0,0,0

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

## File: static\src\js\action_manager_board_action.js

```javascript
odoo.define('board.ActionManager', function (require) {
"use strict";

/**
 * The purpose of this file is to patch the ActionManager to properly generate
 * the flags for the 'ir.actions.act_window' of model 'board.board'.
 */

var ActionManager = require('web.ActionManager');

ActionManager.include({
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _executeWindowAction: function (action) {
        if (action.res_model === 'board.board' && action.view_mode === 'form') {
            action.target = 'inline';
            _.extend(action.flags, {
                hasSearchView: false,
                hasSidebar: false,
                headless: true,
            });
        }
        return this._super.apply(this, arguments);
    },
});

});

```

## File: static\src\js\add_to_board_menu.js

```javascript
odoo.define('board.AddToBoardMenu', function (require) {
"use strict";

var ActionManager = require('web.ActionManager');
var Context = require('web.Context');
var core = require('web.core');
var Domain = require('web.Domain');
var favorites_submenus_registry = require('web.favorites_submenus_registry');
var pyUtils = require('web.py_utils');
var Widget = require('web.Widget');

var _t = core._t;
var QWeb = core.qweb;

var AddToBoardMenu = Widget.extend({
    events: _.extend({}, Widget.prototype.events, {
        'click .o_add_to_board.o_menu_header': '_onMenuHeaderClick',
        'click .o_add_to_board_confirm_button': '_onAddToBoardConfirmButtonClick',
        'click .o_add_to_board_input': '_onAddToBoardInputClick',
        'keyup .o_add_to_board_input': '_onKeyUp',
    }),
    /**
     * @override
     * @param {Object} params
     * @param {Object} params.action an ir.actions description
     */
    init: function (parent, params) {
        this._super(parent);
        this.action = params.action;
        this.isOpen = false;
    },
    /**
     * @override
     */
    start: function () {
        if (this.action.id && this.action.type === 'ir.actions.act_window') {
            this._render();
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Closes the menu and render it.
     *
     */
    closeMenu: function () {
        this.isOpen = false;
        if (this.action.id && this.action.type === 'ir.actions.act_window') {
            this._render();
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * This is the main function for actually saving the dashboard.  This method
     * is supposed to call the route /board/add_to_dashboard with proper
     * information.
     *
     * @private
     * @returns {Promise}
     */
    _addToBoard: function () {
        var self = this;
        var searchQuery;
        // TO DO: for now the domains in query are evaluated.
        // This should be changed I think.
        this.trigger_up('get_search_query', {
            callback: function (query) {
                searchQuery = query;
            }
        });
        // TO DO: replace direct reference to action manager, controller, and currentAction in code below

        // AAB: trigger_up an event that will be intercepted by the controller,
        // as soon as the controller is the parent of the control panel
        var actionManager = this.findAncestor(function (ancestor) {
            return ancestor instanceof ActionManager;
        });
        var controller = actionManager.getCurrentController();

        var context = new Context(this.action.context);
        context.add(searchQuery.context);
        context.add({
            group_by: searchQuery.groupBy,
            orderedBy: searchQuery.orderedBy,
        });

        this.trigger_up('get_controller_query_params', {
            callback: function (controllerQueryParams) {
                var queryContext = controllerQueryParams.context;
                var allContext = _.extend(
                    _.omit(controllerQueryParams, ['context']),
                    queryContext
                );
                context.add(allContext);
            }
        });

        var domain = new Domain(this.action.domain || []);
        domain = Domain.prototype.normalizeArray(domain.toArray().concat(searchQuery.domain));

        var evalutatedContext = pyUtils.eval('context', context);
        for (var key in evalutatedContext) {
            if (evalutatedContext.hasOwnProperty(key) && /^search_default_/.test(key)) {
                delete evalutatedContext[key];
            }
        }
        evalutatedContext.dashboard_merge_domains_contexts = false;

        var name = this.$input.val();

        this.closeMenu();

        return self._rpc({
                route: '/board/add_to_dashboard',
                params: {
                    action_id: self.action.id || false,
                    context_to_save: evalutatedContext,
                    domain: domain,
                    view_mode: controller.viewType,
                    name: name,
                },
            })
            .then(function (r) {
                if (r) {
                    self.do_notify(
                        _.str.sprintf(_t("'%s' added to dashboard"), name),
                        _t('Please refresh your browser for the changes to take effect.')
                    );
                } else {
                    self.do_warn(_t("Could not add filter to dashboard"));
                }
            });
    },
    /**
     * Renders and focuses the unique input if it is visible.
     *
     * @private
     */
    _render: function () {
        var $el = QWeb.render('AddToBoardMenu', {widget: this});
        this._replaceElement($el);
        if (this.isOpen) {
            this.$input = this.$('.o_add_to_board_input');
            this.$input.val(this.action.name);
            this.$input.focus();
        }
    },
    /**
     * Hides and displays the submenu which allows adding custom filters.
     *
     * @private
     */
    _toggleMenu: function () {
        this.isOpen = !this.isOpen;
        this._render();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {jQueryEvent} event
     */
    _onAddToBoardInputClick: function (event) {
        event.preventDefault();
        event.stopPropagation();
        this.$input.focus();
    },
    /**
     * @private
     * @param {jQueryEvent} event
     */
    _onAddToBoardConfirmButtonClick: function (event) {
        event.preventDefault();
        event.stopPropagation();
        this._addToBoard();
    },
    /**
     * @private
     * @param {jQueryEvent} event
     */
    _onKeyUp: function (event) {
        if (event.which === $.ui.keyCode.ENTER) {
            this._addToBoard();
        }
    },
    /**
     * @private
     * @param {jQueryEvent} event
     */
    _onMenuHeaderClick: function (event) {
        event.preventDefault();
        event.stopPropagation();
        this._toggleMenu();
    },

});

favorites_submenus_registry.add('add_to_board_menu', AddToBoardMenu, 10);

return AddToBoardMenu;

});

```

## File: static\src\js\board_view.js

```javascript
odoo.define('board.BoardView', function (require) {
"use strict";

var Context = require('web.Context');
var config = require('web.config');
var core = require('web.core');
var dataManager = require('web.data_manager');
var Dialog = require('web.Dialog');
var Domain = require('web.Domain');
var FormController = require('web.FormController');
var FormRenderer = require('web.FormRenderer');
var FormView = require('web.FormView');
var pyUtils = require('web.py_utils');
var session  = require('web.session');
var viewRegistry = require('web.view_registry');

var _t = core._t;
var _lt = core._lt;
var QWeb = core.qweb;

var BoardController = FormController.extend({
    custom_events: _.extend({}, FormController.prototype.custom_events, {
        change_layout: '_onChangeLayout',
        enable_dashboard: '_onEnableDashboard',
        save_dashboard: '_saveDashboard',
        switch_view: '_onSwitchView',
    }),

    /**
     * @override
     */
    init: function (parent, model, renderer, params) {
        this._super.apply(this, arguments);
        this.customViewID = params.customViewID;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    getTitle: function () {
        if (this.inDashboard) {
            return _t("My Dashboard");
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Actually save a dashboard
     *
     * @returns {Promise}
     */
    _saveDashboard: function () {
        var board = this.renderer.getBoard();
        var arch = QWeb.render('DashBoard.xml', _.extend({}, board));
        return this._rpc({
                route: '/web/view/edit_custom',
                params: {
                    custom_id: this.customViewID,
                    arch: arch,
                }
            }).then(dataManager.invalidate.bind(dataManager));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {OdooEvent} event
     */
    _onChangeLayout: function (event) {
        var self = this;
        var dialog = new Dialog(this, {
            title: _t("Edit Layout"),
            $content: QWeb.render('DashBoard.layouts', _.clone(event.data))
        });
        dialog.opened().then(function () {
            dialog.$('li').click(function () {
                var layout = $(this).attr('data-layout');
                self.renderer.changeLayout(layout);
                self._saveDashboard();
                dialog.close();
            });
        });
        dialog.open();
    },
    /**
     * @private
     */
    _onEnableDashboard: function () {
        this.inDashboard = true;
    },
    /**
     * We need to intercept switch_view event coming from sub views, because we
     * don't actually want to switch view in dashboard, we want to do a
     * do_action (which will open the record in a different breadcrumb).
     *
     * @private
     * @param {OdooEvent} event
     */
    _onSwitchView: function (event) {
        event.stopPropagation();
        this.do_action({
            type: 'ir.actions.act_window',
            res_model: event.data.model,
            views: [[event.data.formViewID || false, 'form']],
            res_id: event.data.res_id,
        });
    },
});

var BoardRenderer = FormRenderer.extend({
    custom_events: _.extend({}, FormRenderer.prototype.custom_events, {
        update_filters: '_onUpdateFilters',
        switch_view: '_onSwitchView',
    }),
    events: _.extend({}, FormRenderer.prototype.events, {
        'click .oe_dashboard_column .oe_fold': '_onFoldClick',
        'click .oe_dashboard_link_change_layout': '_onChangeLayout',
        'click .oe_dashboard_column .oe_close': '_onCloseAction',
    }),

    /**
     * @override
     */
    init: function (parent, state, params) {
        this._super.apply(this, arguments);
        this.noContentHelp = params.noContentHelp;
        this.actionsDescr = {};
        this._boardSubcontrollers = []; // for board: controllers of subviews
        this._boardFormViewIDs = {}; // for board: mapping subview controller to form view id
    },
    /**
     * Call `on_attach_callback` for each subview
     *
     * @override
     */
    on_attach_callback: function () {
        _.each(this._boardSubcontrollers, function (controller) {
            if ('on_attach_callback' in controller) {
                controller.on_attach_callback();
            }
        });
    },
    /**
     * Call `on_detach_callback` for each subview
     *
     * @override
     */
    on_detach_callback: function () {
        _.each(this._boardSubcontrollers, function (controller) {
            if ('on_detach_callback' in controller) {
                controller.on_detach_callback();
            }
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @param {string} layout
     */
    changeLayout: function (layout) {
        var $dashboard = this.$('.oe_dashboard');
        var current_layout = $dashboard.attr('data-layout');
        if (current_layout !== layout) {
            var clayout = current_layout.split('-').length,
                nlayout = layout.split('-').length,
                column_diff = clayout - nlayout;
            if (column_diff > 0) {
                var $last_column = $();
                $dashboard.find('.oe_dashboard_column').each(function (k, v) {
                    if (k >= nlayout) {
                        $(v).find('.oe_action').appendTo($last_column);
                    } else {
                        $last_column = $(v);
                    }
                });
            }
            $dashboard.toggleClass('oe_dashboard_layout_' + current_layout + ' oe_dashboard_layout_' + layout);
            $dashboard.attr('data-layout', layout);
        }
    },
    /**
     * Returns a representation of the current dashboard
     *
     * @returns {Object}
     */
    getBoard: function () {
        var self = this;
        var board = {
            form_title : this.arch.attrs.string,
            style : this.$('.oe_dashboard').attr('data-layout'),
            columns : [],
        };
        this.$('.oe_dashboard_column').each(function () {
            var actions = [];
            $(this).find('.oe_action').each(function () {
                var actionID = $(this).attr('data-id');
                var newAttrs = _.clone(self.actionsDescr[actionID]);

                /* prepare attributes as they should be saved */
                if (newAttrs.modifiers) {
                    newAttrs.modifiers = JSON.stringify(newAttrs.modifiers);
                }
                actions.push(newAttrs);
            });
            board.columns.push(actions);
        });
        return board;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} params
     * @param {jQueryElement} params.$node
     * @param {integer} params.actionID
     * @param {Object} params.context
     * @param {any[]} params.domain
     * @param {string} params.viewType
     * @returns {Promise}
     */
    _createController: function (params) {
        var self = this;
        return this._rpc({
                route: '/web/action/load',
                params: {action_id: params.actionID}
            })
            .then(function (action) {
                if (!action) {
                    // the action does not exist anymore
                    return Promise.resolve();
                }
                var evalContext = new Context(session.user_context, params.context).eval();
                if (evalContext.group_by && evalContext.group_by.length === 0) {
                    delete evalContext.group_by;
                }
                // tz and lang are saved in the custom view
                // override the language to take the current one
                var rawContext = new Context(action.context, evalContext, {lang: session.user_context.lang});
                var context = pyUtils.eval('context', rawContext, evalContext);
                var domain = params.domain || pyUtils.eval('domain', action.domain || '[]', action.context);
                action.context = context;
                action.domain = domain;

                // When creating a view, `action.views` is expected to be an array of dicts, while
                // '/web/action/load' returns an array of arrays.
                action._views = action.views;
                action.views = $.map(action.views, function (view) { return {viewID: view[0], type: view[1]}});

                var viewType = params.viewType || action._views[0][1];
                var view = _.find(action._views, function (descr) {
                    return descr[1] === viewType;
                }) || [false, viewType];
                return self.loadViews(action.res_model, context, [view])
                           .then(function (viewsInfo) {
                    var viewInfo = viewsInfo[viewType];
                    var View = viewRegistry.get(viewType);
                    var view = new View(viewInfo, {
                        action: action,
                        hasSelectors: false,
                        modelName: action.res_model,
                        searchQuery: {
                            context: context,
                            domain: domain,
                            groupBy: typeof context.group_by === 'string' && context.group_by ? [context.group_by] : context.group_by || [],
                            orderedBy: context.orderedBy || [],
                        },
                        withControlPanel: false,
                    });
                    return view.getController(self).then(function (controller) {
                        self._boardFormViewIDs[controller.handle] = _.first(
                            _.find(action._views, function (descr) {
                                return descr[1] === 'form';
                            })
                        );
                        self._boardSubcontrollers.push(controller);
                        return controller.appendTo(params.$node);
                    });
                });
            });
    },
    /**
     * @private
     * @param {Object} node
     * @returns {jQueryElement}
     */
    _renderTagBoard: function (node) {
        var self = this;
        // we add the o_dashboard class to the renderer's $el. This means that
        // this function has a side effect.  This is ok because we assume that
        // once we have a '<board>' tag, we are in a special dashboard mode.
        this.$el.addClass('o_dashboard');
        this.trigger_up('enable_dashboard');

        var hasAction = _.detect(node.children, function (column) {
            return _.detect(column.children,function (element){
                return element.tag === "action"? element: false;
            });
        });
        if (!hasAction) {
            return $(QWeb.render('DashBoard.NoContent'));
        }

        // We should start with three columns available
        node = $.extend(true, {}, node);

        // no idea why master works without this, but whatever
        if (!('layout' in node.attrs)) {
            node.attrs.layout = node.attrs.style;
        }
        for (var i = node.children.length; i < 3; i++) {
            node.children.push({
                tag: 'column',
                attrs: {},
                children: []
            });
        }

        // register actions, alongside a generated unique ID
        _.each(node.children, function (column, column_index) {
            _.each(column.children, function (action, action_index) {
                action.attrs.id = 'action_' + column_index + '_' + action_index;
                self.actionsDescr[action.attrs.id] = action.attrs;
            });
        });

        var $html = $('<div>').append($(QWeb.render('DashBoard', {node: node, isMobile: config.device.isMobile})));
        this._boardSubcontrollers = []; // dashboard controllers are reset on re-render

        // render each view
        _.each(this.actionsDescr, function (action) {
            self.defs.push(self._createController({
                $node: $html.find('.oe_action[data-id=' + action.id + '] .oe_content'),
                actionID: _.str.toNumber(action.name),
                context: action.context,
                domain: Domain.prototype.stringToArray(action.domain, {}),
                viewType: action.view_mode,
            }));
        });
        $html.find('.oe_dashboard_column').sortable({
            connectWith: '.oe_dashboard_column',
            handle: '.oe_header',
            scroll: false
        }).bind('sortstop', function () {
            self.trigger_up('save_dashboard');
        });

        return $html;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onChangeLayout: function () {
        var currentLayout = this.$('.oe_dashboard').attr('data-layout');
        this.trigger_up('change_layout', {currentLayout: currentLayout});
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onCloseAction: function (event) {
        var self = this;
        var $container = $(event.currentTarget).parents('.oe_action:first');
        Dialog.confirm(this, (_t("Are you sure you want to remove this item?")), {
            confirm_callback: function () {
                $container.remove();
                self.trigger_up('save_dashboard');
            },
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onFoldClick: function (event) {
        var $e = $(event.currentTarget);
        var $action = $e.closest('.oe_action');
        var id = $action.data('id');
        var actionAttrs = this.actionsDescr[id];

        if ($e.is('.oe_minimize')) {
            actionAttrs.fold = '1';
        } else {
            delete(actionAttrs.fold);
        }
        $e.toggleClass('oe_minimize oe_maximize');
        $action.find('.oe_content').toggle();
        this.trigger_up('save_dashboard');
    },
    /**
     * Let FormController know which form view it should display based on the
     * window action of the sub controller that is switching view
     *
     * @private
     * @param {OdooEvent} event
     */
    _onSwitchView: function (event) {
        event.data.formViewID = this._boardFormViewIDs[event.target.handle];
    },
    /**
     * Stops the propagation of 'update_filters' events triggered by the
     * controllers instantiated by the dashboard to prevent them from
     * interfering with the ActionManager.
     *
     * @private
     * @param {OdooEvent} event
     */
    _onUpdateFilters: function (event) {
        event.stopPropagation();
    },
});

var BoardView = FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller: BoardController,
        Renderer: BoardRenderer,
    }),
    display_name: _lt('Board'),

    /**
     * @override
     */
    init: function (viewInfo) {
        this._super.apply(this, arguments);
        this.controllerParams.customViewID = viewInfo.custom_view_id;
    },
});

return BoardView;

});


odoo.define('board.viewRegistry', function (require) {
"use strict";

var BoardView = require('board.BoardView');

var viewRegistry = require('web.view_registry');

viewRegistry.add('board', BoardView);

});

```

## File: static\src\xml\board.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<template>
<t t-name="DashBoard">
    <t t-if="isMobile">
        <t t-set="node.attrs.layout" t-value="1"/>
    </t>
    <t t-if="!isMobile">
        <div class="oe_dashboard_links">
            <button type="button" class="button oe_dashboard_link_change_layout btn btn-secondary"
                title="Change Layout..">
                <img src="/board/static/src/img/layout_1-1-1.png" width="16" height="16" alt=""/>
                <span> Change Layout </span>
            </button>
        </div>
    </t>
    <table t-att-data-layout="node.attrs.layout" t-attf-class="oe_dashboard oe_dashboard_layout_#{node.attrs.layout}" cellspacing="0" cellpadding="0" border="0">
    <tr>
        <td t-foreach="node.children" t-as="column" t-if="column.tag == 'column'"
             t-att-id="'column_' + column_index" t-attf-class="oe_dashboard_column index_#{column_index}">

            <t t-foreach="column.children" t-as="action" t-if="action.tag == 'action'" t-call="DashBoard.action"/>
        </td>
    </tr>
    </table>
</t>
<t t-name="DashBoard.action">
    <div t-att-data-id="action.attrs.id" class="oe_action">
        <h2 t-attf-class="oe_header #{action.attrs.string ? '' : 'oe_header_empty'}">
            <span class="oe_header_txt"> <t t-esc="action.attrs.string"/> </span>
            <input class = "oe_header_text" type="text"/>
            <t t-if="!action.attrs.string">&amp;nbsp;</t>
            <span class='oe_icon oe_close'></span>
            <span class='oe_icon oe_minimize oe_fold' t-if="!action.attrs.fold"></span>
            <span class='oe_icon oe_maximize oe_fold' t-if="action.attrs.fold"></span>
        </h2>
        <div t-att-class="'oe_content' + (action.attrs.fold ? ' oe_folded' : '')"/>
    </div>
</t>
<t t-name="DashBoard.layouts">
    <div class="oe_dashboard_layout_selector">
        <p>
            <strong>Choose dashboard layout</strong>
        </p>
        <ul>
            <li t-foreach="'1 1-1 1-1-1 1-2 2-1'.split(' ')" t-as="layout" t-att-data-layout="layout">
                <img t-attf-src="/board/static/src/img/layout_#{layout}.png" alt=""/>
                <i t-if="layout == currentLayout" class="oe_dashboard_selected_layout fa fa-check fa-lg text-success" aria-label='Layout' role="img" title="Layout"/>
            </li>
        </ul>
    </div>
</t>
<t t-name="DashBoard.NoContent">
    <div class="o_view_nocontent">
        <div class="o_nocontent_help">
            <p class="o_view_nocontent_neutral_face">
                Your personal dashboard is empty
            </p><p>
                To add your first report into this dashboard, go to any
                menu, switch to list or graph view, and click <i>"Add to
                Dashboard"</i> in the extended search options.
            </p><p>
                You can filter and group data before inserting into the
                dashboard using the search options.
            </p>
        </div>
    </div>
</t>
<t t-name="DashBoard.xml">
    <form t-att-string="form_title">
        <board t-att-style="style">
            <column t-foreach="columns" t-as="column">
                <action t-foreach="column" t-as="action" t-att="action"/>
            </column>
        </board>
    </form>
</t>
<div t-name="HomeWidget" class="oe_dashboard_home_widget"/>
<t t-name="HomeWidget.content">
    <h3><t t-esc="widget.title"/></h3>
    <iframe width="100%" frameborder="0" t-att-src="url"/>
</t>

<div t-name="AddToBoardMenu">
    <button type="button" class="dropdown-item o_add_to_board o_menu_header">Add to my Dashboard</button>
    <div t-if="widget.isOpen" class="dropdown-item-text o_add_to_board">
        <input class="o_input o_add_to_board_input" type="text"/>
    </div>
    <div t-if="widget.isOpen"  class="dropdown-item-text o_add_to_board">
        <button type="button" class="btn btn-primary o_add_to_board_confirm_button">Add</button>
    </div>
</div>

<t t-name="SearchView.addtodashboard">
    <a href="#" class="dropdown-item o_add_to_dashboard_link o_closed_menu">Add to my Dashboard</a>
    <div class="dropdown-item-text o_add_to_dashboard">
        <input class="o_input o_add_to_dashboard_input" type="text"/>
    </div>
    <div class="dropdown-item-text o_add_to_dashboard">
        <button type="button" class="btn btn-primary o_add_to_dashboard_button">Add</button>
    </div>
</t>
</template>

```

## File: views\board_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="assets_backend" name="board assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/board/static/src/scss/dashboard.scss"/>
                <script type="text/javascript" src="/board/static/src/js/action_manager_board_action.js"></script>
                <script type="text/javascript" src="/board/static/src/js/board_view.js"></script>
                <script type="text/javascript" src="/board/static/src/js/add_to_board_menu.js"></script>
            </xpath>
        </template>

         <template id="qunit_suite" name="dashboard_tests" inherit_id="web.qunit_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/board/static/tests/dashboard_tests.js"></script>
            </xpath>
        </template>
</odoo>

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

        <!--My Dashboard Menu-->
        <menuitem 
            id="menu_board_my_dash"
            parent="base.menu_board_root"
            action="open_board_my_dash_action"
            sequence="5"/>
    </data>
</odoo>

```

