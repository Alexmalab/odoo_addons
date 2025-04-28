# Odoo Module: web_tour

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
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
    'version': '1.0',
    'depends': ['web'],
    'data': [
        'security/ir.model.access.csv',
        'views/tour_views.xml',
        'views/res_users_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'web_tour/static/src/**/*',
            'web/static/lib/hoot-dom/**/*',
        ],
        'web.assets_frontend': [
            'web_tour/static/src/tour_pointer/**/*',
            'web_tour/static/src/tour_service/**/*',
            'web/static/lib/hoot-dom/**/*',
        ],
        'web.assets_unit_tests': [
            'web_tour/static/tests/*.test.js',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\ir_http.py

```python
from odoo import models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super().session_info()
        result["tour_enabled"] = self.env.user.tour_enabled
        result['current_tour'] = self.env["web_tour.tour"].get_current_tour()
        return result

```

## File: models\res_users.py

```python
from odoo import models, fields, api


class ResUsers(models.Model):
    _inherit = "res.users"

    tour_enabled = fields.Boolean(compute='_compute_tour_enabled', store=True, readonly=False, string="Onboarding")

    @api.depends("create_date")
    def _compute_tour_enabled(self):
        demo_modules_count = self.env['ir.module.module'].sudo().search_count([('demo', '=', True)])
        for user in self:
            user.tour_enabled = user._is_admin() and demo_modules_count == 0

    @api.model
    def switch_tour_enabled(self, val):
        self.env.user.sudo().tour_enabled = val
        return self.env.user.tour_enabled

```

## File: models\tour.py

```python
from odoo import api, fields, models, Command
import json
import base64


class Tour(models.Model):
    _name = "web_tour.tour"
    _description = "Tours"
    _order = "sequence, name, id"

    name = fields.Char(required=True)
    step_ids = fields.One2many("web_tour.tour.step", "tour_id")
    url = fields.Char(string="Starting URL", default="/odoo")
    sharing_url = fields.Char(compute="_compute_sharing_url", string="Sharing URL")
    rainbow_man_message = fields.Html(default="<b>Good job!</b> You went through all steps of this tour.", translate=True)
    sequence = fields.Integer(default=1000)
    custom = fields.Boolean(string="Custom")
    user_consumed_ids = fields.Many2many("res.users")

    _sql_constraints = [
        ('uniq_name', 'unique(name)', "A tour already exists with this name . Tour's name must be unique!"),
    ]

    @api.depends("name")
    def _compute_sharing_url(self):
        for tour in self:
            tour.sharing_url = f"{tour.get_base_url()}/odoo?tour={tour.name}"

    @api.model
    def consume(self, tourName):
        if self.env.user and self.env.user._is_internal():
            tour_id = self.search([("name", "=", tourName)])
            if tour_id:
                tour_id.sudo().user_consumed_ids = [Command.link(self.env.user.id)]
        return self.get_current_tour()

    @api.model
    def get_current_tour(self):
        if self.env.user and self.env.user.tour_enabled and self.env.user._is_internal():
            tours_to_run = self.search([("custom", "=", False), ("user_consumed_ids", "not in", self.env.user.id)])
            return bool(tours_to_run[:1]) and tours_to_run[:1]._get_tour_json()

    @api.model
    def get_tour_json_by_name(self, tour_name):
        tour_id = self.search([("name", "=", tour_name)])
        return tour_id._get_tour_json()

    def _get_tour_json(self):
        tour_json = self.read(fields={
            "name",
            "url",
            "custom"
        })[0]

        del tour_json["id"]
        tour_json["steps"] = self.step_ids.get_steps_json()
        tour_json["rainbowManMessage"] = self.rainbow_man_message
        return tour_json

    def export_js_file(self):
        js_content = f"""import {{ registry }} from '@web/core/registry';

registry.category("web_tour.tours").add("{self.name}", {{
    url: "{self.url}",
    steps: () => {json.dumps(self.step_ids.get_steps_json(), indent=4)}
}})"""

        attachment_id = self.env["ir.attachment"].create({
            "datas": base64.b64encode(bytes(js_content, 'utf-8')),
            "name": f"{self.name}.js",
            "mimetype": "application/javascript",
            "res_model": "web_tour.tour",
            "res_id": self.id,
        })

        return {
            "type": "ir.actions.act_url",
            "url": f"/web/content/{attachment_id.id}?download=true",
        }


class TourStep(models.Model):
    _name = "web_tour.tour.step"
    _description = "Tour's step"
    _order = "sequence, id"

    trigger = fields.Char(required=True)
    content = fields.Char()
    tour_id = fields.Many2one("web_tour.tour", required=True, ondelete="cascade")
    run = fields.Char()
    sequence = fields.Integer()

    def get_steps_json(self):
        steps = []

        for step in self.read(fields=["trigger", "content", "run"]):
            del step["id"]
            if not step["content"]:
                del step["content"]
            steps.append(step)

        return steps

```

## File: models\__init__.py

```python
from . import res_users
from . import ir_http
from . import tour

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
web_tour.access_web_tour_tour_admin,access_web_tour_tour_admin,model_web_tour_tour,base.group_system,1,1,1,1
web_tour.access_web_tour_tour_readonly,access_web_tour_tour_readonly,model_web_tour_tour,base.group_user,1,0,0,0
web_tour.access_web_tour_tour_step_admin,access_web_tour_tour_step_admin,web_tour.model_web_tour_tour_step,base.group_system,1,1,1,1
web_tour.access_web_tour_tour_step_readonly,access_web_tour_tour_step_readonly,web_tour.model_web_tour_tour_step,base.group_user,1,0,0,0

```

## File: static\src\@types\registries.d.ts

```ts
declare module "registries" {
    interface TourStep {
        content: string;
        trigger: string;
        run: string | (() => (void | Promise<void>));
    }

    export interface ToursRegistryShape {
        test?: boolean;
        url: string;
        steps(): TourStep[];
    }

    export interface GlobalRegistryCategories {
        "web_tour.tours": ToursRegistryShape;
    }
}

```

## File: static\src\tour_pointer\tour_pointer.js

```javascript
/** @odoo-module **/

import { Component, useEffect, useRef } from "@odoo/owl";
import { usePosition } from "@web/core/position/position_hook";

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
                isZone: { type: Boolean, optional: true },
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
        Object.defineProperty(positionOptions, "position", {
            get: () => this.position,
            enumerable: true,
        });
        const position = usePosition(
            "pointer",
            () => this.props.pointerState.anchor,
            positionOptions
        );
        const rootRef = useRef("pointer");
        const zoneRef = useRef("zone");
        /** @type {DOMREct | null} */
        let dimensions = null;
        let lastMeasuredContent = null;
        let lastOpenState = this.isOpen;
        let lastAnchor;
        let [anchorX, anchorY] = [0, 0];
        useEffect(() => {
            const { el: pointer } = rootRef;
            const { el: zone } = zoneRef;
            if (pointer) {
                const hasContentChanged = lastMeasuredContent !== this.content;
                const hasOpenStateChanged = lastOpenState !== this.isOpen;
                lastOpenState = this.isOpen;

                // Check is the pointed element is a zone
                if (this.props.pointerState.isZone) {
                    const { anchor } = this.props.pointerState;
                    let offsetLeft = 0;
                    let offsetTop = 0;
                    if (document !== anchor.ownerDocument) {
                        const iframe = [...document.querySelectorAll("iframe")].filter(
                            (e) => e.contentDocument === anchor.ownerDocument
                        )[0];
                        offsetLeft = iframe.getBoundingClientRect().left;
                        offsetTop = iframe.getBoundingClientRect().top;
                    }
                    const { left, top, width, height } = anchor.getBoundingClientRect();
                    zone.style.minWidth = width + "px";
                    zone.style.minHeight = height + "px";
                    zone.style.left = left + offsetLeft + "px";
                    zone.style.top = top + offsetTop + "px";
                }

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
        return this.props.pointerState.isOpen && this.content;
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
                t-att-class="{ 'invisible': !isOpen }"
            >
                <t t-out="content" />
            </div>
        </div>
        <div class="o_tour_dropzone position-fixed pe-none" t-if="props.pointerState.isVisible and props.pointerState.isZone" t-ref="zone" style="border: 3px dashed #714b67;"/>
    </t>
</templates>

```

## File: static\src\tour_service\tour_automatic.js

```javascript
import { tourState } from "./tour_state";
import { config as transitionConfig } from "@web/core/transition";
import { TourStepAutomatic } from "./tour_step_automatic";
import { Macro } from "@web/core/macro";
import { browser } from "@web/core/browser/browser";
import { setupEventActions } from "@web/../lib/hoot-dom/helpers/events";
import * as hoot from "@odoo/hoot-dom";

export class TourAutomatic {
    mode = "auto";
    constructor(data) {
        Object.assign(this, data);
        this.steps = this.steps.map((step, index) => new TourStepAutomatic(step, this, index));
        this.config = tourState.getCurrentConfig() || {};
    }

    get currentIndex() {
        return tourState.getCurrentIndex();
    }

    get currentStep() {
        return this.steps[this.currentIndex];
    }

    get debugMode() {
        return this.config.debug !== false;
    }

    start() {
        setupEventActions(document.createElement("div"), { allowSubmit: true });
        const { delayToCheckUndeterminisms, stepDelay } = this.config;
        const macroSteps = this.steps
            .filter((step) => step.index >= this.currentIndex)
            .flatMap((step) => [
                {
                    action: async () => {
                        if (this.debugMode) {
                            console.groupCollapsed(step.describeMe);
                            console.log(step.stringify);
                            if (step.break) {
                                // eslint-disable-next-line no-debugger
                                debugger;
                            }
                        } else {
                            console.log(step.describeMe);
                        }
                        // This delay is important for making the current set of tour tests pass.
                        // IMPROVEMENT: Find a way to remove this delay.
                        await new Promise((resolve) => requestAnimationFrame(resolve));
                        if (stepDelay > 0) {
                            await hoot.delay(stepDelay);
                        }
                    },
                },
                {
                    trigger: step.trigger ? () => step.findTrigger() : null,
                    timeout:
                        step.pause && this.debugMode
                            ? 9999999
                            : step.timeout || this.timeout || 10000,
                    action: async (trigger) => {
                        if (delayToCheckUndeterminisms > 0) {
                            await step.checkForUndeterminisms(trigger, delayToCheckUndeterminisms);
                        }
                        const result = await step.doAction();
                        if (this.debugMode) {
                            console.log(trigger);
                            if (step.skipped) {
                                console.log("This step has been skipped");
                            } else {
                                console.log("This step has run successfully");
                            }
                            console.groupEnd();
                            if (step.pause) {
                                await this.pause();
                            }
                        }
                        tourState.setCurrentIndex(step.index + 1);
                        return result;
                    },
                },
            ]);

        const end = () => {
            delete window.hoot;
            transitionConfig.disabled = false;
            tourState.clear();
            //No need to catch error yet.
            window.addEventListener(
                "error",
                (ev) => {
                    ev.preventDefault();
                    ev.stopImmediatePropagation();
                },
                true
            );
            window.addEventListener(
                "unhandledrejection",
                (ev) => {
                    ev.preventDefault();
                    ev.stopImmediatePropagation();
                },
                true
            );
        };

        this.macro = new Macro({
            name: this.name,
            steps: macroSteps,
            onError: (error) => {
                if (error.type === "Timeout") {
                    this.throwError(...this.currentStep.describeWhyIFailed, error.message);
                } else {
                    this.throwError(error.message);
                }
                end();
            },
            onComplete: () => {
                browser.console.log("tour succeeded");
                // Used to see easily in the python console and to know which tour has been succeeded in suite tours case.
                const succeeded = `║ TOUR ${this.name} SUCCEEDED ║`;
                const msg = [succeeded];
                msg.unshift("╔" + "═".repeat(succeeded.length - 2) + "╗");
                msg.push("╚" + "═".repeat(succeeded.length - 2) + "╝");
                browser.console.log(`\n\n${msg.join("\n")}\n`);
                end();
            },
        });
        if (this.debugMode && this.currentIndex === 0) {
            // Starts the tour with a debugger to allow you to choose devtools configuration.
            // eslint-disable-next-line no-debugger
            debugger;
        }
        transitionConfig.disabled = true;
        window.hoot = hoot;
        this.macro.start();
    }

    get describeWhereIFailed() {
        const offset = 3;
        const start = Math.max(this.currentIndex - offset, 0);
        const end = Math.min(this.currentIndex + offset, this.steps.length - 1);
        const result = [];
        for (let i = start; i <= end; i++) {
            const step = this.steps[i];
            const stepString = step.stringify;
            const text = [stepString];
            if (i === this.currentIndex) {
                const line = "-".repeat(10);
                const failing_step = `${line} FAILED: ${step.describeMe} ${line}`;
                text.unshift(failing_step);
                text.push("-".repeat(failing_step.length));
            }
            result.push(...text);
        }
        return result.join("\n");
    }

    /**
     * @param {string} [error]
     */
    throwError(...args) {
        console.groupEnd();
        tourState.setCurrentTourOnError();
        // console.error notifies the test runner that the tour failed.
        browser.console.error([`FAILED: ${this.currentStep.describeMe}.`, ...args].join("\n"));
        // The logged text shows the relative position of the failed step.
        // Useful for finding the failed step.
        browser.console.dir(this.describeWhereIFailed);
        if (this.debugMode) {
            // eslint-disable-next-line no-debugger
            debugger;
        }
    }

    async pause() {
        const styles = [
            "background: black; color: white; font-size: 14px",
            "background: black; color: orange; font-size: 14px",
        ];
        console.log(
            `%cTour is paused. Use %cplay()%c to continue.`,
            styles[0],
            styles[1],
            styles[0]
        );
        await new Promise((resolve) => {
            window.play = () => {
                resolve();
                delete window.play;
            };
        });
    }
}

```

## File: static\src\tour_service\tour_helpers.js

```javascript
import * as hoot from "@odoo/hoot-dom";
import { waitForStable } from "@web/core/macro";

export class TourHelpers {
    /**
     * @typedef {string|Node} Selector
     */

    constructor(anchor) {
        this.anchor = anchor;
        this.delay = 20;
    }

    /**
     * Ensures that the given {@link Selector} is checked.
     * @description
     * If it is not checked, a click is triggered on the input.
     * If the input is still not checked after the click, an error is thrown.
     *
     * @param {string|Node} selector
     * @example
     *  run: "check", //Checks the action element
     * @example
     *  run: "check input[type=checkbox]", // Checks the selector
     */
    async check(selector) {
        const element = this._get_action_element(selector);
        await hoot.check(element);
    }

    /**
     * Clears the **value** of the **{@link Selector}**.
     * @description
     * This is done using the following sequence:
     * - pressing "Control" + "A" to select the whole value;
     * - pressing "Backspace" to delete the value;
     * - (optional) triggering a "change" event by pressing "Enter".
     *
     * @param {Selector} selector
     * @example
     *  run: "clear", // Clears the value of the action element
     * @example
     *  run: "clear input#my_input", // Clears the value of the selector
     */
    async clear(selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        await hoot.clear();
    }

    /**
     * Performs a click sequence on the given **{@link Selector}**
     * @description Let's see more informations about click sequence here: {@link hoot.click}
     * @param {Selector} selector
     * @example
     *  run: "click", // Click on the action element
     * @example
     *  run: "click .o_rows:first", // Click on the selector
     */
    async click(selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
    }

    /**
     * Performs two click sequences on the given **{@link Selector}**.
     * @description Let's see more informations about click sequence here: {@link hoot.dblclick}
     * @param {Selector} selector
     * @example
     *  run: "dblclick", // Double click on the action element
     * @example
     *  run: "dblclick .o_rows:first", // Double click on the selector
     */
    async dblclick(selector) {
        const element = this._get_action_element(selector);
        await hoot.dblclick(element);
    }

    /**
     * Starts a drag sequence on the active element (anchor) and drop it on the given **{@link Selector}**.
     * @param {Selector} selector
     * @param {hoot.PointerOptions} options
     * @example
     *  run: "drag_and_drop .o_rows:first", // Drag the active element and drop it in the selector
     * @example
     *  async run(helpers) {
     *      await helpers.drag_and_drop(".o_rows:first", {
     *          position: {
     *              top: 40,
     *              left: 5,
     *          },
     *          relative: true,
     *      });
     *  }
     */
    async drag_and_drop(selector, options) {
        if (typeof options !== "object") {
            options = { position: "top", relative: true };
        }
        const dragEffectDelay = async () => {
            await new Promise((resolve) => requestAnimationFrame(resolve));
            await new Promise((resolve) => setTimeout(resolve, this.delay));
        };
        const element = this.anchor;
        const { drop, moveTo } = await hoot.drag(element);
        await dragEffectDelay();
        await hoot.hover(element, {
            position: {
                top: 20,
                left: 20,
            },
            relative: true,
        });
        await dragEffectDelay();
        const target = await hoot.waitFor(selector, {
            visible: true,
            timeout: 1000,
        });
        await moveTo(target, options);
        await dragEffectDelay();
        await drop();
        await dragEffectDelay();
    }

    /**
     * Edit input or textarea given by **{@link selector}**
     * @param {string} text
     * @param {Selector} selector
     * @example
     *  run: "edit Hello Mr. Doku",
     */
    async edit(text, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        await hoot.edit(text);
    }

    /**
     * Edit only editable wysiwyg element given by **{@link Selector}**
     * @param {string} text
     * @param {Selector} selector
     */
    async editor(text, selector) {
        const element = this._get_action_element(selector);
        const InEditor = Boolean(element.closest(".odoo-editor-editable"));
        if (!InEditor) {
            throw new Error("run 'editor' always on an element in an editor");
        }
        await hoot.click(element);
        this._set_range(element, "start");
        await hoot.keyDown("_");
        element.textContent = text;
        await hoot.manuallyDispatchProgrammaticEvent(element, "input");
        this._set_range(element, "stop");
        await hoot.keyUp("_");
        await hoot.manuallyDispatchProgrammaticEvent(element, "change");
    }

    /**
     * Fills the **{@link Selector}** with the given `value`.
     * @description This helper is intended for `<input>` and `<textarea>` elements,
     * with the exception of `"checkbox"` and `"radio"` types, which should be
     * selected using the {@link check} helper.
     * In tour, it's mainly usefull for autocomplete components.
     * @param {string} value
     * @param {Selector} selector
     */
    async fill(value, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        await hoot.fill(value);
    }

    /**
     * Performs a hover sequence on the given **{@link Selector}**.
     * @param {Selector} selector
     * @example
     *  run: "hover",
     */
    async hover(selector) {
        const element = this._get_action_element(selector);
        await hoot.hover(element);
    }

    /**
     * Only for input[type="range"]
     * @param {string|number} value
     * @param {Selector} selector
     */
    async range(value, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        await hoot.setInputRange(element, value);
    }

    /**
     * Performs a keyboard event sequence.
     * @example
     *  run : "press Enter",
     */
    async press(...args) {
        await hoot.press(args.flatMap((arg) => typeof arg === "string" && arg.split("+")));
    }

    /**
     * Performs a selection event sequence on **{@link Selector}**. This helper is intended
     * for `<select>` elements only.
     * @description Select the option by its value
     * @param {string} value
     * @param {Selector} selector
     * @example
     * run(helpers) => {
     *  helpers.select("Kevin17", "select#mySelect");
     * },
     * @example
     * run: "select Foden47",
     */
    async select(value, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        await hoot.select(value, { target: element });
    }

    /**
     * Performs a selection event sequence on **{@link Selector}**
     * @description Select the option by its index
     * @param {number} index starts at 0
     * @param {Selector} selector
     * @example
     *  run: "selectByIndex 2", //Select the third option
     */
    async selectByIndex(index, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        const value = hoot.queryValue(`option:eq(${index})`, { root: element });
        if (value) {
            await hoot.select(value, { target: element });
            await hoot.manuallyDispatchProgrammaticEvent(element, "input");
        }
    }

    /**
     * Performs a selection event sequence on **{@link Selector}**
     * @description Select option(s) by there labels
     * @param {string|RegExp} contains
     * @param {Selector} selector
     * @example
     *  run: "selectByLabel Jeremy Doku", //Select all options where label contains Jeremy Doku
     */
    async selectByLabel(contains, selector) {
        const element = this._get_action_element(selector);
        await hoot.click(element);
        const values = hoot.queryAllValues(`option:contains(${contains})`, { root: element });
        await hoot.select(values, { target: element });
    }

    /**
     * Ensures that the given {@link Selector} is unchecked.
     * @description
     * If it is checked, a click is triggered on the input.
     * If the input is still checked after the click, an error is thrown.
     *
     * @param {string|Node} selector
     * @example
     *  run: "uncheck", // Unchecks the action element
     * @example
     *  run: "uncheck input[type=checkbox]", // Unchecks the selector
     */
    async uncheck(selector) {
        const element = this._get_action_element(selector);
        await hoot.uncheck(element);
    }

    /**
     * Navigate to {@link url}.
     *
     * @param {string} url
     * @example
     *  run: "goToUrl /shop", // Go to /shop
     */
    async goToUrl(url) {
        const linkEl = document.createElement("a");
        linkEl.href = url;
        //We want DOM is stable before quit it.
        await waitForStable();
        await hoot.click(linkEl);
    }

    /**
     * Get Node for **{@link Selector}**
     * @param {Selector} selector
     * @returns {Node}
     * @default this.anchor
     */
    _get_action_element(selector) {
        if (typeof selector === "string" && selector.length) {
            const nodes = hoot.queryAll(selector);
            return nodes.find(hoot.isVisible) || nodes.at(0);
        } else if (typeof selector === "object" && Boolean(selector?.nodeType)) {
            return selector;
        }
        return this.anchor;
    }

    // Useful for wysiwyg editor.
    _set_range(element, start_or_stop) {
        function _node_length(node) {
            if (node.nodeType === Node.TEXT_NODE) {
                return node.nodeValue.length;
            } else {
                return node.childNodes.length;
            }
        }
        const selection = element.ownerDocument.getSelection();
        selection.removeAllRanges();
        const range = new Range();
        let node = element;
        let length = 0;
        if (start_or_stop === "start") {
            while (node.firstChild) {
                node = node.firstChild;
            }
        } else {
            while (node.lastChild) {
                node = node.lastChild;
            }
            length = _node_length(node);
        }
        range.setStart(node, length);
        range.setEnd(node, length);
        selection.addRange(range);
    }
}

```

## File: static\src\tour_service\tour_interactive.js

```javascript
import { tourState } from "@web_tour/tour_service/tour_state";
import { debounce } from "@web/core/utils/timing";
import { getScrollParent } from "@web_tour/tour_service/tour_utils";
import * as hoot from "@odoo/hoot-dom";
import { utils } from "@web/core/ui/ui_service";
import { TourStep } from "./tour_step";
import { MacroMutationObserver } from "@web/core/macro";

/**
 * @typedef ConsumeEvent
 * @property {string} name
 * @property {Element} target
 * @property {(ev: Event) => boolean} conditional
 */

export class TourInteractive {
    mode = "manual";
    currentAction;
    currentActionIndex;
    anchorEls = [];
    removeListeners = () => {};

    /**
     * @param {Tour} data
     */
    constructor(data) {
        Object.assign(this, data);
        this.steps = this.steps.map((step) => new TourStep(step, this));
        this.actions = this.steps.flatMap((s) => this.getSubActions(s));
    }

    /**
     * @param {import("@web_tour/tour_pointer/tour_pointer").TourPointer} pointer
     * @param {Function} onTourEnd
     */
    start(pointer, onTourEnd) {
        this.pointer = pointer;
        this.debouncedToggleOpen = debounce(this.pointer.showContent, 50, true);
        this.onTourEnd = onTourEnd;
        this.observer = new MacroMutationObserver(() => this._onMutation());
        this.observer.observe(document.body);
        this.currentActionIndex = tourState.getCurrentIndex();
        this.play();
    }

    backward() {
        let tempIndex = this.currentActionIndex;
        let tempAction,
            tempAnchors = [];
        while (!tempAnchors.length && tempIndex >= 0) {
            tempIndex--;
            tempAction = this.actions.at(tempIndex);
            if (!tempAction.step.active) {
                continue;
            }
            tempAnchors = tempAction && this.findTriggers(tempAction.anchor);
        }

        if (tempIndex >= 0) {
            this.currentActionIndex = tempIndex;
            this.play();
        }
    }

    /**
     * @returns {HTMLElement[]}
     */
    findTriggers(anchor) {
        if (!anchor) {
            anchor = this.currentAction.anchor;
        }

        return anchor
            .split(/,\s*(?![^(]*\))/)
            .map((part) => hoot.queryFirst(part, { visible: true }))
            .filter((el) => !!el)
            .map((el) => this.getAnchorEl(el, this.currentAction.event))
            .filter((el) => !!el);
    }

    play() {
        this.removeListeners();
        if (this.currentActionIndex === this.actions.length) {
            this.observer.disconnect();
            this.onTourEnd();
            return;
        }

        this.currentAction = this.actions.at(this.currentActionIndex);

        if (!this.currentAction.step.active || this.currentAction.event === "warn") {
            if (this.currentAction.event === "warn") {
                console.warn(`Step '${this.currentAction.anchor}' ignored.`);
            }
            this.currentActionIndex++;
            this.play();
            return;
        }

        console.log(this.currentAction.event, this.currentAction.anchor);

        tourState.setCurrentIndex(this.currentActionIndex);
        this.anchorEls = this.findTriggers();
        this.setActionListeners();
        this.updatePointer();
    }

    updatePointer() {
        if (this.anchorEls.length) {
            this.pointer.pointTo(
                this.anchorEls[0],
                this.currentAction.pointerInfo,
                this.currentAction.event === "drop"
            );
            this.pointer.setState({
                onMouseEnter: () => this.debouncedToggleOpen(true),
                onMouseLeave: () => this.debouncedToggleOpen(false),
            });
        }
    }

    setActionListeners() {
        const cleanups = this.anchorEls.flatMap((anchorEl, index) => {
            const toListen = {
                anchorEl,
                consumeEvents: this.getConsumeEventType(anchorEl, this.currentAction.event),
                onConsume: () => {
                    this.pointer.hide();
                    this.currentActionIndex++;
                    this.play();
                },
                onError: () => {
                    if (this.currentAction.event === "drop") {
                        this.pointer.hide();
                        this.currentActionIndex--;
                        this.play();
                    }
                },
            };
            if (index === 0) {
                return this.setupListeners({
                    ...toListen,
                    onMouseEnter: () => this.pointer.showContent(true),
                    onMouseLeave: () => this.pointer.showContent(false),
                    onScroll: () => this.updatePointer(),
                });
            } else {
                return this.setupListeners({
                    ...toListen,
                    onScroll: () => {},
                });
            }
        });
        this.removeListeners = () => {
            this.anchorEls = [];
            while (cleanups.length) {
                cleanups.pop()();
            }
        };
    }

    /**
     * @param {HTMLElement} params.anchorEl
     * @param {import("./tour_utils").ConsumeEvent[]} params.consumeEvents
     * @param {() => void} params.onMouseEnter
     * @param {() => void} params.onMouseLeave
     * @param {(ev: Event) => any} params.onScroll
     * @param {(ev: Event) => any} params.onConsume
     * @param {() => any} params.onError
     */
    setupListeners({
        anchorEl,
        consumeEvents,
        onMouseEnter,
        onMouseLeave,
        onScroll,
        onConsume,
        onError = () => {},
    }) {
        consumeEvents = consumeEvents.map((c) => ({
            target: c.target,
            type: c.name,
            listener: function (ev) {
                if (!c.conditional || c.conditional(ev)) {
                    onConsume();
                } else {
                    onError();
                }
            },
        }));

        for (const consume of consumeEvents) {
            consume.target.addEventListener(consume.type, consume.listener, true);
        }
        anchorEl.addEventListener("mouseenter", onMouseEnter);
        anchorEl.addEventListener("mouseleave", onMouseLeave);

        const cleanups = [
            () => {
                for (const consume of consumeEvents) {
                    consume.target.removeEventListener(consume.type, consume.listener, true);
                }
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

        return cleanups;
    }

    /**
     *
     * @param {import("./tour_service").TourStep} step
     * @returns {{
     *  event: string,
     *  anchor: string,
     *  pointerInfo: { tooltipPosition: string?, content: string? },
     * }[]}
     */
    getSubActions(step) {
        const actions = [];
        if (!step.run || typeof step.run === "function") {
            actions.push({
                step,
                event: "warn",
                anchor: step.trigger,
            });
            return actions;
        }

        for (const todo of step.run.split("&&")) {
            const m = String(todo)
                .trim()
                .match(/^(?<action>\w*) *\(? *(?<arguments>.*?)\)?$/);

            let action = m.groups?.action;
            const anchor = m.groups?.arguments || step.trigger;
            const pointerInfo = {
                content: step.content,
                tooltipPosition: step.tooltipPosition,
            };

            if (action === "drag_and_drop") {
                actions.push({
                    step,
                    event: "drag",
                    anchor: step.trigger,
                    pointerInfo,
                });
                action = "drop";
            }

            actions.push({
                step,
                event: action,
                anchor: action === "edit" ? step.trigger : anchor,
                pointerInfo,
            });
        }

        return actions;
    }

    /**
     * @param {HTMLElement} [element]
     * @param {string} [runCommand]
     * @returns {ConsumeEvent[]}
     */
    getConsumeEventType(element, runCommand) {
        const consumeEvents = [];
        if (runCommand === "click") {
            consumeEvents.push({
                name: "click",
                target: element,
            });

            // Click on a field widget with an autocomplete should be also completed with a selection though Enter or Tab
            // This case is for the steps that click on field_widget
            if (element.querySelector(".o-autocomplete--input")) {
                consumeEvents.push({
                    name: "keydown",
                    target: element.querySelector(".o-autocomplete--input"),
                    conditional: (ev) =>
                        ["Tab", "Enter"].includes(ev.key) &&
                        ev.target.parentElement.querySelector(
                            ".o-autocomplete--dropdown-item .ui-state-active"
                        ),
                });
            }

            // Click on an element of a dropdown should be also completed with a selection though Enter or Tab
            // This case is for the steps that click on a dropdown-item
            if (element.closest(".o-autocomplete--dropdown-menu")) {
                consumeEvents.push({
                    name: "keydown",
                    target: element.closest(".o-autocomplete").querySelector("input"),
                    conditional: (ev) => ["Tab", "Enter"].includes(ev.key),
                });
            }

            // Press enter on a button do the same as a click
            if (element.tagName === "BUTTON") {
                consumeEvents.push({
                    name: "keydown",
                    target: element,
                    conditional: (ev) => ev.key === "Enter",
                });

                // Pressing enter in the input group does the same as clicking on the button
                if (element.closest(".input-group")) {
                    for (const inputEl of element.parentElement.querySelectorAll("input")) {
                        consumeEvents.push({
                            name: "keydown",
                            target: inputEl,
                            conditional: (ev) => ev.key === "Enter",
                        });
                    }
                }
            }
        }

        if (["fill", "edit"].includes(runCommand)) {
            if (
                utils.isSmall() &&
                element.closest(".o_field_widget")?.matches(".o_field_many2one, .o_field_many2many")
            ) {
                consumeEvents.push({
                    name: "click",
                    target: element,
                });
            } else {
                consumeEvents.push({
                    name: "input",
                    target: element,
                });
                if (element.classList.contains("o-autocomplete--input")) {
                    consumeEvents.push({
                        name: "keydown",
                        target: element,
                        conditional: (ev) => {
                            if (
                                ["Tab", "Enter"].includes(ev.key) &&
                                ev.target.parentElement.querySelector(
                                    ".o-autocomplete--dropdown-item .ui-state-active"
                                )
                            ) {
                                const nextStep = this.actions.at(this.currentActionIndex + 1);
                                if (
                                    this.findTriggers(nextStep.anchor)
                                        .at(0)
                                        ?.closest(".o-autocomplete--dropdown-item")
                                ) {
                                    // Skip the next step if the next one is a click on a dropdown item
                                    this.currentActionIndex++;
                                }
                                return true;
                            }
                        },
                    });
                    consumeEvents.push({
                        name: "click",
                        target: element.ownerDocument,
                        conditional: (ev) => {
                            if (ev.target.closest(".o-autocomplete--dropdown-item")) {
                                const nextStep = this.actions.at(this.currentActionIndex + 1);
                                if (
                                    this.findTriggers(nextStep.anchor)
                                        .at(0)
                                        ?.closest(".o-autocomplete--dropdown-item")
                                ) {
                                    // Skip the next step if the next one is a click on a dropdown item
                                    this.currentActionIndex++;
                                }
                                return true;
                            }
                        },
                    });
                }
            }
        }

        // Drag & drop run command
        if (runCommand === "drag") {
            consumeEvents.push({
                name: "pointerdown",
                target: element,
            });
        }

        if (runCommand === "drop") {
            consumeEvents.push({
                name: "pointerup",
                target: element.ownerDocument,
                conditional: (ev) =>
                    element.ownerDocument
                        .elementsFromPoint(ev.clientX, ev.clientY)
                        .includes(element),
            });
            consumeEvents.push({
                name: "drop",
                target: element.ownerDocument,
                conditional: (ev) =>
                    element.ownerDocument
                        .elementsFromPoint(ev.clientX, ev.clientY)
                        .includes(element),
            });
        }

        return consumeEvents;
    }

    /**
     * Returns the element that will be used in listening to the `consumeEvent`.
     * @param {HTMLElement} el
     * @param {string} consumeEvent
     */
    getAnchorEl(el, consumeEvent) {
        if (consumeEvent === "drag") {
            // jQuery-ui draggable triggers 'drag' events on the .ui-draggable element,
            // but the tip is attached to the .ui-draggable-handle element which may
            // be one of its children (or the element itself
            return el.closest(
                ".ui-draggable, .o_draggable, .o_we_draggable, .o-draggable, [draggable='true']"
            );
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

    _onMutation() {
        if (this.currentAction) {
            const tempAnchors = this.findTriggers();
            if (
                tempAnchors.length &&
                (tempAnchors.some((a) => !this.anchorEls.includes(a)) ||
                    this.anchorEls.some((a) => !tempAnchors.includes(a)))
            ) {
                this.removeListeners();
                this.anchorEls = tempAnchors;
                this.setActionListeners();
            } else if (!tempAnchors.length && this.anchorEls.length) {
                this.pointer.hide();
                if (
                    !hoot.queryFirst(".o_home_menu", { visible: true }) &&
                    !hoot.queryFirst(".dropdown-item.o_loading", { visible: true })
                ) {
                    this.backward();
                }
                return;
            }
            this.updatePointer();
        }
    }
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
 * @typedef {import("@web/core/position/position_hook").Direction} Direction
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
 * @property {boolean} isZone
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
                const scrollParentElement =
                    getScrollParent(this.currentTarget) || document.documentElement;
                const targetBounds = this.currentTarget.getBoundingClientRect();
                if (targetBounds.bottom > scrollParentElement.clientHeight) {
                    this._targetPosition = "out-below";
                } else if (targetBounds.top < 0) {
                    this._targetPosition = "out-above";
                } else if (targetBounds.left < 0) {
                    this._targetPosition = "out-left";
                } else if (targetBounds.right > scrollParentElement.clientWidth) {
                    this._targetPosition = "out-right";
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
     * @param {boolean} [isZone] will border de zone. e.g.: a dropzone
     */
    const pointTo = (anchor, step, isZone) => {
        intersection.setTarget(anchor);
        if (anchor) {
            let { tooltipPosition, content } = step;
            switch (intersection.targetPosition) {
                case "unknown": {
                    // Do nothing for unknown target position.
                    break;
                }
                case "in": {
                    if (document.body.contains(floatingAnchor)) {
                        floatingAnchor.remove();
                    }
                    setState({
                        anchor,
                        content,
                        isZone,
                        onClick: null,
                        position: tooltipPosition,
                        isVisible: true,
                    });
                    break;
                }
                default: {
                    const onClick = () => {
                        anchor.scrollIntoView({ behavior: "smooth", block: "nearest" });
                        hide();
                    };

                    const scrollParent = getScrollParent(anchor);
                    if (!scrollParent) {
                        setState({
                            anchor,
                            content,
                            isZone,
                            onClick: null,
                            position: tooltipPosition,
                            isVisible: true,
                        });
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
                    if (intersection.targetPosition === "out-below") {
                        tooltipPosition = "top";
                        content = _t("Scroll down to reach the next step.");
                        floatingAnchor.style.top = `${y + height - TourPointer.height}px`;
                        floatingAnchor.style.left = `${x + width / 2}px`;
                    } else if (intersection.targetPosition === "out-above") {
                        tooltipPosition = "bottom";
                        content = _t("Scroll up to reach the next step.");
                        floatingAnchor.style.top = `${y + TourPointer.height}px`;
                        floatingAnchor.style.left = `${x + width / 2}px`;
                    }
                    if (intersection.targetPosition === "out-left") {
                        tooltipPosition = "right";
                        content = _t("Scroll left to reach the next step.");
                        floatingAnchor.style.top = `${y + height / 2}px`;
                        floatingAnchor.style.left = `${x + TourPointer.width}px`;
                    } else if (intersection.targetPosition === "out-right") {
                        tooltipPosition = "left";
                        content = _t("Scroll right to reach the next step.");
                        floatingAnchor.style.top = `${y + height / 2}px`;
                        floatingAnchor.style.left = `${x + width - TourPointer.width}px`;
                    }
                    if (!document.contains(floatingAnchor)) {
                        document.body.appendChild(floatingAnchor);
                    }
                    setState({
                        anchor: floatingAnchor,
                        content,
                        onClick,
                        position: tooltipPosition,
                        isZone,
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

    return { state, setState, showContent, pointTo, hide, destroy };
}

```

## File: static\src\tour_service\tour_service.js

```javascript
/** @odoo-module **/

import { markup, whenReady, validate } from "@odoo/owl";
import { browser } from "@web/core/browser/browser";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { session } from "@web/session";
import { TourPointer } from "../tour_pointer/tour_pointer";
import { createPointerState } from "./tour_pointer_state";
import { tourState } from "./tour_state";
import { TourInteractive } from "./tour_interactive";
import { TourAutomatic } from "./tour_automatic";
import { callWithUnloadCheck } from "./tour_utils";
import {
    TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY,
    TourRecorder,
} from "@web_tour/tour_service/tour_recorder/tour_recorder";
import { redirect } from "@web/core/utils/urls";
import { tourRecorderState } from "@web_tour/tour_service/tour_recorder/tour_recorder_state";

const StepSchema = {
    id: { type: [String], optional: true },
    content: { type: [String, Object], optional: true }, //allow object(_t && markup)
    debugHelp: { type: String, optional: true },
    isActive: { type: Array, element: String, optional: true },
    run: { type: [String, Function, Boolean], optional: true },
    timeout: {
        optional: true,
        validate(value) {
            return value >= 0 && value <= 60000;
        },
    },
    tooltipPosition: {
        optional: true,
        validate(value) {
            return ["top", "bottom", "left", "right"].includes(value);
        },
    },
    trigger: { type: String },
    //ONLY IN DEBUG MODE
    pause: { type: Boolean, optional: true },
    break: { type: Boolean, optional: true },
};

const TourSchema = {
    name: { type: String, optional: true },
    steps: Function,
    url: { type: String, optional: true },
    wait_for: { type: [Function, Object], optional: true },
};

registry.category("web_tour.tours").addValidation(TourSchema);
const userMenuRegistry = registry.category("user_menuitems");

export const tourService = {
    // localization dependency to make sure translations used by tours are loaded
    dependencies: ["orm", "effect", "overlay", "localization"],
    start: async (_env, { orm, effect, overlay }) => {
        await whenReady();
        let toursEnabled = session?.tour_enabled;
        const tourRegistry = registry.category("web_tour.tours");
        const pointer = createPointerState();
        pointer.stop = () => {};

        userMenuRegistry.add("web_tour.tour_enabled", () => ({
            type: "switch",
            id: "web_tour.tour_enabled",
            description: _t("Onboarding"),
            callback: async () => {
                tourState.clear();
                toursEnabled = await orm.call("res.users", "switch_tour_enabled", [!toursEnabled]);
                browser.location.reload();
            },
            isChecked: toursEnabled,
            sequence: 30,
        }));

        function getTourFromRegistry(tourName) {
            if (!tourRegistry.contains(tourName)) {
                return;
            }
            const tour = tourRegistry.get(tourName);
            return {
                ...tour,
                steps: tour.steps(),
                name: tourName,
                wait_for: tour.wait_for || Promise.resolve(),
            };
        }

        async function getTourFromDB(tourName) {
            const tour = await orm.call("web_tour.tour", "get_tour_json_by_name", [tourName]);
            if (!tour) {
                throw new Error(`Tour '${tourName}' is not found in the database.`);
            }

            if (!tour.steps.length && tourRegistry.contains(tour.name)) {
                tour.steps = tourRegistry.get(tour.name).steps();
            }

            return tour;
        }

        function validateStep(step) {
            try {
                validate(step, StepSchema);
            } catch (error) {
                console.error(
                    `Error in schema for TourStep ${JSON.stringify(step, null, 4)}\n${
                        error.message
                    }`
                );
            }
        }

        async function startTour(tourName, options = {}) {
            pointer.stop();
            const tourFromRegistry = getTourFromRegistry(tourName);

            if (!tourFromRegistry && !options.fromDB) {
                // Sometime tours are not loaded depending on the modules.
                // For example, point_of_sale do not load all tours assets.
                return;
            }

            const tour = options.fromDB ? { name: tourName, url: options.url } : tourFromRegistry;
            if (!session.is_public && !toursEnabled && options.mode === "manual") {
                toursEnabled = await orm.call("res.users", "switch_tour_enabled", [!toursEnabled]);
            }

            let tourConfig = {
                delayToCheckUndeterminisms: 0,
                stepDelay: 0,
                keepWatchBrowser: false,
                mode: "auto",
                showPointerDuration: 0,
                debug: false,
                redirect: true,
            };

            tourConfig = Object.assign(tourConfig, options);
            tourState.setCurrentConfig(tourConfig);
            tourState.setCurrentTour(tour.name);
            tourState.setCurrentIndex(0);

            const willUnload = callWithUnloadCheck(() => {
                if (tour.url && tourConfig.startUrl != tour.url && tourConfig.redirect) {
                    redirect(tour.url);
                }
            });
            if (!willUnload) {
                resumeTour();
            }
        }

        async function resumeTour() {
            const tourName = tourState.getCurrentTour();
            const tourConfig = tourState.getCurrentConfig();

            let tour = getTourFromRegistry(tourName);
            if (tourConfig.fromDB) {
                tour = await getTourFromDB(tourName);
            }
            if (!tour) {
                return;
            }

            tour.steps.forEach((step) => validateStep(step));
            pointer.stop = overlay.add(
                TourPointer,
                {
                    pointerState: pointer.state,
                    bounce: !(tourConfig.mode === "auto" && tourConfig.keepWatchBrowser),
                },
                {
                    sequence: 1100, // sequence based on bootstrap z-index values.
                }
            );

            if (tourConfig.mode === "auto") {
                new TourAutomatic(tour).start();
            } else {
                new TourInteractive(tour).start(pointer, async () => {
                    pointer.stop();
                    tourState.clear();
                    browser.console.log("tour succeeded");
                    let message = tourConfig.rainbowManMessage || tour.rainbowManMessage;
                    if (message) {
                        message = window.DOMPurify.sanitize(tourConfig.rainbowManMessage);
                        effect.add({
                            type: "rainbow_man",
                            message: markup(message),
                        });
                    }

                    const nextTour = await orm.call("web_tour.tour", "consume", [tour.name]);
                    if (nextTour) {
                        startTour(nextTour.name, {
                            mode: "manual",
                            redirect: false,
                            rainbowManMessage: nextTour.rainbowManMessage,
                        });
                    }
                });
            }
        }

        function startTourRecorder() {
            if (!browser.localStorage.getItem(TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY)) {
                const remove = overlay.add(
                    TourRecorder,
                    {
                        onClose: () => {
                            remove();
                            browser.localStorage.removeItem(TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY);
                            tourRecorderState.clear();
                        },
                    },
                    { sequence: 99999 }
                );
            }
            browser.localStorage.setItem(TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY, "1");
        }

        if (!window.frameElement) {
            const paramsTourName = new URLSearchParams(browser.location.search).get("tour");
            if (paramsTourName) {
                startTour(paramsTourName, { mode: "manual", fromDB: true });
            }

            if (tourState.getCurrentTour()) {
                if (tourState.getCurrentConfig().mode === "auto" || toursEnabled) {
                    resumeTour();
                } else {
                    tourState.clear();
                }
            } else if (session.current_tour) {
                startTour(session.current_tour.name, {
                    mode: "manual",
                    redirect: false,
                    rainbowManMessage: session.current_tour.rainbowManMessage,
                });
            }

            if (
                browser.localStorage.getItem(TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY) &&
                !session.is_public
            ) {
                const remove = overlay.add(
                    TourRecorder,
                    {
                        onClose: () => {
                            remove();
                            browser.localStorage.removeItem(TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY);
                            tourRecorderState.clear();
                        },
                    },
                    { sequence: 99999 }
                );
            }
        }

        odoo.startTour = startTour;
        odoo.isTourReady = (tourName) => getTourFromRegistry(tourName).wait_for.then(() => true);

        return {
            startTour,
            startTourRecorder,
        };
    },
};

registry.category("services").add("tour_service", tourService);

```

## File: static\src\tour_service\tour_state.js

```javascript
/** @odoo-module **/

import { browser } from "@web/core/browser/browser";

const CURRENT_TOUR_LOCAL_STORAGE = "current_tour";
const CURRENT_TOUR_CONFIG_LOCAL_STORAGE = "current_tour.config";
const CURRENT_TOUR_INDEX_LOCAL_STORAGE = "current_tour.index";
const CURRENT_TOUR_ON_ERROR_LOCAL_STORAGE = "current_tour.on_error";

/**
 * Wrapper around localStorage for persistence of the running tours.
 * Useful for resuming running tours when the page refreshed.
 */
export const tourState = {
    getCurrentTour() {
        return browser.localStorage.getItem(CURRENT_TOUR_LOCAL_STORAGE);
    },
    setCurrentTour(tourName) {
        browser.localStorage.setItem(CURRENT_TOUR_LOCAL_STORAGE, tourName);
    },
    getCurrentIndex() {
        const index = browser.localStorage.getItem(CURRENT_TOUR_INDEX_LOCAL_STORAGE, "0");
        return parseInt(index, 10);
    },
    setCurrentIndex(index) {
        browser.localStorage.setItem(CURRENT_TOUR_INDEX_LOCAL_STORAGE, index.toString());
    },
    getCurrentConfig() {
        const config = browser.localStorage.getItem(CURRENT_TOUR_CONFIG_LOCAL_STORAGE, "{}");
        return JSON.parse(config);
    },
    setCurrentConfig(config) {
        config = JSON.stringify(config);
        browser.localStorage.setItem(CURRENT_TOUR_CONFIG_LOCAL_STORAGE, config);
    },
    getCurrentTourOnError() {
        return browser.localStorage.getItem(CURRENT_TOUR_ON_ERROR_LOCAL_STORAGE);
    },
    setCurrentTourOnError() {
        browser.localStorage.setItem(CURRENT_TOUR_ON_ERROR_LOCAL_STORAGE, "1");
    },
    clear() {
        browser.localStorage.removeItem(CURRENT_TOUR_ON_ERROR_LOCAL_STORAGE);
        browser.localStorage.removeItem(CURRENT_TOUR_CONFIG_LOCAL_STORAGE);
        browser.localStorage.removeItem(CURRENT_TOUR_INDEX_LOCAL_STORAGE);
        browser.localStorage.removeItem(CURRENT_TOUR_LOCAL_STORAGE);
    },
};

```

## File: static\src\tour_service\tour_step.js

```javascript
import { session } from "@web/session";
import { utils } from "@web/core/ui/ui_service";
import * as hoot from "@odoo/hoot-dom";
import { pick } from "@web/core/utils/objects";

/**
 * @typedef TourStep
 * @property {"enterprise"|"community"|"mobile"|"desktop"|HootSelector[][]} isActive Active the step following {@link isActiveStep} filter
 * @property {string} [id]
 * @property {HootSelector} trigger The node on which the action will be executed.
 * @property {string} [content] Description of the step.
 * @property {"top" | "bottom" | "left" | "right"} [position] The position where the UI helper is shown.
 * @property {RunCommand} [run] The action to perform when trigger conditions are verified.
 * @property {number} [timeout] By default, when the trigger node isn't found after 10000 milliseconds, it throws an error.
 * You can change this value to lengthen or shorten the time before the error occurs [ms].
 */
export class TourStep {
    constructor(data, tour) {
        Object.assign(this, data);
        this.tour = tour;
    }

    /**
     * Check if a step is active dependant on step.isActive property
     * Note that when step.isActive is not defined, the step is active by default.
     * When a step is not active, it's just skipped and the tour continues to the next step.
     */
    get active() {
        this.checkHasTour();
        const mode = this.tour.mode;
        const isSmall = utils.isSmall();
        const standardKeyWords = ["enterprise", "community", "mobile", "desktop", "auto", "manual"];
        const isActiveArray = Array.isArray(this.isActive) ? this.isActive : [];
        if (isActiveArray.length === 0) {
            return true;
        }
        const selectors = isActiveArray.filter((key) => !standardKeyWords.includes(key));
        if (selectors.length) {
            // if one of selectors is not found, step is skipped
            for (const selector of selectors) {
                const el = hoot.queryFirst(selector);
                if (!el) {
                    return false;
                }
            }
        }
        const checkMode =
            isActiveArray.includes(mode) ||
            (!isActiveArray.includes("manual") && !isActiveArray.includes("auto"));
        const edition =
            (session.server_version_info || "").at(-1) === "e" ? "enterprise" : "community";
        const checkEdition =
            isActiveArray.includes(edition) ||
            (!isActiveArray.includes("enterprise") && !isActiveArray.includes("community"));
        const onlyForMobile = isActiveArray.includes("mobile") && isSmall;
        const onlyForDesktop = isActiveArray.includes("desktop") && !isSmall;
        const checkDevice =
            onlyForMobile ||
            onlyForDesktop ||
            (!isActiveArray.includes("mobile") && !isActiveArray.includes("desktop"));
        return checkEdition && checkDevice && checkMode;
    }

    checkHasTour() {
        if (!this.tour) {
            throw new Error(`TourStep instance must have a tour`);
        }
    }

    get describeMe() {
        this.checkHasTour();
        return (
            `[${this.index + 1}/${this.tour.steps.length}] Tour ${this.tour.name} → Step ` +
            (this.content ? `${this.content} (trigger: ${this.trigger})` : this.trigger)
        );
    }

    get stringify() {
        return (
            JSON.stringify(
                pick(this, "isActive", "content", "trigger", "run", "tooltipPosition", "timeout"),
                (_key, value) => {
                    if (typeof value === "function") {
                        return "[function]";
                    } else {
                        return value;
                    }
                },
                2
            ) + ","
        );
    }
}

```

## File: static\src\tour_service\tour_step_automatic.js

```javascript
import { tourState } from "./tour_state";
import * as hoot from "@odoo/hoot-dom";
import { callWithUnloadCheck, serializeChanges, serializeMutation } from "./tour_utils";
import { TourHelpers } from "./tour_helpers";
import { TourStep } from "./tour_step";
import { getTag } from "@web/core/utils/xml";
import { waitForStable } from "@web/core/macro";

export class TourStepAutomatic extends TourStep {
    skipped = false;
    error = "";
    constructor(data, tour, index) {
        super(data, tour);
        this.index = index;
        this.tourConfig = tourState.getCurrentConfig();
    }

    async checkForUndeterminisms(initialElement, delay) {
        if (delay <= 0 || !initialElement) {
            return;
        }
        const tagName = initialElement.tagName?.toLowerCase();
        if (["body", "html"].includes(tagName) || !tagName) {
            return;
        }
        const snapshot = initialElement.cloneNode(true);
        const mutations = await waitForStable(initialElement, delay);
        let reason;
        if (!hoot.isVisible(initialElement)) {
            reason = `Initial element is no longer visible`;
        } else if (!initialElement.isEqualNode(snapshot)) {
            reason =
                `Initial element has changed:\n` +
                JSON.stringify(serializeChanges(snapshot, initialElement), null, 2);
        } else if (mutations.length) {
            const changes = [...new Set(mutations.map(serializeMutation))];
            reason =
                `Initial element has mutated ${mutations.length} times:\n` +
                JSON.stringify(changes, null, 2);
        }
        if (reason) {
            throw new Error(
                `Potential non deterministic behavior found in ${delay}ms for trigger ${this.trigger}.\n${reason}`
            );
        }
    }

    get describeWhyIFailed() {
        const errors = [];
        if (this.element) {
            errors.push(`Element has been found.`);
            if (this.isUIBlocked) {
                errors.push("BUT: DOM is blocked by UI.");
            }
            if (!this.elementIsInModal) {
                errors.push(
                    `BUT: It is not allowed to do action on an element that's below a modal.`
                );
            }
            if (!this.elementIsEnabled) {
                errors.push(
                    `BUT: Element is not enabled. TIP: You can use :enable to wait the element is enabled before doing action on it.`
                );
            }
            if (!this.parentFrameIsReady) {
                errors.push(`BUT: parent frame is not ready ([is-ready='false']).`);
            }
        } else {
            const checkElement = hoot.queryFirst(this.trigger);
            if (checkElement) {
                errors.push(`Element has been found.`);
                errors.push(
                    `BUT: Element is not visible. TIP: You can use :not(:visible) to force the search for an invisible element.`
                );
            } else {
                errors.push(`Element (${this.trigger}) has not been found.`);
            }
        }
        return errors;
    }

    /**
     * When return null or false, macro continues.
     */
    async doAction() {
        if (this.skipped) {
            return;
        }
        return await callWithUnloadCheck(async () => {
            const actionHelper = new TourHelpers(this.element);
            if (typeof this.run === "function") {
                await this.run.call({ anchor: this.element }, actionHelper);
            } else if (typeof this.run === "string") {
                for (const todo of this.run.split("&&")) {
                    const m = String(todo)
                        .trim()
                        .match(/^(?<action>\w*) *\(? *(?<arguments>.*?)\)?$/);
                    await actionHelper[m.groups?.action](m.groups?.arguments);
                }
            }
        });
    }

    /**
     * Each time it returns false, tour engine wait for a mutation
     * to retry to find the trigger.
     * @returns {(HTMLElement|Boolean)}
     */
    findTrigger() {
        if (!this.active) {
            this.skipped = true;
            return true;
        }
        const visible = !/:(hidden|visible)\b/.test(this.trigger);
        this.element = hoot.queryFirst(this.trigger, { visible });
        if (this.element) {
            return !this.isUIBlocked &&
                this.elementIsEnabled &&
                this.elementIsInModal &&
                this.parentFrameIsReady
                ? this.element
                : false;
        }
        return false;
    }

    get isUIBlocked() {
        return (
            document.body.classList.contains("o_lazy_js_waiting") ||
            document.body.classList.contains("o_ui_blocked") ||
            document.querySelector(".o_blockUI") ||
            document.querySelector(".o_is_blocked")
        );
    }

    get parentFrameIsReady() {
        if (this.trigger.match(/\[is-ready=(true|false)\]/)) {
            return true;
        }
        const parentFrame = hoot.getParentFrame(this.element);
        return parentFrame && parentFrame.hasAttribute("is-ready")
            ? parentFrame.getAttribute("is-ready") === "true"
            : true;
    }

    get elementIsInModal() {
        if (this.hasAction) {
            const overlays = hoot.queryFirst(".popover, .o-we-command, .o_notification");
            const modal = hoot.queryFirst(".modal:visible:not(.o_inactive_modal):last");
            if (modal && !overlays && !this.trigger.startsWith("body")) {
                return (
                    modal.contains(hoot.getParentFrame(this.element)) ||
                    modal.contains(this.element)
                );
            }
        }
        return true;
    }

    get elementIsEnabled() {
        const isTag = (array) => array.includes(getTag(this.element, true));
        if (this.hasAction) {
            if (isTag(["input", "textarea"])) {
                return hoot.isEditable(this.element);
            } else if (isTag(["button", "select"])) {
                return !this.element.disabled;
            }
        }
        return true;
    }

    get hasAction() {
        return ["string", "function"].includes(typeof this.run) && !this.skipped;
    }
}

```

## File: static\src\tour_service\tour_utils.js

```javascript
/** @odoo-module **/
import * as hoot from "@odoo/hoot-dom";
import { _t } from "@web/core/l10n/translation";

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

function formatValue(key, value, maxLength = 200) {
    if (!value) {
        return "(empty)";
    }
    return value.length > maxLength ? value.slice(0, maxLength) + "..." : value;
}

function serializeNode(node) {
    if (node.nodeType === Node.TEXT_NODE) {
        return `"${node.nodeValue.trim()}"`;
    }
    return node.outerHTML ? formatValue("node", node.outerHTML, 500) : "[Unknown Node]";
}

export function serializeChanges(snapshot, current) {
    const changes = {
        node: serializeNode(current),
    };
    function pushChanges(key, obj) {
        changes[key] = changes[key] || [];
        changes[key].push(obj);
    }

    if (snapshot.textContent !== current.textContent) {
        pushChanges("modifiedText", { before: snapshot.textContent, after: current.textContent });
    }

    const oldChildren = [...snapshot.childNodes].filter((node) => node.nodeType !== Node.TEXT_NODE);
    const newChildren = [...current.childNodes].filter((node) => node.nodeType !== Node.TEXT_NODE);
    oldChildren.forEach((oldNode, index) => {
        if (!newChildren[index] || !oldNode.isEqualNode(newChildren[index])) {
            pushChanges("removedNodes", { oldNode: serializeNode(oldNode) });
        }
    });
    newChildren.forEach((newNode, index) => {
        if (!oldChildren[index] || !newNode.isEqualNode(oldChildren[index])) {
            pushChanges("addedNodes", { newNode: serializeNode(newNode) });
        }
    });

    const oldAttrNames = new Set([...snapshot.attributes].map((attr) => attr.name));
    const newAttrNames = new Set([...current.attributes].map((attr) => attr.name));
    new Set([...oldAttrNames, ...newAttrNames]).forEach((attributeName) => {
        const oldValue = snapshot.getAttribute(attributeName);
        const newValue = current.getAttribute(attributeName);
        const before = oldValue !== newValue || !newAttrNames.has(attributeName) ? oldValue : null;
        const after = oldValue !== newValue || !oldAttrNames.has(attributeName) ? newValue : null;
        if (before || after) {
            pushChanges("modifiedAttributes", { attributeName, before, after });
        }
    });
    return changes;
}

export function serializeMutation(mutation) {
    const { type, attributeName } = mutation;
    if (type === "attributes" && attributeName) {
        return `attribute: ${attributeName}`;
    } else {
        return type;
    }
}

/**
 * @param {HTMLElement} element
 * @returns {HTMLElement | null}
 */
export function getScrollParent(element) {
    if (!element) {
        return null;
    }
    // We cannot only rely on the fact that the element’s scrollHeight is
    // greater than its clientHeight. This might not be the case when a step
    // starts, and the scrollbar could appear later. For example, when clicking
    // on a "building block" in the "building block previews modal" during a
    // tour (in website edit mode). When the modal opens, not all "building
    // blocks" are loaded yet, and the scrollbar is not present initially.
    const overflowY = window.getComputedStyle(element).overflowY;
    const isScrollable =
        overflowY === "auto" ||
        overflowY === "scroll" ||
        (overflowY === "visible" && element === element.ownerDocument.scrollingElement);
    if (isScrollable) {
        return element;
    } else {
        return getScrollParent(element.parentNode);
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

    showAppsMenuItem() {
        return {
            isActive: ["auto", "community", "desktop"],
            trigger: ".o_navbar_apps_menu button:enabled",
            tooltipPosition: "bottom",
            run: "click",
        };
    },

    toggleHomeMenu() {
        return [
            {
                isActive: [".o_main_navbar .o_menu_toggle"],
                trigger: ".o_main_navbar .o_menu_toggle",
                content: _t("Click the top left corner to navigate across apps."),
                tooltipPosition: "bottom",
                run: "click",
            },
            {
                isActive: ["mobile"],
                trigger: ".o_sidebar_topbar a.btn-primary",
                tooltipPosition: "right",
                run: "click",
            },
        ];
    },

    autoExpandMoreButtons(isActiveMobile = false) {
        const isActive = ["auto"];
        if (isActiveMobile) {
            isActive.push("mobile");
        }
        return {
            isActive,
            content: `autoExpandMoreButtons`,
            trigger: ".o-form-buttonbox",
            run() {
                const more = hoot.queryFirst(".o-form-buttonbox .o_button_more");
                if (more) {
                    hoot.click(more);
                }
            },
        };
    },

    goToAppSteps(dataMenuXmlid, description) {
        return [
            this.showAppsMenuItem(),
            {
                isActive: ["community"],
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                tooltipPosition: "right",
                run: "click",
            },
            {
                isActive: ["enterprise"],
                trigger: `.o_app[data-menu-xmlid="${dataMenuXmlid}"]`,
                content: description,
                tooltipPosition: "bottom",
                run: "click",
            },
        ].map((step) =>
            this.addDebugHelp(this._getHelpMessage("goToApp", dataMenuXmlid, description), step)
        );
    },

    statusbarButtonsSteps(innerTextButton, description, trigger) {
        const steps = [];
        if (trigger) {
            steps.push({
                isActive: ["auto", "mobile"],
                trigger,
            });
        }
        steps.push(
            {
                isActive: ["auto", "mobile"],
                trigger: ".o_cp_action_menus",
                run: (actions) => {
                    const node = hoot.queryFirst(".o_cp_action_menus .fa-cog");
                    if (node) {
                        hoot.click(node);
                    }
                },
            },
            {
                trigger: `.o_statusbar_buttons button:enabled:contains('${innerTextButton}'), .dropdown-item button:enabled:contains('${innerTextButton}')`,
                content: description,
                tooltipPosition: "bottom",
                run: "click",
            }
        );
        return steps.map((step) =>
            this.addDebugHelp(
                this._getHelpMessage("statusbarButtonsSteps", innerTextButton, description),
                step
            )
        );
    },

    mobileKanbanSearchMany2X(modalTitle, valueSearched) {
        return [
            {
                isActive: ["mobile"],
                trigger: `.modal:not(.o_inactive_modal) .o_control_panel_navigation .btn .fa-search`,
                tooltipPosition: "bottom",
                run: "click",
            },
            {
                isActive: ["mobile"],
                trigger: ".o_searchview_input",
                tooltipPosition: "bottom",
                run: `edit ${valueSearched}`,
            },
            {
                isActive: ["mobile"],
                trigger: ".dropdown-menu.o_searchview_autocomplete",
            },
            {
                isActive: ["mobile"],
                trigger: ".o_searchview_input",
                tooltipPosition: "bottom",
                run: "press Enter",
            },
            {
                isActive: ["mobile"],
                trigger: `.o_kanban_record:contains('${valueSearched}')`,
                tooltipPosition: "bottom",
                run: "click",
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
     */
    saveForm() {
        return [
            {
                isActive: ["auto"],
                content: "save form",
                trigger: ".o_form_button_save:enabled",
                run: "click",
            },
            {
                content: "wait for save completion",
                trigger: ".o_form_readonly, .o_form_saved",
            },
        ];
    },
    /**
     * Utility steps to cancel a form creation or edition.
     *
     * Supports creation/edition from either a form or a list view (so checks
     * for both states).
     */
    discardForm() {
        return [
            {
                isActive: ["auto"],
                content: "discard the form",
                trigger: ".o_form_button_cancel",
                run: "click",
            },
            {
                content: "wait for cancellation to complete",
                trigger:
                    ".o_view_controller.o_list_view, .o_form_view > div > div > .o_form_readonly, .o_form_view > div > div > .o_form_saved",
            },
        ];
    },

    waitIframeIsReady() {
        return {
            content: "Wait until the iframe is ready",
            trigger: `iframe[is-ready=true]:iframe html`,
        };
    },

    goToUrl(url) {
        return {
            isActive: ["auto"],
            content: `Navigate to ${url}`,
            trigger: "body",
            run: `goToUrl ${url}`,
        };
    },
};

```

## File: static\src\tour_service\tour_recorder\tour_recorder.js

```javascript
import { useService } from "@web/core/utils/hooks";
import { Dropdown } from "@web/core/dropdown/dropdown";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { browser } from "@web/core/browser/browser";
import { queryAll, queryFirst, queryOne } from "@odoo/hoot-dom";
import { Component, useState, useExternalListener } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { x2ManyCommands } from "@web/core/orm_service";
import { tourRecorderState } from "./tour_recorder_state";

export const TOUR_RECORDER_ACTIVE_LOCAL_STORAGE_KEY = "tour_recorder_active";
const PRECISE_IDENTIFIERS = ["data-menu-xmlid", "name", "contenteditable"];
const ODOO_CLASS_REGEX = /^oe?(-|_)[\w-]+$/;
const VALIDATING_KEYS = ["Enter", "Tab"];

/**
 * @param {EventTarget[]} paths composedPath of an click event
 * @returns {string}
 */
const getShortestSelector = (paths) => {
    paths.reverse();
    let filteredPath = [];
    let hasOdooClass = false;
    for (
        let currentElem = paths.pop();
        (currentElem && queryAll(filteredPath.join(" > ")).length !== 1) || !hasOdooClass;
        currentElem = paths.pop()
    ) {
        if (currentElem.parentElement.contentEditable === "true") {
            continue;
        }

        let currentPredicate = currentElem.tagName.toLowerCase();
        const odooClass = [...currentElem.classList].find((c) => c.match(ODOO_CLASS_REGEX));
        if (odooClass) {
            currentPredicate = `.${odooClass}`;
            hasOdooClass = true;
        }

        // If we are inside a link or button the previous elements, like <i></i>, <span></span>, etc., can be removed
        if (["BUTTON", "A"].includes(currentElem.tagName)) {
            filteredPath = [];
        }

        for (const identifier of PRECISE_IDENTIFIERS) {
            const identifierValue = currentElem.getAttribute(identifier);
            if (identifierValue) {
                currentPredicate += `[${identifier}='${CSS.escape(identifierValue)}']`;
            }
        }

        const siblingNodes = queryAll(":scope > " + currentPredicate, {
            root: currentElem.parentElement,
        });
        if (siblingNodes.length > 1) {
            currentPredicate += `:nth-child(${
                [...currentElem.parentElement.children].indexOf(currentElem) + 1
            })`;
        }

        filteredPath.unshift(currentPredicate);
    }

    if (filteredPath.length > 2) {
        return reducePath(filteredPath);
    }

    return filteredPath.join(" > ");
};

/**
 * @param {string[]} paths
 * @returns {string}
 */
const reducePath = (paths) => {
    const numberOfElement = paths.length - 2;
    let currentElement = "";
    let hasReduced = false;
    let path = paths.shift();
    for (let i = 0; i < numberOfElement; i++) {
        currentElement = paths.shift();
        if (queryAll(`${path} ${paths.join(" > ")}`).length === 1) {
            hasReduced = true;
        } else {
            path += `${hasReduced ? " " : " > "}${currentElement}`;
            hasReduced = false;
        }
    }
    path += `${hasReduced ? " " : " > "}${paths.shift()}`;
    return path;
};

export class TourRecorder extends Component {
    static template = "web_tour.TourRecorder";
    static components = { Dropdown, DropdownItem };
    static props = {
        onClose: { type: Function },
    };
    static defaultState = {
        recording: false,
        url: "",
        editedElement: undefined,
        tourName: "",
    };

    setup() {
        this.originClickEvent = false;
        this.notification = useService("notification");
        this.orm = useService("orm");
        this.state = useState({
            ...TourRecorder.defaultState,
            steps: [],
        });

        this.state.steps = tourRecorderState.getCurrentTourRecorder();
        this.state.recording = tourRecorderState.isRecording() === "1";
        useExternalListener(document, "pointerdown", this.setStartingEvent, { capture: true });
        useExternalListener(document, "pointerup", this.recordClickEvent, { capture: true });
        useExternalListener(document, "keydown", this.recordConfirmationKeyboardEvent, {
            capture: true,
        });
        useExternalListener(document, "keyup", this.recordKeyboardEvent, { capture: true });
    }

    /**
     * @param {PointerEvent} ev
     */
    setStartingEvent(ev) {
        if (!this.state.recording || ev.target.closest(".o_tour_recorder")) {
            return;
        }
        this.originClickEvent = ev.composedPath().filter((p) => p instanceof Element);
    }

    /**
     * @param {PointerEvent} ev
     */
    recordClickEvent(ev) {
        if (!this.state.recording || ev.target.closest(".o_tour_recorder")) {
            return;
        }
        const pathElements = ev.composedPath().filter((p) => p instanceof Element);
        this.addTourStep([...pathElements]);

        const lastStepInput = this.state.steps.at(-1);
        // Check that pointerdown and pointerup paths are different to know if it's a drag&drop or a click
        if (
            JSON.stringify(pathElements.map((e) => e.tagName)) !==
            JSON.stringify(this.originClickEvent.map((e) => e.tagName))
        ) {
            lastStepInput.run = `drag_and_drop ${lastStepInput.trigger}`;
            lastStepInput.trigger = getShortestSelector(this.originClickEvent);
        } else {
            const lastStepInput = this.state.steps.at(-1);
            lastStepInput.run = "click";
        }

        tourRecorderState.setCurrentTourRecorder(this.state.steps);
    }

    /**
     * @param {KeyboardEvent} ev
     */
    recordConfirmationKeyboardEvent(ev) {
        if (
            !this.state.recording ||
            !this.state.editedElement ||
            ev.target.closest(".o_tour_recorder")
        ) {
            return;
        }

        if (
            [...this.state.editedElement.classList].includes("o-autocomplete--input") &&
            VALIDATING_KEYS.includes(ev.key)
        ) {
            const selectedRow = queryFirst(".ui-state-active", {
                root: this.state.editedElement.parentElement,
            });
            this.state.steps.push({
                trigger: `.o-autocomplete--dropdown-item > a:contains('${selectedRow.textContent}'), .fa-circle-o-notch`,
                run: "click",
            });
            this.state.editedElement = undefined;
        }
        tourRecorderState.setCurrentTourRecorder(this.state.steps);
    }

    /**
     * @param {KeyboardEvent} ev
     */
    recordKeyboardEvent(ev) {
        if (
            !this.state.recording ||
            VALIDATING_KEYS.includes(ev.key) ||
            ev.target.closest(".o_tour_recorder")
        ) {
            return;
        }

        if (!this.state.editedElement) {
            if (
                ev.target.matches(
                    "input:not(:disabled), textarea:not(:disabled), [contenteditable=true]"
                )
            ) {
                this.state.editedElement = ev.target;
                this.state.steps.push({
                    trigger: getShortestSelector(ev.composedPath()),
                });
            } else {
                return;
            }
        }

        if (!this.state.editedElement) {
            return;
        }

        const lastStep = this.state.steps.at(-1);
        if (this.state.editedElement.contentEditable === "true") {
            lastStep.run = `editor ${this.state.editedElement.textContent}`;
        } else {
            lastStep.run = `edit ${this.state.editedElement.value}`;
        }
        tourRecorderState.setCurrentTourRecorder(this.state.steps);
    }

    toggleRecording() {
        this.state.recording = !this.state.recording;
        tourRecorderState.setIsRecording(this.state.recording);
        this.state.editedElement = undefined;
        if (this.state.recording && !this.state.url) {
            this.state.url = browser.location.pathname + browser.location.search;
        }
    }

    async saveTour() {
        const newTour = {
            name: this.state.tourName.replaceAll(" ", "_"),
            url: this.state.url,
            step_ids: this.state.steps.map((s) => x2ManyCommands.create(undefined, s)),
            custom: true,
        };

        const result = await this.orm.create("web_tour.tour", [newTour]);
        if (result) {
            this.notification.add(_t("Custom tour '%s' has been added.", newTour.name), {
                type: "success",
            });
            this.resetTourRecorderState();
        } else {
            this.notification.add(_t("Custom tour '%s' couldn't be saved!", newTour.name), {
                type: "danger",
            });
        }
    }

    resetTourRecorderState() {
        Object.assign(this.state, { ...TourRecorder.defaultState, steps: [] });
        tourRecorderState.clear();
    }

    /**
     * @param {Element[]} path
     */
    addTourStep(path) {
        const shortestPath = getShortestSelector(path);
        const target = queryOne(shortestPath);
        this.state.editedElement =
            target.matches(
                "input:not(:disabled), textarea:not(:disabled), [contenteditable=true]"
            ) && target;
        this.state.steps.push({
            trigger: shortestPath,
        });
    }
}

```

## File: static\src\tour_service\tour_recorder\tour_recorder.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
<t t-name="web_tour.TourRecorder">
    <div class="d-flex position-fixed bottom-0 start-0 bg-primary o_tour_recorder">
        <div t-ref="tour_recorder" class="d-flex">
            <button class="o_button_record btn btn-primary rounded-0" t-on-click.prevent.stop="toggleRecording">
                <span class="px-2 me-1 rounded-circle" t-att-class="state.recording ? 'bg-danger': 'bg-secondary'" role="status" aria-hidden="true"></span>
                Record
                <span class="fst-italic fw-lighter" t-if="state.editedElement"> (recording keyboard)</span>
            </button>
            <Dropdown position="'top-end'">
                <button class="o_button_steps btn btn-primary rounded-0">
                    Steps <span class="badge rounded-pill bg-danger"><t t-esc="state.steps.length"/></span>
                </button>
                <t t-set-slot="content">
                    <div class="o_tour_recorder p-2">
                        <h4>Steps:</h4>
                        <table class="table table-striped">
                            <thead>
                                <tr>
                                    <td>n°</td>
                                    <td>trigger</td>
                                    <td></td>
                                </tr>
                            </thead>
                            <tbody>
                                <t t-foreach="state.steps" t-as="step" t-key="step_index">
                                    <tr class="o_tour_step" t-att-class="step.triggerNotUnique ? 'text-danger' : ''">
                                        <td><t t-esc="step_index + 1"/>.</td>
                                        <td class="o_tour_step_trigger">
                                            <t t-esc="step.trigger"/>
                                            <span t-if="step.run and step.run != 'click'" class="fst-italic fw-lighter"><br/>(run: <t t-esc="step.run"/>) </span>
                                        </td>
                                        <td><button class="o_button_delete_step btn btn-link text-danger fa fa-trash mx-1" t-on-click.prevent.stop="() => state.steps.splice(step_index, 1)"/></td>
                                    </tr>
                                </t>
                            </tbody>
                        </table>
                    </div>
                </t>
            </Dropdown>
            <Dropdown t-if="state.steps.length" position="'top-end'">
                <button class="o_button_save btn btn-primary px-1 rounded-0">
                    <i class="fa fa-floppy-o"></i>
                </button>
                <t t-set-slot="content">
                    <div class="o_tour_recorder p-1" style="min-width: 30vw;">
                        <form class="p-1" t-on-submit.prevent="saveTour">
                            <label for="name" class="o_form_label my-1">Name:</label><br/>
                            <input t-att-value="state.tourName" t-on-change="(ev) => state.tourName = ev.target.value" class="o_input" placeholder="name_of_the_tour" type="text" name="name"/>
                            <label for="url" class="o_form_label my-1">Url:</label><br/>
                            <input t-att-value="state.url" t-on-change="(ev) => state.url = ev.target.value" class="o_input" type="text" name="url"/>
                            <button class="o_button_save_confirm btn btn-primary mt-3">Save</button>
                        </form>
                    </div>
                </t>
            </Dropdown>
            <button t-if="state.steps.length" class="btn btn-primary px-1" t-on-click="resetTourRecorderState"><i class="fa fa-undo"></i></button>
            <button class="btn btn-primary position-absolute bottom-0 start-100 rounded-0 border-1 o_tour_recorder_close_button" t-on-click.prevent.stop="() => props.onClose()"><i class="fa fa-close"></i></button>
        </div>
    </div>
</t>
</templates>

```

## File: static\src\tour_service\tour_recorder\tour_recorder_state.js

```javascript
import { browser } from "@web/core/browser/browser";

const CURRENT_TOUR_RECORDER_LOCAL_STORAGE = "current_tour_recorder";
const CURRENT_TOUR_RECORDER_RECORD_LOCAL_STORAGE = "current_tour_recorder.record";

/**
 * Wrapper around localStorage for persistence of the current recording.
 * Useful for resuming recording when the page refreshed.
 */
export const tourRecorderState = {
    isRecording() {
        return browser.localStorage.getItem(CURRENT_TOUR_RECORDER_RECORD_LOCAL_STORAGE) || "0";
    },
    setIsRecording(isRecording) {
        browser.localStorage.setItem(
            CURRENT_TOUR_RECORDER_RECORD_LOCAL_STORAGE,
            isRecording ? "1" : "0"
        );
    },
    setCurrentTourRecorder(tour) {
        tour = JSON.stringify(tour);
        browser.localStorage.setItem(CURRENT_TOUR_RECORDER_LOCAL_STORAGE, tour);
    },
    getCurrentTourRecorder() {
        const tour = browser.localStorage.getItem(CURRENT_TOUR_RECORDER_LOCAL_STORAGE) || "[]";
        return JSON.parse(tour);
    },
    clear() {
        browser.localStorage.removeItem(CURRENT_TOUR_RECORDER_LOCAL_STORAGE);
        browser.localStorage.removeItem(CURRENT_TOUR_RECORDER_RECORD_LOCAL_STORAGE);
    },
};

```

## File: static\src\views\tour_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="web_tour.TourListController.Buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary">
        <xpath expr="." position="inside">
            <button class="btn btn-primary me-1 o_button_tour_recorder" title="Record Tour"
                t-on-click="() => this.env.services.tour_service.startTourRecorder()">
                Record
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\tour_list.js

```javascript
import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";

registry.category("views").add("tour_list", {
    ...listView,
    buttonTemplate: "web_tour.TourListController.Buttons",
});

```

## File: static\src\widgets\tour_start.js

```javascript
import { charField, CharField } from "@web/views/fields/char/char_field";
import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";

export class TourStartWidget extends CharField {
    static template = "web_tour.TourStartWidget";
    static props = {
        ...CharField.props,
        link: { type: Boolean, optional: true },
    };

    setup() {
        this.tour = useService("tour_service");
    }

    get tourData() {
        return this.props.record.data;
    }

    _onStartTour() {
        this.tour.startTour(this.tourData.name, {
            mode: "manual",
            url: this.tourData.url,
            fromDB: this.tourData.custom,
            rainbowManMessage: this.tourData.rainbow_man_message,
        });
    }

    _onTestTour() {
        this.tour.startTour(this.tourData.name, {
            mode: "auto",
            url: this.tourData.url,
            fromDB: this.tourData.custom,
            stepDelay: 500,
            showPointerDuration: 250,
            rainbowManMessage: this.tourData.rainbow_man_message,
        });
    }
}

export const tourStartWidgetField = {
    ...charField,
    component: TourStartWidget,
    extractProps: ({ options }) => ({
        link: options.link,
    }),
};

registry.category("fields").add("tour_start_widget", tourStartWidgetField);

```

## File: static\src\widgets\tour_start.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="web_tour.TourStartWidget">
        <button title="Start Tour"
                t-on-click.prevent.stop="_onStartTour"
                t-att-class="'o_start_tour btn ' + (this.props.link ? 'btn-link py-0' : 'btn-primary')">
            Onboarding
        </button>
        <button title="Test Tour"
                t-on-click.prevent.stop="_onTestTour"
                t-att-class="'o_test_tour btn ' + (this.props.link ? 'btn-link py-0' : 'btn-primary ms-1')">
            Testing
        </button>
    </t>
</templates>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <record id="res_users_view_form" model="ir.ui.view">
    <field name="name">res.users.form.tour.inherit</field>
    <field name="model">res.users</field>
    <field name="inherit_id" ref="base.view_users_form"/>
    <field name="arch" type="xml">
        <xpath expr="//field[@name='tz']" position="after">
            <field name="tour_enabled" widget="boolean_toggle"/>
        </xpath>
    </field>
  </record>
</odoo>

```

## File: views\tour_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tour_form" model="ir.ui.view">
        <field name="model">web_tour.tour</field>
        <field name="arch" type="xml">
            <form create="0">
                <header>
                    <div>
                        <field name="name" widget="tour_start_widget"/>
                    </div>
                </header>
                <sheet>
                    <div class="oe_title">
                        <h1><field class="text-break" options="{'line_breaks': False}" widget="text" name="name" placeholder="e.g. My_Tour"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="sequence"/>
                            <field name="url"/>
                        </group>
                        <group>
                            <field name="custom" readonly="1"/>
                            <field name="sharing_url" widget="CopyClipboardURL"/>
                        </group>
                        <notebook>
                            <page name="steps" string="Steps" invisible="not custom">
                                <field name="step_ids">
                                    <list>
                                        <field name="sequence" widget="handle"/>
                                        <field name="trigger"/>
                                        <field name="run"/>
                                        <field name="content"/>
                                    </list>
                                </field>
                            </page>
                        </notebook>
                        <field colspan="2" name="rainbow_man_message" placeholder="Rainbow Man Message..."/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="tour_list" model="ir.ui.view">
        <field name="model">web_tour.tour</field>
        <field name="arch" type="xml">
            <list string="Menu" create="0" edit="0" js_class="tour_list">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="url"/>
                <field name="custom"/>
                <field name="rainbow_man_message" column_invisible="1"/> <!-- Invisible to get the data in JS -->
                <field name="name" widget="tour_start_widget" options="{'link': true}" string=""/>
            </list>
        </field>
    </record>

    <record id="tour_search" model="ir.ui.view">
        <field name="name">tour.search</field>
        <field name="model">web_tour.tour</field>
        <field name="arch" type="xml">
            <search string="Tip">
                <field name="name"/>
            </search>
        </field>
    </record>

    <record id="tour_action" model="ir.actions.act_window">
        <field name="name">Tours</field>
        <field name="res_model">web_tour.tour</field>
        <field name="view_id" ref="tour_list"/>
        <field name="search_view_id" ref="tour_search"/>
    </record>
    <menuitem action="tour_action" id="menu_tour_action" parent="base.next_id_2" sequence="5"/>


    <record id="tour_export_js_action" model="ir.actions.server">
        <field name="name">Export JS</field>
        <field name="model_id" ref="web_tour.model_web_tour_tour"/>
        <field name="binding_model_id" ref="web_tour.model_web_tour_tour"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">
if records:
    action = records.export_js_file()
        </field>
    </record>
</odoo>

```

