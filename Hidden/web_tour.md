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
        'views/tour_views.xml'
    ],
    'assets': {
        'web.assets_common': [
            'web_tour/static/src/scss/**/*',
            'web_tour/static/src/js/running_tour_action_helper.js',
            'web_tour/static/src/js/tip.js',
            'web_tour/static/src/js/tour_manager.js',
            'web_tour/static/src/js/tour_service.js',
            'web_tour/static/src/js/tour_step_utils.js',
            'web_tour/static/src/js/tour_utils.js',
            '/web_tour/static/src/xml/tip.xml',
        ],
        'web.assets_backend': [
            'web_tour/static/src/debug/debug_manager.js',
            'web_tour/static/src/debug/tour_dialog_component.js',
            'web_tour/static/src/services/*.js',
            'web_tour/static/src/debug/tour_dialog_component.xml',
        ],
        'web.assets_frontend': [
            'web_tour/static/src/scss/**/*',
            'web_tour/static/src/js/running_tour_action_helper.js',
            'web_tour/static/src/js/tip.js',
            'web_tour/static/src/js/tour_manager.js',
            'web_tour/static/src/js/tour_service.js',
            'web_tour/static/src/js/tour_step_utils.js',
            'web_tour/static/src/js/tour_utils.js',
            '/web_tour/static/src/xml/tip.xml',

            'web_tour/static/src/js/public/**/*',
        ],
        'web.qunit_suite_tests': [
            'web_tour/static/tests/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super().session_info()
        if result['is_admin']:
            demo_modules_count = self.env['ir.module.module'].sudo().search_count([('demo', '=', True)])
            result['web_tours'] = self.env['web_tour.tour'].get_consumed_tours()
            result['tour_disable'] = demo_modules_count > 0
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
        if not self.env.user.has_group('base.group_user'):
            # Only internal users can use this method.
            # TODO master: update ir.model.access records instead of using sudo()
            return
        for name in tour_names:
            self.sudo().create({'name': name, 'user_id': self.env.uid})

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

## File: static\src\debug\debug_manager.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { registry } from "@web/core/registry";
import ToursDialog from "@web_tour/debug/tour_dialog_component";
import utils from "web_tour.utils";

export function disableTours({ env }) {
    if (!env.services.user.isSystem) {
        return null;
    }
    const activeTours = env.services.tour.getActiveTours();
    if (activeTours.length === 0) {
        return null;
    }
    return {
        type: "item",
        description: env._t("Disable Tours"),
        callback: async () => {
            const tourNames = activeTours.map(tour => tour.name);
            await env.services.orm.call("web_tour.tour", "consume", [tourNames]);
            for (const tourName of tourNames) {
                browser.localStorage.removeItem(utils.get_debugging_key(tourName));
            }
            browser.location.reload();
        },
        sequence: 50,
    };
}

export function startTour({ env }) {
    if (!env.services.user.isSystem) {
        return null;
    }
    return {
        type: "item",
        description: env._t("Start Tour"),
        callback: async () => {
            env.services.dialog.add(ToursDialog);
        },
        sequence: 60,
    };
}

registry
    .category("debug")
    .category("default")
    .add("web_tour.startTour", startTour)
    .add("web_tour.disableTours", disableTours);

```

## File: static\src\debug\tour_dialog_component.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { Dialog } from "@web/core/dialog/dialog";
import { _lt } from "@web/core/l10n/translation";

import { Component } from "@odoo/owl";

export default class ToursDialog extends Component {
    setup() {
        this.tourService = useService("tour");
        this.onboardingTours = this.tourService.getOnboardingTours();
        this.testingTours = this.tourService.getTestingTours();
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Resets the given tour to its initial step, in onboarding mode.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onStartTour(ev) {
        this.tourService.reset(ev.target.dataset.name);
        this.props.close();
    }
    /**
     * Starts the given tour in test mode.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onTestTour(ev) {
        this.tourService.run(ev.target.dataset.name);
        this.props.close();
    }
}
ToursDialog.template = "web_tour.ToursDialog";
ToursDialog.components = { Dialog };
ToursDialog.title = _lt("Tours");

```

## File: static\src\debug\tour_dialog_component.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="web_tour.ToursDialog" owl="1">
    <Dialog title="this.constructor.title">
        <t t-call="web_tour.ToursDialog.Table">
            <t t-set="caption">Onboarding tours</t>
            <t t-set="tours" t-value="onboardingTours"/>
        </t>
        <t t-if="testingTours.length" t-call="web_tour.ToursDialog.Table">
            <t t-set="caption">Testing tours</t>
            <t t-set="tours" t-value="testingTours"/>
        </t>
    </Dialog>
</t>

<t t-name="web_tour.ToursDialog.Table" owl="1">
    <div class="table-responsive">
        <table class="table table-sm table-striped">
            <caption style="caption-side: top; font-size: 14px">
                <t t-esc="caption"/>
            </caption>
            <thead>
                <tr>
                    <th>Sequence</th>
                    <th width="50%">Name</th>
                    <th width="50%">Path</th>
                    <th>Start</th>
                    <th>Test</th>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="tours" t-as="tour" t-key="tour.name">
                    <td><t t-esc="tour.sequence"/></td>
                    <td><t t-esc="tour.name"/></td>
                    <td><t t-esc="tour.url"/></td>
                    <td>
                        <button type="button"
                            class="btn btn-primary fa fa-play o_start_tour"
                            t-on-click.prevent="_onStartTour"
                            t-att-data-name="tour.name"
                            aria-label="Start tour"
                            title="Start tour"/>
                    </td>
                    <td>
                        <button type="button"
                            class="btn btn-primary fa fa-cogs o_test_tour"
                            t-on-click.prevent="_onTestTour"
                            t-att-data-name="tour.name"
                            aria-label="Test tour"
                            title="Test tour"/>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</t>

</templates>

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
    remove_text(text, element) {
        this._text(this._get_action_values(element), '\n');
    },
    text_blur: function (text, element) {
        this._text_blur(this._get_action_values(element), text);
    },
    drag_and_drop: function (to, element) {
        this._drag_and_drop_jquery(this._get_action_values(element), to);
    },
    drag_and_drop_native: function (toSel, element) {
        const to = get_jquery_element_from_selector(toSel)[0];
        this._drag_and_drop(this._get_action_values(element).$element[0], to);
    },
    drag_move_and_drop: function (to, element) {
        this._drag_move_and_drop(this._get_action_values(element), to);
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
            values.$element
                .trigger({ type: 'keydown', key: text[text.length - 1] })
                .val(text)
                .trigger({ type: 'keyup', key: text[text.length - 1] });
            values.$element[0].dispatchEvent(new InputEvent('input', {
                bubbles: true,
            }));
        } else if (values.$element.is("select")) {
            var $options = values.$element.find("option");
            $options.prop("selected", false).removeProp("selected");
            var $selectedOption = $options.filter(function () { return $(this).val() === text; });
            if ($selectedOption.length === 0) {
                $selectedOption = $options.filter(function () { return $(this).text().trim() === text; });
            }
            const regex = /option\s+([0-9]+)/;
            if ($selectedOption.length === 0 && regex.test(text)) {
                // Extract position as 1-based, as the nth selectors.
                const position = parseInt(regex.exec(text)[1]);
                $selectedOption = $options.eq(position - 1); // eq is 0-based.
            }
            $selectedOption.prop("selected", true);
            this._click(values);
            // For situations where an `oninput` is defined.
            values.$element.trigger("input");
        } else {
            values.$element.focusIn();
            values.$element.trigger($.Event( "keydown", {key: '_', keyCode: 95}));
            values.$element.text(text).trigger("input");
            values.$element.focusInEnd();
            values.$element.trigger($.Event( "keyup", {key: '_', keyCode: 95}));
        }
        values.$element[0].dispatchEvent(new Event("change", { bubbles: true, cancelable: false }));
    },
    _text_blur: function (values, text) {
        this._text(values, text);
        values.$element.trigger('focusout');
        values.$element.trigger('blur');
    },
    _calculateCenter: function ($el, selector) {
        const center = $el.offset();
        if (selector && selector.indexOf('iframe') !== -1) {
            const iFrameOffset = $('iframe').offset();
            center.left += iFrameOffset.left;
            center.top += iFrameOffset.top;
        }
        center.left += $el.outerWidth() / 2;
        center.top += $el.outerHeight() / 2;
        return center;
    },
    _drag_and_drop_jquery: function (values, to) {
        var $to;
        const elementCenter = this._calculateCenter(values.$element);
        if (to) {
            $to = get_jquery_element_from_selector(to);
        } else {
            $to = $(document.body);
        }

        values.$element.trigger($.Event("mouseenter"));
        values.$element.trigger($.Event("mousedown", {which: 1, pageX: elementCenter.left, pageY: elementCenter.top}));
        // Some tests depends on elements present only when the element to drag
        // start to move while some other tests break while moving.
        if (!$to.length) {
            values.$element.trigger($.Event("mousemove", {which: 1, pageX: elementCenter.left + 1, pageY: elementCenter.top}));
            $to = get_jquery_element_from_selector(to);
        }

        let toCenter = this._calculateCenter($to, to);
        values.$element.trigger($.Event("mousemove", {which: 1, pageX: toCenter.left, pageY: toCenter.top}));
        // Recalculate the center as the mousemove might have made the element bigger.
        toCenter = this._calculateCenter($to, to);
        values.$element.trigger($.Event("mouseup", {which: 1, pageX: toCenter.left, pageY: toCenter.top}));
    },
    _drag_and_drop: function (element, to) {
        const elementCenter = this._calculateCenter($(element));
        const toCenter = this._calculateCenter($(to));
        element.dispatchEvent(new Event("mouseenter"));
        element.dispatchEvent(new MouseEvent("mousedown", {
            bubbles: true,
            cancelable: true,
            button: 0,
            which: 1,
            clientX: elementCenter.left,
            clientY: elementCenter.top,
        }));
        element.dispatchEvent(new MouseEvent("mousemove", { bubbles: true, cancelable: true, clientX: toCenter.left, clientY: toCenter.top}));
        to.dispatchEvent(new Event("mouseenter", { clientX: toCenter.left, clientY: toCenter.top }));
        element.dispatchEvent(new Event("mouseup", {
            bubbles: true,
            cancelable: true,
            button: 0,
            which: 1,
        }));
    },
    _drag_move_and_drop: function (values, params) {
        // Extract parameters from string: '[deltaX,deltaY]@from => actualTo'.
        const parts = /^\[(.+),(.+)\]@(.+) => (.+)/.exec(params);
        const initialMoveOffset = [parseInt(parts[1]), parseInt(parts[2])];
        const fromSelector = parts[3];
        const toSelector = parts[4];
        // Click on element.
        values.$element.trigger($.Event("mouseenter"));
        const elementCenter = this._calculateCenter(values.$element);
        values.$element.trigger($.Event("mousedown", {which: 1, pageX: elementCenter.left, pageY: elementCenter.top}));
        // Drag through "from".
        const fromCenter = this._calculateCenter(get_jquery_element_from_selector(fromSelector), fromSelector);
        values.$element.trigger($.Event("mousemove", {
            which: 1,
            pageX: fromCenter.left + initialMoveOffset[0],
            pageY: fromCenter.top + initialMoveOffset[1],
        }));
        // Drop into "to".
        const toCenter = this._calculateCenter(get_jquery_element_from_selector(toSelector), toSelector);
        values.$element.trigger($.Event("mouseup", {which: 1, pageX: toCenter.left, pageY: toCenter.top}));
    },
    _keydown: function (values, keyCodes) {
        while (keyCodes.length) {
            const eventOptions = {};
            const keyCode = keyCodes.shift();
            let insertedText = null;
            if (isNaN(keyCode)) {
                eventOptions.key = keyCode;
            } else {
                const code = parseInt(keyCode, 10);
                eventOptions.keyCode = code;
                eventOptions.which = code;
                if (
                    code === 32 || // spacebar
                    (code > 47 && code < 58) || // number keys
                    (code > 64 && code < 91) || // letter keys
                    (code > 95 && code < 112) || // numpad keys
                    (code > 185 && code < 193) || // ;=,-./` (in order)
                    (code > 218 && code < 223) // [\]' (in order))
                ) {
                    insertedText = String.fromCharCode(code);
                }
            }
            values.$element.trigger(Object.assign({ type: "keydown" }, eventOptions));
            if (insertedText) {
                values.$element[0].ownerDocument.execCommand("insertText", 0, insertedText);
            }
            values.$element.trigger(Object.assign({ type: "keyup" }, eventOptions));
        }
    },
});

return RunningTourActionHelper;
});

```

## File: static\src\js\tip.js

```javascript
odoo.define('web_tour.Tip', function (require) {
"use strict";

var config = require('web.config');
var core = require('web.core');
var Widget = require('web.Widget');
var _t = core._t;

var Tip = Widget.extend({
    template: "Tip",
    events: {
        click: '_onTipClicked',
        mouseenter: '_onMouseEnter',
        mouseleave: '_onMouseLeave',
        transitionend: '_onTransitionEnd',
    },
    CENTER_ON_TEXT_TAGS: ['P', 'H1', 'H2', 'H3', 'H4', 'H5', 'H6'],

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
     *  - space [int] space in px between anchor and tip, default to 0, added to
     *    the natural space chosen in css
     *  - hidden [boolean] if true, the tip won't be visible (but the handlers will still be
     *    bound on the anchor, so that the tip is consumed if the user clicks on it)
     *  - overlay [Object] x and y values for the number of pixels the mouseout detection area
     *    overlaps the opened tip, default {x: 50, y: 50}
     */
    init: function(parent, info) {
        this._super(parent);
        this.info = _.defaults(info, {
            position: "right",
            width: 270,
            space: 0,
            overlay: {
                x: 50,
                y: 50,
            },
            content: _t("Click here to go to the next step."),
            scrollContent: _t("Scroll to reach the next step."),
        });
        this.position = {
            top: "50%",
            left: "50%",
        };
        this.initialPosition = this.info.position;
        this.viewPortState = 'in';
        this._onAncestorScroll = _.throttle(this._onAncestorScroll, 0.1);
    },
    /**
     * Attaches the tip to the provided $anchor and $altAnchor.
     * $altAnchor is an alternative trigger that can consume the step. The tip is
     * however only displayed on the $anchor.
     *
     * Note that the returned promise stays pending if the Tip widget was
     * destroyed in the meantime.
     *
     * @param {jQuery} $anchor the node on which the tip should be placed
     * @param {jQuery} $altAnchor an alternative node that can consume the step
     * @return {Promise}
     */
    attach_to: async function ($anchor, $altAnchor) {
        this._setupAnchor($anchor, $altAnchor);

        this.is_anchor_fixed_position = this.$anchor.css("position") === "fixed";

        // The body never needs to have the o_tooltip_parent class. It is a
        // safe place to put the tip in the DOM at initialization and be able
        // to compute its dimensions and reposition it if required.
        await this.appendTo(document.body);
        if (this.isDestroyed()) {
            return new Promise(() => {});
        }
    },
    start() {
        this.$tooltip_overlay = this.$(".o_tooltip_overlay");
        this.$tooltip_content = this.$(".o_tooltip_content");
        this.init_width = this.$el.outerWidth();
        this.init_height = this.$el.outerHeight();
        this.$el.addClass('active');
        this.el.style.setProperty('width', `${this.info.width}px`, 'important');
        this.el.style.setProperty('height', 'auto', 'important');
        this.el.style.setProperty('transition', 'none', 'important');
        this.content_width = this.$el.outerWidth(true);
        this.content_height = this.$el.outerHeight(true);
        this.$tooltip_content.html(this.info.scrollContent);
        this.scrollContentWidth = this.$el.outerWidth(true);
        this.scrollContentHeight = this.$el.outerHeight(true);
        this.$el.removeClass('active');
        this.el.style.removeProperty('width');
        this.el.style.removeProperty('height');
        this.el.style.removeProperty('transition');
        this.$tooltip_content.html(this.info.content);
        this.$window = $(window);
        // Fix the content font size as it was used to compute the height and
        // width of the container.
        this.$tooltip_content[0].style.fontSize = getComputedStyle(this.$tooltip_content[0])['font-size'];

        this.$tooltip_content.css({
            width: "100%",
            height: "100%",
        });

        _.each(this.info.event_handlers, data => {
            this.$tooltip_content.on(data.event, data.selector, data.handler);
        });

        this._bind_anchor_events();
        this._updatePosition(true);

        this.$el.toggleClass('d-none', !!this.info.hidden);
        this.el.classList.add('o_tooltip_visible');
        core.bus.on("resize", this, _.debounce(function () {
            if (this.isDestroyed()) {
                // Because of the debounce, destroy() might have been called in the meantime.
                return;
            }
            if (this.tip_opened) {
                this._to_bubble_mode(true);
            } else {
                this._reposition();
            }
        }, 500));

        return this._super.apply(this, arguments);
    },
    destroy: function () {
        this._unbind_anchor_events();
        clearTimeout(this.timerIn);
        clearTimeout(this.timerOut);
        // clear this timeout so that we won't call _updatePosition after we
        // destroy the widget and leave an undesired bubble.
        clearTimeout(this._transitionEndTimer);

        // Do not remove the parent class if it contains other tooltips
        const _removeParentClass = $el => {
            if ($el.children(".o_tooltip").not(this.$el[0]).length === 0) {
                $el.removeClass("o_tooltip_parent");
            }
        };
        if (this.$el && this.$ideal_location) {
            _removeParentClass(this.$ideal_location);
        }
        if (this.$el && this.$furtherIdealLocation) {
            _removeParentClass(this.$furtherIdealLocation);
        }

        return this._super.apply(this, arguments);
    },
    /**
     * Updates the $anchor and $altAnchor the tip is attached to.
     * $altAnchor is an alternative trigger that can consume the step. The tip is
     * however only displayed on the $anchor.
     *
     * @param {jQuery} $anchor the node on which the tip should be placed
     * @param {jQuery} $altAnchor an alternative node that can consume the step
     */
    update: function ($anchor, $altAnchor) {
        // We unbind/rebind events on each update because we support widgets
        // detaching and re-attaching nodes to their DOM element without keeping
        // the initial event handlers, with said node being potential tip
        // anchors (e.g. FieldMonetary > input element).
        this._unbind_anchor_events();
        if (!$anchor.is(this.$anchor)) {
            this._setupAnchor($anchor, $altAnchor);
        }
        this._bind_anchor_events();
        if (!this.$el) {
            // Ideally this case should not happen but this is still possible,
            // as update may be called before the `start` method is called.
            // The `start` method is calling _updatePosition too anyway.
            return;
        }
        this._delegateEvents();
        this._updatePosition(true);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @return {boolean} true if tip is visible
     */
    isShown() {
        return this.el && !this.info.hidden;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Sets the $anchor and $altAnchor the tip is attached to.
     * $altAnchor is an alternative trigger that can consume the step. The tip is
     * however only displayed on the $anchor.
     *
     * @param {jQuery} $anchor the node on which the tip should be placed
     * @param {jQuery} $altAnchor an alternative node that can consume the step
     */
    _setupAnchor: function ($anchor, $altAnchor) {
        this.$anchor = $anchor;
        this.$altAnchor = $altAnchor;
        this.$ideal_location = this._get_ideal_location();
        this.$furtherIdealLocation = this._get_ideal_location(this.$ideal_location);
    },
    /**
     * Figures out which direction the tip should take and if it is at the
     * bottom or the top of the targeted element or if it's an indicator to
     * scroll. Relocates and repositions if necessary.
     *
     * @private
     * @param {boolean} [forceReposition=false]
     */
    _updatePosition: function (forceReposition = false) {
        if (this.info.hidden) {
            return;
        }
        if (this.isDestroyed()) {
            // TODO This should not be needed if the chain of events leading
            // here was fully cancelled by destroy().
            return;
        }
        let halfHeight = 0;
        if (this.initialPosition === 'right' || this.initialPosition === 'left') {
            halfHeight = this.$anchor.innerHeight() / 2;
        }

        const paddingTop = parseInt(this.$ideal_location.css('padding-top'));
        const topViewport = window.pageYOffset + paddingTop;
        const botViewport = window.pageYOffset + window.innerHeight;
        const topOffset = this.$anchor.offset().top;
        const botOffset = topOffset + this.$anchor.innerHeight();

        // Check if the viewport state change to know if we need to move the anchor of the tip.
        // up : the target element is above the current viewport
        // down : the target element is below the current viewport
        // in : the target element is in the current viewport
        let viewPortState = 'in';
        let position = this.info.position;
        if (botOffset - halfHeight < topViewport) {
            viewPortState = 'up';
            position = 'bottom';
        } else if (topOffset + halfHeight > botViewport) {
            viewPortState = 'down';
            position = 'top';
        } else {
            // Adjust the placement of the tip regarding its anchor depending
            // if we came from the bottom or the top.
            if (topOffset < topViewport + this.$el.innerHeight()) {
                position = halfHeight ? this.initialPosition : "bottom";
            } else if (botOffset > botViewport - this.$el.innerHeight()) {
                position = halfHeight ? this.initialPosition : "top";
            }
        }

        // If the direction or the anchor change : The tip position is updated.
        if (forceReposition || this.info.position !== position || this.viewPortState !== viewPortState) {
            this.$el.removeClass('top right bottom left').addClass(position);
            this.viewPortState = viewPortState;
            this.info.position = position;
            let $location;
            if (this.viewPortState === 'in') {
                this.$tooltip_content.html(this.info.content);
                $location = this.$ideal_location;
            } else {
                this.$tooltip_content.html(this.info.scrollContent);
                $location = this.$furtherIdealLocation;
            }
            // Update o_tooltip_parent class and tip DOM location. Note:
            // important to only remove/add the class when necessary to not
            // notify a DOM mutation which could retrigger this function.
            const $oldLocation = this.$el.parent();
            if (!this.tip_opened) {
                if (!$location.is($oldLocation)) {
                    $oldLocation.removeClass('o_tooltip_parent');
                    const cssPosition = $location.css("position");
                    if (cssPosition === "static" || cssPosition === "relative") {
                        $location.addClass("o_tooltip_parent");
                    }
                    this.$el.appendTo($location);
                }
                this._reposition();
            }
        }
    },
    _get_ideal_location: function ($anchor = this.$anchor) {
        var $location = this.info.location ? $(this.info.location) : $anchor;
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
            $location.hasClass('o_notebook_headers') ||
            $location.hasClass('o_forbidden_tooltip_parent') ||
            (
                (o === "visible" || o.includes("hidden")) && // Possible case where the overflow = "hidden auto"
                p !== "fixed" &&
                $location[0].tagName.toUpperCase() !== 'BODY'
            )
        );

        return $location;
    },
    _reposition: function () {
        this.$el.removeClass("o_animated");

        // Reverse left/right position if direction is right to left
        var appendAt = this.info.position;
        var rtlMap = {left: 'right', right: 'left'};
        if (rtlMap[appendAt] && _t.database.parameters.direction === 'rtl') {
            appendAt = rtlMap[appendAt];
        }

        // Get the correct tip's position depending of the tip's state
        let $parent = this.$ideal_location;
        if ($parent.is('html,body') && this.viewPortState !== "in") {
            this.el.style.setProperty('position', 'fixed', 'important');
        } else {
            this.el.style.removeProperty('position');
        }

        if (this.viewPortState === 'in') {
            this.$el.position({
                my: this._get_spaced_inverted_position(appendAt),
                at: appendAt,
                of: this.$anchor,
                collision: "none",
                using: props => {
                    const {top} = props;
                    let {left} = props;
                    const anchorEl = this.$anchor[0];
                    if (this.CENTER_ON_TEXT_TAGS.includes(anchorEl.nodeName) && anchorEl.hasChildNodes()) {
                        const textContainerWidth = anchorEl.getBoundingClientRect().width;
                        const textNode = anchorEl.firstChild;
                        const range = document.createRange();
                        range.selectNodeContents(textNode);
                        const textWidth = range.getBoundingClientRect().width;

                        const alignment = window.getComputedStyle(anchorEl).getPropertyValue('text-align');
                        const posVertical = (this.info.position === 'top' || this.info.position === 'bottom');
                        if (alignment === 'left') {
                            if (posVertical) {
                                left = left - textContainerWidth / 2 + textWidth / 2;
                            } else if (this.info.position === 'right') {
                                left = left - textContainerWidth + textWidth;
                            }
                        } else if (alignment === 'right') {
                            if (posVertical) {
                                left = left + textContainerWidth / 2 - textWidth / 2;
                            } else if (this.info.position === 'left') {
                                left = left + textContainerWidth - textWidth;
                            }
                        } else if (alignment === 'center') {
                            if (this.info.position === 'left') {
                                left = left + textContainerWidth / 2 - textWidth / 2;
                            } else if (this.info.position === 'right') {
                                left = left - textContainerWidth / 2 + textWidth / 2;
                            }
                        }
                    }
                    this.el.style.setProperty('top', `${top}px`, 'important');
                    this.el.style.setProperty('left', `${left}px`, 'important');
                },
            });
        } else {
            const paddingTop = parseInt($parent.css('padding-top'));
            const paddingLeft = parseInt($parent.css('padding-left'));
            const paddingRight = parseInt($parent.css('padding-right'));
            const topPosition = $parent[0].offsetTop;
            const center = (paddingLeft + paddingRight) + ((($parent[0].clientWidth - (paddingLeft + paddingRight)) / 2) - this.$el[0].offsetWidth / 2);
            let top;
            if (this.viewPortState === 'up') {
                top = topPosition + this.$el.innerHeight() + paddingTop;
            } else {
                top = topPosition + $parent.innerHeight() - this.$el.innerHeight() * 2;
            }
            this.el.style.setProperty('top', `${top}px`, 'important');
            this.el.style.setProperty('left', `${center}px`, 'important');
        }

        // Reverse overlay if direction is right to left
        var positionRight = _t.database.parameters.direction === 'rtl' ? "right" : "left";
        var positionLeft = _t.database.parameters.direction === 'rtl' ? "left" : "right";

        // get the offset position of this.$el
        // Couldn't use offset() or position() because their values are not the desired ones in all cases
        const offset = {top: this.$el[0].offsetTop, left: this.$el[0].offsetLeft};
        this.$tooltip_overlay.css({
            top: -Math.min((this.info.position === "bottom" ? this.info.space : this.info.overlay.y), offset.top),
            right: -Math.min((this.info.position === positionRight ? this.info.space : this.info.overlay.x), this.$window.width() - (offset.left + this.init_width)),
            bottom: -Math.min((this.info.position === "top" ? this.info.space : this.info.overlay.y), this.$window.height() - (offset.top + this.init_height)),
            left: -Math.min((this.info.position === positionLeft ? this.info.space : this.info.overlay.x), offset.left),
        });
        this.position = offset;

        this.$el.addClass("o_animated");
    },
    _bind_anchor_events: function () {
        // The consume_event taken for RunningTourActionHelper is the one of $anchor and not $altAnchor.
        this.consume_event = this.info.consumeEvent || Tip.getConsumeEventType(this.$anchor, this.info.run);
        this.$consumeEventAnchors = this._getAnchorAndCreateEvent(this.consume_event, this.$anchor);
        if (this.$altAnchor.length) {
            const consumeEvent  = this.info.consumeEvent || Tip.getConsumeEventType(this.$altAnchor, this.info.run);
            this.$consumeEventAnchors = this.$consumeEventAnchors.add(
                this._getAnchorAndCreateEvent(consumeEvent, this.$altAnchor)
            );
        }
        this.$anchor.on('mouseenter.anchor', () => this._to_info_mode());
        this.$anchor.on('mouseleave.anchor', () => this._to_bubble_mode());

        this.$scrolableElement = this.$ideal_location.is('html,body') ? $(window) : this.$ideal_location;
        this.$scrolableElement.on('scroll.Tip', () => this._onAncestorScroll());
    },
    /**
     * Gets the anchor corresponding to the provided arguments and attaches the
     * event to the $anchor in order to consume the step accordingly.
     *
     * @private
     * @param {String} consumeEvent
     * @param {jQuery} $anchor the node on which the tip should be placed
     * @return {jQuery}
     */
    _getAnchorAndCreateEvent: function(consumeEvent, $anchor) {
        let $consumeEventAnchors = $anchor;
        if (consumeEvent === "drag") {
            // jQuery-ui draggable triggers 'drag' events on the .ui-draggable element,
            // but the tip is attached to the .ui-draggable-handle element which may
            // be one of its children (or the element itself)
            $consumeEventAnchors = $anchor.closest('.ui-draggable');
        } else if (consumeEvent === "input" && !$anchor.is('textarea, input')) {
            $consumeEventAnchors = $anchor.closest("[contenteditable='true']");
        } else if (consumeEvent.includes('apply.daterangepicker')) {
            $consumeEventAnchors = $anchor.parent().children('.o_field_date_range');
        } else if (consumeEvent === "sort") {
            // when an element is dragged inside a sortable container (with classname
            // 'ui-sortable'), jQuery triggers the 'sort' event on the container
            $consumeEventAnchors = $anchor.closest('.ui-sortable');
        }
        $consumeEventAnchors.on(consumeEvent + ".anchor", (function (e) {
            if (e.type !== "mousedown" || e.which === 1) { // only left click
                if (this.info.consumeVisibleOnly && !this.isShown()) {
                    // Do not consume non-displayed tips.
                    return;
                }
                this.trigger("tip_consumed");
                this._unbind_anchor_events();
            }
        }).bind(this));
        return $consumeEventAnchors;
    },
    _unbind_anchor_events: function () {
        if (this.$anchor) {
            this.$anchor.off(".anchor");
        }
        if (this.$consumeEventAnchors) {
            this.$consumeEventAnchors.off(".anchor");
        }
        if (this.$scrolableElement) {
            this.$scrolableElement.off('.Tip');
        }
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

        // When this.$el doesn't have any parents, it means that the tip is no
        // longer in the DOM and so, it shouldn't be open. It happens when the
        // tip is opened after being destroyed.
        if (!this.$el.parent().length) {
            return;
        }

        if (this.$el.parent()[0] !== this.$el[0].ownerDocument.body) {
            this.$el.detach();
            this.el.style.setProperty('top', `${offset.top}px`, 'important');
            this.el.style.setProperty('left', `${offset.left}px`, 'important');
            this.$el.appendTo(this.$el[0].ownerDocument.body);
        }

        var mbLeft = 0;
        var mbTop = 0;
        var overflow = false;
        var posVertical = (this.info.position === "top" || this.info.position === "bottom");
        if (posVertical) {
            overflow = (offset.left + this.content_width + this.info.overlay.x > this.$window.width());
        } else {
            overflow = (offset.top + this.content_height + this.info.overlay.y > this.$window.height());
        }
        if (posVertical && overflow || this.info.position === "left" || (_t.database.parameters.direction === 'rtl' && this.info.position == "right")) {
            mbLeft -= (this.content_width - this.init_width);
        }
        if (!posVertical && overflow || this.info.position === "top") {
            mbTop -= (this.viewPortState === 'down') ? this.init_height - 5 : (this.content_height - this.init_height);
        }


        const [contentWidth, contentHeight] = this.viewPortState === 'in'
            ? [this.content_width, this.content_height]
            : [this.scrollContentWidth, this.scrollContentHeight];
        this.$el.toggleClass("inverse", overflow);
        this.$el.removeClass("o_animated").addClass("active");
        this.el.style.setProperty('width', `${contentWidth}px`, 'important');
        this.el.style.setProperty('height', `${contentHeight}px`, 'important');
        this.el.style.setProperty('margin-left', `${mbLeft}px`, 'important');
        this.el.style.setProperty('margin-top', `${mbTop}px`, 'important');

        this._transitionEndTimer = setTimeout(() => this._onTransitionEnd(), 400);
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
        this.el.style.setProperty('width', `${this.init_width}px`, 'important');
        this.el.style.setProperty('height', `${this.init_height}px`, 'important');
        this.el.style.setProperty('margin', '0', 'important');

        this._transitionEndTimer = setTimeout(() => this._onTransitionEnd(), 400);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onAncestorScroll: function () {
        if (this.tip_opened) {
            this._to_bubble_mode(true);
        } else {
            this._updatePosition(true);
        }
    },
    /**
     * @private
     */
    _onMouseEnter: function () {
        this._to_info_mode();
    },
    /**
     * @private
     */
    _onMouseLeave: function () {
        this._to_bubble_mode();
    },
    /**
     * On touch devices, closes the tip when clicked.
     *
     * Also stop propagation to avoid undesired behavior, such as the kanban
     * quick create closing when the user clicks on the tooltip.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onTipClicked: function (ev) {
        if (config.device.touch && this.tip_opened) {
            this._to_bubble_mode();
        }

        ev.stopPropagation();
    },
    /**
     * @private
     */
    _onTransitionEnd: function () {
        if (this._transitionEndTimer) {
            clearTimeout(this._transitionEndTimer);
            this._transitionEndTimer = undefined;
            if (!this.tip_opened) {
                this._updatePosition(true);
            }
        }
    },
});

/**
 * @static
 * @param {jQuery} $element
 * @param {string} [run] the run parameter of the tip (only strings are useful)
 */
Tip.getConsumeEventType = function ($element, run) {
    if ($element.has("input.o-autocomplete--input.o_input").length > 0) {
        // Components that utilizes the AutoComplete component is expected to
        // contain an input with the class o-autocomplete--input.
        // And when an option is selected, the component triggers
        // 'AutoComplete:OPTION_SELECTED' event.
        return 'AutoComplete:OPTION_SELECTED';
    } else if ($element.hasClass('o_field_many2one') || $element.hasClass('o_field_many2manytags')) {
        return 'autocompleteselect';
    } else if ($element.is("textarea") || $element.filter("input").is(function () {
        var type = $(this).attr("type");
        return !type || !!type.match(/^(email|number|password|search|tel|text|url)$/);
    })) {
        // FieldDateRange triggers a special event when using the widget
        if ($element.hasClass("o_field_date_range")) {
            return "apply.daterangepicker input";
        }
        if (config.device.isMobile &&
            $element.closest('.o_field_widget').is('.o_field_many2one, .o_field_many2many')) {
            return "click";
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
        if (run.indexOf("drag_and_drop_native") === 0 && $element.hasClass('o_record_draggable') || $element.closest('.o_record_draggable').length) {
            return 'mousedown';
        }
    }
    return "click";
};

return Tip;

});

```

## File: static\src\js\tour_manager.js

```javascript
odoo.define('web_tour.TourManager', function(require) {
"use strict";

var core = require('web.core');
var config = require('web.config');
var local_storage = require('web.local_storage');
var mixins = require('web.mixins');
var utils = require('web_tour.utils');
var TourStepUtils = require('web_tour.TourStepUtils');
var RunningTourActionHelper = require('web_tour.RunningTourActionHelper');
var ServicesMixin = require('web.ServicesMixin');
var session = require('web.session');
var Tip = require('web_tour.Tip');
const {Markup} = require('web.utils');
const { config: transitionConfig } = require("@web/core/transition");

var _t = core._t;
const { markup } = require("@odoo/owl");

var RUNNING_TOUR_TIMEOUT = 10000;

var get_step_key = utils.get_step_key;
var get_debugging_key = utils.get_debugging_key;
var get_running_key = utils.get_running_key;
var get_running_delay_key = utils.get_running_delay_key;
var get_first_visible_element = utils.get_first_visible_element;
var do_before_unload = utils.do_before_unload;
var get_jquery_element_from_selector = utils.get_jquery_element_from_selector;

return core.Class.extend(mixins.EventDispatcherMixin, ServicesMixin, {
    init: function(parent, consumed_tours, disabled = false) {
        mixins.EventDispatcherMixin.init.call(this);
        this.setParent(parent);

        this.$body = $('body');
        this.active_tooltips = {};
        this.tours = {};
        // remove the tours being debug from the list of consumed tours
        this.consumed_tours = (consumed_tours || []).filter(tourName => {
            return !local_storage.getItem(get_debugging_key(tourName));
        });
        this.disabled = disabled;
        this.running_tour = local_storage.getItem(get_running_key());
        if (this.running_tour) {
            // Transitions can cause DOM mutations which will cause the tour_service
            // MutationObserver to wait longer before proceeding to the next step
            // this can slow down tours
            transitionConfig.disabled = true;
        }
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
     * @param {string} [options.fadeout]
     *        Delay for rainbowman to disappear. 'fast' will make rainbowman dissapear, quickly,
     *        'medium', 'slow' and 'very_slow' will wait little longer before disappearing, no
     *        will keep rainbowman on screen until user clicks anywhere outside rainbowman
     * @param {boolean} [options.sequence=1000]
     *        priority sequence of the tour (lowest is first, tours with the same
     *        sequence will be executed in a non deterministic order).
     * @param {Promise} [options.wait_for]
     *        indicates when the tour can be started
     * @param {string|function} [options.rainbowManMessage]
              text or function returning the text displayed under the rainbowman
              at the end of the tour.
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
            name: options.saveAs || name,
            steps: steps,
            url: options.url,
            rainbowMan: options.rainbowMan === undefined ? true : !!options.rainbowMan,
            rainbowManMessage: options.rainbowManMessage,
            fadeout: options.fadeout || 'medium',
            sequence: options.sequence || 1000,
            test: options.test,
            wait_for: options.wait_for || Promise.resolve(),
        };
        if (options.skip_enabled) {
            tour.skip_link = Markup`<p><span class="o_skip_tour">${_t('Skip tour')}</span></p>`;
            tour.skip_handler = function (tip) {
                this._deactivate_tip(tip);
                this._consume_tour(name);
            };
        }
        this.tours[tour.name] = tour;
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
        const debuggingTour = local_storage.getItem(get_debugging_key(name));
        if (this.disabled && !this.running_tour && !debuggingTour) {
            this.consumed_tours.push(name);
        }

        if (tour.ready) return Promise.resolve();

        const tour_is_consumed = this._isTourConsumed(name);

        return tour.wait_for.then((function () {
            tour.current_step = parseInt(local_storage.getItem(get_step_key(name))) || 0;
            tour.steps = _.filter(tour.steps, (function (step) {
                return (!step.edition || step.edition === this.edition) &&
                    (step.mobile === undefined || step.mobile === config.device.isMobile);
            }).bind(this));

            if (tour_is_consumed || tour.current_step >= tour.steps.length) {
                local_storage.removeItem(get_step_key(name));
                tour.current_step = 0;
            }

            tour.ready = true;

            if (debuggingTour ||
                (do_update && (this.running_tour === name ||
                              (!this.running_tour && !tour.test && !tour_is_consumed)))) {
                this._to_next_step(name, 0);
            }
        }).bind(this));
    },
    /**
     * Resets the given tour to its initial step, and prevent it from being
     * marked as consumed at reload.
     *
     * @param {string} tourName
     */
    reset: function (tourName) {
        // remove it from the list of consumed tours
        const index = this.consumed_tours.indexOf(tourName);
        if (index >= 0) {
            this.consumed_tours.splice(index, 1);
        }
        // mark it as being debugged
        local_storage.setItem(get_debugging_key(tourName), true);
        // reset it to the first step
        const tour = this.tours[tourName];
        tour.current_step = 0;
        local_storage.removeItem(get_step_key(tourName));
        this._to_next_step(tourName, 0);
        // redirect to its starting point (or /web by default)
        window.location.href = window.location.origin + (tour.url || '/web');
    },
    run: async function (tour_name, step_delay) {
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

        tour_name = this.running_tour || tour_name;
        if (tour_name) {
            var tour = this.tours[tour_name];
            if (!tour || !tour.ready) return;

            let self = this;
            self._check_for_skipping_step(self.active_tooltips[tour_name], tour_name);

            if (this.running_tour && this.running_tour_timeout === undefined) {
                this._set_running_tour_timeout(this.running_tour, this.active_tooltips[this.running_tour]);
            }
            setTimeout(function () {
                self._check_for_tooltip(self.active_tooltips[tour_name], tour_name);
            });
        } else {
            const sortedTooltips = Object.keys(this.active_tooltips).sort(
                (a, b) => this.tours[a].sequence - this.tours[b].sequence
            );
            let visibleTip = false;
            for (const tourName of sortedTooltips) {
                var tip = this.active_tooltips[tourName];
                this._check_for_skipping_step(tip, tourName)
                tip.hidden = visibleTip;
                visibleTip = this._check_for_tooltip(tip, tourName) || visibleTip;
            }
        }
    },
    /**
     *  Check if the current step of a tour needs to be skipped. If so, skip the step and update
     *
     * @param {Object} step
     * @param {string} tour_name
     */
    _check_for_skipping_step: function (step, tour_name) {
        if (step && step.skip_trigger) {
            let $skip_trigger;
            $skip_trigger = get_jquery_element_from_selector(step.skip_trigger);
            let skipping = get_first_visible_element($skip_trigger).length;
            if (skipping) {
                this._to_next_step(tour_name);
                this.update(tour_name);
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
        this.$modal_displayed = $('.modal:visible').last();

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

        var $visible_alt_trigger = $();
        if (tip.alt_trigger) {
            var $alt_trigger;
            if (tip.in_modal !== false && this.$modal_displayed.length) {
                $alt_trigger = this.$modal_displayed.find(tip.alt_trigger);
            } else {
                $alt_trigger = get_jquery_element_from_selector(tip.alt_trigger);
            }
            $visible_alt_trigger = get_first_visible_element($alt_trigger);
        }

        var triggered = $visible_trigger.length && extra_trigger;
        if (triggered) {
            if (!tip.widget) {
                this._activate_tip(tip, tour_name, $visible_trigger, $visible_alt_trigger);
            } else {
                tip.widget.update($visible_trigger, $visible_alt_trigger);
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
    /**
     * Activates the provided tip for the provided tour, $anchor and $alt_trigger.
     * $alt_trigger is an alternative trigger that can consume the step. The tip is
     * however only displayed on the $anchor.
     *
     * @param {Object} tip
     * @param {String} tour_name
     * @param {jQuery} $anchor
     * @param {jQuery} $alt_trigger
     * @private
     */
    _activate_tip: function(tip, tour_name, $anchor, $alt_trigger) {
        var tour = this.tours[tour_name];
        var tip_info = tip;
        if (tour.skip_link) {
            tip_info = _.extend(_.omit(tip_info, 'content'), {
                content: Markup`${tip.content}${tour.skip_link}`,
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
        tip.widget.attach_to($anchor, $alt_trigger).then(this._to_next_running_step.bind(this, tip, tour_name));
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
    /**
     * @private
     * @param {string} tourName
     * @returns {boolean}
     */
    _isTourConsumed(tourName) {
        return this.consumed_tours.includes(tourName);
    },
    _consume_tour: function (tour_name, error) {
        delete this.active_tooltips[tour_name];
        //display rainbow at the end of any tour
        if (this.tours[tour_name].rainbowMan && this.running_tour !== tour_name &&
            this.tours[tour_name].current_step === this.tours[tour_name].steps.length) {
            let message = this.tours[tour_name].rainbowManMessage;
            if (message) {
                message = typeof message === 'function' ? message() : message;
            } else {
                message = markup(_t('<strong><b>Good job!</b> You went through all steps of this tour.</strong>'));
            }
            const fadeout = this.tours[tour_name].fadeout;
            core.bus.trigger('show-effect', {
                type: "rainbow_man",
                message,
                fadeout,
            });
        }
        this.tours[tour_name].current_step = 0;
        local_storage.removeItem(get_step_key(tour_name));
        local_storage.removeItem(get_debugging_key(tour_name));
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
                let documentHTML = document.documentElement.outerHTML;
                // Replace empty iframe tags with their content
                const iframeEls = Array.from(document.body.querySelectorAll('iframe.o_iframe'));
                if (iframeEls.length) {
                    const matches = documentHTML.match(/<iframe[^>]+class=["'][^"']*\bo_iframe\b[^>]+><\/iframe>/g);
                    for (let i = 0; i < matches.length; i++) {
                        const iframeWithContent = matches[i].replace(/></, `>${iframeEls[i].contentDocument.documentElement.outerHTML}<`);
                        documentHTML = documentHTML.replace(matches[i], `\n${iframeWithContent}\n`);
                    }
                }
                console.log(documentHTML);
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
            const anchorIsInDocument = tip.widget.$anchor[0].ownerDocument.contains(tip.widget.$anchor[0]);
            const uiIsBlocked = $('body').hasClass('o_ui_blocked');
            if (!anchorIsInDocument || uiIsBlocked) {
                // trigger is no longer in the DOM, or UI is now blocked, so run the same step again
                self._deactivate_tip(self.active_tooltips[tour_name]);
                self._to_next_step(tour_name, 0);
                self.update();
                return;
            }
            var action_helper = new RunningTourActionHelper(tip.widget);
            do_before_unload(self._consume_tip.bind(self, tip, tour_name));

            var tour = self.tours[tour_name];
            if (typeof tip.run === "function") {
                try {
                    tip.run.call(tip.widget, action_helper);
                } catch (e) {
                    console.error(`Tour ${tour_name} failed at step ${self._describeTip(tip)}: ${e.message}`);
                    throw e;
                }
            } else if (tip.run !== undefined) {
                var m = tip.run.match(/^([a-zA-Z0-9_]+) *(?:\(? *(.+?) *\)?)?$/);
                try {
                    action_helper[m[1]](m[2]);
                } catch (e) {
                    console.error(`Tour ${tour_name} failed at step ${self._describeTip(tip)}: ${e.message}`);
                    throw e;
                }
            } else if (tour.current_step === tour.steps.length - 1) {
                console.log('Tour %s: ignoring action (auto) of last step', tour_name);
            } else {
                action_helper.auto();
            }
        }
    },
    stepUtils: new TourStepUtils(this)
});
});

```

## File: static\src\js\tour_service.js

```javascript
odoo.define('web_tour.tour', function (require) {
"use strict";

var rootWidget = require('root.widget');
var rpc = require('web.rpc');
var session = require('web.session');
var TourManager = require('web_tour.TourManager');
const { device } = require('web.config');

const untrackedClassnames = ["o_tooltip", "o_tooltip_content", "o_tooltip_overlay"];

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
        const disabled = session.tour_disable || device.isMobile;
        var tour_manager = new TourManager(rootWidget, consumed_tours, disabled);

        // The tests can be loaded inside an iframe. The tour manager should
        // not run in that context, as it will already run in its parent
        // window.
        const isInIframe = window.frameElement && window.frameElement.classList.contains('o_iframe');
        if (isInIframe) {
            return tour_manager;
        }

        function _isTrackedNode(node) {
            if (node.classList) {
                return !untrackedClassnames
                    .some(className => node.classList.contains(className));
            }
            return true;
        }

        const classSplitRegex = /\s+/g;
        const tooltipParentRegex = /\bo_tooltip_parent\b/;
        let currentMutations = [];
        function _processMutations() {
            const hasTrackedMutation = currentMutations.some(mutation => {
                // First check if the mutation applied on an element we do not
                // track (like the tour tips themself).
                if (!_isTrackedNode(mutation.target)) {
                    return false;
                }

                if (mutation.type === 'characterData') {
                    return true;
                }

                if (mutation.type === 'childList') {
                    // If it is a modification to the DOM hierarchy, only
                    // consider the addition/removal of tracked nodes.
                    for (const nodes of [mutation.addedNodes, mutation.removedNodes]) {
                        for (const node of nodes) {
                            if (_isTrackedNode(node)) {
                                return true;
                            }
                        }
                    }
                    return false;
                } else if (mutation.type === 'attributes') {
                    // Get old and new value of the attribute. Note: as we
                    // compute the new value after a setTimeout, this might not
                    // actually be the new value for that particular mutation
                    // record but this is the one after all mutations. This is
                    // normally not an issue: e.g. "a" -> "a b" -> "a" will be
                    // seen as "a" -> "a" (not "a b") + "a b" -> "a" but we
                    // only need to detect *one* tracked mutation to know we
                    // have to update tips anyway.
                    const oldV = mutation.oldValue ? mutation.oldValue.trim() : '';
                    const newV = (mutation.target.getAttribute(mutation.attributeName) || '').trim();

                    // Not sure why but this occurs, especially on ID change
                    // (probably some strange jQuery behavior, see below).
                    // Also sometimes, a class is just considered changed while
                    // it just loses the spaces around the class names.
                    if (oldV === newV) {
                        return false;
                    }

                    if (mutation.attributeName === 'id') {
                        // Check if this is not an ID change done by jQuery for
                        // performance reasons.
                        return !(oldV.includes('sizzle') || newV.includes('sizzle'));
                    } else if (mutation.attributeName === 'class') {
                        // Check if the change is *only* about receiving or
                        // losing the 'o_tooltip_parent' class, which is linked
                        // to the tour service system. We have to check the
                        // potential addition of another class as we compute
                        // the new value after a setTimeout. So this case:
                        // 'a' -> 'a b' -> 'a b o_tooltip_parent' produces 2
                        // mutation records but will be seen here as
                        // 1) 'a' -> 'a b o_tooltip_parent'
                        // 2) 'a b' -> 'a b o_tooltip_parent'
                        const hadClass = tooltipParentRegex.test(oldV);
                        const newClasses = mutation.target.classList;
                        const hasClass = newClasses.contains('o_tooltip_parent');
                        return !(hadClass !== hasClass
                            && Math.abs(oldV.split(classSplitRegex).length - newClasses.length) === 1);
                    }
                }

                return true;
            });

            // Either all the mutations have been ignored or one was detected as
            // tracked and will trigger a tour manager update.
            currentMutations = [];

            // Update the tour manager if required.
            if (hasTrackedMutation) {
                tour_manager.update();
            }
        }

        // Use a MutationObserver to detect DOM changes. When a mutation occurs,
        // only add it to the list of mutations to process and delay the
        // mutation processing. We have to record them all and not in a
        // debounced way otherwise we may ignore tracked ones in a serie of
        // 10 tracked mutations followed by an untracked one. Most of them
        // will trigger a tip check anyway so, most of the time, processing the
        // first ones will be enough to ensure that a tip update has to be done.
        let mutationTimer;
        const observer = new MutationObserver(mutations => {
            clearTimeout(mutationTimer);
            currentMutations = currentMutations.concat(mutations);
            mutationTimer = setTimeout(() => _processMutations(), 750);
        });

        // Now that the observer is configured, we have to start it when needed.
        const observerOptions = {
            attributes: true,
            childList: true,
            subtree: true,
            attributeOldValue: true,
            characterData: true,
        };

        var start_service = (function () {
            return function (observe) {
                return new Promise(function (resolve, reject) {
                    tour_manager._register_all(observe).then(function () {
                        if (observe) {
                            observer.observe(document.body, observerOptions);

                            // If an iframe is added during the tour, its DOM
                            // mutations should also be observed to update the
                            // tour manager.
                            const findIframe = mutations => {
                                for (const mutation of mutations) {
                                    for (const addedNode of Array.from(mutation.addedNodes)) {
                                        if (addedNode.nodeType === Node.ELEMENT_NODE) {
                                            if (addedNode.classList.contains('o_iframe')) {
                                                return addedNode;
                                            }
                                            const iframeChildEl = addedNode.querySelector('.o_iframe');
                                            if (iframeChildEl) {
                                                return iframeChildEl;
                                            }
                                        }
                                    }
                                }
                            };
                            const iframeObserver = new MutationObserver(mutations => {
                                const iframeEl = findIframe(mutations);
                                if (iframeEl) {
                                    iframeEl.addEventListener('load', () => {
                                        observer.observe(iframeEl.contentDocument, observerOptions);
                                    });
                                    // If the iframe was added without a src,
                                    // its load event was immediately fired and
                                    // will not fire again unless another src is
                                    // set. Unfortunately, the case of this
                                    // happening and the iframe content being
                                    // altered programmaticaly may happen.
                                    // (E.g. at the moment this was written,
                                    // the mass mailing editor iframe is added
                                    // without src and its content rewritten
                                    // immediately afterwards).
                                    if (!iframeEl.src) {
                                        observer.observe(iframeEl.contentDocument, observerOptions);
                                    }
                                }
                            });
                            iframeObserver.observe(document.body, { childList: true, subtree: true });
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

## File: static\src\js\tour_step_utils.js

```javascript
odoo.define('web_tour.TourStepUtils', function (require) {
'use strict';

const {_t, Class} = require('web.core');
const {Markup} = require('web.utils');

return Class.extend({
    _getHelpMessage: (functionName, ...args) => `Generated by function tour utils ${functionName}(${args.join(', ')})`,

    addDebugHelp: helpMessage => step => {
        if (typeof step.debugHelp === 'string') {
            step.debugHelp = step.debugHelp + '\n' + helpMessage;
        } else {
            step.debugHelp = helpMessage;
        }
        return step;
    },

    editionEnterpriseModifier(step) {
        step.edition = 'enterprise';
        return step;
    },

    mobileModifier(step) {
        step.mobile = true;
        return step;
    },

    showAppsMenuItem() {
        return {
            edition: 'community',
            trigger: '.o_navbar_apps_menu button',
            auto: true,
            position: 'bottom',
        };
    },

    toggleHomeMenu() {
        return {
            edition: 'enterprise',
            trigger: '.o_main_navbar .o_menu_toggle',
            content: Markup(_t('Click on the <i>Home icon</i> to navigate across apps.')),
            position: 'bottom',
        };
    },

    autoExpandMoreButtons(extra_trigger) {
        return {
            trigger: '.oe_button_box',
            extra_trigger: extra_trigger,
            auto: true,
            run: actions => {
                const $more = $('.oe_button_box .o_button_more');
                if ($more.length) {
                    actions.click($more);
                }
            },
        };
    },

    goBackBreadcrumbsMobile(description, ...extraTrigger) {
        return extraTrigger.map(element => ({
            mobile: true,
            trigger: '.breadcrumb-item.o_back_button',
            extra_trigger: element,
            content: description,
            position: 'bottom',
            debugHelp: this._getHelpMessage('goBackBreadcrumbsMobile', description, ...extraTrigger),
        }));
    },

    goToAppSteps(dataMenuXmlid, description) {
        return [
            this.showAppsMenuItem(),
            {
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                position: 'right',
                edition: 'community',
            },
            {
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                position: 'bottom',
                edition: 'enterprise',
            },
        ].map(this.addDebugHelp(this._getHelpMessage('goToApp', dataMenuXmlid, description)));
    },

    openBuggerMenu(extraTrigger) {
        return {
            mobile: true,
            trigger: '.o_mobile_menu_toggle',
            extra_trigger: extraTrigger,
            content: _t('Open bugger menu.'),
            position: 'bottom',
            debugHelp: this._getHelpMessage('openBuggerMenu', extraTrigger),
        };
    },

    statusbarButtonsSteps(innerTextButton, description, extraTrigger) {
        return [
            {
                mobile: true,
                auto: true,
                trigger: '.o_statusbar_buttons',
                extra_trigger: extraTrigger,
                run: actions => {
                    const $action = $('.o_statusbar_buttons .btn.dropdown-toggle:contains(Action)');
                    if ($action.length) {
                        actions.click($action);
                    }
                },
            }, {
                trigger: `.o_statusbar_buttons button:enabled:contains('${innerTextButton}')`,
                content: description,
                position: 'bottom',
            },
        ].map(this.addDebugHelp(this._getHelpMessage('statusbarButtonsSteps', innerTextButton, description, extraTrigger)));
    },

    simulateEnterKeyboardInSearchModal() {
        return {
            mobile: true,
            trigger: '.o_searchview_input',
            extra_trigger: '.modal:not(.o_inactive_modal) .dropdown-menu.o_searchview_autocomplete',
            position: 'bottom',
            run: action => {
                const keyEventEnter = new KeyboardEvent('keydown', {
                    bubbles: true,
                    cancelable: true,
                    key: 'Enter',
                    code: 'Enter',
                    which: 13,
                    keyCode: 13,
                });
                action.tip_widget.$anchor[0].dispatchEvent(keyEventEnter);
            },
            debugHelp: this._getHelpMessage('simulateEnterKeyboardInSearchModal'),
        };
    },

    mobileKanbanSearchMany2X(modalTitle, valueSearched) {
        return [
            {
                mobile: true,
                trigger: '.o_searchview_input',
                extra_trigger: `.modal:not(.o_inactive_modal) .modal-title:contains('${modalTitle}')`,
                position: 'bottom',
                run: `text ${valueSearched}`,
            },
            this.simulateEnterKeyboardInSearchModal(),
            {
                mobile: true,
                trigger: `.o_kanban_record .o_kanban_record_title :contains('${valueSearched}')`,
                position: 'bottom',
            },
        ].map(this.addDebugHelp(this._getHelpMessage('mobileKanbanSearchMany2X', modalTitle, valueSearched)));
    },
    /**
     * Utility steps to save a form and wait for the save to complete
     *
     * @param {object} [options]
     * @param {string} [options.content]
     * @param {string} [options.extra_trigger] additional save-condition selector
     */
    saveForm(options = {}) {
        return [{
            content: options.content || "save form",
            trigger: ".o_form_button_save",
            extra_trigger: options.extra_trigger,
            run: "click",
            auto: true,
        }, {
            content: "wait for save completion",
            trigger: '.o_form_readonly, .o_form_saved',
            run() {},
            auto: true,
        }];
    },
    /**
     * Utility steps to cancel a form creation or edition.
     *
     * Supports creation/edition from either a form or a list view (so checks
     * for both states).
     */
    discardForm(options = {}) {
        return [{
            content: options.content || "exit the form",
            trigger: ".o_form_button_cancel",
            extra_trigger: options.extra_trigger,
            run: "click",
            auto: true,
        }, {
            content: "wait for cancellation to complete",
            trigger: ".o_list_view, .o_form_view > div > div > .o_form_readonly, .o_form_view > div > div > .o_form_saved",
            run() {},
            auto: true,
        }];
    }
});
});

```

## File: static\src\js\tour_utils.js

```javascript
odoo.define('web_tour.utils', function(require) {
"use strict";

const { _legacyIsVisible } = require("@web/core/utils/ui");

function get_step_key(name) {
    return 'tour_' + name + '_step';
}

function get_running_key() {
    return 'running_tour';
}

function get_debugging_key(name) {
    return `debugging_tour_${name}`;
}

function get_running_delay_key() {
    return get_running_key() + "_delay";
}

function get_first_visible_element($elements) {
    for (var i = 0 ; i < $elements.length ; i++) {
        var $i = $elements.eq(i);
        if (_legacyIsVisible($i[0])) {
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
    const iframeSplit = _.isString(selector) && selector.match(/(.*\biframe[^ ]*)(.*)/);
    if (iframeSplit && iframeSplit[2]) {
        var $iframe = $(`${iframeSplit[1]}:not(.o_ignore_in_tour)`);
        if ($iframe.is('[is-ready="false"]')) {
            return $();
        }
        var $el = $iframe.contents()
            .find(iframeSplit[2]);
        $el.iframeContainer = $iframe[0];
        return $el;
    } else {
        return $(selector);
    }
}


return {
    get_debugging_key: get_debugging_key,
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

## File: static\src\services\tour_service.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import tourManager from "web_tour.tour";

export const tourService = {
    start() {
        /**
         * @private
         * @returns {Object} All the tours as a map
         */
        function _getAllTourMap() {
            return tourManager.tours;
        }

        /**
         * @private
         * @returns {Object} All the active tours as a map
         */
        function _getActiveTourMap() {
            return Object.fromEntries(
                Object.entries(_getAllTourMap()).filter(
                    ([key, value]) => !tourManager.consumed_tours.includes(key)
                )
            );
        }

        /**
         * @private
         * @returns {Array} Takes an Object (map) of tours and returns all the values
         */
        function _fromTourMapToArray(tourMap) {
            return Object.values(tourMap).sort((t1, t2) => {
                return t1.sequence - t2.sequence || (t1.name < t2.name ? -1 : 1);
            });
        }

        /**
         * @returns {Array} All the tours
         */
        function getAllTours() {
            return _fromTourMapToArray(_getAllTourMap());
        }

        /**
         * @returns {Array} All the active tours
         */
        function getActiveTours() {
            return _fromTourMapToArray(_getActiveTourMap());
        }

        /**
         * @returns {Array} The onboarding tours
         */
        function getOnboardingTours() {
            return getAllTours().filter((t) => !t.test);
        }

        /**
         * @returns {Array} The testing tours
         */
        function getTestingTours() {
            return getAllTours().filter((t) => t.test);
        }

        /**
         * @param {string} tourName
         * Run a tour
         */
        function run(tourName) {
            return tourManager.run(tourName);
        }

        /**
         * @param {string} tourName
         * Reset a tour
         */
        function reset(tourName) {
            return tourManager.reset(tourName);
        }

        return {
            getAllTours,
            getActiveTours,
            getOnboardingTours,
            getTestingTours,
            run,
            reset,
        };
    },
};

registry.category("services").add("tour", tourService);

```

## File: static\src\xml\tip.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="Tip" t-attf-class="o_tooltip #{widget.info.position} #{widget.is_anchor_fixed_position ? 'o_tooltip_fixed' : ''}">
        <div class="o_tooltip_overlay"/>
        <div class="o_tooltip_content">
            <t t-out="widget.info.content"/>
        </div>
    </div>
</templates>

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

