# Odoo Module: hr_gamification

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'HR Gamification',
    'version': '1.0',
    'category': 'Human Resources',
    'depends': ['gamification', 'hr'],
    'description': """Use the HR resources for the gamification process.

The HR officer can now manage challenges and badges.
This allow the user to send badges to employees instead of simple users.
Badge received are displayed on the user profile.
""",
    'data': [
        'security/gamification_security.xml',
        'security/ir.model.access.csv',
        'wizard/gamification_badge_user_wizard_views.xml',
        'views/gamification_views.xml',
        'views/hr_employee_views.xml',
        ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\gamification.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class GamificationBadgeUser(models.Model):
    """User having received a badge"""
    _inherit = 'gamification.badge.user'

    employee_id = fields.Many2one('hr.employee', string='Employee', index=True)

    @api.constrains('employee_id')
    def _check_employee_related_user(self):
        for badge_user in self:
            if badge_user.employee_id and badge_user.employee_id not in badge_user.user_id.\
                with_context(allowed_company_ids=self.env.user.company_ids.ids).employee_ids:
                raise ValidationError(_('The selected employee does not correspond to the selected user.'))

    def action_open_badge(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'gamification.badge',
            'view_mode': 'form',
            'res_id': self.badge_id.id,
        }

class GamificationBadge(models.Model):
    _inherit = 'gamification.badge'

    granted_employees_count = fields.Integer(compute="_compute_granted_employees_count")

    @api.depends('owner_ids.employee_id')
    def _compute_granted_employees_count(self):
        for badge in self:
            badge.granted_employees_count = self.env['gamification.badge.user'].search_count([
                ('badge_id', '=', badge.id),
                ('employee_id', '!=', False)
            ])

    def get_granted_employees(self):
        employee_ids = self.mapped('owner_ids.employee_id').ids
        return {
            'type': 'ir.actions.act_window',
            'name': 'Granted Employees',
            'view_mode': 'kanban,list,form',
            'res_model': 'hr.employee.public',
            'domain': [('id', 'in', employee_ids)]
        }

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    goal_ids = fields.One2many('gamification.goal', string='Employee HR Goals', compute='_compute_employee_goals')
    badge_ids = fields.One2many(
        'gamification.badge.user', string='Employee Badges', compute='_compute_employee_badges',
        help="All employee badges, linked to the employee either directly or through the user"
    )
    has_badges = fields.Boolean(compute='_compute_employee_badges')
    # necessary for correct dependencies of badge_ids and has_badges
    direct_badge_ids = fields.One2many(
        'gamification.badge.user', 'employee_id',
        help="Badges directly linked to the employee")

    @api.depends('user_id.goal_ids.challenge_id.challenge_category')
    def _compute_employee_goals(self):
        for employee in self:
            employee.goal_ids = self.env['gamification.goal'].search([
                ('user_id', '=', employee.user_id.id),
                ('challenge_id.challenge_category', '=', 'hr'),
            ])

    @api.depends('direct_badge_ids', 'user_id.badge_ids.employee_id')
    def _compute_employee_badges(self):
        for employee in self:
            badge_ids = self.env['gamification.badge.user'].search([
                '|', ('employee_id', 'in', employee.ids),
                     '&', ('employee_id', '=', False),
                          ('user_id', 'in', employee.user_id.ids)
            ])
            employee.has_badges = bool(badge_ids)
            employee.badge_ids = badge_ids

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    goal_ids = fields.One2many('gamification.goal', 'user_id')
    badge_ids = fields.One2many('gamification.badge.user', 'user_id')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gamification
from . import hr_employee
from . import res_users

```

## File: security\gamification_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="goal_gamification_hr_user_visibility" model="ir.rule">
        <field name="name">HR Officer can see any goal</field>
        <field name="model_id" ref="gamification.model_gamification_goal"/>
        <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
challenge_officer,"Challenge Officer",gamification.model_gamification_challenge,hr.group_hr_user,1,1,1,1
challenge_line_officer,"Challenge Line Officer",gamification.model_gamification_challenge_line,hr.group_hr_user,1,1,1,1
badge_officer,"Badge Officer",gamification.model_gamification_badge,hr.group_hr_user,1,1,1,1
badge_user_officer,"Badge-user Officer",gamification.model_gamification_badge_user,hr.group_hr_user,1,1,1,1

```

## File: views\gamification_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

        <record id="hr_badge_form_view" model="ir.ui.view">
            <field name="name">gamification.badge.form.inherit</field>
            <field name="model">gamification.badge</field>
            <field name="inherit_id" ref="gamification.badge_form_view"/>
            <field name="arch" type="xml">
                <div name="button_box" position="inside">
                    <button class="oe_stat_button" type="object" name="get_granted_employees" invisible="granted_count == 0" icon="fa-user">
                        <field name="granted_employees_count" string="Granted" widget="statinfo"/>
                    </button>
                </div>
            </field>
        </record>

        <record id="goals_menu_groupby_action2" model="ir.actions.act_window">
            <field name="res_model">gamification.goal</field>
            <field name="name">Goals History</field>
            <field name="view_mode">list,kanban</field>
            <field name="context">{'search_default_group_by_user': True, 'search_default_group_by_definition': True}</field>
            <field name="domain">[('challenge_id.challenge_category', '=', 'hr')]</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new goal
                </p><p>
                    A goal is defined by a user and a goal type.
                    Goals can be created automatically by using challenges.
                </p>
            </field>
        </record>


        <record id="challenge_list_action2" model="ir.actions.act_window">
            <field name="name">Challenges</field>
            <field name="res_model">gamification.challenge</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="domain">[('challenge_category', '=', 'hr')]</field>
            <field name="context">{'search_default_inprogress':True, 'default_inprogress':True}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a new challenge
                </p><p>
                    Assign a list of goals to chosen users to evaluate them.
                    The challenge can use a period (weekly, monthly...) for automatic creation of goals.
                    The goals are created for the specified users or member of the group.
                </p>
            </field>
        </record>
        <record id="challenge_list_action2_view1" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="challenge_list_action2"/>
            <field name="view_id" ref="gamification.view_challenge_kanban"/>
        </record>
        <record id="challenge_list_action2_view2" model="ir.actions.act_window.view">
            <field name="sequence" eval="10"/>
            <field name="view_mode">form</field>
            <field name="act_window_id" ref="challenge_list_action2"/>
            <field name="view_id" ref="gamification.challenge_form_view"/>
        </record>

        <menuitem id="menu_hr_gamification" parent="hr.menu_human_resources_configuration" name="Challenges" sequence="100"/>

        <menuitem id="gamification_badge_menu_hr" parent="menu_hr_gamification" action="gamification.badge_list_action" />
        <menuitem id="gamification_challenge_menu_hr" parent="menu_hr_gamification" action="challenge_list_action2" groups="hr.group_hr_user"/>
        <menuitem id="gamification_goal_menu_hr" parent="menu_hr_gamification" action="goals_menu_groupby_action2" groups="hr.group_hr_user"/>

</odoo>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="hr_hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">

            <xpath expr="//page[@name='public']" position="after">
                <page string="Received Badges" name="received_badges" invisible="not user_id">
                    <field name="has_badges" invisible="1"/>
                    <button string="Grant a Badge" type="action" name="%(action_reward_wizard)d"/> to reward this employee for a good action
                    <div class="o_field_nocontent mt-2" invisible="has_badges">
                        <p>
                            Grant this employee his first badge
                        </p><p class="oe_grey">
                            Badges are rewards of good work. Give them to people you believe deserve it.
                        </p>
                    </div>
                    <div class="mt-2">
                        <field name="badge_ids" mode="kanban" />
                    </div>
                </page>
            </xpath>

        </field>
    </record>

    <record id="hr_employee_public_view_form" model="ir.ui.view">
        <field name="name">hr.employee.public.view.form.inherit</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">

            <xpath expr="//page[@name='public']" position="after">
                <page string="Received Badges" name="received_badges" invisible="not user_id">
                    <field name="has_badges" invisible="1"/>
                    <button string="Grant a Badge" type="action" name="%(action_reward_wizard)d"/> to reward this employee for a good action
                    <div class="o_field_nocontent" invisible="has_badges">
                        <p>
                            Grant this employee his first badge
                        </p><p class="oe_grey">
                            Badges are rewards of good work. Give them to people you believe deserve it.
                        </p>
                    </div>
                    <field name="badge_ids" mode="kanban" widget="many2many"/>
                </page>
            </xpath>

        </field>
    </record>

</odoo>

```

## File: wizard\gamification_badge_user_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, AccessError


class GamificationBadgeUserWizard(models.TransientModel):
    _inherit = 'gamification.badge.user.wizard'

    employee_id = fields.Many2one('hr.employee', string='Employee', required=False)
    user_id = fields.Many2one('res.users', string='User', compute='_compute_user_id',
        store=True, readonly=False, compute_sudo=True)

    def action_grant_badge(self):
        """Wizard action for sending a badge to a chosen employee"""
        if self.env.uid == self.user_id.id:
            raise UserError(_('You can not send a badge to yourself.'))
        values = {
            'user_id': self.user_id.id,
            'sender_id': self.env.uid,
            'badge_id': self.badge_id.id,
            'employee_id': self.user_id.employee_id.id,
            'comment': self.comment,
        }

        return self.env['gamification.badge.user'].create(values)._send_badge()

    @api.depends('employee_id')
    def _compute_user_id(self):
        for wizard in self:
            wizard.user_id = wizard.employee_id.user_id

```

## File: wizard\gamification_badge_user_wizard_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

        <record id="view_badge_wizard_grant_employee" model="ir.ui.view">
            <field name="name">gamification.badge.user.wizard.form.inherit</field>
            <field name="model">gamification.badge.user.wizard</field>
            <field name="inherit_id" ref="gamification.view_badge_wizard_grant" />
            <field name="arch" type="xml">
                <data>
                    <!--remove in master-->
                    <xpath expr="//field[@name='user_id']" position="after">
                        <field name="employee_id" nolabel="1" invisible="1" domain="[('user_id', '!=', False),('user_id', '!=', uid)]" colspan="2"/>
                    </xpath>
                </data>
            </field>
        </record>

        <record id="view_badge_wizard_reward" model="ir.ui.view">
            <field name="name">gamification.badge.user.wizard.form</field>
            <field name="model">gamification.badge.user.wizard</field>
            <field name="arch" type="xml">
                <form string="Reward Employee with">
                    What are you thankful for?
                    <group>
                        <group>
                            <field name="employee_id" invisible="1" />
                            <field name="user_id" invisible="1" />
                            <field name="badge_id" nolabel="1" colspan="4" />
                        </group>
                    </group>
                    <field name="comment" nolabel="1" placeholder="Describe what they did and why it matters (will be public)" />
                    <footer>
                        <button string="Reward Employee" type="object" name="action_grant_badge" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" special="cancel" data-hotkey="x" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_reward_wizard" model="ir.actions.act_window">
            <field name="name">Reward Employee</field>
            <field name="res_model">gamification.badge.user.wizard</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_badge_wizard_reward"/>
            <field name="target">new</field>
            <field name="domain">[]</field>
            <field name="context">{'default_employee_id': active_id, 'employee_id': active_id}</field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gamification_badge_user_wizard

```

