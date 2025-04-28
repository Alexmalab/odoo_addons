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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Skills Certification',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add certification to resumé of your employees',
    'description':
        """
Certification and Skills for HR
===============================

This module adds certification to resumé for employees.
        """,
    'depends': ['hr_skills', 'survey'],
    'data': [
        'views/hr_templates.xml',
        'data/hr_resume_data.xml',
    ],
    'qweb': [
        'static/src/xml/resume_templates.xml',
    ],
    'auto_install': True,
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

## File: models\survey_user.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools import html2plaintext


class SurveyUserInput(models.Model):
    _inherit = 'survey.user_input'

    def _mark_done(self):
        """ Will add certification to employee's resumé if
        - The survey is a certification
        - The user is linked to an employee
        - The user succeeded the test """

        super(SurveyUserInput, self)._mark_done()

        certificate_user_inputs = self.filtered(lambda user_input: user_input.survey_id.certificate and user_input.quizz_passed)
        partner_has_completed = {user_input.partner_id.id: user_input.survey_id for user_input in certificate_user_inputs}
        employees = self.env['hr.employee'].sudo().search([('user_id.partner_id', 'in', certificate_user_inputs.mapped('partner_id').ids)])
        for employee in employees:
            line_type = self.env.ref('hr_skills_survey.resume_type_certification', raise_if_not_found=False)
            survey = partner_has_completed.get(employee.user_id.partner_id.id)
            self.env['hr.resume.line'].create({
                'employee_id': employee.id,
                'name': survey.title,
                'date_start': fields.Date.today(),
                'date_end': fields.Date.today(),
                'description': html2plaintext(survey.description),
                'line_type_id': line_type and line_type.id,
                'display_type': 'certification',
                'survey_id': survey.id
            })


class ResumeLine(models.Model):
    _inherit = 'hr.resume.line'

    display_type = fields.Selection(selection_add=[('certification', 'Certification')])
    survey_id = fields.Many2one('survey.survey', string='Certification', readonly=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import survey_user

```

## File: static\src\xml\resume_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-extend="hr_resume_data_row">
    <t t-jquery="tr.o_data_row" t-operation="append">
        <t t-if="data.display_type === 'certification'">
            <td class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-esc="data.date_start"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3><t t-esc="data.name"/></h3>
                        <t t-if="data.description" t-esc="data.description"/>
                    </div>
                    <div class="o_resume_line_icon icon_trophy col-lg-1">
                        <i class="fa fa-trophy fa-2x"></i>
                    </div>
                </div>
            </td>
        </t>
    </t>
</t>

</templates>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="hr_skills_survey_assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr_skills_survey/static/src/scss/hr_skills.scss"/>
        </xpath>
    </template>

    <record id="resume_survey_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="inherit_id" ref="hr_skills.resume_line_view_form"/>
        <field eval="20" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='line_type_id']" position="after">
                <field name="survey_id" attrs="{'invisible': [('display_type', '!=', 'certification')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

