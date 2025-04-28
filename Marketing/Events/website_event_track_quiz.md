# Odoo Module: website_event_track_quiz

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Quizzes on Tracks',
    'category': 'Marketing/Events',
    'sequence': 1007,
    'version': '1.0',
    'summary': 'Quizzes on tracks',
    'website': 'https://www.odoo.com/app/events',
    'depends': [
        'website_profile',
        'website_event_track',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/event_leaderboard_templates.xml',
        'views/event_quiz_views.xml',
        'views/event_quiz_question_views.xml',
        'views/event_track_views.xml',
        'views/event_track_visitor_views.xml',
        'views/event_menus.xml',
        'views/event_quiz_templates.xml',
        'views/event_track_templates_page.xml',
        'views/event_event_views.xml',
        'views/event_type_views.xml'
    ],
    'demo': [
        'data/quiz_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_event_track_quiz/static/src/scss/event_quiz.scss',
            'website_event_track_quiz/static/src/js/event_quiz.js',
            'website_event_track_quiz/static/src/js/event_quiz_leaderboard.js',
            'website_event_track_quiz/static/src/xml/quiz_templates.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\community.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math

from odoo import http
from odoo.addons.http_routing.models.ir_http import slug
from odoo.addons.website_event.controllers.community import EventCommunityController
from odoo.http import request


class WebsiteEventTrackQuizCommunityController(EventCommunityController):

    _visitors_per_page = 30
    _pager_max_pages = 5

    @http.route(['/event/<model("event.event"):event>/community/leaderboard/results',
                 '/event/<model("event.event"):event>/community/leaderboard/results/page/<int:page>'],
                type='http', auth="public", website=True, sitemap=False)
    def leaderboard(self, event, page=1, lang=None, **kwargs):
        values = self._get_community_leaderboard_render_values(event, kwargs.get('search'), page)
        return request.render('website_event_track_quiz.event_leaderboard', values)

    @http.route('/event/<model("event.event"):event>/community/leaderboard',
                type='http', auth="public", website=True, sitemap=False)
    def community_leaderboard(self, event, **kwargs):
        values = self._get_community_leaderboard_render_values(event, None, None)
        return request.render('website_event_track_quiz.event_leaderboard', values)

    @http.route()
    def community(self, event, **kwargs):
        values = self._get_community_leaderboard_render_values(event, None, None)
        return request.render('website_event_track_quiz.event_leaderboard', values)

    def _get_community_leaderboard_render_values(self, event, search_term, page):
        values = self._get_leaderboard(event, search_term)
        values.update({'event': event, 'search': search_term})

        user_count = len(values['visitors'])
        if user_count:
            page_count = math.ceil(user_count / self._visitors_per_page)
            url = '/event/%s/community/leaderboard/results' % (slug(event))
            if values.get('current_visitor_position') and not page:
                values['scroll_to_position'] = True
                page = math.ceil(values['current_visitor_position'] / self._visitors_per_page)
            elif not page:
                page = 1
            pager = request.website.pager(url=url, total=user_count, page=page, step=self._visitors_per_page,
                                          scope=page_count if page_count < self._pager_max_pages else self._pager_max_pages,
                                          url_args={'search': search_term})
            values['visitors'] = values['visitors'][(page - 1) * self._visitors_per_page: (page) * self._visitors_per_page]
        else:
            pager = {'page_count': 0}
        values.update({'pager': pager})
        return values

    def _get_leaderboard(self, event, searched_name=None):
        current_visitor = request.env['website.visitor']._get_visitor_from_request(force_create=False)
        track_visitor_data = request.env['event.track.visitor'].sudo()._read_group(
            [('track_id', 'in', event.track_ids.ids),
             ('visitor_id', '!=', False),
             ('quiz_points', '>', 0)],
            ['visitor_id'],
            ['quiz_points:sum'], order='quiz_points:sum DESC, visitor_id ASC')
        data_map = {visitor.id: points for visitor, points in track_visitor_data}
        leaderboard = []
        position = 1
        current_visitor_position = False
        visitors_by_id = {
            visitor.id: visitor
            for visitor in request.env['website.visitor'].sudo().browse(data_map.keys())
        }
        for visitor_id, points in data_map.items():
            visitor = visitors_by_id.get(visitor_id)
            if not visitor:
                continue
            if (searched_name and searched_name.lower() in visitor.display_name.lower()) or not searched_name:
                leaderboard.append({'visitor': visitor, 'points': points, 'position': position})
                if current_visitor and current_visitor == visitor:
                    current_visitor_position = position
            position = position + 1

        return {
            'top3_visitors': leaderboard[:3],
            'visitors': leaderboard,
            'current_visitor_position': current_visitor_position,
            'current_visitor': current_visitor,
            'searched_name': searched_name
        }

```

## File: controllers\event_track_quiz.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.addons.website_event_track.controllers.event_track import EventTrackController
from odoo.http import request


class WebsiteEventTrackQuiz(EventTrackController):

    # QUIZZES IN PAGE
    # ----------------------------------------------------------

    @http.route('/event_track/quiz/submit', type="json", auth="public", website=True)
    def event_track_quiz_submit(self, event_id, track_id, answer_ids):
        track = self._fetch_track(track_id)
        track_sudo = track.sudo()

        event_track_visitor = track._get_event_track_visitors(force_create=True)
        if event_track_visitor.quiz_completed:
            return {'error': 'track_quiz_done'}

        # fetch as sudo because questions / answers may not be freely available to public
        answers_details = self._get_quiz_answers_details(track_sudo, answer_ids)
        if answers_details.get('error'):
            return answers_details

        event_track_visitor.write({
            'quiz_completed': True,
            'quiz_points': answers_details['points'],
        })

        result = {
            'answers': {
                answer.question_id.id: {
                    'awarded_points': answer.awarded_points,
                    'correct_answer': answer.question_id.correct_answer_id.text_value,
                    'is_correct': answer.is_correct,
                    'comment': answer.comment
                } for answer in answers_details['user_answers']
            },
            'quiz_completed': event_track_visitor.quiz_completed,
            'quiz_points': answers_details['points']
        }
        return result

    @http.route('/event_track/quiz/reset', type="json", auth="public", website=True)
    def quiz_reset(self, event_id, track_id):
        track = self._fetch_track(track_id)
        # When the 'unlimited tries' option is disabled and the user is not
        # identifed as an event manager, we do not allow the user to reset
        # the quiz. The event managers will always be able to reset the quiz
        # even if the option is disabled (for testing purposes).
        if not request.env.user.has_group('event.group_event_manager') and not track.sudo().quiz_id.repeatable:
            raise Forbidden()

        event_track_visitor = track._get_event_track_visitors(force_create=True)
        event_track_visitor.write({
            'quiz_completed': False,
            'quiz_points': 0,
        })

    def _get_quiz_answers_details(self, track, answer_ids):
        questions_count = track.quiz_questions_count
        user_answers = request.env['event.quiz.answer'].sudo().search([('id', 'in', answer_ids)])

        if len(user_answers.mapped('question_id')) != questions_count:
            return {'error': 'quiz_incomplete'}

        return {
            'user_answers': user_answers,
            'points': sum([
                answer.awarded_points
                for answer in user_answers
            ])
        }

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_track_quiz
from . import community

```

## File: data\quiz_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="event_7_track_1_quiz" model="event.quiz">
        <field name="name">What This Event Is All About</field>
        <field name="event_track_id" ref="website_event_track.event_7_track_1"/>
    </record>

    <record id="event_7_track_1_question_0" model="event.quiz.question">
        <field name="name">What will we talk about during this event?</field>
        <field name="sequence">1</field>
        <field name="quiz_id" ref="website_event_track_quiz.event_7_track_1_quiz"/>
    </record>
    <record id="event_7_track_1_question_0_0" model="event.quiz.answer">
        <field name="text_value">Wood</field>
        <field name="sequence">1</field>
        <field name="awarded_points">1</field>
        <field name="is_correct" eval="True"/>
        <field name="comment">You're really smart!</field>
        <field name="question_id" ref="event_7_track_1_question_0"/>
    </record>
    <record id="event_7_track_1_question_0_1" model="event.quiz.answer">
        <field name="text_value">Music</field>
        <field name="sequence">2</field>
        <field name="comment">Even if there will be some.</field>
        <field name="question_id" ref="event_7_track_1_question_0"/>
    </record>
    <record id="event_7_track_1_question_0_2" model="event.quiz.answer">
        <field name="text_value">Open Source Apps</field>
        <field name="sequence">3</field>
        <field name="comment">OpenWood is not an Open Source congres about Apps.</field>
        <field name="question_id" ref="event_7_track_1_question_0"/>
    </record>
    <record id="event_7_track_1_question_1" model="event.quiz.question">
        <field name="name">Where does lumber comes from?</field>
        <field name="sequence">2</field>
        <field name="quiz_id" ref="website_event_track_quiz.event_7_track_1_quiz"/>
    </record>
    <record id="event_7_track_1_question_1_0" model="event.quiz.answer">
        <field name="text_value">Trees!</field>
        <field name="sequence">1</field>
        <field name="awarded_points">1</field>
        <field name="is_correct" eval="True"/>
        <field name="question_id" ref="event_7_track_1_question_1"/>
    </record>
    <record id="event_7_track_1_question_1_1" model="event.quiz.answer">
        <field name="text_value">Stores!</field>
        <field name="sequence">2</field>
        <field name="comment">Lumbers need first to be cut from trees!</field>
        <field name="question_id" ref="event_7_track_1_question_1"/>
    </record>

    <record id="event_7_track_5_quiz" model="event.quiz">
        <field name="name">Securing your Lumber during transport</field>
        <field name="event_track_id" ref="website_event_track.event_7_track_5"/>
    </record>

    <record id="event_7_track_5_question_0" model="event.quiz.question">
        <field name="name">Transporting lumber from stores to your house is safe.</field>
        <field name="sequence">1</field>
        <field name="quiz_id" ref="website_event_track_quiz.event_7_track_5_quiz"/>
    </record>
    <record id="event_7_track_5_question_0_0" model="event.quiz.answer">
        <field name="text_value">Yes</field>
        <field name="sequence">1</field>
        <field name="comment">In order to avoid accident, you need to secure any product of this kind during transportation!</field>
        <field name="question_id" ref="event_7_track_5_question_0"/>
    </record>
    <record id="event_7_track_5_question_0_1" model="event.quiz.answer">
        <field name="text_value">No</field>
        <field name="sequence">2</field>
        <field name="awarded_points">1</field>
        <field name="is_correct" eval="True"/>
        <field name="comment">Even if you have a big trunk, some long products need to be secured.</field>
        <field name="question_id" ref="event_7_track_5_question_0"/>
    </record>
    <record id="event_7_track_5_question_1" model="event.quiz.question">
        <field name="name">What kind of tool are needed to secure your lumber?</field>
        <field name="sequence">2</field>
        <field name="quiz_id" ref="website_event_track_quiz.event_7_track_5_quiz"/>
    </record>
    <record id="event_7_track_5_question_1_0" model="event.quiz.answer">
        <field name="text_value">Hammer</field>
        <field name="sequence">1</field>
        <field name="comment">Hammer won't be of any help here!</field>
        <field name="question_id" ref="event_7_track_5_question_1"/>
    </record>
    <record id="event_7_track_5_question_1_1" model="event.quiz.answer">
        <field name="text_value">Tie-down straps and other wooden blocks</field>
        <field name="sequence">2</field>
        <field name="awarded_points">1</field>
        <field name="is_correct" eval="True"/>
        <field name="question_id" ref="event_7_track_5_question_1"/>
    </record>
        <record id="event_7_track_5_question_1_2" model="event.quiz.answer">
        <field name="text_value">Scotch tape</field>
        <field name="sequence">3</field>
        <field name="comment">Well, it could work but you will need a lot of tape!</field>
        <field name="question_id" ref="event_7_track_5_question_1"/>
    </record>

    <record id="event_7_track_13_quiz" model="event.quiz">
        <field name="name">Pretty. Ugly. Lovely.</field>
        <field name="event_track_id" ref="website_event_track.event_7_track_13"/>
    </record>

    <record id="event_7_track_13_question_0" model="event.quiz.question">
        <field name="name">What kind of wall is transformed here?</field>
        <field name="sequence">1</field>
        <field name="quiz_id" ref="website_event_track_quiz.event_7_track_13_quiz"/>
    </record>
    <record id="event_7_track_13_question_0_0" model="event.quiz.answer">
        <field name="text_value">Concrete Blocks Wall</field>
        <field name="sequence">1</field>
        <field name="awarded_points">1</field>
        <field name="is_correct" eval="True"/>
        <field name="question_id" ref="event_7_track_13_question_0"/>
    </record>
    <record id="event_7_track_13_question_0_1" model="event.quiz.answer">
        <field name="text_value">Steel Wall</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="event_7_track_13_question_0"/>
    </record>
    <record id="event_7_track_13_question_0_2" model="event.quiz.answer">
        <field name="text_value">Mud Wall</field>
        <field name="sequence">3</field>
        <field name="question_id" ref="event_7_track_13_question_0"/>
    </record>

    <record id="event.event_7" model="event.event">
        <field name="community_menu" eval="True"/>
    </record>

    <record id="event_track_visitor_admin_event_7_track_1" model="event.track.visitor">
        <field name="track_id" ref="website_event_track.event_7_track_1"/>
        <field name="visitor_id" ref="website.website_visitor_0"/>
        <field name="quiz_completed" eval="True"/>
        <field name="quiz_points">3</field>
    </record>

    <record id="event_track_visitor_demo_event_7_track_1" model="event.track.visitor">
        <field name="track_id" ref="website_event_track.event_7_track_1"/>
        <field name="visitor_id" ref="website.website_visitor_1"/>
        <field name="quiz_completed" eval="True"/>
        <field name="quiz_points">2</field>
    </record>

    <record id="event_track_visitor_portal_event_7_track_1" model="event.track.visitor">
        <field name="track_id" ref="website_event_track.event_7_track_1"/>
        <field name="visitor_id" ref="website.website_visitor_2"/>
        <field name="quiz_completed" eval="True"/>
        <field name="quiz_points">1</field>
    </record>

</data></odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Event(models.Model):
    _inherit = "event.event"

    @api.depends("event_type_id", "website_menu", "community_menu")
    def _compute_community_menu(self):
        """ At type onchange: synchronize. At website_menu update: synchronize. """
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.community_menu = event.event_type_id.community_menu
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.community_menu):
                event.community_menu = True
            elif not event.website_menu:
                event.community_menu = False

```

## File: models\event_quiz.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class Quiz(models.Model):
    _name = "event.quiz"
    _description = "Quiz"

    name = fields.Char('Name', required=True, translate=True)
    question_ids = fields.One2many('event.quiz.question', 'quiz_id', string="Questions")
    event_track_id = fields.Many2one('event.track', readonly=True)
    event_id = fields.Many2one(
        'event.event', related='event_track_id.event_id',
        readonly=True, store=True)
    repeatable = fields.Boolean('Unlimited Tries',
        help='Let attendees reset the quiz and try again.')


class QuizQuestion(models.Model):
    _name = "event.quiz.question"
    _description = "Content Quiz Question"
    _order = "quiz_id, sequence, id"

    name = fields.Char("Question", required=True, translate=True)
    sequence = fields.Integer("Sequence")
    quiz_id = fields.Many2one("event.quiz", "Quiz", required=True, ondelete='cascade')
    correct_answer_id = fields.One2many('event.quiz.answer', compute='_compute_correct_answer_id')
    awarded_points = fields.Integer("Number of Points", compute='_compute_awarded_points')
    answer_ids = fields.One2many('event.quiz.answer', 'question_id', string="Answer")

    @api.depends('answer_ids.awarded_points')
    def _compute_awarded_points(self):
        for question in self:
            question.awarded_points = sum(question.answer_ids.mapped('awarded_points'))

    @api.depends('answer_ids.is_correct')
    def _compute_correct_answer_id(self):
        for question in self:
            question.correct_answer_id = question.answer_ids.filtered(lambda e: e.is_correct)

    @api.constrains('answer_ids')
    def _check_answers_integrity(self):
        for question in self:
            if len(question.correct_answer_id) != 1:
                raise ValidationError(_('Question "%s" must have 1 correct answer to be valid.', question.name))
            if len(question.answer_ids) < 2:
                raise ValidationError(_('Question "%s" must have 1 correct answer and at least 1 incorrect answer to be valid.', question.name))


class QuizAnswer(models.Model):
    _name = "event.quiz.answer"
    _rec_name = "text_value"
    _description = "Question's Answer"
    _order = 'question_id, sequence, id'

    sequence = fields.Integer("Sequence")
    question_id = fields.Many2one('event.quiz.question', string="Question", required=True, ondelete='cascade')
    text_value = fields.Char("Answer", required=True, translate=True)
    is_correct = fields.Boolean('Correct', default=False)
    comment = fields.Text(
        'Extra Comment', translate=True,
        help='''This comment will be displayed to the user if they select this answer, after submitting the quiz.
                It is used as a small informational text helping to understand why this answer is correct / incorrect.''')
    awarded_points = fields.Integer('Points', default=0)

```

## File: models\event_track.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


class EventTrack(models.Model):
    _inherit = ['event.track']

    quiz_id = fields.Many2one('event.quiz', string="Quiz", compute='_compute_quiz_id', store=True, groups="event.group_event_user")
    quiz_ids = fields.One2many('event.quiz', 'event_track_id', string="Quizzes")
    quiz_questions_count = fields.Integer(string="# Quiz Questions", compute='_compute_quiz_questions_count', groups="event.group_event_user")
    is_quiz_completed = fields.Boolean('Is Quiz Done', compute='_compute_quiz_data')
    quiz_points = fields.Integer('Quiz Points', compute='_compute_quiz_data')

    @api.depends('quiz_ids.event_track_id')
    def _compute_quiz_id(self):
        for track in self:
            track.quiz_id = track.quiz_ids[0] if track.quiz_ids else False

    @api.depends('quiz_id.question_ids')
    def _compute_quiz_questions_count(self):
        for track in self:
            track.quiz_questions_count = len(track.quiz_id.question_ids)

    @api.depends('quiz_id', 'event_track_visitor_ids.visitor_id',
                 'event_track_visitor_ids.partner_id', 'event_track_visitor_ids.quiz_completed',
                 'event_track_visitor_ids.quiz_points')
    @api.depends_context('uid')
    def _compute_quiz_data(self):
        tracks_quiz = self.filtered(lambda track: track.quiz_id)
        (self - tracks_quiz).is_quiz_completed = False
        (self - tracks_quiz).quiz_points = 0
        if tracks_quiz:
            current_visitor = self.env['website.visitor']._get_visitor_from_request(force_create=False)
            if self.env.user._is_public() and not current_visitor:
                for track in tracks_quiz:
                    track.is_quiz_completed = False
                    track.quiz_points = 0
            else:
                if self.env.user._is_public():
                    domain = [('visitor_id', '=', current_visitor.id)]
                elif current_visitor:
                    domain = [
                        '|',
                        ('partner_id', '=', self.env.user.partner_id.id),
                        ('visitor_id', '=', current_visitor.id)
                    ]
                else:
                    domain = [('partner_id', '=', self.env.user.partner_id.id)]

                event_track_visitors = self.env['event.track.visitor'].sudo().search_read(
                    expression.AND([
                        domain,
                        [('track_id', 'in', tracks_quiz.ids)]
                    ]), fields=['track_id', 'quiz_completed', 'quiz_points']
                )

                quiz_visitor_map = {
                    track_visitor['track_id'][0]: {
                        'quiz_completed': track_visitor['quiz_completed'],
                        'quiz_points': track_visitor['quiz_points']
                    } for track_visitor in event_track_visitors
                }
                for track in tracks_quiz:
                    if quiz_visitor_map.get(track.id):
                        track.is_quiz_completed = quiz_visitor_map[track.id]['quiz_completed']
                        track.quiz_points = quiz_visitor_map[track.id]['quiz_points']
                    else:
                        track.is_quiz_completed = False
                        track.quiz_points = 0

    def action_add_quiz(self):
        self.ensure_one()
        event_quiz_form = self.env.ref('website_event_track_quiz.event_quiz_view_form')
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'event.quiz',
            'view_id': event_quiz_form.id,
            'context': {
                'default_event_track_id': self.id,
                'create': False,
            },
        }

    def action_view_quiz(self):
        self.ensure_one()
        event_quiz_form = self.env.ref('website_event_track_quiz.event_quiz_view_form')
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'event.quiz',
            'res_id' : self.quiz_id.id,
            'view_id': event_quiz_form.id,
            'context': {
                'create': False,
            }
        }

```

## File: models\event_track_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class TrackVisitor(models.Model):
    _name = 'event.track.visitor'
    _inherit = ['event.track.visitor']

    quiz_completed = fields.Boolean('Completed')
    quiz_points = fields.Integer("Quiz Points", default=0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_event
from . import event_track
from . import event_track_visitor
from . import event_quiz

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
event_quiz_access_event_user,event.quiz.access.event.user,model_event_quiz,event.group_event_user,1,1,1,1
event_quiz_question_access_event_user,event.quiz.question.access.event.user,model_event_quiz_question,event.group_event_user,1,1,1,1
event_quiz_answer_access_event_user,event.quiz.answer.event.user,model_event_quiz_answer,event.group_event_user,1,1,1,1

```

## File: static\src\js\event_quiz.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { session } from "@web/session";
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";

/**
 * This widget is responsible of displaying quiz questions and propositions. Submitting the quiz will fetch the
 * correction and decorate the answers according to the result. Error message can be displayed.
 *
 * This widget can be attached to DOM rendered server-side by `gamification_quiz.`
 *
 */
var Quiz = publicWidget.Widget.extend({
    template: 'quiz.main',
    events: {
        "click .o_quiz_quiz_answer": '_onAnswerClick',
        "click .o_quiz_js_quiz_submit": '_submitQuiz',
        "click .o_quiz_js_quiz_reset": '_onClickReset',
    },

    /**
    * @override
    * @param {Object} parent
    * @param {Object} data holding all the container information
    * @param {Object} quizData : quiz data to display
    */
    init: function (parent, data, quizData) {
        this._super.apply(this, arguments);
        this.track = Object.assign({
            id: 0,
            name: '',
            eventId: '',
            completed: false,
            isMember: false,
            progressBar: false,
            isEventUser: false,
            repeatable: false
        }, data);
        this.quiz = quizData || false;
        if (this.quiz) {
            this.quiz.questionsCount = quizData.questions.length;
        }
        this.isMember = data.isMember || false;
        this.userId = session.user_id;
        this.redirectURL = encodeURIComponent(document.URL);

        this.rpc = this.bindService("rpc");
        this.notification = this.bindService("notification");
    },

    /**
     * @override
     */
    willStart: function () {
        var defs = [this._super.apply(this, arguments)];
        if (!this.quiz) {
            defs.push(this._fetchQuiz());
        }
        return Promise.all(defs);
    },

    /**
     * Overridden to add custom rendering behavior upon start of the widget.
     *
     * If the user has answered the quiz before having joined the course, we check
     * their answers (saved into their session) here as well.
     *
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function ()  {
            self._renderValidationInfo();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _alertShow: function (alertCode) {
        var message = _t('There was an error validating this quiz.');
        if (alertCode === 'quiz_incomplete') {
            message = _t('All questions must be answered!');
        } else if (alertCode === 'quiz_done') {
            message = _t('This quiz is already done. Retaking it is not possible.');
        }

        this.notification.add(message, {
            type: 'warning',
            title: _t('Quiz validation error'),
            sticky: true
        });
    },

    /**
     * @private
     * Decorate the answers according to state
     */
    _disableAnswers: function () {
        this.$('.o_quiz_js_quiz_question').addClass('completed-disabled');
        this.$('input[type=radio]').prop('disabled', true);
    },

    /**
     * @private
     * Decorate the answers according to state
     */
    _enableAnswers: function() {
        this.$('.o_quiz_js_quiz_question').removeClass('completed-disabled');
        this.$('input[type=radio]').prop('disabled', false);
    },

    /**
     * Get all the questions ID from the displayed Quiz
     * @returns {Array}
     * @private
     */
    _getQuestionsIds: function () {
        return this.$('.o_quiz_js_quiz_question').map(function () {
            return $(this).data('question-id');
        }).get();
    },

    /**
     * Get the quiz answers filled in by the User
     *
     * @private
     */
    _getQuizAnswers: function () {
        return this.$('input[type=radio]:checked').map(function (index, element) {
            return parseInt($(element).val());
        }).get();
    },

    /**
     * Decorate the answer inputs according to the correction and adds the answer comment if
     * any.
     *
     * @private
     */
    _renderAnswersHighlightingAndComments: function () {
        var self = this;
        this.$('.o_quiz_js_quiz_question').each(function () {
            var $question = $(this);
            var questionId = $question.data('questionId');
            var answer = self.quiz.answers[questionId];
            $question.find('a.o_quiz_quiz_answer').each(function () {
                var $answer = $(this);
                $answer.find('i.fa').addClass('d-none');
                if ($answer.find('input[type=radio]').is(':checked')) {
                    if (answer.is_correct) {
                        $answer.find('i.fa-check-circle').removeClass('d-none');
                    } else {
                        $answer.find('label input').prop('checked', false);
                        $answer.find('i.fa-times-circle').removeClass('d-none');
                    }
                    if (answer.awarded_points > 0) {
                        $answer.append(renderToElement('quiz.badge', {'answer': answer}));
                    }
                } else {
                    $answer.find('i.fa-circle').removeClass('d-none');
                }
            });
            var $list = $question.find('.list-group');
            $list.append(renderToElement('quiz.comment', {'answer': answer}));
        });
    },

    /*
        * @private
        * Update validation box (karma, buttons) according to widget state
        */
    _renderValidationInfo: function () {
        var $validationElem = this.$('.o_quiz_js_quiz_validation');
        $validationElem.empty().append(
            renderToElement('quiz.validation', {'widget': this})
        );
    },

    /**
     * Remove the answer decorators
     */
     _resetQuiz: function () {
        this.$('.o_quiz_js_quiz_question').each(function () {
            var $question = $(this);
            $question.find('a.o_quiz_quiz_answer').each(function () {
                var $answer = $(this);
                $answer.find('i.fa').addClass('d-none');
                $answer.find('i.fa-circle').removeClass('d-none');
                $answer.find('span.badge').remove();
                $answer.find('input[type=radio]').prop('checked', false);
            });
            var $info = $question.find('.o_quiz_quiz_answer_info');
            $info.remove();
        });
        this.track.completed = false;
        this._enableAnswers();
        this._renderValidationInfo();
    },

    /**
     * Submit a quiz and get the correction. It will display messages
     * according to quiz result.
     *
     * @private
     */
    _submitQuiz: function () {
        var self = this;

        return this.rpc('/event_track/quiz/submit', {
            event_id: self.track.eventId,
            track_id: self.track.id,
            answer_ids: this._getQuizAnswers(),
        }).then(function (data) {
            if (data.error) {
                self._alertShow(data.error);
            } else {
                self.quiz = Object.assign(self.quiz, data);
                self.quiz.quizPointsGained = data.quiz_points;
                if (data.quiz_completed) {
                    self._disableAnswers();
                    self.track.completed = data.quiz_completed;
                }
                self._renderAnswersHighlightingAndComments();
                self._renderValidationInfo();
            }

            return Promise.resolve(data);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * When clicking on an answer, this one should be marked as "checked".
     *
     * @private
     * @param OdooEvent ev
     */
    _onAnswerClick: function (ev) {
        ev.preventDefault();
        if (!this.track.completed) {
            $(ev.currentTarget).find('input[type=radio]').prop('checked', true);
        }
    },

    /**
     * Resets the completion of the track so the user can take
     * the quiz again
     *
     * @private
     */
    _onClickReset: function () {
        this.rpc('/event_track/quiz/reset', {
            event_id: this.track.eventId,
            track_id: this.track.id
        }).then(this._resetQuiz.bind(this));
    },

});

publicWidget.registry.Quiz = publicWidget.Widget.extend({
    selector: '.o_quiz_main',

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * @override
     * @param {Object} parent
     */
    start: function () {
        var self = this;
        this.quizWidgets = [];
        var defs = [this._super.apply(this, arguments)];
        this.$('.o_quiz_js_quiz').each(function () {
            var data = $(this).data();
            data.quizData = {
                questions: self._extractQuestionsAndAnswers(),
                sessionAnswers: data.sessionAnswers || [],
                quizKarmaMax: data.quizKarmaMax,
                quizKarmaWon: data.quizKarmaWon,
                quizKarmaGain: data.quizKarmaGain,
                quizPointsGained: data.quizPointsGained,
                quizAttemptsCount: data.quizAttemptsCount,
            };
            defs.push(new Quiz(self, data, data.quizData).attachTo($(this)));
        });
        return Promise.all(defs);
    },

    //----------------------------------------------------------------------
    // Private
    //---------------------------------------------------------------------

    /**
     * Extract data from exiting DOM rendered server-side, to have the list of questions with their
     * relative answers.
     * This method should return the same format as /gamification_quiz/quiz/get controller.
     *
     * @return {Array<Object>} list of questions with answers
     */
    _extractQuestionsAndAnswers: function () {
        var questions = [];
        this.$('.o_quiz_js_quiz_question').each(function () {
            var $question = $(this);
            var answers = [];
            $question.find('.o_quiz_quiz_answer').each(function () {
                var $answer = $(this);
                answers.push({
                    id: $answer.data('answerId'),
                    text: $answer.data('text'),
                });
            });
            questions.push({
                id: $question.data('questionId'),
                title: $question.data('title'),
                answer_ids: answers,
            });
        });
        return questions;
    },
});

export default Quiz;

```

## File: static\src\js\event_quiz_leaderboard.js

```javascript
/** @odoo-module **/


import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.EventLeaderboard = publicWidget.Widget.extend({
    selector: '.o_wevent_quiz_leaderboard',

    /**
     * Basic override to scroll to current visitor's position.
     */
    start: function () {
        var self = this;
        return this._super(...arguments).then(function () {
            var $scrollTo = self.$('.o_wevent_quiz_scroll_to');
            if ($scrollTo.length !== 0) {
                var offset = $('.o_header_standard').height();
                var $appMenu = $('.o_main_navbar');
                if ($appMenu.length !== 0) {
                    offset += $appMenu.height();
                }
                window.scrollTo({
                    top: $scrollTo.offset().top - offset,
                    behavior: 'smooth'
                });
            }
        });
    }
});

export default publicWidget.registry.EventLeaderboard;

```

## File: static\src\xml\quiz_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="quiz.main">
        <div class="h-100 w-100 overflow-auto px-2 py-2">
            <div class="container">
                <div t-foreach="widget.quiz.questions" t-as="question" t-key="question_index"
                     t-attf-class="o_quiz_js_quiz_question mt-3 mb-4 #{widget.track.completed ? 'completed-disabled' : ''}"
                     t-att-data-question-id="question.id" t-att-data-title="question.question">
                    <div class="h4">
                        <small class="text-muted"><span t-out="question_index+1"/>. </small> <span t-out="question.question"/>
                    </div>
                    <div class="list-group">
                        <t t-foreach="question.answer_ids" t-as="answer" t-key="answer_index">
                            <a t-att-data-answer-id="answer.id" href="#"
                                t-att-data-text="answer.text_value"
                                t-attf-class="o_quiz_quiz_answer list-group-item d-flex align-items-center list-group-item-action #{widget.track.completed  &amp;&amp; answer.is_correct ? 'list-group-item-success' : '' }">

                                <label class="my-0 d-flex align-items-center justify-content-center me-2">
                                    <input type="radio"
                                        t-att-name="question.id"
                                        t-att-value="answer.id"
                                        class="d-none"/>
                                    <i t-att-class="'fa fa-circle text-400' + (!(widget.track.completed &amp;&amp; answer.is_correct) ? '' : ' d-none')"></i>
                                    <i class="fa fa-times-circle text-danger d-none"></i>
                                    <i t-att-class="'fa fa-check-circle text-success' + (widget.track.completed &amp;&amp; answer.is_correct ? '' :  ' d-none')"></i>
                                </label>
                                <span t-out="answer.text_value"/>
                            </a>
                        </t>
                    </div>
                </div>
            </div>
        </div>
    </t>

    <t t-name="quiz.badge">
        <span class="badge text-bg-warning ms-2">
            <span>+ <t t-out="answer.awarded_points"/></span>
            <span t-if="answer.awarded_points == 1">Point</span>
            <span t-else="">Points</span>
        </span>
    </t>

    <t t-name="quiz.comment">
        <t t-if="answer.is_correct">
            <div class="o_quiz_quiz_answer_info list-group-item list-group-item-success">
                <i class="fa fa-info-circle"/>
                Correct.
                <t t-if="answer.comment">
                    <br/>
                    <t t-out="answer.comment"/>
                </t>
            </div>
        </t>
        <t t-else="">
            <div class="o_quiz_quiz_answer_info list-group-item list-group-item-danger">
                <i class="fa fa-info-circle"/>
                Incorrect. <t t-if="answer.correct_answer">The correct answer was: <t t-out="answer.correct_answer"/></t>
                <t t-if="answer.comment">
                    <br/>
                    <t t-out="answer.comment"/>
                </t>
            </div>
        </t>
    </t>

    <t t-name="quiz.validation">
        <div id="validation" class="d-md-flex">
            <div class="flex-grow-1">
                <button t-if="!widget.track.completed" role="button" title="Check answers" aria-label="Check answers"
                    class="btn btn-primary text-uppercase fw-bold o_quiz_js_quiz_submit">
                    Check your answers
                </button>
                <t t-else="">
                    <div t-if="widget.quiz.quizPointsGained > 0" class="alert alert-warning me-md-3">
                        Congratulations, you scored a total of <span t-out="widget.quiz.quizPointsGained" class="fw-bold"/>
                        <span t-if="widget.quiz.quizPointsGained == 1">point!</span>
                        <span t-else="">points!</span>
                    </div>
                    <div t-else="" class="alert alert-warning me-md-3">
                        Oopsie, you did not score any point on this quiz.
                    </div>
                </t>
            </div>
            <div class="o_quiz_js_quiz_actions mt-3 mt-md-0">
                <button t-if="widget.track.isEventUser or widget.track.repeatable"
                    class="btn border o_quiz_js_quiz_reset">
                    Reset
                </button>
            </div>
        </div>
    </t>

</templates>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.track.quiz</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="website_event.event_event_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//label[@for='community_menu']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>

</data></odoo>


```

## File: views\event_leaderboard_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

<template id="event_leaderboard" name="Leaderboard">
    <t t-call="website_event.layout">
        <div t-if="visitors" class="pt32 pb32 o_wevent_quiz_leaderboard">
            <t t-call="website_event_track_quiz.leaderboard_search_bar"/>
            <div class="container mt32">
                <div t-if="not search" class="row mb-3">
                    <div class="col-md-4 d-flex flex-grow-1" t-foreach="top3_visitors" t-as="visitor">
                        <t t-call="website_event_track_quiz.top3_visitor_card"></t>
                    </div>
                </div>
                <table class="table table-sm">
                    <tr t-foreach="visitors" t-as="visitor"
                        t-attf-class="#{'alert-info' if visitor['visitor'] == current_visitor else ''} #{'o_wevent_quiz_scroll_to' if scroll_to_position and visitor['visitor'] == current_visitor else ''}">
                        <t t-call="website_event_track_quiz.all_visitor_card"/>
                    </tr>
                </table>
                <div class="d-flex justify-content-center">
                    <t t-call="website_event_track_quiz.pager_nobox"/>
                </div>
            </div>
        </div>
        <div t-if="not visitors and search" class="container mt32">
            <t t-call="website_event_track_quiz.leaderboard_search_bar"/>
            <div class='alert alert-warning mt32'>No user found for <strong><t t-out="search"/></strong>. Try another search.</div>
        </div>
        <div t-if="not visitors and not search" class="vh-100 d-flex justify-content-center align-items-center">
            <h4 class="text-muted fw-bold">There is currently no leaderboard available</h4>
        </div>
    </t>
</template>

<template id="pager_nobox" name="Pager (not box display)">
    <ul t-if="pager['page_count'] > 1" t-attf-class="o_wprofile_pager fw-bold pagination m-0">
        <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == 1 else ''}">
            <a t-att-href=" pager['page_first']['url'] if pager['page']['num'] != 1 else None" class="page-link"><i class="fa fa-step-backward"/></a>
        </li>
        <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == 1 else ''}">
            <a t-att-href=" pager['page_previous']['url'] if pager['page']['num'] != 1 else None" class="page-link"><i class="fa fa-caret-left"/></a>
        </li>
        <t t-foreach="pager['pages']" t-as="page">
            <li t-attf-class="page-item #{'active disabled bg-primary rounded-circle' if page['num'] == pager['page']['num'] else ''}"> <a t-att-href="page['url']" class="page-link" t-out="page['num']"/></li>
        </t>
        <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == pager['page_count'] else ''}">
            <a t-att-href="pager['page_next']['url'] if pager['page']['num'] != pager['page_count'] else None" class="page-link"><i class="fa fa-caret-right"/></a>
        </li>
        <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == pager['page_count'] else ''}">
            <a t-att-href=" pager['page_last']['url'] if pager['page']['num'] != pager['page_count'] else None" class="page-link"><i class="fa fa-step-forward"/></a>
        </li>
    </ul>
</template>

<template id="top3_visitor_card" name="Top 3 Visitor Card">
    <div class="card w-100 text-center mb-2 border-bottom-0">
        <div class="card-body">
            <div class="d-inline-block position-relative">
                <img class="rounded-circle img-fluid"
                    style="width: 128px; height: 128px; object-fit: cover;"
                    t-att-src="image_data_uri(visitor['visitor'].partner_image) if visitor['visitor'].partner_image else '/web/static/img/user_placeholder.jpg'"/>
                <img class="position-absolute" t-attf-src="/website_profile/static/src/img/rank_#{visitor['position']}.svg" alt="User rank" style="bottom: 0; right: -10px"/>
            </div>
            <h3 t-if="visitor['visitor'] == current_visitor and not visitor['visitor'].partner_id" class="mt-2 mb-0">You</h3>
            <h3 t-else="" class="mt-2 mb-0" t-out="visitor['visitor'].display_name"/>
        </div>
        <div class="row mx-0 o_wprofile_top3_card_footer text-nowrap">
            <div class="col py-3"><b t-out="visitor['points']"/> <span class="text-muted">Points</span></div>
        </div>
    </div>
</template>

<template id="all_visitor_card" name="All VIsitor Card">
    <td class="align-middle text-end text-muted" style="width: 0">
        <span t-out="visitor['position']"/>
    </td>
    <td class="align-middle d-none d-sm-table-cell">
        <img class="o_object_fit_cover rounded-circle o_wprofile_img_small"
        width="30"
        height="30"
        t-att-src="image_data_uri(visitor['visitor'].partner_image) if visitor['visitor'].partner_image else '/web/static/img/user_placeholder.jpg'"/>
    </td>
    <td class="align-middle w-md-75">
        <span t-if="visitor['visitor'] == current_visitor and not visitor['visitor'].partner_id" class="fw-bold">You</span>
        <span t-else="" class="fw-bold" t-out="visitor['visitor'].display_name"/><br/>
    </td>
    <td class="align-middle fw-bold text-end text-nowrap">
        <b t-out="visitor['points']"/> <span class="text-muted small fw-bold">Points</span>
    </td>
</template>

<!-- Sub nav -->
<template id="leaderboard_search_bar" name="Leaderboard search bar">
    <div class="container">
        <div class="row align-items-center justify-content-between">
            <!-- Desktop Mode -->
            <div class="col d-none d-md-flex flex-row align-items-center justify-content-end">
                <!-- search -->
                <form t-attf-action="#{'/event/%s/community/leaderboard/results' % (slug(event))}" role="search" method="get">
                    <div class="input-group ms-1 position-relative">
                        <button class="btn btn-link text-white rounded-0 pe-1" type="submit" aria-label="Search" title="Search">
                            <i class="fa fa-search"></i>
                        </button>
                        <input type="text" class="form-control rounded-0" name="search" placeholder="Search Attendees" t-att-value="searched_name or ''"/>
                    </div>
                </form>
            </div>

            <!-- Mobile Mode -->
            <div class="col d-md-none py-1 o_wprofile_user_profile_sub_nav_mobile_col">
                <div class="btn-group w-100 position-relative" role="group" aria-label="Mobile sub-nav">

                    <div class="btn-group ms-1 position-static me-2">
                        <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false"><i class="fa fa-search"></i></a>
                        <div class="dropdown-menu dropdown-menu-end w-100" style="right: 10px;">
                            <form class="px-3" t-attf-action="#{'/event/%s/community/leaderboard' % (slug(event))}" role="search" method="get">
                                <div class="input-group">
                                    <input type="text" class="form-control" name="search" placeholder="Search users"/>
                                    <button class="btn btn-primary" type="submit" aria-label="Search" title="Search">
                                        <i class="fa fa-search"/>
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
</odoo>

```

## File: views\event_menus.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <menuitem id="event_quiz_menu"
        name="Quizzes"
        action="event_quiz_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="50"/>
    <menuitem id="event_quiz_question_menu"
        name="Quiz Questions"
        action="event_quiz_question_action"
        parent="event.menu_event_configuration"
        groups="base.group_no_one"
        sequence="55"/>

</data></odoo>

```

## File: views\event_quiz_question_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_quiz_question_view_search" model="ir.ui.view">
        <field name="name">event.quiz.question.view.search</field>
        <field name="model">event.quiz.question</field>
        <field name="arch" type="xml">
            <search string="Quiz Questions">
                <field name="name"/>
                <field name="quiz_id"/>
                <group string="Group By" expand="0">
                    <filter string="Quiz" name="groupby_quiz_id" context="{'group_by': 'quiz_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_quiz_question_view_tree" model="ir.ui.view">
        <field name="name">event.quiz.question.view.tree</field>
        <field name="model">event.quiz.question</field>
        <field name="arch" type="xml">
            <tree string="Quiz Questions">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="quiz_id"/>
                <field name="awarded_points"/>
            </tree>
        </field>
    </record>

    <record id="event_quiz_question_view_tree_from_quiz" model="ir.ui.view">
        <field name="name">event.quiz.question.view.tree.from.quiz</field>
        <field name="model">event.quiz.question</field>
        <field name="inherit_id" ref="website_event_track_quiz.event_quiz_question_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='quiz_id']" position="replace">
            </xpath>
        </field>
    </record>

    <record id="event_quiz_question_view_form" model="ir.ui.view">
        <field name="name">event.quiz.question.view.form</field>
        <field name="model">event.quiz.question</field>
        <field name="arch" type="xml">
            <form string="Quiz Question">
                <sheet>
                    <h1>
                        <field name="name" default_focus="1"
                            placeholder="e.g. What is Joe's favorite motto?"/>
                    </h1>
                    <group>
                        <field name="quiz_id"/>
                        <field name="awarded_points" invisible="1"/>
                    </group>
                    <group name="questions">
                        <field name="answer_ids" nolabel="1" colspan="2">
                            <tree editable="bottom" create="true" delete="true">
                                <field name="sequence" widget="handle"/>
                                <field name="text_value"/>
                                <field name="is_correct"/>
                                <field name="awarded_points"/>
                                <field name="comment"/>
                            </tree>
                        </field>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_quiz_question_view_form_from_quiz" model="ir.ui.view">
        <field name="name">event.quiz.question.view.form.from.quiz</field>
        <field name="model">event.quiz.question</field>
        <field name="inherit_id" ref="website_event_track_quiz.event_quiz_question_view_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='quiz_id']" position="replace">
            </xpath>
        </field>
    </record>

    <record id="event_quiz_question_action" model="ir.actions.act_window">
        <field name="name">Event Quiz Questions</field>
        <field name="res_model">event.quiz.question</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No Quiz Question yet!
            </p><p>
                From here you will be able to examine all quiz questions you have linked to Tracks.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\event_quiz_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="quiz_content" name="Track: Quiz specific content">
        <t t-set="quiz_completed" t-value="quiz_completed or False"/>

        <div class="o_quiz_js_quiz col"
            t-att-data-id="track.id"
            t-att-data-event-id="track.event_id.id"
            t-att-data-completed="1 if quiz_completed else 0"
            t-att-data-quiz-attempts-count="quiz_attempts_count or 0"
            t-att-data-quiz-points-gained="quiz_points"
            t-att-data-is-event-user="is_event_user or 0"
            t-att-data-repeatable="track.quiz_id.repeatable">
            <t t-foreach="track.quiz_id.question_ids" t-as="question">
                <t t-call="website_event_track_quiz.quiz_question"/>
            </t>
            <div class="o_quiz_js_quiz_validation pt-3"/>
        </div>
    </template>

    <template id="quiz_question" name="Quiz question template">
        <div t-att-class="'o_quiz_js_quiz_question mt-3 %s' % ('completed-disabled' if quiz_completed else '')"
            t-att-data-question-id="question['id']" t-att-data-title="question['name']" >
            <div class="row d-flex mb-2 mx-0">
                <div class="h4">
                    <span t-out="question['name']"/>
                </div>
            </div>
            <div class="list-group">
                <t t-foreach="question['answer_ids']" t-as="answer">
                    <a t-att-data-answer-id="answer['id']" href="#"
                        t-att-data-text="answer['text_value']"
                        class="o_quiz_quiz_answer list-group-item list-group-item-action d-flex align-items-center">
                        <label class="my-0 d-flex align-items-center justify-content-center me-2">
                            <input type="radio"
                                t-att-name="question['id']"
                                t-att-value="answer['id']"
                                class="d-none"
                                t-att-disabled="quiz_completed"/>
                            <i t-att-class="'fa fa-circle text-400 %s' % ('d-none' if quiz_completed and answer['is_correct'] else '')"/>
                            <i class="fa fa-times-circle text-danger d-none"></i>
                            <i t-att-class="'fa fa-check-circle text-success %s' % ('d-none' if not (quiz_completed and answer['is_correct']) else '')"></i>
                        </label>
                        <span t-out="answer['text_value']"/>
                    </a>
                </t>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\event_quiz_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_quiz_view_search" model="ir.ui.view">
        <field name="name">event.quiz.view.search</field>
        <field name="model">event.quiz</field>
        <field name="arch" type="xml">
            <search string="Quizzes">
                <field name="name"/>
                <field name="event_track_id"/>
                <field name="event_id"/>
                <group string="Group By" expand="0">
                    <filter string="Track" name="groupby_event_track_id" context="{'group_by': 'event_track_id'}"/>
                    <filter string="Event" name="groupby_event_id" context="{'group_by': 'event_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="event_quiz_view_tree" model="ir.ui.view">
        <field name="name">event.quiz.view.tree</field>
        <field name="model">event.quiz</field>
        <field name="arch" type="xml">
            <tree string="Quizzes">
                <field name="name"/>
                <field name="event_id"/>
                <field name="event_track_id"/>
            </tree>
        </field>
    </record>

    <record id="event_quiz_view_form" model="ir.ui.view">
        <field name="name">event.quiz.view.form</field>
        <field name="model">event.quiz</field>
        <field name="arch" type="xml">
            <form string="Quiz">
                <sheet>
                    <h1>
                        <field name="name" default_focus="1"
                            placeholder="e.g. Test your Knowledge"/>
                    </h1>
                    <group>
                        <group>
                            <field name="repeatable" string="Allow multiple tries"/>
                        </group>
                        <group>
                            <field name="event_id"/>
                            <field name="event_track_id"/>
                        </group>
                    </group>
                    <group name="questions">
                        <field name="question_ids" nolabel="1" colspan="2"
                            context="{
                                'tree_view_ref': 'website_event_track_quiz.event_quiz_question_view_tree_from_quiz',
                                'form_view_ref': 'website_event_track_quiz.event_quiz_question_view_form_from_quiz'
                            }"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_quiz_action" model="ir.actions.act_window">
        <field name="name">Event Quizzes</field>
        <field name="res_model">event.quiz</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
              No Quiz added yet!
            </p><p>
              From here you will be able to overview all quizzes you have linked to Tracks.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\event_track_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_track_content"
    name="Track: Main Description: add quiz"
    inherit_id="website_event_track.event_track_content">
    <xpath expr="//div[hasclass('o_wesession_track_main_description')]" position="after">
        <div id="we_track_quiz_container" t-if="track.quiz_id"
             t-att-class="'o_quiz_js_quiz_container o_quiz_main border-top col-12 p-3 %s' % ('' if track.is_quiz_completed else 'd-none')"
             t-att-data-object-id="track.id">
            <h3 class="col-12">Quiz</h3>
            <t t-call="website_event_track_quiz.quiz_content">
                <t t-set="track" t-value="track"/>
                <t t-set="quiz_completed" t-value="track.is_quiz_completed"/>
                <t t-set="quiz_points" t-value="track.quiz_points"/>
            </t>
        </div>
    </xpath>
    <xpath expr="//div[hasclass('o_we_track_reminder_button')]" position="before">
        <div class="o_we_track_quiz_button me-2 my-1" t-if="track.quiz_id and not track.is_quiz_completed and not track.is_track_upcoming">
            <a class="btn btn-primary" href="#we_track_quiz_container" onclick="$('.o_quiz_js_quiz_container').removeClass('d-none'); ">
                Take the Quiz
            </a>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\event_track_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_track_view_form" model="ir.ui.view" >
        <field name="name">event.track.view.form.inherit.event.track.quiz</field>
        <field name="model">event.track</field>
        <field name="inherit_id" ref="website_event_track.view_event_track_form"/>
        <field name="arch" type="xml">
            <field name='stage_id' position="before">
                <field name="quiz_id" invisible="1"/>
                <button name="action_add_quiz"
                    type="object" class="btn btn-primary" string="Add Quiz"
                    invisible="quiz_id"
                    groups="event.group_event_user">
                </button>
            </field>
            <field name='is_published' position="before">
                <button name="action_view_quiz" type="object" class="oe_stat_button"
                    string="Go to Quiz" icon="fa-question-circle" widget="stat_info"
                    invisible="not quiz_id"
                    groups="event.group_event_user">
                </button>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\event_track_visitor_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>

    <record id="event_track_visitor_view_search" model="ir.ui.view" >
        <field name="name">event.track.visitor.view.search.inherit.quiz</field>
        <field name="model">event.track.visitor</field>
        <field name="inherit_id" ref="website_event_track.event_track_visitor_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_wishlisted']" position="after">
                <field name="quiz_completed"/>
            </xpath>
        </field>
    </record>

    <record id="event_track_visitor_view_form" model="ir.ui.view">
        <field name="name">event.track.visitor.view.form.inherit.quiz</field>
        <field name="model">event.track.visitor</field>
        <field name="inherit_id" ref="website_event_track.event_track_visitor_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_wishlisted']" position="after">
                <field name="quiz_completed"/>
                <field name="quiz_points"/>
            </xpath>
        </field>
    </record>

    <record id="event_track_visitor_view_list" model="ir.ui.view">
        <field name="name">event.track.visitor.view.list.inherit.quiz</field>
        <field name="model">event.track.visitor</field>
        <field name="inherit_id" ref="website_event_track.event_track_visitor_view_list"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='is_wishlisted']" position="after">
                <field name="quiz_completed"/>
                <field name="quiz_points"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.track.quiz</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//span[@name='community_menu']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
        </field>
    </record>
</data></odoo>

```

