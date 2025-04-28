# Odoo Module: web_tour

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Tours',
    'category': 'Hidden',
    'description': """
Odoo Web tours.
========================

""",
    'version': '0.1',
    'depends': ['web'],
    'data': [
        'security/ir.model.access.csv',
        'security/ir.rule.csv',
        'views/tour_templates.xml',
        'views/tour_views.xml'
    ],
    'demo': [
        'data/web_tour_demo.xml',
    ],
    'qweb': [
        "static/src/xml/debug_manager.xml",
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\web_tour_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Disable tours if Odoo installed with demo data. -->
    <template id="assets_common_disable_tour" name="disable tour" inherit_id="web.assets_common">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/web_tour/static/src/js/tour_disable.js"></script>
        </xpath>
    </template>

</odoo>

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(Http, self).session_info()
        if result['is_admin']:
            result['web_tours'] = request.env['web_tour.tour'].get_consumed_tours()
        return result

```

## File: models\tour.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Tour(models.Model):

    _name = "web_tour.tour"
    _description = "Tours"
    _log_access = False

    name = fields.Char(string="Tour name", required=True)
    user_id = fields.Many2one('res.users', string='Consumed by')

    @api.model
    def consume(self, tour_names):
        """ Sets given tours as consumed, meaning that
            these tours won't be active anymore for that user """
        for name in tour_names:
            self.create({'name': name, 'user_id': self.env.uid})

    @api.model
    def get_consumed_tours(self):
        """ Returns the list of consumed tours for the current user """
        return [t.name for t in self.search([('user_id', '=', self.env.uid)])]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import tour

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_web_tour_tour_admin,access_web_tour_tour_admin,model_web_tour_tour,base.group_system,1,0,1,0
access_web_tour_tour,access_web_tour_tour,model_web_tour_tour,,1,0,0,0

```

## File: security\ir.rule.csv

```csv
"id","model_id/id","name","active","perm_create","perm_unlink","perm_read","perm_write","domain_force"
"own_tours","model_web_tour_tour","own tours","True","True","True","True","True","[('user_id','=', user.id)]"

```

## File: static\src\js\debug_manager.js

```javascript
odoo.define('web_tour.DebugManager.Backend', function (require) {
"use strict";

var core = require("web.core");
var DebugManager = require('web.DebugManager.Backend');
var Dialog = require("web.Dialog");

var tour = require('web_tour.tour');

function get_active_tours () {
    return _.difference(_.keys(tour.tours), tour.consumed_tours);
}

DebugManager.include({
    start: function () {
        this.consume_tours_enabled = get_active_tours().length > 0;
        return this._super.apply(this, arguments);
    },
    consume_tours: function () {
        var active_tours = get_active_tours();
        if (active_tours.length > 0) { // tours might have been consumed meanwhile
            this._rpc({
                    model: 'web_tour.tour',
                    method: 'consume',
                    args: [active_tours],
                })
                .then(function () {
                    window.location.reload();
                });
        }
    },
    start_tour: function () {
        var dialog = new Dialog(this, {
            title: 'Tours',
            $content: core.qweb.render('WebClient.DebugManager.ToursDialog', {
                tours: tour.tours
            }),
        });
        dialog.opened().then(function () {
            dialog.$('.o_start_tour').on('click', function (e) {
                e.preventDefault();
                tour.run($(e.target).data('name'));
            });
        });
        dialog.open();
    },
});

});

```

## File: static\src\js\running_tour_action_helper.js

```javascript

odoo.define('web_tour.RunningTourActionHelper', function (require) {
"use strict";

var core = require('web.core');
var utils = require('web_tour.utils');
var Tip = require('web_tour.Tip');

var get_first_visible_element = utils.get_first_visible_element;
var get_jquery_element_from_selector = utils.get_jquery_element_from_selector;

var RunningTourActionHelper = core.Class.extend({
    init: function (tip_widget) {
        this.tip_widget = tip_widget;
    },
    click: function (element) {
        this._click(this._get_action_values(element));
    },
    dblclick: function (element) {
        this._click(this._get_action_values(element), 2);
    },
    tripleclick: function (element) {
        this._click(this._get_action_values(element), 3);
    },
    clicknoleave: function (element) {
        this._click(this._get_action_values(element), 1, false);
    },
    text: function (text, element) {
        this._text(this._get_action_values(element), text);
    },
    drag_and_drop: function (to, element) {
        this._drag_and_drop(this._get_action_values(element), to);
    },
    keydown: function (keyCodes, element) {
        this._keydown(this._get_action_values(element), keyCodes.split(/[,\s]+/));
    },
    auto: function (element) {
        var values = this._get_action_values(element);
        if (values.consume_event === "input") {
            this._text(values);
        } else {
            this._click(values);
        }
    },
    _get_action_values: function (element) {
        var $e = get_jquery_element_from_selector(element);
        var $element = element ? get_first_visible_element($e) : this.tip_widget.$anchor;
        if ($element.length === 0) {
            $element = $e.first();
        }
        var consume_event = element ? Tip.getConsumeEventType($element) : this.tip_widget.consume_event;
        return {
            $element: $element,
            consume_event: consume_event,
        };
    },
    _click: function (values, nb, leave) {
        trigger_mouse_event(values.$element, "mouseover");
        values.$element.trigger("mouseenter");
        for (var i = 1 ; i <= (nb || 1) ; i++) {
            trigger_mouse_event(values.$element, "mousedown");
            trigger_mouse_event(values.$element, "mouseup");
            trigger_mouse_event(values.$element, "click", i);
            if (i % 2 === 0) {
                trigger_mouse_event(values.$element, "dblclick");
            }
        }
        if (leave !== false) {
            trigger_mouse_event(values.$element, "mouseout");
            values.$element.trigger("mouseleave");
        }

        function trigger_mouse_event($element, type, count) {
            var e = document.createEvent("MouseEvents");
            e.initMouseEvent(type, true, true, window, count || 0, 0, 0, 0, 0, false, false, false, false, 0, $element[0]);
            $element[0].dispatchEvent(e);
        }
    },
    _text: function (values, text) {
        this._click(values);

        text = text || "Test";
        if (values.consume_event === "input") {
            values.$element.trigger("keydown").val(text).trigger("keyup").trigger("input");
        } else if (values.$element.is("select")) {
            var $options = values.$element.children("option");
            $options.prop("selected", false).removeProp("selected");
            var $selectedOption = $options.filter(function () { return $(this).val() === text; });
            if ($selectedOption.length === 0) {
                $selectedOption = $options.filter(function () { return $(this).text().trim() === text; });
            }
            $selectedOption.prop("selected", true);
            this._click(values);
        } else {
            values.$element.focusIn();
            values.$element.trigger($.Event( "keydown", {key: '_', keyCode: 95}));
            values.$element.text(text).trigger("input");
            values.$element.focusInEnd();
            values.$element.trigger($.Event( "keyup", {key: '_', keyCode: 95}));
        }
        values.$element.trigger("change");
    },
    _drag_and_drop: function (values, to) {
        var $to;
        if (to) {
            $to = get_jquery_element_from_selector(to);
        } else {
            $to = $(document.body);
        }
        var elementCenter = values.$element.offset();
        elementCenter.left += values.$element.outerWidth()/2;
        elementCenter.top += values.$element.outerHeight()/2;

        var toCenter = $to.offset();

        if (to && to.indexOf('iframe') !== -1) {
            var iFrameOffset = $('iframe').offset();
            toCenter.left += iFrameOffset.left;
            toCenter.top += iFrameOffset.top;
        }
        toCenter.left += $to.outerWidth()/2;
        toCenter.top += $to.outerHeight()/2;

        values.$element.trigger($.Event("mouseenter"));
        values.$element.trigger($.Event("mousedown", {which: 1, pageX: elementCenter.left, pageY: elementCenter.top}));
        values.$element.trigger($.Event("mousemove", {which: 1, pageX: toCenter.left, pageY: toCenter.top}));
        values.$element.trigger($.Event("mouseup", {which: 1, pageX: toCenter.left, pageY: toCenter.top}));
     },
    _keydown: function (values, keyCodes) {
        while (keyCodes.length) {
            var keyCode = +keyCodes.shift();
            values.$element.trigger({type: "keydown", keyCode: keyCode});
            if ((keyCode > 47 && keyCode < 58) // number keys
                || keyCode === 32 // spacebar
                || (keyCode > 64 && keyCode < 91) // letter keys
                || (keyCode > 95 && keyCode < 112) // numpad keys
                || (keyCode > 185 && keyCode < 193) // ;=,-./` (in order)
                || (keyCode > 218 && keyCode < 223)) {   // [\]' (in order))
                document.execCommand("insertText", 0, String.fromCharCode(keyCode));
            }
            values.$element.trigger({type: "keyup", keyCode: keyCode});
        }
    },
});

return RunningTourActionHelper;
});

```

## File: static\src\js\tip.js

```javascript
odoo.define('web_tour.Tip', function(require) {
"use strict";

var config = require('web.config');
var core = require('web.core');
var Widget = require('web.Widget');
var _t = core._t;

var Tip = Widget.extend({
    template: "Tip",
    xmlDependencies: ['/web_tour/static/src/xml/tip.xml'],
    events: {
        click: '_onTipClicked',
        mouseenter: "_to_info_mode",
        mouseleave: "_to_bubble_mode",
    },
    /**
     * @param {Widget} parent
     * @param {Object} [info] description of the tip, containing the following keys:
     *  - content [String] the html content of the tip
     *  - event_handlers [Object] description of optional event handlers to bind to the tip:
     *    - event [String] the event name
     *    - selector [String] the jQuery selector on which the event should be bound
     *    - handler [function] the handler
     *  - position [String] tip's position ('top', 'right', 'left' or 'bottom'), default 'right'
     *  - width [int] the width in px of the tip when opened, default 270
     *  - space [int] space in px between anchor and tip, default 10
     *  - overlay [Object] x and y values for the number of pixels the mouseout detection area
     *    overlaps the opened tip, default {x: 50, y: 50}
     */
    init: function(parent, info) {
        this._super(parent);
        this.info = _.defaults(info, {
            position: "right",
            width: 270,
            space: 10,
            overlay: {
                x: 50,
                y: 50,
            },
        });
        this.position = {
            top: "50%",
            left: "50%",
        };
    },
    /**
     * @param {jQuery} $anchor the node on which the tip should be placed
     */
    attach_to: function ($anchor) {
        this.$anchor = $anchor;
        this.$ideal_location = this._get_ideal_location();

        var position = this.$ideal_location.css("position");
        if (position === "static" || position === "relative") {
            this.$ideal_location.addClass("o_tooltip_parent");
        }

        return this.appendTo(this.$ideal_location);
    },
    start: function() {
        this.$tooltip_overlay = this.$(".o_tooltip_overlay");
        this.$tooltip_content = this.$(".o_tooltip_content");
        this.init_width = this.$el.innerWidth();
        this.init_height = this.$el.innerHeight();
        this.double_border_width = this.$el.outerWidth() - this.init_width;
        this.content_width = this.$tooltip_content.outerWidth(true);
        this.content_height = this.$tooltip_content.outerHeight(true);
        this.$window = $(window);

        this.$tooltip_content.css({
            width: "100%",
            height: "100%",
        });

        _.each(this.info.event_handlers, (function(data) {
            this.$tooltip_content.on(data.event, data.selector, data.handler);
        }).bind(this));
        this._bind_anchor_events();

        this._reposition();
        this.$el.css("opacity", 1);
        core.bus.on("resize", this, _.debounce(function () {
            if (this.tip_opened) {
                this._to_bubble_mode(true);
            }
            this._reposition();
        }, 500));

        this.$el.on("transitionend oTransitionEnd webkitTransitionEnd", (function () {
            if (!this.tip_opened && this.$el.parent()[0] === document.body) {
                this.$el.detach();
                this.$el.css(this.position);
                this.$el.appendTo(this.$ideal_location);
            }
        }).bind(this));

        return this._super.apply(this, arguments);
    },
    destroy: function () {
        this._unbind_anchor_events();
        clearTimeout(this.timerIn);
        clearTimeout(this.timerOut);

        // Do not remove the parent class if it contains other tooltips
        if (this.$ideal_location.children(".o_tooltip").not(this.$el[0]).length === 0) {
            this.$ideal_location.removeClass("o_tooltip_parent");
        }

        return this._super.apply(this, arguments);
    },
    update: function ($anchor) {
        if (!$anchor.is(this.$anchor)) {
            this._unbind_anchor_events();
            this.$anchor = $anchor;
            this.$ideal_location = this._get_ideal_location();
            if (this.$el.parent()[0] !== document.body) {
                this.$el.appendTo(this.$ideal_location);
            }
            this._bind_anchor_events();
        }
        this._reposition();
    },
    _get_ideal_location: function () {
        var $location = this.$anchor;
        if ($location.is("html,body")) {
            return $(document.body);
        }

        var o;
        var p;
        do {
            $location = $location.parent();
            o = $location.css("overflow");
            p = $location.css("position");
        } while (
            $location.hasClass('dropdown-menu') ||
            (
                (o === "visible" || o === "hidden") &&
                p !== "fixed" &&
                $location[0].tagName.toUpperCase() !== 'BODY'
            )
        );

        return $location;
    },
    _reposition: function () {
        if (this.tip_opened) return;
        if (!this.$el) return;
        this.$el.removeClass("o_animated");

        // Reverse left/right position if direction is right to left
        var appendAt = this.info.position;
        var rtlMap = {left: 'right', right: 'left'};
        if (rtlMap[appendAt] && _t.database.parameters.direction === 'rtl') {
            appendAt = rtlMap[appendAt];
        }
        this.$el.position({
            my: this._get_spaced_inverted_position(appendAt),
            at: appendAt,
            of: this.$anchor,
            collision: "none",
        });

        // Reverse overlay if direction is right to left
        var positionRight = _t.database.parameters.direction === 'rtl' ? "right" : "left";
        var positionLeft = _t.database.parameters.direction === 'rtl' ? "left" : "right";
        var offset = this.$el.offset();
        this.$tooltip_overlay.css({
            top: -Math.min((this.info.position === "bottom" ? this.info.space : this.info.overlay.y), offset.top),
            right: -Math.min((this.info.position === positionRight ? this.info.space : this.info.overlay.x), this.$window.width() - (offset.left + this.init_width + this.double_border_width)),
            bottom: -Math.min((this.info.position === "top" ? this.info.space : this.info.overlay.y), this.$window.height() - (offset.top + this.init_height + this.double_border_width)),
            left: -Math.min((this.info.position === positionLeft ? this.info.space : this.info.overlay.x), offset.left),
        });

        this.position = this.$el.position();

        this.$el.addClass("o_animated");
    },
    _bind_anchor_events: function () {
        this.consume_event = this.info.consumeEvent || Tip.getConsumeEventType(this.$anchor, this.info.run);
        this.$consumeEventAnchor = this.$anchor;
        // jQuery-ui draggable triggers 'drag' events on the .ui-draggable element,
        // but the tip is attached to the .ui-draggable-handle element which may
        // be one of its children (or the element itself)
        if (this.consume_event === "drag") {
            this.$consumeEventAnchor = this.$anchor.closest('.ui-draggable');
        } else if (this.consume_event.includes('apply.daterangepicker')) {
            this.$consumeEventAnchor = this.$anchor.parent().children('.o_field_date_range');
        }
        // when an element is dragged inside a sortable container (with classname
        // 'ui-sortable'), jQuery triggers the 'sort' event on the container
        if (this.consume_event === "sort") {
            this.$consumeEventAnchor = this.$anchor.closest('.ui-sortable');
        }
        this.$consumeEventAnchor.on(this.consume_event + ".anchor", (function (e) {
            if (e.type !== "mousedown" || e.which === 1) { // only left click
                this.trigger("tip_consumed");
                this._unbind_anchor_events();
            }
        }).bind(this));
        this.$anchor.on('mouseenter.anchor', this._to_info_mode.bind(this));
        this.$anchor.on('mouseleave.anchor', this._to_bubble_mode.bind(this));
    },
    _unbind_anchor_events: function () {
        this.$anchor.off(".anchor");
        this.$consumeEventAnchor.off(".anchor");
    },
    _get_spaced_inverted_position: function (position) {
        if (position === "right") return "left+" + this.info.space;
        if (position === "left") return "right-" + this.info.space;
        if (position === "bottom") return "top+" + this.info.space;
        return "bottom-" + this.info.space;
    },
    _to_info_mode: function (force) {
        if (this.timerOut !== undefined) {
            clearTimeout(this.timerOut);
            this.timerOut = undefined;
            return;
        }
        if (this.tip_opened) {
            return;
        }

        if (force === true) {
            this._build_info_mode();
        } else {
            this.timerIn = setTimeout(this._build_info_mode.bind(this), 100);
        }
    },
    _build_info_mode: function () {
        clearTimeout(this.timerIn);
        this.timerIn = undefined;

        this.tip_opened = true;

        var offset = this.$el.offset();

        if (this.$el.parent()[0] !== document.body) {
            this.$el.detach();
            this.$el.css(offset);
            this.$el.appendTo(document.body);
        }

        var mbLeft = 0;
        var mbTop = 0;
        var overflow = false;
        var posVertical = (this.info.position === "top" || this.info.position === "bottom");
        if (posVertical) {
            overflow = (offset.left + this.content_width + this.double_border_width + this.info.overlay.x > this.$window.width());
        } else {
            overflow = (offset.top + this.content_height + this.double_border_width + this.info.overlay.y > this.$window.height());
        }
        if (posVertical && overflow || this.info.position === "left" || (_t.database.parameters.direction === 'rtl' && this.info.position == "right")) {
            mbLeft -= (this.content_width - this.init_width);
        }
        if (!posVertical && overflow || this.info.position === "top") {
            mbTop -= (this.content_height - this.init_height);
        }

        this.$el.toggleClass("inverse", overflow);
        this.$el.removeClass("o_animated").addClass("active");
        this.$el.css({
            width: this.content_width,
            height: this.content_height,
            "margin-left": mbLeft,
            "margin-top": mbTop,
        });
    },
    _to_bubble_mode: function (force) {
        if (this.timerIn !== undefined) {
            clearTimeout(this.timerIn);
            this.timerIn = undefined;
            return;
        }
        if (!this.tip_opened) {
            return;
        }

        if (force === true) {
            this._build_bubble_mode();
        } else {
            this.timerOut = setTimeout(this._build_bubble_mode.bind(this), 300);
        }
    },
    _build_bubble_mode: function () {
        clearTimeout(this.timerOut);
        this.timerOut = undefined;

        this.tip_opened = false;

        this.$el.removeClass("active").addClass("o_animated");
        this.$el.css({
            width: this.init_width,
            height: this.init_height,
            margin: 0,
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * On touch devices, closes the tip when clicked.
     *
     * @private
     */
    _onTipClicked: function () {
        if (config.device.touch && this.tip_opened) {
            this._to_bubble_mode();
        }
    },
});

/**
 * @static
 * @param {jQuery} $element
 * @param {string} [run] the run parameter of the tip (only strings are useful)
 */
Tip.getConsumeEventType = function ($element, run) {
    if ($element.hasClass('o_field_many2one') || $element.hasClass('o_field_many2manytags')) {
        return 'autocompleteselect';
    } else if ($element.is("textarea") || $element.filter("input").is(function () {
        var type = $(this).attr("type");
        return !type || !!type.match(/^(email|number|password|search|tel|text|url)$/);
    })) {
        // FieldDateRange triggers a special event when using the widget
        if ($element.hasClass("o_field_date_range")) {
            return "apply.daterangepicker input";
        }
        return "input";
    } else if ($element.hasClass('ui-draggable-handle')) {
        return "drag";
    } else if (typeof run === 'string' && run.indexOf('drag_and_drop') === 0) {
        // this is a heuristic: the element has to be dragged and dropped but it
        // doesn't have class 'ui-draggable-handle', so we check if it has an
        // ui-sortable parent, and if so, we conclude that its event type is 'sort'
        if ($element.closest('.ui-sortable').length) {
            return 'sort';
        }
    }
    return "click";
};

return Tip;

});

```

## File: static\src\js\tour_disable.js

```javascript
odoo.define('web_tour.DisableTour', function (require) {
"use strict";

var TourManager = require('web_tour.TourManager');

TourManager.include({
    /**
     * Disables tours if Odoo installed with demo data.
     *
     * @override
     */
    _register: function (do_update, tour, name) {
        // Consuming tours which are not run by test case
        if (!this.running_tour) {
            this.consumed_tours.push(name);
        }
        return this._super.apply(this, arguments);
    },
});

});

```

## File: static\src\js\tour_manager.js

```javascript
odoo.define('web_tour.TourManager', function(require) {
"use strict";

var core = require('web.core');
var local_storage = require('web.local_storage');
var mixins = require('web.mixins');
var utils = require('web_tour.utils');
var RainbowMan = require('web.RainbowMan');
var RunningTourActionHelper = require('web_tour.RunningTourActionHelper');
var ServicesMixin = require('web.ServicesMixin');
var session = require('web.session');
var Tip = require('web_tour.Tip');

var _t = core._t;

var RUNNING_TOUR_TIMEOUT = 10000;

var get_step_key = utils.get_step_key;
var get_running_key = utils.get_running_key;
var get_running_delay_key = utils.get_running_delay_key;
var get_first_visible_element = utils.get_first_visible_element;
var do_before_unload = utils.do_before_unload;
var get_jquery_element_from_selector = utils.get_jquery_element_from_selector;

return core.Class.extend(mixins.EventDispatcherMixin, ServicesMixin, {
    init: function(parent, consumed_tours) {
        mixins.EventDispatcherMixin.init.call(this);
        this.setParent(parent);

        this.$body = $('body');
        this.active_tooltips = {};
        this.tours = {};
        this.consumed_tours = consumed_tours || [];
        this.running_tour = local_storage.getItem(get_running_key());
        this.running_step_delay = parseInt(local_storage.getItem(get_running_delay_key()), 10) || 0;
        this.edition = (_.last(session.server_version_info) === 'e') ? 'enterprise' : 'community';
        this._log = [];
        console.log('Tour Manager is ready.  running_tour=' + this.running_tour);
    },
    /**
     * Registers a tour described by the following arguments *in order*
     *
     * @param {string} name - tour's name
     * @param {Object} [options] - options (optional), available options are:
     * @param {boolean} [options.test=false] - true if this is only for tests
     * @param {boolean} [options.skip_enabled=false]
     *        true to add a link in its tips to consume the whole tour
     * @param {string} [options.url]
     *        the url to load when manually running the tour
     * @param {boolean} [options.rainbowMan=true]
     *        whether or not the rainbowman must be shown at the end of the tour
     * @param {Promise} [options.wait_for]
     *        indicates when the tour can be started
     * @param {Object[]} steps - steps' descriptions, each step being an object
     *                     containing a tip description
     */
    register: function() {
        var args = Array.prototype.slice.call(arguments);
        var last_arg = args[args.length - 1];
        var name = args[0];
        if (this.tours[name]) {
            console.warn(_.str.sprintf("Tour %s is already defined", name));
            return;
        }
        var options = args.length === 2 ? {} : args[1];
        var steps = last_arg instanceof Array ? last_arg : [last_arg];
        var tour = {
            name: name,
            steps: steps,
            url: options.url,
            rainbowMan: options.rainbowMan === undefined ? true : !!options.rainbowMan,
            test: options.test,
            wait_for: options.wait_for || Promise.resolve(),
        };
        if (options.skip_enabled) {
            tour.skip_link = '<p><span class="o_skip_tour">' + _t('Skip tour') + '</span></p>';
            tour.skip_handler = function (tip) {
                this._deactivate_tip(tip);
                this._consume_tour(name);
            };
        }
        this.tours[name] = tour;
    },
    /**
     * Returns a promise which is resolved once the tour can be started. This
     * is when the DOM is ready and at the end of the execution stack so that
     * all tours have potentially been extended by all apps.
     *
     * @private
     * @returns {Promise}
     */
    _waitBeforeTourStart: function () {
        return new Promise(function (resolve) {
            $(function () {
                setTimeout(resolve);
            });
        });
    },
    _register_all: function (do_update) {
        var self = this;
        if (this._allRegistered) {
            return Promise.resolve();
        }
        this._allRegistered = true;
        return self._waitBeforeTourStart().then(function () {
            return Promise.all(_.map(self.tours, function (tour, name) {
                return self._register(do_update, tour, name);
            })).then(() => self.update());
        });
    },
    _register: function (do_update, tour, name) {
        if (tour.ready) return Promise.resolve();

        var tour_is_consumed = _.contains(this.consumed_tours, name);

        return tour.wait_for.then((function () {
            tour.current_step = parseInt(local_storage.getItem(get_step_key(name))) || 0;
            tour.steps = _.filter(tour.steps, (function (step) {
                return !step.edition || step.edition === this.edition;
            }).bind(this));

            if (tour_is_consumed || tour.current_step >= tour.steps.length) {
                local_storage.removeItem(get_step_key(name));
                tour.current_step = 0;
            }

            tour.ready = true;

            if (do_update && (this.running_tour === name || (!this.running_tour && !tour.test && !tour_is_consumed))) {
                this._to_next_step(name, 0);
            }
        }).bind(this));
    },
    run: function (tour_name, step_delay) {
        console.log(_.str.sprintf("Preparing tour %s", tour_name));
        if (this.running_tour) {
            this._deactivate_tip(this.active_tooltips[this.running_tour]);
            this._consume_tour(this.running_tour, _.str.sprintf("Killing tour %s", this.running_tour));
            return;
        }
        var tour = this.tours[tour_name];
        if (!tour) {
            console.warn(_.str.sprintf("Unknown Tour %s", name));
            return;
        }
        console.log(_.str.sprintf("Running tour %s", tour_name));
        this.running_tour = tour_name;
        this.running_step_delay = step_delay || this.running_step_delay;
        local_storage.setItem(get_running_key(), this.running_tour);
        local_storage.setItem(get_running_delay_key(), this.running_step_delay);

        this._deactivate_tip(this.active_tooltips[tour_name]);

        tour.current_step = 0;
        this._to_next_step(tour_name, 0);
        local_storage.setItem(get_step_key(tour_name), tour.current_step);

        if (tour.url) {
            this.pause();
            do_before_unload(null, (function () {
                this.play();
                this.update();
            }).bind(this));

            window.location.href = window.location.origin + tour.url;
        } else {
            this.update();
        }
    },
    pause: function () {
        this.paused = true;
    },
    play: function () {
        this.paused = false;
    },
    /**
     * Checks for tooltips to activate (only from the running tour or specified tour if there
     * is one, from all active tours otherwise). Should be called each time the DOM changes.
     */
    update: function (tour_name) {
        if (this.paused) return;

        this.$modal_displayed = $('.modal:visible').last();

        tour_name = this.running_tour || tour_name;
        if (tour_name) {
            var tour = this.tours[tour_name];
            if (!tour || !tour.ready) return;

            if (this.running_tour && this.running_tour_timeout === undefined) {
                this._set_running_tour_timeout(this.running_tour, this.active_tooltips[this.running_tour]);
            }
            var self = this;
            setTimeout(function () {
                self._check_for_tooltip(self.active_tooltips[tour_name], tour_name);
            });
        } else {
            for (var tourName in this.active_tooltips) {
                var tip = this.active_tooltips[tourName];
                var activated = this._check_for_tooltip(tip, tourName);
                if (activated) {
                    break;
                }
            }
        }
    },
    /**
     *  Check (and activate or update) a help tooltip for a tour.
     *
     * @param {Object} tip
     * @param {string} tour_name
     * @returns {boolean} true if a tip was found and activated/updated
     */
    _check_for_tooltip: function (tip, tour_name) {
        if (tip === undefined) {
            return true;
        }
        if ($('body').hasClass('o_ui_blocked')) {
            this._deactivate_tip(tip);
            this._log.push("blockUI is preventing the tip to be consumed");
            return false;
        }

        var $trigger;
        if (tip.in_modal !== false && this.$modal_displayed.length) {
            $trigger = this.$modal_displayed.find(tip.trigger);
        } else {
            $trigger = get_jquery_element_from_selector(tip.trigger);
        }
        var $visible_trigger = get_first_visible_element($trigger);

        var extra_trigger = true;
        var $extra_trigger;
        if (tip.extra_trigger) {
            $extra_trigger = get_jquery_element_from_selector(tip.extra_trigger);
            extra_trigger = get_first_visible_element($extra_trigger).length;
        }

        var triggered = $visible_trigger.length && extra_trigger;
        if (triggered) {
            if (!tip.widget) {
                this._activate_tip(tip, tour_name, $visible_trigger);
            } else {
                tip.widget.update($visible_trigger);
            }
        } else {
            if ($trigger.iframeContainer || ($extra_trigger && $extra_trigger.iframeContainer)) {
                var $el = $();
                if ($trigger.iframeContainer) {
                    $el = $el.add($trigger.iframeContainer);
                }
                if (($extra_trigger && $extra_trigger.iframeContainer) && $trigger.iframeContainer !== $extra_trigger.iframeContainer) {
                    $el = $el.add($extra_trigger.iframeContainer);
                }
                var self = this;
                $el.off('load').one('load', function () {
                    $el.off('load');
                    if (self.active_tooltips[tour_name] === tip) {
                        self.update(tour_name);
                    }
                });
            }
            this._deactivate_tip(tip);

            if (this.running_tour === tour_name) {
                this._log.push("_check_for_tooltip");
                this._log.push("- modal_displayed: " + this.$modal_displayed.length);
                this._log.push("- trigger '" + tip.trigger + "': " + $trigger.length);
                this._log.push("- visible trigger '" + tip.trigger + "': " + $visible_trigger.length);
                if ($extra_trigger !== undefined) {
                    this._log.push("- extra_trigger '" + tip.extra_trigger + "': " + $extra_trigger.length);
                    this._log.push("- visible extra_trigger '" + tip.extra_trigger + "': " + extra_trigger);
                }
            }
        }
        return !!triggered;
    },
    _activate_tip: function(tip, tour_name, $anchor) {
        var tour = this.tours[tour_name];
        var tip_info = tip;
        if (tour.skip_link) {
            tip_info = _.extend(_.omit(tip_info, 'content'), {
                content: tip.content + tour.skip_link,
                event_handlers: [{
                    event: 'click',
                    selector: '.o_skip_tour',
                    handler: tour.skip_handler.bind(this, tip),
                }],
            });
        }
        tip.widget = new Tip(this, tip_info);
        if (this.running_tour !== tour_name) {
            tip.widget.on('tip_consumed', this, this._consume_tip.bind(this, tip, tour_name));
        }
        tip.widget.attach_to($anchor).then(this._to_next_running_step.bind(this, tip, tour_name));
    },
    _deactivate_tip: function(tip) {
        if (tip && tip.widget) {
            tip.widget.destroy();
            delete tip.widget;
        }
    },
    _describeTip: function(tip) {
        return tip.content ? tip.content + ' (trigger: ' + tip.trigger + ')' : tip.trigger;
    },
    _consume_tip: function(tip, tour_name) {
        this._deactivate_tip(tip);
        this._to_next_step(tour_name);

        var is_running = (this.running_tour === tour_name);
        if (is_running) {
            var stepDescription = this._describeTip(tip);
            console.log(_.str.sprintf("Tour %s: step '%s' succeeded", tour_name, stepDescription));
        }

        if (this.active_tooltips[tour_name]) {
            local_storage.setItem(get_step_key(tour_name), this.tours[tour_name].current_step);
            if (is_running) {
                this._log = [];
                this._set_running_tour_timeout(tour_name, this.active_tooltips[tour_name]);
            }
            this.update(tour_name);
        } else {
            this._consume_tour(tour_name);
        }
    },
    _to_next_step: function (tour_name, inc) {
        var tour = this.tours[tour_name];
        tour.current_step += (inc !== undefined ? inc : 1);
        if (this.running_tour !== tour_name) {
            var index = _.findIndex(tour.steps.slice(tour.current_step), function (tip) {
                return !tip.auto;
            });
            if (index >= 0) {
                tour.current_step += index;
            } else {
                tour.current_step = tour.steps.length;
            }
        }
        this.active_tooltips[tour_name] = tour.steps[tour.current_step];
    },
    _consume_tour: function (tour_name, error) {
        delete this.active_tooltips[tour_name];
        //display rainbow at the end of any tour
        if (this.tours[tour_name].rainbowMan && this.running_tour !== tour_name &&
            this.tours[tour_name].current_step === this.tours[tour_name].steps.length) {
            var $rainbow_message = $('<strong>' +
                                '<b>Good job!</b>' +
                                ' You went through all steps of this tour.' +
                                '</strong>');
            new RainbowMan({message: $rainbow_message}).appendTo(this.$body);
        }
        this.tours[tour_name].current_step = 0;
        local_storage.removeItem(get_step_key(tour_name));
        if (this.running_tour === tour_name) {
            this._stop_running_tour_timeout();
            local_storage.removeItem(get_running_key());
            local_storage.removeItem(get_running_delay_key());
            this.running_tour = undefined;
            this.running_step_delay = undefined;
            if (error) {
                _.each(this._log, function (log) {
                    console.log(log);
                });
                console.log(document.body.parentElement.outerHTML);
                console.error(error); // will be displayed as error info
            } else {
                console.log(_.str.sprintf("Tour %s succeeded", tour_name));
                console.log("test successful"); // browser_js wait for message "test successful"
            }
            this._log = [];
        } else {
            var self = this;
            this._rpc({
                    model: 'web_tour.tour',
                    method: 'consume',
                    args: [[tour_name]],
                })
                .then(function () {
                    self.consumed_tours.push(tour_name);
                });
        }
    },
    _set_running_tour_timeout: function (tour_name, step) {
        this._stop_running_tour_timeout();
        this.running_tour_timeout = setTimeout((function() {
            var descr = this._describeTip(step);
            this._consume_tour(tour_name, _.str.sprintf("Tour %s failed at step %s", tour_name, descr));
        }).bind(this), (step.timeout || RUNNING_TOUR_TIMEOUT) + this.running_step_delay);
    },
    _stop_running_tour_timeout: function () {
        clearTimeout(this.running_tour_timeout);
        this.running_tour_timeout = undefined;
    },
    _to_next_running_step: function (tip, tour_name) {
        if (this.running_tour !== tour_name) return;
        var self = this;
        this._stop_running_tour_timeout();
        if (this.running_step_delay) {
            // warning: due to the delay, it may happen that the $anchor isn't
            // in the DOM anymore when exec is called, either because:
            // - it has been removed from the DOM meanwhile and the tip's
            //   selector doesn't match anything anymore
            // - it has been re-rendered and thus the selector still has a match
            //   in the DOM, but executing the step with that $anchor won't work
            _.delay(exec, this.running_step_delay);
        } else {
            exec();
        }

        function exec() {
            var action_helper = new RunningTourActionHelper(tip.widget);
            do_before_unload(self._consume_tip.bind(self, tip, tour_name));

            var tour = self.tours[tour_name];
            if (typeof tip.run === "function") {
                tip.run.call(tip.widget, action_helper);
            } else if (tip.run !== undefined) {
                var m = tip.run.match(/^([a-zA-Z0-9_]+) *(?:\(? *(.+?) *\)?)?$/);
                action_helper[m[1]](m[2]);
            } else if (tour.current_step === tour.steps.length - 1) {
                console.log('Tour %s: ignoring action (auto) of last step', tour_name);
            } else {
                action_helper.auto();
            }
        }
    },

    /**
     * Tour predefined steps
     */
    STEPS: {
        SHOW_APPS_MENU_ITEM: {
            edition: 'community',
            trigger: '.o_menu_apps a',
            auto: true,
            position: "bottom",
        },

        TOGGLE_HOME_MENU: {
            edition: "enterprise",
            trigger: ".o_main_navbar .o_menu_toggle",
            content: _t('Click on the <i>Home icon</i> to navigate across apps.'),
            position: "bottom",
        },

        WEBSITE_NEW_PAGE: {
            trigger: "#new-content-menu > a",
            auto: true,
            position: "bottom",
        },
    },
});
});

```

## File: static\src\js\tour_service.js

```javascript
odoo.define('web_tour.tour', function (require) {
"use strict";

var config = require('web.config');
var rootWidget = require('root.widget');
var rpc = require('web.rpc');
var session = require('web.session');
var TourManager = require('web_tour.TourManager');

if (config.device.isMobile) {
    return Promise.reject();
}
/**
 * @namespace
 * @property {Object} active_tooltips
 * @property {Object} tours
 * @property {Array} consumed_tours
 * @property {String} running_tour
 * @property {Number} running_step_delay
 * @property {'community' | 'enterprise'} edition
 * @property {Array} _log
 */
return session.is_bound.then(function () {
    var defs = [];
    // Load the list of consumed tours and the tip template only if we are admin, in the frontend,
    // tours being only available for the admin. For the backend, the list of consumed is directly
    // in the page source.
    if (session.is_frontend && session.is_admin) {
        var def = rpc.query({
                model: 'web_tour.tour',
                method: 'get_consumed_tours',
            });
        defs.push(def);
    }
    return Promise.all(defs).then(function (results) {
        var consumed_tours = session.is_frontend ? results[0] : session.web_tours;
        var tour_manager = new TourManager(rootWidget, consumed_tours);

        // Use a MutationObserver to detect DOM changes
        var untracked_classnames = ["o_tooltip", "o_tooltip_content", "o_tooltip_overlay"];
        var check_tooltip = _.debounce(function (records) {
            var update = _.some(records, function (record) {
                return !(is_untracked(record.target) ||
                    _.some(record.addedNodes, is_untracked) ||
                    _.some(record.removedNodes, is_untracked));

                function is_untracked(node) {
                    var record_class = node.className;
                    return (_.isString(record_class) &&
                        _.intersection(record_class.split(' '), untracked_classnames).length !== 0);
                }
            });
            if (update) { // ignore mutations which concern the tooltips
                tour_manager.update();
            }
        }, 500);
        var observer = new MutationObserver(check_tooltip);
        var start_service = (function () {
            return function (observe) {
                return new Promise(function (resolve, reject) {
                    tour_manager._register_all(observe).then(function () {
                        if (observe) {
                            observer.observe(document.body, {
                                attributes: true,
                                childList: true,
                                subtree: true,
                            });
                        }
                        resolve();
                    });
                });
            };
        })();

        // Enable the MutationObserver for the admin or if a tour is running, when the DOM is ready
        start_service(session.is_admin || tour_manager.running_tour);

        // Override the TourManager so that it enables/disables the observer when necessary
        if (!session.is_admin) {
            var run = tour_manager.run;
            tour_manager.run = function () {
                var self = this;
                var args = arguments;

                start_service(true).then(function () {
                    run.apply(self, args);
                    if (!self.running_tour) {
                        observer.disconnect();
                    }
                });
            };
            var _consume_tour = tour_manager._consume_tour;
            tour_manager._consume_tour = function () {
                _consume_tour.apply(this, arguments);
                observer.disconnect();
            };
        }
        // helper to start a tour manually (or from a python test with its counterpart start_tour function)
        odoo.startTour = tour_manager.run.bind(tour_manager);
        return tour_manager;
    });
});

});

```

## File: static\src\js\tour_utils.js

```javascript
odoo.define('web_tour.utils', function(require) {
"use strict";

function get_step_key(name) {
    return 'tour_' + name + '_step';
}

function get_running_key() {
    return 'running_tour';
}

function get_running_delay_key() {
    return get_running_key() + "_delay";
}

function get_first_visible_element($elements) {
    for (var i = 0 ; i < $elements.length ; i++) {
        var $i = $elements.eq(i);
        if ($i.is(':visible:hasVisibility')) {
            return $i;
        }
    }
    return $();
}

function do_before_unload(if_unload_callback, if_not_unload_callback) {
    if_unload_callback = if_unload_callback || function () {};
    if_not_unload_callback = if_not_unload_callback || if_unload_callback;

    var old_before = window.onbeforeunload;
    var reload_timeout;
    window.onbeforeunload = function () {
        clearTimeout(reload_timeout);
        window.onbeforeunload = old_before;
        if_unload_callback();
        if (old_before) return old_before.apply(this, arguments);
    };
    reload_timeout = _.defer(function () {
        window.onbeforeunload = old_before;
        if_not_unload_callback();
    });
}

function get_jquery_element_from_selector(selector) {
    if (_.isString(selector) && selector.indexOf('iframe') !== -1) {
        var $iframe = $(selector.split('iframe')[0] + ' iframe');
        var $el = $iframe.contents()
            .find(selector.split('iframe')[1]);
        $el.iframeContainer = $iframe[0];
        return $el;
    } else {
        return $(selector);
    }
}


return {

    'get_step_key': get_step_key,
    'get_running_key': get_running_key,
    'get_running_delay_key': get_running_delay_key,
    'get_first_visible_element': get_first_visible_element,
    'do_before_unload': do_before_unload,
    'get_jquery_element_from_selector' : get_jquery_element_from_selector,
};

});


```

## File: static\src\js\public\tour_manager.js

```javascript
odoo.define('web_tour.public.TourManager', function (require) {
'use strict';

var TourManager = require('web_tour.TourManager');
var lazyloader = require('web.public.lazyloader');

TourManager.include({
    /**
     * @override
     */
    _waitBeforeTourStart: function () {
        return this._super.apply(this, arguments).then(function () {
            return lazyloader.allScriptsLoaded;
        }).then(function () {
            return new Promise(function (resolve) {
                setTimeout(resolve);
            });
        });
    },
});
});

```

## File: static\src\xml\debug_manager.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-extend="WebClient.DebugManager.Backend">
    <t t-jquery="a[data-action='select_view']" t-operation="after">
        <t t-if="manager._is_admin">
            <a t-if="manager.consume_tours_enabled" href="#" class="dropdown-item" data-action="consume_tours">Disable Tours</a>
            <a href="#" class="dropdown-item" data-action="start_tour">Start Tour</a>
        </t>
    </t>
</t>

<t t-name="WebClient.DebugManager.ToursDialog">
    <div>
        <table class="table table-sm table-striped table-responsive">
            <tr>
                <th>Name</th>
                <th>Path</th>
                <th/>
            </tr>

            <tr t-foreach="tours" t-as="tour">
                <td><t t-esc="tour"/></td>
                <td><t t-esc="tours[tour].url"/></td>
                <td><button type="button" class="btn btn-primary fa fa-play o_start_tour" t-att-data-name="tour" aria-label="Tour" title="Tour"/></td>
            </tr>
        </table>
    </div>
</t>

</templates>

```

## File: static\src\xml\tip.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="Tip" t-attf-class="o_tooltip #{widget.info.position}">
        <div class="o_tooltip_overlay"/>
        <div class="o_tooltip_content" t-attf-style="width: #{widget.info.width}px;">
            <t t-raw="widget.info.content"/>
        </div>
    </div>
</templates>

```

## File: views\tour_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_common" name="tours assets" inherit_id="web.assets_common">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/web_tour/static/src/scss/tip.scss"/>
                <link rel="stylesheet" type="text/scss" href="/web_tour/static/src/scss/keyframes.scss"/>
                <script type="text/javascript" src="/web_tour/static/src/js/tip.js"></script>
                <script type="text/javascript" src="/web_tour/static/src/js/tour_utils.js"></script>
                <script type="text/javascript" src="/web_tour/static/src/js/running_tour_action_helper.js"></script>
                <script type="text/javascript" src="/web_tour/static/src/js/tour_manager.js"></script>
                <script type="text/javascript" src="/web_tour/static/src/js/tour_service.js"></script>
            </xpath>
        </template>

        <template id="assets_backend" name="tours backend assets" inherit_id="web.assets_backend">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/web_tour/static/src/js/debug_manager.js"></script>
            </xpath>
        </template>

        <template id="assets_frontend" inherit_id="web.assets_frontend">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/web_tour/static/src/js/public/tour_manager.js"/>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\tour_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="edit_tour_list" model="ir.ui.view">
            <field name="model">web_tour.tour</field>
            <field name="arch" type="xml">
                <tree string="Menu" editable="bottom">
                    <field name="name"/>
                    <field name="user_id"/>
                </tree>
            </field>
        </record>
        <record id="edit_tour_search" model="ir.ui.view">
            <field name="name">tour.search</field>
            <field name="model">web_tour.tour</field>
            <field name="arch" type="xml">
                <search string="Tip">
                    <field name="name"/>
                </search>
            </field>
        </record>
        <record id="edit_tour_action" model="ir.actions.act_window">
            <field name="name">Tours</field>
            <field name="res_model">web_tour.tour</field>
            <field name="view_id" ref="edit_tour_list"/>
            <field name="search_view_id" ref="edit_tour_search"/>
        </record>
        <menuitem action="edit_tour_action" id="menu_tour_action" parent="base.next_id_2" sequence="5"/>
</odoo>

```

