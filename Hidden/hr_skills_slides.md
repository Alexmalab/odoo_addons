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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Skills e-learning',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add completed courses to resume of your employees',
    'description':
        """
E-learning and Skills for HR
============================

This module add completed courses to resume for employees.
        """,
    'depends': ['hr_skills', 'website_slides'],
    'data': [
        'views/hr_employee_views.xml',
        'views/hr_templates.xml',
        'data/hr_resume_data.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'hr_skills_slides/static/src/scss/**/*',
            'hr_skills_slides/static/src/fields/**/*',
            'hr_skills_slides/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_resume_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="resume_type_training" model="hr.resume.line.type">
            <field name="name">Completed Internal Training</field>
            <field name="sequence">26</field>
        </record>
    </data>

</odoo>

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class Employee(models.Model):
    _inherit = 'hr.employee'

    subscribed_courses = fields.Many2many('slide.channel', related='user_partner_id.slide_channel_ids')
    has_subscribed_courses = fields.Boolean(compute='_compute_courses_completion_text')
    courses_completion_text = fields.Char(compute="_compute_courses_completion_text")

    @api.depends_context('lang')
    @api.depends('subscribed_courses', 'user_partner_id.slide_channel_completed_ids')
    def _compute_courses_completion_text(self):
        for employee in self:
            if not employee.user_partner_id:
                employee.courses_completion_text = False
                employee.has_subscribed_courses = False
                continue
            total_completed_courses = len(employee.user_partner_id.slide_channel_completed_ids)
            total = len(employee.subscribed_courses)
            employee.courses_completion_text = _("%(completed)s / %(total)s",
                completed=total_completed_courses,
                total=total)
            employee.has_subscribed_courses = total > 0

    def action_open_courses(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': '/profile/user/%s' % self.user_id.id,
        }

```

## File: models\hr_resume_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResumeLine(models.Model):
    _inherit = 'hr.resume.line'

    display_type = fields.Selection(selection_add=[('course', 'Course')])
    channel_id = fields.Many2one('slide.channel', string="Course", readonly=True, index='btree_not_null')
    course_url = fields.Char(compute="_compute_course_url", default=False)

    @api.depends('channel_id')
    def _compute_course_url(self):
        for line in self:
            if line.display_type == 'course':
                line.course_url = line.channel_id.website_url
            else:
                line.course_url = False

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import fields, models, _
from odoo.tools import html2plaintext


class SlideChannelPartner(models.Model):
    _inherit = 'slide.channel.partner'

    def _recompute_completion(self):
        res = super(SlideChannelPartner, self)._recompute_completion()
        completed_membership = self.filtered(lambda m: m.member_status == 'completed')
        if not completed_membership:
            return res
        partner_has_completed = {
            membership.partner_id.id: membership.channel_id
            for membership in completed_membership
        }
        employees = self.env['hr.employee'].sudo().search(
            [('user_id.partner_id', 'in', completed_membership.partner_id.ids)])

        if employees:
            HrResumeLine = self.env['hr.resume.line'].sudo()
            line_type = self.env.ref('hr_skills_slides.resume_type_training', raise_if_not_found=False)
            line_type_id = line_type and line_type.id

            lines_for_channel_by_employee = dict(HrResumeLine._read_group([
                ('employee_id', 'in', employees.ids),
                ('channel_id', 'in', completed_membership.channel_id.ids),
                ('line_type_id', '=', line_type_id),
                ('display_type', '=', 'course')
            ], ['employee_id'], ['channel_id:array_agg']))

            lines_to_create = []
            for employee in employees:
                channel = partner_has_completed[employee.user_id.partner_id.id]

                if channel.id not in lines_for_channel_by_employee.get(employee, []):
                    lines_to_create.append({
                        'employee_id': employee.id,
                        'name': channel.name,
                        'date_start': fields.Date.today(),
                        'date_end': fields.Date.today(),
                        'description': html2plaintext(channel.description),
                        'line_type_id': line_type_id,
                        'display_type': 'course',
                        'channel_id': channel.id
                    })
            if lines_to_create:
                HrResumeLine.create(lines_to_create)
        return res

    def _send_completed_mail(self):
        super()._send_completed_mail()
        for scp in self:
            if self.env.user.employee_ids:
                msg = _('The employee has completed the course %s',
                    Markup('<a href="%(link)s">%(course)s</a>') % {
                        'link': scp.channel_id.website_url,
                        'course': scp.channel_id.name,
                })
                self.env.user.employee_id.message_post(body=msg)

class Channel(models.Model):
    _inherit = 'slide.channel'

    def _action_add_members(self, target_partners, member_status='joined', raise_on_access=False):
        res = super()._action_add_members(target_partners, member_status=member_status, raise_on_access=raise_on_access)
        if member_status == 'joined':
            for channel in self:
                channel._message_employee_chatter(
                    _('The employee subscribed to the course %s',
                        Markup('<a href="%(link)s">%(course)s</a>') % {
                            'link': channel.website_url,
                            'course': channel.name
                    }),
                    target_partners
                )
        return res

    def _remove_membership(self, partner_ids):
        res = super()._remove_membership(partner_ids)

        partners = self.env['res.partner'].browse(partner_ids)

        for channel in self:
            channel._message_employee_chatter(
                _('The employee left the course %s',
                    Markup('<a href="%(link)s">%(course)s</a>') % {
                        'link': channel.website_url,
                        'course': channel.name,
                }),
                partners)
        return res

    def _message_employee_chatter(self, msg, partners):
        for partner in partners:
            employee = partner.user_ids.sudo().filtered(
                lambda u: u.employee_id and (not partner.company_id or u.employee_id.company_id == partner.company_id)
            ).employee_id

            if employee:
                employee.sudo().message_post(body=msg)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import hr_resume_line
from . import slide_channel

```

## File: static\src\fields\resume_one2many\resume_one2many.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

<t t-inherit="hr_skills.ResumeListRenderer.RecordRow" t-inherit-mode="extension">
    <xpath expr="//t[@id='row']" position='after'>
        <t t-if="data.display_type === 'course'">
            <td t-on-click="(ev) => this.onCellClicked(record, null, ev)"
                class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-out="formatDate(data.date_end)"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3>
                            <t t-esc="data.name"/>
                            <a t-attf-href="#{data.course_url}" t-if="data.course_url"
                                style="font-size: 1rem;"
                                class="ms-2 fa fa-external-link btn-secondary"/>
                        </h3>
                        <t t-if="data.description" class="o_resume_line_desc" t-out="data.description"/>
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

<t t-inherit="hr_skills.hr_resume_data_row" t-inherit-mode="extension">
    <xpath expr="//tr[hasclass('o_data_row')]" position="inside">
        <t t-if="data.display_type === 'course'">
            <td class="o_data_cell container" colspan="2">
                <div class="o_resume_line row" t-att-data-id="id">
                    <div class="o_resume_line_dates col-lg-3">
                        <span><t t-esc="data.date_end"/></span>
                    </div>
                    <div class="o_resume_line_desc col-lg-8">
                        <h3>
                            <t t-esc="data.name"/>
                            <a t-attf-href="#{data.course_url}" t-if="data.course_url"
                                style="font-size: 1rem;"
                                class="ms-2 fa fa-external-link btn-secondary"/>
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

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.resume.slides</field>
        <field name="model">hr.employee</field>
        <field name="priority" eval="45" />
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='line_type_id']" position="after">
                <field name="course_url" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='company_country_code']" position="after">
                <field name="user_id" invisible="1"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="has_subscribed_courses" invisible="1" groups="website_slides.group_website_slides_officer"/>
                <button name="action_open_courses"
                    class="oe_stat_button"
                    groups="website_slides.group_website_slides_officer"
                    icon="fa-graduation-cap"
                    type="object"
                    invisible="not user_id or not has_subscribed_courses">
                    <field name="courses_completion_text" widget="statinfo" string="Courses"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\hr_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="resume_slides_line_view_form" model="ir.ui.view">
        <field name="name">hr.resume.line.form</field>
        <field name="model">hr.resume.line</field>
        <field name="inherit_id" ref="hr_skills.resume_line_view_form"/>
        <field eval="20" name="priority"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='display_type']" position="after">
                <field name="channel_id"
                    readonly="0"
                    invisible="display_type != 'course'"/>
                <field name="course_url" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

