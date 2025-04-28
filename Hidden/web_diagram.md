# Odoo Module: web_diagram

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Odoo Web Diagram',
    'category': 'Hidden',
    'description': """
Openerp Web Diagram view.
=========================

""",
    'version': '2.0',
    'depends': ['web'],
    'data': [
        'views/web_diagram_templates.xml',
    ],
    'qweb': [
        'static/src/xml/*.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import odoo.http as http

from odoo.tools.safe_eval import safe_eval


class DiagramView(http.Controller):

    @http.route('/web_diagram/diagram/get_diagram_info', type='json', auth='user')
    def get_diagram_info(self, id, model, node, connector,
                         src_node, des_node, label, **kw):

        visible_node_fields = kw.get('visible_node_fields', [])
        invisible_node_fields = kw.get('invisible_node_fields', [])
        node_fields_string = kw.get('node_fields_string', [])
        connector_fields = kw.get('connector_fields', [])
        connector_fields_string = kw.get('connector_fields_string', [])

        bgcolors = {}
        shapes = {}
        bgcolor = kw.get('bgcolor', '')
        shape = kw.get('shape', '')

        if bgcolor:
            for color_spec in bgcolor.split(';'):
                if color_spec:
                    colour, color_state = color_spec.split(':')
                    bgcolors[colour] = color_state

        if shape:
            for shape_spec in shape.split(';'):
                if shape_spec:
                    shape_colour, shape_color_state = shape_spec.split(':')
                    shapes[shape_colour] = shape_color_state

        ir_view = http.request.env['ir.ui.view']
        graphs = ir_view.graph_get(int(id), model, node, connector, src_node,
                                   des_node, label, (140, 180))
        nodes = graphs['nodes']
        transitions = graphs['transitions']
        isolate_nodes = {}
        for blnk_node in graphs['blank_nodes']:
            isolate_nodes[blnk_node['id']] = blnk_node
        y = [
            t['y']
            for t in nodes.values()
            if t['x'] == 20
            if t['y']
        ]
        y_max = (y and max(y)) or 120

        connectors = {}
        list_tr = []

        for tr in transitions:
            list_tr.append(tr)
            connectors.setdefault(tr, {
                'id': int(tr),
                's_id': transitions[tr][0],
                'd_id': transitions[tr][1]
            })

        connector_model = http.request.env[connector]
        data_connectors = connector_model.search([('id', 'in', list_tr)]).read(connector_fields)

        for tr in data_connectors:
            transition_id = str(tr['id'])
            _sourceid, label = graphs['label'][transition_id]
            t = connectors[transition_id]
            t.update(
                source=tr[src_node][1],
                destination=tr[des_node][1],
                options={},
                signal=label
            )

            for i, fld in enumerate(connector_fields):
                t['options'][connector_fields_string[i]] = tr[fld]

        fields = http.request.env['ir.model.fields']
        field = fields.search([('model', '=', model), ('relation', '=', node)])
        node_act = http.request.env[node]
        search_acts = node_act.search([(field.relation_field, '=', id)])
        data_acts = search_acts.read(invisible_node_fields + visible_node_fields)

        for act in data_acts:
            n = nodes.get(str(act['id']))
            if not n:
                n = isolate_nodes.get(act['id'], {})
                y_max += 140
                n.update(x=20, y=y_max)
                nodes[act['id']] = n

            n.update(
                id=act['id'],
                color='white',
                options={}
            )
            for color, expr in bgcolors.items():
                if safe_eval(expr, act):
                    n['color'] = color

            for shape, expr in shapes.items():
                if safe_eval(expr, act):
                    n['shape'] = shape

            for i, fld in enumerate(visible_node_fields):
                n['options'][node_fields_string[i]] = act[fld]

        _id, name = http.request.env[model].browse([id]).name_get()[0]
        return dict(nodes=nodes,
                    conn=connectors,
                    display_name=name,
                    parent_field=graphs['node_parent_field'])

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: static\lib\js\jquery.mousewheel.js

```javascript
/*! Copyright (c) 2011 Brandon Aaron (http://brandonaaron.net)
 * Licensed under the MIT License (LICENSE.txt).
 *
 * Thanks to: http://adomas.org/javascript-mouse-wheel/ for some pointers.
 * Thanks to: Mathias Bank(http://www.mathias-bank.de) for a scope bug fix.
 * Thanks to: Seamus Leahy for adding deltaX and deltaY
 *
 * Version: 3.0.6
 * 
 * Requires: 1.2.2+
 */

(function($) {

var types = ['DOMMouseScroll', 'mousewheel'];

if ($.event.fixHooks) {
    for ( var i=types.length; i; ) {
        $.event.fixHooks[ types[--i] ] = $.event.mouseHooks;
    }
}

$.event.special.mousewheel = {
    setup: function() {
        if ( this.addEventListener ) {
            for ( var i=types.length; i; ) {
                this.addEventListener( types[--i], handler, false );
            }
        } else {
            this.onmousewheel = handler;
        }
    },
    
    teardown: function() {
        if ( this.removeEventListener ) {
            for ( var i=types.length; i; ) {
                this.removeEventListener( types[--i], handler, false );
            }
        } else {
            this.onmousewheel = null;
        }
    }
};

$.fn.extend({
    mousewheel: function(fn) {
        return fn ? this.bind("mousewheel", fn) : this.trigger("mousewheel");
    },
    
    unmousewheel: function(fn) {
        return this.unbind("mousewheel", fn);
    }
});


function handler(event) {
    var orgEvent = event || window.event, args = [].slice.call( arguments, 1 ), delta = 0, returnValue = true, deltaX = 0, deltaY = 0;
    event = $.event.fix(orgEvent);
    event.type = "mousewheel";
    
    // Old school scrollwheel delta
    if ( orgEvent.wheelDelta ) { delta = orgEvent.wheelDelta/120; }
    if ( orgEvent.detail     ) { delta = -orgEvent.detail/3; }
    
    // New school multidimensional scroll (touchpads) deltas
    deltaY = delta;
    
    // Gecko
    if ( orgEvent.axis !== undefined && orgEvent.axis === orgEvent.HORIZONTAL_AXIS ) {
        deltaY = 0;
        deltaX = -1*delta;
    }
    
    // Webkit
    if ( orgEvent.wheelDeltaY !== undefined ) { deltaY = orgEvent.wheelDeltaY/120; }
    if ( orgEvent.wheelDeltaX !== undefined ) { deltaX = -1*orgEvent.wheelDeltaX/120; }
    
    // Add event and delta to the front of the arguments
    args.unshift(event, delta, deltaX, deltaY);
    
    return ($.event.dispatch || $.event.handle).apply(this, args);
}

})(jQuery);

```

## File: static\src\js\diagram_controller.js

```javascript
odoo.define('web_diagram.DiagramController', function (require) {
"use strict";

var AbstractController = require('web.AbstractController');
var core = require('web.core');
var Dialog = require('web.Dialog');
var view_dialogs = require('web.view_dialogs');

var _t = core._t;
var QWeb = core.qweb;
var FormViewDialog = view_dialogs.FormViewDialog;

/**
 * Diagram Controller
 */
var DiagramController = AbstractController.extend({
    className: 'o_diagram_view',
    custom_events: {
        add_edge: '_onAddEdge',
        edit_edge: '_onEditEdge',
        edit_node: '_onEditNode',
        remove_edge: '_onRemoveEdge',
        remove_node: '_onRemoveNode',
    },
    /**
     * @override
     * @param {Widget} parent
     * @param {DiagramModel} model
     * @param {DiagramRenderer} renderer
     * @param {Object} params
     */
    init: function (parent, model, renderer, params) {
        this._super.apply(this, arguments);
        this.domain = params.domain || [];
        this.context = params.context;
        this.ids = params.ids;
        this.currentId = params.currentId;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Render the buttons according to the DiagramView.buttons template and add
     * listeners on it. Set this.$buttons with the produced jQuery element
     *
     * @param {jQuery} [$node] a jQuery node where the rendered buttons should
     *   be inserted $node may be undefined, in which case they are inserted
     *   into this.options.$buttons
     */
    renderButtons: function ($node) {
        this.$buttons = $(QWeb.render("DiagramView.buttons", {widget: this}));
        this.$buttons.on('click', '.o_diagram_new_button', this._addNode.bind(this));
        this.$buttons.appendTo($node);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Creates a popup to add a node to the diagram
     *
     * @private
     */
    _addNode: function () {
        var state = this.model.get();
        var pop = new FormViewDialog(this, {
            res_model: state.node_model,
            domain: this.domain,
            context: this.context,
            title: _.str.sprintf("%s %s", _t("Create:"), _t('Activity')),
            disable_multiple_selection: true,
            on_saved: this.reload.bind(this, {}),
        }).open();

        // manually trigger a 'field_changed' on the dialog's form_view to set
        // the default value of the parent_id field
        pop.opened().then(function () {
            var changes = {};
            changes[state.parent_field] = {
                id: state.res_id
            };
            pop.form_view.trigger_up('field_changed', {
                dataPointID: pop.form_view.handle,
                changes: changes,
            });
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Custom event handler that opens a popup to add an edge from given source
     * and dest nodes.
     *
     * @private
     * @param {OdooEvent} event
     */
    _onAddEdge: function (event) {
        var self = this;
        var state = this.model.get();
        var pop = new FormViewDialog(self, {
            res_model: state.connector_model,
            domain: this.domain,
            context: this.context,
            title: _.str.sprintf("%s %s", _t("Create:"), _t('Transition')),
            disable_multiple_selection: true,
        }).open();

        // manually trigger a 'field_changed' on the dialog's form_view to set
        // the default source and destination values
        pop.opened().then(function () {
            var changes = {};
            changes[state.connectors.attrs.source] = {
                id: event.data.source_id
            };
            changes[state.connectors.attrs.destination] = {
                id: event.data.dest_id
            };
            pop.form_view.trigger_up('field_changed', {
                dataPointID: pop.form_view.handle,
                changes: changes,
            });
        });
        pop.on('closed', this, this.reload.bind(this, {}));
    },
    /**
     * Custom event handler that opens a popup to edit an edge given its id
     *
     * @private
     * @param {OdooEvent} event
     */
    _onEditEdge: function (event) {
        var state = this.model.get();
        new FormViewDialog(this, {
            res_model: state.connector_model,
            res_id: parseInt(event.data.id, 10),
            context: this.context,
            title: _.str.sprintf("%s %s", _t("Open:"), _t('Transition')),
            on_saved: this.reload.bind(this, {}),
        }).open();
    },
    /**
     * Custom event handler that opens a popup to edit the content of a node
     * given its id
     *
     * @private
     * @param {OdooEvent} event
     */
    _onEditNode: function (event) {
        var state = this.model.get();
        new FormViewDialog(this, {
            res_model: state.node_model,
            res_id: event.data.id,
            context: this.context,
            title: _.str.sprintf("%s %s", _t("Open:"), _t('Activity')),
            on_saved: this.reload.bind(this, {}),
        }).open();
    },
    /**
     * Custom event handler that removes an edge given its id
     *
     * @private
     * @param {OdooEvent} event
     */
    _onRemoveEdge: function (event) {
        var self = this;
        Dialog.confirm(this, (_t("Are you sure you want to remove this transition?")), {
            confirm_callback: function () {
                var state = self.model.get();
                self._rpc({
                        model: state.connector_model,
                        method: 'unlink',
                        args: [event.data.id],
                    })
                    .then(self.reload.bind(self, {}));
            },
        });
    },
    /**
     * Custom event handler that removes a node given its id
     *
     * @private
     * @param {OdooEvent} event
     */
    _onRemoveNode: function (event) {
        var self = this;
        var msg = _t("Are you sure you want to remove this node ? This will remove its connected transitions as well.");
        Dialog.confirm(this, (msg), {
            confirm_callback: function () {
                var state = self.model.get();
                self._rpc({
                        model: state.node_model,
                        method: 'unlink',
                        args: [event.data.id],
                    })
                    .then(self.reload.bind(self, {}));
            },
        });
    },
});

return DiagramController;

});

```

## File: static\src\js\diagram_model.js

```javascript
odoo.define('web_diagram.DiagramModel', function (require) {
"use strict";

var AbstractModel = require('web.AbstractModel');

/**
 * DiagramModel
 */
var DiagramModel = AbstractModel.extend({
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {Object}
     */
    get: function () {
        return $.extend(true, {}, {
            labels: this.labels,
            nodes: this.datanodes,
            edges: this.edges,
            node_model: this.node_model,
            parent_field: this.parent_field,
            res_id: this.res_id,
            connector_model: this.connector_model,
            connectors: this.connectors,
        });
    },
    /**
     * @override
     * @param {Object} params
     * @returns {Promise}
     */
    load: function (params) {
        this.modelName = params.modelName;
        this.res_id = params.res_id;
        this.node_model = params.node_model;
        this.connector_model = params.connector_model;
        this.connectors = params.connectors;
        this.nodes = params.nodes;
        this.visible_nodes = params.visible_nodes;
        this.invisible_nodes = params.invisible_nodes;
        this.node_fields_string = params.node_fields_string;
        this.connector_fields_string = params.connector_fields_string;
        this.labels = params.labels;

        return this._fetchDiagramInfo();
    },
    reload: function () {
        return this._fetchDiagramInfo();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {any} record
     * @returns {Promise}
     */
    _fetchDiagramInfo: function () {
        var self = this;
        return this._rpc({
                route: '/web_diagram/diagram/get_diagram_info',
                params: {
                    id: this.res_id,
                    model: this.modelName,
                    node: this.node_model,
                    connector: this.connector_model,
                    src_node: this.connectors.attrs.source,
                    des_node: this.connectors.attrs.destination,
                    label: this.connectors.attrs.label || false,
                    bgcolor: this.nodes.attrs.bgcolor,
                    shape: this.nodes.attrs.shape,
                    visible_nodes: this.visible_nodes,
                    invisible_nodes: this.invisible_nodes,
                    node_fields_string: this.node_fields_string,
                    connector_fields_string: this.connector_fields_string,
                },
            })
            .then(function (data) {
                self.datanodes = data.nodes;
                self.edges = data.conn;
                self.parent_field = data.parent_field;
            });
    },
});

return DiagramModel;
});

```

## File: static\src\js\diagram_renderer.js

```javascript
odoo.define('web_diagram.DiagramRenderer', function (require) {
"use strict";

var AbstractRenderer = require('web.AbstractRenderer');

/**
 * Diagram Renderer
 *
 * The diagram renderer responsability is to render a diagram view, that is, a
 * set of (labelled) nodes and edges.  To do that, it uses the Raphael.js
 * library.
 */
var DiagramRenderer = AbstractRenderer.extend({
    template: 'DiagramView',
    /**
     * @override
     * @returns {Promise}
     */
    start: function () {
        var $header = this.$el.filter('.o_diagram_header');
        _.each(this.state.labels, function (label) {
            $header.append($('<span>').html(label));
        });
        this.$diagram_container = this.$el.filter('.o_diagram');

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {Promise}
     */
    _render: function () {
        var self = this;
        var nodes  = this.state.nodes;
        var edges  = this.state.edges;
        var id_to_node = {};
        var style = {
            edge_color: "#A0A0A0",
            edge_label_color: "#555",
            edge_label_font_size: 10,
            edge_width: 2,
            edge_spacing: 100,
            edge_loop_radius: 100,

            node_label_color: "#333",
            node_label_font_size: 12,
            node_outline_color: "#333",
            node_outline_width: 1,
            node_selected_color: "#0097BE",
            node_selected_width: 2,
            node_size_x: 110,
            node_size_y: 80,
            connector_active_color: "#FFF",
            connector_radius: 4,

            close_button_radius: 8,
            close_button_color: "#333",
            close_button_x_color: "#FFF",

            gray: "#DCDCDC",
            white: "#FFF",

            viewport_margin: 50
        };

        // remove previous diagram
        this.$diagram_container.empty();

        // for the node and edge's label to be correctly positioned, the diagram
        // must be rendered directly in the DOM, so we render it in a fake
        // element appended in the body, and then move it to this widget's $el
        var $div = $('<div>')
                        .css({position: 'absolute', top: -10000, right: -10000})
                        .appendTo($('body'));
        var r  = new Raphael($div[0], '100%','100%');
        var graph  = new CuteGraph(r, style, this.$diagram_container[0]);
        _.each(nodes, function (node) {
            var n = new CuteNode(
                graph,
                node.x + 50,  // FIXME the +50 should be in the layout algorithm
                node.y + 50,
                CuteGraph.wordwrap(node.name, 14),
                node.shape === 'rectangle' ? 'rect' : 'circle',
                node.color === 'white' ? style.white : style.gray);

            n.id = node.id;
            id_to_node[node.id] = n;
        });
        _.each(edges, function (edge) {
            var e =  new CuteEdge(
                graph,
                CuteGraph.wordwrap(edge.signal, 32),
                id_to_node[edge.s_id],
                id_to_node[edge.d_id] || id_to_node[edge.s_id]);  // WORKAROUND
            e.id = edge.id;
        });

        // move the renderered diagram to the widget's $el
        $div.contents().appendTo(this.$diagram_container);
        $div.remove();

        CuteNode.double_click_callback = function (cutenode) {
            self.trigger_up('edit_node', {id: cutenode.id});
        };
        CuteNode.destruction_callback = function (cutenode) {
            self.trigger_up('remove_node', {id: cutenode.id});
            // return a rejected promise to prevent the library from removing
            // the node directly,as the diagram will be redrawn once the node is
            // deleted
            return Promise.reject();
        };
        CuteEdge.double_click_callback = function (cuteedge) {
            self.trigger_up('edit_edge', {id: cuteedge.id});
        };

        CuteEdge.creation_callback = function (node_start, node_end) {
            return {label: ''};
        };
        CuteEdge.new_edge_callback = function (cuteedge) {
            self.trigger_up('add_edge', {
                source_id: cuteedge.get_start().id,
                dest_id: cuteedge.get_end().id,
            });
        };
        CuteEdge.destruction_callback = function (cuteedge) {
            self.trigger_up('remove_edge', {id: cuteedge.id});
            // return a rejected promise to prevent the library from removing
            // the edge directly, as the diagram will be redrawn once the edge
            // is deleted
            return Promise.reject();
        };
        return this._super.apply(this, arguments);
    },
});

return DiagramRenderer;

});

```

## File: static\src\js\diagram_view.js

```javascript
odoo.define('web_diagram.DiagramView', function (require) {
"use strict";

var BasicView = require('web.BasicView');
var core = require('web.core');
var DiagramModel = require('web_diagram.DiagramModel');
var DiagramRenderer = require('web_diagram.DiagramRenderer');
var DiagramController = require('web_diagram.DiagramController');

var _lt = core._lt;

/**
 * Diagram View
 */
var DiagramView = BasicView.extend({
    display_name: _lt('Diagram'),
    icon: 'fa-code-fork',
    multi_record: false,
    withSearchBar: false,
    searchMenuTypes: [],
    jsLibs: [[
        '/web_diagram/static/lib/js/jquery.mousewheel.js',
        '/web_diagram/static/lib/js/raphael.js',
    ]],
    config: _.extend({}, BasicView.prototype.config, {
        Model: DiagramModel,
        Renderer: DiagramRenderer,
        Controller: DiagramController,
    }),
    viewType: 'diagram',

    /**
     * @override
     * @param {Object} viewInfo
     * @param {Object} params
     */
    init: function (viewInfo, params) {
        this._super.apply(this, arguments);
        var self = this;
        var arch = this.arch;
        // Compute additional data for diagram model
        function toTitleCase(str) {
            return str.replace(/\w\S*/g, function (txt) {
                return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();
            });
        }

        var nodes = arch.children[0];
        var connectors = arch.children[1];
        var node_model = nodes.attrs.object;
        var connector_model = connectors.attrs.object;
        var labels = _.map(_.where(arch.children, {tag: 'label'}), function (label) {
            return label.attrs.string;
        });

        var invisible_nodes = [];
        var visible_nodes = [];
        var node_fields_string = [];
        _.each(nodes.children, function (child) {
            if (child.attrs.invisible === '1')
                invisible_nodes.push(child.attrs.name);
            else {
                var fieldString = self.fields[child.attrs.name].string || toTitleCase(child.attrs.name);
                visible_nodes.push(child.attrs.name);
                node_fields_string.push(fieldString);
            }
        });

        var connector_fields_string = _.map(connectors.children, function (conn) {
            return self.fields[conn.attrs.name].string || toTitleCase(conn.attrs.name);
        });

        this.loadParams = _.extend({}, this.loadParams, {
            currentId: params.currentId,
            nodes: nodes,
            labels: labels,
            invisible_nodes: invisible_nodes,
            visible_nodes: visible_nodes,
            node_fields_string: node_fields_string,
            node_model: node_model,
            connectors: connectors,
            connector_model: connector_model,
            connector_fields_string: connector_fields_string,
        });

        this.controllerParams = _.extend({}, this.controllerParams, {
            domain: params.domain,
            context: params.context,
            ids: params.ids,
            currentId: params.currentId,
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * This override is quite tricky: the graph renderer uses Raphael.js to
     * render itself, so it needs it to be loaded in the window before rendering
     * However, the raphael.js library is built in such a way that if it detects
     * that a module system is present, it will try to use it.  So, in that
     * case, it is not available on window.Raphael.  This means that the diagram
     * view is then broken.
     *
     * As a workaround, we simply remove and restore the define function, if
     * present, while we are loading Raphael.
     *
     * @override
     */
    getController: function () {
        var oldDefine = window.define;
        delete window.define;
        return this._super.apply(this, arguments).then(function (view) {
            window.define = oldDefine;
            return view;
        });
    },
});

return DiagramView;

});

```

## File: static\src\js\graph.js

```javascript

(function(window){

    // this serves as the end of an edge when creating a link
    function EdgeEnd(pos_x,pos_y){
        this.x = pos_x;
        this.y = pos_y;

        this.get_pos = function(){
            return new Vec2(this.x,this.y);
        }
    }

    // A close button, 
    // if entity_type == "node":
    //      GraphNode.destruction_callback(entity) is called where entity is a node.
    //      If it returns true the node and all connected edges are destroyed.
    // if entity_type == "edge":
    //      GraphEdge.destruction_callback(entity) is called where entity is an edge
    //      If it returns true the edge is destroyed
    // pos_x,pos_y is the relative position of the close button to the entity position (entity.get_pos())

    function CloseButton(graph, entity, entity_type, pos_x,pos_y){
        var self = this;
        var visible = false;
        var close_button_radius = graph.style.close_button_radius || 8;
        var close_circle = graph.r.circle(  entity.get_pos().x + pos_x, 
                                            entity.get_pos().y + pos_y, 
                                            close_button_radius           );
        //the outer gray circle
        close_circle.attr({ 'opacity':  0,
                            'fill':     graph.style.close_button_color || "black",
                            'cursor':   'pointer',
                            'stroke':   'none'  });
        close_circle.transform(graph.get_transform());
        graph.set_scrolling(close_circle);
        
        //the 'x' inside the circle
        var close_label = graph.r.text( entity.get_pos().x + pos_x, entity.get_pos().y + pos_y,"x");
        close_label.attr({  'fill':         graph.style.close_button_x_color || "white",
                            'font-size':    close_button_radius,
                            'cursor':       'pointer'   });
        
        close_label.transform(graph.get_transform());
        graph.set_scrolling(close_label);
        
        // the dummy_circle is used to catch events, and avoid hover in/out madness 
        // between the 'x' and the button
        var dummy_circle = graph.r.circle(  entity.get_pos().x + pos_x,
                                            entity.get_pos().y + pos_y,
                                            close_button_radius           );
        dummy_circle.attr({'opacity':1, 'fill': 'transparent', 'stroke':'none', 'cursor':'pointer'});
        dummy_circle.transform(graph.get_transform());
        graph.set_scrolling(dummy_circle);

        this.get_pos = function(){
            return entity.get_pos().add_xy(pos_x,pos_y);
        };

        this.update_pos = function(){
            var pos = self.get_pos(); 
            close_circle.attr({'cx':pos.x, 'cy':pos.y});
            dummy_circle.attr({'cx':pos.x, 'cy':pos.y});
            close_label.attr({'x':pos.x, 'y':pos.y});
        };
        
        function hover_in(){
            if(!visible){ return; }
            close_circle.animate({'r': close_button_radius * 1.5}, 300, 'elastic');
            dummy_circle.animate({'r': close_button_radius * 1.5}, 300, 'elastic');
        }
        function hover_out(){
            if(!visible){ return; }
            close_circle.animate({'r': close_button_radius},400,'linear');
            dummy_circle.animate({'r': close_button_radius},400,'linear');
        }
        dummy_circle.hover(hover_in,hover_out);
        close_circle.hover(hover_in,hover_out);
        close_label.hover(hover_in,hover_out);

        function click_action(){
            if(!visible){ return; }

            close_circle.attr({'r': close_button_radius * 2 });
            dummy_circle.attr({'r': close_button_radius * 2 });
            close_circle.animate({'r': close_button_radius }, 400, 'linear');
            dummy_circle.animate({'r': close_button_radius }, 400, 'linear');

            if(entity_type == "node"){
                Promise.resolve(GraphNode.destruction_callback(entity)).then(function () {
                    entity.remove();
                });
            }else if(entity_type == "edge"){
                Promise.resolve(GraphEdge.destruction_callback(entity)).then(function () {
                    entity.remove();
                });
            }
        }
        dummy_circle.click(click_action);
        close_circle.click(click_action);
        close_label.click(click_action);

        this.show = function(){
            if(!visible){
                close_circle.animate({'opacity':1}, 100, 'linear');
                close_label.animate({'opacity':1}, 100, 'linear');
                visible = true;
            }
        }
        this.hide = function(){
            if(visible){
                close_circle.animate({'opacity':0}, 100, 'linear');
                close_label.animate({'opacity':0}, 100, 'linear');
                visible = false;
            }
        }
        //destroy this object and remove it from the graph
        this.remove = function(){
            if(visible){
                visible = false;
                close_circle.animate({'opacity':0}, 100, 'linear');
                close_label.animate({'opacity':0}, 100, 'linear',self.remove);
            }else{
                close_circle.remove();
                close_label.remove();
                dummy_circle.remove();
            }
        }
    }

    // connectors are start and end point of edge creation drags.
    function Connector(graph,node,pos_x,pos_y){
        var visible = false;
        var conn_circle = graph.r.circle(node.get_pos().x + pos_x, node.get_pos().y + pos_y,4);
        conn_circle.attr({  'opacity':  0, 
                            'fill':     graph.style.node_outline_color,
                            'stroke':   'none' });
        conn_circle.transform(graph.get_transform());
        graph.set_scrolling(conn_circle);

        var self = this;

        this.update_pos = function(){
            conn_circle.attr({'cx':node.get_pos().x + pos_x, 'cy':node.get_pos().y + pos_y});
        };
        this.get_pos = function(){
            return new node.get_pos().add_xy(pos_x,pos_y);
        };
        this.remove = function(){
            conn_circle.remove();
        }
        function hover_in(){
            if(!visible){ return;}
            conn_circle.animate({'r':8},300,'elastic');
            if(graph.creating_edge){
                graph.target_node = node; 
                conn_circle.animate({   'fill':         graph.style.connector_active_color,
                                        'stroke':       graph.style.node_outline_color,
                                        'stroke-width': graph.style.node_selected_width,
                                    },100,'linear');
            }
        }
        function hover_out(){
            if(!visible){ return;}
            conn_circle.animate({   'r':graph.style.connector_radius, 
                                    'fill':graph.style.node_outline_color, 
                                    'stroke':'none'},400,'linear');
            graph.target_node = null;
        }
        conn_circle.hover(hover_in,hover_out);


        var drag_down = function(){
            if(!visible){ return; }
            self.ox = conn_circle.attr("cx");
            self.oy = conn_circle.attr("cy");
            self.edge_start = new EdgeEnd(self.ox,self.oy);
            self.edge_end = new EdgeEnd(self.ox, self.oy);
            self.edge_tmp = new GraphEdge(graph,'',self.edge_start,self.edge_end,true);
            graph.creating_edge = true;
        };
        var drag_move = function(dx,dy){
            if(!visible){ return; }
            self.edge_end.x = self.ox + dx;
            self.edge_end.y = self.oy + dy;
            self.edge_tmp.update();
        };
        var drag_up = function(){
            if(!visible){ return; }
            graph.creating_edge = false;
            self.edge_tmp.remove();
            if(graph.target_node){  
                var edge_prop = GraphEdge.creation_callback(node,graph.target_node);
                if(edge_prop){
                    var new_edge = new GraphEdge(graph,edge_prop.label, node,graph.target_node);
                    GraphEdge.new_edge_callback(new_edge);
                }
            }
        };
        conn_circle.drag(drag_move,drag_down,drag_up);

        function show(){
            if(!visible){
                conn_circle.animate({'opacity':1}, 100, 'linear');
                visible = true;
            }
        }
        function hide(){
            if(visible){
                conn_circle.animate({'opacity':0}, 100, 'linear');
                visible = false;
            }
        }
        this.show = show;
        this.hide = hide;
    }
    
    //Creates a new graph on raphael document r.
    //style is a dictionary containing the style definitions
    //viewport (optional) is the dom element representing the viewport of the graph. It is used
    //to prevent scrolling to scroll the graph outside the viewport.

    function Graph(r,style,viewport){
        var self = this;
        var nodes = [];  // list of all nodes in the graph
        var edges = [];  // list of all edges in the graph
        var graph = {};  // graph[n1.uid][n2.uid] -> list of all edges from n1 to n2
        var links = {};  // links[n.uid] -> list of all edges from or to n
        var uid = 1;     // all nodes and edges have an uid used to order their display when they are curved
        var selected_entity = null; //the selected entity (node or edge) 
        
        self.creating_edge = false; // true if we are dragging a new edge onto a node
        self.target_node = null;    // this holds the target node when creating an edge and hovering a connector
        self.r = r;                 // the raphael instance
        self.style  = style;        // definition of the colors, spacing, fonts, ... used by the elements
        var tr_x = 0, tr_y = 0;         // global translation coordinate

        var background = r.rect(0,0,'100%','100%').attr({'fill':'white', 'stroke':'none', 'opacity':0, 'cursor':'move'});
        
        // return the global transform of the scene
        this.get_transform = function(){
            return "T"+tr_x+","+tr_y
        };

        
        // translate every element of the graph except the background. 
        // elements inserted in the graph after a translate_all() must manually apply transformation 
        // via get_transform() 
        var translate_all = function(dx,dy){
            tr_x += dx;
            tr_y += dy;
            var tstr = self.get_transform();
            
            r.forEach(function(el){
                if(el != background){
                    el.transform(tstr);
                }
            });
        };
        //returns {minx, miny, maxx, maxy}, the translated bounds containing all nodes
        var get_bounds = function(){
            var minx = Number.MAX_VALUE;
            var miny = Number.MAX_VALUE;
            var maxx = Number.MIN_VALUE;
            var maxy = Number.MIN_VALUE;
            
            for(var i = 0; i < nodes.length; i++){
                var pos = nodes[i].get_pos();
                minx = Math.min(minx,pos.x);
                miny = Math.min(miny,pos.y);
                maxx = Math.max(maxx,pos.x);
                maxy = Math.max(maxy,pos.y);
            }

            minx = minx - style.node_size_x / 2 + tr_x;
            miny = miny - style.node_size_y / 2 + tr_y;
            maxx = maxx + style.node_size_x / 2 + tr_x;
            maxy = maxy + style.node_size_y / 2 + tr_y;

            return { minx:minx, miny:miny, maxx:maxx, maxy:maxy };
        
        };
        // returns false if the translation dx,dy of the viewport 
        // hides the graph (with optional margin)
        var translation_respects_viewport = function(dx,dy,margin){
            if(!viewport){
                return true;
            }
            margin = margin || 0;
            var b = get_bounds();
            var width = viewport.offsetWidth; 
            var height = viewport.offsetHeight;
            
            if( ( dy < 0 && b.maxy + dy < margin )   ||
                ( dy > 0 && b.miny + dy > height - margin ) ||
                ( dx < 0 && b.maxx + dx < margin ) ||
                ( dx > 0 && b.minx + dx > width - margin ) ){
                return false;
            }

            return true;
        }
        //Adds a mousewheel event callback to raph_element that scrolls the viewport
        this.set_scrolling = function(raph_element){
            $(raph_element.node).bind('mousewheel',function(event,delta){
                var dy = delta * 20;
                if( translation_respects_viewport(0,dy, style.viewport_margin) ){
                    translate_all(0,dy);
                }
            });
        };

        var px, py;
        // Graph translation when background is dragged
        var bg_drag_down = function(){
            px = py = 0;
        };
        var bg_drag_move = function(x,y){
            var dx = x - px;
            var dy = y - py;
            px = x;
            py = y;
            if( translation_respects_viewport(dx,dy, style.viewport_margin) ){
                translate_all(dx,dy);
            }
        };
        var bg_drag_up   = function(){};
        background.drag( bg_drag_move, bg_drag_down, bg_drag_up);
        
        this.set_scrolling(background);

        //adds a node to the graph and sets its uid.
        this.add_node = function (n){
            nodes.push(n);
            n.uid = uid++;
        };

        //return the list of all nodes in the graph
        this.get_node_list = function(){
            return nodes;
        };

        //adds an edge to the graph and sets its uid
        this.add_edge = function (n1,n2,e){
            edges.push(e);
            e.uid = uid++;
            if(!graph[n1.uid])          graph[n1.uid] = {};
            if(!graph[n1.uid][n2.uid])  graph[n1.uid][n2.uid] = [];
            if(!links[n1.uid]) links[n1.uid] = [];
            if(!links[n2.uid]) links[n2.uid] = [];

            graph[n1.uid][n2.uid].push(e);
            links[n1.uid].push(e);
            if(n1 != n2){
                links[n2.uid].push(e);
            }
        };

        //removes an edge from the graph
        this.remove_edge = function(edge){
            edges = _.without(edges,edge);
            var n1 = edge.get_start();
            var n2 = edge.get_end();
            links[n1.uid] = _.without(links[n1.uid],edge);
            links[n2.uid] = _.without(links[n2.uid],edge);
            graph[n1.uid][n2.uid] = _.without(graph[n1.uid][n2.uid],edge);
            if ( selected_entity == edge ){
                selected_entity = null;
            }
        };
        //removes a node and all connected edges from the graph
        this.remove_node = function(node){
            var linked_edges = self.get_linked_edge_list(node);
            for(var i = 0; i < linked_edges.length; i++){
                linked_edges[i].remove();
            }
            nodes = _.without(nodes,node);

            if ( selected_entity == node ){
                selected_entity = null;
            }
        }


        //return the list of edges from n1 to n2
        this.get_edge_list = function(n1,n2){
            var list = [];
            if(!graph[n1.uid]) return list;
            if(!graph[n1.uid][n2.uid]) return list;
            return graph[n1.uid][n2.uid];
        };
        //returns the list of all edge connected to n
        this.get_linked_edge_list = function(n){
            if(!links[n.uid]) return [];
            return links[n.uid];
        };
        //return a curvature index so that all edges connecting n1,n2 have different curvatures
        this.get_edge_curvature = function(n1,n2,e){
            var el_12 = this.get_edge_list(n1,n2);
            var c12   = el_12.length;
            var el_21 = this.get_edge_list(n2,n1);
            var c21   = el_21.length;
            if(c12 + c21 == 1){ // only one edge 
                return 0;
            }else{ 
                var index = 0;
                for(var i = 0; i < c12; i++){
                    if (el_12[i].uid < e.uid){
                        index++;
                    }
                }
                if(c21 == 0){   // all edges in the same direction
                    return index - (c12-1)/2.0;
                }else{
                    return index + 0.5;
                }
            }
        };
        

        // Returns the angle in degrees of the edge loop. We do not support more than 8 loops on one node
        this.get_loop_angle = function(n,e){
            var loop_list = this.get_edge_list(n,n);

            var slots = []; // the 8 angles where we can put the loops 
            for(var angle = 0; angle < 360; angle += 45){
                slots.push(Vec2.new_polar_deg(1,angle));
            }
            
            //we assign to each slot a score. The higher the score, the closer it is to other edges.
            var links = this.get_linked_edge_list(n);
            for(var i = 0; i < links.length; i++){
                var edge = links[i];
                if(!edge.is_loop || edge.is_loop()){
                    continue;
                }
                var end = edge.get_end();
                if (end == n){
                    end = edge.get_start();
                }
                var dir = end.get_pos().sub(n.get_pos()).normalize();
                for(var s = 0; s < slots.length; s++){
                    var score = slots[s].dot(dir);
                    if(score < 0){
                        score = -0.2*Math.pow(score,2);
                    }else{
                        score = Math.pow(score,2);
                    }
                    if(!slots[s].score){
                        slots[s].score = score;
                    }else{
                        slots[s].score += score;
                    }
                }
            }
            //we want the loops with lower uid to get the slots with the lower score
            slots.sort(function(a,b){ return a.score < b.score ? -1: 1; });
            
            var index = 0;
            for(var i = 0; i < links.length; i++){
                var edge = links[i];
                if(!edge.is_loop || !edge.is_loop()){
                    continue;
                }
                if(edge.uid < e.uid){
                    index++;
                }
            }
            index %= slots.length;
            
            return slots[index].angle_deg();
        }

        //selects a node or an edge and deselects everything else
        this.select = function(entity){
            if(selected_entity){
                if(selected_entity == entity){
                    return;
                }else{
                    if(selected_entity.set_not_selected){
                        selected_entity.set_not_selected();
                    }
                    selected_entity = null;
                }
            }
            selected_entity = entity;
            if(entity && entity.set_selected){
                entity.set_selected();
            }
        };
    }

    // creates a new Graph Node on Raphael document r, centered on [pos_x,pos_y], with label 'label', 
    // and of type 'circle' or 'rect', and of color 'color'
    function GraphNode(graph,pos_x, pos_y,label,type,color){
        var self = this;
        var r  = graph.r;
        var sy = graph.style.node_size_y;
        var sx = graph.style.node_size_x;
        var node_fig = null;
        var selected = false;
        this.connectors = [];
        this.close_button = null;
        this.uid = 0;
        
        graph.add_node(this);

        if(type == 'circle'){
            node_fig = r.ellipse(pos_x,pos_y,sx/2,sy/2);
        }else{
            node_fig = r.rect(pos_x-sx/2,pos_y-sy/2,sx,sy);
        }
        node_fig.attr({ 'fill':         color, 
                        'stroke':       graph.style.node_outline_color,
                        'stroke-width': graph.style.node_outline_width,
                        'cursor':'pointer'  });
        node_fig.transform(graph.get_transform());
        graph.set_scrolling(node_fig);

        var node_label = r.text(pos_x,pos_y,label);
        node_label.attr({   'fill':         graph.style.node_label_color,
                            'font-size':    graph.style.node_label_font_size,
                            'cursor':       'pointer'   });
        node_label.transform(graph.get_transform());
        graph.set_scrolling(node_label);

        // redraws all edges linked to this node 
        var update_linked_edges = function(){
            var edges = graph.get_linked_edge_list(self);
            for(var i = 0; i < edges.length; i++){
                edges[i].update();
            }
        };

        // sets the center position of the node
        var set_pos = function(pos){
            if(type == 'circle'){
                node_fig.attr({'cx':pos.x,'cy':pos.y});
            }else{
                node_fig.attr({'x':pos.x-sx/2,'y':pos.y-sy/2});
            }
            node_label.attr({'x':pos.x,'y':pos.y});
            for(var i = 0; i < self.connectors.length; i++){
                self.connectors[i].update_pos();
            }
            if(self.close_button){
                self.close_button.update_pos();
            }
            update_linked_edges();
        };
        // returns the figure used to draw the node
        var get_fig = function(){
            return node_fig;
        };
        // returns the center coordinates
        var get_pos = function(){
            if(type == 'circle'){ 
                return new Vec2(node_fig.attr('cx'), node_fig.attr('cy')); 
            }else{ 
                return new Vec2(node_fig.attr('x') + sx/2, node_fig.attr('y') + sy/2); 
            }
        };
        // return the label string
        var get_label = function(){
            return node_label.attr("text");
        };
        // sets the label string
        var set_label = function(text){
            node_label.attr({'text':text});
        };
        var get_bound = function(){
            if(type == 'circle'){
                return new BEllipse(get_pos().x,get_pos().y,sx/2,sy/2);
            }else{
                return BRect.new_centered(get_pos().x,get_pos().y,sx,sy);
            }
        };
        // selects this node and deselects all other nodes
        var set_selected = function(){
            if(!selected){
                selected = true;
                node_fig.attr({ 'stroke':       graph.style.node_selected_color, 
                                'stroke-width': graph.style.node_selected_width });
                if(!self.close_button){
                    self.close_button = new CloseButton(graph,self, "node" ,sx/2 , - sy/2);
                    self.close_button.show();
                }
                for(var i = 0; i < self.connectors.length; i++){
                    self.connectors[i].show();
                }
            }
        };
        // deselect this node
        var set_not_selected = function(){
            if(selected){
                node_fig.animate({  'stroke':       graph.style.node_outline_color,
                                    'stroke-width': graph.style.node_outline_width },
                                    100,'linear');
                if(self.close_button){
                    self.close_button.remove();
                    self.close_button = null;
                }
                selected = false;
            }
            for(var i = 0; i < self.connectors.length; i++){
                self.connectors[i].hide();
            }
        };
        var remove = function(){
            if(self.close_button){
                self.close_button.remove();
            }
            for(var i = 0; i < self.connectors.length; i++){
                self.connectors[i].remove();
            }
            graph.remove_node(self);
            node_fig.remove();
            node_label.remove();
        }


        this.set_pos = set_pos;
        this.get_pos = get_pos;
        this.set_label = set_label;
        this.get_label = get_label;
        this.get_bound = get_bound;
        this.get_fig   = get_fig;
        this.set_selected = set_selected;
        this.set_not_selected = set_not_selected;
        this.update_linked_edges = update_linked_edges;
        this.remove = remove;

       
        //select the node and play an animation when clicked
        var click_action = function(){
            if(type == 'circle'){
                node_fig.attr({'rx':sx/2 + 3, 'ry':sy/2+ 3});
                node_fig.animate({'rx':sx/2, 'ry':sy/2},500,'elastic');
            }else{
                var cx = get_pos().x;
                var cy = get_pos().y;
                node_fig.attr({'x':cx - (sx/2) - 3, 'y':cy - (sy/2) - 3, 'ẃidth':sx+6, 'height':sy+6});
                node_fig.animate({'x':cx - sx/2, 'y':cy - sy/2, 'ẃidth':sx, 'height':sy},500,'elastic');
            }
            graph.select(self);
        };
        node_fig.click(click_action);
        node_label.click(click_action);

        //move the node when dragged
        var drag_down = function(){
            this.opos = get_pos();
        };
        var drag_move = function(dx,dy){
            // we disable labels when moving for performance reasons, 
            // updating the label position is quite expensive
            // we put this here because drag_down is also called on simple clicks ... and this causes unwanted flicker
            var edges = graph.get_linked_edge_list(self);
            for(var i = 0; i < edges.length; i++){
                edges[i].label_disable();
            }
            if(self.close_button){
                self.close_button.hide();
            }
            set_pos(this.opos.add_xy(dx,dy));
        };
        var drag_up = function(){
            //we re-enable the 
            var edges = graph.get_linked_edge_list(self);
            for(var i = 0; i < edges.length; i++){
                edges[i].label_enable();
            }
            if(self.close_button){
                self.close_button.show();
            }
        };
        node_fig.drag(drag_move,drag_down,drag_up);
        node_label.drag(drag_move,drag_down,drag_up);

        //allow the user to create edges by dragging onto the node
        function hover_in(){
            if(graph.creating_edge){
                graph.target_node = self; 
            }
        }
        function hover_out(){
            graph.target_node = null;
        }
        node_fig.hover(hover_in,hover_out);
        node_label.hover(hover_in,hover_out);

        function double_click(){
            GraphNode.double_click_callback(self);
        }
        node_fig.dblclick(double_click);
        node_label.dblclick(double_click);

        this.connectors.push(new Connector(graph,this,-sx/2,0));
        this.connectors.push(new Connector(graph,this,sx/2,0));
        this.connectors.push(new Connector(graph,this,0,-sy/2));
        this.connectors.push(new Connector(graph,this,0,sy/2));

        this.close_button = new CloseButton(graph,this,"node",sx/2 , - sy/2 );
    }

    GraphNode.double_click_callback = function(node){
        console.log("double click from node:",node);
    };

    // this is the default node destruction callback. It is called before the node is removed from the graph
    // and before the connected edges are destroyed 
    GraphNode.destruction_callback = function(node){ return true; };

    // creates a new edge with label 'label' from start to end. start and end must implement get_pos_*, 
    // if tmp is true, the edge is not added to the graph, used for drag edges. 
    // replace tmp == false by graph == null 
    function GraphEdge(graph,label,start,end,tmp){
        var self = this;
        var r = graph.r;
        var curvature = 0;  // 0 = straight, != 0 curved
        var s,e;            // positions of the start and end point of the line between start and end
        var mc;             // position of the middle of the curve (bezier control point) 
        var mc1,mc2;        // control points of the cubic bezier for the loop edges
        var elfs =  graph.style.edge_label_font_size || 10 ; 
        var label_enabled = true;
        this.uid = 0;       // unique id used to order the curved edges
        var edge_path = ""; // svg definition of the edge vector path
        var selected = false;

        if(!tmp){
            graph.add_edge(start,end,this);
        }
        
        //Return the position of the label
        function get_label_pos(path){
            var cpos = path.getTotalLength() * 0.5;
            var cindex = Math.abs(Math.floor(curvature));
            var mod = ((cindex % 3)) * (elfs * 3.1) - (elfs * 0.5);
            var verticality = Math.abs(end.get_pos().sub(start.get_pos()).normalize().dot_xy(0,1));
            verticality = Math.max(verticality-0.5,0)*2;

            var lpos = path.getPointAtLength(cpos + mod * verticality);
            return new Vec2(lpos.x,lpos.y - elfs *(1-verticality));
        }
        
        //used by close_button
        this.get_pos = function(){
            if(!edge){
                return start.get_pos().lerp(end.get_pos(),0.5);
            }
            return get_label_pos(edge);
            /*  
            var bbox = edge_label.getBBox(); Does not work... :(
            return new Vec2(bbox.x + bbox.width, bbox.y);*/
        }

        //Straight line from s to e
        function make_line(){
            return "M" + s.x + "," + s.y + "L" + e.x + "," + e.y ;
        }
        //Curved line from s to e by mc
        function make_curve(){
            return "M" + s.x + "," + s.y + "Q" + mc.x + "," + mc.y + " " + e.x + "," + e.y;
        }
        //Curved line from s to e by mc1 mc2
        function make_loop(){
            return "M" + s.x + " " + s.y + 
                   "C" + mc1.x + " " + mc1.y + " " + mc2.x + " " + mc2.y + " " + e.x + " " + e.y;
        }
            
        //computes new start and end line coordinates
        function update_curve(){
            if(start != end){
                if(!tmp){
                    curvature = graph.get_edge_curvature(start,end,self);
                }else{
                    curvature = 0;
                }
                s = start.get_pos();
                e = end.get_pos();
                
                mc = s.lerp(e,0.5); //middle of the line s->e
                var se = e.sub(s);
                se = se.normalize();
                se = se.rotate_deg(-90);
                se = se.scale(curvature * graph.style.edge_spacing);
                mc = mc.add(se);

                if(start.get_bound){
                    var col = start.get_bound().collide_segment(s,mc);
                    if(col.length > 0){
                        s = col[0];
                    }
                }
                if(end.get_bound){
                    var col = end.get_bound().collide_segment(mc,e);
                    if(col.length > 0){
                        e = col[0];
                    }
                }
                
                if(curvature != 0){
                    edge_path = make_curve();
                }else{
                    edge_path = make_line();
                }
            }else{ // start == end
                var rad = graph.style.edge_loop_radius || 100;
                s = start.get_pos();
                e = end.get_pos();

                var r = Vec2.new_polar_deg(rad,graph.get_loop_angle(start,self));
                mc = s.add(r);
                var p = r.rotate_deg(90);
                mc1 = mc.add(p.set_len(rad*0.5));
                mc2 = mc.add(p.set_len(-rad*0.5));
                
                if(start.get_bound){
                    var col = start.get_bound().collide_segment(s,mc1);
                    if(col.length > 0){
                        s = col[0];
                    }
                    var col = start.get_bound().collide_segment(e,mc2);
                    if(col.length > 0){
                        e = col[0];
                    }
                }
                edge_path = make_loop();
            }
        }
        
        update_curve();
        var edge = r.path(edge_path).attr({ 'stroke':       graph.style.edge_color, 
                                            'stroke-width': graph.style.edge_width, 
                                            'arrow-end':    'block-wide-long', 
                                            'cursor':'pointer'  }).insertBefore(graph.get_node_list()[0].get_fig());       
        var labelpos = get_label_pos(edge);
        var edge_label = r.text(labelpos.x, labelpos.y - elfs, label).attr({
            'fill':         graph.style.edge_label_color, 
            'cursor':       'pointer', 
            'font-size':    elfs    });

        edge.transform(graph.get_transform());
        graph.set_scrolling(edge);

        edge_label.transform(graph.get_transform());
        graph.set_scrolling(edge_label);
        

        //since we create an edge we need to recompute the edges that have the same start and end positions as this one
        if(!tmp){
            var edges_start = graph.get_linked_edge_list(start);
            var edges_end   = graph.get_linked_edge_list(end);
            var edges = edges_start.length < edges_end.length ? edges_start : edges_end;
            for(var i = 0; i < edges.length; i ++){
                if(edges[i] != self){
                    edges[i].update();
                }
            }
        }
        function label_enable(){
            if(!label_enabled){
                label_enabled = true;
                edge_label.animate({'opacity':1},100,'linear');
                if(self.close_button){
                    self.close_button.show();
                }
                self.update();
            }
        }
        function label_disable(){
            if(label_enabled){
                label_enabled = false;
                edge_label.animate({'opacity':0},100,'linear');
                if(self.close_button){
                    self.close_button.hide();
                }
            }
        }
        //update the positions 
        function update(){
            update_curve();
            edge.attr({'path':edge_path});
            if(label_enabled){
                var labelpos = get_label_pos(edge);
                edge_label.attr({'x':labelpos.x, 'y':labelpos.y - 14});
            }
        }
        // removes the edge from the scene, disconnects it from linked        
        // nodes, destroy its drawable elements.
        function remove(){
            edge.remove();
            edge_label.remove();
            if(!tmp){
                graph.remove_edge(self);
            }
            if(start.update_linked_edges){
                start.update_linked_edges();
            }
            if(start != end && end.update_linked_edges){
                end.update_linked_edges();
            }
            if(self.close_button){
                self.close_button.remove();
            }
        }

        this.set_selected = function(){
            if(!selected){
                selected = true;
                edge.attr({ 'stroke': graph.style.node_selected_color, 
                            'stroke-width': graph.style.node_selected_width });
                edge_label.attr({ 'fill': graph.style.node_selected_color });
                if(!self.close_button){
                    self.close_button = new CloseButton(graph,self,"edge",0,30);
                    self.close_button.show();
                }
            }
        };

        this.set_not_selected = function(){
            if(selected){
                selected = false;
                edge.animate({  'stroke':       graph.style.edge_color,
                                'stroke-width': graph.style.edge_width }, 100,'linear');
                edge_label.animate({ 'fill':    graph.style.edge_label_color}, 100, 'linear');
                if(self.close_button){
                    self.close_button.remove();
                    self.close_button = null;
                }
            }
        };
        function click_action(){
            graph.select(self);
        }
        edge.click(click_action);
        edge_label.click(click_action);

        function double_click_action(){
            GraphEdge.double_click_callback(self);
        }

        edge.dblclick(double_click_action);
        edge_label.dblclick(double_click_action);


        this.label_enable  = label_enable;
        this.label_disable = label_disable;
        this.update = update;
        this.remove = remove;
        this.is_loop = function(){ return start == end; };
        this.get_start = function(){ return start; };
        this.get_end   = function(){ return end; };
    }

    GraphEdge.double_click_callback = function(edge){
        console.log("double click from edge:",edge);
    };

    // this is the default edge creation callback. It is called before an edge is created
    // It returns an object containing the properties of the edge.
    // If it returns null, the edge is not created.
    GraphEdge.creation_callback = function(start,end){
        var edge_prop = {};
        edge_prop.label = 'new edge!';
        return edge_prop;
    };
    // This is is called after a new edge is created, with the new edge
    // as parameter
    GraphEdge.new_edge_callback = function(new_edge){};

    // this is the default edge destruction callback. It is called before 
    // an edge is removed from the graph.
    GraphEdge.destruction_callback = function(edge){ return true; };

    

    // returns a new string with the same content as str, but with lines of maximum 'width' characters.
    // lines are broken on words, or into words if a word is longer than 'width'
    function wordwrap( str, width) {
        // http://james.padolsey.com/javascript/wordwrap-for-javascript/
        width = width || 32;
        var cut = true;
        var brk = '\n';
        if (!str) { return str; }
        var regex = '.{1,' +width+ '}(\\s|$)' + (cut ? '|.{' +width+ '}|.+$' : '|\\S+?(\\s|$)');
        return str.match(new RegExp(regex, 'g') ).join( brk );
    }

    window.CuteGraph   = Graph;
    window.CuteNode    = GraphNode;
    window.CuteEdge    = GraphEdge;

    window.CuteGraph.wordwrap = wordwrap;


})(window);


```

## File: static\src\js\vec2.js

```javascript

(function(window){
    
    // A Javascript 2D vector library
    // conventions :
    // method that returns a float value do not modify the vector
    // method that implement operators return a new vector with the modifications without
    // modifying the calling vector or the parameters.
    // 
    //      v3 = v1.add(v2); // v3 is set to v1 + v2, v1, v2 are not modified
    //
    // methods that take a single vector as a parameter are usually also available with
    // q '_xy' suffix. Those method takes two floats representing the x,y coordinates of
    // the vector parameter and allow you to avoid to needlessly create a vector object : 
    //
    //      v2 = v1.add(new Vec2(3,4));
    //      v2 = v1.add_xy(3,4);             //equivalent to previous line
    //
    // angles are in radians by default but method that takes angle as parameters 
    // or return angle values usually have a variant with a '_deg' suffix that works in degrees
    //
     
    // The 2D vector object 
    function Vec2(x,y){
        this.x = x;
        this.y = y;
    }

    window.Vec2 = Vec2;
    
    // Multiply a number expressed in radiant by rad2deg to convert it in degrees
    var rad2deg = 57.29577951308232;
    // Multiply a number expressed in degrees by deg2rad to convert it to radiant
    var deg2rad = 0.017453292519943295;
    // The numerical precision used to compare vector equality
    var epsilon   = 0.0000001;

    // This static method creates a new vector from polar coordinates with the angle expressed
    // in degrees
    Vec2.new_polar_deg = function(len,angle){
        var v = new Vec2(len,0);
        return v.rotate_deg(angle);
    };
    // This static method creates a new vector from polar coordinates with the angle expressed in
    // radians
    Vec2.new_polar = function(len,angle){
        var v = new Vec2(len,0);
        v.rotate(angle);
        return v;
    };
    // returns the length or modulus or magnitude of the vector
    Vec2.prototype.len = function(){
        return Math.sqrt(this.x*this.x + this.y*this.y);
    };
    // returns the squared length of the vector, this method is much faster than len()
    Vec2.prototype.len_sq = function(){
        return this.x*this.x + this.y*this.y;
    };
    // return the distance between this vector and the vector v
    Vec2.prototype.dist = function(v){
        var dx = this.x - v.x;
        var dy = this.y - v.y;
        return Math.sqrt(dx*dx + dy*dy);
    };
    // return the distance between this vector and the vector of coordinates (x,y)
    Vec2.prototype.dist_xy = function(x,y){
        var dx = this.x - x;
        var dy = this.y - y;
        return Math.sqrt(dx*dx + dy*dy);
    };
    // return the squared distance between this vector and the vector and the vector v
    Vec2.prototype.dist_sq = function(v){
        var dx = this.x - v.x;
        var dy = this.y - v.y;
        return dx*dx + dy*dy;
    };
    // return the squared distance between this vector and the vector of coordinates (x,y)
    Vec2.prototype.dist_sq_xy = function(x,y){
        var dx = this.x - x;
        var dy = this.y - y;
        return dx*dx + dy*dy;
    };
    // return the dot product between this vector and the vector v
    Vec2.prototype.dot = function(v){
        return this.x*v.x + this.y*v.y;
    };
    // return the dot product between this vector and the vector of coordinate (x,y)
    Vec2.prototype.dot_xy = function(x,y){
        return this.x*x + this.y*y;
    };
    // return a new vector with the same coordinates as this 
    Vec2.prototype.clone = function(){
        return new Vec2(this.x,this.y);
    };
    // return the sum of this and vector v as a new vector
    Vec2.prototype.add = function(v){
        return new Vec2(this.x+v.x,this.y+v.y);
    };
    // return the sum of this and vector (x,y) as a new vector
    Vec2.prototype.add_xy = function(x,y){
        return new Vec2(this.x+x,this.y+y);
    };
    // returns (this - v) as a new vector where v is a vector and - is the vector subtraction
    Vec2.prototype.sub = function(v){
        return new Vec2(this.x-v.x,this.y-v.y);
    };
    // returns (this - (x,y)) as a new vector where - is vector subtraction
    Vec2.prototype.sub_xy = function(x,y){
        return new Vec2(this.x-x,this.y-y);
    };
    // return (this * v) as a new vector where v is a vector and * is the by component product
    Vec2.prototype.mult = function(v){
        return new Vec2(this.x*v.x,this.y*v.y);
    };
    // return (this * (x,y)) as a new vector where * is the by component product
    Vec2.prototype.mult_xy = function(x,y){
        return new Vec2(this.x*x,this.y*y);
    };
    // return this scaled by float f as a new fector
    Vec2.prototype.scale = function(f){
        return new Vec2(this.x*f, this.y*f);
    };
    // return the negation of this vector
    Vec2.prototype.neg = function(f){
        return new Vec2(-this.x,-this.y);
    };
    // return this vector normalized as a new vector
    Vec2.prototype.normalize = function(){
        var len = this.len();
        if(len == 0){
            return new Vec2(0,1);
        }else if(len != 1){
            return this.scale(1.0/len);
        }
        return new Vec2(this.x,this.y);
    };
    // return a new vector with the same direction as this vector of length float l. (negative values of l will invert direction)
    Vec2.prototype.set_len = function(l){
        return this.normalize().scale(l);
    };
    // return the projection of this onto the vector v as a new vector
    Vec2.prototype.project = function(v){
        return v.set_len(this.dot(v));
    };
    // return a string representation of this vector
    Vec2.prototype.toString = function(){
        var str = "";
        str += "[";
        str += this.x;
        str += ",";
        str += this.y;
        str += "]";
        return str;
    };
    //return this vector counterclockwise rotated by rad radians as a new vector
    Vec2.prototype.rotate = function(rad){
        var c = Math.cos(rad);
        var s = Math.sin(rad);
        var px = this.x * c - this.y *s;
        var py = this.x * s + this.y *c;
        return new Vec2(px,py);
    };
    //return this vector counterclockwise rotated by deg degrees as a new vector
    Vec2.prototype.rotate_deg = function(deg){
        return this.rotate(deg * deg2rad);
    };
    //linearly interpolate this vector towards the vector v by float factor alpha.
    // alpha == 0 : does nothing
    // alpha == 1 : sets this to v
    Vec2.prototype.lerp = function(v,alpha){
        var inv_alpha = 1 - alpha;
        return new Vec2(    this.x * inv_alpha + v.x * alpha,
                            this.y * inv_alpha + v.y * alpha    );
    };
    // returns the angle between this vector and the vector (1,0) in radians
    Vec2.prototype.angle = function(){
        return Math.atan2(this.y,this.x);
    };
    // returns the angle between this vector and the vector (1,0) in degrees
    Vec2.prototype.angle_deg = function(){
        return Math.atan2(this.y,this.x) * rad2deg;
    };
    // returns true if this vector is equal to the vector v, with a tolerance defined by the epsilon module constant
    Vec2.prototype.equals = function(v){
        if(Math.abs(this.x-v.x) > epsilon){
            return false;
        }else if(Math.abs(this.y-v.y) > epsilon){
            return false;
        }
        return true;
    };
    // returns true if this vector is equal to the vector (x,y) with a tolerance defined by the epsilon module constant
    Vec2.prototype.equals_xy = function(x,y){
        if(Math.abs(this.x-x) > epsilon){
            return false;
        }else if(Math.abs(this.y-y) > epsilon){
            return false;
        }
        return true;
    };
})(window);

(function(window){
    // A Bounding Shapes Library


    // A Bounding Ellipse
    // cx,cy : center of the ellipse
    // rx,ry : radius of the ellipse
    function BEllipse(cx,cy,rx,ry){
        this.type = 'ellipse';
        this.x = cx-rx;     // minimum x coordinate contained in the ellipse     
        this.y = cy-ry;     // minimum y coordinate contained in the ellipse
        this.sx = 2*rx;     // width of the ellipse on the x axis
        this.sy = 2*ry;     // width of the ellipse on the y axis
        this.hx = rx;       // half of the ellipse width on the x axis
        this.hy = ry;       // half of the ellipse width on the y axis
        this.cx = cx;       // x coordinate of the ellipse center
        this.cy = cy;       // y coordinate of the ellipse center
        this.mx = cx + rx;  // maximum x coordinate contained in the ellipse
        this.my = cy + ry;  // maximum x coordinate contained in the ellipse
    }
    window.BEllipse = BEllipse;

    // returns an unordered list of vector defining the positions of the intersections between the ellipse's
    // boundary and a line segment defined by the start and end vectors a,b
    BEllipse.prototype.collide_segment = function(a,b){
        // http://paulbourke.net/geometry/sphereline/
        var collisions = [];

        if(a.equals(b)){  //we do not compute the intersection in this case. TODO ?     
            return collisions;
        }

        // make all computations in a space where the ellipse is a circle 
        // centered on zero
        var c = new Vec2(this.cx,this.cy);
        a = a.sub(c).mult_xy(1/this.hx,1/this.hy);
        b = b.sub(c).mult_xy(1/this.hx,1/this.hy);


        if(a.len_sq() < 1 && b.len_sq() < 1){   //both points inside the ellipse
            return collisions;
        }

        // compute the roots of the intersection
        var ab = b.sub(a);
        var A = (ab.x*ab.x + ab.y*ab.y);
        var B = 2*( ab.x*a.x + ab.y*a.y);
        var C = a.x*a.x + a.y*a.y - 1;
        var u  = B * B - 4*A*C;
        
        if(u < 0){
            return collisions;
        }

        u = Math.sqrt(u);
        var u1 = (-B + u) / (2*A);
        var u2 = (-B - u) / (2*A);

        if(u1 >= 0 && u1 <= 1){
            var pos = a.add(ab.scale(u1));
            collisions.push(pos);
        }
        if(u1 != u2 && u2 >= 0 && u2 <= 1){
            var pos = a.add(ab.scale(u2));
            collisions.push(pos);
        }
        for(var i = 0; i < collisions.length; i++){
            collisions[i] = collisions[i].mult_xy(this.hx,this.hy);
            collisions[i] = collisions[i].add_xy(this.cx,this.cy);
        }
        return collisions;
    };
    
    // A bounding rectangle
    // x,y the minimum coordinate contained in the rectangle
    // sx,sy the size of the rectangle along the x,y axis
    function BRect(x,y,sx,sy){
        this.type = 'rect';
        this.x = x;              // minimum x coordinate contained in the rectangle  
        this.y = y;              // minimum y coordinate contained in the rectangle
        this.sx = sx;            // width of the rectangle on the x axis
        this.sy = sy;            // width of the rectangle on the y axis
        this.hx = sx/2;          // half of the rectangle width on the x axis
        this.hy = sy/2;          // half of the rectangle width on the y axis
        this.cx = x + this.hx;   // x coordinate of the rectangle center
        this.cy = y + this.hy;   // y coordinate of the rectangle center
        this.mx = x + sx;        // maximum x coordinate contained in the rectangle
        this.my = y + sy;        // maximum x coordinate contained in the rectangle
    }

    window.BRect = BRect;
    // Static method creating a new bounding rectangle of size (sx,sy) centered on (cx,cy)
    BRect.new_centered = function(cx,cy,sx,sy){
        return new BRect(cx-sx/2,cy-sy/2,sx,sy);
    };
    //intersect line a,b with line c,d, returns null if no intersection
    function line_intersect(a,b,c,d){
        // http://paulbourke.net/geometry/lineline2d/
        var f = ((d.y - c.y)*(b.x - a.x) - (d.x - c.x)*(b.y - a.y)); 
        if(f == 0){
            return null;
        }
        f = 1 / f;
        var fab = ((d.x - c.x)*(a.y - c.y) - (d.y - c.y)*(a.x - c.x)) * f ;
        if(fab < 0 || fab > 1){
            return null;
        }
        var fcd = ((b.x - a.x)*(a.y - c.y) - (b.y - a.y)*(a.x - c.x)) * f ;
        if(fcd < 0 || fcd > 1){
            return null;
        }
        return new Vec2(a.x + fab * (b.x-a.x), a.y + fab * (b.y - a.y) );
    }

    // returns an unordered list of vector defining the positions of the intersections between the ellipse's
    // boundary and a line segment defined by the start and end vectors a,b

    BRect.prototype.collide_segment = function(a,b){
        var collisions = [];
        var corners = [ new Vec2(this.x,this.y), new Vec2(this.x,this.my), 
                        new Vec2(this.mx,this.my), new Vec2(this.mx,this.y) ];
        var pos = line_intersect(a,b,corners[0],corners[1]);
        if(pos) collisions.push(pos);
        pos = line_intersect(a,b,corners[1],corners[2]);
        if(pos) collisions.push(pos);
        pos = line_intersect(a,b,corners[2],corners[3]);
        if(pos) collisions.push(pos);
        pos = line_intersect(a,b,corners[3],corners[0]);
        if(pos) collisions.push(pos);
        return collisions;
    };

    // returns true if the rectangle contains the position defined by the vector 'vec'
    BRect.prototype.contains_vec = function(vec){
        return ( vec.x >= this.x && vec.x <= this.mx && 
                 vec.y >= this.y && vec.y <= this.my  );
    };
    // returns true if the rectangle contains the position (x,y) 
    BRect.prototype.contains_xy = function(x,y){
        return ( x >= this.x && x <= this.mx && 
                 y >= this.y && y <= this.my  );
    };
    // returns true if the ellipse contains the position defined by the vector 'vec'
    BEllipse.prototype.contains_vec = function(v){
        v = v.mult_xy(this.hx,this.hy);
        return v.len_sq() <= 1;
    };
    // returns true if the ellipse contains the position (x,y) 
    BEllipse.prototype.contains_xy = function(x,y){
        return this.contains(new Vec2(x,y));
    };


})(window);
        


```

## File: static\src\js\view_registry.js

```javascript
odoo.define('web_diagram.view_registry', function (require) {
"use strict";

var view_registry = require('web.view_registry');

var DiagramView = require('web_diagram.DiagramView');

view_registry.add('diagram', DiagramView);

});

```

## File: static\src\xml\base_diagram.xml

```xml
<template>

<t t-name="DiagramView.buttons">
	<div t-if="widget.is_action_enabled('create')">
        <button type="button" class="btn btn-primary o_diagram_new_button">
            New Node
        </button>
    </div>
</t>

<t t-name="DiagramView">
	<div class="o_diagram_header"/>
    <div class="o_diagram"/>
</t>

</template>

```

## File: views\web_diagram_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="web_diagram assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/web_diagram/static/src/scss/diagram_view.scss"/>
                <script type="text/javascript" src="/web_diagram/static/src/js/vec2.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/graph.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/diagram_model.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/diagram_controller.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/diagram_renderer.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/diagram_view.js"></script>
                <script type="text/javascript" src="/web_diagram/static/src/js/view_registry.js"></script>
            </xpath>
        </template>

        <template id="qunit_suite" name="web_diagram tests" inherit_id="web.qunit_suite">
            <xpath expr="//script[contains(@src, '/web/static/tests/views/kanban_tests.js')]" position="after">
                <script type="text/javascript" src="/web_diagram/static/tests/diagram_tests.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

