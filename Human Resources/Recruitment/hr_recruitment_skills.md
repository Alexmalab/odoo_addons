# Odoo Module: hr_recruitment_skills

Category: Human Resources/Recruitment

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
    'name': 'Recruitment - Skills Management',
    'category': 'Human Resources/Recruitment',
    'sequence': 270,
    'version': '1.0',
    'summary': 'Manage skills of your employees',
    'description': """""",
    'depends': ['hr_skills', 'hr_recruitment'],
    'data': [
        'security/hr_recruitment_skills_security.xml',
        'views/hr_applicant_views.xml',
        'views/hr_applicant_skill_views.xml',
        'security/ir.model.access.csv',
    ],
    'demo': [
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\hr_applicant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    applicant_skill_ids = fields.One2many('hr.applicant.skill', 'applicant_id', string="Skills")
    skill_ids = fields.Many2many('hr.skill', compute='_compute_skill_ids', store=True)
    is_interviewer = fields.Boolean(compute='_compute_is_interviewer')

    @api.depends_context('uid')
    @api.depends('interviewer_ids', 'job_id.interviewer_ids')
    def _compute_is_interviewer(self):
        is_recruiter = self.user_has_groups('hr_recruitment.group_hr_recruitment_user')
        for applicant in self:
            applicant.is_interviewer = not is_recruiter and self.env.user in (applicant.interviewer_ids | applicant.job_id.interviewer_ids)

    @api.depends('applicant_skill_ids.skill_id')
    def _compute_skill_ids(self):
        for applicant in self:
            applicant.skill_ids = applicant.applicant_skill_ids.skill_id

    def create_employee_from_applicant(self):
        self.ensure_one()
        action = super().create_employee_from_applicant()
        action['context']['default_employee_skill_ids'] = [(0, 0, {
            'skill_id': applicant_skill.skill_id.id,
            'skill_level_id': applicant_skill.skill_level_id.id,
            'skill_type_id': applicant_skill.skill_type_id.id,
        }) for applicant_skill in self.applicant_skill_ids]
        return action

    def _update_employee_from_applicant(self):
        vals_list = []
        for applicant in self:
            existing_skills = applicant.emp_id.employee_skill_ids.skill_id
            skills_to_create = applicant.applicant_skill_ids.skill_id - existing_skills
            vals_list.extend([{
                'employee_id': applicant.emp_id.id,
                'skill_id': skill.id,
                'skill_level_id': applicant.applicant_skill_ids.filtered(lambda s: s.skill_id == skill).skill_level_id.id,
                'skill_type_id': skill.skill_type_id.id,
            } for skill in skills_to_create])
        self.env['hr.employee.skill'].create(vals_list)
        return super()._update_employee_from_applicant()

```

## File: models\hr_applicant_skill.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ApplicantSkill(models.Model):
    _name = 'hr.applicant.skill'
    _description = "Skill level for an applicant"
    _rec_name = 'skill_id'
    _order = "skill_level_id"

    applicant_id = fields.Many2one(
        comodel_name='hr.applicant',
        required=True,
        ondelete='cascade')
    skill_id = fields.Many2one(
        comodel_name='hr.skill',
        compute='_compute_skill_id',
        store=True,
        domain="[('skill_type_id', '=', skill_type_id)]",
        readonly=False,
        required=True)
    skill_level_id = fields.Many2one(
        comodel_name='hr.skill.level',
        compute='_compute_skill_level_id',
        domain="[('skill_type_id', '=', skill_type_id)]",
        store=True,
        readonly=False,
        required=True)
    skill_type_id = fields.Many2one(
        comodel_name='hr.skill.type',
        required=True)
    level_progress = fields.Integer(
        related='skill_level_id.level_progress')

    _sql_constraints = [
        ('_unique_skill', 'unique (applicant_id, skill_id)', "Two levels for the same skill is not allowed"),
    ]

    @api.constrains('skill_id', 'skill_type_id')
    def _check_skill_type(self):
        for applicant in self:
            if applicant.skill_id not in applicant.skill_type_id.skill_ids:
                raise ValidationError(_("The skill %(name)s and skill type %(type)s doesn't match", name=applicant.skill_id.name, type=applicant.skill_type_id.name))

    @api.constrains('skill_type_id', 'skill_level_id')
    def _check_skill_level(self):
        for applicant in self:
            if applicant.skill_level_id not in applicant.skill_type_id.skill_level_ids:
                raise ValidationError(_("The skill level %(level)s is not valid for skill type: %(type)s", level=applicant.skill_level_id.name, type=applicant.skill_type_id.name))

    @api.depends('skill_type_id')
    def _compute_skill_id(self):
        for applicant in self:
            if applicant.skill_id.skill_type_id != applicant.skill_type_id:
                applicant.skill_id = False

    @api.depends('skill_id')
    def _compute_skill_level_id(self):
        for applicant in self:
            if not applicant.skill_id:
                applicant.skill_level_id = False
            else:
                skill_levels = applicant.skill_type_id.skill_level_ids
                applicant.skill_level_id = skill_levels.filtered('default_level') or skill_levels[0] if skill_levels else False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_applicant
from . import hr_applicant_skill

```

## File: security\hr_recruitment_skills_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Interviewer Access Rules -->
    <record id="hr_applicant_skill_interviewer_rule" model="ir.rule">
        <field name="name">Applicant Skill: Interviewer</field>
        <field name="model_id" ref="model_hr_applicant_skill"/>
        <field name="domain_force">[
            '|',
                ('applicant_id.job_id.interviewer_ids', 'in', user.id),
                ('applicant_id.interviewer_ids', 'in', user.id),
        ]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
hr_recruitment_skills.access_hr_applicant_skill,access_hr_applicant_skill,hr_recruitment_skills.model_hr_applicant_skill,hr_recruitment.group_hr_recruitment_user,1,1,1,1
hr_recruitment_skills.access_hr_applicant_skill_interviewer,access_hr_applicant_skill_interviewer,hr_recruitment_skills.model_hr_applicant_skill,hr_recruitment.group_hr_recruitment_interviewer,1,0,0,0

```

## File: views\hr_applicant_skill_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_applicant_skill_view_form" model="ir.ui.view">
        <field name="name">hr.applicant.skill.view.form</field>
        <field name="model">hr.applicant.skill</field>
        <field name="arch" type="xml">
            <form string="Skills" class="o_hr_skills_dialog_form">
                <sheet>
                    <group>
                        <group>
                            <field name="applicant_id" invisible="1"/>
                            <field name="skill_type_id" widget="radio" />
                        </group>
                        <group>
                            <field name="skill_id" options="{'no_open': True, 'no_create_edit': True}"
                                    context="{'default_skill_type_id': skill_type_id}"
                                    domain="[('skill_type_id', '=', skill_type_id)]"
                                    attrs="{'invisible': [('skill_type_id', '=', False)]}"/>
                            <label for="skill_level_id"
                                    attrs="{'invisible': ['|', ('skill_id', '=', False), ('skill_type_id', '=', False)]}"/>
                            <div class="o_row"
                                    attrs="{'invisible': ['|', ('skill_id', '=', False), ('skill_type_id', '=', False)]}">
                                <span class="ps-0" style="flex:1">
                                    <field name="skill_level_id"
                                            attrs="{'readonly': [('skill_id', '=', False)]}"
                                            context="{'from_skill_level_dropdown': True}" />
                                </span>
                                <span style="flex:1">
                                    <field name="level_progress" widget="progressbar" class="o_hr_skills_progress" attrs="{'invisible': [('skill_level_id', '=', False)]}" />
                                </span>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\hr_applicant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_applicant_view_form" model="ir.ui.view">
        <field name="name">hr.applicant.view.form.inherit.hr.recruitment.skills</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page string="Skills">
                    <div class="row">
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field name="id" invisible="1"/>
                            <field name="is_interviewer" invisible="1"/>
                            <field mode="tree" nolabel="1" name="applicant_skill_ids" widget="skills_one2many"
                                context="{'default_applicant_id': id}" attrs="{'readonly': [('is_interviewer', '=', True)]}">
                                <tree>
                                    <field name="skill_type_id" invisible="1"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </tree>
                            </field>
                        </div>
                    </div>
                </page>
            </notebook>
        </field>
    </record>

    <record id="hr_applicant_view_search_bis" model="ir.ui.view">
        <field name="name">hr.applicant.view.search.inherit.skills.bis</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_search_bis"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='email_from']" position="after">
                <field name="applicant_skill_ids"/>
            </xpath>
            <filter name="refuse_reason_id" position="after">
                <filter string="Skills" name="groupby_skills" context="{'group_by': 'skill_ids'}"/>
            </filter>
        </field>
    </record>

    <record id="hr_applicant_view_search" model="ir.ui.view">
        <field name="name">hr.applicant.view.search.inherit.skills</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="applicant_skill_ids"/>
            </xpath>
            <filter name="stage" position="after">
                <filter string="Skills" name="groupby_skills" context="{'group_by': 'skill_ids'}"/>
            </filter>
        </field>
    </record>
</odoo>

```

