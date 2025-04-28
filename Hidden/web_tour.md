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
        'web.assets_backend': [
            'web_tour/static/src/**/*',
        ],
        'web.assets_frontend': [
            'web_tour/static/src/tour_pointer/**/*',
            'web_tour/static/src/tour_service/**/*',
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
access_web_tour_tour,access_web_tour_tour,model_web_tour_tour,base.group_user,1,0,0,0

```

## File: security\ir.rule.csv

```csv
"id","model_id/id","name","active","perm_create","perm_unlink","perm_read","perm_write","domain_force"
"own_tours","model_web_tour_tour","own tours","True","True","True","True","True","[('user_id','=', user.id)]"

```

## File: static\src\debug\debug_manager.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { browser } from "@web/core/browser/browser";
import { registry } from "@web/core/registry";
import ToursDialog from "@web_tour/debug/tour_dialog_component";
import { tourState } from "../tour_service/tour_state";

export function disableTours({ env }) {
    if (!env.services.user.isSystem) {
        return null;
    }
    const activeTourNames = tourState.getActiveTourNames();
    if (activeTourNames.length === 0) {
        return null;
    }
    return {
        type: "item",
        description: _t("Disable Tours"),
        callback: async () => {
            await env.services.orm.call("web_tour.tour", "consume", [activeTourNames]);
            for (const tourName of activeTourNames) {
                tourState.clear(tourName);
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
        description: _t("Start Tour"),
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
import { _t } from "@web/core/l10n/translation";

import { Component } from "@odoo/owl";

export default class ToursDialog extends Component {
    setup() {
        this.title = _t("Tours");
        this.tourService = useService("tour_service");
        this.onboardingTours = this.tourService.getSortedTours().filter(tour => !tour.test);
        this.testingTours = this.tourService.getSortedTours().filter(tour => tour.test);
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
        this.tourService.startTour(ev.target.dataset.name, { mode: 'manual' });
        this.props.close();
    }
    /**
     * Starts the given tour in test mode.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onTestTour(ev) {
        this.tourService.startTour(ev.target.dataset.name, { mode: 'auto', stepDelay: 500, showPointerDuration: 250 });
        this.props.close();
    }
}
ToursDialog.template = "web_tour.ToursDialog";
ToursDialog.components = { Dialog };

```

## File: static\src\debug\tour_dialog_component.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="web_tour.ToursDialog">
    <Dialog title="title">
        <t t-call="web_tour.ToursDialog.Table">
            <t t-set="caption">Onboarding tours</t>
            <t t-set="tours" t-value="onboardingTours"/>
        </t>
        <t t-if="testingTours.length" t-call="web_tour.ToursDialog.Table">
            <t t-set="caption">Testing tours</t>
            <t t-set="tours" t-value="testingTours"/>
        </t>
        <t t-set-slot="footer">
            <button class="btn btn-primary o-default-button" t-on-click="() => props.close()">Close</button>
        </t>
    </Dialog>
</t>

<t t-name="web_tour.ToursDialog.Table">
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

## File: static\src\tour_pointer\tour_pointer.js

```javascript
/** @odoo-module **/

import { Component, useEffect, useRef } from "@odoo/owl";
import { usePosition } from "@web/core/position_hook";

/**
 * @typedef {import("../tour_service/tour_pointer_state").TourPointerState} TourPointerState
 *
 * @typedef TourPointerProps
 * @property {TourPointerState} pointerState
 * @property {boolean} bounce
 */

/** @extends {Component<TourPointerProps, any>} */
export class TourPointer extends Component {
    static props = {
        pointerState: {
            type: Object,
            shape: {
                anchor: { type: HTMLElement, optional: true },
                content: { type: String, optional: true },
                isOpen: { type: Boolean, optional: true },
                isVisible: { type: Boolean, optional: true },
                onClick: { type: [Function, { value: null }], optional: true },
                onMouseEnter: { type: [Function, { value: null }], optional: true },
                onMouseLeave: { type: [Function, { value: null }], optional: true },
                position: {
                    type: [
                        { value: "left" },
                        { value: "right" },
                        { value: "top" },
                        { value: "bottom" },
                    ],
                    optional: true,
                },
                rev: { type: Number, optional: true },
            },
        },
        bounce: { type: Boolean, optional: true },
    };

    static defaultProps = {
        bounce: true,
    };

    static template = "web_tour.TourPointer";
    static width = 28; // in pixels
    static height = 28; // in pixels

    setup() {
        const positionOptions = {
            margin: 6,
            onPositioned: (pointer, position) => {
                const popperRect = pointer.getBoundingClientRect();
                const { top, left, direction } = position;
                if (direction === "top") {
                    // position from the bottom instead of the top as it is needed
                    // to ensure the expand animation is properly done
                    pointer.style.bottom = `${window.innerHeight - top - popperRect.height}px`;
                    pointer.style.removeProperty("top");
                } else if (direction === "left") {
                    // position from the right instead of the left as it is needed
                    // to ensure the expand animation is properly done
                    pointer.style.right = `${window.innerWidth - left - popperRect.width}px`;
                    pointer.style.removeProperty("left");
                }
            },
        };
        Object.defineProperty(positionOptions, "position", { get: () => this.position, enumerable: true });
        const position = usePosition("pointer", () => this.props.pointerState.anchor, positionOptions);
        const rootRef = useRef("pointer");
        /** @type {DOMREct | null} */
        let dimensions = null;
        let lastMeasuredContent = null;
        let lastOpenState = this.isOpen;
        let lastAnchor;
        let [anchorX, anchorY] = [0, 0];
        useEffect(() => {
            const { el: pointer } = rootRef;
            if (pointer) {
                const hasContentChanged = lastMeasuredContent !== this.content;
                const hasOpenStateChanged = lastOpenState !== this.isOpen;
                lastOpenState = this.isOpen;

                // Content changed: we must re-measure the dimensions of the text.
                if (hasContentChanged) {
                    lastMeasuredContent = this.content;
                    pointer.style.removeProperty("width");
                    pointer.style.removeProperty("height");
                    dimensions = pointer.getBoundingClientRect();
                }

                // If the content or the "is open" state changed: we must apply
                // new width and height properties
                if (hasContentChanged || hasOpenStateChanged) {
                    const [width, height] = this.isOpen
                        ? [dimensions.width, dimensions.height]
                        : [this.constructor.width, this.constructor.height];
                    if (this.isOpen) {
                        pointer.style.removeProperty("transition");
                    } else {
                        // No transition if switching from open to closed
                        pointer.style.setProperty("transition", "none");
                    }
                    pointer.style.setProperty("width", `${width}px`);
                    pointer.style.setProperty("height", `${height}px`);
                }

                if (!this.isOpen) {
                    const { anchor } = this.props.pointerState;
                    if (anchor === lastAnchor) {
                        const { x, y, width } = anchor.getBoundingClientRect();
                        const [lastAnchorX, lastAnchorY] = [anchorX, anchorY];
                        [anchorX, anchorY] = [x, y];
                        // Let's just say that the anchor is static if it moved less than 1px.
                        const delta = Math.sqrt(
                            Math.pow(x - lastAnchorX, 2) + Math.pow(y - lastAnchorY, 2)
                        );
                        if (delta < 1) {
                            position.lock();
                            return;
                        }
                        const wouldOverflow = window.innerWidth - x - width / 2 < dimensions?.width;
                        pointer.classList.toggle("o_expand_left", wouldOverflow);
                    }
                    lastAnchor = anchor;
                    pointer.style.bottom = "";
                    pointer.style.right = "";
                    position.unlock();
                }
            } else {
                lastMeasuredContent = null;
                lastOpenState = false;
                lastAnchor = null;
                dimensions = null;
            }
        });
    }

    get content() {
        return this.props.pointerState.content || "";
    }

    get isOpen() {
        return this.props.pointerState.isOpen;
    }

    get position() {
        return this.props.pointerState.position || "top";
    }
}

```

## File: static\src\tour_pointer\tour_pointer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="web_tour.TourPointer">
        <div
            t-if="props.pointerState.isVisible"
            t-ref="pointer"
            t-attf-class="
                o_tour_pointer
                o_{{ position }}
                {{ isOpen ? 'o_open' : (props.bounce ? 'o_bouncing' : '') }}
                {{ props.pointerState.onClick ? 'cursor-pointer' : '' }}
            "
            t-attf-style="
                --TourPointer__width: {{ constructor.width }}px;
                --TourPointer__height: {{ constructor.height }}px;
            "
            t-on-mouseenter="props.pointerState.onMouseEnter or (() => {})"
            t-on-mouseleave="props.pointerState.onMouseLeave or (() => {})"
            t-on-click="props.pointerState.onClick or (() => {})"
        >
            <div class="o_tour_pointer_tip position-absolute" />
            <div
                class="o_tour_pointer_content rounded overflow-hidden px-3 py-2 w-100 h-100 position-relative"
                t-att-class="{ invisible: content and !isOpen }"
            >
                <t t-out="content" />
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\tour_service\tour_compilers.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";
import { debounce } from "@web/core/utils/timing";
import { isVisible } from "@web/core/utils/ui";
import { tourState } from "./tour_state";
import {
    callWithUnloadCheck,
    getConsumeEventType,
    getFirstVisibleElement,
    getJQueryElementFromSelector,
    getScrollParent,
    RunningTourActionHelper,
} from "./tour_utils";

/**
 * @typedef {import("@web/core/macro").MacroDescriptor} MacroDescriptor
 *
 * @typedef {import("../tour_service/tour_pointer_state").TourPointerState} TourPointerState
 *
 * @typedef {import("./tour_service").TourStep} TourStep
 *
 * @typedef {(stepIndex: number, step: TourStep, options: TourCompilerOptions) => MacroDescriptor[]} TourStepCompiler
 *
 * @typedef TourCompilerOptions
 * @property {Tour} tour
 * @property {number} stepDelay
 * @property {keepWatchBrowser} boolean
 * @property {showPointerDuration} number
 * @property {*} pointer - used for controlling the pointer of the tour
 */

/**
 * @param {string} selector - any valid jquery selector
 * @param {boolean} inModal
 * @param {string|undefined} shadowDOM - selector of the shadow root host
 * @returns {Element | undefined}
 */
function findTrigger(selector, inModal, shadowDOM) {
    const $target = $(shadowDOM ? document.querySelector(shadowDOM)?.shadowRoot : document);
    const $visibleModal = $target.find(".modal:visible").last();
    let $el;
    if (inModal !== false && $visibleModal.length) {
        $el = $visibleModal.find(selector);
    } else {
        $el = getJQueryElementFromSelector(selector, $target);
    }
    return getFirstVisibleElement($el).get(0);
}

/**
 * @param {string|undefined} shadowDOM - selector of the shadow root host
 */
function findExtraTrigger(selector, shadowDOM) {
    const $target = $(shadowDOM ? document.querySelector(shadowDOM)?.shadowRoot : document);
    const $el = getJQueryElementFromSelector(selector, $target);
    return getFirstVisibleElement($el).get(0);
}

function findStepTriggers(step) {
    const triggerEl = findTrigger(step.trigger, step.in_modal, step.shadow_dom);
    const altEl = findTrigger(step.alt_trigger, step.in_modal, step.shadow_dom);
    const skipEl = findTrigger(step.skip_trigger, step.in_modal, step.shadow_dom);

    // `extraTriggerOkay` should be true when `step.extra_trigger` is undefined.
    // No need for it to be in the modal.
    const extraTriggerOkay = step.extra_trigger
        ? findExtraTrigger(step.extra_trigger, step.shadow_dom)
        : true;

    return { triggerEl, altEl, extraTriggerOkay, skipEl };
}

/**
 * @param {TourStep} step
 */
function describeStep(step) {
    return step.content ? `${step.content} (trigger: ${step.trigger})` : step.trigger;
}

/**
 * @param {TourStep} step
 */
function describeFailedStepSimple(step, tour) {
    return `Tour ${tour.name} failed at step ${describeStep(step)}`;
}

/**
 * @param {TourStep} step
 * @param {Tour} tour
 */
function describeFailedStepDetailed(step, tour) {
    const offset = 3;
    const stepIndex = tour.steps.findIndex((s) => s === step);
    const start = stepIndex - offset >= 0 ? stepIndex - offset : 0;
    const end =
        stepIndex + offset + 1 <= tour.steps.length ? stepIndex + offset + 1 : tour.steps.length;
    let result = "";
    for (let i = start; i < end; i++) {
        const highlight = i === stepIndex;
        const stepString = JSON.stringify(
            tour.steps[i],
            (_key, value) => {
                if (typeof value === "function") {
                    return "[function]";
                } else {
                    return value;
                }
            },
            2
        );
        result += `\n${highlight ? "----- FAILING STEP -----\n" : ""}${stepString},${
            highlight ? "\n-----------------------" : ""
        }`;
    }
    return `${describeFailedStepSimple(step, tour)}\n\n${result.trim()}`;
}

/**
 * Returns the element that will be used in listening to the `consumeEvent`.
 * @param {HTMLElement} el
 * @param {string} consumeEvent
 */
function getAnchorEl(el, consumeEvent) {
    if (consumeEvent === "drag") {
        // jQuery-ui draggable triggers 'drag' events on the .ui-draggable element,
        // but the tip is attached to the .ui-draggable-handle element which may
        // be one of its children (or the element itself)
        return el.closest(".ui-draggable, .o_draggable");
    }
    if (consumeEvent === "input" && !["textarea", "input"].includes(el.tagName.toLowerCase())) {
        return el.closest("[contenteditable='true']");
    }
    if (consumeEvent === "sort") {
        // when an element is dragged inside a sortable container (with classname
        // 'ui-sortable'), jQuery triggers the 'sort' event on the container
        return el.closest(".ui-sortable, .o_sortable");
    }
    return el;
}

/**
 * IMPROVEMENT: Consider transitioning (moving) elements?
 * @param {Element} el
 * @param {TourStep} step
 */
function canContinue(el, step) {
    const rootNode = el.getRootNode();
    const isInDoc =
        rootNode instanceof ShadowRoot
            ? el.ownerDocument.contains(rootNode.host)
            : el.ownerDocument.contains(el);
    const isElement = el instanceof el.ownerDocument.defaultView.Element || el instanceof Element;
    const isBlocked = document.body.classList.contains("o_ui_blocked") || document.querySelector(".o_blockUI");
    return (
        isInDoc &&
        isElement &&
        !isBlocked &&
        (!step.allowInvisible ? isVisible(el) : true) &&
        (!el.disabled || step.isCheck)
    );
}

/**
 * @param {Object} params
 * @param {HTMLElement} params.anchorEl
 * @param {string} params.consumeEvent
 * @param {() => void} params.onMouseEnter
 * @param {() => void} params.onMouseLeave
 * @param {(ev: Event) => any} params.onScroll
 * @param {(ev: Event) => any} params.onConsume
 */
function setupListeners({
    anchorEl,
    consumeEvent,
    onMouseEnter,
    onMouseLeave,
    onScroll,
    onConsume,
}) {
    anchorEl.addEventListener(consumeEvent, onConsume);
    anchorEl.addEventListener("mouseenter", onMouseEnter);
    anchorEl.addEventListener("mouseleave", onMouseLeave);

    const cleanups = [
        () => {
            anchorEl.removeEventListener(consumeEvent, onConsume);
            anchorEl.removeEventListener("mouseenter", onMouseEnter);
            anchorEl.removeEventListener("mouseleave", onMouseLeave);
        },
    ];

    const scrollEl = getScrollParent(anchorEl);
    if (scrollEl) {
        const debouncedOnScroll = debounce(onScroll, 50);
        scrollEl.addEventListener("scroll", debouncedOnScroll);
        cleanups.push(() => scrollEl.removeEventListener("scroll", debouncedOnScroll));
    }

    return () => {
        while (cleanups.length) {
            cleanups.pop()();
        }
    };
}

/** @type {TourStepCompiler} */
export function compileStepManual(stepIndex, step, options) {
    const { tour, pointer, onStepConsummed } = options;
    let proceedWith = null;
    let removeListeners = () => {};

    return [
        {
            action: () => console.log(step.trigger),
        },
        {
            trigger: () => {
                removeListeners();

                if (proceedWith) {
                    return proceedWith;
                }

                const { triggerEl, altEl, extraTriggerOkay, skipEl } = findStepTriggers(step);

                if (skipEl) {
                    return skipEl;
                }

                const stepEl = extraTriggerOkay && (triggerEl || altEl);

                if (stepEl && canContinue(stepEl, step)) {
                    const consumeEvent = step.consumeEvent || getConsumeEventType(stepEl, step.run);
                    const anchorEl = getAnchorEl(stepEl, consumeEvent);
                    const debouncedToggleOpen = debounce(pointer.showContent, 50, true);

                    const updatePointer = () => {
                        pointer.setState({
                            onMouseEnter: () => debouncedToggleOpen(true),
                            onMouseLeave: () => debouncedToggleOpen(false),
                        });
                        pointer.pointTo(anchorEl, step);
                    };

                    removeListeners = setupListeners({
                        anchorEl,
                        consumeEvent,
                        onMouseEnter: () => pointer.showContent(true),
                        onMouseLeave: () => pointer.showContent(false),
                        onScroll: updatePointer,
                        onConsume: () => {
                            proceedWith = stepEl;
                            pointer.hide();
                        },
                    });

                    updatePointer();
                } else {
                    pointer.hide();
                }
            },
            action: () => {
                tourState.set(tour.name, "currentIndex", stepIndex + 1);
                pointer.hide();
                proceedWith = null;
                onStepConsummed(tour, step);
            },
        },
    ];
}

let tourTimeout;

/** @type {TourStepCompiler} */
export function compileStepAuto(stepIndex, step, options) {
    const { tour, pointer, stepDelay, keepWatchBrowser, showPointerDuration, onStepConsummed } = options;
    let skipAction = false;
    return [
        {
            action: async () => {
                // This delay is important for making the current set of tour tests pass.
                // IMPROVEMENT: Find a way to remove this delay.
                await new Promise(resolve => requestAnimationFrame(resolve))
            },
        },
        {
            action: async () => {
                skipAction = false;
                console.log(`Tour ${tour.name} on step: '${describeStep(step)}'`);
                if (!keepWatchBrowser) {
                    browser.clearTimeout(tourTimeout);
                    tourTimeout = browser.setTimeout(() => {
                        // The logged text shows the relative position of the failed step.
                        // Useful for finding the failed step.
                        console.warn(describeFailedStepDetailed(step, tour));
                        // console.error notifies the test runner that the tour failed.
                        console.error(describeFailedStepSimple(step, tour));
                    }, (step.timeout || 10000) + stepDelay);
                }
                await new Promise((resolve) => browser.setTimeout(resolve, stepDelay));
            },
        },
        {
            trigger: () => {
                const { triggerEl, altEl, extraTriggerOkay, skipEl } = findStepTriggers(step);

                let stepEl = extraTriggerOkay && (triggerEl || altEl);

                if (skipEl) {
                    skipAction = true;
                    stepEl = skipEl;
                }

                if (!stepEl) {
                    return false;
                }

                return canContinue(stepEl, step) && stepEl;
            },
            action: async (stepEl) => {
                tourState.set(tour.name, "currentIndex", stepIndex + 1);

                if (skipAction) {
                    return;
                }

                const consumeEvent = step.consumeEvent || getConsumeEventType(stepEl, step.run);
                // When in auto mode, we are not waiting for an event to be consumed, so the
                // anchor is just the step element.
                const $anchorEl = $(stepEl);

                if (showPointerDuration > 0) {
                    // Useful in watch mode.
                    pointer.pointTo($anchorEl.get(0), step);
                    await new Promise((r) => browser.setTimeout(r, showPointerDuration));
                    pointer.hide();
                }

                // TODO: Delegate the following routine to the `ACTION_HELPERS` in the macro module.
                const actionHelper = new RunningTourActionHelper({
                    consume_event: consumeEvent,
                    $anchor: $anchorEl,
                });

                let result;
                if (typeof step.run === "function") {
                    const willUnload = await callWithUnloadCheck(() =>
                        // `this.$anchor` is expected in many `step.run`.
                        step.run.call({ $anchor: $anchorEl }, actionHelper)
                    );
                    result = willUnload && "will unload";
                } else if (step.run !== undefined) {
                    const m = step.run.match(/^([a-zA-Z0-9_]+) *(?:\(? *(.+?) *\)?)?$/);
                    actionHelper[m[1]](m[2]);
                } else if (!step.isCheck) {
                    if (stepIndex === tour.steps.length - 1) {
                        console.warn('Tour %s: ignoring action (auto) of last step', tour.name);
                    } else {
                        actionHelper.auto();
                    }
                }

                return result;
            },
        },
        {
            action: () => {
                onStepConsummed(tour, step);
            },
        },
    ];
}

/**
 * @param {import("./tour_service").Tour} tour
 * @param {object} options
 * @param {TourStep[]} options.filteredSteps
 * @param {TourStepCompiler} options.stepCompiler
 * @param {*} options.pointer
 * @param {number} options.stepDelay
 * @param {boolean} options.keepWatchBrowser
 * @param {number} options.showPointerDuration
 * @param {number} options.checkDelay
 * @param {(import("./tour_service").Tour) => void} options.onTourEnd
 */
export function compileTourToMacro(tour, options) {
    const {
        filteredSteps,
        stepCompiler,
        pointer,
        stepDelay,
        keepWatchBrowser,
        showPointerDuration,
        checkDelay,
        onStepConsummed,
        onTourEnd,
    } = options;
    const currentStepIndex = tourState.get(tour.name, "currentIndex");
    return {
        ...tour,
        checkDelay,
        steps: filteredSteps
            .reduce((newSteps, step, i) => {
                if (i < currentStepIndex) {
                    // Don't include steps before the current index because they're already done.
                    return newSteps;
                } else {
                    return [
                        ...newSteps,
                        ...stepCompiler(i, step, {
                            tour,
                            pointer,
                            stepDelay,
                            keepWatchBrowser,
                            showPointerDuration,
                            onStepConsummed,
                        }),
                    ];
                }
            }, [])
            .concat([
                {
                    action() {
                        tourState.clear(tour.name);
                        onTourEnd(tour);
                        clearTimeout(tourTimeout);
                    },
                },
            ]),
    };
}

```

## File: static\src\tour_service\tour_pointer_state.js

```javascript
/** @odoo-module **/

import { reactive } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { TourPointer } from "@web_tour/tour_pointer/tour_pointer";
import { getScrollParent } from "./tour_utils";

/**
 * @typedef {import("@web/core/position_hook").Direction} Direction
 *
 * @typedef {"in" | "out-below" | "out-above" | "unknown"} IntersectionPosition
 *
 * @typedef {ReturnType<createPointerState>["methods"]} TourPointerMethods
 *
 * @typedef TourPointerState
 * @property {HTMLElement} [anchor]
 * @property {string} [content]
 * @property {boolean} [isOpen]
 * @property {() => {}} [onClick]
 * @property {() => {}} [onMouseEnter]
 * @property {() => {}} [onMouseLeave]
 * @property {boolean} isVisible
 * @property {Direction} position
 * @property {number} rev
 *
 * @typedef {import("./tour_service").TourStep} TourStep
 */

class Intersection {
    constructor() {
        /** @type {Element | null} */
        this.currentTarget = null;
        this.rootBounds = null;
        /** @type {IntersectionPosition} */
        this._targetPosition = "unknown";
        this._observer = new IntersectionObserver((observations) =>
            this._handleObservations(observations)
        );
    }

    /** @type {IntersectionObserverCallback} */
    _handleObservations(observations) {
        if (observations.length < 1) {
            return;
        }
        const observation = observations[observations.length - 1];
        this.rootBounds = observation.rootBounds;
        if (this.rootBounds && this.currentTarget) {
            if (observation.isIntersecting) {
                this._targetPosition = "in";
            } else {
                const targetBounds = this.currentTarget.getBoundingClientRect();
                if (targetBounds.bottom < this.rootBounds.height / 2) {
                    this._targetPosition = "out-above";
                } else if (targetBounds.top > this.rootBounds.height / 2) {
                    this._targetPosition = "out-below";
                }
            }
        } else {
            this._targetPosition = "unknown";
        }
    }

    get targetPosition() {
        if (!this.rootBounds) {
            return this.currentTarget ? "in" : "unknown";
        } else {
            return this._targetPosition;
        }
    }

    /**
     * @param {Element} newTarget
     */
    setTarget(newTarget) {
        if (this.currentTarget !== newTarget) {
            if (this.currentTarget) {
                this._observer.unobserve(this.currentTarget);
            }
            if (newTarget) {
                this._observer.observe(newTarget);
            }
            this.currentTarget = newTarget;
        }
    }

    stop() {
        this._observer.disconnect();
    }
}

export function createPointerState() {
    /**
     * @param {Partial<TourPointerState>} newState
     */
    const setState = (newState) => {
        Object.assign(state, newState);
    };

    /**
     * @param {TourStep} step
     * @param {HTMLElement} [anchor]
     */
    const pointTo = (anchor, step) => {
        intersection.setTarget(anchor);
        if (anchor) {
            let { position, content } = step;
            switch (intersection.targetPosition) {
                case "unknown": {
                    // Do nothing for unknown target position.
                    break;
                }
                case "in": {
                    if (document.body.contains(floatingAnchor)) {
                        floatingAnchor.remove();
                    }
                    setState({ anchor, content, onClick: null, position, isVisible: true });
                    break;
                }
                default: {
                    const onClick = () => {
                        anchor.scrollIntoView({ behavior: "smooth", block: "nearest" });
                        hide();
                    };

                    const scrollParent = getScrollParent(anchor);
                    if (!scrollParent) {
                        setState({ anchor, content, onClick: null, position, isVisible: true });
                        return;
                    }
                    let { x, y, width, height } = scrollParent.getBoundingClientRect();

                    // If the scrolling element is within an iframe the offsets
                    // must be computed taking into account the iframe.
                    const iframeEl = scrollParent.ownerDocument.defaultView.frameElement;
                    if (iframeEl) {
                        const iframeOffset = iframeEl.getBoundingClientRect();
                        x += iframeOffset.x;
                        y += iframeOffset.y;
                    }
                    floatingAnchor.style.left = `${x + width / 2}px`;
                    if (intersection.targetPosition === "out-below") {
                        position = "top";
                        content = _t("Scroll down to reach the next step.");
                        floatingAnchor.style.top = `${y + height - TourPointer.height}px`;
                    } else if (intersection.targetPosition === "out-above") {
                        position = "bottom";
                        content = _t("Scroll up to reach the next step.");
                        floatingAnchor.style.top = `${y + TourPointer.height}px`;
                    }
                    if (!document.contains(floatingAnchor)) {
                        document.body.appendChild(floatingAnchor);
                    }
                    setState({
                        anchor: floatingAnchor,
                        content,
                        onClick,
                        position,
                        isVisible: true,
                    });
                }
            }
        } else {
            hide();
        }
    };

    function hide() {
        setState({ content: "", isVisible: false, isOpen: false });
    }

    function showContent(isOpen) {
        setState({ isOpen });
    }

    function destroy() {
        intersection.stop();
        if (document.body.contains(floatingAnchor)) {
            floatingAnchor.remove();
        }
    }

    /** @type {TourPointerState} */
    const state = reactive({});
    const intersection = new Intersection();
    const floatingAnchor = document.createElement("div");
    floatingAnchor.className = "position-fixed";

    return { state, methods: { setState, showContent, pointTo, hide, destroy } };
}

```

## File: static\src\tour_service\tour_service.js

```javascript
/** @odoo-module **/

import { EventBus, markup, whenReady, reactive } from "@odoo/owl";
import { browser } from "@web/core/browser/browser";
import { _t } from "@web/core/l10n/translation";
import { MacroEngine } from "@web/core/macro";
import { registry } from "@web/core/registry";
import { config as transitionConfig } from "@web/core/transition";
import { session } from "@web/session";
import { TourPointer } from "../tour_pointer/tour_pointer";
import { compileStepAuto, compileStepManual, compileTourToMacro } from "./tour_compilers";
import { createPointerState } from "./tour_pointer_state";
import { tourState } from "./tour_state";
import { callWithUnloadCheck } from "./tour_utils";

/**
 * @typedef {string} JQuerySelector
 * @typedef {import("./tour_utils").RunCommand} RunCommand
 *
 * @typedef Tour
 * @property {string} url
 * @property {string} name
 * @property {() => TourStep[]} steps
 * @property {boolean} [rainbowMan]
 * @property {number} [sequence]
 * @property {boolean} [test]
 * @property {Promise<any>} [wait_for]
 * @property {string} [saveAs]
 * @property {string} [fadeout]
 * @property {number} [checkDelay]
 * @property {string|undefined} [shadow_dom]
 *
 * @typedef TourStep
 * @property {string} [id]
 * @property {JQuerySelector} trigger
 * @property {JQuerySelector} [extra_trigger]
 * @property {JQuerySelector} [alt_trigger]
 * @property {JQuerySelector} [skip_trigger]
 * @property {string} [content]
 * @property {"top" | "botton" | "left" | "right"} [position]
 * @property {"community" | "enterprise"} [edition]
 * @property {RunCommand} [run]
 * @property {boolean} [auto]
 * @property {boolean} [in_modal]
 * @property {number} [width]
 * @property {number} [timeout]
 * @property {boolean} [consumeVisibleOnly]
 * @property {boolean} [noPrepend]
 * @property {string} [consumeEvent]
 * @property {boolean} [mobile]
 * @property {string} [title]
 * @property {string|false|undefined} [shadow_dom]
 *
 * @typedef {"manual" | "auto"} TourMode
 */

export const tourService = {
    // localization dependency to make sure translations used by tours are loaded
    dependencies: ["orm", "effect", "ui", "overlay", "localization"],
    start: async (_env, { orm, effect, ui, overlay }) => {
        await whenReady();
        const toursEnabled = "tour_disable" in session && !session.tour_disable;
        const consumedTours = new Set(session.web_tours);

        /** @type {{ [k: string]: Tour }} */
        const tours = {};
        const tourRegistry = registry.category("web_tour.tours");
        function register(name, tour) {
            name = tour.saveAs || name;
            const wait_for = tour.wait_for || Promise.resolve();
            let steps;
            tours[name] = {
                wait_for,
                name,
                get steps() {
                    if (typeof tour.steps !== "function") {
                        throw new Error(`tour.steps has to be a function that returns TourStep[]`);
                    }
                    if (!steps) {
                        steps = tour.steps().map((step) => {
                            step.shadow_dom = step.shadow_dom ?? tour.shadow_dom;
                            return step;
                        });
                    }
                    return steps;
                },
                shadow_dom: tour.shadow_dom,
                url: tour.url,
                rainbowMan: tour.rainbowMan === undefined ? true : !!tour.rainbowMan,
                rainbowManMessage: tour.rainbowManMessage,
                fadeout: tour.fadeout || "medium",
                sequence: tour.sequence || 1000,
                test: tour.test,
                checkDelay: tour.checkDelay,
            };
            wait_for.then(() => {
                if (
                    !tour.test &&
                    toursEnabled &&
                    !consumedTours.has(name) &&
                    !tourState.getActiveTourNames().includes(name)
                ) {
                    startTour(name, { mode: "manual", redirect: false });
                }
            });
        }
        for (const [name, tour] of tourRegistry.getEntries()) {
            register(name, tour);
        }
        tourRegistry.addEventListener("UPDATE", ({ detail: { key, value } }) => {
            if (tourRegistry.contains(key)) {
                register(key, value);
                if (
                    tourState.getActiveTourNames().includes(key) &&
                    // Don't resume onboarding tours when tours are disabled
                    (toursEnabled || tourState.get(key, "mode") === "auto")
                ) {
                    resumeTour(key);
                }
            } else {
                delete tours[value];
            }
        });

        const bus = new EventBus();
        const macroEngine = new MacroEngine({ target: document });

        const pointers = reactive({});
        /** @type {Set<string>} */
        const runningTours = new Set();

        // FIXME: this is a hack for stable: whenever the macros advance, for each call to pointTo,
        // we push a function that will do the pointing as well as the tour name. Then after
        // a microtask tick, when all pointTo calls have been made by the macro system, we can sort
        // these by tour priority/sequence and only call the one with the highest priority so we
        // show the correct pointer.
        const possiblePointTos = [];
        function createPointer(tourName, config) {
            const { state: pointerState, methods } = createPointerState();
            let remove;
            return {
                start() {
                    pointers[tourName] = {
                        methods,
                        id: tourName,
                        component: TourPointer,
                        props: { pointerState, ...config },
                    };
                    remove = overlay.add(pointers[tourName].component, pointers[tourName].props);
                },
                stop() {
                    remove?.();
                    delete pointers[tourName];
                    methods.destroy();
                },
                ...methods,
                async pointTo(anchor, step) {
                    possiblePointTos.push([tourName, () => methods.pointTo(anchor, step)]);
                    await Promise.resolve();
                    // only done once per macro advance
                    if (!possiblePointTos.length) {
                        return;
                    }
                    const toursByPriority = Object.fromEntries(
                        getSortedTours().map((t, i) => [t.name, i])
                    );
                    const sortedPointTos = possiblePointTos
                        .slice(0)
                        .sort(([a], [b]) => toursByPriority[a] - toursByPriority[b]);
                    possiblePointTos.splice(0); // reset for the next macro advance

                    const active = sortedPointTos[0];
                    const [activeId, enablePointer] = active || [];
                    for (const { id, methods } of Object.values(pointers)) {
                        if (id === activeId) {
                            enablePointer();
                        } else {
                            methods.hide();
                        }
                    }
                },
            };
        }

        /**
         * @param {TourStep} step
         * @param {TourMode} mode
         */
        function shouldOmit(step, mode) {
            const isDefined = (key, obj) => key in obj && obj[key] !== undefined;
            const getEdition = () =>
                (session.server_version_info || []).at(-1) === "e" ? "enterprise" : "community";
            const correctEdition = isDefined("edition", step)
                ? step.edition === getEdition()
                : true;
            const correctDevice = isDefined("mobile", step) ? step.mobile === ui.isSmall : true;
            return (
                !correctEdition ||
                !correctDevice ||
                // `step.auto = true` means omitting a step in a manual tour.
                (mode === "manual" && step.auto)
            );
        }

        /**
         * @param {Tour} tour
         * @param {ReturnType<typeof createPointer>} pointer
         * @param {Object} options
         * @param {TourMode} options.mode
         * @param {number} options.stepDelay
         * @param {boolean} options.keepWatchBrowser - do not close watch browser when the tour failed
         * @param {number} options.showPointerDuration
         * - Useful when watching auto tour.
         * - Show the pointer for some duration before performing calling the run method.
         */
        function convertToMacro(
            tour,
            pointer,
            { mode, stepDelay, keepWatchBrowser, showPointerDuration }
        ) {
            // IMPROVEMENTS: Custom step compiler. Will probably require decoupling from `mode`.
            const stepCompiler = mode === "auto" ? compileStepAuto : compileStepManual;
            const checkDelay = mode === "auto" ? tour.checkDelay : 100;
            const filteredSteps = tour.steps.filter((step) => !shouldOmit(step, mode));
            return compileTourToMacro(tour, {
                filteredSteps,
                stepCompiler,
                pointer,
                stepDelay,
                keepWatchBrowser,
                showPointerDuration,
                checkDelay,
                onStepConsummed(tour, step) {
                    bus.trigger("STEP-CONSUMMED", { tour, step });
                },
                onTourEnd({ name, rainbowManMessage, fadeout }) {
                    if (mode === "auto") {
                        transitionConfig.disabled = false;
                    }
                    let message;
                    if (typeof rainbowManMessage === "function") {
                        message = rainbowManMessage({
                            isTourConsumed: (name) => consumedTours.has(name),
                        });
                    } else if (typeof rainbowManMessage === "string") {
                        message = rainbowManMessage;
                    } else {
                        message = markup(
                            _t(
                                "<strong><b>Good job!</b> You went through all steps of this tour.</strong>"
                            )
                        );
                    }
                    effect.add({ type: "rainbow_man", message, fadeout });
                    if (mode === "manual") {
                        consumedTours.add(name);
                        orm.call("web_tour.tour", "consume", [[name]]);
                    }
                    pointer.stop();
                    // Used to signal the python test runner that the tour finished without error.
                    browser.console.log("test successful");
                    runningTours.delete(name);
                },
            });
        }

        /**
         * Wait for the shadow hosts matching the given selectors to
         * appear in the DOM then, register the underlying shadow roots
         * to the macro engine observer in order to listen to the
         * changes in the shadow DOM.
         *
         * @param {Set<string>} shadowHostSelectors
         */
        function observeShadows(shadowHostSelectors) {
            const observer = new MutationObserver(() => {
                const shadowRoots = [];
                for (const selector of shadowHostSelectors) {
                    const shadowHost = document.querySelector(selector);
                    if (shadowHost) {
                        shadowRoots.push(shadowHost.shadowRoot);
                        shadowHostSelectors.delete(selector);
                    }
                }
                for (const shadowRoot of shadowRoots) {
                    macroEngine.observer.observe(shadowRoot, macroEngine.observerOptions);
                }
                if (shadowHostSelectors.size === 0) {
                    observer.disconnect();
                }
            });
            observer.observe(macroEngine.target, { childList: true, subtree: true });
        }

        /**
         * Register shadow roots that must be observed by the tour to
         * the macro engine.
         *
         * @param {Tour} tour
         */
        function setupShadowObservers(tour) {
            const shadowDOMs = new Set(
                tour.steps.filter((step) => step.shadow_dom).map((step) => step.shadow_dom)
            );
            if (shadowDOMs.size > 0) {
                observeShadows(shadowDOMs);
            }
        }

        /**
         * Disable transition before starting an "auto" tour.
         * @param {Macro} macro
         * @param {'auto' | 'manual'} mode
         */
        function activateMacro(macro, mode) {
            if (mode === "auto") {
                transitionConfig.disabled = true;
            }
            macroEngine.activate(macro, mode === "auto");
        }

        function startTour(tourName, options = {}) {
            if (runningTours.has(tourName) && options.mode === "manual") {
                return;
            }
            runningTours.add(tourName);
            const defaultOptions = {
                stepDelay: 0,
                keepWatchBrowser: false,
                mode: "auto",
                startUrl: "",
                showPointerDuration: 0,
                redirect: true,
            };
            options = Object.assign(defaultOptions, options);
            const tour = tours[tourName];
            if (!tour) {
                throw new Error(`Tour '${tourName}' is not found.`);
            }
            tourState.set(tourName, "currentIndex", 0);
            tourState.set(tourName, "stepDelay", options.stepDelay);
            tourState.set(tourName, "keepWatchBrowser", options.keepWatchBrowser);
            tourState.set(tourName, "showPointerDuration", options.showPointerDuration);
            tourState.set(tourName, "mode", options.mode);
            tourState.set(tourName, "sequence", tour.sequence);
            const pointer = createPointer(tourName, {
                bounce: !(options.mode === "auto" && options.keepWatchBrowser),
            });
            const macro = convertToMacro(tour, pointer, options);
            const willUnload = callWithUnloadCheck(() => {
                if (tour.url && tour.url !== options.startUrl && options.redirect) {
                    window.location.href = window.location.origin + tour.url;
                }
            });
            if (!willUnload) {
                setupShadowObservers(tour);
                pointer.start();
                activateMacro(macro, options.mode);
            }
        }

        function resumeTour(tourName) {
            if (runningTours.has(tourName)) {
                return;
            }
            runningTours.add(tourName);
            const tour = tours[tourName];
            const stepDelay = tourState.get(tourName, "stepDelay");
            const keepWatchBrowser = tourState.get(tourName, "keepWatchBrowser");
            const showPointerDuration = tourState.get(tourName, "showPointerDuration");
            const mode = tourState.get(tourName, "mode");
            const pointer = createPointer(tourName, {
                bounce: !(mode === "auto" && keepWatchBrowser),
            });
            const macro = convertToMacro(tour, pointer, {
                mode,
                stepDelay,
                keepWatchBrowser,
                showPointerDuration,
            });
            setupShadowObservers(tour);
            pointer.start();
            activateMacro(macro, mode);
        }

        function getSortedTours() {
            return Object.values(tours).sort((t1, t2) => {
                return t1.sequence - t2.sequence || (t1.name < t2.name ? -1 : 1);
            });
        }

        if (!window.frameElement) {
            // Resume running tours.
            for (const tourName of tourState.getActiveTourNames()) {
                if (tourName in tours) {
                    resumeTour(tourName);
                }
            }
        }

        odoo.startTour = startTour;
        odoo.isTourReady = (tourName) => tours[tourName].wait_for.then(() => true);

        return {
            bus,
            startTour,
            getSortedTours,
        };
    },
};

registry.category("services").add("tour_service", tourService);

```

## File: static\src\tour_service\tour_state.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";

const BOOLEAN = {
    toLocalStorage: (val) => (val ? "1" : "0"),
    fromLocalStorage: (val) => (val === "1" ? true : false),
};

const INTEGER = {
    toLocalStorage: (val) => val.toString(),
    fromLocalStorage: (val) => parseInt(val, 10),
};

const STRING = {
    toLocalStorage: (x) => x,
    fromLocalStorage: (x) => x,
};

const ALLOWED_KEYS = {
    // Don't close the 'watch' browser when the tour failed.
    keepWatchBrowser: BOOLEAN,

    // Duration at which the pointer is shown in auto mode.
    showPointerDuration: INTEGER,

    // Index of the current step.
    currentIndex: INTEGER,

    // Global step delay that is specified before starting the tour.
    stepDelay: INTEGER,

    // 'auto' | 'manual' - important that it's persisted because it's only specified during start of tour.
    mode: STRING,

    // Used to order the tours.
    sequence: INTEGER,
};

function getPrefixedName(tourName, key) {
    return `tour__${tourName}__${key}`;
}

function destructurePrefixedName(prefixedName) {
    const match = prefixedName.match(/tour__([.\w]+)__([\w]+)/);
    return match ? [match[1], match[2]] : null;
}

/**
 * Wrapper around localStorage for persistence of the running tours.
 * Useful for resuming running tours when the page refreshed.
 */
export const tourState = {
    get(tourName, key) {
        if (!(key in ALLOWED_KEYS)) {
            throw new Error(`Invalid key: '${key}' (tourName = '${tourName}')`);
        }
        const prefixedName = getPrefixedName(tourName, key);
        const savedValue = browser.localStorage.getItem(prefixedName);
        return ALLOWED_KEYS[key].fromLocalStorage(savedValue);
    },
    set(tourName, key, value) {
        if (!(key in ALLOWED_KEYS)) {
            throw new Error(`Invalid key: '${key}' (tourName = '${tourName}')`);
        }
        const prefixedName = getPrefixedName(tourName, key);
        browser.localStorage.setItem(prefixedName, ALLOWED_KEYS[key].toLocalStorage(value));
    },
    clear(tourName) {
        for (const key in ALLOWED_KEYS) {
            const prefixedName = getPrefixedName(tourName, key);
            browser.localStorage.removeItem(prefixedName);
        }
    },
    getActiveTourNames() {
        const tourNames = new Set();
        for (const key of Object.keys(browser.localStorage)) {
            const [tourName] = destructurePrefixedName(key) || [false];
            if (tourName) {
                tourNames.add(tourName);
            }
        }
        return [...tourNames].sort((a, b) => this.get(a, "sequence") - this.get(b, "sequence"));
    },
};

```

## File: static\src\tour_service\tour_utils.js

```javascript
/** @odoo-module **/

import { markup } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { utils } from "@web/core/ui/ui_service";
import { _legacyIsVisible } from "@web/core/utils/ui";

/**
 * @typedef {string | (actions: RunningTourActionHelper) => void | Promise<void>} RunCommand
 */

export class TourError extends Error {
    constructor(message, ...args) {
        super(message, ...args);
        this.message = `(TourError) ${message}`;
    }
}

/**
 * Calls the given `func` then returns/resolves to `true`
 * if it will result to unloading of the page.
 * @param {(...args: any[]) => void} func
 * @param  {any[]} args
 * @returns {boolean | Promise<boolean>}
 */
export function callWithUnloadCheck(func, ...args) {
    let willUnload = false;
    const beforeunload = () => (willUnload = true);
    window.addEventListener("beforeunload", beforeunload);
    const result = func(...args);
    if (result instanceof Promise) {
        return result.then(() => {
            window.removeEventListener("beforeunload", beforeunload);
            return willUnload;
        });
    } else {
        window.removeEventListener("beforeunload", beforeunload);
        return willUnload;
    }
}

export function getFirstVisibleElement($elements) {
    for (var i = 0; i < $elements.length; i++) {
        var $i = $elements.eq(i);
        if (_legacyIsVisible($i[0])) {
            return $i;
        }
    }
    return $();
}

/**
 * @param {JQuery|undefined} target
 */
export function getJQueryElementFromSelector(selector, $target) {
    $target = $target || $(document);
    const iframeSplit = typeof selector === "string" && selector.match(/(.*\biframe[^ ]*)(.*)/);
    if (iframeSplit && iframeSplit[2]) {
        var $iframe = $target.find(`${iframeSplit[1]}:not(.o_ignore_in_tour)`);
        if ($iframe.is('[is-ready="false"]')) {
            return $();
        }
        var $el = $iframe.contents().find(iframeSplit[2]);
        $el.iframeContainer = $iframe[0];
        return $el;
    } else if (typeof selector === "string") {
        return $target.find(selector);
    } else {
        return $(selector);
    }
}

/**
 * @param {HTMLElement} [element]
 * @param {RunCommand} [runCommand]
 * @returns {string}
 */
export function getConsumeEventType(element, runCommand) {
    if (!element) {
        return "click";
    }
    const { classList, tagName, type } = element;
    const tag = tagName.toLowerCase();

    // Many2one
    if (classList.contains("o_field_many2one")) {
        return "autocompleteselect";
    }

    // Inputs and textareas
    if (
        tag === "textarea" ||
        (tag === "input" &&
            (!type ||
                ["email", "number", "password", "search", "tel", "text", "url"].includes(type)))
    ) {
        if (
            utils.isSmall() &&
            element.closest(".o_field_widget")?.matches(".o_field_many2one, .o_field_many2many")
        ) {
            return "click";
        }
        return "input";
    }

    // jQuery draggable
    if (classList.contains("ui-draggable-handle")) {
        return "mousedown";
    }

    // Drag & drop run command
    if (typeof runCommand === "string" && /^drag_and_drop/.test(runCommand)) {
        // this is a heuristic: the element has to be dragged and dropped but it
        // doesn't have class 'ui-draggable-handle', so we check if it has an
        // ui-sortable parent, and if so, we conclude that its event type is 'sort'
        if (element.closest(".ui-sortable")) {
            return "sort";
        }
        if (
            (/^drag_and_drop_native/.test(runCommand) && classList.contains("o_draggable")) ||
            element.closest(".o_draggable")
        ) {
            return "pointerdown";
        }
    }

    // Default: click
    return "click";
}

/**
 * ! This function is a copy-paste of its namesake in web/static/tests/helpers/utils.js
 * TODO: Unify utils for tests and tours since they're doing the exact same thing
 * @param {Node} n1
 * @param {Node} n2
 * @returns {Node[]}
 */
export function getDifferentParents(n1, n2) {
    const parents = [n2];
    while (parents[0].parentNode) {
        const parent = parents[0].parentNode;
        if (parent.contains(n1)) {
            break;
        }
        parents.unshift(parent);
    }
    return parents;
}

/**
 * @param {HTMLElement} element
 * @returns {HTMLElement | null}
 */
export function getScrollParent(element) {
    if (!element) {
        return null;
    }
    if (element.scrollHeight > element.clientHeight) {
        return element;
    } else {
        return getScrollParent(element.parentNode);
    }
}

/**
 * @param {HTMLElement} el
 * @param {string} type
 * @param {boolean} canBubbleAndBeCanceled
 * @param {PointerEventInit} [additionalParams]
 */
export const triggerPointerEvent = (el, type, canBubbleAndBeCanceled, additionalParams) => {
    const eventInit = {
        bubbles: canBubbleAndBeCanceled,
        cancelable: canBubbleAndBeCanceled,
        view: window,
        ...additionalParams,
    };

    el.dispatchEvent(new PointerEvent(type, eventInit));
    if (type.startsWith("pointer")) {
        el.dispatchEvent(new MouseEvent(type.replace("pointer", "mouse"), eventInit));
    }
};

export class RunningTourActionHelper {
    constructor(tip_widget) {
        this.tip_widget = tip_widget;
    }
    click(element) {
        this._click(this._get_action_values(element));
    }
    dblclick(element) {
        this._click(this._get_action_values(element), 2);
    }
    tripleclick(element) {
        this._click(this._get_action_values(element), 3);
    }
    clicknoleave(element) {
        this._click(this._get_action_values(element), 1, false);
    }
    text(text, element) {
        this._text(this._get_action_values(element), text);
    }
    remove_text(text, element) {
        this._text(this._get_action_values(element), "\n");
    }
    text_blur(text, element) {
        this._text_blur(this._get_action_values(element), text);
    }
    range(text, element) {
        this._range(this._get_action_values(element), text);
    }
    drag_and_drop(to, element) {
        this._drag_and_drop_jquery(this._get_action_values(element), to);
    }
    drag_and_drop_native(toSel, element) {
        const to = getJQueryElementFromSelector(toSel)[0];
        this._drag_and_drop(this._get_action_values(element).$element[0], to);
    }
    keydown(keyCodes, element) {
        this._keydown(this._get_action_values(element), keyCodes.split(/[,\s]+/));
    }
    auto(element) {
        var values = this._get_action_values(element);
        if (values.consume_event === "input") {
            this._text(values);
        } else {
            this._click(values);
        }
    }
    _get_action_values(element) {
        var $e = getJQueryElementFromSelector(element);
        var $element = element ? getFirstVisibleElement($e) : this.tip_widget.$anchor;
        if ($element.length === 0) {
            $element = $e.first();
        }
        var consume_event = element
            ? getConsumeEventType($element[0])
            : this.tip_widget.consume_event;
        return {
            $element: $element,
            consume_event: consume_event,
        };
    }
    _click(values, nb, leave) {
        const target = values.$element[0];
        triggerPointerEvent(target, "pointerover", true);
        triggerPointerEvent(target, "pointerenter", false);
        triggerPointerEvent(target, "pointermove", true);
        for (let i = 1; i <= (nb || 1); i++) {
            triggerPointerEvent(target, "pointerdown", true);
            triggerPointerEvent(target, "pointerup", true);
            triggerPointerEvent(target, "click", true, { detail: i });
            if (i % 2 === 0) {
                triggerPointerEvent(target, "dblclick", true);
            }
        }
        if (leave !== false) {
            triggerPointerEvent(target, "pointerout", true);
            triggerPointerEvent(target, "pointerleave", false);
        }
    }
    _text(values, text) {
        this._click(values);

        text = text || "Test";
        if (values.consume_event === "input") {
            values.$element
                .trigger({ type: "keydown", key: text[text.length - 1] })
                .val(text)
                .trigger({ type: "keyup", key: text[text.length - 1] });
            values.$element[0].dispatchEvent(
                new InputEvent("input", {
                    bubbles: true,
                })
            );
        } else if (values.$element.is("select")) {
            var $options = values.$element.find("option");
            $options.prop("selected", false).removeProp("selected");
            var $selectedOption = $options.filter(function () {
                return $(this).val() === text;
            });
            if ($selectedOption.length === 0) {
                $selectedOption = $options.filter(function () {
                    return $(this).text().trim() === text;
                });
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
            values.$element.trigger($.Event("keydown", { key: "_" }));
            values.$element.text(text).trigger("input");
            values.$element.focusInEnd();
            values.$element.trigger($.Event("keyup", { key: "_" }));
        }
        values.$element[0].dispatchEvent(new Event("change", { bubbles: true, cancelable: false }));
    }
    _text_blur(values, text) {
        this._text(values, text);
        values.$element.trigger("focusout");
        values.$element.trigger("blur");
    }
    _range(values, text) {
        values.$element[0].value = text;
        values.$element[0].dispatchEvent(new Event('change', { bubbles: true, cancelable: false }));
    }
    _calculateCenter($el, selector) {
        const center = $el.offset();
        if (selector && selector.indexOf("iframe") !== -1) {
            const iFrameOffset = $("iframe").offset();
            center.left += iFrameOffset.left;
            center.top += iFrameOffset.top;
        }
        center.left += $el.outerWidth() / 2;
        center.top += $el.outerHeight() / 2;
        return center;
    }
    _drag_and_drop_jquery(values, to) {
        var $to;
        const elementCenter = this._calculateCenter(values.$element);
        if (to) {
            $to = getJQueryElementFromSelector(to);
        } else {
            $to = $(document.body);
        }

        values.$element.trigger($.Event("mouseenter"));
        // Make the web_studio tour test happy. My guess is that 50%+ of the length of the dragged element
        // must be situated to the right of the $to element.
        values.$element.trigger(
            $.Event("mousedown", {
                which: 1,
                pageX: elementCenter.left + 1,
                pageY: elementCenter.top,
            })
        );
        // Some tests depends on elements present only when the element to drag
        // start to move while some other tests break while moving.
        if (!$to.length) {
            values.$element.trigger(
                $.Event("mousemove", {
                    which: 1,
                    pageX: elementCenter.left + 1,
                    pageY: elementCenter.top,
                })
            );
            $to = getJQueryElementFromSelector(to);
        }

        let toCenter = this._calculateCenter($to, to);
        values.$element.trigger(
            $.Event("mousemove", { which: 1, pageX: toCenter.left, pageY: toCenter.top })
        );
        // Recalculate the center as the mousemove might have made the element bigger.
        toCenter = this._calculateCenter($to, to);
        values.$element.trigger(
            $.Event("mouseup", { which: 1, pageX: toCenter.left, pageY: toCenter.top })
        );
    }
    /**
     * ! This function is a reduced version of "drag" in web/static/tests/helpers/utils.js
     * TODO: Unify utils for tests and tours since they're doing the exact same thing
     * @param {HTMLElement} source
     * @param {HTMLElement} target
     */
    _drag_and_drop(source, target) {
        const sourceRect = source.getBoundingClientRect();
        const sourcePosition = {
            clientX: sourceRect.x + sourceRect.width / 2,
            clientY: sourceRect.y + sourceRect.height / 2,
        };

        const targetRect = target.getBoundingClientRect();
        const targetPosition = {
            clientX: targetRect.x + targetRect.width / 2,
            clientY: targetRect.y + targetRect.height / 2,
        };

        triggerPointerEvent(source, "pointerdown", true, sourcePosition);
        triggerPointerEvent(source, "pointermove", true, targetPosition);

        for (const parent of getDifferentParents(source, target)) {
            triggerPointerEvent(parent, "pointerenter", false, targetPosition);
        }

        triggerPointerEvent(target, "pointerup", true, targetPosition);
    }
    _keydown(values, keyCodes) {
        while (keyCodes.length) {
            const eventOptions = {};
            const keyCode = keyCodes.shift();
            let insertedText = null;
            if (isNaN(keyCode)) {
                eventOptions.key = keyCode;
            } else {
                const code = parseInt(keyCode, 10);
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
    }
}

export const stepUtils = {
    _getHelpMessage(functionName, ...args) {
        return `Generated by function tour utils ${functionName}(${args.join(", ")})`;
    },

    addDebugHelp(helpMessage, step) {
        if (typeof step.debugHelp === "string") {
            step.debugHelp = step.debugHelp + "\n" + helpMessage;
        } else {
            step.debugHelp = helpMessage;
        }
        return step;
    },

    editionEnterpriseModifier(step) {
        step.edition = "enterprise";
        return step;
    },

    mobileModifier(step) {
        step.mobile = true;
        return step;
    },

    showAppsMenuItem() {
        return {
            edition: "community",
            trigger: ".o_navbar_apps_menu button",
            auto: true,
            position: "bottom",
        };
    },

    toggleHomeMenu() {
        return {
            edition: "enterprise",
            trigger: ".o_main_navbar .o_menu_toggle",
            content: markup(_t("Click on the <i>Home icon</i> to navigate across apps.")),
            position: "bottom",
        };
    },

    autoExpandMoreButtons(extra_trigger) {
        return {
            trigger: ".o-form-buttonbox",
            extra_trigger: extra_trigger,
            auto: true,
            run: (actions) => {
                const $more = $(".o-form-buttonbox .o_button_more");
                if ($more.length) {
                    actions.click($more);
                }
            },
        };
    },

    goBackBreadcrumbsMobile(description, ...extraTrigger) {
        return extraTrigger.map((element) => ({
            mobile: true,
            trigger: ".o_back_button",
            extra_trigger: element,
            content: description,
            position: "bottom",
            debugHelp: this._getHelpMessage(
                "goBackBreadcrumbsMobile",
                description,
                ...extraTrigger
            ),
        }));
    },

    goToAppSteps(dataMenuXmlid, description) {
        return [
            this.showAppsMenuItem(),
            {
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                position: "right",
                edition: "community",
            },
            {
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                position: "bottom",
                edition: "enterprise",
            },
        ].map((step) =>
            this.addDebugHelp(this._getHelpMessage("goToApp", dataMenuXmlid, description), step)
        );
    },

    openBurgerMenu(extraTrigger) {
        return {
            mobile: true,
            trigger: ".o_mobile_menu_toggle",
            extra_trigger: extraTrigger,
            content: _t("Open bugger menu."),
            position: "bottom",
            debugHelp: this._getHelpMessage("openBurgerMenu", extraTrigger),
        };
    },

    statusbarButtonsSteps(innerTextButton, description, extraTrigger) {
        return [
            {
                mobile: true,
                auto: true,
                trigger: ".o_statusbar_buttons",
                extra_trigger: extraTrigger,
                run: (actions) => {
                    const $action = $(".o_statusbar_buttons .btn.dropdown-toggle:contains(Action)");
                    if ($action.length) {
                        actions.click($action);
                    }
                },
            },
            {
                trigger: `.o_statusbar_buttons button:enabled:contains('${innerTextButton}')`,
                content: description,
                position: "bottom",
            },
        ].map((step) =>
            this.addDebugHelp(
                this._getHelpMessage(
                    "statusbarButtonsSteps",
                    innerTextButton,
                    description,
                    extraTrigger
                ),
                step
            )
        );
    },

    simulateEnterKeyboardInSearchModal() {
        return {
            mobile: true,
            trigger: ".o_searchview_input",
            extra_trigger: ".modal:not(.o_inactive_modal) .dropdown-menu.o_searchview_autocomplete",
            position: "bottom",
            run: (action) => {
                const keyEventEnter = new KeyboardEvent("keydown", {
                    bubbles: true,
                    cancelable: true,
                    key: "Enter",
                    code: "Enter",
                });
                action.tip_widget.$anchor[0].dispatchEvent(keyEventEnter);
            },
            debugHelp: this._getHelpMessage("simulateEnterKeyboardInSearchModal"),
        };
    },

    mobileKanbanSearchMany2X(modalTitle, valueSearched) {
        return [
            {
                mobile: true,
                trigger: `.o_control_panel_navigation .btn .fa-search`,
                position: "bottom",
            },
            {
                mobile: true,
                trigger: ".o_searchview_input",
                extra_trigger: `.modal:not(.o_inactive_modal) .modal-title:contains('${modalTitle}')`,
                position: "bottom",
                run: `text ${valueSearched}`,
            },
            this.simulateEnterKeyboardInSearchModal(),
            {
                mobile: true,
                trigger: `.o_kanban_record .o_kanban_record_title :contains('${valueSearched}')`,
                position: "bottom",
            },
        ].map((step) =>
            this.addDebugHelp(
                this._getHelpMessage("mobileKanbanSearchMany2X", modalTitle, valueSearched),
                step
            )
        );
    },
    /**
     * Utility steps to save a form and wait for the save to complete
     *
     * @param {object} [options]
     * @param {string} [options.content]
     * @param {string} [options.extra_trigger] additional save-condition selector
     */
    saveForm(options = {}) {
        return [
            {
                content: options.content || "save form",
                trigger: ".o_form_button_save",
                extra_trigger: options.extra_trigger,
                run: "click",
                auto: true,
            },
            {
                content: "wait for save completion",
                trigger: ".o_form_readonly, .o_form_saved",
                run() {},
                auto: true,
            },
        ];
    },
    /**
     * Utility steps to cancel a form creation or edition.
     *
     * Supports creation/edition from either a form or a list view (so checks
     * for both states).
     */
    discardForm(options = {}) {
        return [
            {
                content: options.content || "exit the form",
                trigger: ".o_form_button_cancel",
                extra_trigger: options.extra_trigger,
                run: "click",
                auto: true,
            },
            {
                content: "wait for cancellation to complete",
                trigger:
                    ".o_view_controller.o_list_view, .o_form_view > div > div > .o_form_readonly, .o_form_view > div > div > .o_form_saved",
                run() {},
                auto: true,
            },
        ];
    },
};

```

## File: views\tour_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="edit_tour_form" model="ir.ui.view">
            <field name="model">web_tour.tour</field>
            <field name="arch" type="xml">
                <form create="0" edit="0">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                            </group>
                            <group>
                                <field name="user_id"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="edit_tour_list" model="ir.ui.view">
            <field name="model">web_tour.tour</field>
            <field name="arch" type="xml">
                <tree string="Menu" create="0" edit="0">
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

