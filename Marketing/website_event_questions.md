# Odoo Module: website_event_questions

Category: Marketing

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

{
    'name': 'Questions on Events',
    'description': 'Questions on Events',
    'category': 'Marketing',
    'version': '1.2',
    'depends': ['website_event'],
    'data': [
        'views/event_views.xml',
        'views/event_registration_answer_views.xml',
        'views/event_registration_views.xml',
        'views/event_question_views.xml',
        'views/event_templates.xml',
        'security/security.xml',
        'security/ir.model.access.csv',
    ],
    'demo': [
        'data/event_question_demo.xml',
        'data/event_demo.xml',
        'data/event_registration_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_tests': [
            'website_event_questions/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request

from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEvent(WebsiteEventController):

    def _process_attendees_form(self, event, form_details):
        """ Process data posted from the attendee details form.
        Extracts question answers:
        - For both questions asked 'once_per_order' and questions asked to every attendee
        - For questions of type 'simple_choice', extracting the suggested answer id
        - For questions of type 'text_box', extracting the text answer of the attendee. """
        registrations = super(WebsiteEvent, self)._process_attendees_form(event, form_details)

        for registration in registrations:
            registration['registration_answer_ids'] = []

        general_answer_ids = []
        for key, value in form_details.items():
            if 'question_answer' in key and value:
                dummy, registration_index, question_id = key.split('-')
                question_sudo = request.env['event.question'].browse(int(question_id))
                answer_values = None
                if question_sudo.question_type == 'simple_choice':
                    answer_values = {
                        'question_id': int(question_id),
                        'value_answer_id': int(value)
                    }
                elif question_sudo.question_type == 'text_box':
                    answer_values = {
                        'question_id': int(question_id),
                        'value_text_box': value
                    }

                if answer_values and not int(registration_index):
                    general_answer_ids.append((0, 0, answer_values))
                elif answer_values:
                    registrations[int(registration_index) - 1]['registration_answer_ids'].append((0, 0, answer_values))

        for registration in registrations:
            registration['registration_answer_ids'].extend(general_answer_ids)

        return registrations

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="event.event_type_data_conference" model="event.type">
        <field name="default_timezone">Europe/Brussels</field>
    </record>
</data></odoo>
```

## File: data\event_question_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!-- EVENT TYPE SPECIFIC -->
    <record id="event_type_data_conference_question_0" model="event.question">
        <field name="title">Participate in Social Event</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_type_id" ref="event.event_type_data_conference"/>
    </record>
    <record id="event_type_data_conference_question_0_answer_0" model="event.question.answer">
        <field name="name">Yes</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event_questions.event_type_data_conference_question_0"/>
    </record>
    <record id="event_type_data_conference_question_0_answer_1" model="event.question.answer">
        <field name="name">No</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event_questions.event_type_data_conference_question_0"/>
    </record>

    <!-- EVENT SPECIFIC -->
    <record id="event_0_question_0" model="event.question">
        <field name="title">Meal Type</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_0_answer_0" model="event.question.answer">
        <field name="name">Mixed</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event_questions.event_0_question_0"/>
    </record>
    <record id="event_0_question_0_answer_1" model="event.question.answer">
        <field name="name">Vegetarian</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event_questions.event_0_question_0"/>
    </record>
    <record id="event_0_question_0_answer_2" model="event.question.answer">
        <field name="name">Pastafarian</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event_questions.event_0_question_0"/>
    </record>
    <record id="event_0_question_1" model="event.question">
        <field name="title">Allergies</field>
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_2" model="event.question">
        <field name="title">How did you learn about this event?</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="True"/>
        <field name="event_id" ref="event.event_0"/>
    </record>
    <record id="event_0_question_2_answer_0" model="event.question.answer">
        <field name="name">Our website</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event_questions.event_0_question_2"/>
    </record>
    <record id="event_0_question_2_answer_1" model="event.question.answer">
        <field name="name">Commercials</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event_questions.event_0_question_2"/>
    </record>
    <record id="event_0_question_2_answer_2" model="event.question.answer">
        <field name="name">A friend</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event_questions.event_0_question_2"/>
    </record>

    <!-- Questions of: "OpenWood: Furniture Collection Online Reveal" -->
    <record id="event_7_question_0" model="event.question">
        <field name="title">Which field are you working in</field>
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_id" ref="event.event_7"/>
    </record>
    <record id="event_7_question_0_answer_0" model="event.question.answer">
        <field name="name">Consumers</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="website_event_questions.event_7_question_0"/>
    </record>
    <record id="event_7_question_0_answer_1" model="event.question.answer">
        <field name="name">Sales</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="website_event_questions.event_7_question_0"/>
    </record>
    <record id="event_7_question_0_answer_2" model="event.question.answer">
        <field name="name">Research</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="website_event_questions.event_7_question_0"/>
    </record>
    <record id="event_7_question_1" model="event.question">
        <field name="title">How did you hear about us ?</field>
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="True"/>
        <field name="event_id" ref="event.event_7"/>
    </record>

</data></odoo>

```

## File: data\event_registration_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <record id="event_registration_0_0_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_0_answer_0" />
        <field name="question_id" ref="website_event_questions.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_0_registration_answer_1" model="event.registration.answer">
        <field name="value_text_box">Fish
Nuts</field>
        <field name="question_id" ref="website_event_questions.event_0_question_1" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_0_registration_answer_2" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_2_answer_0" />
        <field name="question_id" ref="website_event_questions.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_0" />
    </record>
    <record id="event_registration_0_1_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_0_answer_1" />
        <field name="question_id" ref="website_event_questions.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_1" />
    </record>
    <record id="event_registration_0_1_registration_answer_1" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_2_answer_0" />
        <field name="question_id" ref="website_event_questions.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_1" />
    </record>
    <record id="event_registration_0_2_registration_answer_0" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_0_answer_2" />
        <field name="question_id" ref="website_event_questions.event_0_question_0" />
        <field name="registration_id" ref="event.event_registration_0_2" />
    </record>
    <record id="event_registration_0_2_registration_answer_1" model="event.registration.answer">
        <field name="value_answer_id" ref="website_event_questions.event_0_question_2_answer_2" />
        <field name="question_id" ref="website_event_questions.event_0_question_2" />
        <field name="registration_id" ref="event.event_registration_0_2" />
    </record>
</data>
</odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _inherit = 'event.type'

    question_ids = fields.One2many(
        'event.question', 'event_type_id',
        string='Questions', copy=True)


class EventEvent(models.Model):
    """ Override Event model to add optional questions when buying tickets. """
    _inherit = 'event.event'

    question_ids = fields.One2many(
        'event.question', 'event_id', 'Questions', copy=True,
        compute='_compute_question_ids', readonly=False, store=True)
    general_question_ids = fields.One2many('event.question', 'event_id', 'General Questions',
                                           domain=[('once_per_order', '=', True)])
    specific_question_ids = fields.One2many('event.question', 'event_id', 'Specific Questions',
                                            domain=[('once_per_order', '=', False)])

    @api.depends('event_type_id')
    def _compute_question_ids(self):
        """ Update event questions from its event type. Depends are set only on
        event_type_id itself to emulate an onchange. Changing event type content
        itself should not trigger this method.

        When synchronizing questions:

          * lines with no registered answers are removed;
          * type lines are added;
        """
        if self._origin.question_ids:
            # lines to keep: those with already given answers
            questions_tokeep_ids = self.env['event.registration.answer'].search(
                [('question_id', 'in', self._origin.question_ids.ids)]
            ).question_id.ids
        else:
            questions_tokeep_ids = []
        for event in self:
            if not event.event_type_id and not event.question_ids:
                event.question_ids = False
                continue

            if questions_tokeep_ids:
                questions_toremove = event._origin.question_ids.filtered(lambda question: question.id not in questions_tokeep_ids)
                command = [(3, question.id) for question in questions_toremove]
            else:
                command = [(5, 0)]
            event.question_ids = command

            # copy questions so changes in the event don't affect the event type
            for question in event.event_type_id.question_ids:
                event.question_ids += question.copy({'event_type_id': False})

```

## File: models\event_question.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class EventQuestion(models.Model):
    _name = 'event.question'
    _rec_name = 'title'
    _order = 'sequence,id'
    _description = 'Event Question'

    title = fields.Char(required=True, translate=True)
    question_type = fields.Selection([
        ('simple_choice', 'Selection'),
        ('text_box', 'Text Input')], default='simple_choice', string="Question Type", required=True)
    event_type_id = fields.Many2one('event.type', 'Event Type', ondelete='cascade')
    event_id = fields.Many2one('event.event', 'Event', ondelete='cascade')
    answer_ids = fields.One2many('event.question.answer', 'question_id', "Answers", copy=True)
    sequence = fields.Integer(default=10)
    once_per_order = fields.Boolean('Ask only once per order',
                                    help="If True, this question will be asked only once and its value will be propagated to every attendees."
                                         "If not it will be asked for every attendee of a reservation.")

    @api.constrains('event_type_id', 'event_id')
    def _constrains_event(self):
        if any(question.event_type_id and question.event_id for question in self):
            raise UserError(_("Question cannot be linked to both an Event and an Event Type."))

    def write(self, vals):
        """ We add a check to prevent changing the question_type of a question that already has answers.
        Indeed, it would mess up the event.registration.answer (answer type not matching the question type). """

        if 'question_type' in vals:
            questions_new_type = self.filtered(lambda question: question.question_type != vals['question_type'])
            if questions_new_type:
                answer_count = self.env['event.registration.answer'].search_count([('question_id', 'in', questions_new_type.ids)])
                if answer_count > 0:
                    raise UserError(_("You cannot change the question type of a question that already has answers!"))
        return super(EventQuestion, self).write(vals)

    def action_view_question_answers(self):
        """ Allow analyzing the attendees answers to event questions in a convenient way:
        - A graph view showing counts of each suggestions for simple_choice questions
          (Along with secondary pivot and tree views)
        - A tree view showing textual answers values for text_box questions. """
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("website_event_questions.action_event_registration_report")
        action['domain'] = [('question_id', '=', self.id)]
        if self.question_type == 'simple_choice':
            action['views'] = [(False, 'graph'), (False, 'pivot'), (False, 'tree')]
        elif self.question_type == 'text_box':
            action['views'] = [(False, 'tree')]
        return action

class EventQuestionAnswer(models.Model):
    """ Contains suggested answers to a 'simple_choice' event.question. """
    _name = 'event.question.answer'
    _order = 'sequence,id'
    _description = 'Event Question Answer'

    name = fields.Char('Answer', required=True, translate=True)
    question_id = fields.Many2one('event.question', required=True, ondelete='cascade')
    sequence = fields.Integer(default=10)

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventRegistration(models.Model):
    """ Store answers on attendees. """
    _inherit = 'event.registration'

    registration_answer_ids = fields.One2many('event.registration.answer', 'registration_id', string='Attendee Answers')

class EventRegistrationAnswer(models.Model):
    """ Represents the user input answer for a single event.question """
    _name = 'event.registration.answer'
    _description = 'Event Registration Answer'

    question_id = fields.Many2one(
        'event.question', ondelete='restrict', required=True,
        domain="[('event_id', '=', event_id)]")
    registration_id = fields.Many2one('event.registration', required=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', related='registration_id.partner_id')
    event_id = fields.Many2one('event.event', related='registration_id.event_id')
    question_type = fields.Selection(related='question_id.question_type')
    value_answer_id = fields.Many2one('event.question.answer', string="Suggested answer")
    value_text_box = fields.Text('Text answer')

    _sql_constraints = [
        ('value_check', "CHECK(value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> '')", "There must be a suggested value or a text value.")
    ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import event_event
from . import event_question
from . import event_registration

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_question,event.question,model_event_question,,1,0,0,0
access_event_question_user,event.question.user,model_event_question,event.group_event_user,1,1,1,1
access_event_question_answer,event.question.answer,model_event_question_answer,,1,0,0,0
access_event_question_answer_registration,event.question.answer.registration,model_event_question_answer,event.group_event_registration_desk,1,1,0,0
access_event_question_answer_user,event.question.answer.user,model_event_question_answer,event.group_event_user,1,1,1,1
access_event_registration_answer_registration,event.registration.answer.registration,model_event_registration_answer,event.group_event_registration_desk,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="ir_rule_event_question_published" model="ir.rule">
        <field name="name">Event Question: not event groups: event published read</field>
        <field name="model_id" ref="website_event_questions.model_event_question"/>
        <field name="domain_force">[('event_id.is_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="ir_rule_event_question_event_user" model="ir.rule">
        <field name="name">Event Question: event user: read all</field>
        <field name="model_id" ref="website_event_questions.model_event_question"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('event.group_event_registration_desk'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

    <record id="ir_rule_event_question_answer_published" model="ir.rule">
        <field name="name">Event Question Answer: not event groups: event published read</field>
        <field name="model_id" ref="website_event_questions.model_event_question_answer"/>
        <field name="domain_force">[('question_id.event_id.is_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="ir_rule_event_question_answer_event_user" model="ir.rule">
        <field name="name">Event Question Answer: event user: read all</field>
        <field name="model_id" ref="website_event_questions.model_event_question_answer"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('event.group_event_registration_desk'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</data></odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-.146-4-4.074v-21.23l18-17.474 9-7.13 10-1.018 8 3.056L46 14h9v11.204l-4 3.055 2 7.13-1 9.167L39.478 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M24.762 52.559V50c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 32l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 55.286c3.28 0 6.343-.93 8.94-2.538L41 47c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 38.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zm7.052 3.772v2.19h2.19v-2.19h-2.19zm-2.116-5.016h1.963c0-.267.027-.517.08-.749.054-.232.136-.435.247-.608a1.29 1.29 0 0 1 .428-.415c.173-.104.38-.156.62-.156.357 0 .635.109.835.326.2.218.301.554.301 1.009.009.267-.033.49-.127.667a1.959 1.959 0 0 1-.367.49 8.245 8.245 0 0 1-.494.444 3.532 3.532 0 0 0-.508.527c-.16.202-.3.447-.42.734s-.194.642-.221 1.068v.667h1.803v-.564a1.61 1.61 0 0 1 .26-.741 2.82 2.82 0 0 1 .475-.527c.178-.153.367-.306.567-.46.2-.152.383-.338.548-.555.165-.218.303-.48.414-.786.111-.307.167-.697.167-1.172a2.92 2.92 0 0 0-.167-.927 2.637 2.637 0 0 0-.554-.926 3.058 3.058 0 0 0-1.022-.72c-.423-.192-.95-.289-1.583-.289-.49 0-.933.092-1.329.275a2.956 2.956 0 0 0-1.015.763c-.28.326-.499.712-.654 1.157a4.653 4.653 0 0 0-.247 1.468z" opacity=".3"/><path fill="#FFF" d="M24.762 50.559V48c-3.65-2.952-4.016-8.81-1.095-17.571l9.767 1.485A79.883 79.883 0 0 1 33 30l10-3c3.333 10 3.667 15.667 1 17l1.4 4.898c3.439-3.111 5.6-7.61 5.6-12.612 0-9.39-7.611-17-17-17s-17 7.61-17 17c0 5.982 3.09 11.243 7.762 14.273zm2.566 1.367A16.945 16.945 0 0 0 34 53.286c3.28 0 6.343-.93 8.94-2.538L41 45c-2.868-.574-5.242-4.355-7.123-11.344-.816 9.87-2.759 15-5.83 15.392l-.72 2.878zm15.289-32.579a8.215 8.215 0 1 1 8.68 9.066A18.93 18.93 0 0 1 53 36.286c0 10.493-8.507 19-19 19s-19-8.507-19-19c0-10.494 8.507-19 19-19 3.102 0 6.03.743 8.617 2.061zm7.052 3.772v2.19h2.19v-2.19h-2.19zm-2.116-5.016h1.963c0-.267.027-.517.08-.749.054-.232.136-.435.247-.608a1.29 1.29 0 0 1 .428-.415c.173-.104.38-.156.62-.156.357 0 .635.109.835.326.2.218.301.554.301 1.009.009.267-.033.49-.127.667a1.959 1.959 0 0 1-.367.49 8.245 8.245 0 0 1-.494.444 3.532 3.532 0 0 0-.508.527c-.16.202-.3.447-.42.734s-.194.642-.221 1.068v.667h1.803v-.564a1.61 1.61 0 0 1 .26-.741 2.82 2.82 0 0 1 .475-.527c.178-.153.367-.306.567-.46.2-.152.383-.338.548-.555.165-.218.303-.48.414-.786.111-.307.167-.697.167-1.172a2.92 2.92 0 0 0-.167-.927 2.637 2.637 0 0 0-.554-.926 3.058 3.058 0 0 0-1.022-.72c-.423-.192-.95-.289-1.583-.289-.49 0-.933.092-1.329.275a2.956 2.956 0 0 0-1.015.763c-.28.326-.499.712-.654 1.157a4.653 4.653 0 0 0-.247 1.468z"/></g></g></svg>
```

## File: views\event_question_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_question_view_form" model="ir.ui.view">
        <field name="name">event.question.view.form</field>
        <field name="model">event.question</field>
        <field name="arch" type="xml">
            <form string="Question">
                <sheet>
                    <h1><field name="title" /></h1>
                    <group>
                        <group>
                            <div colspan="2">
                                <field name="once_per_order"/>
                                <label for="once_per_order"/>
                            </div>
                            <field name="question_type" widget="radio" options="{'horizontal': true}" />
                        </group>
                    </group>
                    <notebook attrs="{'invisible': [('question_type', '!=', 'simple_choice')]}">
                        <page string="Answers" name="answers">
                            <field name="answer_ids">
                                <tree editable="bottom">
                                    <!-- 'display_name' is necessary for the many2many_tags to work on the event view -->
                                    <field name="display_name" invisible="1" />
                                    <field name="sequence" widget="handle" />
                                    <field name="name"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: views\event_registration_answer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_registration_answer_view_search" model="ir.ui.view">
        <field name="name">event.registration.answer.view.search</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <search>
                <field name="value_text_box" />
                <field name="value_answer_id" />
                <field name="question_id" />
            </search>
        </field>
    </record>

    <record id="event_registration_answer_view_tree" model="ir.ui.view">
        <field name="name">event.registration.answer.view.tree</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <tree string="Answer Breakdown" create="0">
                <field name="registration_id" optional="show" />
                <field name="partner_id" optional="hide" />
                <field name="question_id" optional="show" />
                <field name="value_text_box" />
                <field name="value_answer_id" string="Selected answer" />
            </tree>
        </field>
    </record>

    <record id="event_registration_answer_view_graph" model="ir.ui.view">
        <field name="name">event.registration.answer.view.graph</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <graph string="Answer Breakdown" sample="1">
                <field name="value_answer_id" />
            </graph>
        </field>
    </record>

    <record id="event_registration_answer_view_pivot" model="ir.ui.view">
        <field name="name">event.registration.answer.view.pivot</field>
        <field name="model">event.registration.answer</field>
        <field name="arch" type="xml">
            <pivot string="Answer Breakdown" sample="1">
                <field name="registration_id" type="row"/>
                <field name="value_answer_id" type="col"/>
            </pivot>
        </field>
    </record>

    <record id="action_event_registration_report" model="ir.actions.act_window">
        <field name="name">Answer Breakdown</field>
        <field name="res_model">event.registration.answer</field>
        <field name="view_mode">search,tree,graph,pivot</field>
    </record>
</odoo>

```

## File: views\event_registration_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_registration_view_form_inherit_question" model="ir.ui.view">
        <field name="name">event.registration.view.form.inherit.question</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_form" />
        <field name="arch" type="xml">
            <sheet position="inside">
                <notebook>
                    <page string="Questions" name="questions">
                        <field name="registration_answer_ids" widget="one2many">
                            <tree editable="bottom">
                                <field name="event_id" invisible="1" />
                                <field name="question_id" domain="[('event_id', '=', event_id)]" options="{'no_create': True}" />
                                <field name="question_type" string="Type" />
                                <field name="value_answer_id"
                                    attrs="{'invisible': [('question_type', '!=', 'simple_choice')]}"
                                    domain="[('question_id', '=', question_id)]" options="{'no_create': True}"/>
                                <field name="value_text_box" attrs="{'invisible': [('question_type', '!=', 'text_box')]}" />
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
        </field>
    </record>
</odoo>

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
                    <t t-call="website_event_questions.registration_event_question">
                        <t t-set="registration_index" t-value="counter"/>
                    </t>
                </div>
            </div>
        </t>
    </xpath>
    <!-- Generic questions -->
    <xpath expr="//*[hasclass('modal-footer')]" position="before">
        <t t-if="event.general_question_ids">
            <div class="modal-body bg-light border-bottom o_wevent_registration_question_global">
                <div t-foreach="event.general_question_ids" t-as="question" class="row">
                    <div class="col-lg-6 mt-2">
                        <t t-call="website_event_questions.registration_event_question">
                            <t t-set="registration_index" t-value="0"/>
                        </t>
                    </div>
                </div>
            </div>
        </t>
    </xpath>
</template>

<template id="registration_event_question" name="Registration Event Question">
    <label t-esc="question.title"/>
    <t t-if="question.question_type == 'simple_choice'">
        <select t-attf-name="question_answer-#{registration_index}-#{question.id}" class="custom-select" required="true">
            <t t-foreach="question.answer_ids" t-as="answer">
                <option t-esc="answer.name" t-att-value="answer.id"/>
            </t>
        </select>
    </t>
    <t t-elif="question.question_type == 'text_box'">
        <textarea t-attf-name="question_answer-#{registration_index}-#{question.id}" class="col-lg-12 form-control"/>
    </t>
</template>

</odoo>

```

## File: views\event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_type_view_form_inherit_question" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.question</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
            <page name="event_type_communication" position="after">
                <page string="Questions">
                     <field name="question_ids" class="w-100">
                         <tree sample="1">
                             <field name="title"/>
                             <field name="question_type" />
                             <field name="answer_ids" widget = "many2many_tags"/>
                         </tree>
                     </field>
                </page>
            </page>
        </field>
    </record>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.question</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="website_event.event_event_view_form"/>
        <field name="arch" type="xml">
            <data>
                <page name="event_notes" position="before">
                    <page string="Questions" name="questions">
                        <field name="question_ids" nolabel="1">
                            <tree>
                                <field name="sequence" widget="handle" />
                                <field name="title"/>
                                <field name="once_per_order"/>
                                <field name="question_type" string="Type" />
                                <field name="answer_ids" widget="many2many_tags"
                                    attrs="{'invisible': [('question_type', '!=', 'simple_choice')]}" />
                                <button name="action_view_question_answers" type="object" class="fa fa-bar-chart p-0" title="Answer Breakdown" />
                            </tree>
                            <!-- Need to repeat the whole tree form here to be able to create answers properly
                                Without this, the sub-fields of answer_ids are unknown to the web framework.
                                We need this because we create questions and answers when the event type changes. -->
                            <form string="Question">
                                <sheet>
                                    <h1><field name="title" /></h1>
                                    <group class="mb-0">
                                        <group class="mb-0">
                                            <div colspan="2">
                                                <field name="once_per_order"/>
                                                <label for="once_per_order"/>
                                            </div>
                                            <field name="question_type" widget="radio" options="{'horizontal': true}" />
                                        </group>
                                    </group>
                                    <notebook attrs="{'invisible': [('question_type', '!=', 'simple_choice')]}">
                                        <page string="Answers" name="answers">
                                            <field name="answer_ids">
                                                <tree editable="bottom">
                                                    <!-- 'display_name' is necessary for the many2many_tags to work on the event view -->
                                                    <field name="display_name" invisible="1" />
                                                    <field name="sequence" widget="handle" />
                                                    <field name="name"/>
                                                </tree>
                                            </field>
                                        </page>
                                    </notebook>
                                </sheet>
                            </form>
                        </field>
                    </page>
                </page>
        </data>
        </field>
    </record>
</odoo>

```

