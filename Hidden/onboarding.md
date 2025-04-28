# Odoo Module: onboarding

Category: Hidden

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
    'name': 'Onboarding toolbox',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 9001,
    'description': """
This module allows to manage onboardings and their progress
================================================================================
    """,
    'depends': ['base'],
    'installable': True,
    'data': [
        'data/onboarding_data.xml',
        'views/onboarding_views.xml',
        'views/onboarding_menus.xml',
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\onboarding.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class OnboardingController(http.Controller):
    @http.route('/onboarding/<string:route_name>', auth='user', type='json')
    def get_onboarding_data(self, route_name=None):
        if not request.env.user.has_group('base.group_system'):
            return {}

        onboarding = request.env['onboarding.onboarding'].search([('route_name', '=', route_name)])
        if onboarding and not onboarding._search_or_create_progress().is_onboarding_closed:
            # JS implementation of the onboarding panel expects this data structure
            return {
                'html': request.env['ir.qweb']._render(
                    'onboarding.onboarding_panel', onboarding._prepare_rendering_values())
            }

        return {}

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import onboarding

```

## File: data\onboarding_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="onboarding_panel">
        <t t-call="onboarding.onboarding_container">
            <t t-foreach="steps" t-as="step">
                <t t-call="base.onboarding_step">
                    <t t-set="title" t-value="step.title"/>
                    <t t-set="description" t-value="step.description"/>
                    <t t-set="done_icon" t-value="step.done_icon"/>
                    <t t-set="btn_text" t-value="step.button_text"/>
                    <t t-set="done_text" t-value="step.done_text"/>
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

    <template id="onboarding_container" inherit_id="base.onboarding_container">
        <xpath expr="//div[hasclass('o_onboarding_completed_message')]" position="replace"/>
    </template>
</odoo>

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
    step_ids = fields.One2many('onboarding.onboarding.step', 'onboarding_id', 'Onboarding steps')

    is_per_company = fields.Boolean('Should be done per company?', default=True)

    panel_background_color = fields.Selection(
        [('orange', 'Orange'), ('blue', 'Blue'), ('violet', 'Violet'), ('none', 'None')],
        string="Panel's Background color", default='orange',
        help="Color gradient added to the panel's background.")
    panel_background_image = fields.Binary("Panel's background image")
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

    @api.depends_context('company')
    @api.depends('progress_ids', 'progress_ids.is_onboarding_closed', 'progress_ids.onboarding_state')
    def _compute_current_progress(self):
        for onboarding in self:
            current_progress_id = onboarding.progress_ids.filtered(
                lambda progress: progress.company_id.id in {False, self.env.company.id})
            if current_progress_id:
                if len(current_progress_id) > 1:
                    current_progress_id = current_progress_id.sorted('create_date', reverse=True)[0]
                onboarding.current_onboarding_state = current_progress_id.onboarding_state
                onboarding.current_progress_id = current_progress_id
                onboarding.is_onboarding_closed = current_progress_id.is_onboarding_closed
            else:
                onboarding.current_onboarding_state = 'not_done'
                onboarding.current_progress_id = False
                onboarding.is_onboarding_closed = False

    def action_close(self):
        """Close the onboarding panel."""
        self.current_progress_id.action_close()

    def action_toggle_visibility(self):
        self.current_progress_id.action_toggle_visibility()

    def write(self, values):
        if 'is_per_company' in values:
            onboardings_per_company_update = self.filtered(
                lambda onboarding: onboarding.is_per_company != values['is_per_company'])

        res = super().write(values)

        if 'is_per_company' in values:
            # When changing this parameter, all progress (onboarding and steps) is reset.
            onboardings_per_company_update.progress_ids.unlink()
        return res

    def _search_or_create_progress(self):
        """Create Progress record(s) as necessary for the context.
        """
        onboardings_without_progress = self.filtered(lambda onboarding: not onboarding.current_progress_id)
        onboardings_without_progress._create_progress()
        return self.current_progress_id

    def _create_progress(self):
        return self.env['onboarding.progress'].create([
            {
                'company_id': self.env.company.id if onboarding.is_per_company else False,
                'onboarding_id': onboarding.id
            } for onboarding in self
        ])

    def _prepare_rendering_values(self):
        self.ensure_one()
        values = {
            'bg_image': f'/web/image/onboarding.onboarding/{self.id}/panel_background_image',
            'classes': f'o_onboarding_{self.panel_background_color}'
                       if self.panel_background_color not in {False, 'none'} else '',
            'close_method': self.panel_close_action_name,
            'close_model': 'onboarding.onboarding',
            'steps': self.step_ids,
            'state': self.current_progress_id._get_and_update_onboarding_state(),
        }

        return values

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
    company_id = fields.Many2one('res.company')
    onboarding_id = fields.Many2one(
        'onboarding.onboarding', 'Related onboarding tracked', required=True, ondelete='cascade')
    progress_step_ids = fields.One2many('onboarding.progress.step', 'progress_id', 'Progress Steps Trackers')
    _sql_constraints = [
        ('onboarding_company_uniq', 'unique (onboarding_id,company_id)',
         'There cannot be multiple records of the same onboarding completion for the same company.'),
    ]

    @api.depends('onboarding_id.step_ids', 'progress_step_ids', 'progress_step_ids.step_state')
    def _compute_onboarding_state(self):
        progress_steps_data = self.env['onboarding.progress.step'].read_group(
            [('progress_id', 'in', self.ids), ('step_state', 'in', ['just_done', 'done'])],
            ['progress_id'], ['progress_id']
        )
        result = dict((data['progress_id'][0], data['progress_id_count']) for data in progress_steps_data)
        for progress in self:
            progress.onboarding_state = (
                'not_done' if result.get(progress.id, 0) != len(progress.onboarding_id.step_ids)
                else 'done')

    def action_close(self):
        self.is_onboarding_closed = True

    def action_toggle_visibility(self):
        for progress in self:
            progress.is_onboarding_closed = not progress.is_onboarding_closed

    def _get_and_update_onboarding_state(self):
        """Used to fetch the progress of an onboarding for rendering its panel and is expected to
        be called by the onboarding controller. It also has the responsibility of updating the
        'just_done' states into 'done' so that the 'just_done' states are only rendered once.
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

    progress_id = fields.Many2one(
        'onboarding.progress', 'Related Onboarding Progress Tracker', required=True, ondelete='cascade')
    step_state = fields.Selection(
        ONBOARDING_PROGRESS_STATES, string='Onboarding Step Progress', default='not_done')
    onboarding_id = fields.Many2one(related='progress_id.onboarding_id', string='Onboarding')
    step_id = fields.Many2one(
        'onboarding.onboarding.step', string='Onboarding Step', required=True, ondelete='cascade')

    _sql_constraints = [
        ('progress_step_uniq', 'unique (progress_id, step_id)',
         'There cannot be multiple records of the same onboarding step completion for the same Progress record.'),
    ]

    def action_consolidate_just_done(self):
        was_just_done = self.filtered(lambda progress: progress.step_state == 'just_done')
        was_just_done.step_state = 'done'
        return was_just_done

    def action_set_just_done(self):
        not_done = self.filtered_domain([('step_state', '=', 'not_done')])
        not_done.step_state = 'just_done'
        return not_done

```

## File: models\onboarding_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.addons.onboarding.models.onboarding_progress import ONBOARDING_PROGRESS_STATES


class OnboardingStep(models.Model):
    _name = 'onboarding.onboarding.step'
    _description = 'Onboarding Step'
    _order = 'sequence asc, id asc'
    _rec_name = 'title'

    onboarding_id = fields.Many2one(
        'onboarding.onboarding', string='Onboarding', readonly=True, required=True, ondelete='cascade')

    title = fields.Char('Title', translate=True)
    description = fields.Char('Description', translate=True)
    button_text = fields.Char(
        'Button text', required=True, default=_("Let's do it"), translate=True,
        help="Text on the panel's button to start this step")
    done_icon = fields.Char('Font Awesome Icon when completed', default='fa-star')
    done_text = fields.Char(
        'Text to show when step is completed', default=_('Step Completed! - Click to review'), translate=True)
    panel_step_open_action_name = fields.Char(
        string='Opening action', required=True,
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

    sequence = fields.Integer(default=10)

    @api.depends_context('company')
    @api.depends('progress_ids', 'progress_ids.step_state')
    def _compute_current_progress(self):
        existing_progress_steps = self.progress_ids.filtered_domain([
            ('step_id', 'in', self.ids),
            ('progress_id.company_id', 'in', [False, self.env.company.id]),
        ])
        for step in self:
            if step in existing_progress_steps.step_id:
                current_progress_step_id = existing_progress_steps.filtered(
                    lambda progress_step: progress_step.step_id == step)
                if len(current_progress_step_id) > 1:
                    current_progress_step_id = current_progress_step_id.sorted('create_date', reverse=True)[0]
                step.current_progress_step_id = current_progress_step_id
                step.current_step_state = current_progress_step_id.step_state
            else:
                step.current_progress_step_id = False
                step.current_step_state = 'not_done'

    def action_set_just_done(self):
        # Make sure progress records exist for the current context (company)
        steps_without_progress = self.filtered(lambda step: not step.current_progress_step_id)
        steps_without_progress._create_progress_steps()
        return self.current_progress_step_id.action_set_just_done()

    def _create_progress_steps(self):
        onboarding_progress_records = self.env['onboarding.progress'].search([
            ('onboarding_id', 'in', self.onboarding_id.ids),
            ('company_id', 'in', [False, self.env.company.id])
        ])
        progress_step_values = []
        for onboarding_progress_record in onboarding_progress_records:
            progress_step_values += [
                {
                    'onboarding_id': onboarding_progress_record.onboarding_id.id,
                    'progress_id': onboarding_progress_record.id,
                    'step_id': step_id.id,
                }
                for step_id
                in self.filtered(lambda step: self.onboarding_id == onboarding_progress_record.onboarding_id)
                if step_id not in onboarding_progress_record.progress_step_ids.step_id
            ]

        return self.env['onboarding.progress.step'].create(progress_step_values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import onboarding_onboarding
from . import onboarding_step
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

## File: views\onboarding_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="onboarding_onboarding_view_tree" model="ir.ui.view">
        <field name="name">onboarding.onboarding.view.tree</field>
        <field name="model">onboarding.onboarding</field>
        <field name="arch" type="xml">
            <tree string="Onboardings">
                <header>
                    <button name="action_toggle_visibility" type="object" string="Toggle visibility"/>
                </header>
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="current_onboarding_state"/>
                <field name="is_onboarding_closed"/>
                <field name="is_per_company" optional="hide"/>
            </tree>
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
                            attrs="{'invisible': [('current_progress_id', '=', False)]}"/>
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
                            <field name="step_ids"/>
                        </page>
                        <page name="Panel Rendering">
                            <group>
                                <field name="panel_background_color" string="Background color"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="onboarding_onboarding_step_view_tree" model="ir.ui.view">
        <field name="name">onboarding.onboarding.step.view.tree</field>
        <field name="model">onboarding.onboarding.step</field>
        <field name="arch" type="xml">
            <tree string="Onboarding Steps">
                <field name="sequence" widget="handle"/>
                <field name="title"/>
                <field name="onboarding_id" optional="hide"/>
                <field name="current_step_state"/>
            </tree>
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
                        <field name="onboarding_id"/>
                        <field name="panel_step_open_action_name"/>
                    </group>
                    <notebook>
                        <page name="Step rendering">
                            <group>
                                <field name="description"/>
                                <field name="button_text"/>
                                <field name="done_text"/>
                                <field name="done_icon"/>
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
        <field name="view_mode">tree,form</field>
    </record>

    <record id="action_view_onboarding_step" model="ir.actions.act_window">
        <field name="name">Onboarding Steps</field>
        <field name="res_model">onboarding.onboarding.step</field>
        <field name="view_mode">tree,form</field>
    </record>
</odoo>

```

