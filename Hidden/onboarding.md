# Odoo Module: onboarding

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
    'name': 'Onboarding Toolbox',
    'version': '1.2',
    'category': 'Hidden',
    'sequence': 9001,
    'description': """
This module allows to manage onboardings and their progress
================================================================================
    """,
    'depends': ['web'],
    'installable': True,
    'data': [
        'views/onboarding_templates.xml',
        'views/onboarding_views.xml',
        'views/onboarding_menus.xml',
        'security/ir.model.access.csv',
    ],
    'assets': {
        'web.assets_backend': [
            'onboarding/static/src/**/*',
            ("remove", "onboarding/static/src/scss/onboarding.variables.dark.scss"),
        ],
        "web.dark_mode_variables": [
            ('before', 'onboarding/static/src/scss/onboarding.variables.scss', 'onboarding/static/src/scss/onboarding.variables.dark.scss'),
        ],
        'web._assets_primary_variables': [
            'onboarding/static/src/scss/onboarding.variables.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\onboarding_onboarding.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.onboarding.models.onboarding_progress import ONBOARDING_PROGRESS_STATES


class Onboarding(models.Model):
    _name = 'onboarding.onboarding'
    _description = 'Onboarding'
    _order = 'sequence asc, id desc'

    name = fields.Char('Name of the onboarding', translate=True)
    # One word identifier used to define the onboarding panel's route: `/onboarding/{route_name}`.
    route_name = fields.Char('One word name', required=True)
    step_ids = fields.Many2many('onboarding.onboarding.step', string='Onboarding steps')

    text_completed = fields.Char(
        'Message at completion', default=lambda s: s.env._('Nice work! Your configuration is done.'),
        help='Text shown on onboarding when completed')

    is_per_company = fields.Boolean(
        'Should be done per company?', compute='_compute_is_per_company', readonly=True, store=False,
    )
    panel_close_action_name = fields.Char(
        'Closing action', help='Name of the onboarding model action to execute when closing the panel.')

    current_progress_id = fields.Many2one(
        'onboarding.progress', 'Onboarding Progress', compute='_compute_current_progress',
        help='Onboarding Progress for the current context (company).')
    current_onboarding_state = fields.Selection(
        ONBOARDING_PROGRESS_STATES, string='Completion State', compute='_compute_current_progress', readonly=True)
    is_onboarding_closed = fields.Boolean(string='Was panel closed?', compute='_compute_current_progress')

    progress_ids = fields.One2many(
        'onboarding.progress', 'onboarding_id', string='Onboarding Progress Records', readonly=True,
        help='All Onboarding Progress Records (across companies).')

    sequence = fields.Integer(default=10)
    _sql_constraints = [
        ('route_name_uniq', 'UNIQUE (route_name)', 'Onboarding alias must be unique.'),
    ]

    @api.depends('progress_ids', 'progress_ids.company_id', 'step_ids', 'step_ids.is_per_company')
    def _compute_is_per_company(self):
        # Once an onboarding is made "per-company", there is no drawback to simply still consider
        # it per-company even when if its last per-company step is unlinked. This allows to avoid
        # handling the merging of existing progress (step) records.

        onboardings_with_per_company_steps_or_progress = self.filtered(
            lambda o: o.progress_ids.company_id or (True in o.step_ids.mapped('is_per_company')))
        onboardings_with_per_company_steps_or_progress.is_per_company = True
        (self - onboardings_with_per_company_steps_or_progress).is_per_company = False

    @api.depends_context('company')
    @api.depends('progress_ids', 'progress_ids.is_onboarding_closed', 'progress_ids.onboarding_state', 'progress_ids.company_id')
    def _compute_current_progress(self):
        for onboarding in self:
            current_progress_id = onboarding.progress_ids.filtered(
                lambda progress: progress.company_id.id in {False, self.env.company.id})
            if current_progress_id:
                onboarding.current_onboarding_state = current_progress_id.onboarding_state
                onboarding.current_progress_id = current_progress_id
                onboarding.is_onboarding_closed = current_progress_id.is_onboarding_closed
            else:
                onboarding.current_onboarding_state = 'not_done'
                onboarding.current_progress_id = False
                onboarding.is_onboarding_closed = False

    def write(self, vals):
        """Recompute progress step ids if new steps are added/removed."""
        already_linked_steps = self.step_ids
        res = super().write(vals)
        if self.step_ids != already_linked_steps:
            self.progress_ids._recompute_progress_step_ids()
        return res

    def action_close(self):
        """Close the onboarding panel."""
        self.current_progress_id.action_close()

    @api.model
    def action_close_panel(self, xmlid):
        """Close the onboarding panel identified by its `xmlid`.

        If not found, quietly do nothing.
        """
        if onboarding := self.env.ref(xmlid, raise_if_not_found=False):
            onboarding.action_close()

    def action_refresh_progress_ids(self):
        """Re-initialize onboarding progress records (after step is_per_company change).

        Meant to be called when `is_per_company` of linked steps is modified (or per-company
        steps are added to an onboarding).
        """
        onboardings_to_refresh_progress = self.filtered(
            lambda o: o.is_per_company and o.progress_ids and not o.progress_ids.company_id
        )
        onboardings_to_refresh_progress.progress_ids.unlink()
        onboardings_to_refresh_progress._create_progress()

    def action_toggle_visibility(self):
        self.current_progress_id.action_toggle_visibility()

    def _search_or_create_progress(self):
        """Create Progress record(s) as necessary for the context."""
        onboardings_without_progress = self.filtered(lambda onboarding: not onboarding.current_progress_id)
        onboardings_without_progress._create_progress()
        return self.current_progress_id

    def _create_progress(self):
        return self.env['onboarding.progress'].create([
            {
                'company_id': self.env.company.id if onboarding.is_per_company else False,
                'onboarding_id': onboarding.id,
                'progress_step_ids': onboarding.step_ids.progress_ids.filtered(
                    lambda p: p.company_id.id in [False, self.env.company.id]
                ),
            }
            for onboarding in self
        ])

    def _prepare_rendering_values(self):
        self.ensure_one()
        values = {
            'close_method': self.panel_close_action_name,
            'close_model': 'onboarding.onboarding',
            'steps': self.step_ids,
            'state': self.current_progress_id._get_and_update_onboarding_state(),
            'text_completed': self.text_completed,
        }

        return values

```

## File: models\onboarding_onboarding_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, Command, fields, models
from odoo.addons.onboarding.models.onboarding_progress import ONBOARDING_PROGRESS_STATES
from odoo.exceptions import ValidationError


class OnboardingStep(models.Model):
    _name = 'onboarding.onboarding.step'
    _description = 'Onboarding Step'
    _order = 'sequence asc, id asc'
    _rec_name = 'title'

    onboarding_ids = fields.Many2many('onboarding.onboarding', string='Onboardings')

    title = fields.Char('Title', translate=True)
    description = fields.Char('Description', translate=True)
    button_text = fields.Char(
        'Button text', required=True, default=lambda s: s.env._("Let's do it"), translate=True,
        help="Text on the panel's button to start this step")
    done_icon = fields.Char('Font Awesome Icon when completed', default='fa-star')
    done_text = fields.Char(
        'Text to show when step is completed', default=lambda s: s.env._('Step Completed!'), translate=True)
    step_image = fields.Binary("Step Image")
    step_image_filename = fields.Char("Step Image Filename")
    step_image_alt = fields.Char(
        'Alt Text for the Step Image', default='Onboarding Step Image', translate=True,
        help='Show when impossible to load the image')
    panel_step_open_action_name = fields.Char(
        string='Opening action', required=False,
        help='Name of the onboarding step model action to execute when opening the step, '
             'e.g. action_open_onboarding_1_step_1')

    current_progress_step_id = fields.Many2one(
        'onboarding.progress.step', string='Step Progress',
        compute='_compute_current_progress', help='Onboarding Progress Step for the current context (company).')
    current_step_state = fields.Selection(
        ONBOARDING_PROGRESS_STATES, string='Completion State', compute='_compute_current_progress')
    progress_ids = fields.One2many(
        'onboarding.progress.step', 'step_id', string='Onboarding Progress Step Records', readonly=True,
        help='All related Onboarding Progress Step Records (across companies)')

    is_per_company = fields.Boolean('Is per company', default=True)
    sequence = fields.Integer(default=10)

    @api.depends_context('company')
    @api.depends('progress_ids', 'progress_ids.step_state')
    def _compute_current_progress(self):
        # When `is_per_company` is changed, `progress_ids` is updated (see `write`) which triggers this `_compute`.
        existing_progress_steps = self.progress_ids.filtered_domain([
            ('step_id', 'in', self.ids),
            ('company_id', 'in', [False, self.env.company.id]),
        ])
        for step in self:
            if step in existing_progress_steps.step_id:
                current_progress_step_id = existing_progress_steps.filtered(
                    lambda progress_step: progress_step.step_id == step)
                step.current_progress_step_id = current_progress_step_id
                step.current_step_state = current_progress_step_id.step_state
            else:
                step.current_progress_step_id = False
                step.current_step_state = 'not_done'

    @api.constrains('onboarding_ids')
    def check_step_on_onboarding_has_action(self):
        if steps_without_action := self.filtered(lambda step: step.onboarding_ids and not step.panel_step_open_action_name):
            raise ValidationError(_(
                'An "Opening Action" is required for the following steps to be '
                'linked to an onboarding panel: %(step_titles)s',
                step_titles=steps_without_action.mapped('title'),
            ))

    def write(self, vals):
        new_is_per_company = vals.get('is_per_company')
        steps_changing_is_per_company = (
            self.browse() if new_is_per_company is None
            else self.filtered(lambda step: step.is_per_company != new_is_per_company)
        )
        already_linked_onboardings = self.onboarding_ids

        res = super().write(vals)

        # Progress is reset (to be done per-company or, for steps, to have a single record)
        if steps_changing_is_per_company:
            steps_changing_is_per_company.progress_ids.unlink()
        self.onboarding_ids.action_refresh_progress_ids()

        if self.onboarding_ids - already_linked_onboardings:
            self.onboarding_ids.progress_ids._recompute_progress_step_ids()

        return res

    def action_set_just_done(self):
        # Make sure progress records exist for the current context (company)
        steps_without_progress = self.filtered(lambda step: not step.current_progress_step_id)
        steps_without_progress._create_progress_steps()
        return self.current_progress_step_id.action_set_just_done().step_id

    @api.model
    def action_validate_step(self, xml_id):
        step = self.env.ref(xml_id, raise_if_not_found=False)
        if not step:
            return "NOT_FOUND"
        return "JUST_DONE" if step.action_set_just_done() else "WAS_DONE"

    @api.model
    def _get_placeholder_filename(self, field):
        if field == "step_image":
            return 'base/static/img/onboarding_default.png'
        return super()._get_placeholder_filename(field)

    def _create_progress_steps(self):
        """Create progress step records as necessary to validate steps.

        Only considers existing `onboarding.progress` records for the current
        company or without company (depending on `is_per_company`).
        """
        onboarding_progress_records = self.env['onboarding.progress'].search([
            ('onboarding_id', 'in', self.onboarding_ids.ids),
            ('company_id', 'in', [False, self.env.company.id])
        ])
        progress_step_values = [
            {
                'step_id': step_id.id,
                'progress_ids': [
                    Command.link(onboarding_progress_record.id)
                    for onboarding_progress_record
                    in onboarding_progress_records.filtered(lambda p: step_id in p.onboarding_id.step_ids)],
                'company_id': self.env.company.id if step_id.is_per_company else False,
            } for step_id in self
        ]
        return self.env['onboarding.progress.step'].create(progress_step_values)

```

## File: models\onboarding_progress.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


ONBOARDING_PROGRESS_STATES = [
    ('not_done', 'Not done'),
    ('just_done', 'Just done'),
    ('done', 'Done'),
]


class OnboardingProgress(models.Model):
    _name = 'onboarding.progress'
    _description = 'Onboarding Progress Tracker'
    _rec_name = 'onboarding_id'

    onboarding_state = fields.Selection(
        ONBOARDING_PROGRESS_STATES, string='Onboarding progress', compute='_compute_onboarding_state', store=True)
    is_onboarding_closed = fields.Boolean('Was panel closed?')
    company_id = fields.Many2one('res.company', ondelete='cascade')
    onboarding_id = fields.Many2one(
        'onboarding.onboarding', 'Related onboarding tracked', required=True, ondelete='cascade')
    progress_step_ids = fields.Many2many('onboarding.progress.step', string='Progress Steps Trackers')

    def init(self):
        """Make sure there aren't multiple records for the same onboarding and company."""
        # not in _sql_constraint because COALESCE is not supported for PostgreSQL constraint
        self.env.cr.execute("""
            CREATE UNIQUE INDEX IF NOT EXISTS onboarding_progress_onboarding_company_uniq
            ON onboarding_progress (onboarding_id, COALESCE(company_id, 0))
        """)

    @api.depends('onboarding_id.step_ids', 'progress_step_ids', 'progress_step_ids.step_state')
    def _compute_onboarding_state(self):
        for progress in self:
            progress.onboarding_state = (
                'not_done' if (
                    len(progress.progress_step_ids.filtered(lambda p: p.step_state in {'just_done', 'done'}))
                    != len(progress.onboarding_id.step_ids)
                )
                else 'done'
            )

    def _recompute_progress_step_ids(self):
        """Update progress steps when a step (with existing progress) is added to an onboarding."""
        for progress in self:
            progress.progress_step_ids = progress.onboarding_id.step_ids.current_progress_step_id

    def action_close(self):
        self.is_onboarding_closed = True

    def action_toggle_visibility(self):
        for progress in self:
            progress.is_onboarding_closed = not progress.is_onboarding_closed

    def _get_and_update_onboarding_state(self):
        """Fetch the progress of an onboarding for rendering its panel.

        This method is expected to only be called by the onboarding controller.
        It also has the responsibility of updating the 'just_done' state into
        'done' so that the 'just_done' states are only rendered once.
        """
        self.ensure_one()
        onboarding_states_values = {}
        progress_steps_to_consolidate = self.env['onboarding.progress.step']

        # Iterate over onboarding step_ids and not self.progress_step_ids because 'not_done' steps
        # may not have a progress_step record.
        for step in self.onboarding_id.step_ids:
            step_state = step.current_step_state
            if step_state == 'just_done':
                progress_steps_to_consolidate |= step.current_progress_step_id
            onboarding_states_values[step.id] = step_state

        progress_steps_to_consolidate.action_consolidate_just_done()

        if self.is_onboarding_closed:
            onboarding_states_values['onboarding_state'] = 'closed'
        elif self.onboarding_state == 'done':
            onboarding_states_values['onboarding_state'] = 'just_done' if progress_steps_to_consolidate else 'done'
        return onboarding_states_values

```

## File: models\onboarding_progress_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.addons.onboarding.models.onboarding_progress import ONBOARDING_PROGRESS_STATES


class OnboardingProgressStep(models.Model):
    _name = 'onboarding.progress.step'
    _description = 'Onboarding Progress Step Tracker'
    _rec_name = 'step_id'

    progress_ids = fields.Many2many('onboarding.progress', string='Related Onboarding Progress Tracker')
    step_state = fields.Selection(
        ONBOARDING_PROGRESS_STATES, string='Onboarding Step Progress', default='not_done')
    step_id = fields.Many2one(
        'onboarding.onboarding.step', string='Onboarding Step', required=True, ondelete='cascade')

    company_id = fields.Many2one('res.company', ondelete='cascade')

    def init(self):
        """Make sure there aren't multiple records for the same onboarding step and company."""
        # not in _sql_constraint because COALESCE is not supported for PostgreSQL constraint
        self.env.cr.execute("""
            CREATE UNIQUE INDEX IF NOT EXISTS onboarding_progress_step_company_uniq
            ON onboarding_progress_step (step_id, COALESCE(company_id, 0))
        """)

    def action_consolidate_just_done(self):
        was_just_done = self.filtered(lambda progress: progress.step_state == 'just_done')
        was_just_done.step_state = 'done'
        return was_just_done

    def action_set_just_done(self):
        not_done = self.filtered_domain([('step_state', '=', 'not_done')])
        not_done.step_state = 'just_done'
        return not_done

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import onboarding_onboarding
from . import onboarding_onboarding_step
from . import onboarding_progress
from . import onboarding_progress_step

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_onboarding_all,onboarding.onboarding.all,model_onboarding_onboarding,,0,0,0,0
access_onboarding_user,onboarding.onboarding.user,model_onboarding_onboarding,base.group_user,0,0,0,0
access_onboarding_manager,onboarding.onboarding.manager,model_onboarding_onboarding,base.group_system,1,1,1,1
access_onboarding_step_all,onboarding.onboarding.step.all,model_onboarding_onboarding_step,,0,0,0,0
access_onboarding_step_user,onboarding.onboarding.step.user,model_onboarding_onboarding_step,base.group_user,0,0,0,0
access_onboarding_step_manager,onboarding.onboarding.step.manager,model_onboarding_onboarding_step,base.group_system,1,1,1,1
access_onboarding_progress_all,onboarding.progress.all,model_onboarding_progress,,0,0,0,0
access_onboarding_progress_user,onboarding.progress.user,model_onboarding_progress,base.group_user,0,0,0,0
access_onboarding_progress_manager,onboarding.progress.manager,model_onboarding_progress,base.group_system,1,1,1,1
access_onboarding_progress_step_all,onboarding.progress.step.all,model_onboarding_progress_step,,0,0,0,0
access_onboarding_progress_step_user,onboarding.progress.step.user,model_onboarding_progress_step,base.group_user,0,0,0,0
access_onboarding_progress_step_manager,onboarding.progress.step.manager,model_onboarding_progress_step,base.group_system,1,1,1,1

```

## File: static\src\views\form\onboarding_step_form_controller.js

```javascript
/** @odoo-module **/

import { FormController } from "@web/views/form/form_controller";
import { useService } from "@web/core/utils/hooks";

/**
 * Controller to use for an onboarding step dialog, not the
 * onboarding.onboarding.step form view itself.
 */
export default class OnboardingStepFormController extends FormController {
    setup() {
        super.setup();
        this.action = useService('action');
        this.orm = useService('orm');
    }
    /**
     * If necessary, mark the step as done and reload the main view.
     * @override
     */
    async save({ closable, ...otherParams }) {
        const saved = await super.save(otherParams);
        if (saved) {
            const { reloadOnFirstValidation, reloadAlways } = this.stepConfig;
            const validationResponse = await this.orm.call(
                'onboarding.onboarding.step',
                'action_validate_step',
                [this.stepName],
            );
            if (reloadAlways || (reloadOnFirstValidation && validationResponse === "JUST_DONE")) {
                this.action.restore(this.action.currentController.jsId);
            } else if (closable) {
                this.action.doAction({ type: "ir.actions.act_window_close" });
            }
        }
        return saved;
    }
    /**
     * Returns the name of the onboarding step to validate after the dialog
     * record is saved
     *
     * @return {string}
     */
    get stepName() {
        return ''
    }
    /**
     *  Returns whether to reload the page (useful if the current
     * view needs to be updated).
     *
     * @returns {{reloadAlways: boolean, reloadOnFirstValidation: boolean}}
     */
    get stepConfig() {
        return { reloadAlways: false, reloadOnFirstValidation: false };
    }
}

```

## File: views\onboarding_menus.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- Under User Interface -->
    <menuitem name="Onboardings"
        id="menu_onboarding"
        parent="base.next_id_2"
        action="action_view_onboarding_onboarding"
        sequence="1"/>
    <menuitem name="Onboardings Steps"
        id="menu_onboarding_step"
        parent="base.next_id_2"
        action="action_view_onboarding_step"
        sequence="1"/>
</data></odoo>

```

## File: views\onboarding_templates.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <template id="onboarding_panel">
        <t t-call="onboarding.onboarding_container">
            <t t-foreach="steps" t-as="step">
                <t t-call="onboarding.onboarding_step">
                    <t t-set="title" t-value="step.title"/>
                    <t t-set="description" t-value="step.description"/>
                    <t t-set="done_icon" t-value="step.done_icon"/>
                    <t t-set="btn_text" t-value="step.button_text"/>
                    <t t-set="done_text" t-value="step.done_text"/>
                    <t t-set="image" t-value="'/web/image/onboarding.onboarding.step/'+str(step.id)+'/step_image'"/>
                    <t t-set="alt" t-value="step.step_image_alt"/>
                    <!-- Model/method used by JS implementation of banners for each step-->
                    <t t-set="method" t-value="step.panel_step_open_action_name"/>
                    <t t-set="model">onboarding.onboarding.step</t>
                    <!-- to mimic first implementation of onboarding, the 'state' queried holds
                    a rendering state for all steps. See _get_and_update_onboarding_state -->
                    <t t-set="state" t-value="state.get(step.id)"/>
                </t>
            </t>
        </t>
    </template>

    <template id="onboarding_container">
        <div class="modal o_onboarding_modal o_technical_modal" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Hide Onboarding Tips</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"/>
                    </div>
                    <div class="modal-body">
                        <p>Are you sure you want to hide these configuration steps?</p>
                    </div>
                    <div class="modal-footer justify-content-start">
                        <a type="action" class="btn btn-primary me-1" data-bs-dismiss="modal"
                            data-o-hide-banner="true" t-att-data-model="close_model" t-att-data-method="close_method"
                        >
                            Get them out of my sight!
                        </a>
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="o_onboarding_main d-print-none position-relative border-bottom overflow-hidden">
            <div class="o_onboarding_wrap py-3 py-lg-4">
                <a href="#" data-bs-toggle="modal" data-bs-target=".o_onboarding_modal" class="o_onboarding_btn_close position-absolute top-0 end-0 py-2 px-3 h2" title="Close the onboarding panel"><i class="oi oi-close"/></a>
                <div class="o_onboarding_steps d-flex" t-out="0"/>

                <div t-if="state.get('onboarding_state') == 'just_done'"
                        t-att-state="state.get('onboarding_state')"
                        class="o_onboarding_completed_message position-absolute end-0 start-0 border-bottom py-4 bg-white d-flex align-items-center justify-content-center">
                    <span class="h3 m-0">
                        <i class="fa fa-check text-success me-3" />
                        <t t-if="text_completed" t-out="text_completed" />
                    </span>
                    <a type="action" class="btn btn-primary ms-4" t-att-data-model="close_model" t-att-data-method="close_method">
                        Close Panel
                    </a>
                </div>
            </div>
        </div>
    </template>

    <template id="onboarding_step">
        <div class="o_onboarding_step position-relative d-flex flex-column align-items-center justify-content-start text-center" t-att-data-step-state="state">
            <img t-if="state == 'just_done'" class="o_onboarding_confetti position-absolute w-100" src="/base/static/img/onboarding_confetti.svg" alt="o_onboarding_confetti"/>

            <div class="o_onboarding_line position-absolute"/>
            <div class="o_onboarding_step_side d-flex">
                <img class="z-1" t-attf-src="#{image}" t-attf-alt="#{alt}"/>
            </div>

            <div class="o_onboarding_step_content position-relative pt-2 flex-grow-1 d-flex flex-column align-items-center justify-content-around">
                <div class="o_onboarding_step_content_info flex-grow-1 mb-2">
                    <a type="action" data-reload-on-close="true" role="button" t-att-data-method="method" t-att-data-model="model">
                        <h5 class="o_onboarding_step_title mb-1" t-out="title"/>
                    </a>
                    <p class="m-0 small" t-out="description"/>
                </div>
                <a t-if="state == 'not_done'" class="o_onboarding_step_action btn px-4" type="action" data-reload-on-close="true" role="button" t-att-data-method="method" t-att-data-model="model">
                    <span>
                        <t t-if="btn_text" t-out="btn_text" />
                        <t t-else="">Let's do it</t>
                    </span>
                </a>
                <a t-else="" class="o_onboarding_step_action__done btn" type="action" data-reload-on-close="true" role="button" t-att-data-method="method" t-att-data-model="model">
                    <span>
                        <i t-attf-class="p-1 me-1 fa #{done_icon if done_icon else 'fa-check'} rounded-circle" />
                        <t t-if="done_text" t-out="done_text" />
                        <t t-else="">All done!</t>
                    </span>
                </a>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\onboarding_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="onboarding_onboarding_view_tree" model="ir.ui.view">
        <field name="name">onboarding.onboarding.view.list</field>
        <field name="model">onboarding.onboarding</field>
        <field name="arch" type="xml">
            <list string="Onboardings">
                <header>
                    <button name="action_toggle_visibility" type="object" string="Toggle visibility"/>
                </header>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="current_onboarding_state"/>
                <field name="is_onboarding_closed"/>
                <field name="is_per_company" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="onboarding_onboarding_view_form" model="ir.ui.view">
        <field name="name">onboarding.onboarding.view.form</field>
        <field name="model">onboarding.onboarding</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <field name="current_progress_id" invisible="1"/>
                    <button name="action_toggle_visibility" type="object" string="Toggle visibility"
                            invisible="not current_progress_id"/>
                </header>
                <sheet>
                    <group col="2">
                        <field name="name"/>
                        <field name="route_name"/>
                        <field name="is_per_company"/>
                        <field name="is_onboarding_closed"/>
                    </group>
                    <notebook>
                        <page name="Onboarding steps">
                            <group string="Steps">
                                <field name="step_ids" nolabel="1"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="onboarding_onboarding_step_view_tree" model="ir.ui.view">
        <field name="name">onboarding.onboarding.step.view.list</field>
        <field name="model">onboarding.onboarding.step</field>
        <field name="arch" type="xml">
            <list string="Onboarding Steps">
                <field name="sequence" widget="handle"/>
                <field name="title"/>
                <field name="onboarding_ids" optional="hide"/>
                <field name="current_step_state"/>
                <field name="is_per_company" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="onboarding_onboarding_step_view_form" model="ir.ui.view">
        <field name="name">onboarding.onboarding.step.view.form</field>
        <field name="model">onboarding.onboarding.step</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="title"/>
                        <field name="current_step_state"/>
                        <field name="panel_step_open_action_name"/>
                        <field name="is_per_company"/>
                    </group>
                    <notebook>
                        <page name="Onboardings using this step">
                            <group string="Onboardings">
                                <field name="onboarding_ids" nolabel="1"/>
                            </group>
                        </page>
                        <page name="Step rendering">
                            <group>
                                <field name="description"/>
                                <field name="button_text"/>
                                <field name="done_text"/>
                                <field name="done_icon"/>
                                <field name="step_image_filename" invisible="1"/>
                                <field name="step_image" filename="step_image_filename"/>
                                <field name="step_image_alt"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_view_onboarding_onboarding" model="ir.actions.act_window">
        <field name="name">Onboardings</field>
        <field name="res_model">onboarding.onboarding</field>
        <field name="view_mode">list,form</field>
    </record>

    <record id="action_view_onboarding_step" model="ir.actions.act_window">
        <field name="name">Onboarding Steps</field>
        <field name="res_model">onboarding.onboarding.step</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

