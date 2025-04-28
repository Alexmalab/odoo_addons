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
    'depends': ['hr_skills', 'hr_recruitment'],
    'data': [
        'security/hr_recruitment_skills_security.xml',
        'views/hr_applicant_views.xml',
        'views/hr_candidate_views.xml',
        'views/hr_candidate_skill_views.xml',
        'views/hr_job_views.xml',
        'security/ir.model.access.csv',
    ],
    'assets': {
        'web.assets_backend': [
            'hr_recruitment_skills/static/src/**/*',
        ],
    },
    'demo': [
        'data/hr_recruitment_skills_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\hr_recruitment_skills_demo.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="hr_candidate_mkt0_skill_lang_en" model="hr.candidate.skill">
        <field name="candidate_id" ref="hr_recruitment.hr_candidate_mkt0"/>
        <field name="skill_id" ref="hr_skills.hr_skill_english"/>
        <field name="skill_level_id" ref="hr_skills.hr_skill_level_c2"/>
        <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
    </record>
    <record id="hr_candidate_mkt0_skill_lang_fr" model="hr.candidate.skill">
        <field name="candidate_id" ref="hr_recruitment.hr_candidate_mkt0"/>
        <field name="skill_id" ref="hr_skills.hr_skill_french"/>
        <field name="skill_level_id" ref="hr_skills.hr_skill_level_b2"/>
        <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
    </record>
    <record id="hr_candidate_mkt0_skill_softskill_org" model="hr.candidate.skill">
        <field name="candidate_id" ref="hr_recruitment.hr_candidate_mkt0"/>
        <field name="skill_id" ref="hr_skills.hr_skill_organizational"/>
        <field name="skill_level_id" ref="hr_skills.hr_skill_level_intermediate_softskill"/>
        <field name="skill_type_id" ref="hr_skills.hr_skill_type_softskill"/>
    </record>

    <record id="hr_candidate_mkt1_skill_lang_en" model="hr.candidate.skill">
        <field name="candidate_id" ref="hr_recruitment.hr_candidate_mkt1"/>
        <field name="skill_id" ref="hr_skills.hr_skill_english"/>
        <field name="skill_level_id" ref="hr_skills.hr_skill_level_c2"/>
        <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
    </record>
    <record id="hr_candidate_mkt1_skill_lang_fr" model="hr.candidate.skill">
        <field name="candidate_id" ref="hr_recruitment.hr_candidate_mkt1"/>
        <field name="skill_id" ref="hr_skills.hr_skill_french"/>
        <field name="skill_level_id" ref="hr_skills.hr_skill_level_c1"/>
        <field name="skill_type_id" ref="hr_skills.hr_skill_type_lang"/>
    </record>

     <function model="hr.job" name="write">
            <value model="hr.job" search="[
                ('name', '=', 'Chief Executive Officer'),
            ]
            "/>
            <value eval="{
                'skill_ids': [
                    Command.set([
                        ref('hr_skills.hr_skill_french'),
                        ref('hr_skills.hr_skill_english'),
                        ref('hr_skills.hr_skill_organizational'),
                    ]),
                ],
            }"/>
        </function>
</odoo>

```

## File: models\hr_applicant.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrApplicant(models.Model):
    _inherit = 'hr.applicant'

    candidate_skill_ids = fields.One2many(related="candidate_id.candidate_skill_ids", readonly=False)
    skill_ids = fields.Many2many(related="candidate_id.skill_ids", readonly=False)

```

## File: models\hr_candidate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import fields, models, api


class HrCandidate(models.Model):
    _inherit = 'hr.candidate'

    candidate_skill_ids = fields.One2many('hr.candidate.skill', 'candidate_id', string="Skills")
    skill_ids = fields.Many2many('hr.skill', compute='_compute_skill_ids', store=True)
    matching_skill_ids = fields.Many2many(comodel_name='hr.skill', string="Matching Skills", compute="_compute_matching_skill_ids")
    missing_skill_ids = fields.Many2many(comodel_name='hr.skill', string="Missing Skills", compute="_compute_matching_skill_ids")
    matching_score = fields.Float(string="Matching Score(%)", compute="_compute_matching_skill_ids")

    @api.depends_context('active_id')
    @api.depends('skill_ids')
    def _compute_matching_skill_ids(self):
        job_id = self.env.context.get('active_id')
        if not job_id:
            self.matching_skill_ids = False
            self.missing_skill_ids = False
            self.matching_score = 0
        else:
            for candidate in self:
                job_skills = self.env['hr.job'].browse(job_id).skill_ids
                candidate.matching_skill_ids = job_skills & candidate.skill_ids
                candidate.missing_skill_ids = job_skills - candidate.skill_ids
                candidate.matching_score = (len(candidate.matching_skill_ids) / len(job_skills)) * 100 if job_skills else 0

    @api.depends('candidate_skill_ids.skill_id')
    def _compute_skill_ids(self):
        for candidate in self:
            candidate.skill_ids = candidate.candidate_skill_ids.skill_id

    def _get_employee_create_vals(self):
        vals = super()._get_employee_create_vals()
        vals['employee_skill_ids'] = [(0, 0, {
            'skill_id': candidate_skill.skill_id.id,
            'skill_level_id': candidate_skill.skill_level_id.id,
            'skill_type_id': candidate_skill.skill_type_id.id,
        }) for candidate_skill in self.candidate_skill_ids]
        return vals

    def action_create_application(self):
        job = self.env['hr.job'].browse(self.env.context.get('active_id'))
        self.env['hr.applicant'].with_context(just_moved=True).create([{
            'candidate_id': candidate.id,
            'job_id': job.id,
        } for candidate in self])
        action = self.env['ir.actions.actions']._for_xml_id('hr_recruitment.action_hr_job_applications')
        action['context'] = literal_eval(action['context'].replace('active_id', str(job.id)))
        return action

```

## File: models\hr_candidate_skill.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class CandidateSkill(models.Model):
    _name = 'hr.candidate.skill'
    _description = "Skill level for a candidate"
    _rec_name = 'skill_id'
    _order = "skill_level_id"

    candidate_id = fields.Many2one(
        comodel_name='hr.candidate',
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
        ('_unique_skill', 'unique (candidate_id, skill_id)', "Two levels for the same skill is not allowed"),
    ]

    @api.constrains('skill_id', 'skill_type_id')
    def _check_skill_type(self):
        for candidate_skill in self:
            if candidate_skill.skill_id not in candidate_skill.skill_type_id.skill_ids:
                raise ValidationError(_("The skill %(name)s and skill type %(type)s doesn't match", name=candidate_skill.skill_id.name, type=candidate_skill.skill_type_id.name))

    @api.constrains('skill_type_id', 'skill_level_id')
    def _check_skill_level(self):
        for candidate_skill in self:
            if candidate_skill.skill_level_id not in candidate_skill.skill_type_id.skill_level_ids:
                raise ValidationError(_("The skill level %(level)s is not valid for skill type: %(type)s", level=candidate_skill.skill_level_id.name, type=candidate_skill.skill_type_id.name))

    @api.depends('skill_type_id')
    def _compute_skill_id(self):
        for candidate_skill in self:
            if candidate_skill.skill_id.skill_type_id != candidate_skill.skill_type_id:
                candidate_skill.skill_id = False

    @api.depends('skill_id')
    def _compute_skill_level_id(self):
        for candidate_skill in self:
            if not candidate_skill.skill_id:
                candidate_skill.skill_level_id = False
            else:
                skill_levels = candidate_skill.skill_type_id.skill_level_ids
                candidate_skill.skill_level_id = skill_levels.filtered('default_level') or skill_levels[0] if skill_levels else False

```

## File: models\hr_job.py

```python
from markupsafe import Markup
from ast import literal_eval

from odoo import fields, models, _


class HrJob(models.Model):
    _inherit = "hr.job"

    skill_ids = fields.Many2many(comodel_name='hr.skill', string="Expected Skills")

    def action_search_matching_candidates(self):
        self.ensure_one()
        help_message_1 = _("No Matching Candidates")
        help_message_2 = _("We do not have any candidates who meet the skill requirements for this job position in the database at the moment.")
        action = self.env['ir.actions.actions']._for_xml_id('hr_recruitment.action_hr_candidate')
        context = literal_eval(action['context'])
        context['active_id'] = self.id
        matching_candidates = self.env['hr.candidate'].search([('skill_ids', 'in', self.skill_ids.ids)]).filtered(lambda c: self.id not in c.applicant_ids.job_id.ids)
        action.update({
            'name': _("Matching Candidates"),
            'views': [
                (self.env.ref('hr_recruitment_skills.hr_candidate_view_tree').id, 'list'),
                (False, 'form'),
            ],
            'context': context,
            'domain': [('id', 'in', matching_candidates.ids)],
            'help': Markup("<p class='o_view_nocontent_empty_folder'>%s</p><p>%s</p>") % (help_message_1, help_message_2),
        })
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_applicant
from . import hr_candidate
from . import hr_candidate_skill
from . import hr_job

```

## File: security\hr_recruitment_skills_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Interviewer Access Rules -->
    <record id="hr_applicant_skill_interviewer_rule" model="ir.rule">
        <field name="name">Applicant Skill: Interviewer</field>
        <field name="model_id" ref="model_hr_candidate_skill"/>
        <field name="domain_force">[
            '|',
                ('candidate_id.applicant_ids.job_id.interviewer_ids', 'in', user.id),
                ('candidate_id.applicant_ids.interviewer_ids', 'in', user.id),
        ]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]"/>
    </record>

    <record id="hr_applicant_skill_officer_rule" model="ir.rule">
        <field name="name">Applicant Skill: Officer</field>
        <field name="model_id" ref="model_hr_candidate_skill"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_recruitment.group_hr_recruitment_user'))]"/>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
hr_recruitment_skills.access_hr_candidate_skill_interviewer,access_hr_candidate_skill_interviewer,hr_recruitment_skills.model_hr_candidate_skill,hr_recruitment.group_hr_recruitment_interviewer,1,1,1,1

```

## File: static\src\components\search_job_applicant_menu\search_job_applicant_menu.js

```javascript
import { Component, markup } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { STATIC_ACTIONS_GROUP_NUMBER } from "@web/search/action_menus/action_menus";

const cogMenuRegistry = registry.category("cogMenu");

/**
 * 'Search Matching Applicants' menu
 *
 * This component is used to search matching skills among the all applicants
 * It's only available in the kanban and list view of the applicants.
 * @extends Component
 */
export class SearchJobApplicant extends Component {
    static template = "hr_recruitment_skills.SearchJobApplicant";
    static components = { DropdownItem };
    static props = {};

    setup() {
        this.action = useService("action");
    }

    //---------------------------------------------------------------------
    // Protected
    //---------------------------------------------------------------------

    async openMatchingJobApplicants() {
        const { globalContext } = this.env.searchModel;
        const action = await this.env.services.orm.call(
            "hr.job",
            "action_search_matching_candidates",
            [globalContext.active_id],
        );
        action.help = markup(action.help);
        return this.action.doAction(action);
    }
}

export const searchJobApplicant = {
    Component: SearchJobApplicant,
    groupNumber: STATIC_ACTIONS_GROUP_NUMBER,
    isDisplayed: ({ config, searchModel }) => {
        return (
            searchModel.resModel === "hr.applicant" &&
            searchModel.globalContext.allow_search_matching_applicants &&
            config.viewArch.classList.contains('o_search_matching_applicant')
        );
    },
};

cogMenuRegistry.add("search-job-applicants-menu", searchJobApplicant, { sequence: 11 });

```

## File: static\src\components\search_job_applicant_menu\search_job_applicant_menu.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<templates id="template" xml:space="preserve">
    <t t-name="hr_recruitment_skills.SearchJobApplicant">
        <DropdownItem onSelected.bind="openMatchingJobApplicants">
            <i class="fa fa-fw fa-search me-1" aria-hidden="true"></i>Search Matching Applicants
        </DropdownItem>
    </t>
</templates>

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
                    <div class="row ms-2">
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field name="id" invisible="1"/>
                            <field mode="list" nolabel="1" name="candidate_skill_ids" widget="skills_one2many"
                                context="{'default_candidate_id': candidate_id, 'no_timeline': True}">
                                <list>
                                    <field name="skill_type_id" optional="hidden"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </list>
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
                <field name="candidate_skill_ids"/>
            </xpath>
        </field>
    </record>

    <record id="hr_applicant_view_search" model="ir.ui.view">
        <field name="name">hr.applicant.view.search.inherit.skills</field>
        <field name="model">hr.applicant</field>
        <field name="inherit_id" ref="hr_recruitment.hr_applicant_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="candidate_skill_ids"/>
            </xpath>
            <filter name="stage" position="after">
                <filter string="Skills" name="groupby_skills" context="{'group_by': 'skill_ids'}"/>
            </filter>
        </field>
    </record>
</odoo>

```

## File: views\hr_candidate_skill_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_applicant_skill_view_form" model="ir.ui.view">
        <field name="name">hr.candidate.skill.view.form</field>
        <field name="model">hr.candidate.skill</field>
        <field name="arch" type="xml">
            <form string="Skills" class="o_hr_skills_dialog_form">
                <sheet>
                    <group>
                        <group>
                            <field name="candidate_id" invisible="1"/>
                            <field name="skill_type_id" widget="radio" />
                        </group>
                        <group>
                            <field name="skill_id" options="{'no_open': True, 'no_create_edit': True}"
                                    context="{'default_skill_type_id': skill_type_id}"
                                    domain="[('skill_type_id', '=', skill_type_id)]"
                                    invisible="not skill_type_id"/>
                            <label for="skill_level_id"
                                    invisible="not skill_id or not skill_type_id"/>
                            <div class="o_row"
                                    invisible="not skill_id or not skill_type_id">
                                <span class="ps-0" style="flex:1">
                                    <field name="skill_level_id"
                                            readonly="not skill_id"
                                            context="{'from_skill_level_dropdown': True}" />
                                </span>
                                <span style="flex:1">
                                    <field name="level_progress" widget="progressbar" class="o_hr_skills_progress" invisible="not skill_level_id" />
                                </span>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <menuitem
        id="hr_recruitment_skill_type_menu"
        name="Skill Types"
        action="hr_skills.hr_skill_type_action"
        parent="hr_recruitment.menu_hr_recruitment_config_employees"
        sequence="35"
    />
</odoo>

```

## File: views\hr_candidate_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_candidate_view_form" model="ir.ui.view">
        <field name="name">hr.candidate.view.form.inherit.hr.recruitment.skills</field>
        <field name="model">hr.candidate</field>
        <field name="inherit_id" ref="hr_recruitment.hr_candidate_view_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page string="Skills">
                    <div class="row ms-2">
                        <div class="o_hr_skills_editable o_hr_skills_group o_group_skills col-lg-5 d-flex flex-column">
                            <field name="id" invisible="1"/>
                            <field mode="list" nolabel="1" name="candidate_skill_ids" widget="skills_one2many"
                                context="{'default_candidate_id': id, 'no_timeline': True}">
                                <list>
                                    <field name="skill_type_id" optional="hidden"/>
                                    <field name="skill_id"/>
                                    <field name="skill_level_id"/>
                                    <field name="level_progress" widget="progressbar"/>
                                </list>
                            </field>
                        </div>
                    </div>
                </page>
            </notebook>
        </field>
    </record>

    <record id="hr_candidate_view_search" model="ir.ui.view">
        <field name="name">hr.candidate.view.search.inherit.skills</field>
        <field name="model">hr.candidate</field>
        <field name="inherit_id" ref="hr_recruitment.hr_candidate_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="candidate_skill_ids"/>
            </xpath>
            <filter name="unassigned" position="after">
                <filter string="Skills" name="groupby_skills" context="{'group_by': 'skill_ids'}"/>
            </filter>
        </field>
    </record>

    <record id="hr_candidate_view_tree" model="ir.ui.view">
        <field name="name">hr.candidate.view.list.inherit.skills</field>
        <field name="model">hr.candidate</field>
        <field name="inherit_id" ref="hr_recruitment.hr_candidate_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="class" add="o_search_matching_applicant" separator=" "/>
            </xpath>
            <xpath expr="//list" position="inside">
                <header>
                    <button name="action_create_application" string="Create Application" type="object" class="btn-secondary"/>
                </header>
            </xpath>
            <field name="priority" position="attributes">
                <attribute name="optional">hide</attribute>
            </field>
            <field name="company_id" position="after">
                <field name="matching_score"/>
                <field name="matching_skill_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                <field name="missing_skill_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\hr_job_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_job_form_inherit_hr_recruitment_skills" model="ir.ui.view">
        <field name="name">hr.job.view.form.inherit</field>
        <field name="model">hr.job</field>
        <field name="inherit_id" ref="hr.view_hr_job_form"/>
        <field name="arch" type="xml">
            <field name="contract_type_id" position="after">
                <field 
                    name="skill_ids"
                    widget="many2many_tags"
                    options="{'color_field': 'color'}"
                    context="{'search_default_group_skill_type_id': 1}"
                    />
            </field>
        </field>
    </record>

    <record id="action_applicant_search_applicant" model="ir.actions.server">
        <field name="name">Search Matching Applicants</field>
        <field name="model_id" ref="hr_recruitment.model_hr_job"/>
        <field name="binding_model_id" ref="hr_recruitment.model_hr_job"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">action = records.action_search_matching_candidates()</field>
    </record>
</odoo>

```

