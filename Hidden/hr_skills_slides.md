# Odoo Module: hr_skills_slides

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
    'name': 'Skills e-learning',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add completed courses to resumé of your employees',
    'description':
        """
E-learning and Skills for HR
============================

This module add completed courses to resumé for employees.
        """,
    'depends': ['hr_skills', 'website_slides'],
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
        <record id="resume_type_training" model="hr.resume.line.type">
            <field name="name">Internal Training</field>
            <field name="sequence">26</field>
        </record>
    </data>

</odoo>

```

## File: models\hr_resume_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResumeLine(models.Model):
    _inherit = 'hr.resume.line'

    display_type = fields.Selection(selection_add=[('course', 'Course')])
    channel_id = fields.Many2one('slide.channel', string="Course", readonly=True)

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SlideChannelPartner(models.Model):
    _inherit = 'slide.channel.partner'

    def _recompute_completion(self):
        res = super(SlideChannelPartner, self)._recompute_completion()
        partner_has_completed = {channel_partner.partner_id.id: channel_partner.channel_id for channel_partner in self}
        employees = self.env['hr.employee'].sudo().search([('user_id.partner_id', 'in', list(partner_has_completed.keys()))])
        for employee in employees:
            line_type = self.env.ref('hr_skills_slides.resume_type_training', raise_if_not_found=False)
            channel = partner_has_completed[employee.user_id.partner_id.id]
            self.env['hr.resume.line'].create({
                'employee_id': employee.id,
                'name': channel.name,
                'date_start': fields.Date.today(),
                'date_end': fields.Date.today(),
                'description': channel.description,
                'line_type_id': line_type and line_type.id,
                'display_type': 'course',
                'channel_id': channel.id
            })
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_resume_line
from . import slide_channel

```

## File: static\src\xml\resume_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-extend="hr_resume_data_row">
    <t t-jquery="tr.o_data_row" t-operation="append">
        <t t-if="data.display_type === 'course'">
            <td class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-esc="data.date_start"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3><t t-esc="data.name"/></h3>
                        <t t-if="data.description" t-esc="data.description"/>
                    </div>
                    <div class="o_resume_line_icon icon_circle col-lg-1">
                        <i class="fa fa-check-circle fa-2x"></i>
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
    <template id="assets_backend" name="hr_skills_assets_backend" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/hr_skills_slides/static/src/scss/hr_skills.scss"/>
        </xpath>
    </template>

    <record id="resume_slides_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="inherit_id" ref="hr_skills.resume_line_view_form"/>
        <field eval="20" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='line_type_id']" position="after">
                <field name="channel_id" attrs="{'invisible': [('display_type', '!=', 'course')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

