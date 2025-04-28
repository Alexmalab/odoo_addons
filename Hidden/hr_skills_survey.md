# Odoo Module: hr_skills_survey

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Skills Certification',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add certification to resume of your employees',
    'description':
        """
Certification and Skills for HR
===============================

This module adds certification to resume for employees.
        """,
    'depends': ['hr_skills', 'survey'],
    'data': [
        'views/hr_templates.xml',
        'data/hr_resume_data.xml',
        'views/hr_employee_certification_views.xml',
        'views/survey_survey_views.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'hr_skills_survey/static/src/fields/**/*',
            'hr_skills_survey/static/src/xml/**/*',
        ],
    },
    'demo': [
        'data/hr_resume_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\hr_resume_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="resume_type_certification" model="hr.resume.line.type">
            <field name="name">Internal Certification</field>
            <field name="sequence">25</field>
        </record>
    </data>

</odoo>

```

## File: data\hr_resume_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Valid certification -->
        <record id="resume_line_valid" model="hr.resume.line">
            <field name="name">AWS Cloud</field>
            <field name="display_type">certification</field>
            <field name="date_start" eval="DateTime.today() + relativedelta(years=-1, months=-2, days=-13)"/>
            <field name="date_end" eval="DateTime.today() + relativedelta(years=1)"/>
            <field name="employee_id" ref="hr.employee_admin"/>
        </record>

        <!-- Expiring soon certification -->
        <record id="resume_line_expiring" model="hr.resume.line">
            <field name="name">MongoDB Developer</field>
            <field name="display_type">certification</field>
            <field name="date_start" eval="DateTime.today() + relativedelta(years=-1)"/>
            <field name="date_end" eval="DateTime.today() + relativedelta(months=2)"/>
            <field name="employee_id" ref="hr.employee_admin"/>
        </record>

        <!-- Expired -->
        <record id="resume_line_aws" model="hr.resume.line">
            <field name="name">Oracle DB</field>
            <field name="display_type">certification</field>
            <field name="date_start" eval="DateTime.today() + relativedelta(years=-1, months=-6, days=-7)"/>
            <field name="date_end" eval="DateTime.today() + relativedelta(days=-3)"/>
            <field name="employee_id" ref="hr.employee_admin"/>
        </record>
    </data>
</odoo>

```

## File: models\hr_resume_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models


class ResumeLine(models.Model):
    _inherit = 'hr.resume.line'

    display_type = fields.Selection(selection_add=[('certification', 'Certification')])
    department_id = fields.Many2one(related="employee_id.department_id", store=True)
    survey_id = fields.Many2one('survey.survey', string='Certification', readonly=True)
    expiration_status = fields.Selection([
        ('expired', 'Expired'),
        ('expiring', 'Expiring'),
        ('valid', 'Valid')], compute='_compute_expiration_status', store=True)

    @api.depends('date_end')
    def _compute_expiration_status(self):
        self.expiration_status = 'valid'
        for line in self:
            if line.date_end:
                if line.date_end <= fields.Date.today():
                    line.expiration_status = 'expired'
                elif line.date_end + relativedelta(months=-3) <= fields.Date.today():
                    line.expiration_status = 'expiring'

```

## File: models\survey_survey.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class SurveySurvey(models.Model):
    _inherit = 'survey.survey'

    certification_validity_months = fields.Integer(
        'Validity', required=False,
        help='Specify the number of months the certification is valid after being awarded. '
             'Enter 0 for certifications that never expire.')

```

## File: models\survey_user.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import fields, models
from odoo.osv import expression
from odoo.tools import html2plaintext


class SurveyUserInput(models.Model):
    _inherit = 'survey.user_input'

    def _mark_done(self):
        """ Will add certification to employee's resume if
        - The survey is a certification
        - The user is linked to an employee
        - The user succeeded the test """

        super(SurveyUserInput, self)._mark_done()

        certification_user_inputs = self.filtered(lambda user_input: user_input.survey_id.certification and user_input.scoring_success)
        user_inputs_by_partner = certification_user_inputs.grouped('partner_id')
        employees = self.env['hr.employee'].search(
            [('user_id.partner_id', 'in', certification_user_inputs.partner_id.ids)])
        resume_lines = self.env['hr.resume.line'].search(
            expression.OR([
                expression.AND([
                    [('employee_id', '=', employee.id)],
                    [('survey_id', 'in', user_inputs_by_partner[employee.user_id.partner_id].survey_id.ids)]])
                for employee in employees
            ]))
        resume_survey_by_ids = resume_lines.grouped(
            lambda resume_line: (resume_line.employee_id, resume_line.survey_id))
        line_type = self.env.ref('hr_skills_survey.resume_type_certification', raise_if_not_found=False)

        lines_to_create = []
        today = fields.Date.today()
        for employee in employees:
            for user_input in user_inputs_by_partner[employee.user_id.partner_id]:
                survey = user_input.survey_id
                date_start = today
                validity_month = survey.certification_validity_months
                resume_line_vals = {
                    'employee_id': employee.id,
                    'name': survey.title,
                    'date_start': date_start,
                    'date_end': date_start + relativedelta(months=validity_month) if validity_month else False,
                    'description': html2plaintext(survey.description) if survey.description else '',
                    'line_type_id': line_type.id if line_type else False,
                    'display_type': 'certification',
                    'survey_id': survey.id,
                }
                if existing_resume_survey := resume_survey_by_ids.get((employee, survey)):
                    existing_resume_survey.write(resume_line_vals)
                else:
                    lines_to_create.append(resume_line_vals)
        self.env['hr.resume.line'].create(lines_to_create)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import survey_survey
from . import survey_user
from . import hr_resume_line

```

## File: static\src\fields\resume_one2many\resume_one2many.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-inherit="hr_skills.ResumeListRenderer.RecordRow" t-inherit-mode="extension">
    <xpath expr="//t[@id='row']" position='after'>
        <t t-if="data.display_type === 'certification'">
            <td t-on-click="(ev) => this.onCellClicked(record, null, ev)"
                class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-out="formatDate(data.date_end)"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3>
                            <i class="fa fa-trophy text-warning me-1"></i>
                            <t t-esc="data.name"/>
                        </h3>
                        <t t-if="data.description" t-out="data.description"/>
                    </div>
                </div>
            </td>
        </t>
    </xpath>
</t>

</templates>

```

## File: static\src\xml\resume_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-inherit="hr_resume_data_row" t-inherit-mode="extension">
    <xpath expr="//tr[hasclass('o_data_row')]" position="inside">
        <t t-if="data.display_type === 'certification'">
            <td class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-esc="data.date_end"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3>
                            <i class="fa fa-trophy text-warning me-1"></i>
                            <t t-esc="data.name"/>
                        </h3>
                        <t t-if="data.description" t-esc="data.description"/>
                    </div>
                </div>
            </td>
        </t>
    </xpath>
</t>

</templates>

```

## File: views\hr_employee_certification_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_certification_report_view_list" model="ir.ui.view">
        <field name="model">hr.resume.line</field>
        <field name="arch" type="xml">
            <list expand="1"
                decoration-danger="expiration_status == 'expired'"
                decoration-warning="expiration_status == 'expiring'"
                default_group_by="employee_id">
                <field name="employee_id" widget="many2one_avatar_user"/>
                <field name="name"/>
                <field name="date_start" string="Validity Start"/>
                <field name="date_end" string="Validity End"/>
                <field name="survey_id" optional="show"/>
                <field name="expiration_status" column_invisible="1"/>
            </list>
        </field>
    </record>

    <record id="hr_resume_line_view_search" model="ir.ui.view">
        <field name="model">hr.resume.line</field>
        <field name="arch" type="xml">
            <search>
                <field name="employee_id"/>
                <field name="name"/>
                <field name="survey_id"/>
                <separator/>
                <filter string="Expiring Soon" name="certification_expiring" domain="[('expiration_status', '=', 'expiring')]"/>
                <filter string="Expired" name="certification_expired" domain="[('expiration_status', '=', 'expired')]"/>
                <separator/>
                <filter string="Valid Until" name="date_end" date="date_end"/>
                <separator/>
                <filter string="Employee" name="employee" context="{'group_by': 'employee_id'}"/>
                <filter string="Department" name="department" context="{'group_by': 'department_id'}"/>
                <separator/>
                <filter string="Certification" name="certification" context="{'group_by': 'survey_id'}"/>
                <filter string="Expiration date" name="date_end" context="{'group_by': 'date_end'}"/>
            </search>
        </field>
    </record>

    <record id="hr_employee_certification_report_action" model="ir.actions.act_window">
        <field name="name">Employee Certifications</field>
        <field name="res_model">hr.resume.line</field>
        <field name="search_view_id" ref="hr_resume_line_view_search"/>
        <field name="view_id" ref="hr_employee_certification_report_view_list"/>
        <field name="view_mode">list,form</field>
        <field name="domain">[('display_type','=','certification')]</field>
        <field name="target">current</field>
    </record>

    <menuitem
        id="hr_employee_certication_report_menu"
        name="Certifications"
        action="hr_employee_certification_report_action"
        parent="hr.hr_menu_hr_reports"
        groups="hr.group_hr_user"
        sequence="20"/>
</odoo>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="resume_survey_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="inherit_id" ref="hr_skills.resume_line_view_form"/>
        <field eval="20" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='display_type']" position="after">
                <field name="survey_id"
                    domain="[('certification', '=', True)]"
                    readonly="0"
                    invisible="display_type != 'certification'"/>
            </xpath>
            <field name="employee_id" position="attributes">
                <attribute name="invisible">0</attribute>
            </field>
            <field name="survey_id" position="attributes">
                <attribute name="context">
                    {'default_certification': True}
                </attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\survey_survey_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <record id="survey_survey_view_form" model="ir.ui.view">
        <field name="name">survey.survey.view.form.inherit.hr.skills</field>
        <field name="model">survey.survey</field>
        <field name="inherit_id" ref="survey.survey_survey_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='certification']" position="after">
                <field name="certification_validity_months" invisible="not certification"/>
            </xpath>
        </field>
    </record>
</data>
</odoo>

```

