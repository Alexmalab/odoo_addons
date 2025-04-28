# Odoo Module: website_event_questions

Category: Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Questions on Events',
    'description': 'Questions on Events',
    'category': 'Marketing',
    'version': '1.0',
    'depends': ['website_event'],
    'data': [
        'views/event_views.xml',
        'views/event_templates.xml',
        'report/report_event_question_view.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEvent(WebsiteEventController):

    def _process_registration_details(self, details):
        ''' Process data posted from the attendee details form. '''
        registrations = super(WebsiteEvent, self)._process_registration_details(details)
        for registration in registrations:
            answer_ids = []
            for key, value in registration.items():
                if key.startswith('answer_ids-'):
                    answer_ids.append([4, int(value)])
            registration['answer_ids'] = answer_ids
        return registrations

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class EventType(models.Model):
    _inherit = 'event.type'

    use_questions = fields.Boolean('Questions to Attendees')
    question_ids = fields.One2many(
        'event.question', 'event_type_id',
        string='Questions', copy=True)


class EventEvent(models.Model):
    """ Override Event model to add optional questions when buying tickets. """
    _inherit = 'event.event'

    question_ids = fields.One2many('event.question', 'event_id', 'Questions', copy=True)
    general_question_ids = fields.One2many('event.question', 'event_id', 'General Questions',
                                           domain=[('is_individual', '=', False)])
    specific_question_ids = fields.One2many('event.question', 'event_id', 'Specific Questions',
                                            domain=[('is_individual', '=', True)])

    @api.onchange('event_type_id')
    def _onchange_type(self):
        super(EventEvent, self)._onchange_type()
        if self.event_type_id.use_questions and self.event_type_id.question_ids:
            self.question_ids = [(5, 0, 0)] + [
                (0, 0, {
                    'title': question.title,
                    'sequence': question.sequence,
                    'is_individual': question.is_individual,
                })
                for question in self.event_type_id.question_ids
            ]


class EventRegistrationAnswer(models.Model):
    ''' This m2m table has to be explicitly instanciated as we need unique ids
    in the reporting view event.question.report.

    This model is purely technical. '''

    _name = 'event.registration.answer'
    _table = 'event_registration_answer'
    _description = 'Event Registration Answer'

    event_answer_id = fields.Many2one('event.answer', required=True, ondelete='cascade')
    event_registration_id = fields.Many2one('event.registration', required=True, ondelete='cascade')


class EventRegistration(models.Model):
    """ Store answers on attendees. """
    _inherit = 'event.registration'

    answer_ids = fields.Many2many('event.answer', 'event_registration_answer', string='Answers')


class EventQuestion(models.Model):
    _name = 'event.question'
    _rec_name = 'title'
    _order = 'sequence,id'
    _description = 'Event Question'

    title = fields.Char(required=True, translate=True)
    event_type_id = fields.Many2one('event.type', 'Event Type', ondelete='cascade')
    event_id = fields.Many2one('event.event', 'Event', ondelete='cascade')
    answer_ids = fields.One2many('event.answer', 'question_id', "Answers", required=True, copy=True)
    sequence = fields.Integer(default=10)
    is_individual = fields.Boolean('Ask each attendee',
                                   help="If True, this question will be asked for every attendee of a reservation. If "
                                        "not it will be asked only once and its value propagated to every attendees.")

    @api.constrains('event_type_id', 'event_id')
    def _constrains_event(self):
        if any(question.event_type_id and question.event_id for question in self):
            raise UserError(_('Question cannot belong to both the event category and itself.'))

    @api.model
    def create(self, vals):
        event_id = vals.get('event_id', False)
        if event_id:
            event = self.env['event.event'].browse([event_id])
            if event.event_type_id.use_questions and event.event_type_id.question_ids and not vals.get('answer_ids'):
                vals['answer_ids'] = [(0, 0, {
                    'name': answer.name,
                    'sequence': answer.sequence,
                }) for answer in event.event_type_id.question_ids.filtered(lambda question: question.title == vals.get('title')).mapped('answer_ids')]
        return super(EventQuestion, self).create(vals)


class EventAnswer(models.Model):
    _name = 'event.answer'
    _order = 'sequence,id'
    _description = 'Event Answer'

    name = fields.Char('Answer', required=True, translate=True)
    question_id = fields.Many2one('event.question', required=True, ondelete='cascade')
    sequence = fields.Integer(default=10)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import event

```

## File: report\report_event_question_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Graph view for question analysis-->
        <record model="ir.ui.view" id="view_event_question_report_graph">
            <field name="name">event.question.report.graph</field>
            <field name="model">event.question.report</field>
            <field name="arch" type="xml">
                <graph string="Questions Analysis">
                    <field name="answer_id"/>
                    <field name="question_id"/>
                </graph>
            </field>
        </record>

        <!-- Pivot view for question analysis -->
        <record model="ir.ui.view" id="view_event_question_report_pivot">
            <field name="name">event.question.report.pivot</field>
            <field name="model">event.question.report</field>
            <field name="arch" type="xml">
                <pivot string="Questions Analysis" disable_linking="True">
                    <field name="event_id" type="row"/>
                    <field name="attendee_id" type="row"/>
                    <field name="question_id" type="col"/>
                    <field name="answer_id" type="col"/>
                </pivot>
            </field>
        </record>

        <!-- Seach view for question analysis-->
        <record model="ir.ui.view" id="view_event_question_report_search">
            <field name="name">event.question.report.search</field>
            <field name="model">event.question.report</field>
            <field name="arch" type="xml">
                <search string="Questions Analysis">
                    <field name="event_id" string="Event"/>
                    <field name="attendee_id" string="Registration"/>
                    <group expand="1" string="Group By">
                        <filter string="Event" name="group_by_event" context="{'group_by': 'event_id'}" />
                        <filter string="Question" name="group_by_question" context="{'group_by': 'question_id'}"/>
                        <filter string="Answer" name="group_by_answer" context="{'group_by': 'answer_id'}" />
                        <filter name="exclude_cancelled"
                                string="Exclude cancelled registrations"
                                domain="[('attendee_id.state', '!=', 'cancel')]" />
                    </group>
                </search>
            </field>
        </record>

        <!-- Action for reporting -->
       <record model="ir.actions.act_window" id="action_event_question_report">
           <field name="name">Questions Analysis</field>
           <field name="res_model">event.question.report</field>
           <field name="view_mode">graph,pivot</field>
       </record>

       <!-- Menu -->
       <menuitem name="Questions" id="menu_report_event_questions"
            parent="event.menu_reporting_events" action="action_event_question_report" sequence="4"/>

    </data>
</odoo>

```

## File: report\report_event_registrations_questions.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class ReportEventRegistrationQuestions(models.Model):
    _name = "event.question.report"
    _auto = False
    _description = 'Event Question Report'

    attendee_id = fields.Many2one(comodel_name='event.registration', string='Registration')
    question_id = fields.Many2one(comodel_name='event.question', string='Question')
    answer_id = fields.Many2one(comodel_name='event.answer', string='Answer')
    event_id = fields.Many2one(comodel_name='event.event', string='Event')

    def init(self):
        """ Event Question main report """
        tools.drop_view_if_exists(self._cr, 'event_question_report')
        self._cr.execute(""" CREATE VIEW event_question_report AS (
            SELECT
                att_answer.id as id,
                att_answer.event_registration_id as attendee_id,
                answer.question_id as question_id,
                answer.id as answer_id,
                question.event_id as event_id
            FROM
                event_registration_answer as att_answer
            LEFT JOIN
                event_answer as answer ON answer.id = att_answer.event_answer_id
            LEFT JOIN
                event_question as question ON question.id = answer.question_id
            GROUP BY
                attendee_id,
                event_id,
                question_id,
                answer_id,
                att_answer.id
        )""")

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-

from . import report_event_registrations_questions

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
event_question_all,event.question.all,model_event_question,,1,0,0,0
event_question_event_user,event.question.event.user,model_event_question,event.group_event_user,1,1,1,1
event_answer_all,event.answer.all,model_event_answer,,1,0,0,0
event_answer_event_user,event.answer.event.user,model_event_answer,event.group_event_user,1,1,1,1
event_question_report_all,event.question.report.all,model_event_question_report,event.group_event_user,1,1,1,1
event_registration_answer_all,access_event_registration_answer,model_event_registration_answer,,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-.146-4-4.074v-21.23l18-17.474 9-7.13 10-1.018 8 3.056L46 14h9v11.204l-4 3.055 2 7.13-1 9.167L39.478 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M24.762 52.559V50c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 32l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 55.286c3.28 0 6.343-.93 8.94-2.538L41 47c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 38.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zm7.052 3.772v2.19h2.19v-2.19h-2.19zm-2.116-5.016h1.963c0-.267.027-.517.08-.749.054-.232.136-.435.247-.608a1.29 1.29 0 0 1 .428-.415c.173-.104.38-.156.62-.156.357 0 .635.109.835.326.2.218.301.554.301 1.009.009.267-.033.49-.127.667a1.959 1.959 0 0 1-.367.49 8.245 8.245 0 0 1-.494.444 3.532 3.532 0 0 0-.508.527c-.16.202-.3.447-.42.734s-.194.642-.221 1.068v.667h1.803v-.564a1.61 1.61 0 0 1 .26-.741 2.82 2.82 0 0 1 .475-.527c.178-.153.367-.306.567-.46.2-.152.383-.338.548-.555.165-.218.303-.48.414-.786.111-.307.167-.697.167-1.172a2.92 2.92 0 0 0-.167-.927 2.637 2.637 0 0 0-.554-.926 3.058 3.058 0 0 0-1.022-.72c-.423-.192-.95-.289-1.583-.289-.49 0-.933.092-1.329.275a2.956 2.956 0 0 0-1.015.763c-.28.326-.499.712-.654 1.157a4.653 4.653 0 0 0-.247 1.468z" opacity=".3"/><path fill="#FFF" d="M24.762 50.559V48c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 30l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 53.286c3.28 0 6.343-.93 8.94-2.538L41 45c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 36.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zm7.052 3.772v2.19h2.19v-2.19h-2.19zm-2.116-5.016h1.963c0-.267.027-.517.08-.749.054-.232.136-.435.247-.608a1.29 1.29 0 0 1 .428-.415c.173-.104.38-.156.62-.156.357 0 .635.109.835.326.2.218.301.554.301 1.009.009.267-.033.49-.127.667a1.959 1.959 0 0 1-.367.49 8.245 8.245 0 0 1-.494.444 3.532 3.532 0 0 0-.508.527c-.16.202-.3.447-.42.734s-.194.642-.221 1.068v.667h1.803v-.564a1.61 1.61 0 0 1 .26-.741 2.82 2.82 0 0 1 .475-.527c.178-.153.367-.306.567-.46.2-.152.383-.338.548-.555.165-.218.303-.48.414-.786.111-.307.167-.697.167-1.172a2.92 2.92 0 0 0-.167-.927 2.637 2.637 0 0 0-.554-.926 3.058 3.058 0 0 0-1.022-.72c-.423-.192-.95-.289-1.583-.289-.49 0-.933.092-1.329.275a2.956 2.956 0 0 0-1.015.763c-.28.326-.499.712-.654 1.157a4.653 4.653 0 0 0-.247 1.468z"/></g></g></svg>
```

## File: views\event_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="registration_attendee_details_questions" inherit_id="website_event.registration_attendee_details" name="Registration Attendee Details with questions">
    <!-- Attendee specific questions -->
    <xpath expr="//t[@name='attendee_loop']//*[hasclass('modal-body')]" position="inside">
        <t t-if="event.specific_question_ids">
            <div t-foreach="event.specific_question_ids" t-as="question" class="row">
                <div class="col-lg-6 mt-2">
                    <label t-esc="question.title"/>
                    <select t-attf-name="#{counter}-answer_ids-#{question.id}" class="custom-select" required="true">
                        <t t-foreach="question.answer_ids" t-as="answer">
                            <option t-esc="answer.name" t-att-value="answer.id"/>
                        </t>
                    </select>
                </div>
            </div>
        </t>
    </xpath>
    <!-- Generic questions -->
    <xpath expr="//*[hasclass('modal-footer')]" position="before">
        <t t-if="event.general_question_ids">
            <div class="modal-body bg-light border-bottom">
                <div t-foreach="event.general_question_ids" t-as="question" class="row">
                    <div class="col-lg-6">
                        <label class="h5" t-esc="question.title"/>
                        <select t-attf-name="0-answer_ids-#{question.id}" class="custom-select" required="true">
                            <t t-foreach="question.answer_ids" t-as="answer">
                                <option t-esc="answer.name" t-att-value="answer.id"/>
                            </t>
                        </select>
                    </div>
                </div>
            </div>
        </t>
    </xpath>
</template>

</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record model="ir.ui.view" id="view_event_question_form">
    <field name="name">event.question.form</field>
    <field name="model">event.question</field>
    <field name="arch" type="xml">
        <form string="Question">
            <h1><field name="title" /></h1>
            <field name="is_individual"/>
            <label for="is_individual"/>
            <field name="answer_ids">
                <tree editable="bottom">
                    <field name="sequence" widget="handle" />
                    <field name="name"/>
                </tree>
            </field>
        </form>
    </field>
</record>

<record model="ir.ui.view" id="view_event_answer_simplified_form">
    <field name="name">event.answer.simplified.form</field>
    <field name="model">event.answer</field>
    <field name="arch" type="xml">
        <form string="Question">
            <group>
                <field name="name"/>
            </group>
        </form>
    </field>
</record>

<record id="event_type_view_form_inherit_question" model="ir.ui.view">
    <field name="name">event.type.view.form.inherit.question</field>
    <field name="model">event.type</field>
    <field name="inherit_id" ref="website_event.event_type_view_form_inherit_website"/>
    <field name="arch" type="xml">
        <div name="event_type_attendees_auto_confirm" position="after">
            <div class="col-12 col-lg-12 o_setting_box">
                <div class="o_setting_left_pane">
                    <field name="use_questions"/>
                </div>
                <div class="o_setting_right_pane">
                    <label for="use_questions"/>
                    <div class="row mt16" attrs="{'invisible': [('use_questions', '=', False)]}">
                        <div class="col-lg-9">
                            <field name="question_ids"/>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </field>
</record>

<record model="ir.ui.view" id="view_event_form_inherit_question">
    <field name="name">event.event.view.form.inherit.question</field>
    <field name="model">event.event</field>
    <field name="inherit_id" ref="website_event.event_event_view_form_inherit_website"/>
    <field name="arch" type="xml">
        <data>
            <xpath expr="//notebook" position="inside">
                <page string="Questions">
                    <field name="question_ids" nolabel="1">
                        <tree>
                            <field name="sequence" widget="handle" />
                            <field name="title"/>
                            <field name="is_individual"/>
                            <field name="answer_ids"/>
                        </tree>
                    </field>
                </page>
            </xpath>
       </data>
    </field>
</record>

<record model="ir.ui.view" id="view_event_registration_form_inherit_question">
    <field name="name">event.registration.form.inherit.question</field>
    <field name="model">event.registration</field>
    <field name="inherit_id" ref="event.view_event_registration_form" />
    <field name="arch" type="xml">
        <group name="event" position="inside">
            <field name="answer_ids" widget="many2many_tags" readonly="1"/>
        </group>
    </field>
</record>

<record model="ir.ui.view" id="view_registration_search_inherit_question">
    <field name="name">event.registration.search.inherit.question</field>
    <field name="model">event.registration</field>
    <field name="inherit_id" ref="event.view_registration_search"/>
    <field name="arch" type="xml">
        <search position="inside">
            <field name="answer_ids" string="Answers"/>
        </search>
    </field>
</record>

</odoo>

```

