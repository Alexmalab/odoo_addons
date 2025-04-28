# Odoo Module: survey

Category: Marketing/Surveys

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Surveys',
    'version': '3.4',
    'category': 'Marketing/Surveys',
    'description': """
Create beautiful surveys and visualize answers
==============================================

It depends on the answers or reviews of some questions by different users. A
survey may have multiple pages. Each page may contain multiple questions and
each question may have multiple answers. Different users may give different
answers of question and according to that survey is done. Partners are also
sent mails with personal token for the invitation of the survey.
    """,
    'summary': 'Create surveys and analyze answers',
    'website': 'https://www.odoo.com/app/surveys',
    'depends': [
        'auth_signup',
        'http_routing',
        'mail',
        'web_tour',
        'gamification'],
    'data': [
        'views/survey_report_templates.xml',
        'views/survey_reports.xml',
        'data/mail_template_data.xml',
        'data/ir_actions_data.xml',
        'security/survey_security.xml',
        'security/ir.model.access.csv',
        'views/survey_menus.xml',
        'views/survey_survey_views.xml',
        'views/survey_user_views.xml',
        'views/survey_question_views.xml',
        'views/survey_templates.xml',
        'views/survey_templates_management.xml',
        'views/survey_templates_print.xml',
        'views/survey_templates_statistics.xml',
        'views/survey_templates_user_input_session.xml',
        'views/gamification_badge_views.xml',
        'wizard/survey_invite_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/gamification_badge_demo.xml',
        'data/res_users_demo.xml',
        'data/survey_demo_feedback.xml',
        'data/survey_demo_feedback_user_input.xml',
        'data/survey_demo_feedback_user_input_line.xml',
        'data/survey_demo_certification.xml',
        'data/survey_demo_certification_user_input.xml',
        'data/survey_demo_certification_user_input_line.xml',
        'data/survey_demo_quiz.xml',
        'data/survey_demo_quiz_user_input.xml',
        'data/survey_demo_quiz_user_input_line.xml',
        'data/survey_demo_conditional.xml',
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'sequence': 220,
    'assets': {
        'survey.survey_assets': [
            'web/static/lib/Chart/Chart.js',
            'web/static/src/legacy/js/fields/field_utils.js',
            'survey/static/src/js/survey_quick_access.js',
            'survey/static/src/js/survey_timer.js',
            'survey/static/src/js/survey_breadcrumb.js',
            'survey/static/src/js/survey_form.js',
            'survey/static/src/js/survey_print.js',
            'survey/static/src/js/survey_result.js',
            ('include', 'web._assets_helpers'),
            ('include', 'web._assets_frontend_helpers'),
            'web/static/lib/bootstrap/scss/_variables.scss',
            'survey/static/src/css/survey_templates_print.css',
            'survey/static/src/css/survey_templates_result.css',
            'survey/static/src/scss/survey_templates_form.scss',
        ],
        'survey.survey_user_input_session_assets': [
            'survey/static/src/js/libs/chartjs-plugin-datalabels.min.js',
            'survey/static/src/js/survey_session_colors.js',
            'survey/static/src/js/survey_session_chart.js',
            'survey/static/src/js/survey_session_text_answers.js',
            'survey/static/src/js/survey_session_leaderboard.js',
            'survey/static/src/js/survey_session_manage.js',
        ],
        'web.report_assets_common': [
            'survey/static/src/scss/survey_reports.scss',
        ],
        'web.assets_backend': [
            'survey/static/src/js/fields_section_one2many.js',
            'survey/static/src/js/fields_form_page_description.js',
            'survey/static/src/scss/survey_survey_views.scss',
        ],
        'web.assets_tests': [
            'survey/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import werkzeug

from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta

from odoo import fields, http, SUPERUSER_ID, _
from odoo.addons.base.models.ir_ui_view import keep_query
from odoo.exceptions import UserError
from odoo.http import request, content_disposition
from odoo.osv import expression
from odoo.tools import format_datetime, format_date, is_html_empty

_logger = logging.getLogger(__name__)


class Survey(http.Controller):

    # ------------------------------------------------------------
    # ACCESS
    # ------------------------------------------------------------

    def _fetch_from_access_token(self, survey_token, answer_token):
        """ Check that given token matches an answer from the given survey_id.
        Returns a sudo-ed browse record of survey in order to avoid access rights
        issues now that access is granted through token. """
        survey_sudo = request.env['survey.survey'].with_context(active_test=False).sudo().search([('access_token', '=', survey_token)])
        if not answer_token:
            answer_sudo = request.env['survey.user_input'].sudo()
        else:
            answer_sudo = request.env['survey.user_input'].sudo().search([
                ('survey_id', '=', survey_sudo.id),
                ('access_token', '=', answer_token)
            ], limit=1)
        return survey_sudo, answer_sudo

    def _check_validity(self, survey_token, answer_token, ensure_token=True, check_partner=True):
        """ Check survey is open and can be taken. This does not checks for
        security rules, only functional / business rules. It returns a string key
        allowing further manipulation of validity issues

         * survey_wrong: survey does not exist;
         * survey_auth: authentication is required;
         * survey_closed: survey is closed and does not accept input anymore;
         * survey_void: survey is void and should not be taken;
         * token_wrong: given token not recognized;
         * token_required: no token given although it is necessary to access the
           survey;
         * answer_deadline: token linked to an expired answer;

        :param ensure_token: whether user input existence based on given access token
          should be enforced or not, depending on the route requesting a token or
          allowing external world calls;

        :param check_partner: Whether we must check that the partner associated to the target
          answer corresponds to the active user.
        """
        survey_sudo, answer_sudo = self._fetch_from_access_token(survey_token, answer_token)

        if not survey_sudo.exists():
            return 'survey_wrong'

        if answer_token and not answer_sudo:
            return 'token_wrong'

        if not answer_sudo and ensure_token:
            return 'token_required'
        if not answer_sudo and survey_sudo.access_mode == 'token':
            return 'token_required'

        if survey_sudo.users_login_required and request.env.user._is_public():
            return 'survey_auth'

        if not survey_sudo.active and (not answer_sudo or not answer_sudo.test_entry):
            return 'survey_closed'

        if (not survey_sudo.page_ids and survey_sudo.questions_layout == 'page_per_section') or not survey_sudo.question_ids:
            return 'survey_void'

        if answer_sudo and check_partner:
            if request.env.user._is_public() and answer_sudo.partner_id and not answer_token:
                # answers from public user should not have any partner_id; this indicates probably a cookie issue
                return 'answer_wrong_user'
            if not request.env.user._is_public() and answer_sudo.partner_id != request.env.user.partner_id:
                # partner mismatch, probably a cookie issue
                return 'answer_wrong_user'

        if answer_sudo and answer_sudo.deadline and answer_sudo.deadline < datetime.now():
            return 'answer_deadline'

        return True

    def _get_access_data(self, survey_token, answer_token, ensure_token=True, check_partner=True):
        """ Get back data related to survey and user input, given the ID and access
        token provided by the route.

         : param ensure_token: whether user input existence should be enforced or not(see ``_check_validity``)
         : param check_partner: whether the partner of the target answer should be checked (see ``_check_validity``)
        """
        survey_sudo, answer_sudo = request.env['survey.survey'].sudo(), request.env['survey.user_input'].sudo()
        has_survey_access, can_answer = False, False

        validity_code = self._check_validity(survey_token, answer_token, ensure_token=ensure_token, check_partner=check_partner)
        if validity_code != 'survey_wrong':
            survey_sudo, answer_sudo = self._fetch_from_access_token(survey_token, answer_token)
            try:
                survey_user = survey_sudo.with_user(request.env.user)
                survey_user.check_access_rights('read', raise_exception=True)
                survey_user.check_access_rule('read')
            except:
                pass
            else:
                has_survey_access = True
            can_answer = bool(answer_sudo)
            if not can_answer:
                can_answer = survey_sudo.access_mode == 'public'

        return {
            'survey_sudo': survey_sudo,
            'answer_sudo': answer_sudo,
            'has_survey_access': has_survey_access,
            'can_answer': can_answer,
            'validity_code': validity_code,
        }

    def _redirect_with_error(self, access_data, error_key):
        survey_sudo = access_data['survey_sudo']
        answer_sudo = access_data['answer_sudo']

        if error_key == 'survey_void' and access_data['can_answer']:
            return request.render("survey.survey_void_content", {'survey': survey_sudo, 'answer': answer_sudo})
        elif error_key == 'survey_closed' and access_data['can_answer']:
            return request.render("survey.survey_closed_expired", {'survey': survey_sudo})
        elif error_key == 'survey_auth':
            if not answer_sudo:  # survey is not even started
                redirect_url = '/web/login?redirect=/survey/start/%s' % survey_sudo.access_token
            elif answer_sudo.access_token:  # survey is started but user is not logged in anymore.
                if answer_sudo.partner_id and (answer_sudo.partner_id.user_ids or survey_sudo.users_can_signup):
                    if answer_sudo.partner_id.user_ids:
                        answer_sudo.partner_id.signup_cancel()
                    else:
                        answer_sudo.partner_id.signup_prepare(expiration=fields.Datetime.now() + relativedelta(days=1))
                    redirect_url = answer_sudo.partner_id._get_signup_url_for_action(url='/survey/start/%s?answer_token=%s' % (survey_sudo.access_token, answer_sudo.access_token))[answer_sudo.partner_id.id]
                else:
                    redirect_url = '/web/login?redirect=%s' % ('/survey/start/%s?answer_token=%s' % (survey_sudo.access_token, answer_sudo.access_token))
            return request.render("survey.survey_auth_required", {'survey': survey_sudo, 'redirect_url': redirect_url})
        elif error_key == 'answer_deadline' and answer_sudo.access_token:
            return request.render("survey.survey_closed_expired", {'survey': survey_sudo})

        return request.redirect("/")

    # ------------------------------------------------------------
    # TEST / RETRY SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/survey/test/<string:survey_token>', type='http', auth='user', website=True)
    def survey_test(self, survey_token, **kwargs):
        """ Test mode for surveys: create a test answer, only for managers or officers
        testing their surveys """
        survey_sudo, dummy = self._fetch_from_access_token(survey_token, False)
        try:
            answer_sudo = survey_sudo._create_answer(user=request.env.user, test_entry=True)
        except:
            return request.redirect('/')
        return request.redirect('/survey/start/%s?%s' % (survey_sudo.access_token, keep_query('*', answer_token=answer_sudo.access_token)))

    @http.route('/survey/retry/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_retry(self, survey_token, answer_token, **post):
        """ This route is called whenever the user has attempts left and hits the 'Retry' button
        after failing the survey."""
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        if not answer_sudo:
            # attempts to 'retry' without having tried first
            return request.redirect("/")

        try:
            retry_answer_sudo = survey_sudo._create_answer(
                user=request.env.user,
                partner=answer_sudo.partner_id,
                email=answer_sudo.email,
                invite_token=answer_sudo.invite_token,
                test_entry=answer_sudo.test_entry,
                **self._prepare_retry_additional_values(answer_sudo)
            )
        except:
            return request.redirect("/")
        return request.redirect('/survey/start/%s?%s' % (survey_sudo.access_token, keep_query('*', answer_token=retry_answer_sudo.access_token)))

    def _prepare_retry_additional_values(self, answer):
        return {
            'deadline': answer.deadline,
        }

    def _prepare_survey_finished_values(self, survey, answer, token=False):
        values = {'survey': survey, 'answer': answer}
        if token:
            values['token'] = token
        if survey.scoring_type != 'no_scoring' and survey.certification:
            values['graph_data'] = json.dumps(answer._prepare_statistics()[answer])
        return values

    # ------------------------------------------------------------
    # TAKING SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/survey/start/<string:survey_token>', type='http', auth='public', website=True)
    def survey_start(self, survey_token, answer_token=None, email=False, **post):
        """ Start a survey by providing
         * a token linked to a survey;
         * a token linked to an answer or generate a new token if access is allowed;
        """
        # Get the current answer token from cookie
        answer_from_cookie = False
        if not answer_token:
            answer_token = request.httprequest.cookies.get('survey_%s' % survey_token)
            answer_from_cookie = bool(answer_token)

        access_data = self._get_access_data(survey_token, answer_token, ensure_token=False)

        if answer_from_cookie and access_data['validity_code'] in ('answer_wrong_user', 'token_wrong'):
            # If the cookie had been generated for another user or does not correspond to any existing answer object
            # (probably because it has been deleted), ignore it and redo the check.
            # The cookie will be replaced by a legit value when resolving the URL, so we don't clean it further here.
            access_data = self._get_access_data(survey_token, None, ensure_token=False)

        if access_data['validity_code'] is not True:
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        if not answer_sudo:
            try:
                answer_sudo = survey_sudo._create_answer(user=request.env.user, email=email)
            except UserError:
                answer_sudo = False

        if not answer_sudo:
            try:
                survey_sudo.with_user(request.env.user).check_access_rights('read')
                survey_sudo.with_user(request.env.user).check_access_rule('read')
            except:
                return request.redirect("/")
            else:
                return request.render("survey.survey_403_page", {'survey': survey_sudo})

        return request.redirect('/survey/%s/%s' % (survey_sudo.access_token, answer_sudo.access_token))

    def _prepare_survey_data(self, survey_sudo, answer_sudo, **post):
        """ This method prepares all the data needed for template rendering, in function of the survey user input state.
            :param post:
                - previous_page_id : come from the breadcrumb or the back button and force the next questions to load
                                     to be the previous ones. """
        data = {
            'is_html_empty': is_html_empty,
            'survey': survey_sudo,
            'answer': answer_sudo,
            'breadcrumb_pages': [{
                'id': page.id,
                'title': page.title,
            } for page in survey_sudo.page_ids],
            'format_datetime': lambda dt: format_datetime(request.env, dt, dt_format=False),
            'format_date': lambda date: format_date(request.env, date)
        }
        if survey_sudo.questions_layout != 'page_per_question':
            triggering_answer_by_question, triggered_questions_by_answer, selected_answers = answer_sudo._get_conditional_values()
            data.update({
                'triggering_answer_by_question': {
                    question.id: triggering_answer_by_question[question].id for question in triggering_answer_by_question.keys()
                    if triggering_answer_by_question[question]
                },
                'triggered_questions_by_answer': {
                    answer.id: triggered_questions_by_answer[answer].ids
                    for answer in triggered_questions_by_answer.keys()
                },
                'selected_answers': selected_answers.ids
            })

        if not answer_sudo.is_session_answer and survey_sudo.is_time_limited and answer_sudo.start_datetime:
            data.update({
                'server_time': fields.Datetime.now(),
                'timer_start': answer_sudo.start_datetime.isoformat(),
                'time_limit_minutes': survey_sudo.time_limit
            })

        page_or_question_key = 'question' if survey_sudo.questions_layout == 'page_per_question' else 'page'

        # Bypass all if page_id is specified (comes from breadcrumb or previous button)
        if 'previous_page_id' in post:
            previous_page_or_question_id = int(post['previous_page_id'])
            new_previous_id = survey_sudo._get_next_page_or_question(answer_sudo, previous_page_or_question_id, go_back=True).id
            page_or_question = request.env['survey.question'].sudo().browse(previous_page_or_question_id)
            data.update({
                page_or_question_key: page_or_question,
                'previous_page_id': new_previous_id,
                'has_answered': answer_sudo.user_input_line_ids.filtered(lambda line: line.question_id.id == new_previous_id),
                'can_go_back': survey_sudo._can_go_back(answer_sudo, page_or_question),
            })
            return data

        if answer_sudo.state == 'in_progress':
            if answer_sudo.is_session_answer:
                next_page_or_question = survey_sudo.session_question_id
            else:
                next_page_or_question = survey_sudo._get_next_page_or_question(
                    answer_sudo,
                    answer_sudo.last_displayed_page_id.id if answer_sudo.last_displayed_page_id else 0)

                if next_page_or_question:
                    data.update({
                        'survey_last': survey_sudo._is_last_page_or_question(answer_sudo, next_page_or_question)
                    })

            if answer_sudo.is_session_answer and next_page_or_question.is_time_limited:
                data.update({
                    'timer_start': survey_sudo.session_question_start_time.isoformat(),
                    'time_limit_minutes': next_page_or_question.time_limit / 60
                })

            data.update({
                page_or_question_key: next_page_or_question,
                'has_answered': answer_sudo.user_input_line_ids.filtered(lambda line: line.question_id == next_page_or_question),
                'can_go_back': survey_sudo._can_go_back(answer_sudo, next_page_or_question),
            })
            if survey_sudo.questions_layout != 'one_page':
                data.update({
                    'previous_page_id': survey_sudo._get_next_page_or_question(answer_sudo, next_page_or_question.id, go_back=True).id
                })
        elif answer_sudo.state == 'done' or answer_sudo.survey_time_limit_reached:
            # Display success message
            return self._prepare_survey_finished_values(survey_sudo, answer_sudo)

        return data

    def _prepare_question_html(self, survey_sudo, answer_sudo, **post):
        """ Survey page navigation is done in AJAX. This function prepare the 'next page' to display in html
        and send back this html to the survey_form widget that will inject it into the page."""
        survey_data = self._prepare_survey_data(survey_sudo, answer_sudo, **post)

        survey_content = False
        if answer_sudo.state == 'done':
            survey_content = request.env.ref('survey.survey_fill_form_done')._render(survey_data)
        else:
            survey_content = request.env.ref('survey.survey_fill_form_in_progress')._render(survey_data)

        survey_progress = False
        if answer_sudo.state == 'in_progress' and not survey_data.get('question', request.env['survey.question']).is_page:
            if survey_sudo.questions_layout == 'page_per_section':
                page_ids = survey_sudo.page_ids.ids
                survey_progress = request.env.ref('survey.survey_progression')._render({
                    'survey': survey_sudo,
                    'page_ids': page_ids,
                    'page_number': page_ids.index(survey_data['page'].id) + (1 if survey_sudo.progression_mode == 'number' else 0)
                })
            elif survey_sudo.questions_layout == 'page_per_question':
                page_ids = (answer_sudo.predefined_question_ids.ids
                            if not answer_sudo.is_session_answer and survey_sudo.questions_selection == 'random'
                            else survey_sudo.question_ids.ids)
                survey_progress = request.env.ref('survey.survey_progression')._render({
                    'survey': survey_sudo,
                    'page_ids': page_ids,
                    'page_number': page_ids.index(survey_data['question'].id)
                })

        return {
            'survey_content': survey_content,
            'survey_progress': survey_progress,
            'survey_navigation': request.env.ref('survey.survey_navigation')._render(survey_data),
        }

    @http.route('/survey/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_display_page(self, survey_token, answer_token, **post):
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return self._redirect_with_error(access_data, access_data['validity_code'])

        answer_sudo = access_data['answer_sudo']
        if answer_sudo.state != 'done' and answer_sudo.survey_time_limit_reached:
            answer_sudo._mark_done()

        return request.render('survey.survey_page_fill',
            self._prepare_survey_data(access_data['survey_sudo'], answer_sudo, **post))

    @http.route('/survey/get_background_image/<string:survey_token>/<string:answer_token>', type='http', auth="public", website=True, sitemap=False)
    def survey_get_background(self, survey_token, answer_token):
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return werkzeug.exceptions.Forbidden()

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        status, headers, image_base64 = request.env['ir.http'].sudo().binary_content(
            model='survey.survey', id=survey_sudo.id, field='background_image',
            default_mimetype='image/png')

        return request.env['ir.http']._content_image_get_response(status, headers, image_base64)

    @http.route('/survey/get_question_image/<string:survey_token>/<string:answer_token>/<int:question_id>/<int:suggested_answer_id>', type='http', auth="public", website=True, sitemap=False)
    def survey_get_question_image(self, survey_token, answer_token, question_id, suggested_answer_id):
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return werkzeug.exceptions.Forbidden()

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        if not survey_sudo.question_ids.filtered(lambda q: q.id == question_id)\
                          .suggested_answer_ids.filtered(lambda a: a.id == suggested_answer_id):
            return werkzeug.exceptions.NotFound()

        status, headers, image_base64 = request.env['ir.http'].sudo().binary_content(
            model='survey.question.answer', id=suggested_answer_id, field='value_image',
            default_mimetype='image/png')

        return request.env['ir.http']._content_image_get_response(status, headers, image_base64)

    # ----------------------------------------------------------------
    # JSON ROUTES to begin / continue survey (ajax navigation) + Tools
    # ----------------------------------------------------------------

    @http.route('/survey/begin/<string:survey_token>/<string:answer_token>', type='json', auth='public', website=True)
    def survey_begin(self, survey_token, answer_token, **post):
        """ Route used to start the survey user input and display the first survey page. """
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return {'error': access_data['validity_code']}
        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        if answer_sudo.state != "new":
            return {'error': _("The survey has already started.")}

        answer_sudo._mark_in_progress()
        return self._prepare_question_html(survey_sudo, answer_sudo, **post)

    @http.route('/survey/next_question/<string:survey_token>/<string:answer_token>', type='json', auth='public', website=True)
    def survey_next_question(self, survey_token, answer_token, **post):
        """ Method used to display the next survey question in an ongoing session.
        Triggered on all attendees screens when the host goes to the next question. """
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return {'error': access_data['validity_code']}
        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        if answer_sudo.state == 'new' and answer_sudo.is_session_answer:
            answer_sudo._mark_in_progress()

        return self._prepare_question_html(survey_sudo, answer_sudo, **post)

    @http.route('/survey/submit/<string:survey_token>/<string:answer_token>', type='json', auth='public', website=True)
    def survey_submit(self, survey_token, answer_token, **post):
        """ Submit a page from the survey.
        This will take into account the validation errors and store the answers to the questions.
        If the time limit is reached, errors will be skipped, answers will be ignored and
        survey state will be forced to 'done'"""
        # Survey Validation
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return {'error': access_data['validity_code']}
        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        if answer_sudo.state == 'done':
            return {'error': 'unauthorized'}

        questions, page_or_question_id = survey_sudo._get_survey_questions(answer=answer_sudo,
                                                                           page_id=post.get('page_id'),
                                                                           question_id=post.get('question_id'))

        if not answer_sudo.test_entry and not survey_sudo._has_attempts_left(answer_sudo.partner_id, answer_sudo.email, answer_sudo.invite_token):
            # prevent cheating with users creating multiple 'user_input' before their last attempt
            return {'error': 'unauthorized'}

        if answer_sudo.survey_time_limit_reached or answer_sudo.question_time_limit_reached:
            if answer_sudo.question_time_limit_reached:
                time_limit = survey_sudo.session_question_start_time + relativedelta(
                    seconds=survey_sudo.session_question_id.time_limit
                )
                time_limit += timedelta(seconds=3)
            else:
                time_limit = answer_sudo.start_datetime + timedelta(minutes=survey_sudo.time_limit)
                time_limit += timedelta(seconds=10)
            if fields.Datetime.now() > time_limit:
                # prevent cheating with users blocking the JS timer and taking all their time to answer
                return {'error': 'unauthorized'}

        errors = {}
        # Prepare answers / comment by question, validate and save answers
        for question in questions:
            inactive_questions = request.env['survey.question'] if answer_sudo.is_session_answer else answer_sudo._get_inactive_conditional_questions()
            if question in inactive_questions:  # if question is inactive, skip validation and save
                continue
            answer, comment = self._extract_comment_from_answers(question, post.get(str(question.id)))
            errors.update(question.validate_question(answer, comment))
            if not errors.get(question.id):
                answer_sudo.save_lines(question, answer, comment)

        if errors and not (answer_sudo.survey_time_limit_reached or answer_sudo.question_time_limit_reached):
            return {'error': 'validation', 'fields': errors}

        if not answer_sudo.is_session_answer:
            answer_sudo._clear_inactive_conditional_answers()

        if answer_sudo.survey_time_limit_reached or survey_sudo.questions_layout == 'one_page':
            answer_sudo._mark_done()
        elif 'previous_page_id' in post:
            # Go back to specific page using the breadcrumb. Lines are saved and survey continues
            return self._prepare_question_html(survey_sudo, answer_sudo, **post)
        else:
            vals = {'last_displayed_page_id': page_or_question_id}
            if not answer_sudo.is_session_answer:
                next_page = survey_sudo._get_next_page_or_question(answer_sudo, page_or_question_id)
                if not next_page:
                    answer_sudo._mark_done()

            answer_sudo.write(vals)

        return self._prepare_question_html(survey_sudo, answer_sudo)

    def _extract_comment_from_answers(self, question, answers):
        """ Answers is a custom structure depending of the question type
        that can contain question answers but also comments that need to be
        extracted before validating and saving answers.
        If multiple answers, they are listed in an array, except for matrix
        where answers are structured differently. See input and output for
        more info on data structures.
        :param question: survey.question
        :param answers:
          * question_type: free_text, text_box, numerical_box, date, datetime
            answers is a string containing the value
          * question_type: simple_choice with no comment
            answers is a string containing the value ('question_id_1')
          * question_type: simple_choice with comment
            ['question_id_1', {'comment': str}]
          * question_type: multiple choice
            ['question_id_1', 'question_id_2'] + [{'comment': str}] if holds a comment
          * question_type: matrix
            {'matrix_row_id_1': ['question_id_1', 'question_id_2'],
             'matrix_row_id_2': ['question_id_1', 'question_id_2']
            } + {'comment': str} if holds a comment
        :return: tuple(
          same structure without comment,
          extracted comment for given question
        ) """
        comment = None
        answers_no_comment = []
        if answers:
            if question.question_type == 'matrix':
                if 'comment' in answers:
                    comment = answers['comment'].strip()
                    answers.pop('comment')
                answers_no_comment = answers
            else:
                if not isinstance(answers, list):
                    answers = [answers]
                for answer in answers:
                    if isinstance(answer, dict) and 'comment' in answer:
                        comment = answer['comment'].strip()
                    else:
                        answers_no_comment.append(answer)
                if len(answers_no_comment) == 1:
                    answers_no_comment = answers_no_comment[0]
        return answers_no_comment, comment

    # ------------------------------------------------------------
    # COMPLETED SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/survey/print/<string:survey_token>', type='http', auth='public', website=True, sitemap=False)
    def survey_print(self, survey_token, review=False, answer_token=None, **post):
        '''Display an survey in printable view; if <answer_token> is set, it will
        grab the answers of the user_input_id that has <answer_token>.'''
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=False, check_partner=False)
        if access_data['validity_code'] is not True and (
                access_data['has_survey_access'] or
                access_data['validity_code'] not in ['token_required', 'survey_closed', 'survey_void', 'answer_deadline']):
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        return request.render('survey.survey_page_print', {
            'is_html_empty': is_html_empty,
            'review': review,
            'survey': survey_sudo,
            'answer': answer_sudo if survey_sudo.scoring_type != 'scoring_without_answers' else answer_sudo.browse(),
            'questions_to_display': answer_sudo._get_print_questions(),
            'scoring_display_correction': survey_sudo.scoring_type == 'scoring_with_answers' and answer_sudo,
            'format_datetime': lambda dt: format_datetime(request.env, dt, dt_format=False),
            'format_date': lambda date: format_date(request.env, date),
        })

    @http.route('/survey/<model("survey.survey"):survey>/certification_preview', type="http", auth="user", website=True)
    def show_certification_pdf(self, survey, **kwargs):
        preview_url = '/survey/%s/get_certification_preview' % survey.id
        return request.render('survey.certification_preview', {
            'preview_url': preview_url,
            'page_title': survey.title,
        })

    @http.route(['/survey/<model("survey.survey"):survey>/get_certification_preview'], type="http", auth="user", methods=['GET'], website=True)
    def survey_get_certification_preview(self, survey, **kwargs):
        if not request.env.user.has_group('survey.group_survey_user'):
            raise werkzeug.exceptions.Forbidden()

        fake_user_input = survey._create_answer(user=request.env.user, test_entry=True)
        response = self._generate_report(fake_user_input, download=False)
        fake_user_input.sudo().unlink()
        return response

    @http.route(['/survey/<int:survey_id>/get_certification'], type='http', auth='user', methods=['GET'], website=True)
    def survey_get_certification(self, survey_id, **kwargs):
        """ The certification document can be downloaded as long as the user has succeeded the certification """
        survey = request.env['survey.survey'].sudo().search([
            ('id', '=', survey_id),
            ('certification', '=', True)
        ])

        if not survey:
            # no certification found
            return request.redirect("/")

        succeeded_attempt = request.env['survey.user_input'].sudo().search([
            ('partner_id', '=', request.env.user.partner_id.id),
            ('survey_id', '=', survey_id),
            ('scoring_success', '=', True)
        ], limit=1)

        if not succeeded_attempt:
            raise UserError(_("The user has not succeeded the certification"))

        return self._generate_report(succeeded_attempt, download=True)

    # ------------------------------------------------------------
    # REPORTING SURVEY ROUTES AND TOOLS
    # ------------------------------------------------------------

    @http.route('/survey/results/<model("survey.survey"):survey>', type='http', auth='user', website=True)
    def survey_report(self, survey, answer_token=None, **post):
        """ Display survey Results & Statistics for given survey.

        New structure: {
            'survey': current survey browse record,
            'question_and_page_data': see ``SurveyQuestion._prepare_statistics()``,
            'survey_data'= see ``SurveySurvey._prepare_statistics()``
            'search_filters': [],
            'search_finished': either filter on finished inputs only or not,
        }
        """
        user_input_lines, search_filters = self._extract_filters_data(survey, post)
        survey_data = survey._prepare_statistics(user_input_lines)
        question_and_page_data = survey.question_and_page_ids._prepare_statistics(user_input_lines)

        template_values = {
            # survey and its statistics
            'survey': survey,
            'question_and_page_data': question_and_page_data,
            'survey_data': survey_data,
            # search
            'search_filters': search_filters,
            'search_finished': post.get('finished') == 'true',
        }

        if survey.session_show_leaderboard:
            template_values['leaderboard'] = survey._prepare_leaderboard_values()

        return request.render('survey.survey_page_statistics', template_values)

    def _generate_report(self, user_input, download=True):
        report = request.env.ref('survey.certification_report').with_user(SUPERUSER_ID)._render_qweb_pdf([user_input.id], data={'report_type': 'pdf'})[0]

        report_content_disposition = content_disposition('Certification.pdf')
        if not download:
            content_split = report_content_disposition.split(';')
            content_split[0] = 'inline'
            report_content_disposition = ';'.join(content_split)

        return request.make_response(report, headers=[
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(report)),
            ('Content-Disposition', report_content_disposition),
        ])

    def _get_user_input_domain(self, survey, line_filter_domain, **post):
        user_input_domain = ['&', ('test_entry', '=', False), ('survey_id', '=', survey.id)]
        if line_filter_domain:
            matching_line_ids = request.env['survey.user_input.line'].sudo().search(line_filter_domain).ids
            user_input_domain = expression.AND([
                [('user_input_line_ids', 'in', matching_line_ids)],
                user_input_domain
            ])
        if post.get('finished'):
            user_input_domain = expression.AND([[('state', '=', 'done')], user_input_domain])
        else:
            user_input_domain = expression.AND([[('state', '!=', 'new')], user_input_domain])
        return user_input_domain

    def _extract_filters_data(self, survey, post):
        search_filters = []
        line_filter_domain, line_choices = [], []
        for data in post.get('filters', '').split('|'):
            try:
                row_id, answer_id = (int(item) for item in data.split(','))
            except:
                pass
            else:
                if row_id and answer_id:
                    line_filter_domain = expression.AND([
                        ['&', ('matrix_row_id', '=', row_id), ('suggested_answer_id', '=', answer_id)],
                        line_filter_domain
                    ])
                    answers = request.env['survey.question.answer'].browse([row_id, answer_id])
                elif answer_id:
                    line_choices.append(answer_id)
                    answers = request.env['survey.question.answer'].browse([answer_id])
                if answer_id:
                    question_id = answers[0].matrix_question_id or answers[0].question_id
                    search_filters.append({
                        'question': question_id.title,
                        'answers': '%s%s' % (answers[0].value, ': %s' % answers[1].value if len(answers) > 1 else '')
                    })
        if line_choices:
            line_filter_domain = expression.AND([[('suggested_answer_id', 'in', line_choices)], line_filter_domain])

        user_input_domain = self._get_user_input_domain(survey, line_filter_domain, **post)
        user_input_lines = request.env['survey.user_input'].sudo().search(user_input_domain).mapped('user_input_line_ids')

        return user_input_lines, search_filters

```

## File: controllers\survey_session_manage.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import json

from dateutil.relativedelta import relativedelta
from werkzeug.exceptions import NotFound

from odoo import fields, http
from odoo.http import request
from odoo.tools import is_html_empty


class UserInputSession(http.Controller):
    def _fetch_from_token(self, survey_token):
        """ Check that given survey_token matches a survey 'access_token'.
        Unlike the regular survey controller, user trying to access the survey must have full access rights! """
        return request.env['survey.survey'].search([('access_token', '=', survey_token)])

    def _fetch_from_session_code(self, session_code):
        """ Matches a survey against a passed session_code.
        We don't limit the session_state to be reachable (ready / in_progress) here because
        in some cases, we want closed session as well (where session_state = False).
        Instead, when necessary, the reachability is forced in routes calling this method to
        avoid people using those routes to access other (private) surveys.
        We limit to sessions opened within the last 7 days to avoid potential abuses. """
        if session_code:
            matching_survey = request.env['survey.survey'].sudo().search([
                ('session_start_time', '>', fields.Datetime.now() - relativedelta(days=7)),
                ('session_code', '=', session_code),
            ], limit=1)
            if matching_survey:
                return matching_survey

        return False

    # ------------------------------------------------------------
    # SURVEY SESSION MANAGEMENT
    # ------------------------------------------------------------

    @http.route('/survey/session/manage/<string:survey_token>', type='http', auth='user', website=True)
    def survey_session_manage(self, survey_token, **kwargs):
        """ Main route used by the host to 'manager' the session.
        - If the state of the session is 'ready'
          We render a template allowing the host to showcase the different options of the session
          and to actually start the session.
          If there are no questions, a "void content" is displayed instead to avoid displaying a
          blank survey.
        - If the state of the session is 'in_progress'
          We render a template allowing the host to show the question results, display the attendees
          leaderboard or go to the next question of the session. """

        survey = self._fetch_from_token(survey_token)

        if not survey or not survey.session_state:
            # no open session
            return NotFound()

        if survey.session_state == 'ready':
            if not survey.question_ids:
                return request.render('survey.survey_void_content', {
                    'survey': survey,
                    'answer': request.env['survey.user_input'],
                })
            return request.render('survey.user_input_session_open', {
                'survey': survey
            })
        else:
            template_values = self._prepare_manage_session_values(survey)
            return request.render('survey.user_input_session_manage', template_values)

    @http.route('/survey/session/next_question/<string:survey_token>', type='json', auth='user', website=True)
    def survey_session_next_question(self, survey_token, go_back=False, **kwargs):
        """ This route is called when the host goes to the next question of the session.

        It's not a regular 'request.render' route because we handle the transition between
        questions using a AJAX call to be able to display a bioutiful fade in/out effect.

        It triggers the next question of the session.

        We artificially add 1 second to the 'current_question_start_time' to account for server delay.
        As the timing can influence the attendees score, we try to be fair with everyone by giving them
        an extra second before we start counting down.

        Frontend should take the delay into account by displaying the appropriate animations.

        Writing the next question on the survey is sudo'ed to avoid potential access right issues.
        e.g: a survey user can create a live session from any survey but he can only write
        on its own survey. """

        survey = self._fetch_from_token(survey_token)

        if not survey or not survey.session_state:
            # no open session
            return ''

        if survey.session_state == 'ready':
            survey._session_open()

        next_question = survey._get_session_next_question(go_back)

        # using datetime.datetime because we want the millis portion
        if next_question:
            now = datetime.datetime.now()
            survey.sudo().write({
                'session_question_id': next_question.id,
                'session_question_start_time': fields.Datetime.now() + relativedelta(seconds=1)
            })
            request.env['bus.bus']._sendone(survey.access_token, 'next_question', {'question_start': now.timestamp()})

            template_values = self._prepare_manage_session_values(survey)
            template_values['is_rpc_call'] = True
            return request.env.ref('survey.user_input_session_manage_content')._render(template_values)
        else:
            return False

    @http.route('/survey/session/results/<string:survey_token>', type='json', auth='user', website=True)
    def survey_session_results(self, survey_token, **kwargs):
        """ This route is called when the host shows the current question's results.

        It's not a regular 'request.render' route because we handle the display of results using
        an AJAX request to be able to include the results in the currently displayed page. """

        survey = self._fetch_from_token(survey_token)

        if not survey or survey.session_state != 'in_progress':
            # no open session
            return False

        user_input_lines = request.env['survey.user_input.line'].search([
            ('survey_id', '=', survey.id),
            ('question_id', '=', survey.session_question_id.id),
            ('create_date', '>=', survey.session_start_time)
        ])

        return self._prepare_question_results_values(survey, user_input_lines)

    @http.route('/survey/session/leaderboard/<string:survey_token>', type='json', auth='user', website=True)
    def survey_session_leaderboard(self, survey_token, **kwargs):
        """ This route is called when the host shows the current question's attendees leaderboard.

        It's not a regular 'request.render' route because we handle the display of the leaderboard
        using an AJAX request to be able to include the results in the currently displayed page. """

        survey = self._fetch_from_token(survey_token)

        if not survey or survey.session_state != 'in_progress':
            # no open session
            return ''

        return request.env.ref('survey.user_input_session_leaderboard')._render({
            'animate': True,
            'leaderboard': survey._prepare_leaderboard_values()
        })

    # ------------------------------------------------------------
    # QUICK ACCESS SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/s', type='http', auth='public', website=True, sitemap=False)
    def survey_session_code(self, **post):
        """ Renders the survey session code page route.
        This page allows the user to enter the session code of the survey.
        It is mainly used to ease survey access for attendees in session mode. """
        return request.render("survey.survey_session_code")

    @http.route('/s/<string:session_code>', type='http', auth='public', website=True)
    def survey_start_short(self, session_code):
        """" Redirects to 'survey_start' route using a shortened link & token.
        We match the session_code for open surveys.
        This route is used in survey sessions where we need short links for people to type. """

        survey = self._fetch_from_session_code(session_code)
        if survey and survey.session_state in ['ready', 'in_progress']:
            return request.redirect("/survey/start/%s" % survey.access_token)

        return request.redirect("/s")

    @http.route('/survey/check_session_code/<string:session_code>', type='json', auth='public', website=True)
    def survey_check_session_code(self, session_code):
        """ Checks if the given code is matching a survey session_code.
        If yes, redirect to /s/code route.
        If not, return error. The user is invited to type again the code. """
        survey = self._fetch_from_session_code(session_code)
        if survey:
            if survey.session_state in ['ready', 'in_progress']:
                return {"survey_url": "/survey/start/%s" % survey.access_token}
            else:
                return {"error": "survey_session_closed"}

        return {"error": "survey_wrong"}

    def _prepare_manage_session_values(self, survey):
        is_first_question, is_last_question = False, False
        if survey.question_ids:
            most_voted_answers = survey._get_session_most_voted_answers()
            is_first_question = survey._is_first_page_or_question(survey.session_question_id)
            is_last_question = survey._is_last_page_or_question(most_voted_answers, survey.session_question_id)

        values = {
            'survey': survey,
            'is_last_question': is_last_question,
            'is_first_question': is_first_question,
        }

        values.update(self._prepare_question_results_values(survey, request.env['survey.user_input.line']))

        return values

    def _prepare_question_results_values(self, survey, user_input_lines):
        """ Prepares usefull values to display during the host session:

        - question_statistics_graph
          The graph data to display the bar chart for questions of type 'choice'
        - input_lines_values
          The answer values to text/date/datetime questions
        - answers_validity
          An array containing the is_correct value for all question answers.
          We need this special variable because of Chartjs data structure.
          The library determines the parameters (color/label/...) by only passing the answer 'index'
          (and not the id or anything else we can identify).
          In other words, we need to know if the answer at index 2 is correct or not.
        - answer_count
          The number of answers to the current question. """

        question = survey.session_question_id
        answers_validity = []
        if (any(answer.is_correct for answer in question.suggested_answer_ids)):
            answers_validity = [answer.is_correct for answer in question.suggested_answer_ids]
            if question.comment_count_as_answer:
                answers_validity.append(False)

        full_statistics = question._prepare_statistics(user_input_lines)[0]
        input_line_values = []
        if question.question_type in ['char_box', 'date', 'datetime']:
            input_line_values = [{
                'id': line.id,
                'value': line['value_%s' % question.question_type]
            } for line in full_statistics.get('table_data', request.env['survey.user_input.line'])[:100]]

        return {
            'is_html_empty': is_html_empty,
            'question_statistics_graph': full_statistics.get('graph_data'),
            'input_line_values': input_line_values,
            'answers_validity': json.dumps(answers_validity),
            'answer_count': survey.session_question_answer_count,
            'attendees_count': survey.session_answer_count,
        }

```

## File: controllers\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import survey_session_manage

```

## File: data\gamification_badge_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="vendor_certification_badge" model="gamification.badge">
        <field name="name">MyCompany Vendor</field>
        <field name="description">Congratulations, you are now official vendor of MyCompany</field>
        <field name="rule_auth">nobody</field>
        <field name="image_1920" type="base64" file="gamification/static/img/badge_good_job-image.png"/>
    </record>

</data></odoo>

```

## File: data\ir_actions_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="survey_action_server_clean_test_answers" model="ir.actions.server">
            <field name="name">Survey: Clean test answers</field>
            <field name="type">ir.actions.server</field>
            <field name="model_id" ref="model_survey_survey" />
            <field name="binding_model_id" ref="model_survey_survey" />
            <field name="state">code</field>
            <field name="code">
if records:
    env['survey.user_input'].search([('survey_id', 'in', records.ids), ('test_entry', '=', 'True')]).unlink()
            </field>
        </record>
</data>
</odoo>
```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mail_template_user_input_invite" model="mail.template">
            <field name="name">Survey: Invite</field>
            <field name="model_id" ref="model_survey_user_input" />
            <field name="subject">Participate to {{ object.survey_id.display_name }} survey</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="email_to">{{ (object.partner_id.email_formatted or object.email) }}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or 'participant'">participant</t><br/><br/>
        <t t-if="object.survey_id.certification">
            You have been invited to take a new certification.
        </t>
        <t t-else="">
            We are conducting a survey and your response would be appreciated.
        </t>
        <div style="margin: 16px 0px 16px 0px;">
            <a t-att-href="(object.get_start_url())"
                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                <t t-if="object.survey_id.certification">
                    Start Certification
                </t>
                <t t-else="">
                    Start Survey
                </t>
            </a>
        </div>
        <t t-if="object.deadline">
            Please answer the survey for <t t-out="format_date(object.deadline) or ''">05/05/2021</t>.<br/><br/>
        </t>
        Thank you for your participation.
    </p>
</div>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Certification Email template -->
        <record id="mail_template_certification" model="mail.template">
            <field name="name">Survey: Send certification by email</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="subject">Certification: {{ object.survey_id.display_name }}</field>
            <field name="email_from">{{ (object.survey_id.create_uid.email_formatted or user.email_formatted or user.company_id.catchall_formatted) }}</field>
            <field name="email_to">{{ (object.partner_id.email_formatted or object.email) }}</field>
            <field name="body_html" type="html">
<div style="background:#F0F0F0;color:#515166;padding:10px 0px;font-family:Arial,Helvetica,sans-serif;font-size:14px;">
    <table style="width:600px;margin:5px auto;">
        <tbody>
            <tr><td>
                <!-- We use the logo of the company that created the survey (to handle multi company cases) -->
                <a href="/"><img t-attf-src="/logo.png?company={{ object.survey_id.create_uid.company_id.id }}" style="vertical-align:baseline;max-width:100px;" /></a>
            </td><td style="text-align:right;vertical-align:middle;">
                    Certification: <t t-out="object.survey_id.display_name or ''">Feedback Form</t>
            </td></tr>
        </tbody>
    </table>
    <table style="width:600px;margin:0px auto;background:white;border:1px solid #e1e1e1;">
        <tbody>
            <tr><td style="padding:15px 20px 10px 20px;">
                <p>Dear <span t-out="object.partner_id.name or 'participant'">participant</span></p>
                <p>
                    Here is, in attachment, your certification document for
                        <strong t-out="object.survey_id.display_name or ''">Feedback Form</strong>
                </p>
                <p>Congratulations for succeeding the test!</p>
            </td></tr>
        </tbody>
    </table>
</div>
            </field>
            <field name="report_template" ref="certification_report"/>
            <field name="report_name">Certification Document</field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\res_users_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Grant survey permissions to demo user -->
        <record id="base.user_demo" model="res.users">
            <field eval="[(4, ref('group_survey_manager'))]" name="groups_id"/>
        </record>
    </data>
</odoo>

```

## File: data\survey_demo_certification.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Odoo Vendor Certification -->
        <record model="survey.survey" id="vendor_certification">
            <field name="title">MyCompany Vendor Certification</field>
            <field name="access_token">4ead4bc8-b8f2-4760-a682-1fde8ddb95ac</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="access_mode">public</field>
            <field name="questions_layout">page_per_question</field>
            <field name="users_can_go_back" eval="True" />
            <field name="users_login_required" eval="True" />
            <field name="scoring_type" >scoring_with_answers</field>
            <field name="certification" eval="True"/>
            <field name="certification_mail_template_id" ref="mail_template_certification"></field>
            <field name="is_time_limited" >limited</field>
            <field name="time_limit" >10.0</field>
            <field name="is_attempts_limited" eval="True" />
            <field name="attempts_limit">2</field>
            <field name="description" type="html"><p>Test your vendor skills!</p></field>
            <field name="certification_give_badge">True</field>
            <field name="certification_badge_id" ref="vendor_certification_badge"/>
            <field name="background_image" type="base64" file="survey/static/src/img/survey_background_2.jpg"/>
        </record>
        <!-- Page 1 -->
        <record model="survey.question" id="vendor_certification_page_1">
            <field name="title">Products</field>
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">1</field>
            <field name="is_page" eval="True"/>
            <field name="question_type" eval="False" />
            <field name="description" type="html"><p>Test your knowledge of your products!</p></field>
        </record>
        <!-- Question and predefined answer 1 -->
        <record model="survey.question" id="vendor_certification_page_1_question_1">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">2</field>
            <field name="title">Do we sell Acoustic Bloc Screens?</field>
            <field name="question_type">simple_choice</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_1_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_1"/>
            <field name="sequence">1</field>
            <field name="value">No</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_1_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_1"/>
            <field name="sequence">2</field>
            <field name="value">Yes</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <!-- Question and predefined answer 2 -->
        <record model="survey.question" id="vendor_certification_page_1_question_2">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">3</field>
            <field name="title">Select all the existing products</field>
            <field name="question_type">multiple_choice</field>
            <field name="column_nb">4</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_2_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Chair floor protection</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_2_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Fanta</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_2_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Conference chair</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_2_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Drawer</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_2_choice_5">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">5</field>
            <field name="value">Customizable Lamp</field>
            <field name="answer_score">-1.0</field>
        </record>
        <!-- Question and predefined answer 3 -->
        <record model="survey.question" id="vendor_certification_page_1_question_3">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">4</field>
            <field name="title">Select all the available customizations for our Customizable Desk</field>
            <field name="question_type">multiple_choice</field>
            <field name="column_nb">4</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_3_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">1</field>
            <field name="value">Color</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_3_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">2</field>
            <field name="value">Height</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_3_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">3</field>
            <field name="value">Width</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_3_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">4</field>
            <field name="value">Legs</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_3_choice_5">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">5</field>
            <field name="value">Number of drawers</field>
            <field name="answer_score">-1.0</field>
        </record>
        <!-- Question and predefined answer 4 -->
        <record model="survey.question" id="vendor_certification_page_1_question_4">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">5</field>
            <field name="title">How many versions of the Corner Desk do we have?</field>
            <field name="question_type">simple_choice</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_4_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">1</field>
            <field name="value">1</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_4_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">2</field>
            <field name="value">2</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_4_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">3</field>
            <field name="value">3</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_1_question_4_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">4</field>
            <field name="value">4</field>
        </record>
        <!-- Question and predefined answer 5 -->
        <record model="survey.question" id="vendor_certification_page_1_question_5">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">6</field>
            <field name="title">Do you think we have missing products in our catalog? (not rated)</field>
            <field name="question_type">text_box</field>
        </record>
        <!-- Page 2 -->
        <record model="survey.question" id="vendor_certification_page_2">
            <field name="title">Prices</field>
            <field name="survey_id" ref="vendor_certification" />
            <field name="is_page" eval="True"/>
            <field name="question_type" eval="False" />
            <field name="sequence">7</field>
            <field name="description">&lt;p&gt;Test your knowledge of our prices.&lt;/p&gt;</field>
        </record>
        <!-- Question and predefined answer 6 -->
        <record model="survey.question" id="vendor_certification_page_2_question_1">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">8</field>
            <field name="title">How much do we sell our Cable Management Box?</field>
            <field name="question_type">simple_choice</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">1</field>
            <field name="value">$20</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">2</field>
            <field name="value">$50</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">3</field>
            <field name="value">$80</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">4</field>
            <field name="value">$100</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">5</field>
            <field name="value">$200</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_1_choice_6">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">6</field>
            <field name="value">$300</field>
        </record>
        <!-- Question and predefined answer 7 -->
        <record model="survey.question" id="vendor_certification_page_2_question_2">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">9</field>
            <field name="title">Select all the products that sell for $100 or more</field>
            <field name="question_type">multiple_choice</field>
            <field name="column_nb">2</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Corner Desk Right Sit</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Desk Combination</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Cabinet with Doors</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Large Desk</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">5</field>
            <field name="value">Letter Tray</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_2_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">6</field>
            <field name="value">Office Chair Black</field>
            <field name="answer_score">-1.0</field>
        </record>
        <!-- Question and predefined answer 8 -->
        <record model="survey.question" id="vendor_certification_page_2_question_3">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">10</field>
            <field name="title">What do you think about our prices (not rated)?</field>
            <field name="question_type">simple_choice</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_3_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">1</field>
            <field name="value">Very underpriced</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_3_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">2</field>
            <field name="value">Underpriced</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_3_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">3</field>
            <field name="value">Correctly priced</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_3_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">4</field>
            <field name="value">A little bit overpriced</field>
        </record>
        <record model="survey.question.answer" id="vendor_certification_page_2_question_3_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">5</field>
            <field name="value">A lot overpriced</field>
        </record>
        <!-- Page 3 -->
        <record model="survey.question" id="vendor_certification_page_3">
            <field name="title">Policies</field>
            <field name="survey_id" ref="vendor_certification" />
            <field name="is_page" eval="True"/>
            <field name="question_type" eval="False" />
            <field name="sequence">11</field>
            <field name="description">&lt;p&gt;Test your knowledge of our policies.&lt;/p&gt;</field>
        </record>
        <!-- Question and predefined answer 9 -->
        <record model="survey.question" id="vendor_certification_page_3_question_1">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">12</field>
            <field name="title">How many days is our money-back guarantee?</field>
            <field name="question_type">numerical_box</field>
            <field name="is_scored_question" eval="True" />
            <field name="answer_numerical_box">30</field>
            <field name="answer_score">1.0</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <!-- Question and predefined answer 10 -->
        <record model="survey.question" id="vendor_certification_page_3_question_2">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">13</field>
            <field name="title">If a customer purchases a product on 6 January 2020, what is the latest day we expect to ship it?</field>
            <field name="question_type">date</field>
            <field name="is_scored_question" eval="True" />
            <field name="answer_date">2020-01-08</field>
            <field name="answer_score">1.0</field>
        </record>
        <!-- Question and predefined answer 11 -->
        <record model="survey.question" id="vendor_certification_page_3_question_3">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">14</field>
            <field name="title">If a customer purchases a 1 year warranty on 6 January 2020, when do we expect the warranty to expire?</field>
            <field name="question_type">datetime</field>
            <field name="is_scored_question" eval="True" />
            <field name="answer_datetime">2021-01-07 00:00:01</field>
            <field name="answer_score">1.0</field>
        </record>
        <!-- Question and predefined answer 12 -->
        <record model="survey.question" id="vendor_certification_page_3_question_4">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">15</field>
            <field name="title">What day to you think is best for us to start having an annual sale (not rated)?</field>
            <field name="question_type">date</field>
            <field name="answer_score">0</field>
        </record>
        <!-- Question and predefined answer 13 -->
        <record model="survey.question" id="vendor_certification_page_3_question_5">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">16</field>
            <field name="title">What day and time do you think most customers are most likely to call customer service (not rated)?</field>
            <field name="question_type">datetime</field>
            <field name="answer_score">0</field>
        </record>
        <!-- Question and predefined answer 14 -->
        <record model="survey.question" id="vendor_certification_page_3_question_6">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">17</field>
            <field name="title">How many chairs do you think we should aim to sell in a year (not rated)?</field>
            <field name="question_type">numerical_box</field>
            <field name="answer_score">0</field>
        </record>
    </data>
</odoo>

```

## File: data\survey_demo_certification_user_input.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="survey_vendor_certification_answer_1" model="survey.user_input">
        <field name="survey_id" ref="survey.vendor_certification" />
        <field name="partner_id" ref="base.res_partner_address_3"/>
        <field name="email">douglas.fletcher51@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=1, hours=3, minutes=10)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=1, hours=3, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_vendor_certification_answer_2" model="survey.user_input">
        <field name="survey_id" ref="survey.vendor_certification" />
        <field name="partner_id" ref="base.res_partner_address_7"/>
        <field name="email">billy.fox45@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=0, hours=3, minutes=0)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=0, hours=3, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_vendor_certification_answer_3" model="survey.user_input">
        <field name="survey_id" ref="survey.vendor_certification" />
        <field name="partner_id" ref="base.res_partner_address_15"/>
        <field name="email">brandon.freeman55@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=0, hours=3, minutes=30)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=0, hours=3, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_vendor_certification_answer_4" model="survey.user_input">
        <field name="survey_id" ref="survey.vendor_certification" />
        <field name="partner_id" ref="base.res_partner_address_25"/>
        <field name="email">oscar.morgan11@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=0, hours=2, minutes=30)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=0, hours=2, minutes=50)"/>
        <field name="state">done</field>
    </record>

</data></odoo>

```

## File: data\survey_demo_certification_user_input_line.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <!-- User input 1 -->
    <!-- page 1 -->
    <record id="survey_vendor_certification_answer_1_p1_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_1_choice_2"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q2_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_1"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q2_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_3"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q2_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_4"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q3_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_1"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q3_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_3"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q3_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_4"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_4_choice_2"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p1_q5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_1_question_5"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">I think it misses a product but I don't know what</field>
    </record>
    <!-- page 2 -->
    <record id="survey_vendor_certification_answer_1_p2_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_2_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_1_choice_4"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p2_q2_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_1"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p2_q2_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_2"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p2_q2_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_4"/>
    </record>
    <record id="survey_vendor_certification_answer_1_p2_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_2_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_3_choice_3"/>
    </record>
    <!-- page 3 -->
    <record id="survey_vendor_certification_answer_1_p3_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_1"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">30</field>
    </record>
    <record id="survey_vendor_certification_answer_1_p3_q2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_2"/>
        <field name="answer_type">date</field>
        <field name="value_date">2020-01-08</field>
    </record>
    <record id="survey_vendor_certification_answer_1_p3_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_3"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-07 00:00:01</field>
    </record>
    <record id="survey_vendor_certification_answer_1_p3_q4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_4"/>
        <field name="answer_type">date</field>
        <field name="value_date">2020-01-01</field>
    </record>
    <record id="survey_vendor_certification_answer_1_p3_q5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_5"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-01 01:00:01</field>
    </record>
    <record id="survey_vendor_certification_answer_1_p3_q6" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_vendor_certification_answer_1"/>
        <field name="question_id" ref="vendor_certification_page_3_question_6"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">1000</field>
    </record>

    <!-- User input 2 -->
    <!-- page 1 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_1_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q2_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_3"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q3_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q3_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_3"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_4_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p1_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_1_question_5"/>
        <field name="skipped" eval="True"/>
    </record>
    <!-- page 2 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p2_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_2_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_1_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p2_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p2_q2_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p2_q2_3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p2_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_2_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_3_choice_4"/>
    </record>
    <!-- page 3 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_1"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">30</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_2"/>
        <field name="answer_type">date</field>
        <field name="value_date">2020-01-09</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_3"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-07 00:00:01</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_4"/>
        <field name="skipped" eval="True"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_5"/>
        <field name="skipped" eval="True"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_2_p3_q6">
        <field name="user_input_id" ref="survey_vendor_certification_answer_2"/>
        <field name="question_id" ref="vendor_certification_page_3_question_6"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">0</field>
    </record>

    <!-- User input 3 -->
    <!-- page 1 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_1_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q2_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q3_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q3_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_4_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p1_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_1_question_5"/>
        <field name="skipped" eval="True"/>
    </record>
    <!-- page 2 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p2_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_2_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_1_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p2_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p2_q2_2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p2_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_2_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_3_choice_2"/>
    </record>
    <!-- page 3 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_1"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">30</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_2"/>
        <field name="answer_type">date</field>
        <field name="value_date">2020-01-08</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_3"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-06 23:59:59</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_4"/>
        <field name="skipped" eval="True"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_5"/>
        <field name="skipped" eval="True"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_3_p3_q6">
        <field name="user_input_id" ref="survey_vendor_certification_answer_3"/>
        <field name="question_id" ref="vendor_certification_page_3_question_6"/>
        <field name="skipped" eval="True"/>
    </record>

    <!-- User input 4 -->
    <!-- page 1 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p1_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_1_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_1_choice_1"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p1_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_1_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_2_choice_3"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p1_q3_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_1_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_3_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p1_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_1_question_4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_1_question_4_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p1_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_1_question_5"/>
        <field name="skipped" eval="True"/>
    </record>
    <!-- page 2 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p2_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_2_question_1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_1_choice_2"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p2_q2_1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_2_question_2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_2_choice_4"/>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p2_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_2_question_3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="vendor_certification_page_2_question_3_choice_5"/>
    </record>
    <!-- page 3 -->
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q1">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_1"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">2</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q2">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_2"/>
        <field name="answer_type">date</field>
        <field name="value_date">2020-01-08</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q3">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_3"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-07 00:00:01</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q4">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_4"/>
        <field name="answer_type">date</field>
        <field name="value_date">2019-12-31</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q5">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_5"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime">2021-01-01 13:00:01</field>
    </record>
    <record model="survey.user_input.line" id="survey_vendor_certification_answer_4_p3_q6">
        <field name="user_input_id" ref="survey_vendor_certification_answer_4"/>
        <field name="question_id" ref="vendor_certification_page_3_question_6"/>
        <field name="skipped" eval="True"/>
    </record>

</data></odoo>

```

## File: data\survey_demo_conditional.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record id="survey_demo_burger_quiz" model="survey.survey">
        <field name="title">Burger Quiz</field>
        <field name="access_token">burger00-quiz-1234-abcd-344ca2tgb31e</field>
        <field name="user_id" ref="base.user_demo"/>
        <field name="access_mode">public</field>
        <field name="users_can_go_back" eval="True"/>
        <field name="scoring_type">scoring_with_answers</field>
        <field name="scoring_success_min">55</field>
        <field name="is_time_limited" >limited</field>
        <field name="time_limit" >10.0</field>
        <field name="questions_layout">page_per_question</field>
        <field name="description" type="html">
            <p>Choose your favourite subject and show how good you are. Ready ?</p></field>
        <field name="background_image" type="base64" file="survey/static/src/img/burger_quiz_background.jpg"/>
    </record>

    <!-- Page 1: Start -->
    <record id="survey_demo_burger_quiz_p1" model="survey.question">
        <field name="title">Start</field>
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">1</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
    </record>
    <record id="survey_demo_burger_quiz_p1_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">2</field>
        <field name="title">Pick a subject</field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
        <record id="survey_demo_burger_quiz_p1_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p1_q1"/>
            <field name="sequence">1</field>
            <field name="value">Geography</field>
        </record>
        <record id="survey_demo_burger_quiz_p1_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p1_q1"/>
            <field name="sequence">2</field>
            <field name="value">History</field>
        </record>
        <record id="survey_demo_burger_quiz_p1_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p1_q1"/>
            <field name="sequence">3</field>
            <field name="value">Sciences</field>
        </record>
        <record id="survey_demo_burger_quiz_p1_q1_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p1_q1"/>
            <field name="sequence">4</field>
            <field name="value">Art &amp; Culture</field>
        </record>

    <!-- Page 2 : Geography -->
    <record id="survey_demo_burger_quiz_p2" model="survey.question">
        <field name="title">Geography</field>
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">100</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
    </record>
    <record id="survey_demo_burger_quiz_p2_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">110</field>
        <field name="title">How long is the White Nile river?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug1"/>
    </record>
        <record id="survey_demo_burger_quiz_p2_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q1"/>
            <field name="sequence">1</field>
            <field name="value">1450 km</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q1"/>
            <field name="sequence">2</field>
            <field name="value">3700 km</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q1"/>
            <field name="sequence">3</field>
            <field name="value">6650 km</field>
        </record>
        <!-- TODO DBE: Add free text pages with corrections. 1450 km is Blue Nile, 6500 km is the total Nile lenght. -->

    <record id="survey_demo_burger_quiz_p2_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">120</field>
        <field name="title">What is the biggest city in the world ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug1"/>
    </record>
        <record id="survey_demo_burger_quiz_p2_q2_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q2"/>
            <field name="sequence">1</field>
            <field name="value">Shanghai</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q2_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q2"/>
            <field name="sequence">2</field>
            <field name="value">Tokyo</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q2_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q2"/>
            <field name="sequence">3</field>
            <field name="value">New York</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q2_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q2"/>
            <field name="sequence">4</field>
            <field name="value">Istanbul</field>
        </record>

    <record id="survey_demo_burger_quiz_p2_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">130</field>
        <field name="title">Which is the highest volcano in Europe ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug1"/>
    </record>
        <record id="survey_demo_burger_quiz_p2_q3_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q3"/>
            <field name="sequence">1</field>
            <field name="value">Mount Teide (Spain - Tenerife)</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q3_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q3"/>
            <field name="sequence">2</field>
            <field name="value">Eyjafjallajökull (Iceland)</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q3_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q3"/>
            <field name="sequence">3</field>
            <field name="value">Mount Etna (Italy - Sicily)</field>
        </record>
        <record id="survey_demo_burger_quiz_p2_q3_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p2_q3"/>
            <field name="sequence">4</field>
            <field name="value">Mount Elbrus (Russia)</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>

    <!-- Page 3 : History -->
    <record id="survey_demo_burger_quiz_p3" model="survey.question">
        <field name="title">History</field>
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">200</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
    </record>
    <record id="survey_demo_burger_quiz_p3_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">210</field>
        <field name="title">When did Genghis Khan die ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug2"/>
    </record>
        <record id="survey_demo_burger_quiz_p3_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q1"/>
            <field name="sequence">1</field>
            <field name="value">1227</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p3_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q1"/>
            <field name="sequence">2</field>
            <field name="value">1324</field> <!--Marco Polo-->
        </record>
        <record id="survey_demo_burger_quiz_p3_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q1"/>
            <field name="sequence">3</field>
            <field name="value">1055</field> <!-- Emperor Xingzong (Liao Dynasty) -->
        </record>

    <record id="survey_demo_burger_quiz_p3_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">220</field>
        <field name="title">Who is the architect of the Great Pyramid of Giza ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug2"/>
    </record>
        <record id="survey_demo_burger_quiz_p3_q2_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q2"/>
            <field name="sequence">1</field>
            <field name="value">Imhotep</field>  <!-- Djoser's pyramid -->
        </record>
        <record id="survey_demo_burger_quiz_p3_q2_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q2"/>
            <field name="sequence">2</field>
            <field name="value">Amenhotep</field>  <!-- Pharaoh -->
        </record>
        <record id="survey_demo_burger_quiz_p3_q2_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q2"/>
            <field name="sequence">3</field>
            <field name="value">Hemiunu</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p3_q2_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q2"/>
            <field name="sequence">4</field>
            <field name="value">Papyrus</field>
        </record>

    <record id="survey_demo_burger_quiz_p3_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">230</field>
        <field name="title">How many years did the 100 years war last ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug2"/>
    </record>
        <record id="survey_demo_burger_quiz_p3_q3_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q3"/>
            <field name="sequence">1</field>
            <field name="value">99 years</field>
        </record>
        <record id="survey_demo_burger_quiz_p3_q3_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q3"/>
            <field name="sequence">2</field>
            <field name="value">100 years</field>
        </record>
        <record id="survey_demo_burger_quiz_p3_q3_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q3"/>
            <field name="sequence">3</field>
            <field name="value">116 years</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p3_q3_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p3_q3"/>
            <field name="sequence">4</field>
            <field name="value">127 years</field>
        </record>

    <!-- Page 4 : Sciences -->
    <record id="survey_demo_burger_quiz_p4" model="survey.question">
        <field name="title">Sciences</field>
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">300</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
    </record>
    <record id="survey_demo_burger_quiz_p4_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">310</field>
        <field name="title">Who received a Nobel prize in Physics for the discovery of neutrino oscillations, which shows that neutrinos have mass ?</field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug3"/>
    </record>
        <record id="survey_demo_burger_quiz_p4_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q1"/>
            <field name="sequence">1</field>
            <field name="value">Arthur B. McDonald</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">5</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q1"/>
            <field name="sequence">2</field>
            <field name="value">Peter W. Higgs</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q1"/>
            <field name="sequence">3</field>
            <field name="value">Takaaki Kajita</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">5</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q1_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q1"/>
            <field name="sequence">4</field>
            <field name="value">Willard S. Boyle</field>
        </record>

    <record id="survey_demo_burger_quiz_p4_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">320</field>
        <field name="title">What is, approximately, the critical mass of plutonium-239 ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug3"/>
    </record>
        <record id="survey_demo_burger_quiz_p4_q2_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q2"/>
            <field name="sequence">1</field>
            <field name="value">5.7 kg</field>  <!-- Djoser's pyramid -->
        </record>
        <record id="survey_demo_burger_quiz_p4_q2_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q2"/>
            <field name="sequence">2</field>
            <field name="value">10 kg</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q2_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q2"/>
            <field name="sequence">3</field>
            <field name="value">16.2 kg</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q2_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q2"/>
            <field name="sequence">4</field>
            <field name="value">47 kg</field>
        </record>

    <record id="survey_demo_burger_quiz_p4_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">330</field>
        <field name="title">Can Humans ever directly see a photon ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug3"/>
    </record>
        <record id="survey_demo_burger_quiz_p4_q3_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q3"/>
            <field name="sequence">1</field>
            <field name="value">Yes, that's the only thing a human eye can see.</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p4_q3_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p4_q3"/>
            <field name="sequence">2</field>
            <field name="value">No, it's to small for the human eye.</field>
        </record>

    <!-- Page 5 : Art & Culture -->
    <record id="survey_demo_burger_quiz_p5" model="survey.question">
        <field name="title">Art &amp; Culture</field>
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">400</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
    </record>
    <record id="survey_demo_burger_quiz_p5_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">410</field>
        <field name="title">Which Musician is not in the 27th Club ?</field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug4"/>
    </record>
        <record id="survey_demo_burger_quiz_p5_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q1"/>
            <field name="sequence">1</field>
            <field name="value">Kurt Cobain</field>
        </record>
        <record id="survey_demo_burger_quiz_p5_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q1"/>
            <field name="sequence">2</field>
            <field name="value">Kim Jong-hyun</field> <!-- To distinguish from the North Korean Leader Kim Jong-un -->
        </record>
        <record id="survey_demo_burger_quiz_p5_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q1"/>
            <field name="sequence">3</field>
            <field name="value">Avicii</field> <!-- Died at 28 -->
            <field name="is_correct" eval="True"/>
            <field name="answer_score">5</field>
        </record>
        <record id="survey_demo_burger_quiz_p5_q1_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q1"/>
            <field name="sequence">4</field>
            <field name="value">Cliff Burton</field> <!-- Died at 24 -->
            <field name="is_correct" eval="True"/>
            <field name="answer_score">5</field>
        </record>

    <record id="survey_demo_burger_quiz_p5_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">420</field>
        <field name="title">Which painting/drawing was not made by Pablo Picasso ?</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="allow_value_image" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug4"/>
    </record>
        <record id="survey_demo_burger_quiz_p5_q2_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q2"/>
            <field name="sequence">1</field>
            <field name="value"> </field>
            <field name="value_image" type="base64" file="survey/static/src/img/burger_quiz_guernica.jpg"/>
        </record>
        <record id="survey_demo_burger_quiz_p5_q2_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q2"/>
            <field name="sequence">2</field>
            <field name="value"> </field>
            <field name="value_image" type="base64" file="survey/static/src/img/burger_quiz_cubism_klein.jpg"/>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p5_q2_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q2"/>
            <field name="sequence">3</field>
            <field name="value"> </field>
            <field name="value_image" type="base64" file="survey/static/src/img/burger_quiz_don_quixote.jpg"/>
        </record>
        <record id="survey_demo_burger_quiz_p5_q2_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q2"/>
            <field name="sequence">4</field>
            <field name="value"> </field>
            <field name="value_image" type="base64" file="survey/static/src/img/burger_quiz_self_portrait.jpg"/>
        </record>

    <record id="survey_demo_burger_quiz_p5_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_burger_quiz"/>
        <field name="sequence">430</field>
        <field name="title">Which quote is from Jean-Claude Van Damme</field>
        <field name="question_type">simple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_conditional" eval="True"/>
        <field name="triggering_question_id" ref="survey_demo_burger_quiz_p1_q1"/>
        <field name="triggering_answer_id" ref="survey_demo_burger_quiz_p1_q1_sug4"/>
    </record>
        <record id="survey_demo_burger_quiz_p5_q3_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q3"/>
            <field name="sequence">1</field>
            <field name="value">I’ve never really wanted to go to Japan. Simply because I don’t like eating fish. And I know that’s very popular out there in Africa.</field> <!-- Britney Spears -->
        </record>
        <record id="survey_demo_burger_quiz_p5_q3_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q3"/>
            <field name="sequence">2</field>
            <field name="value">I am fascinated by air. If you remove the air from the sky, all the birds would fall to the ground. And all the planes, too.</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_burger_quiz_p5_q3_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q3"/>
            <field name="sequence">3</field>
            <field name="value">I've been noticing gravity since I was very young !</field> <!-- Cameron Diaz -->
        </record>
        <record id="survey_demo_burger_quiz_p5_q3_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_burger_quiz_p5_q3"/>
            <field name="sequence">4</field>
            <field name="value">I actually don't like thinking. I think people think I like to think a lot. And I don't. I do not like to think at all.</field> <!-- Kanye West -->
        </record>

</data></odoo>

```

## File: data\survey_demo_feedback.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record model="survey.survey" id="survey_feedback">
        <field name="title">Feedback Form</field>
        <field name="access_token">b135640d-14d4-4748-9ef6-344ca256531e</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="access_mode">public</field>
        <field name="users_can_go_back" eval="True" />
        <field name="questions_layout">page_per_section</field>
        <field name="description" type="html">
<p>This survey allows you to give a feedback about your experience with our products.
    Filling it helps us improving your experience.</p></field>
    </record>

    <!-- Page1: general information -->
    <record model="survey.question" id="survey_feedback_p1">
        <field name="title">About you</field>
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">1</field>
        <field name="question_type" eval="False" />
        <field name="is_page" eval="True" />
        <field name="description" type="html">
<p>This section is about general information about you. Answering them helps qualifying your answers.</p></field>
    </record>
    <record model="survey.question" id="survey_feedback_p1_q1">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">2</field>
        <field name="title">Where do you live ?</field>
        <field name="question_type">char_box</field>
        <field name="constr_mandatory" eval="False"/>
    </record>
    <record model="survey.question" id="survey_feedback_p1_q2">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">3</field>
        <field name="title">When is your date of birth ?</field>
        <field name="question_type">date</field>
        <field name="constr_mandatory" eval="False"/>
    </record>
    <record model="survey.question" id="survey_feedback_p1_q3">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">4</field>
        <field name="title">How frequently do you buy products online ?</field>
        <field name="question_type">simple_choice</field>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p1_q3_sug1">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">1</field>
        <field name="value">Once a day</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p1_q3_sug2">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">2</field>
        <field name="value">Once a week</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p1_q3_sug3">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">3</field>
        <field name="value">Once a month</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p1_q3_sug4">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">4</field>
        <field name="value">Once a year</field>
    </record>
    <record model="survey.question" id="survey_feedback_p1_q4">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">5</field>
        <field name="title">How many times did you order products on our website ?</field>
        <field name="question_type">numerical_box</field>
        <field name="constr_mandatory" eval="True"/>
    </record>

    <!-- Page 2 -->
    <record model="survey.question" id="survey_feedback_p2">
        <field name="title">About our ecommerce</field>
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">6</field>
        <field name="is_page" eval="True" />
        <field name="question_type" eval="False" />
        <field name="description" type="html">
<p>This section is about our eCommerce experience itself.</p></field>
    </record>
    <record model="survey.question" id="survey_feedback_p2_q1">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">7</field>
        <field name="title">Which of the following words would you use to describe our products ?</field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="False"/>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug1">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">1</field>
        <field name="value">High quality</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug2">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">2</field>
        <field name="value">Useful</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug3">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">3</field>
        <field name="value">Unique</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug4">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">4</field>
        <field name="value">Good value for money</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug5">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">5</field>
        <field name="value">Overpriced</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug6">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">6</field>
        <field name="value">Impractical</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug7">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">7</field>
        <field name="value">Ineffective</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q1_sug8">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">8</field>
        <field name="value">Poor quality</field>
    </record>
    <record model="survey.question" id="survey_feedback_p2_q2">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">8</field>
        <field name="title">What do you think about our new eCommerce ?</field>
        <field name="question_type">matrix</field>
        <field name="matrix_subtype">multiple</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_col1">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">1</field>
        <field name="value">Totally disagree</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_col2">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">2</field>
        <field name="value">Disagree</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_col3">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">3</field>
        <field name="value">Agree</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_col4">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">4</field>
        <field name="value">Totally agree</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_row1">
        <field name="matrix_question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">1</field>
        <field name="value">The new layout and design is fresh and up-to-date</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_row2">
        <field name="matrix_question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">2</field>
        <field name="value">It is easy to find the product that I want</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_row3">
        <field name="matrix_question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">3</field>
        <field name="value">The tool to compare the products is useful to make a choice</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_row4">
        <field name="matrix_question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">4</field>
        <field name="value">The checkout process is clear and secure</field>
    </record>
    <record model="survey.question.answer" id="survey_feedback_p2_q2_row5">
        <field name="matrix_question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">5</field>
        <field name="value">I have added products to my wishlist</field>
    </record>
    <record model="survey.question" id="survey_feedback_p2_q3">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">9</field>
        <field name="title">Do you have any other comments, questions, or concerns ?</field>
        <field name="question_type">text_box</field>
        <field name="constr_mandatory" eval="False"/>
    </record>

</data></odoo>

```

## File: data\survey_demo_feedback_user_input.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record id="survey_answer_1" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="email">mark.brown23@example.com</field>
        <field name="state">done</field>
    </record>
    <record id="survey_answer_2" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="partner_id" ref="base.res_partner_address_7"/>
        <field name="email">billy.fox45@example.com</field>
        <field name="state">done</field>
    </record>
    <record id="survey_answer_3" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="partner_id" eval="False"/>
        <field name="email">Evelyne Gargouillis &lt;evelyne@example.com&gt;</field>
        <field name="state">done</field>
    </record>
    <record id="survey_answer_4" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="partner_id" eval="False"/>
        <field name="email">Martin Tamarre &lt;martin@example.com&gt;</field>
        <field name="state">in_progress</field>
    </record>

</data></odoo>

```

## File: data\survey_demo_feedback_user_input_line.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <!-- User input 1 -->
    <record id="survey_answer_1_p1_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Brussels</field>
    </record>
    <record id="survey_answer_1_p1_q2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p1_q2"/>
        <field name="answer_type">date</field>
        <field name="value_date">1980-01-11</field>
    </record>
    <record id="survey_answer_1_p1_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p1_q3_sug3"/>
    </record>
    <record id="survey_answer_1_p1_q4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">4</field>
    </record>
    <record id="survey_answer_1_p2_q1_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug1"/>
    </record>
    <record id="survey_answer_1_p2_q1_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug2"/>
    </record>
    <record id="survey_answer_1_p2_q2_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row1"/>
    </record>
    <record id="survey_answer_1_p2_q2_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row2"/>
    </record>
    <record id="survey_answer_1_p2_q2_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col2"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row3"/>
    </record>
    <record id="survey_answer_1_p2_q2_4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col4"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row4"/>
    </record>
    <record id="survey_answer_1_p2_q2_5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col2"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row5"/>
    </record>
    <record id="survey_answer_1_p2_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_1"/>
        <field name="question_id" ref="survey_feedback_p2_q3"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">Thanks for the good quality of your products</field>
    </record>

    <!-- User input 2 -->
    <record id="survey_answer_2_p1_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Paris</field>
    </record>
    <record id="survey_answer_2_p1_q2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p1_q2"/>
        <field name="skipped" eval="True"/>
    </record>
    <record id="survey_answer_2_p1_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p1_q3_sug2"/>
    </record>
    <record id="survey_answer_2_p1_q4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">10</field>
    </record>
    <record id="survey_answer_2_p2_q1_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug2"/>
    </record>
    <record id="survey_answer_2_p2_q1_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug3"/>
    </record>
    <record id="survey_answer_2_p2_q1_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug4"/>
    </record>
    <record id="survey_answer_2_p2_q2_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col4"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row1"/>
    </record>
    <record id="survey_answer_2_p2_q2_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row2"/>
    </record>
    <record id="survey_answer_2_p2_q2_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row3"/>
    </record>
    <record id="survey_answer_2_p2_q2_4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col4"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row4"/>
    </record>
    <record id="survey_answer_2_p2_q2_5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row5"/>
    </record>
    <record id="survey_answer_2_p2_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_2"/>
        <field name="question_id" ref="survey_feedback_p2_q3"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">I really appreciate your products. They are awesome !!</field>
    </record>

    <!-- User input 3 -->
    <record id="survey_answer_3_p1_q1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">New York</field>
    </record>
    <record id="survey_answer_3_p1_q2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p1_q2"/>
        <field name="answer_type">date</field>
        <field name="value_date">1966-06-15</field>
    </record>
    <record id="survey_answer_3_p1_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p1_q3_sug4"/>
    </record>
    <record id="survey_answer_3_p1_q4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">1</field>
    </record>
    <record id="survey_answer_3_p2_q1_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug7"/>
    </record>
    <record id="survey_answer_3_p2_q1_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q1_sug8"/>
    </record>
    <record id="survey_answer_3_p2_q2_1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col1"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row1"/>
    </record>
    <record id="survey_answer_3_p2_q2_2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col2"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row2"/>
    </record>
    <record id="survey_answer_3_p2_q2_3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col1"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row3"/>
    </record>
    <record id="survey_answer_3_p2_q2_4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col3"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row4"/>
    </record>
    <record id="survey_answer_3_p2_q2_5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_feedback_p2_q2_col1"/>
        <field name="matrix_row_id" ref="survey_feedback_p2_q2_row5"/>
    </record>
    <record id="survey_answer_3_p2_q3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_answer_3"/>
        <field name="question_id" ref="survey_feedback_p2_q3"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">The customizable desk received is not the one I ordered on your website and the quality is very poor ! Really disappointed.</field>
    </record>

</data></odoo>

```

## File: data\survey_demo_quiz.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record id="survey_demo_quiz" model="survey.survey">
        <field name="title">Quiz about our Company</field>
        <field name="access_token">b137640d-9876-1234-abcd-344ca256531e</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="access_mode">public</field>
        <field name="users_can_go_back" eval="False"/>
        <field name="scoring_type">scoring_with_answers</field>
        <field name="scoring_success_min">55</field>
        <field name="questions_layout">page_per_question</field>
        <field name="description" type="html">
<p>This small quiz will test your knowledge about our Company. Be prepared !</p></field>
        <field name="background_image" type="base64" file="survey/static/src/img/survey_background.jpg"/>
    </record>

    <!-- Page 1: general informations -->
    <record id="survey_demo_quiz_p1" model="survey.question">
        <field name="title">Who are you ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">1</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
        <field name="description" type="html">
<p>Some general information about you. It will be used internally for statistics only.</p></field>
    </record>
    <record id="survey_demo_quiz_p1_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">2</field>
        <field name="title">What is your email ?</field>
        <field name="question_type">char_box</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="validation_email" eval="True"/>
        <field name="save_as_email" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p1_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">3</field>
        <field name="title">What is your nickname ?</field>
        <field name="question_type">char_box</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="save_as_nickname" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p1_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">4</field>
        <field name="title">Where are you from ?</field>
        <field name="question_type">char_box</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p1_q4" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">5</field>
        <field name="title">How old are you ?</field>
        <field name="description" type="html"><p>Just to categorize your answers, don't worry.</p></field>
        <field name="question_type">numerical_box</field>
        <field name="constr_mandatory" eval="True"/>
    </record>

    <!-- Page 2: quiz about company -->
    <record id="survey_demo_quiz_p2" model="survey.question">
        <field name="title">Our Company in a few questions ...</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">10</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
        <field name="description" type="html">
<p>Some questions about our company. Do you really know us?</p></field>
    </record>
    <record id="survey_demo_quiz_p2_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">11</field>
        <field name="title">When is Mitchell Admin born ?</field>
        <field name="description" type="html"><span>Our famous Leader !</span></field>
        <field name="question_type">date</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p2_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">12</field>
        <field name="title">When did precisely Marc Demo crop its first apple tree ?</field>
        <field name="question_type">datetime</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p2_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">13</field>
        <field name="title">Give the list of all types of wood we sell.</field>
        <field name="question_type">text_box</field>
        <field name="constr_mandatory" eval="False"/>
    </record>

    <!-- Page 3: quiz about fruits and vegetables -->
    <record id="survey_demo_quiz_p3" model="survey.question">
        <field name="title">Fruits and vegetables</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">20</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
        <field name="description" type="html">
<p>An apple a day keeps the doctor away.</p></field>
    </record>
    <record id="survey_demo_quiz_p3_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">21</field>
        <field name="title">Which category does a tomato belong to</field>
        <field name="description" type="html"><span>"Red" is not a category, I know what you are trying to do ;)</span></field>
        <field name="question_type">simple_choice</field>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
        <record id="survey_demo_quiz_p3_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
            <field name="sequence">1</field>
            <field name="value">Fruits</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">20</field>
        </record>
        <record id="survey_demo_quiz_p3_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
            <field name="sequence">2</field>
            <field name="value">Vegetables</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_quiz_p3_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
            <field name="sequence">3</field>
            <field name="value">Space stations</field>
        </record>
    <record id="survey_demo_quiz_p3_q2" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">22</field>
        <field name="title">Which of the following would you use to pollinate</field>
        <field name="question_type">simple_choice</field>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="False"/>
        <field name="constr_mandatory" eval="True"/>
        <field name="is_time_limited" eval="True"/>
        <field name="time_limit">15</field>
    </record>
        <record id="survey_demo_quiz_p3_q2_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
            <field name="sequence">1</field>
            <field name="value">Bees</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">20</field>
        </record>
        <record id="survey_demo_quiz_p3_q2_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
            <field name="sequence">2</field>
            <field name="value">Dogs</field>
        </record>
        <record id="survey_demo_quiz_p3_q2_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
            <field name="sequence">3</field>
            <field name="value">Mooses</field>
        </record>
    <record id="survey_demo_quiz_p3_q3" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">23</field>
        <field name="title">Select trees that made more than 20K sales this year</field>
        <field name="description" type="html"><span>Our sales people have an advantage, but you can do it !</span></field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="False"/>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="True"/>
        <field name="is_time_limited" eval="True"/>
        <field name="time_limit">20</field>
    </record>
        <record id="survey_demo_quiz_p3_q3_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
            <field name="sequence">1</field>
            <field name="value">Apple Trees</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">20</field>
        </record>
        <record id="survey_demo_quiz_p3_q3_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
            <field name="sequence">2</field>
            <field name="value">Lemon Trees</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">10</field>
        </record>
        <record id="survey_demo_quiz_p3_q3_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
            <field name="sequence">3</field>
            <field name="value">Baobab Trees</field>
            <field name="answer_score">-10</field>
        </record>
        <record id="survey_demo_quiz_p3_q3_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
            <field name="sequence">4</field>
            <field name="value">Cookies</field>
            <field name="answer_score">-10</field>
        </record>
    <record id="survey_demo_quiz_p3_q4" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">24</field>
        <field name="title">A "Citrus" could give you ...</field>
        <field name="question_type">multiple_choice</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="False"/>
        <field name="is_time_limited" eval="True"/>
        <field name="time_limit">20</field>
    </record>
        <record id="survey_demo_quiz_p3_q4_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
            <field name="sequence">1</field>
            <field name="value">Pomelos</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">20</field>
        </record>
        <record id="survey_demo_quiz_p3_q4_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
            <field name="sequence">2</field>
            <field name="value">Grapefruits</field>
            <field name="is_correct" eval="True"/>
            <field name="answer_score">20</field>
        </record>
        <record id="survey_demo_quiz_p3_q4_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
            <field name="sequence">3</field>
            <field name="value">Cosmic rays</field>
            <field name="answer_score">-10</field>
        </record>
        <record id="survey_demo_quiz_p3_q4_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
            <field name="sequence">4</field>
            <field name="value">Bricks</field>
            <field name="answer_score">-10</field>
        </record>
    <record id="survey_demo_quiz_p3_q5" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">25</field>
        <field name="title">How often should you water those plants</field>
        <field name="question_type">matrix</field>
        <field name="matrix_subtype">simple</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="comments_allowed" eval="True"/>
    </record>
        <record id="survey_demo_quiz_p3_q5_row1" model="survey.question.answer">
            <field name="matrix_question_id" ref="survey_demo_quiz_p3_q5"/>
            <field name="sequence">1</field>
            <field name="value">Cactus</field>
        </record>
        <record id="survey_demo_quiz_p3_q5_row2" model="survey.question.answer">
            <field name="matrix_question_id" ref="survey_demo_quiz_p3_q5"/>
            <field name="sequence">2</field>
            <field name="value">Ficus</field>
        </record>
        <record id="survey_demo_quiz_p3_q5_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
            <field name="sequence">1</field>
            <field name="value">Once a month</field>
        </record>
        <record id="survey_demo_quiz_p3_q5_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
            <field name="sequence">2</field>
            <field name="value">Once a week</field>
        </record>
    <record id="survey_demo_quiz_p3_q6" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">26</field>
        <field name="title">When do you harvest those fruits</field>
        <field name="description" type="html"><span>Best time to do it, is the right time to do it.</span></field>
        <field name="question_type">matrix</field>
        <field name="matrix_subtype">multiple</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="comments_allowed" eval="False"/>
    </record>
        <record id="survey_demo_quiz_p3_q6_row1" model="survey.question.answer">
            <field name="matrix_question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">1</field>
            <field name="value">Apples</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_row2" model="survey.question.answer">
            <field name="matrix_question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">2</field>
            <field name="value">Strawberries</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_row3" model="survey.question.answer">
            <field name="matrix_question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">3</field>
            <field name="value">Clementine</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">1</field>
            <field name="value">Spring</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">2</field>
            <field name="value">Summer</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">3</field>
            <field name="value">Autumn</field>
        </record>
        <record id="survey_demo_quiz_p3_q6_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
            <field name="sequence">4</field>
            <field name="value">Winter</field>
        </record>
    
    <!-- Page 4: Trees -->
    <record id="survey_demo_quiz_p4" model="survey.question">
        <field name="title">Trees</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">30</field>
        <field name="is_page" eval="True"/>
        <field name="question_type" eval="False"/>
        <field name="description" type="html">
            <p>
                We like to say that the apple doesn't fall far from the tree, so here are trees.
            </p>
        </field>
    </record>

    <record id="survey_demo_quiz_p4_q1" model="survey.question">
        <field name="title">Dogwood is from which family of trees ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">31</field>
        <field name="question_type">simple_choice</field>
        <field name="column_nb">6</field>
        <field name="allow_value_image" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p4_q1_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="sequence">1</field>
        <field name="value">Pinaceae</field>
        <field name="value_image" type="base64" file="survey/static/img/pinaceae.jpg"/>
    </record>
    <record id="survey_demo_quiz_p4_q1_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="sequence">2</field>
        <field name="value">Ulmaceae</field>
        <field name="value_image" type="base64" file="survey/static/img/ulmaceae.jpg"/>
    </record>
    <record id="survey_demo_quiz_p4_q1_sug3" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="sequence">3</field>
        <field name="value">Cornaceae</field>
        <field name="value_image" type="base64" file="survey/static/img/cornaceae.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">20</field>
    </record>
    <record id="survey_demo_quiz_p4_q1_sug4" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="sequence">4</field>
        <field name="value">Salicaceae</field>
        <field name="value_image" type="base64" file="survey/static/img/salicaceae.jpg"/>
    </record>

    <record id="survey_demo_quiz_p4_q2" model="survey.question">
        <field name="title">In which country did the bonsai technique develop ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">32</field>
        <field name="question_type">simple_choice</field>
        <field name="column_nb">6</field>
        <field name="allow_value_image" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p4_q2_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="sequence">1</field>
        <field name="value">Japan</field>
        <field name="value_image" type="base64" file="survey/static/img/japan.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">20</field>
    </record>
    <record id="survey_demo_quiz_p4_q2_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="sequence">2</field>
        <field name="value">China</field>
        <field name="value_image" type="base64" file="survey/static/img/china.jpg"/>
    </record>
    <record id="survey_demo_quiz_p4_q2_sug3" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="sequence">3</field>
        <field name="value">Vietnam</field>
        <field name="value_image" type="base64" file="survey/static/img/vietnam.jpg"/>
    </record>
    <record id="survey_demo_quiz_p4_q2_sug4" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="sequence">4</field>
        <field name="value">South Korea</field>
        <field name="value_image" type="base64" file="survey/static/img/south_korea.jpg"/>
    </record>

    <record id="survey_demo_quiz_p4_q3" model="survey.question">
        <field name="title">Is the wood of a coniferous hard or soft ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">33</field>
        <field name="question_type">simple_choice</field>
        <field name="column_nb">6</field>
        <field name="constr_mandatory" eval="True"/>
        <field name="description" type="html">
            <p>
                <img class="img-fluid o_we_custom_image d-block mx-auto"
                    src="/survey/static/img/coniferous.jpg"/><br/>
            </p>
        </field>
    </record>
    <record id="survey_demo_quiz_p4_q3_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q3"/>
        <field name="sequence">1</field>
        <field name="value">Hard</field>
    </record>
    <record id="survey_demo_quiz_p4_q3_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q3"/>
        <field name="sequence">2</field>
        <field name="value">Soft</field>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">10</field>
    </record>

    <record id="survey_demo_quiz_p4_q4" model="survey.question">
        <field name="title">From which continent is native the Scots pine (pinus sylvestris) ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">34</field>
        <field name="question_type">simple_choice</field>
        <field name="allow_value_image" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
        <field name="description" type="html">
            <p>
                <img class="img-fluid o_we_custom_image d-block mx-auto"
                    src="/survey/static/img/pinus_sylvestris.jpg" style="width: 100%;"/><br/>
            </p>
        </field>
    </record>
    <record id="survey_demo_quiz_p4_q4_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="sequence">1</field>
        <field name="value">Africa</field>
        <field name="value_image" type="base64" file="survey/static/img/africa.png"/>
    </record>
    <record id="survey_demo_quiz_p4_q4_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="sequence">2</field>
        <field name="value">Asia</field>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="value_image" type="base64" file="survey/static/img/asia.png"/>
    </record>
    <record id="survey_demo_quiz_p4_q4_sug3" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="sequence">3</field>
        <field name="value">Europe</field>
        <field name="value_image" type="base64" file="survey/static/img/europe.png"/>
    </record>
    <record id="survey_demo_quiz_p4_q4_sug4" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="sequence">4</field>
        <field name="value">South America</field>
        <field name="value_image" type="base64" file="survey/static/img/south_america.png"/>
    </record>

    <record id="survey_demo_quiz_p4_q5" model="survey.question">
        <field name="title">In the list below, select all the coniferous.</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">35</field>
        <field name="question_type">multiple_choice</field>
        <field name="column_nb">6</field>
        <field name="allow_value_image" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record id="survey_demo_quiz_p4_q5_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="sequence">1</field>
        <field name="value">Douglas Fir</field>
        <field name="value_image" type="base64" file="survey/static/img/douglas_fir.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">5</field>
    </record>
    <record id="survey_demo_quiz_p4_q5_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="sequence">2</field>
        <field name="value">Norway Spruce</field>
        <field name="value_image" type="base64" file="survey/static/img/norway_spruce.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">5</field>
    </record>
    <record id="survey_demo_quiz_p4_q5_sug3" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="sequence">3</field>
        <field name="value">European Yew</field>
        <field name="value_image" type="base64" file="survey/static/img/european_yew.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">5</field>
    </record>
    <record id="survey_demo_quiz_p4_q5_sug4" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="sequence">4</field>
        <field name="value">Mountain Pine</field>
        <field name="value_image" type="base64" file="survey/static/img/mountain_pine.jpg"/>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">5</field>
    </record>

    <record id="survey_demo_quiz_p4_q6" model="survey.question">
        <field name="title">After watching this video, will you swear that you are not going to procrastinate to trim your hedge this year ?</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">36</field>
        <field name="question_type">simple_choice</field>
        <field name="column_nb">6</field>
        <field name="description" type="html">
            <div class="text-center">
                <div class="media_iframe_video" data-oe-expression="//www.youtube.com/embed/7y4T6yv5L1k?autoplay=0&amp;rel=0" style="width: 50%;">
                    <div class="css_editable_mode_display"/>
                    <div class="media_iframe_video_size" contenteditable="false"/>
                    <iframe src="//www.youtube.com/embed/7y4T6yv5L1k?autoplay=0&amp;rel=0" frameborder="0" contenteditable="false"></iframe>
                </div><br/>
            </div>
        </field>
    </record>
    <record id="survey_demo_quiz_p4_q6_sug1" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="sequence">1</field>
        <field name="value">Yes</field>
        <field name="is_correct" eval="True"/>
        <field name="answer_score">10</field>
    </record>
    <record id="survey_demo_quiz_p4_q6_sug2" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="sequence">2</field>
        <field name="value">No</field>
    </record>
    <record id="survey_demo_quiz_p4_q6_sug3" model="survey.question.answer">
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="sequence">3</field>
        <field name="value">Perhaps</field>
        <field name="answer_score">-10</field>
    </record>

    <!-- Page 5: Feedback - non scored question -->
    <record id="survey_demo_quiz_p5" model="survey.question">
        <field name="title">Your feeling</field>
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">40</field>
        <field name="question_type" eval="False"/>
        <field name="is_page" eval="True"/>
        <field name="description" type="html">
            <p>We may be interested by your input.</p></field>
    </record>
    <record id="survey_demo_quiz_p5_q1" model="survey.question">
        <field name="survey_id" ref="survey_demo_quiz"/>
        <field name="sequence">41</field>
        <field name="title">What do you think about this survey ?</field>
        <field name="description" type="html"><span>If you don't like us, please try to be as objective as possible.</span></field>
        <field name="question_type">simple_choice</field>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="True"/>
        <field name="constr_mandatory" eval="False"/>
    </record>
        <record id="survey_demo_quiz_p5_q1_sug1" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p5_q1"/>
            <field name="sequence">1</field>
            <field name="value">Good</field>
        </record>
        <record id="survey_demo_quiz_p5_q1_sug2" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p5_q1"/>
            <field name="sequence">2</field>
            <field name="value">Not Good, Not Bad</field>
        </record>
        <record id="survey_demo_quiz_p5_q1_sug3" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p5_q1"/>
            <field name="sequence">3</field>
            <field name="value">Iznogoud</field>
        </record>
        <record id="survey_demo_quiz_p5_q1_sug4" model="survey.question.answer">
            <field name="question_id" ref="survey_demo_quiz_p5_q1"/>
            <field name="sequence">4</field>
            <field name="value">I have no idea, I'm a dog!</field>
        </record>
</data></odoo>

```

## File: data\survey_demo_quiz_user_input.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record id="survey_demo_quiz_answer_1" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_demo_quiz"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="email">mark.brown23@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=1, hours=3, minutes=30)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=1, hours=4, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_demo_quiz_answer_2" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_demo_quiz"/>
        <field name="partner_id" ref="base.partner_admin"/>
        <field name="email">admin@yourcompany.example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=1, hours=2, minutes=50)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=1, hours=3, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_demo_quiz_answer_3" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_demo_quiz"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
        <field name="email">joel.willis63@example.com</field>
        <field name="end_datetime" eval="datetime.now() - timedelta(days=1, hours=2, minutes=10)"/>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=1, hours=2, minutes=50)"/>
        <field name="state">done</field>
    </record>
    <record id="survey_demo_quiz_answer_4" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_demo_quiz"/>
        <field name="partner_id" ref="base.res_partner_address_28"/>
        <field name="email">colleen.diaz83@example.com</field>
        <field name="start_datetime" eval="datetime.now() - timedelta(days=1, hours=0, minutes=50)"/>
        <field name="state">in_progress</field>
    </record>
    <record id="survey_demo_quiz_answer_5" model="survey.user_input">
        <field name="survey_id" ref="survey.survey_demo_quiz"/>
        <field name="partner_id" ref="base.res_partner_address_34"/>
        <field name="email">travis.mendoza24@example.com</field>
        <field name="state">new</field>
    </record>

</data></odoo>

```

## File: data\survey_demo_quiz_user_input_line.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">
    <!-- Page 1: general informations -->
    <record id="survey_demo_quiz_answer_1_p1_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">mark.brown23@example.com</field>
    </record>
    <record id="survey_demo_quiz_answer_1_p1_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q2"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Mark Brown</field>
    </record>
    <record id="survey_demo_quiz_answer_1_p1_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q3"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Brussels</field>
    </record>
    <record id="survey_demo_quiz_answer_1_p1_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">36</field>
    </record>
    <!-- Page 2: quiz about company -->
    <record id="survey_demo_quiz_answer_1_p2_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q1"/>
        <field name="answer_type">date</field>
        <field name="value_date" eval="DateTime.today() - relativedelta(years=36)"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p2_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q2"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime" eval="DateTime.now().replace(year=2017, month=10, day=2, hour=2, minute=27, second=0)"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p2_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q3"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">Oak, ash, pine</field>
    </record>
    <!-- Page 3: quiz about fruits and vegetables -->
    <record id="survey_demo_quiz_answer_1_p3_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q1_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q2_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q2_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Mooses ?? Really ?</field>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q3_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q3_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_score">-10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q3_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q4_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_score">-10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q6_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p3_q6_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row3"/>
    </record>
    <!-- Page 4: trees -->
    <record id="survey_demo_quiz_answer_1_p4_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q1_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q2_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q3_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q4_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_1_p4_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_1"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q6_sug1"/>
    </record>


    <!-- Page 1: general informations -->
    <record id="survey_demo_quiz_answer_2_p1_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">admin@yourcompany.example.com</field>
    </record>
    <record id="survey_demo_quiz_answer_2_p1_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q2"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Mitchell Admin</field>
    </record>
    <record id="survey_demo_quiz_answer_2_p1_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q3"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Ottawa</field>
    </record>
    <record id="survey_demo_quiz_answer_2_p1_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">48</field>
    </record>
    <!-- Page 2: quiz about company -->
    <record id="survey_demo_quiz_answer_2_p2_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q1"/>
        <field name="answer_type">date</field>
        <field name="value_date" eval="DateTime.today() + relativedelta(years=24)"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p2_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q2"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime" eval="DateTime.now().replace(year=2011, month=8, day=21, hour=15, minute=34, second=0)"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p2_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q3"/>
        <field name="skipped" eval="True"/>
    </record>
    <!-- Page 3: quiz about fruits and vegetables -->
    <record id="survey_demo_quiz_answer_2_p3_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q1_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q2_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q2_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Mooses are best pollinators of the world !</field>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q3_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q3_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">I sold a 30K raspberry tree once.</field>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug2"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q6_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug2"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p3_q6_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug3"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row3"/>
    </record>
    <!-- Page 4: trees -->
    <record id="survey_demo_quiz_answer_2_p4_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q1_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q2_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q3"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q3_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q4_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_2_p4_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_2"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q6_sug1"/>
    </record>


    <!-- Page 1: general informations -->
    <record id="survey_demo_quiz_answer_3_p1_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">joel.willis63@example.com</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p1_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q2"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Joël Willis</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p1_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q3"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Brussels</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p1_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p1_q4"/>
        <field name="answer_type">numerical_box</field>
        <field name="value_numerical_box">28</field>
    </record>
    <!-- Page 2: quiz about company -->
    <record id="survey_demo_quiz_answer_3_p2_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q1"/>
        <field name="answer_type">date</field>
        <field name="value_date" eval="DateTime.today() - relativedelta(years=38)"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p2_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q2"/>
        <field name="answer_type">datetime</field>
        <field name="value_datetime" eval="DateTime.now().replace(year=2005, month=4, day=18, hour=10, minute=0, second=0)"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p2_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p2_q3"/>
        <field name="answer_type">text_box</field>
        <field name="value_text_box">Oak, fur, pine, red pine</field>
    </record>
    <!-- Page 3: quiz about fruits and vegetables -->
    <record id="survey_demo_quiz_answer_3_p3_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q1"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Both fruit (seeds, par of the plant) and vegetable (culinary use), obviously.</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q2"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q2_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q3_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q3_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q3_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q3_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q3"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">You forgot the strawberry tree.</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q4_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_is_correct" eval="True"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q4_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_score">-10</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q4_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q4_l4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q4"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Gives the beeest cosmics rays man. So juicy.</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q5_sug2"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q5_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q5_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q5"/>
        <field name="answer_type">char_box</field>
        <field name="value_char_box">Well sometimes I forget them, they survived. Almost.</field>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug3"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q6_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug4"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q6_l3" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug1"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q6_l4" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug2"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p3_q6_l5" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p3_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p3_q6_sug4"/>
        <field name="matrix_row_id" ref="survey_demo_quiz_p3_q6_row3"/>
    </record>
    <!-- Page 4: trees -->
    <record id="survey_demo_quiz_answer_3_p4_q1_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q1"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q1_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q2_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q2"/>
        <field name="answer_score">20</field>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q2_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q3_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q3"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q3_sug1"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q4_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q4"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q4_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q5_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug2"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q5_l2" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q5"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q5_sug3"/>
    </record>
    <record id="survey_demo_quiz_answer_3_p4_q6_l1" model="survey.user_input.line">
        <field name="user_input_id" ref="survey_demo_quiz_answer_3"/>
        <field name="question_id" ref="survey_demo_quiz_p4_q6"/>
        <field name="answer_type">suggestion</field>
        <field name="suggested_answer_id" ref="survey_demo_quiz_p4_q6_sug1"/>
    </record>

</data></odoo>

```

## File: models\badge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class GamificationBadge(models.Model):
    _inherit = 'gamification.badge'

    survey_ids = fields.One2many('survey.survey', 'certification_badge_id', 'Survey Ids')
    survey_id = fields.Many2one('survey.survey', 'Survey', compute='_compute_survey_id', store=True)

    @api.depends('survey_ids.certification_badge_id')
    def _compute_survey_id(self):
        for badge in self:
            badge.survey_id = badge.survey_ids[0] if badge.survey_ids else None

```

## File: models\challenge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Challenge(models.Model):
    _inherit = 'gamification.challenge'

    challenge_category = fields.Selection(selection_add=[
        ('certification', 'Certifications')
    ], ondelete={'certification': 'set default'})

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    certifications_count = fields.Integer('Certifications Count', compute='_compute_certifications_count')
    certifications_company_count = fields.Integer('Company Certifications Count', compute='_compute_certifications_company_count')

    @api.depends('is_company')
    def _compute_certifications_count(self):
        read_group_res = self.env['survey.user_input'].sudo().read_group(
            [('partner_id', 'in', self.ids), ('scoring_success', '=', True)],
            ['partner_id'], 'partner_id'
        )
        data = dict((res['partner_id'][0], res['partner_id_count']) for res in read_group_res)
        for partner in self:
            partner.certifications_count = data.get(partner.id, 0)

    @api.depends('is_company', 'child_ids.certifications_count')
    def _compute_certifications_company_count(self):
        self.certifications_company_count = sum(child.certifications_count for child in self.child_ids)

    def action_view_certifications(self):
        action = self.env["ir.actions.actions"]._for_xml_id("survey.res_partner_action_certifications")
        action['view_mode'] = 'tree'
        action['domain'] = ['|', ('partner_id', 'in', self.ids), ('partner_id', 'in', self.child_ids.ids)]

        return action

```

## File: models\survey_question.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import collections
import json
import itertools
import operator

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError


class SurveyQuestion(models.Model):
    """ Questions that will be asked in a survey.

        Each question can have one of more suggested answers (eg. in case of
        multi-answer checkboxes, radio buttons...).

        Technical note:

        survey.question is also the model used for the survey's pages (with the "is_page" field set to True).

        A page corresponds to a "section" in the interface, and the fact that it separates the survey in
        actual pages in the interface depends on the "questions_layout" parameter on the survey.survey model.
        Pages are also used when randomizing questions. The randomization can happen within a "page".

        Using the same model for questions and pages allows to put all the pages and questions together in a o2m field
        (see survey.survey.question_and_page_ids) on the view side and easily reorganize your survey by dragging the
        items around.

        It also removes on level of encoding by directly having 'Add a page' and 'Add a question'
        links on the tree view of questions, enabling a faster encoding.

        However, this has the downside of making the code reading a little bit more complicated.
        Efforts were made at the model level to create computed fields so that the use of these models
        still seems somewhat logical. That means:
        - A survey still has "page_ids" (question_and_page_ids filtered on is_page = True)
        - These "page_ids" still have question_ids (questions located between this page and the next)
        - These "question_ids" still have a "page_id"

        That makes the use and display of these information at view and controller levels easier to understand.
    """
    _name = 'survey.question'
    _description = 'Survey Question'
    _rec_name = 'title'
    _order = 'sequence,id'

    @api.model
    def default_get(self, fields):
        defaults = super(SurveyQuestion, self).default_get(fields)
        if (not fields or 'question_type' in fields):
            defaults['question_type'] = False if defaults.get('is_page') == True else 'text_box'
        return defaults

    # question generic data
    title = fields.Char('Title', required=True, translate=True)
    description = fields.Html(
        'Description', translate=True, sanitize=False,  # TDE TODO: sanitize but find a way to keep youtube iframe media stuff
        help="Use this field to add additional explanations about your question or to illustrate it with pictures or a video")
    survey_id = fields.Many2one('survey.survey', string='Survey', ondelete='cascade')
    scoring_type = fields.Selection(related='survey_id.scoring_type', string='Scoring Type', readonly=True)
    sequence = fields.Integer('Sequence', default=10)
    # page specific
    is_page = fields.Boolean('Is a page?')
    question_ids = fields.One2many('survey.question', string='Questions', compute="_compute_question_ids")
    questions_selection = fields.Selection(
        related='survey_id.questions_selection', readonly=True,
        help="If randomized is selected, add the number of random questions next to the section.")
    random_questions_count = fields.Integer(
        'Random questions count', default=1,
        help="Used on randomized sections to take X random questions from all the questions of that section.")
    # question specific
    page_id = fields.Many2one('survey.question', string='Page', compute="_compute_page_id", store=True)
    question_type = fields.Selection([
        ('text_box', 'Multiple Lines Text Box'),
        ('char_box', 'Single Line Text Box'),
        ('numerical_box', 'Numerical Value'),
        ('date', 'Date'),
        ('datetime', 'Datetime'),
        ('simple_choice', 'Multiple choice: only one answer'),
        ('multiple_choice', 'Multiple choice: multiple answers allowed'),
        ('matrix', 'Matrix')], string='Question Type',
        compute='_compute_question_type', readonly=False, store=True)
    is_scored_question = fields.Boolean(
        'Scored', compute='_compute_is_scored_question',
        readonly=False, store=True, copy=True,
        help="Include this question as part of quiz scoring. Requires an answer and answer score to be taken into account.")
    # -- scoreable/answerable simple answer_types: numerical_box / date / datetime
    answer_numerical_box = fields.Float('Correct numerical answer', help="Correct number answer for this question.")
    answer_date = fields.Date('Correct date answer', help="Correct date answer for this question.")
    answer_datetime = fields.Datetime('Correct datetime answer', help="Correct date and time answer for this question.")
    answer_score = fields.Float('Score', help="Score value for a correct answer to this question.")
    # -- char_box
    save_as_email = fields.Boolean(
        "Save as user email", compute='_compute_save_as_email', readonly=False, store=True, copy=True,
        help="If checked, this option will save the user's answer as its email address.")
    save_as_nickname = fields.Boolean(
        "Save as user nickname", compute='_compute_save_as_nickname', readonly=False, store=True, copy=True,
        help="If checked, this option will save the user's answer as its nickname.")
    # -- simple choice / multiple choice / matrix
    suggested_answer_ids = fields.One2many(
        'survey.question.answer', 'question_id', string='Types of answers', copy=True,
        help='Labels used for proposed choices: simple choice, multiple choice and columns of matrix')
    allow_value_image = fields.Boolean('Images on answers', help='Display images in addition to answer label. Valid only for simple / multiple choice questions.')
    # -- matrix
    matrix_subtype = fields.Selection([
        ('simple', 'One choice per row'),
        ('multiple', 'Multiple choices per row')], string='Matrix Type', default='simple')
    matrix_row_ids = fields.One2many(
        'survey.question.answer', 'matrix_question_id', string='Matrix Rows', copy=True,
        help='Labels used for proposed choices: rows of matrix')
    # -- display & timing options
    column_nb = fields.Selection([
        ('12', '1'), ('6', '2'), ('4', '3'), ('3', '4'), ('2', '6')],
        string='Number of columns', default='12',
        help='These options refer to col-xx-[12|6|4|3|2] classes in Bootstrap for dropdown-based simple and multiple choice questions.')
    is_time_limited = fields.Boolean("The question is limited in time",
        help="Currently only supported for live sessions.")
    time_limit = fields.Integer("Time limit (seconds)")
    # -- comments (simple choice, multiple choice, matrix (without count as an answer))
    comments_allowed = fields.Boolean('Show Comments Field')
    comments_message = fields.Char('Comment Message', translate=True, default=lambda self: _("If other, please specify:"))
    comment_count_as_answer = fields.Boolean('Comment Field is an Answer Choice')
    # question validation
    validation_required = fields.Boolean('Validate entry')
    validation_email = fields.Boolean('Input must be an email')
    validation_length_min = fields.Integer('Minimum Text Length', default=0)
    validation_length_max = fields.Integer('Maximum Text Length', default=0)
    validation_min_float_value = fields.Float('Minimum value', default=0.0)
    validation_max_float_value = fields.Float('Maximum value', default=0.0)
    validation_min_date = fields.Date('Minimum Date')
    validation_max_date = fields.Date('Maximum Date')
    validation_min_datetime = fields.Datetime('Minimum Datetime')
    validation_max_datetime = fields.Datetime('Maximum Datetime')
    validation_error_msg = fields.Char('Validation Error message', translate=True, default=lambda self: _("The answer you entered is not valid."))
    constr_mandatory = fields.Boolean('Mandatory Answer')
    constr_error_msg = fields.Char('Error message', translate=True, default=lambda self: _("This question requires an answer."))
    # answers
    user_input_line_ids = fields.One2many(
        'survey.user_input.line', 'question_id', string='Answers',
        domain=[('skipped', '=', False)], groups='survey.group_survey_user')

    # Conditional display
    is_conditional = fields.Boolean(
        string='Conditional Display', copy=False, help="""If checked, this question will be displayed only 
        if the specified conditional answer have been selected in a previous question""")
    triggering_question_id = fields.Many2one(
        'survey.question', string="Triggering Question", copy=False, compute="_compute_triggering_question_id",
        store=True, readonly=False, help="Question containing the triggering answer to display the current question.",
        domain="""[('survey_id', '=', survey_id),
                 '&', ('question_type', 'in', ['simple_choice', 'multiple_choice']),
                 '|',
                     ('sequence', '<', sequence),
                     '&', ('sequence', '=', sequence), ('id', '<', id)]""")
    triggering_answer_id = fields.Many2one(
        'survey.question.answer', string="Triggering Answer", copy=False, compute="_compute_triggering_answer_id",
        store=True, readonly=False, help="Answer that will trigger the display of the current question.",
        domain="[('question_id', '=', triggering_question_id)]")

    _sql_constraints = [
        ('positive_len_min', 'CHECK (validation_length_min >= 0)', 'A length must be positive!'),
        ('positive_len_max', 'CHECK (validation_length_max >= 0)', 'A length must be positive!'),
        ('validation_length', 'CHECK (validation_length_min <= validation_length_max)', 'Max length cannot be smaller than min length!'),
        ('validation_float', 'CHECK (validation_min_float_value <= validation_max_float_value)', 'Max value cannot be smaller than min value!'),
        ('validation_date', 'CHECK (validation_min_date <= validation_max_date)', 'Max date cannot be smaller than min date!'),
        ('validation_datetime', 'CHECK (validation_min_datetime <= validation_max_datetime)', 'Max datetime cannot be smaller than min datetime!'),
        ('positive_answer_score', 'CHECK (answer_score >= 0)', 'An answer score for a non-multiple choice question cannot be negative!'),
        ('scored_datetime_have_answers', "CHECK (is_scored_question != True OR question_type != 'datetime' OR answer_datetime is not null)",
            'All "Is a scored question = True" and "Question Type: Datetime" questions need an answer'),
        ('scored_date_have_answers', "CHECK (is_scored_question != True OR question_type != 'date' OR answer_date is not null)",
            'All "Is a scored question = True" and "Question Type: Date" questions need an answer')
    ]

    @api.depends('is_page')
    def _compute_question_type(self):
        for question in self:
            if not question.question_type or question.is_page:
                question.question_type = False

    @api.depends('survey_id.question_and_page_ids.is_page', 'survey_id.question_and_page_ids.sequence')
    def _compute_question_ids(self):
        """Will take all questions of the survey for which the index is higher than the index of this page
        and lower than the index of the next page."""
        for question in self:
            if question.is_page:
                next_page_index = False
                for page in question.survey_id.page_ids:
                    if page._index() > question._index():
                        next_page_index = page._index()
                        break

                question.question_ids = question.survey_id.question_ids.filtered(
                    lambda q: q._index() > question._index() and (not next_page_index or q._index() < next_page_index)
                )
            else:
                question.question_ids = self.env['survey.question']

    @api.depends('survey_id.question_and_page_ids.is_page', 'survey_id.question_and_page_ids.sequence')
    def _compute_page_id(self):
        """Will find the page to which this question belongs to by looking inside the corresponding survey"""
        for question in self:
            if question.is_page:
                question.page_id = None
            else:
                page = None
                for q in question.survey_id.question_and_page_ids.sorted():
                    if q == question:
                        break
                    if q.is_page:
                        page = q
                question.page_id = page

    @api.depends('question_type', 'validation_email')
    def _compute_save_as_email(self):
        for question in self:
            if question.question_type != 'char_box' or not question.validation_email:
                question.save_as_email = False

    @api.depends('question_type')
    def _compute_save_as_nickname(self):
        for question in self:
            if question.question_type != 'char_box':
                question.save_as_nickname = False

    @api.depends('is_conditional')
    def _compute_triggering_question_id(self):
        """ Used as an 'onchange' : Reset the triggering question if user uncheck 'Conditional Display'
            Avoid CacheMiss : set the value to False if the value is not set yet."""
        for question in self:
            if not question.is_conditional or question.triggering_question_id is None:
                question.triggering_question_id = False

    @api.depends('triggering_question_id')
    def _compute_triggering_answer_id(self):
        """ Used as an 'onchange' : Reset the triggering answer if user unset or change the triggering question
            or uncheck 'Conditional Display'.
            Avoid CacheMiss : set the value to False if the value is not set yet."""
        for question in self:
            if not question.triggering_question_id \
                    or question.triggering_question_id != question.triggering_answer_id.question_id\
                    or question.triggering_answer_id is None:
                question.triggering_answer_id = False

    @api.depends('question_type', 'scoring_type', 'answer_date', 'answer_datetime', 'answer_numerical_box')
    def _compute_is_scored_question(self):
        """ Computes whether a question "is scored" or not. Handles following cases:
          - inconsistent Boolean=None edge case that breaks tests => False
          - survey is not scored => False
          - 'date'/'datetime'/'numerical_box' question types w/correct answer => True
            (implied without user having to activate, except for numerical whose correct value is 0.0)
          - 'simple_choice / multiple_choice': set to True even if logic is a bit different (coming from answers)
          - question_type isn't scoreable (note: choice questions scoring logic handled separately) => False
        """
        for question in self:
            if question.is_scored_question is None or question.scoring_type == 'no_scoring':
                question.is_scored_question = False
            elif question.question_type == 'date':
                question.is_scored_question = bool(question.answer_date)
            elif question.question_type == 'datetime':
                question.is_scored_question = bool(question.answer_datetime)
            elif question.question_type == 'numerical_box' and question.answer_numerical_box:
                question.is_scored_question = True
            elif question.question_type in ['simple_choice', 'multiple_choice']:
                question.is_scored_question = True
            else:
                question.is_scored_question = False

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.ondelete(at_uninstall=False)
    def _unlink_except_live_sessions_in_progress(self):
        running_surveys = self.survey_id.filtered(lambda survey: survey.session_state == 'in_progress')
        if running_surveys:
            raise UserError(_(
                'You cannot delete questions from surveys "%(survey_names)s" while live sessions are in progress.',
                survey_names=', '.join(running_surveys.mapped('title')),
            ))

    # ------------------------------------------------------------
    # VALIDATION
    # ------------------------------------------------------------

    def validate_question(self, answer, comment=None):
        """ Validate question, depending on question type and parameters
         for simple choice, text, date and number, answer is simply the answer of the question.
         For other multiple choices questions, answer is a list of answers (the selected choices
         or a list of selected answers per question -for matrix type-):
            - Simple answer : answer = 'example' or 2 or question_answer_id or 2019/10/10
            - Multiple choice : answer = [question_answer_id1, question_answer_id2, question_answer_id3]
            - Matrix: answer = { 'rowId1' : [colId1, colId2,...], 'rowId2' : [colId1, colId3, ...] }

         return dict {question.id (int): error (str)} -> empty dict if no validation error.
         """
        self.ensure_one()
        if isinstance(answer, str):
            answer = answer.strip()
        # Empty answer to mandatory question
        if self.constr_mandatory and not answer and self.question_type not in ['simple_choice', 'multiple_choice']:
            return {self.id: self.constr_error_msg}

        # because in choices question types, comment can count as answer
        if answer or self.question_type in ['simple_choice', 'multiple_choice']:
            if self.question_type == 'char_box':
                return self._validate_char_box(answer)
            elif self.question_type == 'numerical_box':
                return self._validate_numerical_box(answer)
            elif self.question_type in ['date', 'datetime']:
                return self._validate_date(answer)
            elif self.question_type in ['simple_choice', 'multiple_choice']:
                return self._validate_choice(answer, comment)
            elif self.question_type == 'matrix':
                return self._validate_matrix(answer)
        return {}

    def _validate_char_box(self, answer):
        # Email format validation
        # all the strings of the form "<something>@<anything>.<extension>" will be accepted
        if self.validation_email:
            if not tools.email_normalize(answer):
                return {self.id: _('This answer must be an email address')}

        # Answer validation (if properly defined)
        # Length of the answer must be in a range
        if self.validation_required:
            if not (self.validation_length_min <= len(answer) <= self.validation_length_max):
                return {self.id: self.validation_error_msg}
        return {}

    def _validate_numerical_box(self, answer):
        try:
            floatanswer = float(answer)
        except ValueError:
            return {self.id: _('This is not a number')}

        if self.validation_required:
            # Answer is not in the right range
            with tools.ignore(Exception):
                if not (self.validation_min_float_value <= floatanswer <= self.validation_max_float_value):
                    return {self.id: self.validation_error_msg}
        return {}

    def _validate_date(self, answer):
        isDatetime = self.question_type == 'datetime'
        # Checks if user input is a date
        try:
            dateanswer = fields.Datetime.from_string(answer) if isDatetime else fields.Date.from_string(answer)
        except ValueError:
            return {self.id: _('This is not a date')}
        if self.validation_required:
            # Check if answer is in the right range
            if isDatetime:
                min_date = fields.Datetime.from_string(self.validation_min_datetime)
                max_date = fields.Datetime.from_string(self.validation_max_datetime)
                dateanswer = fields.Datetime.from_string(answer)
            else:
                min_date = fields.Date.from_string(self.validation_min_date)
                max_date = fields.Date.from_string(self.validation_max_date)
                dateanswer = fields.Date.from_string(answer)

            if (min_date and max_date and not (min_date <= dateanswer <= max_date))\
                    or (min_date and not min_date <= dateanswer)\
                    or (max_date and not dateanswer <= max_date):
                return {self.id: self.validation_error_msg}
        return {}

    def _validate_choice(self, answer, comment):
        # Empty comment
        if self.constr_mandatory \
                and not answer \
                and not (self.comments_allowed and self.comment_count_as_answer and comment):
            return {self.id: self.constr_error_msg}
        return {}

    def _validate_matrix(self, answers):
        # Validate that each line has been answered
        if self.constr_mandatory and len(self.matrix_row_ids) != len(answers):
            return {self.id: self.constr_error_msg}
        return {}

    def _index(self):
        """We would normally just use the 'sequence' field of questions BUT, if the pages and questions are
        created without ever moving records around, the sequence field can be set to 0 for all the questions.

        However, the order of the recordset is always correct so we can rely on the index method."""
        self.ensure_one()
        return list(self.survey_id.question_and_page_ids).index(self)

    # ------------------------------------------------------------
    # STATISTICS / REPORTING
    # ------------------------------------------------------------

    def _prepare_statistics(self, user_input_lines):
        """ Compute statistical data for questions by counting number of vote per choice on basis of filter """
        all_questions_data = []
        for question in self:
            question_data = {'question': question, 'is_page': question.is_page}

            if question.is_page:
                all_questions_data.append(question_data)
                continue

            # fetch answer lines, separate comments from real answers
            all_lines = user_input_lines.filtered(lambda line: line.question_id == question)
            if question.question_type in ['simple_choice', 'multiple_choice', 'matrix']:
                answer_lines = all_lines.filtered(
                    lambda line: line.answer_type == 'suggestion' or (
                        line.skipped and not line.answer_type) or (
                        line.answer_type == 'char_box' and question.comment_count_as_answer)
                    )
                comment_line_ids = all_lines.filtered(lambda line: line.answer_type == 'char_box')
            else:
                answer_lines = all_lines
                comment_line_ids = self.env['survey.user_input.line']
            skipped_lines = answer_lines.filtered(lambda line: line.skipped)
            done_lines = answer_lines - skipped_lines
            question_data.update(
                answer_line_ids=answer_lines,
                answer_line_done_ids=done_lines,
                answer_input_done_ids=done_lines.mapped('user_input_id'),
                answer_input_skipped_ids=skipped_lines.mapped('user_input_id'),
                comment_line_ids=comment_line_ids)
            question_data.update(question._get_stats_summary_data(answer_lines))

            # prepare table and graph data
            table_data, graph_data = question._get_stats_data(answer_lines)
            question_data['table_data'] = table_data
            question_data['graph_data'] = json.dumps(graph_data)

            all_questions_data.append(question_data)
        return all_questions_data

    def _get_stats_data(self, user_input_lines):
        if self.question_type == 'simple_choice':
            return self._get_stats_data_answers(user_input_lines)
        elif self.question_type == 'multiple_choice':
            table_data, graph_data = self._get_stats_data_answers(user_input_lines)
            return table_data, [{'key': self.title, 'values': graph_data}]
        elif self.question_type == 'matrix':
            return self._get_stats_graph_data_matrix(user_input_lines)
        return [line for line in user_input_lines], []

    def _get_stats_data_answers(self, user_input_lines):
        """ Statistics for question.answer based questions (simple choice, multiple
        choice.). A corner case with a void record survey.question.answer is added
        to count comments that should be considered as valid answers. This small hack
        allow to have everything available in the same standard structure. """
        suggested_answers = [answer for answer in self.mapped('suggested_answer_ids')]
        if self.comment_count_as_answer:
            suggested_answers += [self.env['survey.question.answer']]

        count_data = dict.fromkeys(suggested_answers, 0)
        for line in user_input_lines:
            if line.suggested_answer_id in count_data\
               or (line.value_char_box and self.comment_count_as_answer):
                count_data[line.suggested_answer_id] += 1

        table_data = [{
            'value': _('Other (see comments)') if not sug_answer else sug_answer.value,
            'suggested_answer': sug_answer,
            'count': count_data[sug_answer]
            }
            for sug_answer in suggested_answers]
        graph_data = [{
            'text': _('Other (see comments)') if not sug_answer else sug_answer.value,
            'count': count_data[sug_answer]
            }
            for sug_answer in suggested_answers]

        return table_data, graph_data

    def _get_stats_graph_data_matrix(self, user_input_lines):
        suggested_answers = self.mapped('suggested_answer_ids')
        matrix_rows = self.mapped('matrix_row_ids')

        count_data = dict.fromkeys(itertools.product(matrix_rows, suggested_answers), 0)
        for line in user_input_lines:
            if line.matrix_row_id and line.suggested_answer_id:
                count_data[(line.matrix_row_id, line.suggested_answer_id)] += 1

        table_data = [{
            'row': row,
            'columns': [{
                'suggested_answer': sug_answer,
                'count': count_data[(row, sug_answer)]
            } for sug_answer in suggested_answers],
        } for row in matrix_rows]
        graph_data = [{
            'key': sug_answer.value,
            'values': [{
                'text': row.value,
                'count': count_data[(row, sug_answer)]
                }
                for row in matrix_rows
            ]
        } for sug_answer in suggested_answers]

        return table_data, graph_data

    def _get_stats_summary_data(self, user_input_lines):
        stats = {}
        if self.question_type in ['simple_choice', 'multiple_choice']:
            stats.update(self._get_stats_summary_data_choice(user_input_lines))
        elif self.question_type == 'numerical_box':
            stats.update(self._get_stats_summary_data_numerical(user_input_lines))

        if self.question_type in ['numerical_box', 'date', 'datetime']:
            stats.update(self._get_stats_summary_data_scored(user_input_lines))
        return stats

    def _get_stats_summary_data_choice(self, user_input_lines):
        right_inputs, partial_inputs = self.env['survey.user_input'], self.env['survey.user_input']
        right_answers = self.suggested_answer_ids.filtered(lambda label: label.is_correct)
        if self.question_type == 'multiple_choice':
            for user_input, lines in tools.groupby(user_input_lines, operator.itemgetter('user_input_id')):
                user_input_answers = self.env['survey.user_input.line'].concat(*lines).filtered(lambda l: l.answer_is_correct).mapped('suggested_answer_id')
                if user_input_answers and user_input_answers < right_answers:
                    partial_inputs += user_input
                elif user_input_answers:
                    right_inputs += user_input
        else:
            right_inputs = user_input_lines.filtered(lambda line: line.answer_is_correct).mapped('user_input_id')
        return {
            'right_answers': right_answers,
            'right_inputs_count': len(right_inputs),
            'partial_inputs_count': len(partial_inputs),
        }

    def _get_stats_summary_data_numerical(self, user_input_lines):
        all_values = user_input_lines.filtered(lambda line: not line.skipped).mapped('value_numerical_box')
        lines_sum = sum(all_values)
        return {
            'numerical_max': max(all_values, default=0),
            'numerical_min': min(all_values, default=0),
            'numerical_average': round(lines_sum / (len(all_values) or 1), 2),
        }

    def _get_stats_summary_data_scored(self, user_input_lines):
        return {
            'common_lines': collections.Counter(
                user_input_lines.filtered(lambda line: not line.skipped).mapped('value_%s' % self.question_type)
            ).most_common(5) if self.question_type != 'datetime' else [],
            'right_inputs_count': len(user_input_lines.filtered(lambda line: line.answer_is_correct).mapped('user_input_id'))
        }


class SurveyQuestionAnswer(models.Model):
    """ A preconfigured answer for a question. This model stores values used
    for

      * simple choice, multiple choice: proposed values for the selection /
        radio;
      * matrix: row and column values;

    """
    _name = 'survey.question.answer'
    _rec_name = 'value'
    _order = 'sequence, id'
    _description = 'Survey Label'

    question_id = fields.Many2one('survey.question', string='Question', ondelete='cascade')
    matrix_question_id = fields.Many2one('survey.question', string='Question (as matrix row)', ondelete='cascade')
    sequence = fields.Integer('Label Sequence order', default=10)
    value = fields.Char('Suggested value', translate=True, required=True)
    value_image = fields.Image('Image', max_width=256, max_height=256)
    is_correct = fields.Boolean('Is a correct answer')
    answer_score = fields.Float('Score for this choice', help="A positive score indicates a correct choice; a negative or null score indicates a wrong answer")

    @api.constrains('question_id', 'matrix_question_id')
    def _check_question_not_empty(self):
        """Ensure that field question_id XOR field matrix_question_id is not null"""
        for label in self:
            if not bool(label.question_id) != bool(label.matrix_question_id):
                raise ValidationError(_("A label must be attached to only one question."))

```

## File: models\survey_survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import random
import uuid
import werkzeug

from odoo import api, exceptions, fields, models, _
from odoo.exceptions import AccessError, UserError
from odoo.osv import expression
from odoo.tools import clean_context, is_html_empty


class Survey(models.Model):
    """ Settings for a multi-page/multi-question survey. Each survey can have one or more attached pages
    and each page can display one or more questions. """
    _name = 'survey.survey'
    _description = 'Survey'
    _rec_name = 'title'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    def _get_default_access_token(self):
        return str(uuid.uuid4())

    def _get_default_session_code(self):
        """ Attempt to generate a session code for our survey.
        The method will first try to generate 20 codes with 4 digits each and check if any are colliding.
        If we have at least one non-colliding code, we use it.
        If all 20 generated codes are colliding, we try with 20 codes of 5 digits,
        then 6, ... up to 10 digits. """

        for digits_count in range(4, 10):
            range_lower_bound = 1 * (10 ** (digits_count - 1))
            range_upper_bound = (range_lower_bound * 10) - 1
            code_candidates = set([str(random.randint(range_lower_bound, range_upper_bound)) for i in range(20)])
            colliding_codes = self.sudo().search_read(
                [('session_code', 'in', list(code_candidates))],
                ['session_code']
            )
            code_candidates -= set([colliding_code['session_code'] for colliding_code in colliding_codes])
            if code_candidates:
                return list(code_candidates)[0]

        return False  # could not generate a code

    # description
    title = fields.Char('Survey Title', required=True, translate=True)
    color = fields.Integer('Color Index', default=0)
    description = fields.Html(
        "Description", translate=True, sanitize=False,  # TDE FIXME: find a way to authorize videos
        help="The description will be displayed on the home page of the survey. You can use this to give the purpose and guidelines to your candidates before they start it.")
    description_done = fields.Html(
        "End Message", translate=True,
        help="This message will be displayed when survey is completed")
    background_image = fields.Binary("Background Image")
    active = fields.Boolean("Active", default=True)
    user_id = fields.Many2one('res.users', string='Responsible', tracking=True, default=lambda self: self.env.user)
    # questions
    question_and_page_ids = fields.One2many('survey.question', 'survey_id', string='Sections and Questions', copy=True)
    page_ids = fields.One2many('survey.question', string='Pages', compute="_compute_page_and_question_ids")
    question_ids = fields.One2many('survey.question', string='Questions', compute="_compute_page_and_question_ids")
    questions_layout = fields.Selection([
        ('one_page', 'One page with all the questions'),
        ('page_per_section', 'One page per section'),
        ('page_per_question', 'One page per question')],
        string="Layout", required=True, default='one_page')
    questions_selection = fields.Selection([
        ('all', 'All questions'),
        ('random', 'Randomized per section')],
        string="Selection", required=True, default='all',
        help="If randomized is selected, you can configure the number of random questions by section. This mode is ignored in live session.")
    progression_mode = fields.Selection([
        ('percent', 'Percentage'),
        ('number', 'Number')], string='Progression Mode', default='percent',
        help="If Number is selected, it will display the number of questions answered on the total number of question to answer.")
    # attendees
    user_input_ids = fields.One2many('survey.user_input', 'survey_id', string='User responses', readonly=True, groups='survey.group_survey_user')
    # security / access
    access_mode = fields.Selection([
        ('public', 'Anyone with the link'),
        ('token', 'Invited people only')], string='Access Mode',
        default='public', required=True)
    access_token = fields.Char('Access Token', default=lambda self: self._get_default_access_token(), copy=False)
    users_login_required = fields.Boolean('Login Required', help="If checked, users have to login before answering even with a valid token.")
    users_can_go_back = fields.Boolean('Users can go back', help="If checked, users can go back to previous pages.")
    users_can_signup = fields.Boolean('Users can signup', compute='_compute_users_can_signup')
    # statistics
    answer_count = fields.Integer("Registered", compute="_compute_survey_statistic")
    answer_done_count = fields.Integer("Attempts", compute="_compute_survey_statistic")
    answer_score_avg = fields.Float("Avg Score %", compute="_compute_survey_statistic")
    answer_duration_avg = fields.Float("Average Duration", compute="_compute_answer_duration_avg", help="Average duration of the survey (in hours)")
    success_count = fields.Integer("Success", compute="_compute_survey_statistic")
    success_ratio = fields.Integer("Success Ratio", compute="_compute_survey_statistic")
    # scoring
    scoring_type = fields.Selection([
        ('no_scoring', 'No scoring'),
        ('scoring_with_answers', 'Scoring with answers at the end'),
        ('scoring_without_answers', 'Scoring without answers at the end')],
        string="Scoring", required=True, default='no_scoring')
    scoring_success_min = fields.Float('Success %', default=80.0)
    # attendees context: attempts and time limitation
    is_attempts_limited = fields.Boolean('Limited number of attempts', help="Check this option if you want to limit the number of attempts per user",
                                         compute="_compute_is_attempts_limited", store=True, readonly=False)
    attempts_limit = fields.Integer('Number of attempts', default=1)
    is_time_limited = fields.Boolean('The survey is limited in time')
    time_limit = fields.Float("Time limit (minutes)", default=10)
    # certification
    certification = fields.Boolean('Is a Certification', compute='_compute_certification',
                                   readonly=False, store=True)
    certification_mail_template_id = fields.Many2one(
        'mail.template', 'Email Template',
        domain="[('model', '=', 'survey.user_input')]",
        help="Automated email sent to the user when he succeeds the certification, containing his certification document.")
    certification_report_layout = fields.Selection([
        ('modern_purple', 'Modern Purple'),
        ('modern_blue', 'Modern Blue'),
        ('modern_gold', 'Modern Gold'),
        ('classic_purple', 'Classic Purple'),
        ('classic_blue', 'Classic Blue'),
        ('classic_gold', 'Classic Gold')],
        string='Certification template', default='modern_purple')
    # Certification badge
    #   certification_badge_id_dummy is used to have two different behaviours in the form view :
    #   - If the certification badge is not set, show certification_badge_id and only display create option in the m2o
    #   - If the certification badge is set, show certification_badge_id_dummy in 'no create' mode.
    #       So it can be edited but not removed or replaced.
    certification_give_badge = fields.Boolean('Give Badge', compute='_compute_certification_give_badge',
                                              readonly=False, store=True, copy=False)
    certification_badge_id = fields.Many2one('gamification.badge', 'Certification Badge', copy=False)
    certification_badge_id_dummy = fields.Many2one(related='certification_badge_id', string='Certification Badge ')
    # live sessions
    session_state = fields.Selection([
        ('ready', 'Ready'),
        ('in_progress', 'In Progress'),
        ], string="Session State", copy=False)
    session_code = fields.Char('Session Code', default=lambda self: self._get_default_session_code(), copy=False,
        help="This code will be used by your attendees to reach your session. Feel free to customize it however you like!")
    session_link = fields.Char('Session Link', compute='_compute_session_link')
    # live sessions - current question fields
    session_question_id = fields.Many2one('survey.question', string="Current Question", copy=False,
        help="The current question of the survey session.")
    session_start_time = fields.Datetime("Current Session Start Time", copy=False)
    session_question_start_time = fields.Datetime("Current Question Start Time", copy=False,
        help="The time at which the current question has started, used to handle the timer for attendees.")
    session_answer_count = fields.Integer("Answers Count", compute='_compute_session_answer_count')
    session_question_answer_count = fields.Integer("Question Answers Count", compute='_compute_session_question_answer_count')
    # live sessions - settings
    session_show_leaderboard = fields.Boolean("Show Session Leaderboard", compute='_compute_session_show_leaderboard',
        help="Whether or not we want to show the attendees leaderboard for this survey.")
    session_speed_rating = fields.Boolean("Reward quick answers", help="Attendees get more points if they answer quickly")
    # conditional questions management
    has_conditional_questions = fields.Boolean("Contains conditional questions", compute="_compute_has_conditional_questions")

    _sql_constraints = [
        ('access_token_unique', 'unique(access_token)', 'Access token should be unique'),
        ('session_code_unique', 'unique(session_code)', 'Session code should be unique'),
        ('certification_check', "CHECK( scoring_type!='no_scoring' OR certification=False )",
            'You can only create certifications for surveys that have a scoring mechanism.'),
        ('scoring_success_min_check', "CHECK( scoring_success_min IS NULL OR (scoring_success_min>=0 AND scoring_success_min<=100) )",
            'The percentage of success has to be defined between 0 and 100.'),
        ('time_limit_check', "CHECK( (is_time_limited=False) OR (time_limit is not null AND time_limit > 0) )",
            'The time limit needs to be a positive number if the survey is time limited.'),
        ('attempts_limit_check', "CHECK( (is_attempts_limited=False) OR (attempts_limit is not null AND attempts_limit > 0) )",
            'The attempts limit needs to be a positive number if the survey has a limited number of attempts.'),
        ('badge_uniq', 'unique (certification_badge_id)', "The badge for each survey should be unique!"),
    ]

    def _compute_users_can_signup(self):
        signup_allowed = self.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'
        for survey in self:
            survey.users_can_signup = signup_allowed

    @api.depends('user_input_ids.state', 'user_input_ids.test_entry', 'user_input_ids.scoring_percentage', 'user_input_ids.scoring_success')
    def _compute_survey_statistic(self):
        default_vals = {
            'answer_count': 0, 'answer_done_count': 0, 'success_count': 0,
            'answer_score_avg': 0.0, 'success_ratio': 0.0
        }
        stat = dict((cid, dict(default_vals, answer_score_avg_total=0.0)) for cid in self.ids)
        UserInput = self.env['survey.user_input']
        base_domain = ['&', ('survey_id', 'in', self.ids), ('test_entry', '!=', True)]

        read_group_res = UserInput.read_group(base_domain, ['survey_id', 'state'], ['survey_id', 'state', 'scoring_percentage', 'scoring_success'], lazy=False)
        for item in read_group_res:
            stat[item['survey_id'][0]]['answer_count'] += item['__count']
            stat[item['survey_id'][0]]['answer_score_avg_total'] += item['scoring_percentage']
            if item['state'] == 'done':
                stat[item['survey_id'][0]]['answer_done_count'] += item['__count']
            if item['scoring_success']:
                stat[item['survey_id'][0]]['success_count'] += item['__count']

        for survey_stats in stat.values():
            avg_total = survey_stats.pop('answer_score_avg_total')
            survey_stats['answer_score_avg'] = avg_total / (survey_stats['answer_done_count'] or 1)
            survey_stats['success_ratio'] = (survey_stats['success_count'] / (survey_stats['answer_done_count'] or 1.0))*100

        for survey in self:
            survey.update(stat.get(survey._origin.id, default_vals))

    @api.depends('user_input_ids.survey_id', 'user_input_ids.start_datetime', 'user_input_ids.end_datetime')
    def _compute_answer_duration_avg(self):
        result_per_survey_id = {}
        if self.ids:
            self.env.cr.execute(
                """SELECT survey_id,
                          avg((extract(epoch FROM end_datetime)) - (extract (epoch FROM start_datetime)))
                     FROM survey_user_input
                    WHERE survey_id = any(%s) AND state = 'done'
                          AND end_datetime IS NOT NULL
                          AND start_datetime IS NOT NULL
                 GROUP BY survey_id""",
                [self.ids]
            )
            result_per_survey_id = dict(self.env.cr.fetchall())

        for survey in self:
            # as avg returns None if nothing found, set 0 if it's the case.
            survey.answer_duration_avg = (result_per_survey_id.get(survey.id) or 0) / 3600


    @api.depends('question_and_page_ids')
    def _compute_page_and_question_ids(self):
        for survey in self:
            survey.page_ids = survey.question_and_page_ids.filtered(lambda question: question.is_page)
            survey.question_ids = survey.question_and_page_ids - survey.page_ids

    @api.depends('users_login_required', 'access_mode')
    def _compute_is_attempts_limited(self):
        for survey in self:
            if not survey.is_attempts_limited or \
               (survey.access_mode == 'public' and not survey.users_login_required):
                survey.is_attempts_limited = False

    @api.depends('session_start_time', 'user_input_ids')
    def _compute_session_answer_count(self):
        """ We have to loop since our result is dependent of the survey.session_start_time.
        This field is currently used to display the count about a single survey, in the
        context of sessions, so it should not matter too much. """

        for survey in self:
            answer_count = 0
            input_count = self.env['survey.user_input'].read_group(
                [('survey_id', '=', survey.id),
                 ('is_session_answer', '=', True),
                 ('state', '!=', 'done'),
                 ('create_date', '>=', survey.session_start_time)],
                ['create_uid:count'],
                ['survey_id'],
            )
            if input_count:
                answer_count = input_count[0].get('create_uid', 0)

            survey.session_answer_count = answer_count

    @api.depends('session_question_id', 'session_start_time', 'user_input_ids.user_input_line_ids')
    def _compute_session_question_answer_count(self):
        """ We have to loop since our result is dependent of the survey.session_question_id and
        the survey.session_start_time.
        This field is currently used to display the count about a single survey, in the
        context of sessions, so it should not matter too much. """
        for survey in self:
            answer_count = 0
            input_line_count = self.env['survey.user_input.line'].read_group(
                [('question_id', '=', survey.session_question_id.id),
                 ('survey_id', '=', survey.id),
                 ('create_date', '>=', survey.session_start_time)],
                ['user_input_id:count_distinct'],
                ['question_id'],
            )
            if input_line_count:
                answer_count = input_line_count[0].get('user_input_id', 0)

            survey.session_question_answer_count = answer_count

    @api.depends('session_code')
    def _compute_session_link(self):
        for survey in self:
            if survey.session_code:
                survey.session_link = werkzeug.urls.url_join(
                    survey.get_base_url(),
                    '/s/%s' % survey.session_code)
            else:
                survey.session_link = werkzeug.urls.url_join(
                    survey.get_base_url(),
                    survey.get_start_url())

    @api.depends('scoring_type', 'question_and_page_ids.save_as_nickname')
    def _compute_session_show_leaderboard(self):
        for survey in self:
            survey.session_show_leaderboard = survey.scoring_type != 'no_scoring' and \
                any(question.save_as_nickname for question in survey.question_and_page_ids)

    @api.depends('question_and_page_ids.is_conditional')
    def _compute_has_conditional_questions(self):
        for survey in self:
            survey.has_conditional_questions = any(question.is_conditional for question in survey.question_and_page_ids)

    @api.depends('scoring_type')
    def _compute_certification(self):
        for survey in self:
            if not survey.certification or survey.scoring_type == 'no_scoring':
                survey.certification = False

    @api.depends('users_login_required', 'certification')
    def _compute_certification_give_badge(self):
        for survey in self:
            if not survey.certification_give_badge or \
               not survey.users_login_required or \
               not survey.certification:
                survey.certification_give_badge = False

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model
    def create(self, vals):
        survey = super(Survey, self).create(vals)
        if vals.get('certification_give_badge'):
            survey.sudo()._create_certification_badge_trigger()
        return survey

    def write(self, vals):
        result = super(Survey, self).write(vals)
        if 'certification_give_badge' in vals:
            return self.sudo().with_context(clean_context(self._context))._handle_certification_badges(vals)
        return result

    def copy_data(self, default=None):
        new_defaults = {'title': _("%s (copy)") % (self.title)}
        default = dict(new_defaults, **(default or {}))
        return super(Survey, self).copy_data(default)

    def toggle_active(self):
        super(Survey, self).toggle_active()
        activated = self.filtered(lambda survey: survey.active)
        activated.mapped('certification_badge_id').action_unarchive()
        (self - activated).mapped('certification_badge_id').action_archive()

    # ------------------------------------------------------------
    # ANSWER MANAGEMENT
    # ------------------------------------------------------------

    def _create_answer(self, user=False, partner=False, email=False, test_entry=False, check_attempts=True, **additional_vals):
        """ Main entry point to get a token back or create a new one. This method
        does check for current user access in order to explicitely validate
        security.

          :param user: target user asking for a token; it might be void or a
                       public user in which case an email is welcomed;
          :param email: email of the person asking the token is no user exists;
        """
        self.check_access_rights('read')
        self.check_access_rule('read')

        user_inputs = self.env['survey.user_input']
        for survey in self:
            if partner and not user and partner.user_ids:
                user = partner.user_ids[0]

            invite_token = additional_vals.pop('invite_token', False)
            survey._check_answer_creation(user, partner, email, test_entry=test_entry, check_attempts=check_attempts, invite_token=invite_token)
            answer_vals = {
                'survey_id': survey.id,
                'test_entry': test_entry,
                'is_session_answer': survey.session_state in ['ready', 'in_progress']
            }
            if survey.session_state == 'in_progress':
                # if the session is already in progress, the answer skips the 'new' state
                answer_vals.update({
                    'state': 'in_progress',
                    'start_datetime': fields.Datetime.now(),
                })
            if user and not user._is_public():
                answer_vals['partner_id'] = user.partner_id.id
                answer_vals['email'] = user.email
                answer_vals['nickname'] = user.name
            elif partner:
                answer_vals['partner_id'] = partner.id
                answer_vals['email'] = partner.email
                answer_vals['nickname'] = partner.name
            else:
                answer_vals['email'] = email
                answer_vals['nickname'] = email

            if invite_token:
                answer_vals['invite_token'] = invite_token
            elif survey.is_attempts_limited and survey.access_mode != 'public':
                # attempts limited: create a new invite_token
                # exception made for 'public' access_mode since the attempts pool is global because answers are
                # created every time the user lands on '/start'
                answer_vals['invite_token'] = self.env['survey.user_input']._generate_invite_token()

            answer_vals.update(additional_vals)
            user_inputs += user_inputs.create(answer_vals)

        for question in self.mapped('question_ids').filtered(
                lambda q: q.question_type == 'char_box' and (q.save_as_email or q.save_as_nickname)):
            for user_input in user_inputs:
                if question.save_as_email and user_input.email:
                    user_input.save_lines(question, user_input.email)
                if question.save_as_nickname and user_input.nickname:
                    user_input.save_lines(question, user_input.nickname)

        return user_inputs

    def _check_answer_creation(self, user, partner, email, test_entry=False, check_attempts=True, invite_token=False):
        """ Ensure conditions to create new tokens are met. """
        self.ensure_one()
        if test_entry:
            # the current user must have the access rights to survey
            if not user.has_group('survey.group_survey_user'):
                raise exceptions.UserError(_('Creating test token is not allowed for you.'))
        else:
            if not self.active:
                raise exceptions.UserError(_('Creating token for closed/archived surveys is not allowed.'))
            if self.access_mode == 'authentication':
                # signup possible -> should have at least a partner to create an account
                if self.users_can_signup and not user and not partner:
                    raise exceptions.UserError(_('Creating token for external people is not allowed for surveys requesting authentication.'))
                # no signup possible -> should be a not public user (employee or portal users)
                if not self.users_can_signup and (not user or user._is_public()):
                    raise exceptions.UserError(_('Creating token for external people is not allowed for surveys requesting authentication.'))
            if self.access_mode == 'internal' and (not user or not user.has_group('base.group_user')):
                raise exceptions.UserError(_('Creating token for anybody else than employees is not allowed for internal surveys.'))
            if check_attempts and not self._has_attempts_left(partner or (user and user.partner_id), email, invite_token):
                raise exceptions.UserError(_('No attempts left.'))

    def _prepare_user_input_predefined_questions(self):
        """ Will generate the questions for a randomized survey.
        It uses the random_questions_count of every sections of the survey to
        pick a random number of questions and returns the merged recordset """
        self.ensure_one()

        questions = self.env['survey.question']

        # First append questions without page
        for question in self.question_ids:
            if not question.page_id:
                questions |= question

        # Then, questions in sections

        for page in self.page_ids:
            if self.questions_selection == 'all':
                questions |= page.question_ids
            else:
                if page.random_questions_count > 0 and len(page.question_ids) > page.random_questions_count:
                    questions = questions.concat(*random.sample(page.question_ids, page.random_questions_count))
                else:
                    questions |= page.question_ids

        return questions

    def _can_go_back(self, answer, page_or_question):
        """ Check if the user can go back to the previous question/page for the currently
        viewed question/page.
        Back button needs to be configured on survey and, depending on the layout:
        - In 'page_per_section', we can go back if we're not on the first page
        - In 'page_per_question', we can go back if:
          - It is not a session answer (doesn't make sense to go back in session context)
          - We are not on the first question
          - The survey does not have pages OR this is not the first page of the survey
            (pages are displayed in 'page_per_question' layout when they have a description, see PR#44271)
        """
        self.ensure_one()

        if self.users_can_go_back and answer.state == 'in_progress':
            if self.questions_layout == 'page_per_section' and page_or_question != self.page_ids[0]:
                return True
            elif self.questions_layout == 'page_per_question' and \
                 not answer.is_session_answer and \
                 page_or_question != answer.predefined_question_ids[0] \
                 and (not self.page_ids or page_or_question != self.page_ids[0]):
                return True

        return False

    def _has_attempts_left(self, partner, email, invite_token):
        self.ensure_one()

        if (self.access_mode != 'public' or self.users_login_required) and self.is_attempts_limited:
            return self._get_number_of_attempts_lefts(partner, email, invite_token) > 0

        return True

    def _get_number_of_attempts_lefts(self, partner, email, invite_token):
        """ Returns the number of attempts left. """
        self.ensure_one()

        domain = [
            ('survey_id', '=', self.id),
            ('test_entry', '=', False),
            ('state', '=', 'done')
        ]

        if partner:
            domain = expression.AND([domain, [('partner_id', '=', partner.id)]])
        else:
            domain = expression.AND([domain, [('email', '=', email)]])

        if invite_token:
            domain = expression.AND([domain, [('invite_token', '=', invite_token)]])

        return self.attempts_limit - self.env['survey.user_input'].search_count(domain)

    # ------------------------------------------------------------
    # QUESTIONS MANAGEMENT
    # ------------------------------------------------------------

    @api.model
    def _get_pages_or_questions(self, user_input):
        """ Returns the pages or questions (depending on the layout) that will be shown
        to the user taking the survey.
        In 'page_per_question' layout, we also want to show pages that have a description. """

        result = self.env['survey.question']
        if self.questions_layout == 'page_per_section':
            result = self.page_ids
        elif self.questions_layout == 'page_per_question':
            if self.questions_selection == 'random' and not self.session_state:
                result = user_input.predefined_question_ids
            else:
                result = self._get_pages_and_questions_to_show()

        return result

    def _get_pages_and_questions_to_show(self):
        """
        :return: survey.question recordset excluding invalid conditional questions and pages without description
        """

        self.ensure_one()
        invalid_questions = self.env['survey.question']
        questions_and_valid_pages = self.question_and_page_ids.filtered(
            lambda question: not question.is_page or not is_html_empty(question.description))
        for question in questions_and_valid_pages.filtered(lambda q: q.is_conditional).sorted():
            trigger = question.triggering_question_id
            if (trigger in invalid_questions
                    or trigger.is_page
                    or trigger.question_type not in ['simple_choice', 'multiple_choice']
                    or not trigger.suggested_answer_ids
                    or trigger.sequence > question.sequence
                    or (trigger.sequence == question.sequence and trigger.id > question.id)):
                invalid_questions |= question
        return questions_and_valid_pages - invalid_questions

    def _get_next_page_or_question(self, user_input, page_or_question_id, go_back=False):
        """ Generalized logic to retrieve the next question or page to show on the survey.
        It's based on the page_or_question_id parameter, that is usually the currently displayed question/page.

        There is a special case when the survey is configured with conditional questions:
        - for "page_per_question" layout, the next question to display depends on the selected answers and
          the questions 'hierarchy'.
        - for "page_per_section" layout, before returning the result, we check that it contains at least a question
          (all section questions could be disabled based on previously selected answers)

        The whole logic is inverted if "go_back" is passed as True.

        As pages with description are considered as potential question to display, we show the page
        if it contains at least one active question or a description.

        :param user_input: user's answers
        :param page_or_question_id: current page or question id
        :param go_back: reverse the logic and get the PREVIOUS question/page
        :return: next or previous question/page
        """

        survey = user_input.survey_id
        pages_or_questions = survey._get_pages_or_questions(user_input)
        Question = self.env['survey.question']

        # Get Next
        if not go_back:
            if not pages_or_questions:
                return Question
            # First page
            if page_or_question_id == 0:
                return pages_or_questions[0]

        current_page_index = pages_or_questions.ids.index(page_or_question_id)

        # Get previous and we are on first page  OR Get Next and we are on last page
        if (go_back and current_page_index == 0) or (not go_back and current_page_index == len(pages_or_questions) - 1):
            return Question

        # Conditional Questions Management
        triggering_answer_by_question, triggered_questions_by_answer, selected_answers = user_input._get_conditional_values()
        inactive_questions = user_input._get_inactive_conditional_questions()
        if survey.questions_layout == 'page_per_question':
            question_candidates = pages_or_questions[0:current_page_index] if go_back \
                else pages_or_questions[current_page_index + 1:]
            for question in question_candidates.sorted(reverse=go_back):
                # pages with description are potential questions to display (are part of question_candidates)
                if question.is_page:
                    contains_active_question = any(sub_question not in inactive_questions for sub_question in question.question_ids)
                    is_description_section = not question.question_ids and not is_html_empty(question.description)
                    if contains_active_question or is_description_section:
                        return question
                else:
                    triggering_answer = triggering_answer_by_question.get(question)
                    if not triggering_answer or triggering_answer in selected_answers:
                        # question is visible because not conditioned or conditioned by a selected answer
                        return question
        elif survey.questions_layout == 'page_per_section':
            section_candidates = pages_or_questions[0:current_page_index] if go_back \
                else pages_or_questions[current_page_index + 1:]
            for section in section_candidates.sorted(reverse=go_back):
                contains_active_question = any(question not in inactive_questions for question in section.question_ids)
                is_description_section = not section.question_ids and not is_html_empty(section.description)
                if contains_active_question or is_description_section:
                    return section
            return Question

    def _is_first_page_or_question(self, page_or_question):
        """ This method checks if the given question or page is the first one to display.
            If the first section of the survey as a description, this will be the first screen to display.
            else, the first question will be the first screen to be displayed.
            This methods is used for survey session management where the host should not be able to go back on the
            first page or question."""
        first_section_has_description = self.page_ids and not is_html_empty(self.page_ids[0].description)
        is_first_page_or_question = (first_section_has_description and page_or_question == self.page_ids[0]) or \
            (not first_section_has_description and page_or_question == self.question_ids[0])
        return is_first_page_or_question

    def _is_last_page_or_question(self, user_input, page_or_question):
        """ This method checks if the given question or page is the last one.
        This includes conditional questions configuration. If the given question is normally not the last one but
        every following questions are inactive due to conditional questions configurations (and user choices),
        the given question will be the last one, except if the given question is conditioning at least
        one of the following questions.
        For section, we check in each following section if there is an active question.
        If yes, the given page is not the last one.
        """
        pages_or_questions = self._get_pages_or_questions(user_input)
        current_page_index = pages_or_questions.ids.index(page_or_question.id)
        next_page_or_question_candidates = pages_or_questions[current_page_index + 1:]
        if next_page_or_question_candidates:
            inactive_questions = user_input._get_inactive_conditional_questions()
            triggering_answer_by_question, triggered_questions_by_answer, selected_answers = user_input._get_conditional_values()
            if self.questions_layout == 'page_per_question':
                next_active_question = any(next_question not in inactive_questions for next_question in next_page_or_question_candidates)
                is_triggering_question = any(triggering_answer in triggered_questions_by_answer.keys() for triggering_answer in page_or_question.suggested_answer_ids)
                return not(next_active_question or is_triggering_question)
            elif self.questions_layout == 'page_per_section':
                is_triggering_section = False
                for question in page_or_question.question_ids:
                    if any(triggering_answer in triggered_questions_by_answer.keys() for triggering_answer in
                           question.suggested_answer_ids):
                        is_triggering_section = True
                        break
                next_active_question = False
                for section in next_page_or_question_candidates:
                    next_active_question = any(next_question not in inactive_questions for next_question in section.question_ids)
                    if next_active_question:
                        break
                return not(next_active_question or is_triggering_section)

        return True

    def _get_survey_questions(self, answer=None, page_id=None, question_id=None):
        """ Returns a tuple containing: the survey question and the passed question_id / page_id
        based on the question_layout and the fact that it's a session or not.

        Breakdown of use cases:
        - We are currently running a session
          We return the current session question and it's id
        - The layout is page_per_section
          We return the questions for that page and the passed page_id
        - The layout is page_per_question
          We return the question for the passed question_id and the question_id
        - The layout is one_page
          We return all the questions of the survey and None

        In addition, we cross the returned questions with the answer.predefined_question_ids,
        that allows to handle the randomization of questions. """

        questions, page_or_question_id = None, None

        if answer and answer.is_session_answer:
            return self.session_question_id, self.session_question_id.id
        if self.questions_layout == 'page_per_section':
            if not page_id:
                raise ValueError("Page id is needed for question layout 'page_per_section'")
            page_id = int(page_id)
            questions = self.env['survey.question'].sudo().search([('survey_id', '=', self.id), ('page_id', '=', page_id)])
            page_or_question_id = page_id
        elif self.questions_layout == 'page_per_question':
            if not question_id:
                raise ValueError("Question id is needed for question layout 'page_per_question'")
            question_id = int(question_id)
            questions = self.env['survey.question'].sudo().browse(question_id)
            page_or_question_id = question_id
        else:
            questions = self.question_ids

        # we need the intersection of the questions of this page AND the questions prepared for that user_input
        # (because randomized surveys do not use all the questions of every page)
        if answer:
            questions = questions & answer.predefined_question_ids
        return questions, page_or_question_id

    # ------------------------------------------------------------
    # CONDITIONAL QUESTIONS MANAGEMENT
    # ------------------------------------------------------------

    def _get_conditional_maps(self):
        triggering_answer_by_question = {}
        triggered_questions_by_answer = {}
        for question in self.question_ids:
            triggering_answer_by_question[question] = question.is_conditional and question.triggering_answer_id

            if question.is_conditional:
                if question.triggering_answer_id in triggered_questions_by_answer:
                    triggered_questions_by_answer[question.triggering_answer_id] |= question
                else:
                    triggered_questions_by_answer[question.triggering_answer_id] = question
        return triggering_answer_by_question, triggered_questions_by_answer

    # ------------------------------------------------------------
    # SESSIONS MANAGEMENT
    # ------------------------------------------------------------

    def _session_open(self):
        """ The session start is sudo'ed to allow survey user to manage sessions of surveys
        they do not own.

        We flush after writing to make sure it's updated before bus takes over. """

        if self.env.user.has_group('survey.group_survey_user'):
            self.sudo().write({'session_state': 'in_progress'})
            self.sudo().flush(['session_state'])

    def _get_session_next_question(self, go_back):
        self.ensure_one()

        if not self.question_ids or not self.env.user.has_group('survey.group_survey_user'):
            return

        most_voted_answers = self._get_session_most_voted_answers()
        return self._get_next_page_or_question(
            most_voted_answers,
            self.session_question_id.id if self.session_question_id else 0, go_back=go_back)

    def _get_session_most_voted_answers(self):
        """ In sessions of survey that has conditional questions, as the survey is passed at the same time by
        many users, we need to extract the most chosen answers, to determine the next questions to display. """

        # get user_inputs from current session
        current_user_inputs = self.user_input_ids.filtered(lambda input: input.create_date > self.session_start_time)
        current_user_input_lines = current_user_inputs.mapped('user_input_line_ids').filtered(lambda answer: answer.suggested_answer_id)

        # count the number of vote per answer
        votes_by_answer = dict.fromkeys(current_user_input_lines.mapped('suggested_answer_id'), 0)
        for answer in current_user_input_lines:
            votes_by_answer[answer.suggested_answer_id] += 1

        # extract most voted answer for each question
        most_voted_answer_by_questions = dict.fromkeys(current_user_input_lines.mapped('question_id'))
        for question in most_voted_answer_by_questions.keys():
            for answer in votes_by_answer.keys():
                if answer.question_id != question:
                    continue
                most_voted_answer = most_voted_answer_by_questions[question]
                if not most_voted_answer or votes_by_answer[most_voted_answer] < votes_by_answer[answer]:
                    most_voted_answer_by_questions[question] = answer

        # return a fake 'audience' user_input
        fake_user_input = self.env['survey.user_input'].new({
            'survey_id': self.id,
            'predefined_question_ids': [(6, 0, self._prepare_user_input_predefined_questions().ids)]
        })

        fake_user_input_lines = self.env['survey.user_input.line']
        for question, answer in most_voted_answer_by_questions.items():
            fake_user_input_lines |= self.env['survey.user_input.line'].new({
                'question_id': question.id,
                'suggested_answer_id': answer.id,
                'survey_id': self.id,
                'user_input_id': fake_user_input.id
            })

        return fake_user_input

    def _prepare_leaderboard_values(self):
        """" The leaderboard is descending and takes the total of the attendee points minus the
        current question score.
        We need both the total and the current question points to be able to show the attendees
        leaderboard and shift their position based on the score they have on the current question.
        This prepares a structure containing all the necessary data for the animations done on
        the frontend side.
        The leaderboard is sorted based on attendees score *before* the current question.
        The frontend will shift positions around accordingly. """

        self.ensure_one()

        leaderboard = self.env['survey.user_input'].search_read([
            ('survey_id', '=', self.id),
            ('create_date', '>=', self.session_start_time)
        ], [
            'id',
            'nickname',
            'scoring_total',
        ], limit=15, order="scoring_total desc")

        if leaderboard and self.session_state == 'in_progress' and \
           any(answer.answer_score for answer in self.session_question_id.suggested_answer_ids):
            question_scores = {}
            input_lines = self.env['survey.user_input.line'].search_read(
                    [('user_input_id', 'in', [score['id'] for score in leaderboard]),
                        ('question_id', '=', self.session_question_id.id)],
                    ['user_input_id', 'answer_score'])
            for input_line in input_lines:
                question_scores[input_line['user_input_id'][0]] = \
                    question_scores.get(input_line['user_input_id'][0], 0) + input_line['answer_score']

            score_position = 0
            for leaderboard_item in leaderboard:
                question_score = question_scores.get(leaderboard_item['id'], 0)
                leaderboard_item.update({
                    'updated_score': leaderboard_item['scoring_total'],
                    'scoring_total': leaderboard_item['scoring_total'] - question_score,
                    'leaderboard_position': score_position,
                    'max_question_score': sum(
                        score for score in self.session_question_id.suggested_answer_ids.mapped('answer_score')
                        if score > 0
                    ) or 1,
                    'question_score': question_score
                })
                score_position += 1
            leaderboard = sorted(
                leaderboard,
                key=lambda score: score['scoring_total'],
                reverse=True)

        return leaderboard


    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def action_send_survey(self):
        """ Open a window to compose an email, pre-filled with the survey message """
        # Ensure that this survey has at least one question.
        if not self.question_ids:
            raise UserError(_('You cannot send an invitation for a survey that has no questions.'))

        # Ensure that this survey has at least one section with question(s), if question layout is 'One page per section'.
        if self.questions_layout == 'page_per_section':
            if not self.page_ids:
                raise UserError(_('You cannot send an invitation for a "One page per section" survey if the survey has no sections.'))
            if not self.page_ids.mapped('question_ids'):
                raise UserError(_('You cannot send an invitation for a "One page per section" survey if the survey only contains empty sections.'))

        if not self.active:
            raise exceptions.UserError(_("You cannot send invitations for closed surveys."))

        template = self.env.ref('survey.mail_template_user_input_invite', raise_if_not_found=False)

        local_context = dict(
            self.env.context,
            default_survey_id=self.id,
            default_use_template=bool(template),
            default_template_id=template and template.id or False,
            notif_layout='mail.mail_notification_light',
        )
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'survey.invite',
            'target': 'new',
            'context': local_context,
        }

    def action_start_survey(self, answer=None):
        """ Open the website page with the survey form """
        self.ensure_one()
        url = '%s?%s' % (self.get_start_url(), werkzeug.urls.url_encode({'answer_token': answer and answer.access_token or None}))
        return {
            'type': 'ir.actions.act_url',
            'name': "Start Survey",
            'target': 'self',
            'url': url,
        }

    def action_print_survey(self, answer=None):
        """ Open the website page with the survey printable view """
        self.ensure_one()
        url = '%s?%s' % (self.get_print_url(), werkzeug.urls.url_encode({'answer_token': answer and answer.access_token or None}))
        return {
            'type': 'ir.actions.act_url',
            'name': "Print Survey",
            'target': 'self',
            'url': url
        }

    def action_result_survey(self):
        """ Open the website page with the survey results view """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'name': "Results of the Survey",
            'target': 'self',
            'url': '/survey/results/%s' % self.id
        }

    def action_test_survey(self):
        ''' Open the website page with the survey form into test mode'''
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'name': "Test Survey",
            'target': '_blank',
            'url': '/survey/test/%s' % self.access_token,
        }

    def action_survey_user_input_completed(self):
        action = self.env['ir.actions.act_window']._for_xml_id('survey.action_survey_user_input')
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_completed': 1,
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    def action_survey_user_input_certified(self):
        action = self.env['ir.actions.act_window']._for_xml_id('survey.action_survey_user_input')
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_scoring_success': 1,
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    def action_survey_user_input(self):
        action = self.env['ir.actions.act_window']._for_xml_id('survey.action_survey_user_input')
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    def action_survey_preview_certification_template(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': '_blank',
            'url': '/survey/%s/certification_preview' % (self.id)
        }

    def action_start_session(self):
        """ Sets the necessary fields for the session to take place and starts it.
        The write is sudo'ed because a survey user can start a session even if it's
        not his own survey. """

        if not self.env.user.has_group('survey.group_survey_user'):
            raise AccessError(_('Only survey users can manage sessions.'))

        self.ensure_one()
        self.sudo().write({
            'questions_layout': 'page_per_question',
            'session_start_time': fields.Datetime.now(),
            'session_question_id': None,
            'session_state': 'ready'
        })
        return self.action_open_session_manager()

    def action_open_session_manager(self):
        self.ensure_one()

        return {
            'type': 'ir.actions.act_url',
            'name': "Open Session Manager",
            'target': '_blank',
            'url': '/survey/session/manage/%s' % self.access_token
        }

    def action_end_session(self):
        """ The write is sudo'ed because a survey user can end a session even if it's
        not his own survey. """

        if not self.env.user.has_group('survey.group_survey_user'):
            raise AccessError(_('Only survey users can manage sessions.'))

        self.sudo().write({'session_state': False})
        self.user_input_ids.sudo().write({'state': 'done'})
        self.env['bus.bus']._sendone(self.access_token, 'end_session', {})

    def get_start_url(self):
        return '/survey/start/%s' % self.access_token

    def get_start_short_url(self):
        """ See controller method docstring for more details. """
        return '/s/%s' % self.access_token[:6]

    def get_print_url(self):
        return '/survey/print/%s' % self.access_token

    # ------------------------------------------------------------
    # GRAPH / RESULTS
    # ------------------------------------------------------------

    def _prepare_statistics(self, user_input_lines=None):
        if user_input_lines:
            user_input_domain = [
                ('survey_id', 'in', self.ids),
                ('id', 'in', user_input_lines.mapped('user_input_id').ids)
            ]
        else:
            user_input_domain = [
                ('survey_id', 'in', self.ids),
                ('state', '=', 'done'),
                ('test_entry', '=', False)
            ]
        count_data = self.env['survey.user_input'].sudo().read_group(user_input_domain, ['scoring_success', 'id:count_distinct'], ['scoring_success'])

        scoring_success_count = 0
        scoring_failed_count = 0
        for count_data_item in count_data:
            if count_data_item['scoring_success']:
                scoring_success_count += count_data_item['scoring_success_count']
            else:
                scoring_failed_count += count_data_item['scoring_success_count']

        success_graph = json.dumps([{
            'text': _('Passed'),
            'count': scoring_success_count,
            'color': '#2E7D32'
        }, {
            'text': _('Missed'),
            'count': scoring_failed_count,
            'color': '#C62828'
        }])

        total = scoring_success_count + scoring_failed_count
        return {
            'global_success_rate': round((scoring_success_count / total) * 100, 1) if total > 0 else 0,
            'global_success_graph': success_graph
        }

    # ------------------------------------------------------------
    # GAMIFICATION / BADGES
    # ------------------------------------------------------------

    def _prepare_challenge_category(self):
        return 'certification'

    def _create_certification_badge_trigger(self):
        self.ensure_one()
        if not self.certification_badge_id:
            raise ValueError(_('Certification Badge is not configured for the survey %(survey_name)s', survey_name=self.title))

        goal = self.env['gamification.goal.definition'].create({
            'name': self.title,
            'description': _("%s certification passed", self.title),
            'domain': "['&', ('survey_id', '=', %s), ('scoring_success', '=', True)]" % self.id,
            'computation_mode': 'count',
            'display_mode': 'boolean',
            'model_id': self.env.ref('survey.model_survey_user_input').id,
            'condition': 'higher',
            'batch_mode': True,
            'batch_distinctive_field': self.env.ref('survey.field_survey_user_input__partner_id').id,
            'batch_user_expression': 'user.partner_id.id'
        })
        challenge = self.env['gamification.challenge'].create({
            'name': _('%s challenge certification', self.title),
            'reward_id': self.certification_badge_id.id,
            'state': 'inprogress',
            'period': 'once',
            'challenge_category': self._prepare_challenge_category(),
            'reward_realtime': True,
            'report_message_frequency': 'never',
            'user_domain': [('karma', '>', 0)],
            'visibility_mode': 'personal'
        })
        self.env['gamification.challenge.line'].create({
            'definition_id': goal.id,
            'challenge_id': challenge.id,
            'target_goal': 1
        })

    def _handle_certification_badges(self, vals):
        if vals.get('certification_give_badge'):
            # If badge already set on records, reactivate the ones that are not active.
            surveys_with_badge = self.filtered(lambda survey: survey.certification_badge_id and not survey.certification_badge_id.active)
            surveys_with_badge.mapped('certification_badge_id').action_unarchive()
            # (re-)create challenge and goal
            for survey in self:
                survey._create_certification_badge_trigger()
        else:
            # if badge with owner : archive them, else delete everything (badge, challenge, goal)
            badges = self.mapped('certification_badge_id')
            challenges_to_delete = self.env['gamification.challenge'].search([('reward_id', 'in', badges.ids)])
            goals_to_delete = challenges_to_delete.mapped('line_ids').mapped('definition_id')
            badges.action_archive()
            # delete all challenges and goals because not needed anymore (challenge lines are deleted in cascade)
            challenges_to_delete.unlink()
            goals_to_delete.unlink()

```

## File: models\survey_user_input.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import uuid

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import float_is_zero

_logger = logging.getLogger(__name__)


class SurveyUserInput(models.Model):
    """ Metadata for a set of one user's answers to a particular survey """
    _name = "survey.user_input"
    _description = "Survey User Input"
    _rec_name = "survey_id"
    _order = "create_date desc"

    # answer description
    survey_id = fields.Many2one('survey.survey', string='Survey', required=True, readonly=True, ondelete='cascade')
    scoring_type = fields.Selection(string="Scoring", related="survey_id.scoring_type")
    start_datetime = fields.Datetime('Start date and time', readonly=True)
    end_datetime = fields.Datetime('End date and time', readonly=True)
    deadline = fields.Datetime('Deadline', help="Datetime until customer can open the survey and submit answers")
    state = fields.Selection([
        ('new', 'Not started yet'),
        ('in_progress', 'In Progress'),
        ('done', 'Completed')], string='Status', default='new', readonly=True)
    test_entry = fields.Boolean(readonly=True)
    last_displayed_page_id = fields.Many2one('survey.question', string='Last displayed question/page')
    # attempts management
    is_attempts_limited = fields.Boolean("Limited number of attempts", related='survey_id.is_attempts_limited')
    attempts_limit = fields.Integer("Number of attempts", related='survey_id.attempts_limit')
    attempts_number = fields.Integer("Attempt n°", compute='_compute_attempts_number')
    survey_time_limit_reached = fields.Boolean("Survey Time Limit Reached", compute='_compute_survey_time_limit_reached')
    # identification / access
    access_token = fields.Char('Identification token', default=lambda self: str(uuid.uuid4()), readonly=True, required=True, copy=False)
    invite_token = fields.Char('Invite token', readonly=True, copy=False)  # no unique constraint, as it identifies a pool of attempts
    partner_id = fields.Many2one('res.partner', string='Partner', readonly=True)
    email = fields.Char('Email', readonly=True)
    nickname = fields.Char('Nickname', help="Attendee nickname, mainly used to identify him in the survey session leaderboard.")
    # questions / answers
    user_input_line_ids = fields.One2many('survey.user_input.line', 'user_input_id', string='Answers', copy=True)
    predefined_question_ids = fields.Many2many('survey.question', string='Predefined Questions', readonly=True)
    scoring_percentage = fields.Float("Score (%)", compute="_compute_scoring_values", store=True, compute_sudo=True)  # stored for perf reasons
    scoring_total = fields.Float("Total Score", compute="_compute_scoring_values", store=True, compute_sudo=True)  # stored for perf reasons
    scoring_success = fields.Boolean('Quizz Passed', compute='_compute_scoring_success', store=True, compute_sudo=True)  # stored for perf reasons
    # live sessions
    is_session_answer = fields.Boolean('Is in a Session', help="Is that user input part of a survey session or not.")
    question_time_limit_reached = fields.Boolean("Question Time Limit Reached", compute='_compute_question_time_limit_reached')

    _sql_constraints = [
        ('unique_token', 'UNIQUE (access_token)', 'An access token must be unique!'),
    ]

    @api.depends('user_input_line_ids.answer_score', 'user_input_line_ids.question_id', 'predefined_question_ids.answer_score')
    def _compute_scoring_values(self):
        for user_input in self:
            # sum(multi-choice question scores) + sum(simple answer_type scores)
            total_possible_score = 0
            for question in user_input.predefined_question_ids:
                if question.question_type == 'simple_choice':
                    total_possible_score += max([score for score in question.mapped('suggested_answer_ids.answer_score') if score > 0], default=0)
                elif question.question_type == 'multiple_choice':
                    total_possible_score += sum(score for score in question.mapped('suggested_answer_ids.answer_score') if score > 0)
                elif question.is_scored_question:
                    total_possible_score += question.answer_score

            if total_possible_score == 0:
                user_input.scoring_percentage = 0
                user_input.scoring_total = 0
            else:
                score_total = sum(user_input.user_input_line_ids.mapped('answer_score'))
                user_input.scoring_total = score_total
                score_percentage = (score_total / total_possible_score) * 100
                user_input.scoring_percentage = round(score_percentage, 2) if score_percentage > 0 else 0

    @api.depends('scoring_percentage', 'survey_id')
    def _compute_scoring_success(self):
        for user_input in self:
            user_input.scoring_success = user_input.scoring_percentage >= user_input.survey_id.scoring_success_min

    @api.depends(
        'start_datetime',
        'survey_id.is_time_limited',
        'survey_id.time_limit')
    def _compute_survey_time_limit_reached(self):
        """ Checks that the user_input is not exceeding the survey's time limit. """
        for user_input in self:
            if not user_input.is_session_answer and user_input.start_datetime:
                start_time = user_input.start_datetime
                time_limit = user_input.survey_id.time_limit
                user_input.survey_time_limit_reached = user_input.survey_id.is_time_limited and \
                    fields.Datetime.now() >= start_time + relativedelta(minutes=time_limit)
            else:
                user_input.survey_time_limit_reached = False

    @api.depends(
        'survey_id.session_question_id.time_limit',
        'survey_id.session_question_id.is_time_limited',
        'survey_id.session_question_start_time')
    def _compute_question_time_limit_reached(self):
        """ Checks that the user_input is not exceeding the question's time limit.
        Only used in the context of survey sessions. """
        for user_input in self:
            if user_input.is_session_answer and user_input.survey_id.session_question_start_time:
                start_time = user_input.survey_id.session_question_start_time
                time_limit = user_input.survey_id.session_question_id.time_limit
                user_input.question_time_limit_reached = user_input.survey_id.session_question_id.is_time_limited and \
                    fields.Datetime.now() >= start_time + relativedelta(seconds=time_limit)
            else:
                user_input.question_time_limit_reached = False

    @api.depends('state', 'test_entry', 'survey_id.is_attempts_limited', 'partner_id', 'email', 'invite_token')
    def _compute_attempts_number(self):
        attempts_to_compute = self.filtered(
            lambda user_input: user_input.state == 'done' and not user_input.test_entry and user_input.survey_id.is_attempts_limited
        )

        for user_input in (self - attempts_to_compute):
            user_input.attempts_number = 1

        if attempts_to_compute:
            self.env.cr.execute("""SELECT user_input.id, (COUNT(previous_user_input.id) + 1) AS attempts_number
                FROM survey_user_input user_input
                LEFT OUTER JOIN survey_user_input previous_user_input
                ON user_input.survey_id = previous_user_input.survey_id
                AND previous_user_input.state = 'done'
                AND previous_user_input.test_entry IS NOT TRUE
                AND previous_user_input.id < user_input.id
                AND (user_input.invite_token IS NULL OR user_input.invite_token = previous_user_input.invite_token)
                AND (user_input.partner_id = previous_user_input.partner_id OR user_input.email = previous_user_input.email)
                WHERE user_input.id IN %s
                GROUP BY user_input.id;
            """, (tuple(attempts_to_compute.ids),))

            attempts_count_results = self.env.cr.dictfetchall()

            for user_input in attempts_to_compute:
                attempts_number = 1
                for attempts_count_result in attempts_count_results:
                    if attempts_count_result['id'] == user_input.id:
                        attempts_number = attempts_count_result['attempts_number']
                        break

                user_input.attempts_number = attempts_number

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'predefined_question_ids' not in vals:
                suvey_id = vals.get('survey_id', self.env.context.get('default_survey_id'))
                survey = self.env['survey.survey'].browse(suvey_id)
                vals['predefined_question_ids'] = [(6, 0, survey._prepare_user_input_predefined_questions().ids)]
        return super(SurveyUserInput, self).create(vals_list)

    # ------------------------------------------------------------
    # ACTIONS / BUSINESS
    # ------------------------------------------------------------

    def action_resend(self):
        partners = self.env['res.partner']
        emails = []
        for user_answer in self:
            if user_answer.partner_id:
                partners |= user_answer.partner_id
            elif user_answer.email:
                emails.append(user_answer.email)

        return self.survey_id.with_context(
            default_existing_mode='resend',
            default_partner_ids=partners.ids,
            default_emails=','.join(emails)
        ).action_send_survey()

    def action_print_answers(self):
        """ Open the website page with the survey form """
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'name': "View Answers",
            'target': 'self',
            'url': '/survey/print/%s?answer_token=%s' % (self.survey_id.access_token, self.access_token)
        }

    @api.model
    def _generate_invite_token(self):
        return str(uuid.uuid4())

    def _mark_in_progress(self):
        """ marks the state as 'in_progress' and updates the start_datetime accordingly. """
        self.write({
            'start_datetime': fields.Datetime.now(),
            'state': 'in_progress'
        })

    def _mark_done(self):
        """ This method will:
        1. mark the state as 'done'
        2. send the certification email with attached document if
        - The survey is a certification
        - It has a certification_mail_template_id set
        - The user succeeded the test
        Will also run challenge Cron to give the certification badge if any."""
        self.write({
            'end_datetime': fields.Datetime.now(),
            'state': 'done',
        })

        Challenge = self.env['gamification.challenge'].sudo()
        badge_ids = []
        for user_input in self:
            if user_input.survey_id.certification and user_input.scoring_success:
                if user_input.survey_id.certification_mail_template_id and not user_input.test_entry:
                    user_input.survey_id.certification_mail_template_id.send_mail(user_input.id, notif_layout="mail.mail_notification_light")
                if user_input.survey_id.certification_give_badge:
                    badge_ids.append(user_input.survey_id.certification_badge_id.id)

            # Update predefined_question_id to remove inactive questions
            user_input.predefined_question_ids -= user_input._get_inactive_conditional_questions()

        if badge_ids:
            challenges = Challenge.search([('reward_id', 'in', badge_ids)])
            if challenges:
                Challenge._cron_update(ids=challenges.ids, commit=False)

    def get_start_url(self):
        self.ensure_one()
        return '%s?answer_token=%s' % (self.survey_id.get_start_url(), self.access_token)

    def get_print_url(self):
        self.ensure_one()
        return '%s?answer_token=%s' % (self.survey_id.get_print_url(), self.access_token)

    # ------------------------------------------------------------
    # CREATE / UPDATE LINES FROM SURVEY FRONTEND INPUT
    # ------------------------------------------------------------

    def save_lines(self, question, answer, comment=None):
        """ Save answers to questions, depending on question type

            If an answer already exists for question and user_input_id, it will be
            overwritten (or deleted for 'choice' questions) (in order to maintain data consistency).
        """
        old_answers = self.env['survey.user_input.line'].search([
            ('user_input_id', '=', self.id),
            ('question_id', '=', question.id)
        ])

        if question.question_type in ['char_box', 'text_box', 'numerical_box', 'date', 'datetime']:
            self._save_line_simple_answer(question, old_answers, answer)
            if question.save_as_email and answer:
                self.write({'email': answer})
            if question.save_as_nickname and answer:
                self.write({'nickname': answer})

        elif question.question_type in ['simple_choice', 'multiple_choice']:
            self._save_line_choice(question, old_answers, answer, comment)
        elif question.question_type == 'matrix':
            self._save_line_matrix(question, old_answers, answer, comment)
        else:
            raise AttributeError(question.question_type + ": This type of question has no saving function")

    def _save_line_simple_answer(self, question, old_answers, answer):
        vals = self._get_line_answer_values(question, answer, question.question_type)
        if old_answers:
            old_answers.write(vals)
            return old_answers
        else:
            return self.env['survey.user_input.line'].create(vals)

    def _save_line_choice(self, question, old_answers, answers, comment):
        if not (isinstance(answers, list)):
            answers = [answers]

        if not answers:
            # add a False answer to force saving a skipped line
            # this will make this question correctly considered as skipped in statistics
            answers = [False]

        vals_list = []

        if question.question_type == 'simple_choice':
            if not question.comment_count_as_answer or not question.comments_allowed or not comment:
                vals_list = [self._get_line_answer_values(question, answer, 'suggestion') for answer in answers]
        elif question.question_type == 'multiple_choice':
            vals_list = [self._get_line_answer_values(question, answer, 'suggestion') for answer in answers]

        if comment:
            vals_list.append(self._get_line_comment_values(question, comment))

        old_answers.sudo().unlink()
        return self.env['survey.user_input.line'].create(vals_list)

    def _save_line_matrix(self, question, old_answers, answers, comment):
        vals_list = []

        if not answers and question.matrix_row_ids:
            # add a False answer to force saving a skipped line
            # this will make this question correctly considered as skipped in statistics
            answers = {question.matrix_row_ids[0].id: [False]}

        if answers:
            for row_key, row_answer in answers.items():
                for answer in row_answer:
                    vals = self._get_line_answer_values(question, answer, 'suggestion')
                    vals['matrix_row_id'] = int(row_key)
                    vals_list.append(vals.copy())

        if comment:
            vals_list.append(self._get_line_comment_values(question, comment))

        old_answers.sudo().unlink()
        return self.env['survey.user_input.line'].create(vals_list)

    def _get_line_answer_values(self, question, answer, answer_type):
        vals = {
            'user_input_id': self.id,
            'question_id': question.id,
            'skipped': False,
            'answer_type': answer_type,
        }
        if not answer or (isinstance(answer, str) and not answer.strip()):
            vals.update(answer_type=None, skipped=True)
            return vals

        if answer_type == 'suggestion':
            vals['suggested_answer_id'] = int(answer)
        elif answer_type == 'numerical_box':
            vals['value_numerical_box'] = float(answer)
        else:
            vals['value_%s' % answer_type] = answer
        return vals

    def _get_line_comment_values(self, question, comment):
        return {
            'user_input_id': self.id,
            'question_id': question.id,
            'skipped': False,
            'answer_type': 'char_box',
            'value_char_box': comment,
        }

    # ------------------------------------------------------------
    # STATISTICS / RESULTS
    # ------------------------------------------------------------

    def _prepare_statistics(self):
        """ Prepares survey.user_input's statistics to display various charts on the frontend.
        Returns a structure containing answers statistics "by section" and "totals" for every input in self.

        e.g returned structure:
        {
            survey.user_input(1,): {
                'by_section': {
                    'Uncategorized': {
                        'question_count': 2,
                        'correct': 2,
                        'partial': 0,
                        'incorrect': 0,
                        'skipped': 0,
                    },
                    'Mathematics': {
                        'question_count': 3,
                        'correct': 1,
                        'partial': 1,
                        'incorrect': 0,
                        'skipped': 1,
                    },
                    'Geography': {
                        'question_count': 4,
                        'correct': 2,
                        'partial': 0,
                        'incorrect': 2,
                        'skipped': 0,
                    }
                },
                'totals' [{
                    'text': 'Correct',
                    'count': 5,
                }, {
                    'text': 'Partially',
                    'count': 1,
                }, {
                    'text': 'Incorrect',
                    'count': 2,
                }, {
                    'text': 'Unanswered',
                    'count': 1,
                }]
            }
        }"""
        res = dict((user_input, {
            'by_section': {}
        }) for user_input in self)

        scored_questions = self.mapped('predefined_question_ids').filtered(lambda question: question.is_scored_question)

        for question in scored_questions:
            if question.question_type in ['simple_choice', 'multiple_choice']:
                question_correct_suggested_answers = question.suggested_answer_ids.filtered(lambda answer: answer.is_correct)

            question_section = question.page_id.title or _('Uncategorized')
            for user_input in self:
                user_input_lines = user_input.user_input_line_ids.filtered(lambda line: line.question_id == question)
                if question.question_type in ['simple_choice', 'multiple_choice']:
                    answer_result_key = self._choice_question_answer_result(user_input_lines, question_correct_suggested_answers)
                else:
                    answer_result_key = self._simple_question_answer_result(user_input_lines)

                if question_section not in res[user_input]['by_section']:
                    res[user_input]['by_section'][question_section] = {
                        'question_count': 0,
                        'correct': 0,
                        'partial': 0,
                        'incorrect': 0,
                        'skipped': 0,
                    }

                res[user_input]['by_section'][question_section]['question_count'] += 1
                res[user_input]['by_section'][question_section][answer_result_key] += 1

        for user_input in self:
            correct_count = 0
            partial_count = 0
            incorrect_count = 0
            skipped_count = 0

            for section_counts in res[user_input]['by_section'].values():
                correct_count += section_counts.get('correct', 0)
                partial_count += section_counts.get('partial', 0)
                incorrect_count += section_counts.get('incorrect', 0)
                skipped_count += section_counts.get('skipped', 0)

            res[user_input]['totals'] = [
                {'text': _("Correct"), 'count': correct_count},
                {'text': _("Partially"), 'count': partial_count},
                {'text': _("Incorrect"), 'count': incorrect_count},
                {'text': _("Unanswered"), 'count': skipped_count}
            ]

        return res

    def _choice_question_answer_result(self, user_input_lines, question_correct_suggested_answers):
        correct_user_input_lines = user_input_lines.filtered(lambda line: line.answer_is_correct and not line.skipped).mapped('suggested_answer_id')
        incorrect_user_input_lines = user_input_lines.filtered(lambda line: not line.answer_is_correct and not line.skipped)
        if question_correct_suggested_answers and correct_user_input_lines == question_correct_suggested_answers:
            return 'correct'
        elif correct_user_input_lines and correct_user_input_lines < question_correct_suggested_answers:
            return 'partial'
        elif not correct_user_input_lines and incorrect_user_input_lines:
            return 'incorrect'
        else:
            return 'skipped'

    def _simple_question_answer_result(self, user_input_line):
        if user_input_line.skipped:
            return 'skipped'
        elif user_input_line.answer_is_correct:
            return 'correct'
        else:
            return 'incorrect'

    # ------------------------------------------------------------
    # Conditional Questions Management
    # ------------------------------------------------------------

    def _get_conditional_values(self):
        """ For survey containing conditional questions, we need a triggered_questions_by_answer map that contains
                {key: answer, value: the question that the answer triggers, if selected},
         The idea is to be able to verify, on every answer check, if this answer is triggering the display
         of another question.
         If answer is not in the conditional map:
            - nothing happens.
         If the answer is in the conditional map:
            - If we are in ONE PAGE survey : (handled at CLIENT side)
                -> display immediately the depending question
            - If we are in PAGE PER SECTION : (handled at CLIENT side)
                - If related question is on the same page :
                    -> display immediately the depending question
                - If the related question is not on the same page :
                    -> keep the answers in memory and check at next page load if the depending question is in there and
                       display it, if so.
            - If we are in PAGE PER QUESTION : (handled at SERVER side)
                -> During submit, determine which is the next question to display getting the next question
                   that is the next in sequence and that is either not triggered by another question's answer, or that
                   is triggered by an already selected answer.
         To do all this, we need to return:
            - list of all selected answers: [answer_id1, answer_id2, ...] (for survey reloading, otherwise, this list is
              updated at client side)
            - triggered_questions_by_answer: dict -> for a given answer, list of questions triggered by this answer;
                Used mainly for dynamic show/hide behaviour at client side
            - triggering_answer_by_question: dict -> for a given question, the answer that triggers it
                Used mainly to ease template rendering
        """
        triggering_answer_by_question, triggered_questions_by_answer = {}, {}
        # Ignore conditional configuration if randomised questions selection
        if self.survey_id.questions_selection != 'random':
            triggering_answer_by_question, triggered_questions_by_answer = self.survey_id._get_conditional_maps()
        selected_answers = self._get_selected_suggested_answers()

        return triggering_answer_by_question, triggered_questions_by_answer, selected_answers

    def _get_selected_suggested_answers(self):
        """
        For now, only simple and multiple choices question type are handled by the conditional questions feature.
        Mapping all the suggested answers selected by the user will also include answers from matrix question type,
        Those ones won't be used.
        Maybe someday, conditional questions feature will be extended to work with matrix question.
        :return: all the suggested answer selected by the user.
        """
        return self.mapped('user_input_line_ids.suggested_answer_id')

    def _clear_inactive_conditional_answers(self):
        """
        Clean eventual answers on conditional questions that should not have been displayed to user.
        This method is used mainly for page per question survey, a similar method does the same treatment
        at client side for the other survey layouts.
        E.g.: if depending answer was uncheck after answering conditional question, we need to clear answers
              of that conditional question, for two reasons:
              - ensure correct scoring
              - if the selected answer triggers another question later in the survey, if the answer is not cleared,
                a question that should not be displayed to the user will be.
        
        TODO DBE: Maybe this can be the only cleaning method, even for section_per_page or one_page where 
        conditional questions are, for now, cleared in JS directly. But this can be annoying if user typed a long 
        answer, changed his mind unchecking depending answer and changed again his mind by rechecking the depending 
        answer -> For now, the long answer will be lost. If we use this as the master cleaning method, 
        long answer will be cleared only during submit.
        """
        inactive_questions = self._get_inactive_conditional_questions()

        # delete user.input.line on question that should not be answered.
        answers_to_delete = self.user_input_line_ids.filtered(lambda answer: answer.question_id in inactive_questions)
        answers_to_delete.unlink()

    def _get_inactive_conditional_questions(self):
        triggering_answer_by_question, triggered_questions_by_answer, selected_answers = self._get_conditional_values()

        # get questions that should not be answered
        inactive_questions = self.env['survey.question']
        for answer in triggered_questions_by_answer.keys():
            if answer not in selected_answers:
                for question in triggered_questions_by_answer[answer]:
                    inactive_questions |= question
        return inactive_questions

    def _get_print_questions(self):
        """ Get the questions to display : the ones that should have been answered = active questions
            In case of session, active questions are based on most voted answers
        :return: active survey.question browse records
        """
        survey = self.survey_id
        if self.is_session_answer:
            most_voted_answers = survey._get_session_most_voted_answers()
            inactive_questions = most_voted_answers._get_inactive_conditional_questions()
        else:
            inactive_questions = self._get_inactive_conditional_questions()
        return survey.question_ids - inactive_questions


class SurveyUserInputLine(models.Model):
    _name = 'survey.user_input.line'
    _description = 'Survey User Input Line'
    _rec_name = 'user_input_id'
    _order = 'question_sequence, id'

    # survey data
    user_input_id = fields.Many2one('survey.user_input', string='User Input', ondelete='cascade', required=True)
    survey_id = fields.Many2one(related='user_input_id.survey_id', string='Survey', store=True, readonly=False)
    question_id = fields.Many2one('survey.question', string='Question', ondelete='cascade', required=True)
    page_id = fields.Many2one(related='question_id.page_id', string="Section", readonly=False)
    question_sequence = fields.Integer('Sequence', related='question_id.sequence', store=True)
    # answer
    skipped = fields.Boolean('Skipped')
    answer_type = fields.Selection([
        ('text_box', 'Free Text'),
        ('char_box', 'Text'),
        ('numerical_box', 'Number'),
        ('date', 'Date'),
        ('datetime', 'Datetime'),
        ('suggestion', 'Suggestion')], string='Answer Type')
    value_char_box = fields.Char('Text answer')
    value_numerical_box = fields.Float('Numerical answer')
    value_date = fields.Date('Date answer')
    value_datetime = fields.Datetime('Datetime answer')
    value_text_box = fields.Text('Free Text answer')
    suggested_answer_id = fields.Many2one('survey.question.answer', string="Suggested answer")
    matrix_row_id = fields.Many2one('survey.question.answer', string="Row answer")
    # scoring
    answer_score = fields.Float('Score')
    answer_is_correct = fields.Boolean('Correct')

    @api.constrains('skipped', 'answer_type')
    def _check_answer_type_skipped(self):
        for line in self:
            if (line.skipped == bool(line.answer_type)):
                raise ValidationError(_('A question can either be skipped or answered, not both.'))

            # allow 0 for numerical box
            if line.answer_type == 'numerical_box' and float_is_zero(line['value_numerical_box'], precision_digits=6):
                continue
            if line.answer_type == 'suggestion':
                field_name = 'suggested_answer_id'
            elif line.answer_type:
                field_name = 'value_%s' % line.answer_type
            else:  # skipped
                field_name = False

            if field_name and not line[field_name]:
                raise ValidationError(_('The answer must be in the right type'))

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('answer_score'):
                score_vals = self._get_answer_score_values(vals)
                vals.update(score_vals)
        return super(SurveyUserInputLine, self).create(vals_list)

    def write(self, vals):
        res = True
        for line in self:
            vals_copy = {**vals}
            getter_params = {
                'user_input_id': line.user_input_id.id,
                'answer_type': line.answer_type,
                'question_id': line.question_id.id,
                **vals_copy
            }
            if not vals_copy.get('answer_score'):
                score_vals = self._get_answer_score_values(getter_params, compute_speed_score=False)
                vals_copy.update(score_vals)
            res = super(SurveyUserInputLine, line).write(vals_copy) and res
        return res

    @api.model
    def _get_answer_score_values(self, vals, compute_speed_score=True):
        """ Get values for: answer_is_correct and associated answer_score.

        Requires vals to contain 'answer_type', 'question_id', and 'user_input_id'.
        Depending on 'answer_type' additional value of 'suggested_answer_id' may also be
        required.

        Calculates whether an answer_is_correct and its score based on 'answer_type' and
        corresponding question. Handles choice (answer_type == 'suggestion') questions
        separately from other question types. Each selected choice answer is handled as an
        individual answer.

        If score depends on the speed of the answer, it is adjusted as follows:
         - If the user answers in less than 2 seconds, they receive 100% of the possible points.
         - If user answers after that, they receive 50% of the possible points + the remaining
            50% scaled by the time limit and time taken to answer [i.e. a minimum of 50% of the
            possible points is given to all correct answers]

        Example of returned values:
            * {'answer_is_correct': False, 'answer_score': 0} (default)
            * {'answer_is_correct': True, 'answer_score': 2.0}
        """
        user_input_id = vals.get('user_input_id')
        answer_type = vals.get('answer_type')
        question_id = vals.get('question_id')
        if not question_id:
            raise ValueError(_('Computing score requires a question in arguments.'))
        question = self.env['survey.question'].browse(int(question_id))

        # default and non-scored questions
        answer_is_correct = False
        answer_score = 0

        # record selected suggested choice answer_score (can be: pos, neg, or 0)
        if question.question_type in ['simple_choice', 'multiple_choice']:
            if answer_type == 'suggestion':
                suggested_answer_id = vals.get('suggested_answer_id')
                if suggested_answer_id:
                    question_answer = self.env['survey.question.answer'].browse(int(suggested_answer_id))
                    answer_score = question_answer.answer_score
                    answer_is_correct = question_answer.is_correct
        # for all other scored question cases, record question answer_score (can be: pos or 0)
        elif question.is_scored_question:
            answer = vals.get('value_%s' % answer_type)
            if answer_type == 'numerical_box':
                answer = float(answer)
            elif answer_type == 'date':
                answer = fields.Date.from_string(answer)
            elif answer_type == 'datetime':
                answer = fields.Datetime.from_string(answer)
            if answer and answer == question['answer_%s' % answer_type]:
                answer_is_correct = True
                answer_score = question.answer_score

        if compute_speed_score and answer_score > 0:
            user_input = self.env['survey.user_input'].browse(user_input_id)
            session_speed_rating = user_input.exists() and user_input.is_session_answer and user_input.survey_id.session_speed_rating
            if session_speed_rating:
                max_score_delay = 2
                time_limit = question.time_limit
                now = fields.Datetime.now()
                seconds_to_answer = (now - user_input.survey_id.session_question_start_time).total_seconds()
                question_remaining_time = time_limit - seconds_to_answer
                # if answered within the max_score_delay => leave score as is
                if question_remaining_time < 0:  # if no time left
                    answer_score /= 2
                elif seconds_to_answer > max_score_delay:
                    time_limit -= max_score_delay  # we remove the max_score_delay to have all possible values
                    score_proportion = (time_limit - seconds_to_answer) / time_limit
                    answer_score = (answer_score / 2) * (1 + score_proportion)

        return {
            'answer_is_correct': answer_is_correct,
            'answer_score': answer_score
        }

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import survey_survey
from . import survey_question
from . import survey_user_input
from . import badge
from . import challenge
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_survey_all,survey.survey.all,model_survey_survey,,0,0,0,0
access_survey_user,survey.survey.user,model_survey_survey,base.group_user,0,0,0,0
access_survey_survey_user,survey.survey.survey.user,model_survey_survey,group_survey_user,1,1,1,1
access_survey_survey_manager,survey.survey.survey.manager,model_survey_survey,group_survey_manager,1,1,1,1
access_survey_question_all,survey.question.all,model_survey_question,,0,0,0,0
access_survey_question_user,survey.question.user,model_survey_question,base.group_user,0,0,0,0
access_survey_question_survey_user,survey.question.survey.user,model_survey_question,group_survey_user,1,1,1,1
access_survey_question_survey_manager,survey.question.survey.manager,model_survey_question,group_survey_manager,1,1,1,1
access_survey_question_answer_all,survey.question.answer.all,model_survey_question_answer,,0,0,0,0
access_survey_question_answer_user,survey.question.answer.user,model_survey_question_answer,base.group_user,0,0,0,0
access_survey_question_answer_survey_user,survey.question.answer.survey.user,model_survey_question_answer,group_survey_user,1,1,1,1
access_survey_question_answer_survey_manager,survey.question.answer.survey.manager,model_survey_question_answer,group_survey_manager,1,1,1,1
access_survey_user_input_all,survey.user_input.all,model_survey_user_input,,0,0,0,0
access_survey_user_input_user,survey.user_input.user,model_survey_user_input,base.group_user,0,0,0,0
access_survey_user_input_survey_user,survey.user_input.survey.user,model_survey_user_input,group_survey_user,1,1,1,1
access_survey_user_input_survey_manager,survey.user_input.survey.manager,model_survey_user_input,group_survey_manager,1,1,1,1
access_survey_user_input_line_all,survey.user_input.line.all,model_survey_user_input_line,,0,0,0,0
access_survey_user_input_line_user,survey.user_input.line.user,model_survey_user_input_line,base.group_user,0,0,0,0
access_survey_user_input_line_survey_user,survey.user_input.line.survey.user,model_survey_user_input_line,group_survey_user,1,1,1,1
access_survey_user_input_line_survey_manager,survey.user_input.line.survey.manager,model_survey_user_input_line,group_survey_manager,1,1,1,1
access_gamification_badge_survey_user,gamification.badge.survey.user,model_gamification_badge,group_survey_user,1,1,1,1
access_survey_invite,access.survey.invite,model_survey_invite,survey.group_survey_user,1,1,1,0

```

## File: security\survey_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.module.category" id="base.module_category_marketing_surveys">
            <field name="description">Helps you manage your survey for review of different-different users.</field>
            <field name="sequence">20</field>
        </record>

        <!-- Survey users -->
        <record model="res.groups" id="group_survey_user">
            <field name="name">User</field>
            <field name="category_id" ref="base.module_category_marketing_surveys"/>
        </record>

        <!-- Survey managers -->
        <record model="res.groups" id="group_survey_manager">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_marketing_surveys"/>
            <field name="implied_ids" eval="[(4, ref('group_survey_user'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('group_survey_manager'))]"/>
        </record>

        <!-- SURVEY: SURVEY, PAGE, STAGE, QUESTION, LABEL -->
        <record id="survey_survey_rule_survey_manager" model="ir.rule">
            <field name="name">Survey: manager: all</field>
            <field name="model_id" ref="survey.model_survey_survey"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_survey_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_survey"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_survey_rule_survey_user_cwu" model="ir.rule">
            <field name="name">Survey: officer: create/write/unlink own only</field>
            <field name="model_id" ref="survey.model_survey_survey"/>
            <field name="domain_force">[('create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="survey_question_rule_survey_manager" model="ir.rule">
            <field name="name">Survey question: manager: all</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_question_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey question: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_question_rule_survey_user_cw" model="ir.rule">
            <field name="name">Survey question: officer: create/write/unlink linked to own survey only</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[('survey_id.create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="survey_question_answer_rule_survey_manager" model="ir.rule">
            <field name="name">Survey question answer: manager: all</field>
            <field name="model_id" ref="survey.model_survey_question_answer"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_question_answer_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey question answer: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_question_answer"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_question_answer_rule_survey_user_cw" model="ir.rule">
            <field name="name">Survey question answer: officer: create/write/unlink linked to own survey only</field>
            <field name="model_id" ref="survey.model_survey_question_answer"/>
            <field name="domain_force">['|', ('question_id.survey_id.create_uid', '=', user.id), ('matrix_question_id.survey_id.create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <!-- SURVEY: USER_INPUT, USER_INPUT_LINE -->
        <record id="survey_user_input_rule_survey_manager" model="ir.rule">
            <field name="name">Survey user input: manager: all</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_user_input_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey user input: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_user_input_rule_survey_user_cw" model="ir.rule">
            <field name="name">Survey user input: officer: create/write/unlink linked to own survey only</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="domain_force">[('survey_id.create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>

        <record id="survey_user_input_line_rule_survey_manager" model="ir.rule">
            <field name="name">Survey user input line: manager: all</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_user_input_line_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey user input line: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_user_input_line_rule_survey_user_cw" model="ir.rule">
            <field name="name">Survey user input line: officer: create/write/unlink linked to own survey only</field>
            <field name="model_id" ref="survey.model_survey_user_input_line"/>
            <field name="domain_force">[('user_input_id.survey_id.create_uid', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="1"/>
        </record>
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient><path id="d" d="M48 30l-2 1.882V22H20v32h26v-5.647l2-1.882V54a2 2 0 0 1-2 2H20a2 2 0 0 1-2-2V16a2 2 0 0 1 2-2h6v-2h4v2h6v-2h4l.03 2H46a2 2 0 0 1 2 2v14zM20 16v4h26v-4h-6v2h-4v-2h-6v2h-4v-2h-6zm5 23h12a1 1 0 0 1 1 1v10a1 1 0 0 1-1 1H25a1 1 0 0 1-1-1V40a1 1 0 0 1 1-1zm1 1.984V49h10v-8.016H26zm3.687 7.327l-2.571-2.806a.46.46 0 0 1 0-.61l.56-.61a.373.373 0 0 1 .559 0l1.732 1.89 3.71-4.049a.373.373 0 0 1 .56 0l.56.61a.46.46 0 0 1 0 .611l-4.55 4.964a.373.373 0 0 1-.56 0zM25 25h12a1 1 0 0 1 1 1v9a1 1 0 0 1-1 1H25a1 1 0 0 1-1-1v-9a1 1 0 0 1 1-1zm1 2v7h10v-7H26zm35.5 2.95l-1.914 2.053a.517.517 0 0 1-.731.025l-4.942-4.608a.517.517 0 0 1-.026-.731l1.914-2.052a2.07 2.07 0 0 1 2.921-.102l2.676 2.495c.837.776.883 2.083.102 2.92zm-9.575-1.805l-10.903 11.69-.73 5.259c-.1.71.537 1.299 1.238 1.154l5.195-1.099 10.903-11.69a.517.517 0 0 0-.026-.732l-4.942-4.608a.522.522 0 0 0-.735.026zm-6.45 10.572a.6.6 0 0 1-.03-.852l6.394-6.856a.6.6 0 0 1 .852-.03.6.6 0 0 1 .03.852l-6.395 6.856a.6.6 0 0 1-.851.03zm-1.687 3.492l2.065-.072.055 1.561-2.758.583-1.385-1.291.39-2.792 1.56-.054.073 2.065z"/><path id="e" d="M48 28l-2 1.882V20H20v32h26v-5.647l2-1.882V52a2 2 0 0 1-2 2H20a2 2 0 0 1-2-2V14a2 2 0 0 1 2-2h6v-2h4v2h6v-2h4l.03 2H46a2 2 0 0 1 2 2v14zM20 14v4h26v-4h-6v1h-4v-1h-6v1h-4v-1h-6zm5 23h12a1 1 0 0 1 1 1v10a1 1 0 0 1-1 1H25a1 1 0 0 1-1-1V38a1 1 0 0 1 1-1zm1 1.984V47h10v-8.016H26zm3.687 7.327l-2.571-2.806a.46.46 0 0 1 0-.61l.56-.61a.373.373 0 0 1 .559 0l1.732 1.89 3.71-4.049a.373.373 0 0 1 .56 0l.56.61a.46.46 0 0 1 0 .611l-4.55 4.964a.373.373 0 0 1-.56 0zM25 23h12a1 1 0 0 1 1 1v9a1 1 0 0 1-1 1H25a1 1 0 0 1-1-1v-9a1 1 0 0 1 1-1zm1 2v7h10v-7H26zm35.5 2.95l-1.914 2.053a.517.517 0 0 1-.731.025l-4.942-4.608a.517.517 0 0 1-.026-.731l1.914-2.052a2.07 2.07 0 0 1 2.921-.102l2.676 2.495c.837.776.883 2.083.102 2.92zm-9.575-1.805l-10.903 11.69-.73 5.259c-.1.71.537 1.299 1.238 1.154l5.195-1.099 10.903-11.69a.517.517 0 0 0-.026-.732l-4.942-4.608a.522.522 0 0 0-.735.026zm-6.45 10.572a.6.6 0 0 1-.03-.852l6.394-6.856a.6.6 0 0 1 .852-.03.6.6 0 0 1 .03.852l-6.395 6.856a.6.6 0 0 1-.851.03zm-1.687 3.492l2.065-.072.055 1.561-2.758.583-1.385-1.291.39-2.792 1.56-.054.073 2.065z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M32.889 69H4c-2 0-4-.145-4-4.073v-33.6L20 12h20l8 2V31.327l6-6.109 5 5.091-11 11.2-.245 11.542L32.889 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\fonts\AlexBrush-Regular-ofl.txt

```text
Copyright 2011 The Alex Brush Project Authors (https://github.com/googlefonts/alex-brush)

This Font Software is licensed under the SIL Open Font License, Version 1.1.
This license is copied below, and is also available with a FAQ at:
http://scripts.sil.org/OFL


-----------------------------------------------------------
SIL OPEN FONT LICENSE Version 1.1 - 26 February 2007
-----------------------------------------------------------

PREAMBLE
The goals of the Open Font License (OFL) are to stimulate worldwide
development of collaborative font projects, to support the font creation
efforts of academic and linguistic communities, and to provide a free and
open framework in which fonts may be shared and improved in partnership
with others.

The OFL allows the licensed fonts to be used, studied, modified and
redistributed freely as long as they are not sold by themselves. The
fonts, including any derivative works, can be bundled, embedded, 
redistributed and/or sold with any software provided that any reserved
names are not used by derivative works. The fonts and derivatives,
however, cannot be released under any other type of license. The
requirement for fonts to remain under this license does not apply
to any document created using the fonts or their derivatives.

DEFINITIONS
"Font Software" refers to the set of files released by the Copyright
Holder(s) under this license and clearly marked as such. This may
include source files, build scripts and documentation.

"Reserved Font Name" refers to any names specified as such after the
copyright statement(s).

"Original Version" refers to the collection of Font Software components as
distributed by the Copyright Holder(s).

"Modified Version" refers to any derivative made by adding to, deleting,
or substituting -- in part or in whole -- any of the components of the
Original Version, by changing formats or by porting the Font Software to a
new environment.

"Author" refers to any designer, engineer, programmer, technical
writer or other person who contributed to the Font Software.

PERMISSION & CONDITIONS
Permission is hereby granted, free of charge, to any person obtaining
a copy of the Font Software, to use, study, copy, merge, embed, modify,
redistribute, and sell modified and unmodified copies of the Font
Software, subject to the following conditions:

1) Neither the Font Software nor any of its individual components,
in Original or Modified Versions, may be sold by itself.

2) Original or Modified Versions of the Font Software may be bundled,
redistributed and/or sold with any software, provided that each copy
contains the above copyright notice and this license. These can be
included either as stand-alone text files, human-readable headers or
in the appropriate machine-readable metadata fields within text or
binary files as long as those fields can be easily viewed by the user.

3) No Modified Version of the Font Software may use the Reserved Font
Name(s) unless explicit written permission is granted by the corresponding
Copyright Holder. This restriction only applies to the primary font name as
presented to the users.

4) The name(s) of the Copyright Holder(s) or the Author(s) of the Font
Software shall not be used to promote, endorse or advertise any
Modified Version, except to acknowledge the contribution(s) of the
Copyright Holder(s) and the Author(s) or with their explicit written
permission.

5) The Font Software, modified or unmodified, in part or in whole,
must be distributed entirely under this license, and must not be
distributed under any other license. The requirement for fonts to
remain under this license does not apply to any document created
using the Font Software.

TERMINATION
This license becomes null and void if any of the above conditions are
not met.

DISCLAIMER
THE FONT SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT
OF COPYRIGHT, PATENT, TRADEMARK, OR OTHER RIGHT. IN NO EVENT SHALL THE
COPYRIGHT HOLDER BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
INCLUDING ANY GENERAL, SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL
DAMAGES, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF THE USE OR INABILITY TO USE THE FONT SOFTWARE OR FROM
OTHER DEALINGS IN THE FONT SOFTWARE.

```

## File: static\src\fonts\IbarraRealNova-ofl.txt

```text
Copyright 2007 The Ibarra Real Nova Project Authors (https://github.com/googlefonts/ibarrareal)

This Font Software is licensed under the SIL Open Font License, Version 1.1.
This license is copied below, and is also available with a FAQ at:
http://scripts.sil.org/OFL


-----------------------------------------------------------
SIL OPEN FONT LICENSE Version 1.1 - 26 February 2007
-----------------------------------------------------------

PREAMBLE
The goals of the Open Font License (OFL) are to stimulate worldwide
development of collaborative font projects, to support the font creation
efforts of academic and linguistic communities, and to provide a free and
open framework in which fonts may be shared and improved in partnership
with others.

The OFL allows the licensed fonts to be used, studied, modified and
redistributed freely as long as they are not sold by themselves. The
fonts, including any derivative works, can be bundled, embedded, 
redistributed and/or sold with any software provided that any reserved
names are not used by derivative works. The fonts and derivatives,
however, cannot be released under any other type of license. The
requirement for fonts to remain under this license does not apply
to any document created using the fonts or their derivatives.

DEFINITIONS
"Font Software" refers to the set of files released by the Copyright
Holder(s) under this license and clearly marked as such. This may
include source files, build scripts and documentation.

"Reserved Font Name" refers to any names specified as such after the
copyright statement(s).

"Original Version" refers to the collection of Font Software components as
distributed by the Copyright Holder(s).

"Modified Version" refers to any derivative made by adding to, deleting,
or substituting -- in part or in whole -- any of the components of the
Original Version, by changing formats or by porting the Font Software to a
new environment.

"Author" refers to any designer, engineer, programmer, technical
writer or other person who contributed to the Font Software.

PERMISSION & CONDITIONS
Permission is hereby granted, free of charge, to any person obtaining
a copy of the Font Software, to use, study, copy, merge, embed, modify,
redistribute, and sell modified and unmodified copies of the Font
Software, subject to the following conditions:

1) Neither the Font Software nor any of its individual components,
in Original or Modified Versions, may be sold by itself.

2) Original or Modified Versions of the Font Software may be bundled,
redistributed and/or sold with any software, provided that each copy
contains the above copyright notice and this license. These can be
included either as stand-alone text files, human-readable headers or
in the appropriate machine-readable metadata fields within text or
binary files as long as those fields can be easily viewed by the user.

3) No Modified Version of the Font Software may use the Reserved Font
Name(s) unless explicit written permission is granted by the corresponding
Copyright Holder. This restriction only applies to the primary font name as
presented to the users.

4) The name(s) of the Copyright Holder(s) or the Author(s) of the Font
Software shall not be used to promote, endorse or advertise any
Modified Version, except to acknowledge the contribution(s) of the
Copyright Holder(s) and the Author(s) or with their explicit written
permission.

5) The Font Software, modified or unmodified, in part or in whole,
must be distributed entirely under this license, and must not be
distributed under any other license. The requirement for fonts to
remain under this license does not apply to any document created
using the Font Software.

TERMINATION
This license becomes null and void if any of the above conditions are
not met.

DISCLAIMER
THE FONT SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT
OF COPYRIGHT, PATENT, TRADEMARK, OR OTHER RIGHT. IN NO EVENT SHALL THE
COPYRIGHT HOLDER BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
INCLUDING ANY GENERAL, SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL
DAMAGES, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF THE USE OR INABILITY TO USE THE FONT SOFTWARE OR FROM
OTHER DEALINGS IN THE FONT SOFTWARE.

```

## File: static\src\fonts\Trueno-ofl.txt

```text
Copyright (c) 2011, The Montserrat Project Authors (https://github.com/JulietaUla/Montserrat).
Copyright (c) 2015, Jasper @ Cannot Into Space Fonts.

This Font Software is licensed under the SIL Open Font License, Version 1.1.
This license is copied below, and is also available with a FAQ at:
http://scripts.sil.org/OFL


-----------------------------------------------------------
SIL OPEN FONT LICENSE Version 1.1 - 26 February 2007
-----------------------------------------------------------

PREAMBLE
The goals of the Open Font License (OFL) are to stimulate worldwide
development of collaborative font projects, to support the font creation
efforts of academic and linguistic communities, and to provide a free and
open framework in which fonts may be shared and improved in partnership
with others.

The OFL allows the licensed fonts to be used, studied, modified and
redistributed freely as long as they are not sold by themselves. The
fonts, including any derivative works, can be bundled, embedded, 
redistributed and/or sold with any software provided that any reserved
names are not used by derivative works. The fonts and derivatives,
however, cannot be released under any other type of license. The
requirement for fonts to remain under this license does not apply
to any document created using the fonts or their derivatives.

DEFINITIONS
"Font Software" refers to the set of files released by the Copyright
Holder(s) under this license and clearly marked as such. This may
include source files, build scripts and documentation.

"Reserved Font Name" refers to any names specified as such after the
copyright statement(s).

"Original Version" refers to the collection of Font Software components as
distributed by the Copyright Holder(s).

"Modified Version" refers to any derivative made by adding to, deleting,
or substituting -- in part or in whole -- any of the components of the
Original Version, by changing formats or by porting the Font Software to a
new environment.

"Author" refers to any designer, engineer, programmer, technical
writer or other person who contributed to the Font Software.

PERMISSION & CONDITIONS
Permission is hereby granted, free of charge, to any person obtaining
a copy of the Font Software, to use, study, copy, merge, embed, modify,
redistribute, and sell modified and unmodified copies of the Font
Software, subject to the following conditions:

1) Neither the Font Software nor any of its individual components,
in Original or Modified Versions, may be sold by itself.

2) Original or Modified Versions of the Font Software may be bundled,
redistributed and/or sold with any software, provided that each copy
contains the above copyright notice and this license. These can be
included either as stand-alone text files, human-readable headers or
in the appropriate machine-readable metadata fields within text or
binary files as long as those fields can be easily viewed by the user.

3) No Modified Version of the Font Software may use the Reserved Font
Name(s) unless explicit written permission is granted by the corresponding
Copyright Holder. This restriction only applies to the primary font name as
presented to the users.

4) The name(s) of the Copyright Holder(s) or the Author(s) of the Font
Software shall not be used to promote, endorse or advertise any
Modified Version, except to acknowledge the contribution(s) of the
Copyright Holder(s) and the Author(s) or with their explicit written
permission.

5) The Font Software, modified or unmodified, in part or in whole,
must be distributed entirely under this license, and must not be
distributed under any other license. The requirement for fonts to
remain under this license does not apply to any document created
using the Font Software.

TERMINATION
This license becomes null and void if any of the above conditions are
not met.

DISCLAIMER
THE FONT SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT
OF COPYRIGHT, PATENT, TRADEMARK, OR OTHER RIGHT. IN NO EVENT SHALL THE
COPYRIGHT HOLDER BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
INCLUDING ANY GENERAL, SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL
DAMAGES, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF THE USE OR INABILITY TO USE THE FONT SOFTWARE OR FROM
OTHER DEALINGS IN THE FONT SOFTWARE.

```

## File: static\src\img\certification_bg_classic.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 836.89 590.28"><path d="M15.17,570.38l.09-.12a1,1,0,0,1,.33-.65l.18-.15.11-.09a35.15,35.15,0,0,0,4.08-8.3,35.19,35.19,0,0,0,1.82-10.44v-.09a1.36,1.36,0,0,0-2.49-.75,45.6,45.6,0,0,1-5.51,6.68l-.41.41A4.67,4.67,0,0,0,12,560.21V574c.39-.37.8-.74,1.21-1.1.67-.77,1.28-1.53,1.82-2.26C15.06,570.55,15.12,570.47,15.17,570.38Z" style="fill:#fff"/><path d="M13.91,524.52a37.1,37.1,0,0,1,6.44,9.59,36.82,36.82,0,0,1,1.68,4.3.14.14,0,0,0,.14.11.15.15,0,0,0,.15-.12,42.32,42.32,0,0,0,2.19-12.54c0-.32,0-.63,0-1a42.59,42.59,0,0,0-1.61-11.61,42.1,42.1,0,0,0-5.41-11.76c-.57-.87-1.2-1.74-1.85-2.57a2,2,0,0,0-2.28-.68,2,2,0,0,0-1.4,1.95v19.7a6.31,6.31,0,0,0,1.72,4.35Z" style="fill:#fff"/><path d="M16.36,549.9a43,43,0,0,0,3.76-6.2,4.76,4.76,0,0,0,.52-2.18,4.84,4.84,0,0,0-.19-1.35,35.18,35.18,0,0,0-4.5-9.79,35.67,35.67,0,0,0-4-5V555l.16-.16A43,43,0,0,0,16.36,549.9Z" style="fill:#fff"/><path d="M38.33,567.1a1.37,1.37,0,0,0-.07-.42,1.32,1.32,0,0,0-1.32-.95h0a35.14,35.14,0,0,0-10.44,1.83,34.77,34.77,0,0,0-8.13,4l-.13.14L18,572a1,1,0,0,1-.77.35l-.1.06-.2.15c-.73.53-1.49,1.14-2.26,1.82-.37.42-.75.83-1.14,1.24H27.29a4.6,4.6,0,0,0,3.31-1.41l.44-.45a45.61,45.61,0,0,1,6.69-5.5A1.32,1.32,0,0,0,38.33,567.1Z" style="fill:#fff"/><path d="M820.53,39a43.57,43.57,0,0,0-3.76,6.2,4.85,4.85,0,0,0-.33,3.53,35.05,35.05,0,0,0,4.5,9.79,35.75,35.75,0,0,0,4,5V34l-.16.15A43,43,0,0,0,820.53,39Z" style="fill:#fff"/><path d="M47.35,567.06h0a4.92,4.92,0,0,0-3.51.33,42.73,42.73,0,0,0-6.2,3.77,44.8,44.8,0,0,0-4.91,4.22l-.19.2H62.12a35,35,0,0,0-5-4A35.41,35.41,0,0,0,47.35,567.06Z" style="fill:#fff"/><path d="M823,64.4a36.93,36.93,0,0,1-6.44-9.59,36.14,36.14,0,0,1-1.68-4.3.14.14,0,0,0-.14-.1.15.15,0,0,0-.15.11,42.38,42.38,0,0,0-2.19,12.55c0,.31,0,.63,0,.95A42.46,42.46,0,0,0,814,75.62a42,42,0,0,0,5.41,11.77c.57.87,1.2,1.73,1.85,2.57a2.06,2.06,0,0,0,3.68-1.27V69a6.34,6.34,0,0,0-1.72-4.36Z" style="fill:#fff"/><path d="M86,570a42,42,0,0,0-11.77-5.41,42.47,42.47,0,0,0-25.11.6.15.15,0,0,0,0,.28,37.93,37.93,0,0,1,4.3,1.68A36.93,36.93,0,0,1,63,573.6l.27.26a6.34,6.34,0,0,0,4.36,1.72H90.84a.77.77,0,0,0,.72-.48.93.93,0,0,0,0-.29.74.74,0,0,0-.24-.55A43.36,43.36,0,0,0,86,570Z" style="fill:#fff"/><path d="M13.1,90.64A2,2,0,0,0,15.39,90c.65-.84,1.27-1.7,1.84-2.57a42,42,0,0,0,5.41-11.77A42.46,42.46,0,0,0,24.25,64c0-.32,0-.64,0-.95a42.38,42.38,0,0,0-2.19-12.55.14.14,0,0,0-.14-.11.15.15,0,0,0-.15.1,36.09,36.09,0,0,1-1.67,4.3,37,37,0,0,1-6.45,9.59l-.21.23A6.3,6.3,0,0,0,11.7,69v19.7A2,2,0,0,0,13.1,90.64Z" style="fill:#fff"/><path d="M37.35,17.77a43,43,0,0,0,6.2,3.76,4.86,4.86,0,0,0,3.51.34h0a35.13,35.13,0,0,0,9.78-4.5,35.09,35.09,0,0,0,5-4H32.24c.07.07.13.14.2.2A42.15,42.15,0,0,0,37.35,17.77Z" style="fill:#fff"/><path d="M16.61,16.41l.21.14.09.07a1,1,0,0,1,.77.34,3.47,3.47,0,0,0,.25.29l.12.15a34.77,34.77,0,0,0,8.13,4,35.19,35.19,0,0,0,10.44,1.82h.06A1.32,1.32,0,0,0,38,22.25a1.49,1.49,0,0,0,.07-.43,1.32,1.32,0,0,0-.6-1.11,45.67,45.67,0,0,1-6.69-5.51l-.43-.44A4.65,4.65,0,0,0,27,13.34H13.21c.39.41.77.83,1.14,1.25C15.12,15.26,15.88,15.87,16.61,16.41Z" style="fill:#fff"/><path d="M48.83,23.44a.15.15,0,0,0-.1.14.15.15,0,0,0,.1.15,42.5,42.5,0,0,0,12.56,2.19A42.27,42.27,0,0,0,74,24.32a42.1,42.1,0,0,0,11.76-5.41,42.36,42.36,0,0,0,5.39-4.24.74.74,0,0,0,.24-.55.84.84,0,0,0,0-.29.75.75,0,0,0-.71-.49H67.35A6.26,6.26,0,0,0,63,15.07l-.28.25a37.05,37.05,0,0,1-9.58,6.44A37,37,0,0,1,48.83,23.44Z" style="fill:#fff"/><path d="M821.72,18.54l-.09.12a1,1,0,0,1-.33.66l-.18.14-.11.1a35,35,0,0,0-5.9,18.74v.05s0,0,0,0a1.36,1.36,0,0,0,2.49.74,46.13,46.13,0,0,1,5.51-6.68l.41-.4a4.69,4.69,0,0,0,1.41-3.34V14.92c-.39.38-.8.75-1.21,1.11-.67.77-1.28,1.53-1.82,2.26Z" style="fill:#fff"/><path d="M69.29,513.14a26.27,26.27,0,0,0-3.76.14,9.38,9.38,0,0,0-3.88,1.36h.7a26.66,26.66,0,0,1,3.26.48l1.43.26a11.51,11.51,0,0,0,2.07.18,5,5,0,0,0,1-.07l.4-.09a2.46,2.46,0,0,0,.34-.12,1.17,1.17,0,0,1,1.4.26,1.2,1.2,0,0,1,.26,1.4,4.7,4.7,0,0,0-.25.94,7.83,7.83,0,0,0,0,1,16.36,16.36,0,0,0,.27,2.34c0,.24.09.48.14.73a21.48,21.48,0,0,1,.5,4.14v0a9.56,9.56,0,0,0,1.43-4.48c.09-1.07.09-2.18.09-3.26a22.4,22.4,0,0,1,.49-5.72,16.49,16.49,0,0,1-3.88.5Z" style="fill:#fff"/><path d="M13.52,32.45A46.13,46.13,0,0,1,19,39.13a1.36,1.36,0,0,0,2.49-.74V38.3a35.47,35.47,0,0,0-1.83-10.44,35,35,0,0,0-4.08-8.3l-.11-.1-.18-.14a1,1,0,0,1-.33-.66l-.08-.12-.18-.25c-.54-.73-1.15-1.49-1.82-2.26-.41-.36-.81-.73-1.21-1.11V28.71a4.66,4.66,0,0,0,1.42,3.34Z" style="fill:#fff"/><path d="M20.18,48.75a4.77,4.77,0,0,0,.19-1.34,4.92,4.92,0,0,0-.51-2.19A43.63,43.63,0,0,0,16.09,39a43,43,0,0,0-4.23-4.91L11.7,34V63.5a35,35,0,0,0,4-5A34.77,34.77,0,0,0,20.18,48.75Z" style="fill:#fff"/><path d="M69,75.79c.64,0,1.3,0,1.94,0a16.42,16.42,0,0,1,3.88.5,22.48,22.48,0,0,1-.49-5.73c0-1.07,0-2.19-.08-3.26a9.76,9.76,0,0,0-1.44-4.48v0a21.38,21.38,0,0,1-.5,4.13c0,.25-.09.5-.13.74A14.72,14.72,0,0,0,71.91,70a6.42,6.42,0,0,0,0,1,4.14,4.14,0,0,0,.24.94,1.17,1.17,0,0,1-.26,1.39,1.15,1.15,0,0,1-1.4.26,2.74,2.74,0,0,0-.34-.11,2.46,2.46,0,0,0-.39-.09,5.07,5.07,0,0,0-1-.08,12.71,12.71,0,0,0-2.07.19l-1.43.26a26.77,26.77,0,0,1-3.25.47l-.7,0a9.48,9.48,0,0,0,3.87,1.36A27.81,27.81,0,0,0,69,75.79Z" style="fill:#fff"/><path d="M798.56,21.82a1.49,1.49,0,0,0,.07.43,1.33,1.33,0,0,0,1.32.94h0a35.13,35.13,0,0,0,10.44-1.82,34.77,34.77,0,0,0,8.13-4l.13-.15.24-.29a1,1,0,0,1,.77-.34l.1-.07.2-.14c.73-.54,1.49-1.15,2.26-1.82.37-.42.75-.84,1.14-1.25H809.6a4.61,4.61,0,0,0-3.31,1.42l-.44.44a45.15,45.15,0,0,1-6.69,5.51A1.32,1.32,0,0,0,798.56,21.82Z" style="fill:#fff"/><path d="M767.6,75.79a27.81,27.81,0,0,0,3.76-.14,9.68,9.68,0,0,0,3.88-1.36l-.7,0a28.74,28.74,0,0,1-3.26-.47l-1.43-.26a12.61,12.61,0,0,0-2.07-.19,4.88,4.88,0,0,0-.95.08,2.35,2.35,0,0,0-.4.09,2.74,2.74,0,0,0-.34.11,1.15,1.15,0,0,1-1.4-.26,1.18,1.18,0,0,1-.26-1.39,4.14,4.14,0,0,0,.24-.94,6.42,6.42,0,0,0,0-1,16,16,0,0,0-.27-2.33c-.05-.24-.09-.49-.14-.74a21.38,21.38,0,0,1-.5-4.13v0a9.64,9.64,0,0,0-1.43,4.48c-.09,1.07-.09,2.19-.09,3.26a22.48,22.48,0,0,1-.49,5.73,16.49,16.49,0,0,1,3.88-.5C766.31,75.77,767,75.78,767.6,75.79Z" style="fill:#fff"/><path d="M816.44,540.17a4.91,4.91,0,0,0,.32,3.53,44,44,0,0,0,3.77,6.2,43,43,0,0,0,4.23,4.91l.16.16V525.43a34.87,34.87,0,0,0-8.48,14.74Z" style="fill:#fff"/><path d="M787.79,565.48a.15.15,0,0,0,0-.28A42.31,42.31,0,0,0,750.91,570a43.36,43.36,0,0,0-5.39,4.25.71.71,0,0,0-.24.55.93.93,0,0,0,.05.29.75.75,0,0,0,.71.48h23.23a6.34,6.34,0,0,0,4.36-1.72l.27-.26a36.93,36.93,0,0,1,9.59-6.44A37.93,37.93,0,0,1,787.79,565.48Z" style="fill:#fff"/><path d="M820,572.52l-.21-.15-.09-.06a1,1,0,0,1-.77-.35l-.24-.29-.13-.14a34.81,34.81,0,0,0-8.13-4A35.45,35.45,0,0,0,800,565.73h0a1.32,1.32,0,0,0-1.32.95,1.37,1.37,0,0,0-.07.42,1.32,1.32,0,0,0,.6,1.12,45,45,0,0,1,6.69,5.5l.44.45a4.6,4.6,0,0,0,3.31,1.41h13.81c-.39-.41-.77-.82-1.14-1.24C821.5,573.66,820.74,573.05,820,572.52Z" style="fill:#fff"/><path d="M799.28,571.16a43.27,43.27,0,0,0-6.2-3.77,4.92,4.92,0,0,0-3.51-.33h0a35,35,0,0,0-14.78,8.52h29.62l-.2-.2A43.81,43.81,0,0,0,799.28,571.16Z" style="fill:#fff"/><path d="M767.6,513.14h-1.94a16.49,16.49,0,0,1-3.88-.5,22.4,22.4,0,0,1,.49,5.72c0,1.08,0,2.19.09,3.27a9.64,9.64,0,0,0,1.43,4.48v0a21.48,21.48,0,0,1,.5-4.14c.05-.24.09-.49.14-.73a16.36,16.36,0,0,0,.27-2.34,6.23,6.23,0,0,0,0-1,4.14,4.14,0,0,0-.24-.94,1.2,1.2,0,0,1,.26-1.4,1.17,1.17,0,0,1,1.4-.26,2.46,2.46,0,0,0,.34.12l.4.09a5,5,0,0,0,.95.07,11.51,11.51,0,0,0,2.07-.18l1.43-.26a26.66,26.66,0,0,1,3.26-.48h.7a9.38,9.38,0,0,0-3.88-1.36A26.27,26.27,0,0,0,767.6,513.14Z" style="fill:#fff"/><path d="M789.54,21.86h0a4.87,4.87,0,0,0,3.51-.34,43.57,43.57,0,0,0,6.2-3.76,43,43,0,0,0,4.91-4.23l.19-.2H774.77a35,35,0,0,0,5,4A35.13,35.13,0,0,0,789.54,21.86Z" style="fill:#fff"/><path d="M823.52,498.29a2,2,0,0,0-2.28.68c-.65.83-1.28,1.7-1.85,2.57A42.1,42.1,0,0,0,814,513.3a42.59,42.59,0,0,0-1.61,11.61c0,.32,0,.63,0,1a42.37,42.37,0,0,0,2.19,12.54.15.15,0,0,0,.15.12.17.17,0,0,0,.14-.11,38.62,38.62,0,0,1,1.67-4.3,37.39,37.39,0,0,1,6.45-9.59l.22-.23a6.31,6.31,0,0,0,1.72-4.35v-19.7A2,2,0,0,0,823.52,498.29Z" style="fill:#fff"/><path d="M750.91,18.91a42,42,0,0,0,11.77,5.41,42.2,42.2,0,0,0,12.55,1.6,42.5,42.5,0,0,0,12.56-2.19.15.15,0,0,0,.1-.15.15.15,0,0,0-.1-.14,36.14,36.14,0,0,1-4.3-1.68,36.93,36.93,0,0,1-9.59-6.44l-.27-.25a6.3,6.3,0,0,0-4.36-1.73H746.05a.76.76,0,0,0-.72.49.84.84,0,0,0-.05.29.74.74,0,0,0,.24.55A42.36,42.36,0,0,0,750.91,18.91Z" style="fill:#fff"/><path d="M0,0V590.28H836.89V0ZM9.67,12.33a1,1,0,0,1,.17-.57v-.2h0v0h.21a1,1,0,0,1,.62-.21H120.18a1,1,0,0,1,0,2H97.51a4.21,4.21,0,0,0-3,1.28c-.59.61-1.2,1.21-1.81,1.78a44.28,44.28,0,0,1-11.5,7.72,44.67,44.67,0,0,1-12.88,3.76,45.57,45.57,0,0,1-13.5-.24,43.74,43.74,0,0,1-10.34-3.09c-.77.15-1.45.26-2.08.35a38.28,38.28,0,0,1-11.26-.2c-.84-.14-1.69-.32-2.53-.52a1.55,1.55,0,0,0-1.39,2.67,88.21,88.21,0,0,0,8.32,6.57,43.52,43.52,0,0,0,4.65,2.94,9.49,9.49,0,0,0,4.39,1.24,5.06,5.06,0,0,0,2-.4l.12-.05.21-.1c.17-.09.34-.18.5-.28a3.34,3.34,0,0,0,.34-.23,1.31,1.31,0,0,1,.16-.13,1.54,1.54,0,0,0,.19-.17,3.29,3.29,0,0,0,1-1.45A2.94,2.94,0,0,0,49,33c-.42-1.4-1.82-2.91-3.25-2.92a1.6,1.6,0,0,0-.88.24l-.18.12-.15.15,0,0s0,0,0,.05h0a1,1,0,0,1-.07.15l0,.11a.75.75,0,0,1,0,.15v.14a1.42,1.42,0,0,1,.05.16.54.54,0,0,0,.07.13l0,0h0l0,0,.09.08h0l0,0a1.14,1.14,0,0,1,.79,1.37,1.16,1.16,0,0,1-.55.7,1.09,1.09,0,0,1-.84.11,2.81,2.81,0,0,1-1.92-2.09,3.08,3.08,0,0,1,.85-2.95,3.85,3.85,0,0,1,2.89-1,5.32,5.32,0,0,1,3,1.31,6.33,6.33,0,0,1,2.39,5.27,5.92,5.92,0,0,1-3,4.41,8,8,0,0,1-5.45.94,11.83,11.83,0,0,1-3.22-1C42.52,40.89,45.38,43,48.27,45a61.42,61.42,0,0,0,7.84,4.83,15,15,0,0,0,7.82,1.8,10.24,10.24,0,0,0,3.25-.9,9.78,9.78,0,0,0,1.6-1l.17-.13.4-.34a6.85,6.85,0,0,0,.66-.66,6.25,6.25,0,0,0,.9-1.28l.18-.37a1.19,1.19,0,0,0,.08-.2l0,0a7.1,7.1,0,0,0,.25-.87,3.25,3.25,0,0,0,.08-.43.37.37,0,0,0,0-.11c0-.06,0-.12,0-.14a7.23,7.23,0,0,0,0-1,7.08,7.08,0,0,0-.45-1.8,8.59,8.59,0,0,0-2.26-3.19,5.93,5.93,0,0,0-3.37-1.67,3.36,3.36,0,0,0-3,1.2l-.07.12-.07.1-.11.22s0,0,0,0h0c0,.13-.07.26-.1.38s0,0,0,.08a1.7,1.7,0,0,0,0,.22,1.61,1.61,0,0,0,.08.61,1.09,1.09,0,0,0,.05.17v0a.34.34,0,0,1,0,.08,3.27,3.27,0,0,0,.18.33l0,.07h0l.08.09,0,0c.06.07.14.14.2.2l.05,0,0,0,.13.08.08,0,.14.05h0a1.1,1.1,0,0,1,.77.78,1.24,1.24,0,0,1-.33,1.17,1,1,0,0,1-1,.24,4,4,0,0,1-2.77-3.23,4.38,4.38,0,0,1,1.61-4.18,5.91,5.91,0,0,1,4.69-1.16,8.69,8.69,0,0,1,4.39,2.38,9.56,9.56,0,0,1,3.07,8.63,9.44,9.44,0,0,1-5.2,6.26,12.88,12.88,0,0,1-8.69.91,28.07,28.07,0,0,1-8.58-3.89c-3.55-2.24-7.15-4.77-10.72-7.54,2.21,2.87,4.32,5.8,6.27,8.74,3.43,5.16,6.93,11.17,5,17.29a10.59,10.59,0,0,1-5.09,6.46,8.64,8.64,0,0,1-8.75-.85c-2.46-1.69-5-4.93-4.12-8.47a5.09,5.09,0,0,1,3.08-3.59,4.42,4.42,0,0,1,4.23.79,3.63,3.63,0,0,1,1.17,1.82,1.06,1.06,0,0,1-.76,1.34,1.1,1.1,0,0,1-1.42-.76s0-.08,0-.12l0-.11h0c0-.05-.05-.1-.08-.14h0l-.07-.09-.08-.09-.11-.11h0l-.1-.08-.07,0-.06,0a2.12,2.12,0,0,0-1.28-.33H38a3.59,3.59,0,0,0-.41.11h0l-.21.11a2.53,2.53,0,0,0-.31.19,3.1,3.1,0,0,0-.91,1.24c-.91,2.4.88,5,2.94,6.43a7.22,7.22,0,0,0,3.48,1.36,5.57,5.57,0,0,0,3.22-.65,8,8,0,0,0,.73-.45,6.58,6.58,0,0,0,.59-.49,8.11,8.11,0,0,0,.58-.59,6.78,6.78,0,0,0,.77-1,12.71,12.71,0,0,0,.66-1.17,9.48,9.48,0,0,0,.59-1.63A9.85,9.85,0,0,0,50,63.51a15.5,15.5,0,0,0-1-3.93A38.12,38.12,0,0,0,44.78,52c-2.42-3.64-5-7.21-7.78-10.65a9.44,9.44,0,0,1,.73,7.49A6.53,6.53,0,0,1,34,52.74a5.72,5.72,0,0,1-5.57-1.14c-1.87-1.47-3.06-4.09-1.75-6.24a3.12,3.12,0,0,1,2.93-1.58,2.89,2.89,0,0,1,2.59,2,1.06,1.06,0,0,1-.1.83,1.17,1.17,0,0,1-.7.56,1.15,1.15,0,0,1-1.38-.8v0s0,0,0,0v0h0l-.08-.09-.09-.06h0l-.12-.05-.14,0h-.26l-.14,0h0a.88.88,0,0,0-.27.19l0,.06a1.37,1.37,0,0,0-.33.64c-.33,1.34,1,2.69,2,3.26a3.54,3.54,0,0,0,1.94.56A2.83,2.83,0,0,0,34,50.25l.14-.1.1-.07c.13-.12.27-.25.39-.38a3.37,3.37,0,0,0,.54-.69,5.28,5.28,0,0,0,.74-1.91,6.12,6.12,0,0,0,0-2,10.46,10.46,0,0,0-.68-2.3,27.48,27.48,0,0,0-2.58-4.45,90.69,90.69,0,0,0-7.43-9.56l-.07-.07a1.5,1.5,0,0,0-1.87-.34,1.53,1.53,0,0,0-.79,1.76c.22.87.4,1.76.55,2.63a37.94,37.94,0,0,1,.2,11.26c-.08.63-.2,1.31-.34,2.08A43.91,43.91,0,0,1,26,56.49,45.51,45.51,0,0,1,26.2,70a44.62,44.62,0,0,1-3.75,12.89,44.74,44.74,0,0,1-7.73,11.5h0a11.13,11.13,0,0,0-3,7.57V121.8a1,1,0,0,1-2,0Zm67.19,506a26.33,26.33,0,0,1-.29,5,11.54,11.54,0,0,1-2.1,4.72,12.81,12.81,0,0,1-4.33,3.69A7.23,7.23,0,0,1,67,532.9a11.2,11.2,0,0,1-1.66.23,1.09,1.09,0,0,1-.94-.48,1.06,1.06,0,0,1-.43-.87,1.22,1.22,0,0,1,.41-.88,1,1,0,0,1,.74-.25,10.86,10.86,0,0,0,1.1.06l.41,0a9.39,9.39,0,0,0,1.18-.34,8.55,8.55,0,0,0,1.2-.52A4.22,4.22,0,0,0,70.51,528l.06-.18s0-.08,0-.12a3.11,3.11,0,0,0,.11-.41c0-.12.06-.25.08-.37s0-.12,0-.25a9.61,9.61,0,0,0-.07-2c-.09-.67-.22-1.34-.36-2.06a15.74,15.74,0,0,1-.44-4.77,12.3,12.3,0,0,1-3.09-.13c-.57-.08-1.14-.19-1.69-.3-.84-.16-1.7-.33-2.56-.41a8.59,8.59,0,0,0-1.76,0,5.54,5.54,0,0,0-.87.21,3.31,3.31,0,0,0-.57.25,3.62,3.62,0,0,0-1.13.85,3.41,3.41,0,0,0-.28.33h0l-.07.1a9.6,9.6,0,0,0-.87,2.42s0,.09,0,.14,0,.26,0,.39a7.94,7.94,0,0,0,.06,1,1,1,0,0,1-.25.75,1.24,1.24,0,0,1-.88.41,1,1,0,0,1-.86-.43,1.09,1.09,0,0,1-.49-.88v-.06a11.58,11.58,0,0,1,.22-1.63,7.28,7.28,0,0,1,.77-2.59,4.58,4.58,0,0,1,.36-.61c.17-.31.31-.57.47-.81a12.21,12.21,0,0,1,8.68-5.69,30.75,30.75,0,0,1,4.17-.17h1.93a10.85,10.85,0,0,0,5.15-1.17A1.27,1.27,0,0,1,78,511.38a11.64,11.64,0,0,0-1.18,5.85ZM56.78,67.67a.61.61,0,0,1,0,.14,9.89,9.89,0,0,0,.87,2.42l.07.1h0c.1.13.2.23.29.33a3.54,3.54,0,0,0,1.13.85,3.13,3.13,0,0,0,.56.24,7.37,7.37,0,0,0,.87.22,8.61,8.61,0,0,0,1.76,0,24.34,24.34,0,0,0,2.56-.4c.56-.11,1.13-.22,1.69-.3a12.33,12.33,0,0,1,3.1-.14,15.73,15.73,0,0,1,.43-4.77c.14-.71.27-1.38.36-2.06a9.52,9.52,0,0,0,.07-2c0-.13,0-.17,0-.25s0-.25-.07-.38-.07-.27-.12-.4a.5.5,0,0,0,0-.13L70.24,61a4.22,4.22,0,0,0-1.48-1.86,10.54,10.54,0,0,0-1.19-.52,11.59,11.59,0,0,0-1.19-.33l-.41,0a7,7,0,0,0-1.09.06,1,1,0,0,1-.75-.25,1.25,1.25,0,0,1-.41-.88,1.06,1.06,0,0,1,.44-.87,1,1,0,0,1,.93-.48,12.63,12.63,0,0,1,1.66.22,7.4,7.4,0,0,1,3.13,1.12,13,13,0,0,1,4.32,3.69,11.45,11.45,0,0,1,2.1,4.73,26.22,26.22,0,0,1,.29,5V71.7a11.58,11.58,0,0,0,1.18,5.84,1.28,1.28,0,0,1-1.67,1.68A10.85,10.85,0,0,0,71,78.05c-.64,0-1.3,0-1.93,0a30.77,30.77,0,0,1-4.17-.18,12.17,12.17,0,0,1-8.68-5.69,8.06,8.06,0,0,1-.46-.81,6.23,6.23,0,0,1-.37-.61,7.17,7.17,0,0,1-.77-2.58,11.72,11.72,0,0,1-.22-1.64v0a1.07,1.07,0,0,1,.49-.88,1,1,0,0,1,.86-.43,1.2,1.2,0,0,1,.88.41,1,1,0,0,1,.25.74,8.07,8.07,0,0,0-.06,1C56.77,67.41,56.78,67.54,56.78,67.67Zm63.67,509.94H11a1,1,0,0,1-.62-.2h-.21v0h0v-.21a1,1,0,0,1-.17-.56V467.12a1,1,0,0,1,2,0V487a11.09,11.09,0,0,0,3,7.57l0,0a44.28,44.28,0,0,1,7.72,11.5,44.26,44.26,0,0,1,3.76,12.88,45.58,45.58,0,0,1-.24,13.5,43.74,43.74,0,0,1-3.09,10.34c.15.77.26,1.45.35,2.08a38.28,38.28,0,0,1-.2,11.26c-.15.87-.33,1.75-.55,2.63a1.53,1.53,0,0,0,.79,1.75,1.5,1.5,0,0,0,1.87-.34l.06-.07a86.22,86.22,0,0,0,7.43-9.56,26.76,26.76,0,0,0,2.59-4.44,10.38,10.38,0,0,0,.68-2.31,6.12,6.12,0,0,0,0-2,5.08,5.08,0,0,0-.75-1.9,3,3,0,0,0-.53-.69,3.48,3.48,0,0,0-.39-.38l-.1-.08-.14-.1a2.85,2.85,0,0,0-1.62-.54,3.63,3.63,0,0,0-1.94.56c-1,.58-2.3,1.93-2,3.27a1.53,1.53,0,0,0,.32.64l.06,0a1.11,1.11,0,0,0,.27.2h0l.14,0h.26l.14,0,.12-.06h0l.08-.06.09-.08h0v0l0,0v0a1.13,1.13,0,0,1,1.37-.8,1.15,1.15,0,0,1,.71.55,1.08,1.08,0,0,1,.1.84,2.91,2.91,0,0,1-2.59,2A3.13,3.13,0,0,1,27,543.56c-1.32-2.14-.12-4.76,1.74-6.24a5.79,5.79,0,0,1,5.58-1.14A6.58,6.58,0,0,1,38,540.1a9.42,9.42,0,0,1-.73,7.48c2.75-3.43,5.36-7,7.78-10.64a38.7,38.7,0,0,0,4.22-7.6,15.3,15.3,0,0,0,1-3.93,9.8,9.8,0,0,0-.27-3.66,8.57,8.57,0,0,0-.6-1.64,10.5,10.5,0,0,0-.65-1.16,6.3,6.3,0,0,0-.77-1c-.19-.21-.39-.41-.59-.6s-.42-.36-.58-.48a8.09,8.09,0,0,0-.73-.46,5.57,5.57,0,0,0-3.22-.65,7.34,7.34,0,0,0-3.48,1.37c-2.06,1.41-3.85,4-2.94,6.42a3.1,3.1,0,0,0,.91,1.24,1.94,1.94,0,0,0,.31.2l.2.1h0a3.06,3.06,0,0,0,.41.1h.18a2.27,2.27,0,0,0,1.28-.33l.06,0,.07,0a.39.39,0,0,1,.1-.08h0l.11-.11.08-.09.07-.09h0s0-.1.08-.14h0a.37.37,0,0,1,.05-.11.34.34,0,0,1,0-.12,1.1,1.1,0,0,1,1.42-.76,1.06,1.06,0,0,1,.76,1.34,3.6,3.6,0,0,1-1.17,1.81,4.37,4.37,0,0,1-4.23.79,5,5,0,0,1-3.08-3.59c-.9-3.54,1.65-6.77,4.12-8.47a8.66,8.66,0,0,1,8.75-.85A10.61,10.61,0,0,1,52,520.78c1.89,6.12-1.61,12.14-5,17.3-2,2.93-4.06,5.86-6.27,8.73,3.57-2.76,7.17-5.29,10.72-7.53a28.09,28.09,0,0,1,8.58-3.9,12.94,12.94,0,0,1,8.69.91,9.44,9.44,0,0,1,5.2,6.26,9.55,9.55,0,0,1-3.07,8.63,8.62,8.62,0,0,1-4.39,2.38,5.91,5.91,0,0,1-4.69-1.16,4.37,4.37,0,0,1-1.61-4.17A4,4,0,0,1,62.93,545a1,1,0,0,1,1,.23,1.26,1.26,0,0,1,.33,1.17,1.09,1.09,0,0,1-.77.78h0l-.14,0-.08,0a.57.57,0,0,0-.13.08l0,0,0,0-.21.19,0,0-.07.1h0l0,.06a3.51,3.51,0,0,0-.18.34.69.69,0,0,1,0,.08v0a1.21,1.21,0,0,0,0,.18,1.55,1.55,0,0,0-.08.6,1.62,1.62,0,0,0,0,.22s0,.07,0,.09.06.24.1.37v0l.11.21.06.1s.06.08.08.12a3.34,3.34,0,0,0,3,1.2,5.92,5.92,0,0,0,3.37-1.66,8.51,8.51,0,0,0,2.27-3.19,6.22,6.22,0,0,0,.46-2.85,1.23,1.23,0,0,1,0-.13s0-.09,0-.12-.05-.28-.08-.43a6.1,6.1,0,0,0-.26-.86v0a.88.88,0,0,0-.09-.2,3,3,0,0,0-.18-.36,5.91,5.91,0,0,0-.89-1.29,8.13,8.13,0,0,0-.66-.65l-.4-.35-.17-.12a9.78,9.78,0,0,0-1.6-.95,10.24,10.24,0,0,0-3.25-.9,15.08,15.08,0,0,0-7.82,1.79,62.61,62.61,0,0,0-7.84,4.84c-2.89,2-5.75,4.12-8.52,6.33a11.78,11.78,0,0,1,3.22-1,8,8,0,0,1,5.44.93,5.9,5.9,0,0,1,3,4.42,6.3,6.3,0,0,1-2.39,5.27,5.34,5.34,0,0,1-3,1.31,3.89,3.89,0,0,1-2.89-1,3.09,3.09,0,0,1-.85-3,2.82,2.82,0,0,1,1.92-2.08,1.06,1.06,0,0,1,.83.1,1.12,1.12,0,0,1-.23,2.08h0l0,0-.09.07,0,0h0l0,.05-.06.12c0,.05,0,.1,0,.17v0s0,0,0,0V558l0,.12.07.14v0l0,0,0,0,.15.16a.44.44,0,0,1,.17.12,1.72,1.72,0,0,0,.89.24c1.42,0,2.83-1.52,3.25-2.93a2.94,2.94,0,0,0,0-1.83,3.21,3.21,0,0,0-1-1.44l-.19-.18a.57.57,0,0,1-.16-.13l-.34-.23a4.82,4.82,0,0,0-.5-.27l-.21-.11-.12,0a5.29,5.29,0,0,0-2-.4,9.58,9.58,0,0,0-4.39,1.24,43.15,43.15,0,0,0-4.66,3,88.09,88.09,0,0,0-8.31,6.57,1.55,1.55,0,0,0,1.39,2.67c.84-.21,1.69-.38,2.53-.52a37.61,37.61,0,0,1,11.26-.2c.63.08,1.31.19,2.08.34a44.55,44.55,0,0,1,10.34-3.09,45.24,45.24,0,0,1,13.5-.23A44.38,44.38,0,0,1,93,572.53c.61.56,1.22,1.16,1.81,1.77a4.17,4.17,0,0,0,3,1.28h22.67a1,1,0,0,1,0,2Zm596-566.3H825.91a1,1,0,0,1,.62.21h.21v0h0v.2a1,1,0,0,1,.17.57V121.8a1,1,0,0,1-2,0V102a11.13,11.13,0,0,0-3-7.57h0A44.55,44.55,0,0,1,810.42,70a45.51,45.51,0,0,1,.24-13.49,43.47,43.47,0,0,1,3.09-10.34c-.15-.77-.26-1.45-.35-2.08a38.28,38.28,0,0,1,.2-11.26c.15-.87.33-1.76.55-2.63a1.53,1.53,0,0,0-.79-1.76,1.5,1.5,0,0,0-1.87.34l-.06.07A88.12,88.12,0,0,0,804,38.39a26.84,26.84,0,0,0-2.59,4.45,10.46,10.46,0,0,0-.68,2.3,6.12,6.12,0,0,0,0,2,5.13,5.13,0,0,0,.75,1.91,3,3,0,0,0,.53.69c.12.13.26.26.39.38l.1.07.14.1a2.85,2.85,0,0,0,1.62.54,3.54,3.54,0,0,0,1.94-.56c1-.57,2.3-1.92,2-3.26a1.47,1.47,0,0,0-.32-.64l-.06-.06a.88.88,0,0,0-.27-.19h0l-.14,0h-.26l-.14,0-.12.05h0l-.08.06a.75.75,0,0,0-.09.09h0v0l0,0v0a1.14,1.14,0,0,1-1.37.8,1.16,1.16,0,0,1-.71-.56,1.06,1.06,0,0,1-.1-.83,2.89,2.89,0,0,1,2.59-2,3.12,3.12,0,0,1,2.93,1.58c1.32,2.15.12,4.77-1.74,6.24a5.74,5.74,0,0,1-5.58,1.14,6.56,6.56,0,0,1-3.71-3.91,9.44,9.44,0,0,1,.73-7.49c-2.75,3.44-5.36,7-7.78,10.65a38.63,38.63,0,0,0-4.22,7.59,15.5,15.5,0,0,0-1,3.93,9.85,9.85,0,0,0,.27,3.67,8.72,8.72,0,0,0,.6,1.63,10.6,10.6,0,0,0,.65,1.17,6.78,6.78,0,0,0,.77,1,8.21,8.21,0,0,0,.59.59,6.62,6.62,0,0,0,.58.49,8,8,0,0,0,.73.45,5.57,5.57,0,0,0,3.22.65,7.29,7.29,0,0,0,3.49-1.36c2.05-1.42,3.84-4,2.93-6.43a3.1,3.1,0,0,0-.91-1.24,2.53,2.53,0,0,0-.31-.19l-.2-.11h0a3.11,3.11,0,0,0-.41-.11h-.18a2.12,2.12,0,0,0-1.28.33l-.06,0-.07,0-.1.08h0a.57.57,0,0,0-.11.11l-.08.09-.07.09h0s0,.09-.08.14h0l-.05.11a.42.42,0,0,1,0,.12,1.1,1.1,0,0,1-1.42.76,1.06,1.06,0,0,1-.76-1.34,3.63,3.63,0,0,1,1.17-1.82,4.42,4.42,0,0,1,4.23-.79,5.07,5.07,0,0,1,3.08,3.59c.9,3.54-1.65,6.78-4.12,8.47a8.64,8.64,0,0,1-8.75.85,10.59,10.59,0,0,1-5.09-6.46C783,62,786.46,56,789.89,50.85c2-2.94,4.06-5.87,6.27-8.74-3.57,2.77-7.17,5.3-10.72,7.54a27.85,27.85,0,0,1-8.58,3.89,12.88,12.88,0,0,1-8.69-.91,9.44,9.44,0,0,1-5.2-6.26A9.56,9.56,0,0,1,766,37.74a8.69,8.69,0,0,1,4.39-2.38,5.91,5.91,0,0,1,4.69,1.16,4.38,4.38,0,0,1,1.61,4.18A4,4,0,0,1,774,43.93a1,1,0,0,1-1-.24,1.24,1.24,0,0,1-.33-1.17,1.1,1.1,0,0,1,.77-.78h0l.14-.05.08,0,.13-.08,0,0,.05,0,.21-.2,0,0,.07-.09.05-.07a3.27,3.27,0,0,0,.18-.33.34.34,0,0,1,0-.08v0a1.09,1.09,0,0,0,.05-.17,1.61,1.61,0,0,0,.08-.61,1.7,1.7,0,0,0,0-.22s0-.07,0-.08-.06-.25-.1-.38h0l0,0c0-.08-.07-.15-.11-.22l-.06-.1a.83.83,0,0,1-.08-.12,3.34,3.34,0,0,0-3-1.2,5.88,5.88,0,0,0-3.37,1.67,8.51,8.51,0,0,0-2.27,3.19,7.08,7.08,0,0,0-.45,1.8,6,6,0,0,0,0,1,.88.88,0,0,0,0,.14s0,.08,0,.11,0,.29.08.43a7.31,7.31,0,0,0,.26.87v0a.72.72,0,0,0,.09.2,2.53,2.53,0,0,0,.18.37,5.85,5.85,0,0,0,.89,1.28,6.85,6.85,0,0,0,.66.66l.4.34.17.13a9.89,9.89,0,0,0,4.85,1.85,15,15,0,0,0,7.82-1.8A61.42,61.42,0,0,0,788.35,45c2.89-2,5.75-4.12,8.52-6.34a11.83,11.83,0,0,1-3.22,1,8,8,0,0,1-5.44-.94,5.87,5.87,0,0,1-3-4.41,6.28,6.28,0,0,1,2.39-5.27,5.27,5.27,0,0,1,3-1.31,3.85,3.85,0,0,1,2.89,1,3.08,3.08,0,0,1,.85,2.95,2.81,2.81,0,0,1-1.92,2.09,1.06,1.06,0,0,1-.83-.11,1.12,1.12,0,0,1,.23-2.07l0,0h0l.09-.08,0,0h0l0,0,.06-.13a1.42,1.42,0,0,1,.05-.16v-.05a0,0,0,0,0,0,0v-.21l0-.11-.07-.15h0l0-.05,0,0-.15-.15a.88.88,0,0,1-.17-.12,1.63,1.63,0,0,0-.89-.24c-1.42,0-2.83,1.52-3.25,2.92a2.94,2.94,0,0,0,0,1.83,3.29,3.29,0,0,0,1,1.45,1.54,1.54,0,0,0,.19.17.83.83,0,0,1,.16.13c.1.08.22.15.34.23s.33.19.5.28l.21.1.12.05a5.1,5.1,0,0,0,2,.4,9.46,9.46,0,0,0,4.39-1.24,44.72,44.72,0,0,0,4.66-2.94,88.09,88.09,0,0,0,8.31-6.57A1.55,1.55,0,0,0,808,24.18c-.84.2-1.69.38-2.53.52a38.28,38.28,0,0,1-11.26.2c-.63-.09-1.31-.2-2.08-.35a43.61,43.61,0,0,1-10.34,3.09,45.51,45.51,0,0,1-13.49.24,44.64,44.64,0,0,1-12.89-3.76,44.45,44.45,0,0,1-11.5-7.72c-.61-.57-1.22-1.17-1.81-1.78a4.21,4.21,0,0,0-3-1.28H716.44a1,1,0,0,1,0-2Zm63.4,510s0-.09,0-.14a9.6,9.6,0,0,0-.87-2.42l-.07-.1v0c-.1-.12-.19-.23-.28-.32a3.47,3.47,0,0,0-1.13-.85,3.68,3.68,0,0,0-.57-.25A5.54,5.54,0,0,0,776,517a8,8,0,0,0-1.76,0,24.34,24.34,0,0,0-2.56.4c-.55.11-1.13.22-1.69.3a12.3,12.3,0,0,1-3.09.13,15.74,15.74,0,0,1-.44,4.77c-.14.72-.27,1.39-.36,2.06a9.61,9.61,0,0,0-.07,2c0,.13,0,.16,0,.25s0,.25.08.37a3.11,3.11,0,0,0,.11.41s0,.08,0,.12l.06.18a4.22,4.22,0,0,0,1.48,1.86,8.55,8.55,0,0,0,1.2.52,9.39,9.39,0,0,0,1.18.34l.41,0a10.86,10.86,0,0,0,1.1-.06,1,1,0,0,1,.74.25,1.24,1.24,0,0,1,.41.88,1.06,1.06,0,0,1-.43.87,1.09,1.09,0,0,1-.94.48,11.2,11.2,0,0,1-1.66-.23,7.23,7.23,0,0,1-3.12-1.12,12.81,12.81,0,0,1-4.33-3.69,11.41,11.41,0,0,1-2.1-4.72,26.33,26.33,0,0,1-.29-5v-1.14a11.64,11.64,0,0,0-1.18-5.85,1.27,1.27,0,0,1,1.67-1.67,10.85,10.85,0,0,0,5.15,1.17h1.93a30.75,30.75,0,0,1,4.17.17,12.21,12.21,0,0,1,8.68,5.69c.16.24.3.5.47.81a4.58,4.58,0,0,1,.36.61,7.28,7.28,0,0,1,.77,2.59,11.58,11.58,0,0,1,.22,1.63v.06a1.07,1.07,0,0,1-.49.88,1,1,0,0,1-.86.43,1.21,1.21,0,0,1-.88-.42,1,1,0,0,1-.25-.74,7.94,7.94,0,0,0,.06-1C779.85,521.52,779.85,521.39,779.84,521.26ZM760,70.55a26.18,26.18,0,0,1,.29-5,11.58,11.58,0,0,1,2.1-4.73,12.93,12.93,0,0,1,4.33-3.69A7.36,7.36,0,0,1,769.87,56a11.14,11.14,0,0,1,1.66-.22,1.07,1.07,0,0,1,.94.48,1,1,0,0,1,.43.87,1.25,1.25,0,0,1-.41.88,1,1,0,0,1-.74.25,7.08,7.08,0,0,0-1.1-.06l-.41,0a11.41,11.41,0,0,0-1.18.33,9.91,9.91,0,0,0-1.2.52A4.22,4.22,0,0,0,766.38,61l-.06.18s0,.08,0,.13a3,3,0,0,0-.11.4c0,.13-.06.25-.08.38s0,.12,0,.25a9.52,9.52,0,0,0,.07,2c.09.68.22,1.35.36,2.06a15.74,15.74,0,0,1,.44,4.77,12.27,12.27,0,0,1,3.09.14c.57.08,1.14.19,1.69.3.84.16,1.7.32,2.56.4A8.61,8.61,0,0,0,776,72a6.9,6.9,0,0,0,.87-.22,3.53,3.53,0,0,0,.57-.24,3.78,3.78,0,0,0,1.13-.85c.09-.1.18-.2.28-.33h0l.07-.1a9.89,9.89,0,0,0,.87-2.42s0-.1,0-.14,0-.26,0-.4a8.07,8.07,0,0,0-.06-1,1,1,0,0,1,.25-.74,1.2,1.2,0,0,1,.88-.41,1,1,0,0,1,.86.43,1.07,1.07,0,0,1,.49.88v0a11.72,11.72,0,0,1-.22,1.64,7.17,7.17,0,0,1-.77,2.58,4.58,4.58,0,0,1-.36.61,8,8,0,0,1-.47.81,12.16,12.16,0,0,1-8.68,5.69,30.77,30.77,0,0,1-4.17.18c-.63,0-1.29,0-1.93,0a10.85,10.85,0,0,0-5.15,1.17,1.28,1.28,0,0,1-1.67-1.68A11.58,11.58,0,0,0,760,71.7ZM827,576.6a1,1,0,0,1-.17.56v.21h0v0h-.21a1,1,0,0,1-.62.2H716.44a1,1,0,0,1,0-2h22.67a4.17,4.17,0,0,0,3-1.28c.59-.61,1.2-1.21,1.81-1.77a44.38,44.38,0,0,1,24.38-11.48,45.24,45.24,0,0,1,13.5.23,44.55,44.55,0,0,1,10.34,3.09c.77-.15,1.45-.26,2.08-.34a37.61,37.61,0,0,1,11.26.2c.84.14,1.69.31,2.53.52a1.55,1.55,0,0,0,1.39-2.67,89.39,89.39,0,0,0-8.31-6.57,43.15,43.15,0,0,0-4.66-3,9.58,9.58,0,0,0-4.39-1.24,5.25,5.25,0,0,0-2,.4l-.12,0-.21.1c-.17.09-.34.18-.5.28l-.34.23a.83.83,0,0,1-.16.13l-.19.17a3.29,3.29,0,0,0-1,1.45,2.94,2.94,0,0,0,0,1.83c.42,1.41,1.83,2.92,3.25,2.93a1.75,1.75,0,0,0,.89-.24.64.64,0,0,1,.17-.13l.15-.15,0,0s0,0,0,0v0l.07-.14,0-.12v-.21s0,0,0,0v0c0-.07,0-.12-.05-.17l-.06-.12,0-.05h0l0,0-.09-.07,0,0h0a1.13,1.13,0,0,1-.24-2.08,1.08,1.08,0,0,1,.84-.1,2.82,2.82,0,0,1,1.92,2.08,3.09,3.09,0,0,1-.85,3,3.89,3.89,0,0,1-2.89,1,5.46,5.46,0,0,1-3-1.31,6.32,6.32,0,0,1-2.38-5.27,5.92,5.92,0,0,1,3-4.42,8,8,0,0,1,5.45-.94,12.18,12.18,0,0,1,3.22,1c-2.77-2.22-5.63-4.34-8.52-6.33a61.46,61.46,0,0,0-7.84-4.84,15,15,0,0,0-7.82-1.79,10.24,10.24,0,0,0-3.25.9,9.78,9.78,0,0,0-1.6.95l-.17.12-.4.35a8.13,8.13,0,0,0-.66.65,5.91,5.91,0,0,0-.89,1.29,3,3,0,0,0-.18.36.88.88,0,0,0-.09.2l0,0a7.54,7.54,0,0,0-.25.86c0,.15-.06.29-.08.43s0,.08,0,.12a1.23,1.23,0,0,1,0,.13,7.23,7.23,0,0,0,0,1,7.08,7.08,0,0,0,.45,1.8,8.62,8.62,0,0,0,2.26,3.2,6,6,0,0,0,3.38,1.66,3.34,3.34,0,0,0,3-1.2,1.27,1.27,0,0,0,.07-.12l.07-.1.11-.21v0c0-.13.07-.25.1-.37s0-.06,0-.09a1.62,1.62,0,0,0,0-.22,1.55,1.55,0,0,0-.08-.6,1.21,1.21,0,0,0-.05-.18v0l0-.08a3.51,3.51,0,0,0-.18-.34l-.05-.06-.07-.1,0,0-.21-.19-.05,0,0,0a.57.57,0,0,0-.13-.08l-.08,0-.14,0h0a1.09,1.09,0,0,1-.77-.78,1.26,1.26,0,0,1,.33-1.17,1,1,0,0,1,1-.23,4,4,0,0,1,2.77,3.22,4.38,4.38,0,0,1-1.61,4.18,5.91,5.91,0,0,1-4.69,1.16,8.62,8.62,0,0,1-4.39-2.38,9.55,9.55,0,0,1-3.07-8.63,9.44,9.44,0,0,1,5.2-6.26,12.94,12.94,0,0,1,8.69-.91,28.09,28.09,0,0,1,8.58,3.9c3.55,2.24,7.15,4.77,10.72,7.53-2.21-2.87-4.32-5.8-6.27-8.73-3.43-5.16-6.93-11.18-5-17.3a10.61,10.61,0,0,1,5.09-6.46,8.66,8.66,0,0,1,8.75.85c2.46,1.7,5,4.93,4.12,8.47a5.07,5.07,0,0,1-3.08,3.59,4.37,4.37,0,0,1-4.23-.79,3.64,3.64,0,0,1-1.17-1.81,1.06,1.06,0,0,1,.76-1.34,1.1,1.1,0,0,1,1.42.75.5.5,0,0,1,0,.13.37.37,0,0,1,.05.11h0l.08.13h0l.07.09a.6.6,0,0,0,.08.08h0l.11.12h0l.1.08.07,0,.06,0a2.27,2.27,0,0,0,1.28.33h.18a3.48,3.48,0,0,0,.41-.1h0l.21-.1a1.94,1.94,0,0,0,.31-.2,3.1,3.1,0,0,0,.91-1.24c.91-2.39-.88-5-2.94-6.42a7.34,7.34,0,0,0-3.48-1.37,5.57,5.57,0,0,0-3.22.65,8.09,8.09,0,0,0-.73.46,6.61,6.61,0,0,0-.58.48l-.59.59a6.8,6.8,0,0,0-.77,1,10.5,10.5,0,0,0-.65,1.16,8.57,8.57,0,0,0-.6,1.64,9.8,9.8,0,0,0-.27,3.66,15.3,15.3,0,0,0,1,3.93,38.34,38.34,0,0,0,4.22,7.59c2.42,3.64,5,7.22,7.78,10.65a9.42,9.42,0,0,1-.73-7.48,6.55,6.55,0,0,1,3.71-3.92,5.77,5.77,0,0,1,5.58,1.14c1.86,1.48,3.06,4.1,1.74,6.24a3.13,3.13,0,0,1-2.93,1.59,2.91,2.91,0,0,1-2.59-2,1.08,1.08,0,0,1,.1-.84,1.16,1.16,0,0,1,.7-.55,1.13,1.13,0,0,1,1.38.8v0l0,.06v0h0l.09.08.08.06h0l.12.06.14,0h.26a.58.58,0,0,0,.14,0h0a1.11,1.11,0,0,0,.27-.2l.06,0a1.53,1.53,0,0,0,.32-.64c.34-1.34-1-2.69-2-3.27a3.63,3.63,0,0,0-1.94-.56,2.85,2.85,0,0,0-1.62.54l-.14.1-.1.08a4.51,4.51,0,0,0-.39.37,3.31,3.31,0,0,0-.53.7,5.08,5.08,0,0,0-.75,1.9,6.12,6.12,0,0,0,0,2,10.38,10.38,0,0,0,.68,2.31,26.76,26.76,0,0,0,2.59,4.44,87.34,87.34,0,0,0,7.42,9.56l.07.07a1.5,1.5,0,0,0,1.87.34,1.53,1.53,0,0,0,.79-1.75c-.22-.88-.4-1.76-.55-2.64a37.87,37.87,0,0,1-.2-11.25c.09-.63.2-1.31.34-2.08a44.19,44.19,0,0,1-3.08-10.34,45.58,45.58,0,0,1-.24-13.5,44.48,44.48,0,0,1,11.48-24.38l0,0a11.09,11.09,0,0,0,3-7.57V467.12a1,1,0,0,1,2,0Z" style="fill:#fff"/><path d="M823.1,556.47a45.6,45.6,0,0,1-5.51-6.68,1.36,1.36,0,0,0-2.49.75v.09a35,35,0,0,0,5.9,18.74l.11.09.18.15a1,1,0,0,1,.33.65l.08.12.18.26c.54.73,1.15,1.49,1.82,2.26.41.36.81.73,1.21,1.1V560.21a4.71,4.71,0,0,0-1.41-3.34C823.37,556.74,823.23,556.61,823.1,556.47Z" style="fill:#fff"/></svg>
```

## File: static\src\img\certification_bg_modern.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 841.89 595.28"><polygon points="0 0 0 111.72 644.97 595.28 841.89 595.28 841.89 0 0 0" style="fill:#fff"/></svg>
```

## File: static\src\img\classic-ornament-blue.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 307.99 46.65"><path d="M306.54,42.84H1.44a1,1,0,0,1,0-1.91h305.1a1,1,0,0,1,0,1.91" style="fill:#263e86"/><path d="M222,46.65H86a1,1,0,1,1,0-1.9H222a1,1,0,1,1,0,1.9" style="fill:#263e86"/><path d="M120.49,29.11l.33.34c8.23,8.52,13.68,14.15,23.71,10.2,3.75-1.48,6-4.24,6.44-7.76a11.31,11.31,0,0,0-5-10.35c-2.49-1.72-7.86-4-15.91.19a35.54,35.54,0,0,0-9.61,7.38m17.84,13.76c-7.19,0-12.33-5.32-18.88-12.1l-.26-.27L118,31.77c-4.9,5.43-9.52,10.56-22.12,10.56-6.68,0-12-3.07-14.64-8.44-2.33-4.76-2-10.45.86-14.14,2.55-3.31,6.65-4.58,11.56-3.58,3.58.74,6,2.76,6.92,5.69a10,10,0,0,1-2.5,9.45,6.19,6.19,0,0,1-6.24,1.83A1,1,0,0,1,91.24,32a.94.94,0,0,1,1.18-.64A4.37,4.37,0,0,0,96.77,30a8,8,0,0,0,2-7.55c-.68-2.26-2.58-3.77-5.48-4.37-4.16-.85-7.59.17-9.67,2.87-2.42,3.14-2.68,8-.66,12.14,2.3,4.69,7,7.38,12.93,7.38,11.75,0,15.9-4.61,20.71-9.94l1.23-1.36c-5.78-5.95-13-12.93-23.72-18.27C81,4.31,72,6.53,66.68,9.54c-6.91,3.94-10.43,10.88-10.79,15.65-.4,5.36,1,10.12,3.66,12.72,2.09,2,5,2.84,8.54,2.43,6-.67,6.75-4.58,6.81-6.24.11-2.65-1.35-5.28-3.11-5.63a4.3,4.3,0,0,0-5.12,2.44.95.95,0,0,1-1.77-.69,6.22,6.22,0,0,1,7.25-3.62c2.71.53,4.8,3.93,4.65,7.58-.17,4.46-3.35,7.48-8.49,8.06-4.18.47-7.56-.52-10.08-3-3.14-3-4.68-8.23-4.24-14.24.4-5.26,4.24-12.88,11.75-17.17C71.42,4.64,81.1,2.23,95,9.15c11,5.47,18.3,12.55,24.18,18.59A37.42,37.42,0,0,1,129.23,20c6.52-3.39,13-3.41,17.86-.07a13.24,13.24,0,0,1,5.77,12.14c-.48,4.19-3.26,7.59-7.63,9.31a18.7,18.7,0,0,1-6.9,1.45" style="fill:#263e86"/><path d="M139.75,37.75c-6.95,0-15.23-8-29-21.43C93.54-.38,77,2,76.86,2.07a1,1,0,0,1-1.09-.79,1,1,0,0,1,.79-1.1C77.27.07,94.18-2.41,112.06,15c19.78,19.21,26.38,25,33.6,18.13a.95.95,0,0,1,1.3,1.38c-2.36,2.24-4.7,3.29-7.21,3.29" style="fill:#263e86"/><path d="M39.55,42a16.61,16.61,0,0,1-4.68-.7c-5.22-1.54-6.6-5-6.95-6.91a9,9,0,0,1,3.32-8.58,8.22,8.22,0,0,1,8.83-.64,1,1,0,0,1-.83,1.72,6.28,6.28,0,0,0-6.84.43,7,7,0,0,0-2.61,6.72c.47,2.58,2.47,4.5,5.61,5.43a13.21,13.21,0,0,0,12-2,16.1,16.1,0,0,0,6.53-11.93c0-1.88-.31-4.21-2.13-4.7A3.05,3.05,0,0,0,49,21.5a2,2,0,0,0,.5,2.72,1,1,0,0,1-1.07,1.58,3.93,3.93,0,0,1-1.13-5.15c.59-1.19,2.84-2.18,4.86-1.68,1.39.34,3.72,1.66,3.63,6.6s-3,10.28-7.3,13.44a15,15,0,0,1-9,3" style="fill:#263e86"/><path d="M162.31,19.36a12.55,12.55,0,0,0-7.21,2.18,11.33,11.33,0,0,0-5,10.35c.41,3.52,2.7,6.28,6.45,7.76,10,4,15.47-1.68,23.7-10.2l.33-.34a35.48,35.48,0,0,0-9.6-7.38,18.93,18.93,0,0,0-8.71-2.37m.47,23.51a18.69,18.69,0,0,1-6.89-1.45c-4.37-1.72-7.15-5.12-7.64-9.31A13.26,13.26,0,0,1,154,20c4.84-3.34,11.35-3.32,17.87.07A37.42,37.42,0,0,1,182,27.74c5.88-6,13.21-13.12,24.18-18.59C220,2.23,229.7,4.64,235.38,7.88c7.51,4.29,11.35,11.91,11.74,17.17.45,6-1.1,11.2-4.24,14.24-2.51,2.43-5.9,3.42-10.07,3-5.14-.58-8.32-3.6-8.5-8.06-.14-3.65,1.95-7,4.66-7.58a6.22,6.22,0,0,1,7.24,3.62.95.95,0,1,1-1.77.69,4.28,4.28,0,0,0-5.11-2.44c-1.77.35-3.22,3-3.12,5.63.07,1.66.86,5.57,6.81,6.24,3.58.4,6.46-.41,8.54-2.43,2.7-2.6,4.06-7.35,3.66-12.72-.35-4.77-3.88-11.71-10.78-15.65-5.28-3-14.33-5.23-27.46,1.32C196.26,16.2,189,23.18,183.25,29.13l1.24,1.36c4.8,5.33,8.95,9.94,20.71,9.94,5.92,0,10.63-2.69,12.92-7.38,2-4.13,1.76-9-.65-12.14-2.08-2.7-5.52-3.72-9.67-2.87-2.91.6-4.8,2.11-5.48,4.37a8.05,8.05,0,0,0,2,7.55,4.39,4.39,0,0,0,4.35,1.35.94.94,0,0,1,1.18.64,1,1,0,0,1-.64,1.19A6.16,6.16,0,0,1,203,31.31a9.91,9.91,0,0,1-2.5-9.45c.88-2.93,3.34-4.95,6.92-5.69,4.9-1,9,.27,11.55,3.58,2.85,3.69,3.19,9.38.86,14.14-2.63,5.37-8,8.44-14.63,8.44-12.6,0-17.23-5.13-22.12-10.56-.38-.42-.77-.84-1.15-1.27l-.27.27c-6.54,6.78-11.68,12.1-18.88,12.1" style="fill:#263e86"/><path d="M161.37,37.75c-2.52,0-4.86-1-7.22-3.29a1,1,0,0,1,1.31-1.38c7.21,6.83,13.82,1.08,33.6-18.13C206.93-2.41,223.84.07,224.55.18a1,1,0,0,1,.79,1.1,1,1,0,0,1-1.08.79c-.17,0-16.73-2.41-33.88,14.25-13.79,13.39-22.07,21.43-29,21.43" style="fill:#263e86"/><path d="M261.56,42a15,15,0,0,1-9-3c-4.29-3.16-7.22-8.56-7.31-13.44s2.24-6.26,3.64-6.6c2-.5,4.26.49,4.85,1.68a3.91,3.91,0,0,1-1.13,5.15,1,1,0,1,1-1.06-1.58,2,2,0,0,0,.49-2.72,3,3,0,0,0-2.75-.66c-1.83.49-2.17,2.82-2.14,4.7a16.14,16.14,0,0,0,6.53,11.93,13.22,13.22,0,0,0,12,2c3.15-.93,5.14-2.85,5.62-5.42a7.08,7.08,0,0,0-2.61-6.73,6.28,6.28,0,0,0-6.84-.43,1,1,0,0,1-1.27-.44,1,1,0,0,1,.44-1.28,8.2,8.2,0,0,1,8.82.64,9,9,0,0,1,3.33,8.58c-.36,1.92-1.73,5.37-6.95,6.91a16.68,16.68,0,0,1-4.69.7" style="fill:#263e86"/><path d="M142.43,35.51a6.52,6.52,0,0,1-3.34-.85c-2.08-1.24-3.23-3.71-3.41-7.35-.58-11.3,13.6-20.86,14.2-21.26a.94.94,0,0,1,1.32.26,1,1,0,0,1-.26,1.32c-.14.1-13.88,9.37-13.36,19.58.15,3,1,4.92,2.48,5.81,2.41,1.43,6-.24,6-.26a1,1,0,0,1,.83,1.72,11.44,11.44,0,0,1-4.49,1" style="fill:#263e86"/><path d="M158.62,35.59a10.88,10.88,0,0,1-4.23-1,1,1,0,0,1,.83-1.72s3.36,1.59,5.67.21c1.5-.89,2.34-2.88,2.5-5.91C163.91,17,150.17,7.73,150,7.63a1,1,0,0,1-.27-1.32.94.94,0,0,1,1.32-.26c.6.4,14.78,10,14.2,21.26-.18,3.71-1.34,6.22-3.42,7.46a6.38,6.38,0,0,1-3.24.82" style="fill:#263e86"/><path d="M180.75,25.2a1,1,0,0,1-.58-1.71c.08-.06,8-6.24,6.7-12.4-.16-.77-.53-1.71-1.34-1.87-1.07-.21-2.65.8-3.67,2.34-.47.7-1.86,3.15-.32,5.48a1,1,0,0,1-.26,1.33A1,1,0,0,1,180,18.1a6.73,6.73,0,0,1,.31-7.6c1.5-2.25,3.76-3.51,5.62-3.15.83.16,2.31.81,2.84,3.35,1.53,7.33-7,14-7.4,14.3a.91.91,0,0,1-.58.2" style="fill:#263e86"/><path d="M120.55,25.2A.88.88,0,0,1,120,25c-.37-.28-8.93-7-7.4-14.3.53-2.54,2-3.19,2.84-3.35,1.86-.37,4.11.9,5.61,3.15a6.75,6.75,0,0,1,.32,7.6,1,1,0,0,1-1.32.27,1,1,0,0,1-.27-1.33c1.54-2.33.15-4.78-.31-5.48-1-1.54-2.6-2.55-3.67-2.34-.82.16-1.18,1.1-1.34,1.87-1.29,6.16,6.61,12.34,6.69,12.4a1,1,0,0,1,.18,1.34,1,1,0,0,1-.76.37" style="fill:#263e86"/></svg>

```

## File: static\src\img\classic-ornament-gold.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 307.99 46.65"><path d="M306.54,42.84H1.44a1,1,0,0,1,0-1.91h305.1a1,1,0,0,1,0,1.91" style="fill:#d7a520"/><path d="M222,46.65H86a1,1,0,1,1,0-1.9H222a1,1,0,1,1,0,1.9" style="fill:#d7a520"/><path d="M120.49,29.11l.33.34c8.23,8.52,13.68,14.15,23.71,10.2,3.75-1.48,6-4.24,6.44-7.76a11.31,11.31,0,0,0-5-10.35c-2.49-1.72-7.86-4-15.91.19a35.54,35.54,0,0,0-9.61,7.38m17.84,13.76c-7.19,0-12.33-5.32-18.88-12.1l-.26-.27L118,31.77c-4.9,5.43-9.52,10.56-22.12,10.56-6.68,0-12-3.07-14.64-8.44-2.33-4.76-2-10.45.86-14.14,2.55-3.31,6.65-4.58,11.56-3.58,3.58.74,6,2.76,6.92,5.69a10,10,0,0,1-2.5,9.45,6.19,6.19,0,0,1-6.24,1.83A1,1,0,0,1,91.24,32a.94.94,0,0,1,1.18-.64A4.37,4.37,0,0,0,96.77,30a8,8,0,0,0,2-7.55c-.68-2.26-2.58-3.77-5.48-4.37-4.16-.85-7.59.17-9.67,2.87-2.42,3.14-2.68,8-.66,12.14,2.3,4.69,7,7.38,12.93,7.38,11.75,0,15.9-4.61,20.71-9.94l1.23-1.36c-5.78-5.95-13-12.93-23.72-18.27C81,4.31,72,6.53,66.68,9.54c-6.91,3.94-10.43,10.88-10.79,15.65-.4,5.36,1,10.12,3.66,12.72,2.09,2,5,2.84,8.54,2.43,6-.67,6.75-4.58,6.81-6.24.11-2.65-1.35-5.28-3.11-5.63a4.3,4.3,0,0,0-5.12,2.44.95.95,0,0,1-1.77-.69,6.22,6.22,0,0,1,7.25-3.62c2.71.53,4.8,3.93,4.65,7.58-.17,4.46-3.35,7.48-8.49,8.06-4.18.47-7.56-.52-10.08-3-3.14-3-4.68-8.23-4.24-14.24.4-5.26,4.24-12.88,11.75-17.17C71.42,4.64,81.1,2.23,95,9.15c11,5.47,18.3,12.55,24.18,18.59A37.42,37.42,0,0,1,129.23,20c6.52-3.39,13-3.41,17.86-.07a13.24,13.24,0,0,1,5.77,12.14c-.48,4.19-3.26,7.59-7.63,9.31a18.7,18.7,0,0,1-6.9,1.45" style="fill:#d7a520"/><path d="M139.75,37.75c-6.95,0-15.23-8-29-21.43C93.54-.38,77,2,76.86,2.07a1,1,0,0,1-1.09-.79,1,1,0,0,1,.79-1.1C77.27.07,94.18-2.41,112.06,15c19.78,19.21,26.38,25,33.6,18.13a.95.95,0,0,1,1.3,1.38c-2.36,2.24-4.7,3.29-7.21,3.29" style="fill:#d7a520"/><path d="M39.55,42a16.61,16.61,0,0,1-4.68-.7c-5.22-1.54-6.6-5-6.95-6.91a9,9,0,0,1,3.32-8.58,8.22,8.22,0,0,1,8.83-.64,1,1,0,0,1-.83,1.72,6.28,6.28,0,0,0-6.84.43,7,7,0,0,0-2.61,6.72c.47,2.58,2.47,4.5,5.61,5.43a13.21,13.21,0,0,0,12-2,16.1,16.1,0,0,0,6.53-11.93c0-1.88-.31-4.21-2.13-4.7A3.05,3.05,0,0,0,49,21.5a2,2,0,0,0,.5,2.72,1,1,0,0,1-1.07,1.58,3.93,3.93,0,0,1-1.13-5.15c.59-1.19,2.84-2.18,4.86-1.68,1.39.34,3.72,1.66,3.63,6.6s-3,10.28-7.3,13.44a15,15,0,0,1-9,3" style="fill:#d7a520"/><path d="M162.31,19.36a12.55,12.55,0,0,0-7.21,2.18,11.33,11.33,0,0,0-5,10.35c.41,3.52,2.7,6.28,6.45,7.76,10,4,15.47-1.68,23.7-10.2l.33-.34a35.48,35.48,0,0,0-9.6-7.38,18.93,18.93,0,0,0-8.71-2.37m.47,23.51a18.69,18.69,0,0,1-6.89-1.45c-4.37-1.72-7.15-5.12-7.64-9.31A13.26,13.26,0,0,1,154,20c4.84-3.34,11.35-3.32,17.87.07A37.42,37.42,0,0,1,182,27.74c5.88-6,13.21-13.12,24.18-18.59C220,2.23,229.7,4.64,235.38,7.88c7.51,4.29,11.35,11.91,11.74,17.17.45,6-1.1,11.2-4.24,14.24-2.51,2.43-5.9,3.42-10.07,3-5.14-.58-8.32-3.6-8.5-8.06-.14-3.65,1.95-7,4.66-7.58a6.22,6.22,0,0,1,7.24,3.62.95.95,0,1,1-1.77.69,4.28,4.28,0,0,0-5.11-2.44c-1.77.35-3.22,3-3.12,5.63.07,1.66.86,5.57,6.81,6.24,3.58.4,6.46-.41,8.54-2.43,2.7-2.6,4.06-7.35,3.66-12.72-.35-4.77-3.88-11.71-10.78-15.65-5.28-3-14.33-5.23-27.46,1.32C196.26,16.2,189,23.18,183.25,29.13l1.24,1.36c4.8,5.33,8.95,9.94,20.71,9.94,5.92,0,10.63-2.69,12.92-7.38,2-4.13,1.76-9-.65-12.14-2.08-2.7-5.52-3.72-9.67-2.87-2.91.6-4.8,2.11-5.48,4.37a8.05,8.05,0,0,0,2,7.55,4.39,4.39,0,0,0,4.35,1.35.94.94,0,0,1,1.18.64,1,1,0,0,1-.64,1.19A6.16,6.16,0,0,1,203,31.31a9.91,9.91,0,0,1-2.5-9.45c.88-2.93,3.34-4.95,6.92-5.69,4.9-1,9,.27,11.55,3.58,2.85,3.69,3.19,9.38.86,14.14-2.63,5.37-8,8.44-14.63,8.44-12.6,0-17.23-5.13-22.12-10.56-.38-.42-.77-.84-1.15-1.27l-.27.27c-6.54,6.78-11.68,12.1-18.88,12.1" style="fill:#d7a520"/><path d="M161.37,37.75c-2.52,0-4.86-1-7.22-3.29a1,1,0,0,1,1.31-1.38c7.21,6.83,13.82,1.08,33.6-18.13C206.93-2.41,223.84.07,224.55.18a1,1,0,0,1,.79,1.1,1,1,0,0,1-1.08.79c-.17,0-16.73-2.41-33.88,14.25-13.79,13.39-22.07,21.43-29,21.43" style="fill:#d7a520"/><path d="M261.56,42a15,15,0,0,1-9-3c-4.29-3.16-7.22-8.56-7.31-13.44s2.24-6.26,3.64-6.6c2-.5,4.26.49,4.85,1.68a3.91,3.91,0,0,1-1.13,5.15,1,1,0,1,1-1.06-1.58,2,2,0,0,0,.49-2.72,3,3,0,0,0-2.75-.66c-1.83.49-2.17,2.82-2.14,4.7a16.14,16.14,0,0,0,6.53,11.93,13.22,13.22,0,0,0,12,2c3.15-.93,5.14-2.85,5.62-5.42a7.08,7.08,0,0,0-2.61-6.73,6.28,6.28,0,0,0-6.84-.43,1,1,0,0,1-1.27-.44,1,1,0,0,1,.44-1.28,8.2,8.2,0,0,1,8.82.64,9,9,0,0,1,3.33,8.58c-.36,1.92-1.73,5.37-6.95,6.91a16.68,16.68,0,0,1-4.69.7" style="fill:#d7a520"/><path d="M142.43,35.51a6.52,6.52,0,0,1-3.34-.85c-2.08-1.24-3.23-3.71-3.41-7.35-.58-11.3,13.6-20.86,14.2-21.26a.94.94,0,0,1,1.32.26,1,1,0,0,1-.26,1.32c-.14.1-13.88,9.37-13.36,19.58.15,3,1,4.92,2.48,5.81,2.41,1.43,6-.24,6-.26a1,1,0,0,1,.83,1.72,11.44,11.44,0,0,1-4.49,1" style="fill:#d7a520"/><path d="M158.62,35.59a10.88,10.88,0,0,1-4.23-1,1,1,0,0,1,.83-1.72s3.36,1.59,5.67.21c1.5-.89,2.34-2.88,2.5-5.91C163.91,17,150.17,7.73,150,7.63a1,1,0,0,1-.27-1.32.94.94,0,0,1,1.32-.26c.6.4,14.78,10,14.2,21.26-.18,3.71-1.34,6.22-3.42,7.46a6.38,6.38,0,0,1-3.24.82" style="fill:#d7a520"/><path d="M180.75,25.2a1,1,0,0,1-.58-1.71c.08-.06,8-6.24,6.7-12.4-.16-.77-.53-1.71-1.34-1.87-1.07-.21-2.65.8-3.67,2.34-.47.7-1.86,3.15-.32,5.48a1,1,0,0,1-.26,1.33A1,1,0,0,1,180,18.1a6.73,6.73,0,0,1,.31-7.6c1.5-2.25,3.76-3.51,5.62-3.15.83.16,2.31.81,2.84,3.35,1.53,7.33-7,14-7.4,14.3a.91.91,0,0,1-.58.2" style="fill:#d7a520"/><path d="M120.55,25.2A.88.88,0,0,1,120,25c-.37-.28-8.93-7-7.4-14.3.53-2.54,2-3.19,2.84-3.35,1.86-.37,4.11.9,5.61,3.15a6.75,6.75,0,0,1,.32,7.6,1,1,0,0,1-1.32.27,1,1,0,0,1-.27-1.33c1.54-2.33.15-4.78-.31-5.48-1-1.54-2.6-2.55-3.67-2.34-.82.16-1.18,1.1-1.34,1.87-1.29,6.16,6.61,12.34,6.69,12.4a1,1,0,0,1,.18,1.34,1,1,0,0,1-.76.37" style="fill:#d7a520"/></svg>

```

## File: static\src\img\classic-ornament-purple.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 307.99 46.65"><path d="M306.54,42.84H1.44a1,1,0,0,1,0-1.91h305.1a1,1,0,0,1,0,1.91" style="fill:#875A7B"/><path d="M222,46.65H86a1,1,0,1,1,0-1.9H222a1,1,0,1,1,0,1.9" style="fill:#875A7B"/><path d="M120.49,29.11l.33.34c8.23,8.52,13.68,14.15,23.71,10.2,3.75-1.48,6-4.24,6.44-7.76a11.31,11.31,0,0,0-5-10.35c-2.49-1.72-7.86-4-15.91.19a35.54,35.54,0,0,0-9.61,7.38m17.84,13.76c-7.19,0-12.33-5.32-18.88-12.1l-.26-.27L118,31.77c-4.9,5.43-9.52,10.56-22.12,10.56-6.68,0-12-3.07-14.64-8.44-2.33-4.76-2-10.45.86-14.14,2.55-3.31,6.65-4.58,11.56-3.58,3.58.74,6,2.76,6.92,5.69a10,10,0,0,1-2.5,9.45,6.19,6.19,0,0,1-6.24,1.83A1,1,0,0,1,91.24,32a.94.94,0,0,1,1.18-.64A4.37,4.37,0,0,0,96.77,30a8,8,0,0,0,2-7.55c-.68-2.26-2.58-3.77-5.48-4.37-4.16-.85-7.59.17-9.67,2.87-2.42,3.14-2.68,8-.66,12.14,2.3,4.69,7,7.38,12.93,7.38,11.75,0,15.9-4.61,20.71-9.94l1.23-1.36c-5.78-5.95-13-12.93-23.72-18.27C81,4.31,72,6.53,66.68,9.54c-6.91,3.94-10.43,10.88-10.79,15.65-.4,5.36,1,10.12,3.66,12.72,2.09,2,5,2.84,8.54,2.43,6-.67,6.75-4.58,6.81-6.24.11-2.65-1.35-5.28-3.11-5.63a4.3,4.3,0,0,0-5.12,2.44.95.95,0,0,1-1.77-.69,6.22,6.22,0,0,1,7.25-3.62c2.71.53,4.8,3.93,4.65,7.58-.17,4.46-3.35,7.48-8.49,8.06-4.18.47-7.56-.52-10.08-3-3.14-3-4.68-8.23-4.24-14.24.4-5.26,4.24-12.88,11.75-17.17C71.42,4.64,81.1,2.23,95,9.15c11,5.47,18.3,12.55,24.18,18.59A37.42,37.42,0,0,1,129.23,20c6.52-3.39,13-3.41,17.86-.07a13.24,13.24,0,0,1,5.77,12.14c-.48,4.19-3.26,7.59-7.63,9.31a18.7,18.7,0,0,1-6.9,1.45" style="fill:#875A7B"/><path d="M139.75,37.75c-6.95,0-15.23-8-29-21.43C93.54-.38,77,2,76.86,2.07a1,1,0,0,1-1.09-.79,1,1,0,0,1,.79-1.1C77.27.07,94.18-2.41,112.06,15c19.78,19.21,26.38,25,33.6,18.13a.95.95,0,0,1,1.3,1.38c-2.36,2.24-4.7,3.29-7.21,3.29" style="fill:#875A7B"/><path d="M39.55,42a16.61,16.61,0,0,1-4.68-.7c-5.22-1.54-6.6-5-6.95-6.91a9,9,0,0,1,3.32-8.58,8.22,8.22,0,0,1,8.83-.64,1,1,0,0,1-.83,1.72,6.28,6.28,0,0,0-6.84.43,7,7,0,0,0-2.61,6.72c.47,2.58,2.47,4.5,5.61,5.43a13.21,13.21,0,0,0,12-2,16.1,16.1,0,0,0,6.53-11.93c0-1.88-.31-4.21-2.13-4.7A3.05,3.05,0,0,0,49,21.5a2,2,0,0,0,.5,2.72,1,1,0,0,1-1.07,1.58,3.93,3.93,0,0,1-1.13-5.15c.59-1.19,2.84-2.18,4.86-1.68,1.39.34,3.72,1.66,3.63,6.6s-3,10.28-7.3,13.44a15,15,0,0,1-9,3" style="fill:#875A7B"/><path d="M162.31,19.36a12.55,12.55,0,0,0-7.21,2.18,11.33,11.33,0,0,0-5,10.35c.41,3.52,2.7,6.28,6.45,7.76,10,4,15.47-1.68,23.7-10.2l.33-.34a35.48,35.48,0,0,0-9.6-7.38,18.93,18.93,0,0,0-8.71-2.37m.47,23.51a18.69,18.69,0,0,1-6.89-1.45c-4.37-1.72-7.15-5.12-7.64-9.31A13.26,13.26,0,0,1,154,20c4.84-3.34,11.35-3.32,17.87.07A37.42,37.42,0,0,1,182,27.74c5.88-6,13.21-13.12,24.18-18.59C220,2.23,229.7,4.64,235.38,7.88c7.51,4.29,11.35,11.91,11.74,17.17.45,6-1.1,11.2-4.24,14.24-2.51,2.43-5.9,3.42-10.07,3-5.14-.58-8.32-3.6-8.5-8.06-.14-3.65,1.95-7,4.66-7.58a6.22,6.22,0,0,1,7.24,3.62.95.95,0,1,1-1.77.69,4.28,4.28,0,0,0-5.11-2.44c-1.77.35-3.22,3-3.12,5.63.07,1.66.86,5.57,6.81,6.24,3.58.4,6.46-.41,8.54-2.43,2.7-2.6,4.06-7.35,3.66-12.72-.35-4.77-3.88-11.71-10.78-15.65-5.28-3-14.33-5.23-27.46,1.32C196.26,16.2,189,23.18,183.25,29.13l1.24,1.36c4.8,5.33,8.95,9.94,20.71,9.94,5.92,0,10.63-2.69,12.92-7.38,2-4.13,1.76-9-.65-12.14-2.08-2.7-5.52-3.72-9.67-2.87-2.91.6-4.8,2.11-5.48,4.37a8.05,8.05,0,0,0,2,7.55,4.39,4.39,0,0,0,4.35,1.35.94.94,0,0,1,1.18.64,1,1,0,0,1-.64,1.19A6.16,6.16,0,0,1,203,31.31a9.91,9.91,0,0,1-2.5-9.45c.88-2.93,3.34-4.95,6.92-5.69,4.9-1,9,.27,11.55,3.58,2.85,3.69,3.19,9.38.86,14.14-2.63,5.37-8,8.44-14.63,8.44-12.6,0-17.23-5.13-22.12-10.56-.38-.42-.77-.84-1.15-1.27l-.27.27c-6.54,6.78-11.68,12.1-18.88,12.1" style="fill:#875A7B"/><path d="M161.37,37.75c-2.52,0-4.86-1-7.22-3.29a1,1,0,0,1,1.31-1.38c7.21,6.83,13.82,1.08,33.6-18.13C206.93-2.41,223.84.07,224.55.18a1,1,0,0,1,.79,1.1,1,1,0,0,1-1.08.79c-.17,0-16.73-2.41-33.88,14.25-13.79,13.39-22.07,21.43-29,21.43" style="fill:#875A7B"/><path d="M261.56,42a15,15,0,0,1-9-3c-4.29-3.16-7.22-8.56-7.31-13.44s2.24-6.26,3.64-6.6c2-.5,4.26.49,4.85,1.68a3.91,3.91,0,0,1-1.13,5.15,1,1,0,1,1-1.06-1.58,2,2,0,0,0,.49-2.72,3,3,0,0,0-2.75-.66c-1.83.49-2.17,2.82-2.14,4.7a16.14,16.14,0,0,0,6.53,11.93,13.22,13.22,0,0,0,12,2c3.15-.93,5.14-2.85,5.62-5.42a7.08,7.08,0,0,0-2.61-6.73,6.28,6.28,0,0,0-6.84-.43,1,1,0,0,1-1.27-.44,1,1,0,0,1,.44-1.28,8.2,8.2,0,0,1,8.82.64,9,9,0,0,1,3.33,8.58c-.36,1.92-1.73,5.37-6.95,6.91a16.68,16.68,0,0,1-4.69.7" style="fill:#875A7B"/><path d="M142.43,35.51a6.52,6.52,0,0,1-3.34-.85c-2.08-1.24-3.23-3.71-3.41-7.35-.58-11.3,13.6-20.86,14.2-21.26a.94.94,0,0,1,1.32.26,1,1,0,0,1-.26,1.32c-.14.1-13.88,9.37-13.36,19.58.15,3,1,4.92,2.48,5.81,2.41,1.43,6-.24,6-.26a1,1,0,0,1,.83,1.72,11.44,11.44,0,0,1-4.49,1" style="fill:#875A7B"/><path d="M158.62,35.59a10.88,10.88,0,0,1-4.23-1,1,1,0,0,1,.83-1.72s3.36,1.59,5.67.21c1.5-.89,2.34-2.88,2.5-5.91C163.91,17,150.17,7.73,150,7.63a1,1,0,0,1-.27-1.32.94.94,0,0,1,1.32-.26c.6.4,14.78,10,14.2,21.26-.18,3.71-1.34,6.22-3.42,7.46a6.38,6.38,0,0,1-3.24.82" style="fill:#875A7B"/><path d="M180.75,25.2a1,1,0,0,1-.58-1.71c.08-.06,8-6.24,6.7-12.4-.16-.77-.53-1.71-1.34-1.87-1.07-.21-2.65.8-3.67,2.34-.47.7-1.86,3.15-.32,5.48a1,1,0,0,1-.26,1.33A1,1,0,0,1,180,18.1a6.73,6.73,0,0,1,.31-7.6c1.5-2.25,3.76-3.51,5.62-3.15.83.16,2.31.81,2.84,3.35,1.53,7.33-7,14-7.4,14.3a.91.91,0,0,1-.58.2" style="fill:#875A7B"/><path d="M120.55,25.2A.88.88,0,0,1,120,25c-.37-.28-8.93-7-7.4-14.3.53-2.54,2-3.19,2.84-3.35,1.86-.37,4.11.9,5.61,3.15a6.75,6.75,0,0,1,.32,7.6,1,1,0,0,1-1.32.27,1,1,0,0,1-.27-1.33c1.54-2.33.15-4.78-.31-5.48-1-1.54-2.6-2.55-3.67-2.34-.82.16-1.18,1.1-1.34,1.87-1.29,6.16,6.61,12.34,6.69,12.4a1,1,0,0,1,.18,1.34,1,1,0,0,1-.76.37" style="fill:#875A7B"/></svg>

```

## File: static\src\img\modern-seal-blue.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 62.01 117.54">
    <defs>
        <clipPath id="a" transform="translate(0 0)">
            <rect width="62.01" height="117.54" style="fill:none" />
        </clipPath>
    </defs>
    <g style="clip-path:url(#a)">
        <path d="M31,62A31,31,0,1,0,0,31,31,31,0,0,0,31,62" transform="translate(0 0)" style="fill:#263e86" />
        <path d="M14,59.92v57.62l17-13.9,17,13.9V59.92a31,31,0,0,1-34,0" transform="translate(0 0)"
        style="fill:#263e86" />
        <circle cx="31.01" cy="31.01" r="23.06" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
        <circle cx="31.01" cy="31.01" r="22.23" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
    </g>
</svg>

```

## File: static\src\img\modern-seal-gold.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 62.01 117.54">
    <defs>
        <clipPath id="a" transform="translate(0 0)">
            <rect width="62.01" height="117.54" style="fill:none" />
        </clipPath>
    </defs>
    <g style="clip-path:url(#a)">
        <path d="M31,62A31,31,0,1,0,0,31,31,31,0,0,0,31,62" transform="translate(0 0)" style="fill:#d7a520" />
        <path d="M14,59.92v57.62l17-13.9,17,13.9V59.92a31,31,0,0,1-34,0" transform="translate(0 0)"
        style="fill:#d7a520" />
        <circle cx="31.01" cy="31.01" r="23.06" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
        <circle cx="31.01" cy="31.01" r="22.23" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
    </g>
</svg>

```

## File: static\src\img\modern-seal-purple.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 62.01 117.54">
    <defs>
        <clipPath id="a" transform="translate(0 0)">
            <rect width="62.01" height="117.54" style="fill:none" />
        </clipPath>
    </defs>
    <g style="clip-path:url(#a)">
        <path d="M31,62A31,31,0,1,0,0,31,31,31,0,0,0,31,62" transform="translate(0 0)" style="fill:#875A7B" />
        <path d="M14,59.92v57.62l17-13.9,17,13.9V59.92a31,31,0,0,1-34,0" transform="translate(0 0)"
        style="fill:#875A7B" />
        <circle cx="31.01" cy="31.01" r="23.06" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
        <circle cx="31.01" cy="31.01" r="22.23" style="fill:none;stroke:#f1f2f2;stroke-width:0.16699999570846558px" />
    </g>
</svg>

```

## File: static\src\img\trophy-solid.svg

```svg
<svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="trophy" class="svg-inline--fa fa-trophy fa-w-18" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512"><path fill="currentColor" d="M552 64H448V24c0-13.3-10.7-24-24-24H152c-13.3 0-24 10.7-24 24v40H24C10.7 64 0 74.7 0 88v56c0 35.7 22.5 72.4 61.9 100.7 31.5 22.7 69.8 37.1 110 41.7C203.3 338.5 240 360 240 360v72h-48c-35.3 0-64 20.7-64 56v12c0 6.6 5.4 12 12 12h296c6.6 0 12-5.4 12-12v-12c0-35.3-28.7-56-64-56h-48v-72s36.7-21.5 68.1-73.6c40.3-4.6 78.6-19 110-41.7 39.3-28.3 61.9-65 61.9-100.7V88c0-13.3-10.7-24-24-24zM99.3 192.8C74.9 175.2 64 155.6 64 144v-16h64.2c1 32.6 5.8 61.2 12.8 86.2-15.1-5.2-29.2-12.4-41.7-21.4zM512 144c0 16.1-17.7 36.1-35.3 48.8-12.5 9-26.7 16.2-41.8 21.4 7-25 11.8-53.6 12.8-86.2H512v16z"></path></svg>
```

## File: static\src\js\fields_form_page_description.js

```javascript
odoo.define('survey.fields_form', function (require) {
"use strict";

var FieldRegistry = require('web.field_registry');
var FieldChar = require('web.basic_fields').FieldChar;

var FormDescriptionPage = FieldChar.extend({

    //--------------------------------------------------------------------------
    // Widget API
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _renderEdit: function () {
        var def = this._super.apply(this, arguments);
        this.$el.addClass('col');
        var $inputGroup = $('<div class="input-group">');
        this.$el = $inputGroup.append(this.$el);
        var $button = $(
            '<div class="input-group-append">\
                <button type="button" title="Open section" class="btn oe_edit_only o_icon_button">\
                    <i class="fa fa-fw o_button_icon fa-info-circle"/>\
                </button>\
            </div>'
        );
        this.$el = this.$el.append($button);
        $button.on('click', this._onClickEdit.bind(this));

        return def;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickEdit: function (ev) {
        ev.stopPropagation();
        var id = this.record.id;
        if (id) {
            this.trigger_up('open_record', {id: id, target: ev.target});
        }
    },
});

FieldRegistry.add('survey_description_page', FormDescriptionPage);

});

```

## File: static\src\js\fields_section_one2many.js

```javascript
odoo.define('survey.question_page_one2many', function (require) {
"use strict";

var Context = require('web.Context');
var FieldOne2Many = require('web.relational_fields').FieldOne2Many;
var FieldRegistry = require('web.field_registry');
var ListRenderer = require('web.ListRenderer');
var config = require('web.config');

var SectionListRenderer = ListRenderer.extend({
    init: function (parent, state, params) {
        this.sectionFieldName = "is_page";
        this._super.apply(this, arguments);
    },
    _checkIfRecordIsSection: function (id) {
        var record = this._findRecordById(id);
        return record && record.data[this.sectionFieldName];
    },
    _findRecordById: function (id) {
        return _.find(this.state.data, function (record) {
            return record.id === id;
        });
    },
    /**
     * Allows to hide specific field in case the record is a section
     * and, in this case, makes the 'title' field take the space of all the other
     * fields
     * @private
     * @override
     * @param {*} record
     * @param {*} node
     * @param {*} index
     * @param {*} options
     */
    _renderBodyCell: function (record, node, index, options) {
        var $cell = this._super.apply(this, arguments);

        var isSection = record.data[this.sectionFieldName];

        if (isSection) {
            if (node.attrs.widget === "handle" || node.attrs.name === "random_questions_count") {
                return $cell;
            } else if (node.attrs.name === "title") {
                var nbrColumns = this._getNumberOfCols();
                if (this.handleField) {
                    nbrColumns--;
                }
                if (this.addTrashIcon) {
                    nbrColumns--;
                }
                if (record.data.questions_selection === "random") {
                    nbrColumns--;
                }
                // Render empty cells for buttons to avoid having unaligned elements
                nbrColumns -= this.columns.filter(elem => elem.tag === "button_group").length;
                $cell.attr('colspan', nbrColumns);
            } else if (node.tag === "button_group") {
                $cell.addClass('o_invisible_modifier');
            } else {
                $cell.removeClass('o_invisible_modifier');
                return $cell.addClass('o_hidden');
            }
        }
        return $cell;
    },
    /**
     * Adds specific classes to rows that are sections
     * to apply custom css on them
     * @private
     * @override
     * @param {*} record
     * @param {*} index
     */
    _renderRow: function (record, index) {
        var $row = this._super.apply(this, arguments);
        if (record.data[this.sectionFieldName]) {
            $row.addClass("o_is_section");
        }
        return $row;
    },
    /**
     * Adding this class after the view is rendered allows
     * us to limit the custom css scope to this particular case
     * and no other
     * @private
     * @override
     */
    _renderView: function () {
        var def = this._super.apply(this, arguments);
        var self = this;
        return def.then(function () {
            self.$('table.o_list_table').addClass('o_section_list_view');
        });
    },
    // Handlers
    /**
     * Overridden to allow different behaviours depending on
     * the row the user clicked on.
     * If the row is a section: edit inline
     * else use a normal modal
     * @private
     * @override
     * @param {*} ev
     */
    _onRowClicked: function (ev) {
        var parent = this.getParent();
        var recordId = $(ev.currentTarget).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit") {
            this.editable = "bottom";
        } else {
            this.editable = null;
        }
        this._super.apply(this, arguments);
    },
    /**
     * Overridden to allow different behaviours depending on
     * the cell the user clicked on.
     * If the cell is part of a section: edit inline
     * else use a normal edit modal
     * @private
     * @override
     * @param {*} ev
     */
    _onCellClick: function (ev) {
        var parent = this.getParent();
        var recordId = $(ev.currentTarget.parentElement).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit") {
            this.editable = "bottom";
        } else {
            this.editable = null;
            this.unselectRow();
        }
        this._super.apply(this, arguments);
    },
    /**
     * In this case, navigating in the list caused issues.
     * For example, editing a section then pressing enter would trigger
     * the inline edition of the next element in the list. Which is not desired
     * if the next element ends up being a question and not a section
     * @override
     * @param {*} ev
     */
    _onNavigationMove: function (ev) {
        this.unselectRow();
    },
});

var SectionFieldOne2Many = FieldOne2Many.extend({
    init: function (parent, name, record, options) {
        this._super.apply(this, arguments);
        this.sectionFieldName = "is_page";
        this.rendered = false;
    },
    /**
     * Overridden to use our custom renderer
     * @private
     * @override
     */
    _getRenderer: function () {
        if (this.view.arch.tag === 'tree') {
            return SectionListRenderer;
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Overridden to allow different behaviours depending on
     * the object we want to add. Adding a section would be done inline
     * while adding a question would render a modal.
     * @private
     * @override
     * @param {*} ev
     */
    _onAddRecord: function (ev) {
        this.editable = null;
        if (!config.device.isMobile) {
            var context_str = ev.data.context && ev.data.context[0];
            var context = new Context(context_str).eval();
            if (context['default_' + this.sectionFieldName]) {
                this.editable = "bottom";
            }
        }
        this._super.apply(this, arguments);
    },
});

FieldRegistry.add('question_page_one2many', SectionFieldOne2Many);
});

```

## File: static\src\js\survey_breadcrumb.js

```javascript
odoo.define('survey.breadcrumb', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.SurveyBreadcrumbWidget = publicWidget.Widget.extend({
    xmlDependencies: ['/survey/static/src/xml/survey_breadcrumb_templates.xml'],
    template: "survey.survey_breadcrumb_template",
    events: {
        'click .breadcrumb-item a': '_onBreadcrumbClick',
    },

    /**
     * @override
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.canGoBack = options.canGoBack;
        this.currentPageId = options.currentPageId;
        this.pages = options.pages;
    },

    // Handlers
    // -------------------------------------------------------------------

    _onBreadcrumbClick: function (event) {
        event.preventDefault();
        this.trigger_up('breadcrumb_click', {
            'previousPageId': this.$(event.currentTarget)
                .closest('.breadcrumb-item')
                .data('pageId')
        });
    },

    // PUBLIC METHODS
    // -------------------------------------------------------------------

    updateBreadcrumb: function (pageId) {
        if (pageId) {
            this.currentPageId = pageId;
            this.renderElement();
        } else {
            this.$('.breadcrumb').addClass('d-none');
        }
    },
});

return publicWidget.registry.SurveyBreadcrumbWidget;

});

```

## File: static\src\js\survey_form.js

```javascript
odoo.define('survey.form', function (require) {
'use strict';

var field_utils = require('web.field_utils');
var publicWidget = require('web.public.widget');
var time = require('web.time');
var core = require('web.core');
var Dialog = require('web.Dialog');
var dom = require('web.dom');
var utils = require('web.utils');

var _t = core._t;

publicWidget.registry.SurveyFormWidget = publicWidget.Widget.extend({
    selector: '.o_survey_form',
    events: {
        'change .o_survey_form_choice_item': '_onChangeChoiceItem',
        'click .o_survey_matrix_btn': '_onMatrixBtnClick',
        'click button[type="submit"]': '_onSubmit',
    },
    custom_events: {
        'breadcrumb_click': '_onBreadcrumbClick',
    },

    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
    * @override
    */
    start: function () {
        var self = this;
        this.fadeInOutDelay = 400;
        return this._super.apply(this, arguments).then(function () {
            self.options = self.$target.find('form').data();
            self.readonly = self.options.readonly;
            self.selectedAnswers = self.options.selectedAnswers;

            // Add Survey cookie to retrieve the survey if you quit the page and restart the survey.
            if (!utils.get_cookie('survey_' + self.options.surveyToken)) {
                utils.set_cookie('survey_' + self.options.surveyToken, self.options.answerToken, 60*60*24);
            }

            // Init fields
            if (!self.options.isStartScreen && !self.readonly) {
                self._initTimer();
                self._initBreadcrumb();
            }
            self.$('div.o_survey_form_date').each(function () {
                self._initDateTimePicker($(this));
            });
            self._initChoiceItems();
            self._initTextArea();
            self._focusOnFirstInput();
            // Init event listener
            if (!self.readonly) {
                $(document).on('keydown', self._onKeyDown.bind(self));
            }
            if (self.options.sessionInProgress &&
                (self.options.isStartScreen || self.options.hasAnswered || self.options.isPageDescription)) {
                self.preventEnterSubmit = true;
            }
            self._initSessionManagement();

            // Needs global selector as progress/navigation are not within the survey form, but need
            //to be updated at the same time
            self.$surveyProgress = $('.o_survey_progress_wrapper');
            self.$surveyNavigation = $('.o_survey_navigation_wrapper');
            self.$surveyNavigation.find('.o_survey_navigation_submit').on('click', self._onSubmit.bind(self));
        });
    },

    // -------------------------------------------------------------------------
    // Private
    // -------------------------------------------------------------------------

    // Handlers
    // -------------------------------------------------------------------------

    /**
     * Handle keyboard navigation:
     * - 'enter' or 'arrow-right' => submit form
     * - 'arrow-left' => submit form (but go back backwards)
     * - other alphabetical character ('a', 'b', ...)
     *   Select the related option in the form (if available)
     *
     * @param {Event} event
     */
    _onKeyDown: function (event) {
        var self = this;
        var keyCode = event.keyCode;

        // If user is answering a text input, do not handle keydown
        // CTRL+enter will force submission
        if ((this.$("textarea").is(":focus") || this.$('input').is(':focus')) &&
            (!event.ctrlKey || keyCode !== 13)) {
            return;
        }
        // If in session mode and question already answered, do not handle keydown
        if (this.$('fieldset[disabled="disabled"]').length !== 0) {
            return;
        }

        var letter = String.fromCharCode(keyCode).toUpperCase();

        // Handle Start / Next / Submit
        if (keyCode === 13 || keyCode === 39) {  // Enter or arrow-right: go Next
            event.preventDefault();
            if (!this.preventEnterSubmit) {
                var isFinish = this.$('button[value="finish"]').length !== 0;
                this._submitForm({isFinish: isFinish});
            }
        } else if (keyCode === 37) {  // arrow-left: previous (if available)
            // It's easier to actually click on the button (if in the DOM) as it contains necessary
            // data that are used in the event handler.
            // Again, global selector necessary since the navigation is outside of the form.
            $('.o_survey_navigation_submit[value="previous"]').click();
        } else if (self.options.questionsLayout === 'page_per_question'
                   && letter.match(/[a-z]/i)) {
            var $choiceInput = this.$(`input[data-selection-key=${letter}]`);
            if ($choiceInput.length === 1) {
                $choiceInput.prop("checked", !$choiceInput.prop("checked")).trigger('change');

                // Avoid selection key to be typed into the textbox if 'other' is selected by key
                event.preventDefault();
            }
        }
    },

    /**
    * Checks, if the 'other' choice is checked. Applies only if the comment count as answer.
    *   If not checked : Clear the comment textarea, hide and disable it
    *   If checked : enable the comment textarea, show and focus on it
    *
    * @private
    * @param {Event} event
    */
    _onChangeChoiceItem: function (event) {
        var self = this;
        var $target = $(event.currentTarget);
        var $choiceItemGroup = $target.closest('.o_survey_form_choice');
        var $otherItem = $choiceItemGroup.find('.o_survey_js_form_other_comment');
        var $commentInput = $choiceItemGroup.find('textarea[type="text"]');

        if ($otherItem.prop('checked') || $commentInput.hasClass('o_survey_comment')) {
            $commentInput.enable();
            $commentInput.closest('.o_survey_comment_container').removeClass('d-none');
            if ($otherItem.prop('checked')) {
                $commentInput.focus();
            }
        } else {
            $commentInput.val('');
            $commentInput.closest('.o_survey_comment_container').addClass('d-none');
            $commentInput.enable(false);
        }

        var $matrixBtn = $target.closest('.o_survey_matrix_btn');
        if ($target.attr('type') === 'radio') {
            var isQuestionComplete = false;
            if ($matrixBtn.length > 0) {
                $matrixBtn.closest('tr').find('td').removeClass('o_survey_selected');
                $matrixBtn.addClass('o_survey_selected');
                if (this.options.questionsLayout === 'page_per_question') {
                    var subQuestionsIds = $matrixBtn.closest('table').data('subQuestions');
                    var completedQuestions = [];
                    subQuestionsIds.forEach(function (id) {
                        if (self.$('tr#' + id).find('input:checked').length !== 0) {
                            completedQuestions.push(id);
                        }
                    });
                    isQuestionComplete = completedQuestions.length === subQuestionsIds.length;
                }
            } else {
                var previouslySelectedAnswer = $choiceItemGroup.find('label.o_survey_selected');
                previouslySelectedAnswer.removeClass('o_survey_selected');

                var newlySelectedAnswer = $target.closest('label');
                if (newlySelectedAnswer.find('input').val() !== previouslySelectedAnswer.find('input').val()) {
                    newlySelectedAnswer.addClass('o_survey_selected');
                    isQuestionComplete = this.options.questionsLayout === 'page_per_question';
                }

                // Conditional display
                if (this.options.questionsLayout !== 'page_per_question') {
                    var treatedQuestionIds = [];  // Needed to avoid show (1st 'if') then immediately hide (2nd 'if') question during conditional propagation cascade
                    if (Object.keys(this.options.triggeredQuestionsByAnswer).includes(previouslySelectedAnswer.find('input').val())) {
                        // Hide and clear depending question
                        this.options.triggeredQuestionsByAnswer[previouslySelectedAnswer.find('input').val()].forEach(function (questionId) {
                            var dependingQuestion = $('.js_question-wrapper#' + questionId);

                            dependingQuestion.addClass('d-none');
                            self._clearQuestionInputs(dependingQuestion);

                            treatedQuestionIds.push(questionId);
                        });
                        // Remove answer from selected answer
                        self.selectedAnswers.splice(self.selectedAnswers.indexOf(parseInt($target.val())), 1);
                    }
                    if (Object.keys(this.options.triggeredQuestionsByAnswer).includes($target.val())) {
                        // Display depending question
                        const selectedAnswerId = parseInt($target.val());
                        this.options.triggeredQuestionsByAnswer[$target.val()].forEach(function (questionId) {
                            if (!treatedQuestionIds.includes(questionId)) {
                                var dependingQuestion = $('.js_question-wrapper#' + questionId);
                                dependingQuestion.removeClass('d-none');
                                if (!self.selectedAnswers.includes(selectedAnswerId)) {
                                    // Add answer to selected answer
                                    self.selectedAnswers.push(selectedAnswerId);
                                }
                            }
                        });
                    }
                }
            }
            // Auto Submit Form
            var isLastQuestion = this.$('button[value="finish"]').length !== 0;
            var questionHasComment = $target.closest('.o_survey_form_choice').find('.o_survey_comment').length !== 0
                                        || $target.hasClass('o_survey_js_form_other_comment');
            if (!isLastQuestion && this.options.usersCanGoBack && isQuestionComplete && !questionHasComment) {
                this._submitForm({});
            }
        } else {  // $target.attr('type') === 'checkbox'
            if ($matrixBtn.length > 0) {
                $matrixBtn.toggleClass('o_survey_selected', !$matrixBtn.hasClass('o_survey_selected'));
            } else {
                var $label = $target.closest('label');
                $label.toggleClass('o_survey_selected', !$label.hasClass('o_survey_selected'));

                // Conditional display
                if (this.options.questionsLayout !== 'page_per_question' && Object.keys(this.options.triggeredQuestionsByAnswer).includes($target.val())) {
                    var isInputSelected = $label.hasClass('o_survey_selected');
                    // Hide and clear or display depending question
                    this.options.triggeredQuestionsByAnswer[$target.val()].forEach(function (questionId) {
                        var dependingQuestion = $('.js_question-wrapper#' + questionId);
                        dependingQuestion.toggleClass('d-none', !isInputSelected);
                        if (!isInputSelected) {
                            self._clearQuestionInputs(dependingQuestion);
                        }
                    });
                    // Add/remove answer to/from selected answer
                    if (!isInputSelected) {
                        self.selectedAnswers.splice(self.selectedAnswers.indexOf(parseInt($target.val())), 1);
                    } else {
                        self.selectedAnswers.push(parseInt($target.val()));
                    }
                }
            }
        }
    },

    _onMatrixBtnClick: function (event) {
        if (this.readonly) {
            return;
        }

        var $target = $(event.currentTarget);
        var $input = $target.find('input');
        if ($input.attr('type') === 'radio') {
            $input.prop("checked", true).trigger('change');
        } else {
            $input.prop("checked", !$input.prop("checked")).trigger('change');
        }
    },

    _onSubmit: function (event) {
        event.preventDefault();
        var options = {};
        var $target = $(event.currentTarget);
        if ($target.val() === 'previous') {
            options.previousPageId = $target.data('previousPageId');
        } else if ($target.val() === 'finish') {
            options.isFinish = true;
        }
        this._submitForm(options);
    },

    // Custom Events
    // -------------------------------------------------------------------------

    _onBreadcrumbClick: function (event) {
        this._submitForm({'previousPageId': event.data.previousPageId});
    },

    /**
     * We listen to 'next_question' and 'end_session' events to load the next
     * page of the survey automatically, based on the host pacing.
     *
     * If the trigger is 'next_question', we handle some extra computation to find
     * a suitable "fadeInOutDelay" based on the delay between the time of the question
     * change by the host and the time of reception of the event.
     * This will allow us to account for a little bit of server lag (up to 1 second)
     * while giving everyone a fair experience on the quiz.
     *
     * e.g 1:
     * - The host switches the question
     * - We receive the event 200 ms later due to server lag
     * - -> The fadeInOutDelay will be 400 ms (200ms delay + 400ms * 2 fade in fade out)
     *
     * e.g 2:
     * - The host switches the question
     * - We receive the event 600 ms later due to bigger server lag
     * - -> The fadeInOutDelay will be 200ms (600ms delay + 200ms * 2 fade in fade out)
     *
     * @private
     * @param {Array[]} notifications structured as specified by the bus feature
     */
    _onNotification: function (notifications) {
        var nextPageEvent = false;
        if (notifications && notifications.length !== 0) {
            notifications.forEach(function (notification) {
                if (notification.type === 'next_question' ||
                    notification.type === 'end_session') {
                    nextPageEvent = notification;
                }
            });
        }

        if (this.options.isStartScreen && nextPageEvent.type === 'end_session') {
            // can happen when triggering the same survey session multiple times
            // we received an "old" end_session event that needs to be ignored
            return;
        }

        if (nextPageEvent) {
            if (nextPageEvent.type === 'next_question') {
                var serverDelayMS = moment.utc().valueOf() - moment.unix(nextPageEvent.payload.question_start).utc().valueOf();
                if (serverDelayMS < 0) {
                    serverDelayMS = 0;
                } else if (serverDelayMS > 1000) {
                    serverDelayMS = 1000;
                }
                this.fadeInOutDelay = (1000 - serverDelayMS) / 2;
            } else {
                this.fadeInOutDelay = 400;
            }

            this.$('.o_survey_main_title:visible').fadeOut(400);

            this.preventEnterSubmit = false;
            this.readonly = false;
            this._nextScreen(
                this._rpc({
                    route: `/survey/next_question/${this.options.surveyToken}/${this.options.answerToken}`,
                }), {
                    initTimer: true,
                    isFinish: nextPageEvent.type === 'end_session'
                }
            );
        }
    },

    // SUBMIT
    // -------------------------------------------------------------------------

    /**
    * This function will send a json rpc call to the server to
    * - start the survey (if we are on start screen)
    * - submit the answers of the current page
    * Before submitting the answers, they are first validated to avoid latency from the server
    * and allow a fade out/fade in transition of the next question.
    *
    * @param {Array} [options]
    * @param {Integer} [options.previousPageId] navigates to page id
    * @param {Boolean} [options.skipValidation] skips JS validation
    * @param {Boolean} [options.initTime] will force the re-init of the timer after next
    *   screen transition
    * @param {Boolean} [options.isFinish] fades out breadcrumb and timer
    * @private
    */
    _submitForm: function (options) {
        var self = this;
        var params = {};
        if (options.previousPageId) {
            params.previous_page_id = options.previousPageId;
        }
        var route = "/survey/submit";

        if (this.options.isStartScreen) {
            route = "/survey/begin";
            // Hide survey title in 'page_per_question' layout: it takes too much space
            if (this.options.questionsLayout === 'page_per_question') {
                this.$('.o_survey_main_title').fadeOut(400);
            }
        } else {
            var $form = this.$('form');
            var formData = new FormData($form[0]);

            if (!options.skipValidation) {
                // Validation pre submit
                if (!this._validateForm($form, formData)) {
                    return;
                }
            }

            this._prepareSubmitValues(formData, params);
        }

        // prevent user from submitting more times using enter key
        this.preventEnterSubmit = true;

        if (this.options.sessionInProgress) {
            // reset the fadeInOutDelay when attendee is submitting form
            this.fadeInOutDelay = 400;
            // prevent user from clicking on matrix options when form is submitted
            this.readonly = true;
        }

        var submitPromise = self._rpc({
            route: _.str.sprintf('%s/%s/%s', route, self.options.surveyToken, self.options.answerToken),
            params: params,
        });
        this._nextScreen(submitPromise, options);
    },

    /**
     * Will fade out / fade in the next screen based on passed promise and options.
     *
     * @param {Promise} nextScreenPromise
     * @param {Object} options see '_submitForm' for details
     */
    _nextScreen: function (nextScreenPromise, options) {
        var self = this;

        var resolveFadeOut;
        var fadeOutPromise = new Promise(function (resolve, reject) {resolveFadeOut = resolve;});

        var selectorsToFadeout = ['.o_survey_form_content'];
        if (options.isFinish) {
            selectorsToFadeout.push('.breadcrumb', '.o_survey_timer');
            utils.set_cookie('survey_' + self.options.surveyToken, '', -1);  // delete cookie
        }
        self.$(selectorsToFadeout.join(',')).fadeOut(this.fadeInOutDelay, function () {
            resolveFadeOut();
        });

        Promise.all([fadeOutPromise, nextScreenPromise]).then(function (results) {
            return self._onNextScreenDone(results[1], options);
        });
    },

    /**
     * Handle server side validation and display eventual error messages.
     *
     * @param {string} result the HTML result of the screen to display
     * @param {Object} options see '_submitForm' for details
     */
   _onNextScreenDone: function (result, options) {
        var self = this;

        if (!(options && options.isFinish)
            && !this.options.sessionInProgress) {
            this.preventEnterSubmit = false;
        }

        if (result && !result.error) {
            this.$(".o_survey_form_content").empty();
            this.$(".o_survey_form_content").html(result.survey_content);

            if (result.survey_progress && this.$surveyProgress.length !== 0) {
                this.$surveyProgress.html(result.survey_progress);
            } else if (options.isFinish && this.$surveyProgress.length !== 0) {
                this.$surveyProgress.remove();
            }

            if (result.survey_navigation && this.$surveyNavigation.length !== 0) {
                this.$surveyNavigation.html(result.survey_navigation);
                this.$surveyNavigation.find('.o_survey_navigation_submit').on('click', self._onSubmit.bind(self));
            }

            // Hide timer if end screen (if page_per_question in case of conditional questions)
            if (self.options.questionsLayout === 'page_per_question' && this.$('.o_survey_finished').length > 0) {
                options.isFinish = true;
            }

            this.$('div.o_survey_form_date').each(function () {
                self._initDateTimePicker($(this));
            });
            if (this.options.isStartScreen || (options && options.initTimer)) {
                this._initTimer();
                this.options.isStartScreen = false;
            } else {
                if (this.options.sessionInProgress && this.surveyTimerWidget) {
                    this.surveyTimerWidget.destroy();
                }
            }
            if (options && options.isFinish) {
                this._initResultWidget();
                if (this.surveyBreadcrumbWidget) {
                    this.$('.o_survey_breadcrumb_container').addClass('d-none');
                    this.surveyBreadcrumbWidget.destroy();
                }
                if (this.surveyTimerWidget) {
                    this.surveyTimerWidget.destroy();
                }
            } else {
                this._updateBreadcrumb();
            }
            self._initChoiceItems();
            self._initTextArea();

            if (this.options.sessionInProgress && this.$('.o_survey_form_content_data').data('isPageDescription')) {
                // prevent enter submit if we're on a page description (there is nothing to submit)
                this.preventEnterSubmit = true;
            }

            this.$('.o_survey_form_content').fadeIn(this.fadeInOutDelay);
            $("html, body").animate({ scrollTop: 0 }, this.fadeInOutDelay);
            self._focusOnFirstInput();
        }
        else if (result && result.fields && result.error === 'validation') {
            this.$('.o_survey_form_content').fadeIn(0);
            this._showErrors(result.fields);
        } else {
            var $errorTarget = this.$('.o_survey_error');
            $errorTarget.removeClass("d-none");
            this._scrollToError($errorTarget);
        }
    },

    // VALIDATION TOOLS
    // -------------------------------------------------------------------------
    /**
    * Validation is done in frontend before submit to avoid latency from the server.
    * If the validation is incorrect, the errors are displayed before submitting and
    * fade in / out of submit is avoided.
    *
    * Each question type gets its own validation process.
    *
    * There is a special use case for the 'required' questions, where we use the constraint
    * error message that comes from the question configuration ('constr_error_msg' field).
    *
    * @private
    */
    _validateForm: function ($form, formData) {
        var self = this;
        var errors = {};
        var validationEmailMsg = _t("This answer must be an email address.");
        var validationDateMsg = _t("This is not a date");

        this._resetErrors();

        var data = {};
        formData.forEach(function (value, key) {
            data[key] = value;
        });

        var inactiveQuestionIds = this.options.sessionInProgress ? [] : this._getInactiveConditionalQuestionIds();

        $form.find('[data-question-type]').each(function () {
            var $input = $(this);
            var $questionWrapper = $input.closest(".js_question-wrapper");
            var questionId = $questionWrapper.attr('id');

            // If question is inactive, skip validation.
            if (inactiveQuestionIds.includes(parseInt(questionId))) {
                return;
            }

            var questionRequired = $questionWrapper.data('required');
            var constrErrorMsg = $questionWrapper.data('constrErrorMsg');
            var validationErrorMsg = $questionWrapper.data('validationErrorMsg');
            switch ($input.data('questionType')) {
                case 'char_box':
                    if (questionRequired && !$input.val()) {
                        errors[questionId] = constrErrorMsg;
                    } else if ($input.val() && $input.attr('type') === 'email' && !self._validateEmail($input.val())) {
                        errors[questionId] = validationEmailMsg;
                    } else {
                        var lengthMin = $input.data('validationLengthMin');
                        var lengthMax = $input.data('validationLengthMax');
                        var length = $input.val().length;
                        if (lengthMin && (lengthMin > length || length > lengthMax)) {
                            errors[questionId] = validationErrorMsg;
                        }
                    }
                    break;
                case 'text_box':
                    if (questionRequired && !$input.val()) {
                        errors[questionId] = constrErrorMsg;
                    }
                    break;
                case 'numerical_box':
                    if (questionRequired && !data[questionId]) {
                        errors[questionId] = constrErrorMsg;
                    } else {
                        var floatMin = $input.data('validationFloatMin');
                        var floatMax = $input.data('validationFloatMax');
                        var value = parseFloat($input.val());
                        if (floatMin && (floatMin > value || value > floatMax)) {
                            errors[questionId] = validationErrorMsg;
                        }
                    }
                    break;
                case 'date':
                case 'datetime':
                    if (questionRequired && !data[questionId]) {
                        errors[questionId] = constrErrorMsg;
                    } else if (data[questionId]) {
                        var datetimepickerFormat = $input.data('questionType') === 'datetime' ? time.getLangDatetimeFormat() : time.getLangDateFormat();
                        var momentDate = moment($input.val(), datetimepickerFormat);
                        if (!momentDate.isValid()) {
                            errors[questionId] = validationDateMsg;
                        } else {
                            var $dateDiv = $questionWrapper.find('.o_survey_form_date');
                            var maxDate = $dateDiv.data('maxdate');
                            var minDate = $dateDiv.data('mindate');
                            if ((maxDate && momentDate.isAfter(moment(maxDate)))
                                    || (minDate && momentDate.isBefore(moment(minDate)))) {
                                errors[questionId] = validationErrorMsg;
                            }
                        }
                    }
                    break;
                case 'simple_choice_radio':
                case 'multiple_choice':
                    if (questionRequired) {
                        var $textarea = $questionWrapper.find('textarea');
                        if (!data[questionId]) {
                            errors[questionId] = constrErrorMsg;
                        } else if (data[questionId] === '-1' && !$textarea.val()) {
                            // if other has been checked and value is null
                            errors[questionId] = constrErrorMsg;
                        }
                    }
                    break;
                case 'matrix':
                    if (questionRequired) {
                        var subQuestionsIds = $questionWrapper.find('table').data('subQuestions');
                        subQuestionsIds.forEach(function (id) {
                            if (!((questionId + '_' + id) in data)) {
                                errors[questionId] = constrErrorMsg;
                            }
                        });
                    }
                    break;
            }
        });
        if (_.keys(errors).length > 0) {
            this._showErrors(errors);
            return false;
        }
        return true;
    },

    /**
    * Check if the email has an '@', a left part and a right part
    * @private
    */
    _validateEmail: function (email) {
        var emailParts = email.split('@');
        return emailParts.length === 2 && emailParts[0] && emailParts[1];
    },

    // PREPARE SUBMIT TOOLS
    // -------------------------------------------------------------------------
    /**
    * For each type of question, extract the answer from inputs or textarea (comment or answer)
    *
    *
    * @private
    * @param {Event} event
    */
    _prepareSubmitValues: function (formData, params) {
        var self = this;
        formData.forEach(function (value, key) {
            switch (key) {
                case 'csrf_token':
                case 'token':
                case 'page_id':
                case 'question_id':
                    params[key] = value;
                    break;
            }
        });

        // Get all question answers by question type
        this.$('[data-question-type]').each(function () {
            switch ($(this).data('questionType')) {
                case 'text_box':
                case 'char_box':
                case 'numerical_box':
                    params[this.name] = this.value;
                    break;
                case 'date':
                    params = self._prepareSubmitDates(params, this.name, this.value, false);
                    break;
                case 'datetime':
                    params = self._prepareSubmitDates(params, this.name, this.value, true);
                    break;
                case 'simple_choice_radio':
                case 'multiple_choice':
                    params = self._prepareSubmitChoices(params, $(this), $(this).data('name'));
                    break;
                case 'matrix':
                    params = self._prepareSubmitAnswersMatrix(params, $(this));
                    break;
            }
        });
    },

    /**
    *   Prepare date answer before submitting form.
    *   Convert date value from client current timezone to UTC Date to correspond to the server format.
    *   return params = { 'dateQuestionId' : '2019-05-23', 'datetimeQuestionId' : '2019-05-23 14:05:12' }
    */
    _prepareSubmitDates: function (params, questionId, value, isDateTime) {
        var momentDate = isDateTime ? field_utils.parse.datetime(value, null, {timezone: true}) : field_utils.parse.date(value);
        var formattedDate = momentDate ? momentDate.toJSON() : '';
        params[questionId] = formattedDate;
        return params;
    },

    /**
    *   Prepare choice answer before submitting form.
    *   If the answer is not the 'comment selection' (=Other), calls the _prepareSubmitAnswer method to add the answer to the params
    *   If there is a comment linked to that question, calls the _prepareSubmitComment method to add the comment to the params
    */
    _prepareSubmitChoices: function (params, $parent, questionId) {
        var self = this;
        $parent.find('input:checked').each(function () {
            if (this.value !== '-1') {
                params = self._prepareSubmitAnswer(params, questionId, this.value);
            }
        });
        params = self._prepareSubmitComment(params, $parent, questionId, false);
        return params;
    },


    /**
    *   Prepare matrix answers before submitting form.
    *   This method adds matrix answers one by one and add comment if any to a params key,value like :
    *   params = { 'matrixQuestionId' : {'rowId1': [colId1, colId2,...], 'rowId2': [colId1, colId3, ...], 'comment': comment }}
    */
    _prepareSubmitAnswersMatrix: function (params, $matrixTable) {
        var self = this;
        $matrixTable.find('input:checked').each(function () {
            params = self._prepareSubmitAnswerMatrix(params, $matrixTable.data('name'), $(this).data('rowId'), this.value);
        });
        params = self._prepareSubmitComment(params, $matrixTable.closest('.js_question-wrapper'), $matrixTable.data('name'), true);
        return params;
    },

    /**
    *   Prepare answer before submitting form if question type is matrix.
    *   This method regroups answers by question and by row to make an object like :
    *   params = { 'matrixQuestionId' : { 'rowId1' : [colId1, colId2,...], 'rowId2' : [colId1, colId3, ...] } }
    */
    _prepareSubmitAnswerMatrix: function (params, questionId, rowId, colId, isComment) {
        var value = questionId in params ? params[questionId] : {};
        if (isComment) {
            value['comment'] = colId;
        } else {
            if (rowId in value) {
                value[rowId].push(colId);
            } else {
                value[rowId] = [colId];
            }
        }
        params[questionId] = value;
        return params;
    },

    /**
    *   Prepare answer before submitting form (any kind of answer - except Matrix -).
    *   This method regroups answers by question.
    *   Lonely answer are directly assigned to questionId. Multiple answers are regrouped in an array:
    *   params = { 'questionId1' : lonelyAnswer, 'questionId2' : [multipleAnswer1, multipleAnswer2, ...] }
    */
    _prepareSubmitAnswer: function (params, questionId, value) {
        if (questionId in params) {
            if (params[questionId].constructor === Array) {
                params[questionId].push(value);
            } else {
                params[questionId] = [params[questionId], value];
            }
        } else {
            params[questionId] = value;
        }
        return params;
    },

    /**
    *   Prepare comment before submitting form.
    *   This method extract the comment, encapsulate it in a dict and calls the _prepareSubmitAnswer methods
    *   with the new value. At the end, the result looks like :
    *   params = { 'questionId1' : {'comment': commentValue}, 'questionId2' : [multipleAnswer1, {'comment': commentValue}, ...] }
    */
    _prepareSubmitComment: function (params, $parent, questionId, isMatrix) {
        var self = this;
        $parent.find('textarea').each(function () {
            if (this.value) {
                var value = {'comment': this.value};
                if (isMatrix) {
                    params = self._prepareSubmitAnswerMatrix(params, questionId, this.name, this.value, true);
                } else {
                    params = self._prepareSubmitAnswer(params, questionId, value);
                }
            }
        });
        return params;
    },

    // INIT FIELDS TOOLS
    // -------------------------------------------------------------------------

   /**
    * Will allow the textarea to resize on carriage return instead of showing scrollbar.
    */
    _initTextArea: function () {
        this.$('textarea').each(function () {
            dom.autoresize($(this));
        });
    },

    _initChoiceItems: function () {
        this.$("input[type='radio'],input[type='checkbox']").each(function () {
            var matrixBtn = $(this).parents('.o_survey_matrix_btn');
            if ($(this).prop("checked")) {
                var $target = matrixBtn.length > 0 ? matrixBtn : $(this).closest('label');
                $target.addClass('o_survey_selected');
            }
        });
    },

    /**
     * Will initialize the breadcrumb widget that handles navigation to a previously filled in page.
     *
     * @private
     */
    _initBreadcrumb: function () {
        var $breadcrumb = this.$('.o_survey_breadcrumb_container');
        var pageId = this.$('input[name=page_id]').val();
        if ($breadcrumb.length) {
            this.surveyBreadcrumbWidget = new publicWidget.registry.SurveyBreadcrumbWidget(this, {
                'canGoBack': $breadcrumb.data('canGoBack'),
                'currentPageId': pageId ? parseInt(pageId) : 0,
                'pages': $breadcrumb.data('pages'),
            });
            this.surveyBreadcrumbWidget.appendTo($breadcrumb);
            $breadcrumb.removeClass('d-none');  // hidden by default to avoid having ghost div in start screen
        }
    },

    /**
     * Called after survey submit to update the breadcrumb to the right page.
     */
    _updateBreadcrumb: function () {
        if (this.surveyBreadcrumbWidget) {
            var pageId = this.$('input[name=page_id]').val();
            this.surveyBreadcrumbWidget.updateBreadcrumb(parseInt(pageId));
        } else {
            this._initBreadcrumb();
        }
    },

    /**
     * Will handle bus specific behavior for survey 'sessions'
     *
     * @private
     */
    _initSessionManagement: function () {
        var self = this;
        if (this.options.surveyToken && this.options.sessionInProgress) {
            this.call('bus_service', 'addChannel', this.options.surveyToken);
            this.call('bus_service', 'startPolling');

            if (!this._checkIsMasterTab()) {
                this.shouldReloadMasterTab = true;
                this.masterTabCheckInterval = setInterval(function() {
                     if (self._checkIsMasterTab()) {
                        clearInterval(self.masterTabCheckInterval);
                     }
                }, 2000);
            }

            this.call('bus_service', 'onNotification', this, this._onNotification);
        }
    },

    _initTimer: function () {
        if (this.surveyTimerWidget) {
            this.surveyTimerWidget.destroy();
        }

        var self = this;
        var $timerData = this.$('.o_survey_form_content_data');
        var questionTimeLimitReached = $timerData.data('questionTimeLimitReached');
        var timeLimitMinutes = $timerData.data('timeLimitMinutes');
        var hasAnswered = $timerData.data('hasAnswered');
        const serverTime = $timerData.data('serverTime');

        if (!questionTimeLimitReached && !hasAnswered && timeLimitMinutes) {
            var timer = $timerData.data('timer');
            var $timer = $('<span>', {
                class: 'o_survey_timer'
            });
            this.$('.o_survey_timer_container').append($timer);
            this.surveyTimerWidget = new publicWidget.registry.SurveyTimerWidget(this, {
                'serverTime': serverTime,
                'timer': timer,
                'timeLimitMinutes': timeLimitMinutes
            });
            this.surveyTimerWidget.attachTo($timer);
            this.surveyTimerWidget.on('time_up', this, function (ev) {
                self._submitForm({
                    'skipValidation': true,
                    'isFinish': !this.options.sessionInProgress
                });
            });
        }
    },

    /**
    * Initialize datetimepicker in correct format and with constraints
    */
    _initDateTimePicker: function ($dateGroup) {
        var disabledDates = [];
        var questionType = $dateGroup.find('input').data('questionType');
        var minDateData = $dateGroup.data('mindate');
        var maxDateData = $dateGroup.data('maxdate');

        var datetimepickerFormat = questionType === 'datetime' ? time.getLangDatetimeFormat() : time.getLangDateFormat();

        var minDate = minDateData
            ? this._formatDateTime(minDateData, datetimepickerFormat)
            : moment({ y: 1000 });

        var maxDate = maxDateData
            ? this._formatDateTime(maxDateData, datetimepickerFormat)
            : moment().add(200, "y");

        if (questionType === 'date') {
            // Include min and max date in selectable values
            maxDate = moment(maxDate).add(1, "d");
            minDate = moment(minDate).subtract(1, "d");
            disabledDates = [minDate, maxDate];
        }

        $dateGroup.datetimepicker({
            format : datetimepickerFormat,
            minDate: minDate,
            maxDate: maxDate,
            disabledDates: disabledDates,
            useCurrent: false,
            viewDate: moment(new Date()).hours(minDate.hours()).minutes(minDate.minutes()).seconds(minDate.seconds()).milliseconds(minDate.milliseconds()),
            calendarWeeks: true,
            icons: {
                time: 'fa fa-clock-o',
                date: 'fa fa-calendar',
                next: 'fa fa-chevron-right',
                previous: 'fa fa-chevron-left',
                up: 'fa fa-chevron-up',
                down: 'fa fa-chevron-down',
            },
            locale : moment.locale(),
            allowInputToggle: true,
        });
        $dateGroup.on('error.datetimepicker', function (err) {
            if (err.date) {
                if (err.date < minDate) {
                    Dialog.alert(this, _t('The date you selected is lower than the minimum date: ') + minDate.format(datetimepickerFormat));
                }

                if (err.date > maxDate) {
                    Dialog.alert(this, _t('The date you selected is greater than the maximum date: ') + maxDate.format(datetimepickerFormat));
                }
            }
            return false;
        });
    },

    _formatDateTime: function (datetimeValue, format){
        return moment(field_utils.format.datetime(moment(datetimeValue), null, {timezone: true}), format);
    },

    _initResultWidget: function () {
        var $result = this.$('.o_survey_result');
        if ($result.length) {
            this.surveyResultWidget = new publicWidget.registry.SurveyResultWidget(this);
            this.surveyResultWidget.attachTo($result);
            $result.fadeIn(this.fadeInOutDelay);
        }
    },

   /**
    * Will automatically focus on the first input to allow the user to complete directly the survey,
    * without having to manually get the focus (only if the input has the right type - can write something inside -)
    */
    _focusOnFirstInput: function () {
        var $firstTextInput = this.$('.js_question-wrapper').first()  // Take first question
                              .find("input[type='text'],input[type='number'],textarea")  // get 'text' inputs
                              .filter('.form-control')  // needed for the auto-resize
                              .not('.o_survey_comment');  // remove inputs for comments that does not count as answers
        if ($firstTextInput.length > 0) {
            $firstTextInput.focus();
        }
    },

    /**
    * This method check if the current tab is the master tab at the bus level.
    * If not, the survey could not receive next question notification anymore from session manager.
    * We then ask the participant to close all other tabs on the same hostname before letting them continue.
    *
    * @private
    */
    _checkIsMasterTab: function () {
        var isMasterTab = this.call('bus_service', 'isMasterTab');
        var $errorModal = this.$('#MasterTabErrorModal');
        if (isMasterTab) {
            // Force reload the page when survey is ready to be followed, to force restart long polling
            if (this.shouldReloadMasterTab) {
                window.location.reload();
            }
           return true;
        } else if (!$errorModal.modal._isShown){
            $errorModal.find('.text-danger').text(window.location.hostname);
            $errorModal.modal('show');
        }
        return false;
    },

    // CONDITIONAL QUESTIONS MANAGEMENT TOOLS
    // -------------------------------------------------------------------------

    /**
    * Clear / Un-select all the input from the given question
    * + propagate conditional hierarchy by triggering change on choice inputs.
    *
    * @private
    */
    _clearQuestionInputs: function (question) {
        question.find('input').each(function () {
            if ($(this).attr('type') === 'text' || $(this).attr('type') === 'number') {
                $(this).val('');
            } else if ($(this).prop('checked')) {
                $(this).prop('checked', false).change();
            }
        });
        question.find('textarea').val('');
    },

    /**
    * Get questions that are not supposed to be answered by the user.
    * Those are the ones triggered by answers that the user did not selected.
    *
    * @private
    */
    _getInactiveConditionalQuestionIds: function () {
        var self = this;
        var inactiveQuestionIds = [];
        if (this.options.triggeredQuestionsByAnswer) {
            Object.keys(this.options.triggeredQuestionsByAnswer).forEach(function (answerId) {
                if (!self.selectedAnswers.includes(parseInt(answerId))) {
                     self.options.triggeredQuestionsByAnswer[answerId].forEach(function (questionId) {
                        inactiveQuestionIds.push(questionId);
                     });
                }
            });
        }
        return inactiveQuestionIds;
    },

    // ERRORS TOOLS
    // -------------------------------------------------------------------------

    _showErrors: function (errors) {
        var self = this;
        var errorKeys = _.keys(errors);
        _.each(errorKeys, function (key) {
            self.$("#" + key + '>.o_survey_question_error').append($('<p>', {text: errors[key]})).addClass("slide_in");
            if (errorKeys[0] === key) {
                self._scrollToError(self.$('.js_question-wrapper#' + key));
            }
        });
    },

    _scrollToError: function ($target) {
        var scrollLocation = $target.offset().top;
        var navbarHeight = $('.o_main_navbar').height();
        if (navbarHeight) {
            // In overflow auto, scrollLocation of target can be negative if target is out of screen (up side)
            scrollLocation = scrollLocation >= 0 ? scrollLocation - navbarHeight : scrollLocation + navbarHeight;
        }
        var scrollinside = $("#wrapwrap").scrollTop();
        $('#wrapwrap').animate({
            scrollTop: scrollinside + scrollLocation
        }, 500);
    },

    /**
    * Clean all form errors in order to clean DOM before a new validation
    */
    _resetErrors: function () {
        this.$('.o_survey_question_error').empty().removeClass('slide_in');
        this.$('.o_survey_error').addClass('d-none');
    },

});

return publicWidget.registry.SurveyFormWidget;

});

```

## File: static\src\js\survey_print.js

```javascript
odoo.define('survey.print', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var dom = require('web.dom');

publicWidget.registry.SurveyPrintWidget = publicWidget.Widget.extend({
    selector: '.o_survey_print',

    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
    * @override
    */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            // Will allow the textarea to resize if any carriage return instead of showing scrollbar.
            self.$('textarea').each(function () {
                dom.autoresize($(this));
            });
        });
    },

});

return publicWidget.registry.SurveyPrintWidget;

});

```

## File: static\src\js\survey_quick_access.js

```javascript
odoo.define('survey.quick.access', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.SurveyQuickAccessWidget = publicWidget.Widget.extend({
    selector: '.o_survey_quick_access',
    events: {
        'click button[type="submit"]': '_onSubmit',
        'input #session_code': '_onSessionCodeInput',
    },

        //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
    * @override
    */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            // Init event listener
            if (!self.readonly) {
                $(document).on('keypress', self._onKeyPress.bind(self));
            }

            self.$('input').focus();
        });
    },

    // -------------------------------------------------------------------------
    // Private
    // -------------------------------------------------------------------------

    // Handlers
    // -------------------------------------------------------------------------

    _onSessionCodeInput: function () {
        this.el.querySelectorAll('.o_survey_error > span').forEach((elem) => elem.classList.add('d-none'));
    },

    _onKeyPress: function (event) {
        if (event.keyCode === 13) {  // Enter
            event.preventDefault();
            this._submitCode();
        }
    },

    _onSubmit: function (event) {
        event.preventDefault();
        this._submitCode();
    },

    _submitCode: function () {
        var self = this;
        this.$('.o_survey_error > span').addClass("d-none");
        const sessionCodeInputVal = this.$('input#session_code').val().trim();
        if (!sessionCodeInputVal) {
            self.$('.o_survey_session_error_invalid_code').removeClass("d-none");
            return;
        }
        this._rpc({
            route: `/survey/check_session_code/${sessionCodeInputVal}`,
        }).then(function (response) {
            if (response.survey_url) {
                window.location = response.survey_url;
            } else {
                if (response.error && response.error === 'survey_session_closed') {
                    self.$('.o_survey_session_error_closed').removeClass("d-none");
                } else {
                    self.$('.o_survey_session_error_invalid_code').removeClass("d-none");
                }
            }
        });
    },
});

return publicWidget.registry.SurveyQuickAccessWidget;

});

```

## File: static\src\js\survey_result.js

```javascript
odoo.define('survey.result', function (require) {
'use strict';

var _t = require('web.core')._t;
var ajax = require('web.ajax');
var publicWidget = require('web.public.widget');

// The given colors are the same as those used by D3
var D3_COLORS = ["#1f77b4","#ff7f0e","#aec7e8","#ffbb78","#2ca02c","#98df8a","#d62728",
                    "#ff9896","#9467bd","#c5b0d5","#8c564b","#c49c94","#e377c2","#f7b6d2",
                    "#7f7f7f","#c7c7c7","#bcbd22","#dbdb8d","#17becf","#9edae5"];

// TODO awa: this widget loads all records and only hides some based on page
// -> this is ugly / not efficient, needs to be refactored
publicWidget.registry.SurveyResultPagination = publicWidget.Widget.extend({
    events: {
        'click li.o_survey_js_results_pagination a': '_onPageClick',
    },

    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
     * @override
     * @param {$.Element} params.questionsEl The element containing the actual questions
     *   to be able to hide / show them based on the page number
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        this.$questionsEl = params.questionsEl;
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.limit = self.$el.data("record_limit");
        });
    },

    // -------------------------------------------------------------------------
    // Handlers
    // -------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onPageClick: function (ev) {
        ev.preventDefault();
        this.$('li.o_survey_js_results_pagination').removeClass('active');

        var $target = $(ev.currentTarget);
        $target.closest('li').addClass('active');
        this.$questionsEl.find('tbody tr').addClass('d-none');

        var num = $target.text();
        var min = (this.limit * (num-1))-1;
        if (min === -1){
            this.$questionsEl.find('tbody tr:lt('+ this.limit * num +')')
                .removeClass('d-none');
        } else {
            this.$questionsEl.find('tbody tr:lt('+ this.limit * num +'):gt(' + min + ')')
                .removeClass('d-none');
        }

    },
});

/**
 * Widget responsible for the initialization and the drawing of the various charts.
 *
 */
publicWidget.registry.SurveyResultChart = publicWidget.Widget.extend({
    jsLibs: [
        '/web/static/lib/Chart/Chart.js',
    ],

    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
     * Initializes the widget based on its defined graph_type and loads the chart.
     *
     * @override
     */
    start: function () {
        var self = this;

        return this._super.apply(this, arguments).then(function () {
            self.graphData = self.$el.data("graphData");

            if (self.graphData && self.graphData.length !== 0) {
                switch (self.$el.data("graphType")) {
                    case 'multi_bar':
                        self.chartConfig = self._getMultibarChartConfig();
                        break;
                    case 'bar':
                        self.chartConfig = self._getBarChartConfig();
                        break;
                    case 'pie':
                        self.chartConfig = self._getPieChartConfig();
                        break;
                    case 'doughnut':
                        self.chartConfig = self._getDoughnutChartConfig();
                        break;
                    case 'by_section':
                        self.chartConfig = self._getSectionResultsChartConfig();
                        break;
                }

                self._loadChart();
            }
        });
    },

    // -------------------------------------------------------------------------
    // Private
    // -------------------------------------------------------------------------

    /**
     * Returns a standard multi bar chart configuration.
     *
     * @private
     */
    _getMultibarChartConfig: function () {
        return {
            type: 'bar',
            data: {
                labels: this.graphData[0].values.map(function (value) {
                    return value.text;
                }),
                datasets: this.graphData.map(function (group, index) {
                    var data = group.values.map(function (value) {
                        return value.count;
                    });
                    return {
                        label: group.key,
                        data: data,
                        backgroundColor: D3_COLORS[index % 20],
                    };
                })
            },
            options: {
                scales: {
                    xAxes: [{
                        ticks: {
                            callback: this._customTick(25),
                        },
                    }],
                    yAxes: [{
                        ticks: {
                            beginAtZero: true,
                            precision: 0,
                        },
                    }],
                },
                tooltips: {
                    callbacks: {
                        title: function (tooltipItem, data) {
                            return data.labels[tooltipItem[0].index];
                        }
                    }
                },
            },
        };
    },

    /**
     * Returns a standard bar chart configuration.
     *
     * @private
     */
    _getBarChartConfig: function () {
        return {
            type: 'bar',
            data: {
                labels: this.graphData[0].values.map(function (value) {
                    return value.text;
                }),
                datasets: this.graphData.map(function (group) {
                    var data = group.values.map(function (value) {
                        return value.count;
                    });
                    return {
                        label: group.key,
                        data: data,
                        backgroundColor: data.map(function (val, index) {
                            return D3_COLORS[index % 20];
                        }),
                    };
                })
            },
            options: {
                legend: {
                    display: false,
                },
                scales: {
                    xAxes: [{
                        ticks: {
                            callback: this._customTick(35),
                        },
                    }],
                    yAxes: [{
                        ticks: {
                            beginAtZero: true,
                            precision: 0,
                        },
                    }],
                },
                tooltips: {
                    enabled: false,
                }
            },
        };
    },

    /**
     * Returns a standard pie chart configuration.
     *
     * @private
     */
    _getPieChartConfig: function () {
        var counts = this.graphData.map(function (point) {
            return point.count;
        });

        return {
            type: 'pie',
            data: {
                labels: this.graphData.map(function (point) {
                    return point.text;
                }),
                datasets: [{
                    label: '',
                    data: counts,
                    backgroundColor: counts.map(function (val, index) {
                        return D3_COLORS[index % 20];
                    }),
                }]
            }
        };
    },

    _getDoughnutChartConfig: function () {
        var totalsGraphData = this.graphData.totals;
        var counts = totalsGraphData.map(function (point) {
            return point.count;
        });

        return {
            type: 'doughnut',
            data: {
                labels: totalsGraphData.map(function (point) {
                    return point.text;
                }),
                datasets: [{
                    label: '',
                    data: counts,
                    backgroundColor: counts.map(function (val, index) {
                        return D3_COLORS[index % 20];
                    }),
                    borderColor: 'rgba(0, 0, 0, 0.1)'
                }]
            },
            options: {
                title: {
                    display: true,
                    text: _t("Overall Performance"),
                },
            }
        };
    },

    /**
     * Displays the survey results grouped by section.
     * For each section, user can see the percentage of answers
     * - Correct
     * - Partially correct (multiple choices and not all correct answers ticked)
     * - Incorrect
     * - Unanswered
     *
     * e.g:
     *
     * Mathematics:
     * - Correct 75%
     * - Incorrect 25%
     * - Partially correct 0%
     * - Unanswered 0%
     *
     * Geography:
     * - Correct 0%
     * - Incorrect 0%
     * - Partially correct 50%
     * - Unanswered 50%
     *
     *
     * @private
     */
    _getSectionResultsChartConfig: function () {
        var sectionGraphData = this.graphData.by_section;

        var resultKeys = {
            'correct': _t('Correct'),
            'partial': _t('Partially'),
            'incorrect': _t('Incorrect'),
            'skipped': _t('Unanswered'),
        };
        var resultColorIndex = 0;
        var datasets = [];
        for (var resultKey in resultKeys) {
            var data = [];
            for (var section in sectionGraphData) {
                data.push((sectionGraphData[section][resultKey]) / sectionGraphData[section]['question_count'] * 100);
            }
            datasets.push({
                label: resultKeys[resultKey],
                data: data,
                backgroundColor: D3_COLORS[resultColorIndex % 20],
            });
            resultColorIndex++;
        }

        return {
            type: 'bar',
            data: {
                labels: Object.keys(sectionGraphData),
                datasets: datasets
            },
            options: {
                title: {
                    display: true,
                    text: _t("Performance by Section"),
                },
                legend: {
                    display: true,
                },
                scales: {
                    xAxes: [{
                        ticks: {
                            callback: this._customTick(20),
                        },
                    }],
                    yAxes: [{
                        gridLines: {
                            display: false,
                        },
                        ticks: {
                            beginAtZero: true,
                            precision: 0,
                            callback: function (label) {
                                return label + '%';
                            },
                            suggestedMin: 0,
                            suggestedMax: 100,
                            maxTicksLimit: 5,
                            stepSize: 25,
                        },
                    }],
                },
                tooltips: {
                    callbacks: {
                        label: function (tooltipItem, data) {
                            var datasetLabel = data.datasets[tooltipItem.datasetIndex].label || '';
                            var roundedValue = Math.round(tooltipItem.yLabel * 100) / 100;
                            return `${datasetLabel}: ${roundedValue}%`;
                        }
                    }
                }
            },
        };
    },

    /**
     * Custom Tick function to replace overflowing text with '...'
     *
     * @private
     * @param {Integer} tickLimit
     */
    _customTick: function (tickLimit) {
        return function (label) {
            if (label.length <= tickLimit) {
                return label;
            } else {
                return label.slice(0, tickLimit) + '...';
            }
        };
    },

    /**
     * Loads the chart using the provided Chart library.
     *
     * @private
     */
    _loadChart: function () {
        this.$el.css({position: 'relative'});
        var $canvas = this.$('canvas');
        var ctx = $canvas.get(0).getContext('2d');
        return new Chart(ctx, this.chartConfig);
    }
});

publicWidget.registry.SurveyResultWidget = publicWidget.Widget.extend({
    selector: '.o_survey_result',
    events: {
        'click td.survey_answer i.fa-filter': '_onSurveyAnswerFilterClick',
        'click .clear_survey_filter': '_onClearFilterClick',
        'click span.filter-all': '_onFilterAllClick',
        'click span.filter-finished': '_onFilterFinishedClick',
    },

    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
    * @override
    */
    willStart: function () {
        var url = '/web/webclient/locale/' + (document.documentElement.getAttribute('lang') || 'en_US').replace('-', '_');
        var localeReady = ajax.loadJS(url);
        return Promise.all([this._super.apply(this, arguments), localeReady]);
    },

    /**
    * @override
    */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var allPromises = [];

            self.$('.pagination').each(function (){
                var questionId = $(this).data("question_id");
                allPromises.push(new publicWidget.registry.SurveyResultPagination(self, {
                    'questionsEl': self.$('#survey_table_question_'+ questionId)
                }).attachTo($(this)));
            });

            self.$('.survey_graph').each(function () {
                allPromises.push(new publicWidget.registry.SurveyResultChart(self)
                    .attachTo($(this)));
            });

            if (allPromises.length !== 0) {
                return Promise.all(allPromises);
            } else {
                return Promise.resolve();
            }
        });
    },

    // -------------------------------------------------------------------------
    // Handlers
    // -------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onSurveyAnswerFilterClick: function (ev) {
        var cell = $(ev.target);
        var row_id = cell.data('row_id') | 0;
        var answer_id = cell.data('answer_id');

        var params = new URLSearchParams(window.location.search);
        var filters = params.get('filters') ? params.get('filters') + "|" + row_id + ',' + answer_id : row_id + ',' + answer_id;
        params.set('filters', filters);

        window.location.href = window.location.pathname + '?' + params.toString();
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onClearFilterClick: function (ev) {
        var params = new URLSearchParams(window.location.search);
        params.delete('filters');
        params.delete('finished');
        window.location.href = window.location.pathname + '?' + params.toString();
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onFilterAllClick: function (ev) {
        var params = new URLSearchParams(window.location.search);
        params.delete('finished');
        window.location.href = window.location.pathname + '?' + params.toString();
    },

    /**
     * @private
     * @param {Event} ev
     */
    _onFilterFinishedClick: function (ev) {
        var params = new URLSearchParams(window.location.search);
        params.set('finished', true);
        window.location.href = window.location.pathname + '?' + params.toString();
    },
});

return {
    resultWidget: publicWidget.registry.SurveyResultWidget,
    chartWidget: publicWidget.registry.SurveyResultChart,
    paginationWidget: publicWidget.registry.SurveyResultPagination
};

});

```

## File: static\src\js\survey_session_chart.js

```javascript
odoo.define('survey.session_chart', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var SESSION_CHART_COLORS = require('survey.session_colors');

publicWidget.registry.SurveySessionChart = publicWidget.Widget.extend({
    init: function (parent, options) {
        this._super.apply(this, arguments);

        this.questionType = options.questionType;
        this.answersValidity = options.answersValidity;
        this.hasCorrectAnswers = options.hasCorrectAnswers;
        this.questionStatistics = this._processQuestionStatistics(options.questionStatistics);
        this.showInputs = options.showInputs;
        this.showAnswers = false;
    },

    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self._setupChart();
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Updates the chart data using the latest received question user inputs.
     *
     * By updating the numbers in the dataset, we take advantage of the Chartjs API
     * that will automatically add animations to show the new number.
     *
     * @param {Object} questionStatistics object containing chart data (counts / labels / ...)
     * @param {Integer} newAttendeesCount: max height of chart, not used anymore (deprecated)
     */
    updateChart: function (questionStatistics, newAttendeesCount) {
        if (questionStatistics) {
            this.questionStatistics = this._processQuestionStatistics(questionStatistics);
        }

        if (this.chart) {
            // only a single dataset for our bar charts
            var chartData = this.chart.data.datasets[0].data;
            for (var i = 0; i < chartData.length; i++){
                var value = 0;
                if (this.showInputs) {
                    value = this.questionStatistics[i].count;
                }
                this.chart.data.datasets[0].data[i] = value;
            }

            this.chart.update();
        }
    },

    /**
     * Toggling this parameter will display or hide the correct and incorrect answers of the current
     * question directly on the chart.
     *
     * @param {Boolean} showAnswers
     */
    setShowAnswers: function (showAnswers) {
        this.showAnswers = showAnswers;
    },

    /**
     * Toggling this parameter will display or hide the user inputs of the current question directly
     * on the chart.
     *
     * @param {Boolean} showInputs
     */
    setShowInputs: function (showInputs) {
        this.showInputs = showInputs;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _setupChart: function () {
        var $canvas = this.$('canvas');
        var ctx = $canvas.get(0).getContext('2d');

        this.chart = new Chart(ctx, this._buildChartConfiguration());
    },

    /**
     * Custom bar chart configuration for our survey session use case.
     *
     * Quick summary of enabled features:
     * - background_color is one of the 10 custom colors from SESSION_CHART_COLORS
     *   (see _getBackgroundColor for details)
     * - The ticks are bigger and bolded to be able to see them better on a big screen (projector)
     * - We don't use tooltips to keep it as simple as possible
     * - We don't set a suggestedMin or Max so that Chart will adapt automatically himself based on the given data
     *   The '+1' part is a small trick to avoid the datalabels to be clipped in height
     * - We use a custom 'datalabels' plugin to be able to display the number value on top of the
     *   associated bar of the chart.
     *   This allows the host to discuss results with attendees in a more interactive way.
     *
     * @private
     */
    _buildChartConfiguration: function () {
        return {
            type: 'bar',
            data: {
                labels: this._extractChartLabels(),
                datasets: [{
                    backgroundColor: this._getBackgroundColor.bind(this),
                    data: this._extractChartData(),
                }]
            },
            options: {
                maintainAspectRatio: false,
                plugins: {
                    datalabels: {
                        color: this._getLabelColor.bind(this),
                        font: {
                            size: '50',
                            weight: 'bold',
                        },
                        anchor: 'end',
                        align: 'top',
                    }
                },
                legend: {
                    display: false,
                },
                scales: {
                    yAxes: [{
                        ticks: {
                            display: false,
                        },
                        gridLines: {
                            display: false
                        }
                    }],
                    xAxes: [{
                        ticks: {
                            maxRotation: 0,
                            fontSize: '35',
                            fontStyle: 'bold',
                            fontColor: '#212529',
                            autoSkip: false,
                        },
                        gridLines: {
                            drawOnChartArea: false,
                            color: 'rgba(0, 0, 0, 0.2)'
                        }
                    }]
                },
                tooltips: {
                    enabled: false,
                },
                layout: {
                    padding: {
                        left: 0,
                        right: 0,
                        top: 70,
                        bottom: 0
                    }
                }
            },
            plugins: [{
                /**
                 * The way it works is each label is an array of words.
                 * eg.: if we have a chart label: "this is an example of a label"
                 * The library will split it as: ["this is an example", "of a label"]
                 * Each value of the array represents a line of the label.
                 * So for this example above: it will be displayed as:
                 * "this is an examble<br/>of a label", breaking the label in 2 parts and put on 2 lines visually.
                 * 
                 * What we do here is rework the labels with our own algorithm to make them fit better in screen space
                 * based on breakpoints based on number of columns to display.
                 * So this example will become: ["this is an", "example of", "a label"] if we have a lot of labels to put in the chart.
                 * Which will be displayed as "this is an<br/>example of<br/>a label"
                 * Obviously, the more labels you have, the more columns, and less screen space is available.
                 * When the screen space is too small for long words, those long words are split over multiple rows.
                 * At 6 chars per row, the above example becomes ["this", "is an", "examp-", "le of", "a label"]
                 * Which is displayed as "this<br/>is an<br/>examp-<br/>le of<br/>a label"
                 * 
                 * We also adapt the font size based on the width available in the chart.
                 * 
                 * So we counterbalance multiple times:
                 * - Based on number of columns (i.e. number of survey.question.answer of your current survey.question),
                 *   we split the words of every labels to make them display on more rows.
                 * - Based on the width of the chart (which is equivalent to screen width),
                 *   we reduce the chart font to be able to fit more characters.
                 * - Based on the longest word present in the labels, we apply a certain ratio with the width of the chart
                 *   to get a more accurate font size for the space available.
                 * 
                 * @param {Object} chart 
                 */
                beforeInit: function (chart) {
                    const nbrCol = chart.data.labels.length;
                    const minRatio = 0.4;
                    // Numbers of maximum characters per line to print based on the number of columns and default ratio for the font size
                    // Between 1 and 2 -> 25, 3 and 4 -> 20, 5 and 6 -> 15, ...
                    const charPerLineBreakpoints = [
                        [1, 2, 25, minRatio],
                        [3, 4, 20, minRatio],
                        [5, 6, 15, 0.45],
                        [7, 8, 10, 0.65],
                        [9, null, 7, 0.7],
                    ];

                    let charPerLine;
                    let fontRatio;
                    charPerLineBreakpoints.forEach(([lowerBound, upperBound, value, ratio]) => {
                        if (nbrCol >= lowerBound && (upperBound === null || nbrCol <= upperBound)) {
                            charPerLine = value;
                            fontRatio = ratio;
                        }
                    });

                    // Adapt font size if the number of characters per line is under the maximum
                    if (charPerLine < 25) {
                        const allWords = chart.data.labels.reduce((accumulator, words) => accumulator.concat(' '.concat(words)));
                        const maxWordLength = Math.max(...allWords.split(' ').map((word) => word.length));
                        fontRatio = maxWordLength > charPerLine ? minRatio : fontRatio;
                        chart.options.scales.xAxes[0].ticks.fontSize = Math.min(parseInt(chart.options.scales.xAxes[0].ticks.fontSize), chart.width * fontRatio / (nbrCol));
                    }

                    chart.data.labels.forEach(function (label, index, labelsList) {
                        // Split all the words of the label
                        const words = label.split(" ");
                        let resultLines = [];
                        let currentLine = [];
                        for (let i = 0; i < words.length; i++) {
                            // Chop down words that do not fit on a single line, add each part on its own line.
                            let word = words[i];
                            while (word.length > charPerLine) {
                                resultLines.push(word.slice(0, charPerLine - 1) + '-');
                                word = word.slice(charPerLine - 1);
                            }
                            currentLine.push(word);

                            // Continue to add words in the line if there is enough space and if there is at least one more word to add
                            const nextWord = i+1 < words.length ? words[i+1] : null;
                            if (nextWord) {
                                const nextLength = currentLine.join(' ').length + nextWord.length;
                                if (nextLength <= charPerLine) {
                                    continue;
                                }
                            }
                            // Add the constructed line and reset the variable for the next line
                            const newLabelLine = currentLine.join(' ');
                            resultLines.push(newLabelLine);
                            currentLine = [];
                        }
                        labelsList[index] = resultLines;
                    });
                },
            }],
        };
    },

    /**
     * Returns the label of the associated survey.question.answer.
     *
     * @private
     */
    _extractChartLabels: function () {
        return this.questionStatistics.map(function (point) {
            return point.text;
        });
    },

    /**
     * We simply return an array of zeros as initial value.
     * The chart will update afterwards as attendees add their user inputs.
     *
     * @private
     */
    _extractChartData: function () {
        return this.questionStatistics.map(function () {
            return 0;
        });
    },

    /**
     * Custom method that returns a color from SESSION_CHART_COLORS.
     * It loops through the ten values and assign them sequentially.
     *
     * We have a special mechanic when the host shows the answers of a question.
     * Wrong answers are "faded out" using a 0.3 opacity.
     *
     * @param {Object} metaData
     * @param {Integer} metaData.dataIndex the index of the label, matching the index of the answer
     *   in 'this.answersValidity'
     * @private
     */
    _getBackgroundColor: function (metaData) {
        var opacity = '0.8';
        if (this.showAnswers && this.hasCorrectAnswers) {
            if (!this._isValidAnswer(metaData.dataIndex)){
                opacity = '0.2';
            }
        }
        var rgb = SESSION_CHART_COLORS[metaData.dataIndex];
        return `rgba(${rgb},${opacity})`;
    },

    /**
     * Custom method that returns the survey.question.answer label color.
     *
     * Break-down of use cases:
     * - Red if the host is showing answer, and the associated answer is not correct
     * - Green if the host is showing answer, and the associated answer is correct
     * - Black in all other cases
     *
     * @param {Object} metaData
     * @param {Integer} metaData.dataIndex the index of the label, matching the index of the answer
     *   in 'this.answersValidity'
     * @private
     */
    _getLabelColor: function (metaData) {
        if (this.showAnswers && this.hasCorrectAnswers) {
            if (this._isValidAnswer(metaData.dataIndex)){
                return '#2CBB70';
            } else {
                return '#D9534F';
            }
        }
        return '#212529';
    },

    /**
     * Small helper method that returns the validity of the answer based on its index.
     *
     * We need this special handling because of Chartjs data structure.
     * The library determines the parameters (color/label/...) by only passing the answer 'index'
     * (and not the id or anything else we can identify).
     *
     * @param {Integer} answerIndex
     * @private
     */
    _isValidAnswer: function (answerIndex) {
        return this.answersValidity[answerIndex];
    },

    /**
     * Special utility method that will process the statistics we receive from the
     * survey.question#_prepare_statistics method.
     *
     * For multiple choice questions, the values we need are stored in a different place.
     * We simply return the values to make the use of the statistics common for both simple and
     * multiple choice questions.
     *
     * See survey.question#_get_stats_data for more details
     *
     * @param {Object} rawStatistics
     * @private
     */
    _processQuestionStatistics: function (rawStatistics) {
        if (this.questionType === 'multiple_choice') {
            return rawStatistics[0].values;
        }

        return rawStatistics;
    }
});

return publicWidget.registry.SurveySessionChart;

});

```

## File: static\src\js\survey_session_colors.js

```javascript
odoo.define('survey.session_colors', function (require) {
'use strict';

/**
 * Small tool that returns common colors for survey session widgets.
 * Source: https://www.materialui.co/colors (500)
 */
return [
    '33,150,243',
    '63,81,181',
    '205,220,57',
    '0,150,136',
    '76,175,80',
    '121,85,72',
    '158,158,158',
    '156,39,176',
    '96,125,139',
    '244,67,54',
];

});

```

## File: static\src\js\survey_session_leaderboard.js

```javascript
odoo.define('survey.session_leaderboard', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var SESSION_CHART_COLORS = require('survey.session_colors');

publicWidget.registry.SurveySessionLeaderboard = publicWidget.Widget.extend({
    init: function (parent, options) {
        this._super.apply(this, arguments);

        this.surveyAccessToken = options.surveyAccessToken;
        this.$sessionResults = options.sessionResults;

        this.BAR_MIN_WIDTH = '3rem';
        this.BAR_WIDTH = '24rem';
        this.BAR_HEIGHT = '3.8rem';
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Shows the question leaderboard on screen.
     * It's based on the attendees score (descending).
     *
     * We fade out the $sessionResults to fade in our rendered template.
     *
     * The width of the progress bars is set after the rendering to enable a width css animation.
     */
    showLeaderboard: function (fadeOut, isScoredQuestion) {
        var self = this;

        var resolveFadeOut;
        var fadeOutPromise;
        if (fadeOut) {
            fadeOutPromise = new Promise(function (resolve, reject) { resolveFadeOut = resolve; });
            self.$sessionResults.fadeOut(400, function () {
                resolveFadeOut();
            });
        } else {
            fadeOutPromise = Promise.resolve();
            self.$sessionResults.hide();
            self.$('.o_survey_session_leaderboard_container').empty();
        }

        var leaderboardPromise = this._rpc({
            route: _.str.sprintf('/survey/session/leaderboard/%s', this.surveyAccessToken)
        });

        Promise.all([fadeOutPromise, leaderboardPromise]).then(function (results) {
            var leaderboardResults = results[1];
            var $renderedTemplate = $(leaderboardResults);
            self.$('.o_survey_session_leaderboard_container').append($renderedTemplate);

            self.$('.o_survey_session_leaderboard_item').each(function (index) {
                var rgb = SESSION_CHART_COLORS[index % 10];
                $(this)
                    .find('.o_survey_session_leaderboard_bar')
                    .css('background-color', `rgba(${rgb},1)`);
                $(this)
                    .find('.o_survey_session_leaderboard_bar_question')
                    .css('background-color', `rgba(${rgb},${0.4})`);
            });

            self.$el.fadeIn(400, async function () {
                if (isScoredQuestion) {
                    await self._prepareScores();
                    await self._showQuestionScores();
                    await self._sumScores();
                    await self._reorderScores();
                }
            });
        });
    },

    /**
     * Inverse the process, fading out our template to fade int the $sessionResults.
     */
    hideLeaderboard: function () {
        var self = this;
        this.$el.fadeOut(400, function () {
            self.$('.o_survey_session_leaderboard_container').empty();
            self.$sessionResults.fadeIn(400);
        });
    },

    /**
     * This method animates the passed jQuery element from 0 points to {totalScore} points.
     * It will create a nice "animated" effect of a counter increasing by {increment} until it
     * reaches the actual score.
     *
     * @param {$.Element} $scoreEl the element to animate
     * @param {Integer} currentScore the currently displayed score
     * @param {Integer} totalScore to total score to animate to
     * @param {Integer} increment the base increment of each animation iteration
     * @param {Boolean} plusSign wether or not we add a "+" before the score
     * @private
     */
    _animateScoreCounter: function ($scoreEl, currentScore, totalScore, increment, plusSign) {
        var self = this;
        setTimeout(function () {
            var nextScore = currentScore + increment;
            if (nextScore > totalScore) {
                nextScore = totalScore;
            }
            $scoreEl.text(`${plusSign ? '+ ' : ''}${Math.round(nextScore)} p`);

            if (nextScore < totalScore) {
                self._animateScoreCounter($scoreEl, nextScore, totalScore, increment, plusSign);
            }
        }, 25);
    },

    /**
     * Helper to move a score bar from its current position in the leaderboard
     * to a new position.
     *
     * @param {$.Element} $score the score bar to move
     * @param {Integer} position the new position in the leaderboard
     * @param {Integer} offset an offset in 'rem'
     * @param {Integer} timeout time to wait while moving before resolving the promise
     */
    _animateMoveTo: function ($score, position, offset, timeout) {
        var animationDone;
        var animationPromise = new Promise(function (resolve) {
            animationDone = resolve;
        });
        $score.css('top', `calc(calc(${this.BAR_HEIGHT} * ${position}) + ${offset}rem)`);
        setTimeout(animationDone, timeout);
        return animationPromise;
    },

    /**
     * Takes the leaderboard prior to the current question results
     * and reduce all scores bars to a small width (3rem).
     * We keep the small score bars on screen for 1s.
     *
     * This visually prepares the display of points for the current question.
     *
     * @private
     */
    _prepareScores: function () {
        var self = this;
        var animationDone;
        var animationPromise = new Promise(function (resolve) {
            animationDone = resolve;
        });
        setTimeout(function () {
            this.$('.o_survey_session_leaderboard_bar').each(function () {
                var currentScore = parseInt($(this)
                    .closest('.o_survey_session_leaderboard_item')
                    .data('currentScore'))
                if (currentScore && currentScore !== 0) {
                    $(this).css('transition', `width 1s cubic-bezier(.4,0,.4,1)`);
                    $(this).css('width', self.BAR_MIN_WIDTH);
                }
            });
            setTimeout(animationDone, 1000);
        }, 300);

        return animationPromise;
    },

    /**
     * Now that we have summed the score for the current question to the total score
     * of the user and re-weighted the bars accordingly, we need to re-order everything
     * to match the new ranking.
     *
     * In addition to moving the bars to their new position, we create a "bounce" effect
     * by moving the bar a little bit more to the top or bottom (depending on if it's moving up
     * the ranking or down), the moving it the other way around, then moving it to its final
     * position.
     *
     * (Feels complicated when explained but it's fairly simple once you see what it does).
     *
     * @private
     */
    _reorderScores: function () {
        var self = this;
        var animationDone;
        var animationPromise = new Promise(function (resolve) {
            animationDone = resolve;
        });
        setTimeout(function () {
            self.$('.o_survey_session_leaderboard_item').each(async function () {
                var $score = $(this);
                var currentPosition = parseInt($(this).data('currentPosition'));
                var newPosition = parseInt($(this).data('newPosition'));
                if (currentPosition !== newPosition) {
                    var offset = newPosition > currentPosition ? 2 : -2;
                    await self._animateMoveTo($score, newPosition, offset, 300);
                    $score.css('transition', 'top ease-in-out .1s');
                    await self._animateMoveTo($score, newPosition, offset * -0.3, 100);
                    await self._animateMoveTo($score, newPosition, 0, 0);
                    animationDone();
                }
            });
        }, 1800);

        return animationPromise;
    },

    /**
     * Will display the score for the current question.
     * We simultaneously:
     * - increase the width of "question bar"
     *   (faded out bar right next to the global score one)
     * - animate the score for the question (ex: from + 0 p to + 40 p)
     *
     * (We keep a minimum width of 3rem to be able to display '+30 p' within the bar).
     *
     * @private
     */
    _showQuestionScores: function () {
        var self = this;
        var animationDone;
        var animationPromise = new Promise(function (resolve) {
            animationDone = resolve;
        });
        setTimeout(function () {
            this.$('.o_survey_session_leaderboard_bar_question').each(function () {
                var $barEl = $(this);
                var width = `calc(calc(100% - ${self.BAR_WIDTH}) * ${$barEl.data('widthRatio')} + ${self.BAR_MIN_WIDTH})`;
                $barEl.css('transition', 'width 1s ease-out');
                $barEl.css('width', width);

                var $scoreEl = $barEl
                    .find('.o_survey_session_leaderboard_bar_question_score')
                    .text('0 p');
                var questionScore = parseInt($barEl.data('questionScore'));
                if (questionScore && questionScore > 0) {
                    var increment = parseInt($barEl.data('maxQuestionScore') / 40);
                    if (!increment || increment === 0){
                        increment = 1;
                    }
                    $scoreEl.text('+ 0 p');
                    console.log($barEl.data('maxQuestionScore'));
                    setTimeout(function () {
                        self._animateScoreCounter(
                            $scoreEl,
                            0,
                            questionScore,
                            increment,
                            true);
                    }, 400);
                }
                setTimeout(animationDone, 1400);
            });
        }, 300);

        return animationPromise;
    },

    /**
     * After displaying the score for the current question, we sum the total score
     * of the user so far with the score of the current question.
     *
     * Ex:
     * We have ('#' for total score before question and '=' for current question score):
     * 210 p ####=================================== +30 p John
     * We want:
     * 240 p ###################################==== +30 p John
     *
     * Of course, we also have to weight the bars based on the maximum score.
     * So if John here has 50% of the points of the leader user, both the question score bar
     * and the total score bar need to have their width divided by 2:
     * 240 p ##################== +30 p John
     *
     * The width of both bars move at the same time to reach their new position,
     * with an animation on the width property.
     * The new width of the "question bar" should represent the ratio of won points
     * when compared to the total points.
     * (We keep a minimum width of 3rem to be able to display '+30 p' within the bar).
     *
     * The updated total score is animated towards the new value.
     * we keep this on screen for 500ms before reordering the bars.
     *
     * @private
     */
    _sumScores: function () {
        var self = this;
        var animationDone;
        var animationPromise = new Promise(function (resolve) {
            animationDone = resolve;
        });
        // values that felt the best after a lot of testing
        var growthAnimation = 'cubic-bezier(.5,0,.66,1.11)';
        setTimeout(function () {
            this.$('.o_survey_session_leaderboard_item').each(function () {
                var currentScore = parseInt($(this).data('currentScore'));
                var updatedScore = parseInt($(this).data('updatedScore'));
                var increment = parseInt($(this).data('maxQuestionScore') / 40);
                if (!increment || increment === 0){
                    increment = 1;
                }
                self._animateScoreCounter(
                    $(this).find('.o_survey_session_leaderboard_score'),
                    currentScore,
                    updatedScore,
                    increment,
                    false);

                var maxUpdatedScore = parseInt($(this).data('maxUpdatedScore'));
                var baseRatio = updatedScore / maxUpdatedScore;
                var questionScore = parseInt($(this).data('questionScore'));
                var questionRatio = questionScore /
                    (updatedScore && updatedScore !== 0 ? updatedScore : 1);
                // we keep a min fixed with of 3rem to be able to display "+ 5 p"
                // even if the user already has 1.000.000 points
                var questionWith = `calc(calc(calc(100% - ${self.BAR_WIDTH}) * ${questionRatio * baseRatio}) + ${self.BAR_MIN_WIDTH})`;
                $(this)
                    .find('.o_survey_session_leaderboard_bar_question')
                    .css('transition', `width ease .5s ${growthAnimation}`)
                    .css('width', questionWith);

                var updatedScoreRatio = 1 - questionRatio;
                var updatedScoreWidth = `calc(calc(100% - ${self.BAR_WIDTH}) * ${updatedScoreRatio * baseRatio})`;
                $(this)
                    .find('.o_survey_session_leaderboard_bar')
                    .css('min-width', '0px')
                    .css('transition', `width ease .5s ${growthAnimation}`)
                    .css('width', updatedScoreWidth);

                setTimeout(animationDone, 500);
            });
        }, 1400);

        return animationPromise;
    }
});

return publicWidget.registry.SurveySessionLeaderboard;

});

```

## File: static\src\js\survey_session_manage.js

```javascript
odoo.define('survey.session_manage', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var SurveySessionChart = require('survey.session_chart');
var SurveySessionTextAnswers = require('survey.session_text_answers');
var SurveySessionLeaderBoard = require('survey.session_leaderboard');
var core = require('web.core');
var _t = core._t;

publicWidget.registry.SurveySessionManage = publicWidget.Widget.extend({
    selector: '.o_survey_session_manage',
    events: {
        'click .o_survey_session_copy': '_onCopySessionLink',
        'click .o_survey_session_navigation_next, .o_survey_session_start': '_onNext',
        'click .o_survey_session_navigation_previous': '_onBack',
        'click .o_survey_session_close': '_onEndSessionClick',
    },

    /**
     * Overridden to set a few properties that come from the python template rendering.
     *
     * We also handle the timer IF we're not "transitioning", meaning a fade out of the previous
     * $el to the next question (the fact that we're transitioning is in the isRpcCall data).
     * If we're transitioning, the timer is handled manually at the end of the transition.
     */
    start: function () {
        var self = this;
        this.fadeInOutTime = 500;
        return this._super.apply(this, arguments).then(function () {
            // general survey props
            self.surveyId = self.$el.data('surveyId');
            self.surveyAccessToken = self.$el.data('surveyAccessToken');
            self.isStartScreen = self.$el.data('isStartScreen');
            self.isFirstQuestion = self.$el.data('isFirstQuestion');
            self.isLastQuestion = self.$el.data('isLastQuestion');
            // scoring props
            self.isScoredQuestion = self.$el.data('isScoredQuestion');
            self.sessionShowLeaderboard = self.$el.data('sessionShowLeaderboard');
            self.hasCorrectAnswers = self.$el.data('hasCorrectAnswers');
            // display props
            self.showBarChart = self.$el.data('showBarChart');
            self.showTextAnswers = self.$el.data('showTextAnswers');

            var isRpcCall = self.$el.data('isRpcCall');
            if (!isRpcCall) {
                self._startTimer();
                $(document).on('keydown', self._onKeyDown.bind(self));
            }

            self._setupIntervals();
            self._setupCurrentScreen();
            var setupPromises = [];
            setupPromises.push(self._setupTextAnswers());
            setupPromises.push(self._setupChart());
            setupPromises.push(self._setupLeaderboard());

            return Promise.all(setupPromises);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Copies the survey URL link to the clipboard.
     * We use 'ClipboardJS' to avoid having to print the URL in a standard text input
     *
     * @param {MouseEvent} ev
     */
    _onCopySessionLink: function (ev) {
        var self = this;
        ev.preventDefault();

        var $clipboardBtn = this.$('.o_survey_session_copy');

        $clipboardBtn.popover({
            placement: 'right',
            container: 'body',
            offset: '0, 3',
            content: function () {
                return _t("Copied !");
            }
        });

        var clipboard = new ClipboardJS('.o_survey_session_copy', {
            text: function () {
                return self.$('.o_survey_session_copy_url').val();
            },
            container: this.el
        });

        clipboard.on('success', function () {
            clipboard.destroy();
            $clipboardBtn.popover('show');
            _.delay(function () {
                $clipboardBtn.popover('hide');
            }, 800);
        });

        clipboard.on('error', function (e) {
            clipboard.destroy();
        });
    },

    /**
     * Listeners for keyboard arrow / spacebar keys.
     *
     * - 39 = arrow-right
     * - 32 = spacebar
     * - 37 = arrow-left
     *
     * @param {KeyboardEvent} ev
     */
    _onKeyDown: function (ev) {
        var keyCode = ev.keyCode;

        if (keyCode === 39 || keyCode === 32) {
            this._onNext(ev);
        } else if (keyCode === 37) {
            this._onBack(ev);
        }
    },

    /**
     * Handles the "next screen" behavior.
     * It happens when the host uses the keyboard key / button to go to the next screen.
     * The result depends on the current screen we're on.
     *
     * Possible values of the "next screen" to display are:
     * - 'userInputs' when going from a question to the display of attendees' survey.user_input.line
     *   for that question.
     * - 'results' when going from the inputs to the actual correct / incorrect answers of that
     *   question. Only used for scored simple / multiple choice questions.
     * - 'leaderboard' (or 'leaderboardFinal') when going from the correct answers of a question to
     *   the leaderboard of attendees. Only used for scored simple / multiple choice questions.
     * - If it's not one of the above: we go to the next question, or end the session if we're on
     *   the last question of this session.
     *
     * See '_getNextScreen' for a detailed logic.
     *
     * @param {Event} ev
     */
    _onNext: function (ev) {
        ev.preventDefault();

        var screenToDisplay = this._getNextScreen();

        if (screenToDisplay === 'userInputs') {
            this._setShowInputs(true);
        } else if (screenToDisplay === 'results') {
            this._setShowAnswers(true);
            // when showing results, stop refreshing answers
            clearInterval(this.resultsRefreshInterval);
            delete this.resultsRefreshInterval;
        } else if (['leaderboard', 'leaderboardFinal'].includes(screenToDisplay)
                   && !['leaderboard', 'leaderboardFinal'].includes(this.currentScreen)) {
            if (this.isLastQuestion) {
                this.$('.o_survey_session_navigation_next').addClass('d-none');
            }
            this.leaderBoard.showLeaderboard(true, this.isScoredQuestion);
        } else {
            if (!this.isLastQuestion) {
                this._nextQuestion();
            } else if (!this.sessionShowLeaderboard) {
                // If we have no leaderboard to show, directly end the session
                this.$('.o_survey_session_close').click();
            }
        }

        this.currentScreen = screenToDisplay;
    },

    /**
     * Reverse behavior of '_onNext'.
     *
     * @param {Event} ev
     */
    _onBack: function (ev) {
        ev.preventDefault();

        var screenToDisplay = this._getPreviousScreen();

        if (screenToDisplay === 'question') {
            this._setShowInputs(false);
        } else if (screenToDisplay === 'userInputs') {
            this._setShowAnswers(false);
            // resume refreshing answers if necessary
            if (!this.resultsRefreshInterval) {
                this.resultsRefreshInterval = setInterval(this._refreshResults.bind(this), 2000);
            }
        } else if (screenToDisplay === 'results') {
            if (this.leaderBoard) {
                this.leaderBoard.hideLeaderboard();
            }
            // when showing results, stop refreshing answers
            clearInterval(this.resultsRefreshInterval);
            delete this.resultsRefreshInterval;
        } else if (screenToDisplay === 'previousQuestion') {
            if (this.isFirstQuestion) {
                return;  // nothing to go back to, we're on the first question
            }
            this._nextQuestion(true);
        }

        this.currentScreen = screenToDisplay;
    },

    /**
     * Marks this session as 'done' and redirects the user to the results based on the clicked link.
     *
     * @param {MouseEvent} ev
     * @private
    */
    _onEndSessionClick: function (ev) {
        var self = this;
        ev.preventDefault();

        this._rpc({
            model: 'survey.survey',
            method: 'action_end_session',
            args: [[this.surveyId]],
        }).then(function () {
            if ($(ev.currentTarget).data('showResults')) {
                document.location = _.str.sprintf(
                    '/survey/results/%s',
                    self.surveyId
                );
            } else {
                window.history.back();
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Business logic that determines the 'next screen' based on the current screen and the question
     * configuration.
     *
     * Breakdown of use cases:
     * - If we're on the 'question' screen, and the question is scored, we move to the 'userInputs'
     * - If we're on the 'question' screen and it's NOT scored, then we move to
     *     - 'results' if the question has correct / incorrect answers
     *       (but not scored, which is kind of a corner case)
     *     - 'nextQuestion' otherwise
     * - If we're on the 'userInputs' screen and the question has answers, we move to the 'results'
     * - If we're on the 'results' and the question is scored, we move to the 'leaderboard'
     * - In all other cases, we show the next question
     * - (Small exception for the last question: we show the "final leaderboard")
     *
     * (For details about which screen shows what, see '_onNext')
     */
    _getNextScreen: function () {
        if (this.currentScreen === 'question' && this.isScoredQuestion) {
            return 'userInputs';
        } else if (this.hasCorrectAnswers && ['question', 'userInputs'].includes(this.currentScreen)) {
            return 'results';
        } else if (this.sessionShowLeaderboard) {
            if (['question', 'userInputs', 'results'].includes(this.currentScreen) && this.isScoredQuestion) {
                return 'leaderboard';
            } else if (this.isLastQuestion) {
                return 'leaderboardFinal';
            }
        }
        return 'nextQuestion';
    },

    /**
     * Reverse behavior of '_getNextScreen'.
     *
     * @param {Event} ev
     */
    _getPreviousScreen: function () {
        if (this.currentScreen === 'userInputs' && this.isScoredQuestion) {
            return 'question';
        } else if ((this.currentScreen === 'results' && this.isScoredQuestion) ||
                  (this.currentScreen === 'leaderboard' && !this.isScoredQuestion) ||
                  (this.currentScreen === 'leaderboardFinal' && this.isScoredQuestion)) {
            return 'userInputs';
        } else if ((this.currentScreen === 'leaderboard' && this.isScoredQuestion) ||
                  (this.currentScreen === 'leaderboardFinal' && !this.isScoredQuestion)){
            return 'results';
        }

        return 'previousQuestion';
    },

    /**
    * We use a fade in/out mechanism to display the next question of the session.
    *
    * The fade out happens at the same moment as the _rpc to get the new question template.
    * When they're both finished, we update the HTML of this widget with the new template and then
    * fade in the updated question to the user.
    *
    * The timer (if configured) starts at the end of the fade in animation.
    *
    * @param {MouseEvent} ev
    * @private
    */
    _nextQuestion: function (goBack) {
        var self = this;

        this.isStartScreen = false;
        if (this.surveyTimerWidget) {
            this.surveyTimerWidget.destroy();
        }

        var resolveFadeOut;
        var fadeOutPromise = new Promise(function (resolve, reject) { resolveFadeOut = resolve; });
        this.$el.fadeOut(this.fadeInOutTime, function () {
            resolveFadeOut();
        });

        var nextQuestionPromise = this._rpc({
            route: _.str.sprintf('/survey/session/next_question/%s', self.surveyAccessToken),
            params: {
                'go_back': goBack,
            }
        });

        // avoid refreshing results while transitioning
        if (this.resultsRefreshInterval) {
            clearInterval(this.resultsRefreshInterval);
            delete this.resultsRefreshInterval;
        }

        Promise.all([fadeOutPromise, nextQuestionPromise]).then(async function (results) {
            if (results[1]) {
                var $renderedTemplate = $(results[1]);
                self.$el.replaceWith($renderedTemplate);

                // Ensure new question is fully loaded before force loading previous question screen.
                await self.attachTo($renderedTemplate);
                if (goBack) {
                    // As we arrive on "question" screen, simulate going to the results screen or leaderboard.
                    self._setShowInputs(true);
                    self._setShowAnswers(true);
                    if (self.sessionShowLeaderboard && self.isScoredQuestion) {
                        self.currentScreen = 'leaderboard';
                        self.leaderBoard.showLeaderboard(false, self.isScoredQuestion);
                    } else {
                        self.currentScreen = 'results';
                        self._refreshResults();
                    }
                } else {
                    self._startTimer();
                }
                self.$el.fadeIn(self.fadeInOutTime);
            } else if (self.sessionShowLeaderboard) {
                // Display last screen if leaderboard activated
                self.isLastQuestion = true;
                self._setupLeaderboard().then(function () {
                    self.$('.o_survey_session_leaderboard_title').text(_t('Final Leaderboard'));
                    self.$('.o_survey_session_navigation_next').addClass('d-none');
                    self.$('.o_survey_leaderboard_buttons').removeClass('d-none');
                    self.leaderBoard.showLeaderboard(false, false);
                });
            } else {
                self.$('.o_survey_session_close').click();
            }
        });
    },

    /**
     * Will start the question timer so that the host may know when the question is done to display
     * the results and the leaderboard.
     *
     * If the question is scored, the timer ending triggers the display of attendees inputs.
     */
    _startTimer: function () {
        var self = this;
        var $timer = this.$('.o_survey_timer');

        if ($timer.length) {
            var timeLimitMinutes = this.$el.data('timeLimitMinutes');
            var timer = this.$el.data('timer');
            this.surveyTimerWidget = new publicWidget.registry.SurveyTimerWidget(this, {
                'timer': timer,
                'timeLimitMinutes': timeLimitMinutes
            });
            this.surveyTimerWidget.attachTo($timer);
            this.surveyTimerWidget.on('time_up', this, function () {
                if (self.currentScreen === 'question' && this.isScoredQuestion) {
                    self.$('.o_survey_session_navigation_next').click();
                }
            });
        }
    },

    /**
     * Refreshes the question results.
     *
     * What we get from this call:
     * - The 'question statistics' used to display the bar chart when appropriate
     * - The 'user input lines' that are used to display text/date/datetime answers on the screen
     * - The number of answers, useful for refreshing the progress bar
     */
    _refreshResults: function () {
        var self = this;

        return this._rpc({
            route: _.str.sprintf('/survey/session/results/%s', self.surveyAccessToken)
        }).then(function (questionResults) {
            if (questionResults) {
                self.attendeesCount = questionResults.attendees_count;

                if (self.resultsChart && questionResults.question_statistics_graph) {
                    self.resultsChart.updateChart(JSON.parse(questionResults.question_statistics_graph));
                } else if (self.textAnswers) {
                    self.textAnswers.updateTextAnswers(questionResults.input_line_values);
                }

                var max = self.attendeesCount > 0 ? self.attendeesCount : 1;
                var percentage = Math.min(Math.round((questionResults.answer_count / max) * 100), 100);
                self.$('.progress-bar').css('width', `${percentage}%`);

                if (self.attendeesCount && self.attendeesCount > 0) {
                    var answerCount = Math.min(questionResults.answer_count, self.attendeesCount);
                    self.$('.o_survey_session_answer_count').text(answerCount);
                    self.$('.progress-bar.o_survey_session_progress_small span').text(
                        `${answerCount} / ${self.attendeesCount}`
                    );
                }
            }

            return Promise.resolve();
        }, function () {
            // on failure, stop refreshing
            clearInterval(self.resultsRefreshInterval);
            delete self.resultsRefreshInterval;
        });
    },

    /**
     * We refresh the attendees count every 2 seconds while the user is on the start screen.
     *
     */
    _refreshAttendeesCount: function () {
        var self = this;

        return self._rpc({
            model: 'survey.survey',
            method: 'read',
            args: [[self.surveyId], ['session_answer_count']],
        }).then(function (result) {
            if (result && result.length === 1){
                self.$('.o_survey_session_attendees_count').text(
                    result[0].session_answer_count
                );
            }
        }, function () {
            // on failure, stop refreshing
            clearInterval(self.attendeesRefreshInterval);
        });
    },

    /**
     * For simple/multiple choice questions, we display a bar chart with:
     *
     * - answers of attendees
     * - correct / incorrect answers when relevant
     *
     * see SurveySessionChart widget doc for more information.
     *
     */
    _setupChart: function () {
        if (this.resultsChart) {
            this.resultsChart.setElement(null);
            this.resultsChart.destroy();
            delete this.resultsChart;
        }

        if (!this.isStartScreen && this.showBarChart) {
            this.resultsChart = new SurveySessionChart(this, {
                questionType: this.$el.data('questionType'),
                answersValidity: this.$el.data('answersValidity'),
                hasCorrectAnswers: this.hasCorrectAnswers,
                questionStatistics: this.$el.data('questionStatistics'),
                showInputs: this.showInputs
            });

            return this.resultsChart.attachTo(this.$('.o_survey_session_chart'));
        } else {
            return Promise.resolve();
        }
    },

    /**
     * Leaderboard of all the attendees based on their score.
     * see SurveySessionLeaderBoard widget doc for more information.
     *
     */
    _setupLeaderboard: function () {
        if (this.leaderBoard) {
            this.leaderBoard.setElement(null);
            this.leaderBoard.destroy();
            delete this.leaderBoard;
        }

        if (this.isScoredQuestion || this.isLastQuestion) {
            this.leaderBoard = new SurveySessionLeaderBoard(this, {
                surveyAccessToken: this.surveyAccessToken,
                sessionResults: this.$('.o_survey_session_results')
            });

            return this.leaderBoard.attachTo(this.$('.o_survey_session_leaderboard'));
        } else {
            return Promise.resolve();
        }
    },

    /**
     * Shows attendees answers for char_box/date and datetime questions.
     * see SurveySessionTextAnswers widget doc for more information.
     *
     */
    _setupTextAnswers: function () {
        if (this.textAnswers) {
            this.textAnswers.setElement(null);
            this.textAnswers.destroy();
            delete this.textAnswers;
        }

        if (!this.isStartScreen && this.showTextAnswers) {
            this.textAnswers = new SurveySessionTextAnswers(this, {
                questionType: this.$el.data('questionType')
            });

            return this.textAnswers.attachTo(this.$('.o_survey_session_text_answers_container'));
        } else {
            return Promise.resolve();
        }
    },

    /**
     * Setup the 2 refresh intervals of 2 seconds for our widget:
     * - The refresh of attendees count (only on the start screen)
     * - The refresh of results (used for chart/text answers/progress bar)
     */
    _setupIntervals: function () {
        this.attendeesCount = this.$el.data('attendeesCount') ? this.$el.data('attendeesCount') : 0;

        if (this.isStartScreen) {
            this.attendeesRefreshInterval = setInterval(this._refreshAttendeesCount.bind(this), 2000);
        } else {
            if (this.attendeesRefreshInterval) {
                clearInterval(this.attendeesRefreshInterval);
            }

            if (!this.resultsRefreshInterval) {
                this.resultsRefreshInterval = setInterval(this._refreshResults.bind(this), 2000);
            }
        }
    },

    /**
     * Setup current screen based on question properties.
     * If it's a non-scored question with a chart, we directly display the user inputs.
     */
    _setupCurrentScreen: function () {
        if (this.isStartScreen) {
            this.currentScreen = 'startScreen';
        } else if (!this.isScoredQuestion && this.showBarChart) {
            this.currentScreen = 'userInputs';
        } else {
            this.currentScreen = 'question';
        }

        this.$('.o_survey_session_navigation_previous').toggleClass('d-none', !!this.isFirstQuestion);

        this._setShowInputs(this.currentScreen === 'userInputs');
    },

    /**
     * When we go from the 'question' screen to the 'userInputs' screen, we toggle this boolean
     * and send the information to the chart.
     * The chart will show attendees survey.user_input.lines.
     *
     * @param {Boolean} showInputs
     */
    _setShowInputs(showInputs) {
        this.showInputs = showInputs;

        if (this.resultsChart) {
            this.resultsChart.setShowInputs(showInputs);
            this.resultsChart.updateChart();
        }
    },

    /**
     * When we go from the 'userInputs' screen to the 'results' screen, we toggle this boolean
     * and send the information to the chart.
     * The chart will show the question survey.question.answers.
     * (Only used for simple / multiple choice questions).
     *
     * @param {Boolean} showAnswers
     */
    _setShowAnswers(showAnswers) {
        this.showAnswers = showAnswers;

        if (this.resultsChart) {
            this.resultsChart.setShowAnswers(showAnswers);
            this.resultsChart.updateChart();
        }
    }
});

return publicWidget.registry.SurveySessionManage;

});

```

## File: static\src\js\survey_session_text_answers.js

```javascript
odoo.define('survey.session_text_answers', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var core = require('web.core');
var time = require('web.time');
var SESSION_CHART_COLORS = require('survey.session_colors');

var QWeb = core.qweb;

publicWidget.registry.SurveySessionTextAnswers = publicWidget.Widget.extend({
    xmlDependencies: ['/survey/static/src/xml/survey_session_text_answer_template.xml'],
    init: function (parent, options) {
        this._super.apply(this, arguments);

        this.answerIds = [];
        this.questionType = options.questionType;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Adds the attendees answers on the screen.
     * This is used for char_box/date and datetime questions.
     *
     * We use some tricks with jQuery for wow effect:
     * - force a width on the external div container, to reserve space for that answer
     * - set the actual width of the answer, and enable a css width animation
     * - set the opacity to 1, and enable a css opacity animation
     *
     * @param {Array} inputLineValues array of survey.user_input.line records in the form
     *   {id: line.id, value: line.[value_char_box/value_date/value_datetime]}
     */
    updateTextAnswers: function (inputLineValues) {
        var self = this;

        inputLineValues.forEach(function (inputLineValue) {
            if (!self.answerIds.includes(inputLineValue.id) && inputLineValue.value) {
                var textValue = inputLineValue.value;
                if (self.questionType === 'char_box') {
                    textValue = textValue.length > 25 ?
                        textValue.substring(0, 22) + '...' :
                        textValue;
                } else if (self.questionType === 'date') {
                    textValue = moment(textValue).format(time.getLangDateFormat());
                } else if (self.questionType === 'datetime') {
                    textValue = moment(textValue).format(time.getLangDatetimeFormat());
                }

                var $textAnswer = $(QWeb.render('survey.survey_session_text_answer', {
                    value: textValue,
                    borderColor: `rgb(${SESSION_CHART_COLORS[self.answerIds.length % 10]})`
                }));
                self.$el.append($textAnswer);
                var spanWidth = $textAnswer.find('span').width();
                var calculatedWidth = `calc(${spanWidth}px + 1.2rem)`;
                $textAnswer.css('width', calculatedWidth);
                setTimeout(function () {
                    // setTimeout to force jQuery rendering
                    $textAnswer.find('.o_survey_session_text_answer_container')
                        .css('width', calculatedWidth)
                        .css('opacity', '1');
                }, 1);
                self.answerIds.push(inputLineValue.id);
            }
        });
    },
});

return publicWidget.registry.SurveySessionTextAnswers;

});

```

## File: static\src\js\survey_timer.js

```javascript
odoo.define('survey.timer', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.SurveyTimerWidget = publicWidget.Widget.extend({
    //--------------------------------------------------------------------------
    // Widget
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        this.timer = params.timer;
        this.timeLimitMinutes = params.timeLimitMinutes;
        this.surveyTimerInterval = null;
        this.timeDifference = null;
        if (params.serverTime) {
            this.timeDifference = moment.utc().diff(moment.utc(params.serverTime), 'milliseconds');
        }
    },


    /**
    * Two responsabilities : Validate that time limit is not exceeded and Run timer otherwise.
    * If end-user's clock OR the system clock  is de-synchronized before the survey is started, we apply the
    * difference in timer (if time difference is more than 5 seconds) so that we can
    * display the 'absolute' counter
    *
    * @override
    */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.countDownDate = moment.utc(self.timer).add(self.timeLimitMinutes, 'minutes');
            if (Math.abs(self.timeDifference) >= 5000) {
                self.countDownDate = self.countDownDate.add(self.timeDifference, 'milliseconds');
            }
            if (self.timeLimitMinutes <= 0 || self.countDownDate.diff(moment.utc(), 'seconds') < 0) {
                self.trigger_up('time_up');
            } else {
                self._updateTimer();
                self.surveyTimerInterval = setInterval(self._updateTimer.bind(self), 1000);
            }
        });
    },

    // -------------------------------------------------------------------------
    // Private
    // -------------------------------------------------------------------------

    _formatTime: function (time) {
        return time > 9 ? time : '0' + time;
    },

    /**
    * This function is responsible for the visual update of the timer DOM every second.
    * When the time runs out, it triggers a 'time_up' event to notify the parent widget.
    *
    * We use a diff in millis and not a second, that we round to the nearest second.
    * Indeed, a difference of 999 millis is interpreted as 0 second by moment, which is problematic
    * for our use case.
    */
    _updateTimer: function () {
        var timeLeft = Math.round(this.countDownDate.diff(moment.utc(), 'milliseconds') / 1000);

        if (timeLeft >= 0) {
            var timeLeftMinutes = parseInt(timeLeft / 60);
            var timeLeftSeconds = timeLeft - (timeLeftMinutes * 60);
            this.$el.text(this._formatTime(timeLeftMinutes) + ':' + this._formatTime(timeLeftSeconds));
        } else {
            clearInterval(this.surveyTimerInterval);
            this.trigger_up('time_up');
        }
    },
});

return publicWidget.registry.SurveyTimerWidget;

});

```

## File: static\src\js\libs\chartjs-plugin-datalabels.min.js

```javascript
/*!
 * chartjs-plugin-datalabels v0.7.0
 * https://chartjs-plugin-datalabels.netlify.com
 * (c) 2019 Chart.js Contributors
 * Released under the MIT license
 */
!function(t,e){"object"==typeof exports&&"undefined"!=typeof module?module.exports=e(require("chart.js")):"function"==typeof define&&define.amd?define(["chart.js"],e):(t=t||self).ChartDataLabels=e(t.Chart)}(this,function(t){"use strict";var e=(t=t&&t.hasOwnProperty("default")?t.default:t).helpers,r=function(){if("undefined"!=typeof window){if(window.devicePixelRatio)return window.devicePixelRatio;var t=window.screen;if(t)return(t.deviceXDPI||1)/(t.logicalXDPI||1)}return 1}(),n={toTextLines:function(t){var r,n=[];for(t=[].concat(t);t.length;)"string"==typeof(r=t.pop())?n.unshift.apply(n,r.split("\n")):Array.isArray(r)?t.push.apply(t,r):e.isNullOrUndef(t)||n.unshift(""+r);return n},toFontString:function(t){return!t||e.isNullOrUndef(t.size)||e.isNullOrUndef(t.family)?null:(t.style?t.style+" ":"")+(t.weight?t.weight+" ":"")+t.size+"px "+t.family},textSize:function(t,e,r){var n,i=[].concat(e),o=i.length,a=t.font,l=0;for(t.font=r.string,n=0;n<o;++n)l=Math.max(t.measureText(i[n]).width,l);return t.font=a,{height:o*r.lineHeight,width:l}},parseFont:function(r){var i=t.defaults.global,o=e.valueOrDefault(r.size,i.defaultFontSize),a={family:e.valueOrDefault(r.family,i.defaultFontFamily),lineHeight:e.options.toLineHeight(r.lineHeight,o),size:o,style:e.valueOrDefault(r.style,i.defaultFontStyle),weight:e.valueOrDefault(r.weight,null),string:""};return a.string=n.toFontString(a),a},bound:function(t,e,r){return Math.max(t,Math.min(e,r))},arrayDiff:function(t,e){var r,n,i,o,a=t.slice(),l=[];for(r=0,i=e.length;r<i;++r)o=e[r],-1===(n=a.indexOf(o))?l.push([o,1]):a.splice(n,1);for(r=0,i=a.length;r<i;++r)l.push([a[r],-1]);return l},rasterize:function(t){return Math.round(t*r)/r}};function i(t,e){var r=e.x,n=e.y;if(null===r)return{x:0,y:-1};if(null===n)return{x:1,y:0};var i=t.x-r,o=t.y-n,a=Math.sqrt(i*i+o*o);return{x:a?i/a:0,y:a?o/a:-1}}var o=0,a=1,l=2,s=4,u=8;function f(t,e,r){var n=o;return t<r.left?n|=a:t>r.right&&(n|=l),e<r.top?n|=u:e>r.bottom&&(n|=s),n}function d(t,e){var r,n,i=e.anchor,o=t;return e.clamp&&(o=function(t,e){for(var r,n,i,o=t.x0,d=t.y0,c=t.x1,h=t.y1,x=f(o,d,e),y=f(c,h,e);x|y&&!(x&y);)(r=x||y)&u?(n=o+(c-o)*(e.top-d)/(h-d),i=e.top):r&s?(n=o+(c-o)*(e.bottom-d)/(h-d),i=e.bottom):r&l?(i=d+(h-d)*(e.right-o)/(c-o),n=e.right):r&a&&(i=d+(h-d)*(e.left-o)/(c-o),n=e.left),r===x?x=f(o=n,d=i,e):y=f(c=n,h=i,e);return{x0:o,x1:c,y0:d,y1:h}}(o,e.area)),"start"===i?(r=o.x0,n=o.y0):"end"===i?(r=o.x1,n=o.y1):(r=(o.x0+o.x1)/2,n=(o.y0+o.y1)/2),function(t,e,r,n,i){switch(i){case"center":r=n=0;break;case"bottom":r=0,n=1;break;case"right":r=1,n=0;break;case"left":r=-1,n=0;break;case"top":r=0,n=-1;break;case"start":r=-r,n=-n;break;case"end":break;default:i*=Math.PI/180,r=Math.cos(i),n=Math.sin(i)}return{x:t,y:e,vx:r,vy:n}}(r,n,t.vx,t.vy,e.align)}var c={arc:function(t,e){var r=(t.startAngle+t.endAngle)/2,n=Math.cos(r),i=Math.sin(r),o=t.innerRadius,a=t.outerRadius;return d({x0:t.x+n*o,y0:t.y+i*o,x1:t.x+n*a,y1:t.y+i*a,vx:n,vy:i},e)},point:function(t,e){var r=i(t,e.origin),n=r.x*t.radius,o=r.y*t.radius;return d({x0:t.x-n,y0:t.y-o,x1:t.x+n,y1:t.y+o,vx:r.x,vy:r.y},e)},rect:function(t,e){var r=i(t,e.origin),n=t.x,o=t.y,a=0,l=0;return t.horizontal?(n=Math.min(t.x,t.base),a=Math.abs(t.base-t.x)):(o=Math.min(t.y,t.base),l=Math.abs(t.base-t.y)),d({x0:n,y0:o+l,x1:n+a,y1:o,vx:r.x,vy:r.y},e)},fallback:function(t,e){var r=i(t,e.origin);return d({x0:t.x,y0:t.y,x1:t.x,y1:t.y,vx:r.x,vy:r.y},e)}},h=t.helpers,x=n.rasterize;function y(t){var e=t._model.horizontal,r=t._scale||e&&t._xScale||t._yScale;if(!r)return null;if(void 0!==r.xCenter&&void 0!==r.yCenter)return{x:r.xCenter,y:r.yCenter};var n=r.getBasePixel();return e?{x:n,y:null}:{x:null,y:n}}function v(t,e,r){var n=t.shadowBlur,i=r.stroked,o=x(r.x),a=x(r.y),l=x(r.w);i&&t.strokeText(e,o,a,l),r.filled&&(n&&i&&(t.shadowBlur=0),t.fillText(e,o,a,l),n&&i&&(t.shadowBlur=n))}var _=function(t,e,r,n){var i=this;i._config=t,i._index=n,i._model=null,i._rects=null,i._ctx=e,i._el=r};h.extend(_.prototype,{_modelize:function(e,r,i,o){var a,l=this._index,s=h.options.resolve,u=n.parseFont(s([i.font,{}],o,l)),f=s([i.color,t.defaults.global.defaultFontColor],o,l);return{align:s([i.align,"center"],o,l),anchor:s([i.anchor,"center"],o,l),area:o.chart.chartArea,backgroundColor:s([i.backgroundColor,null],o,l),borderColor:s([i.borderColor,null],o,l),borderRadius:s([i.borderRadius,0],o,l),borderWidth:s([i.borderWidth,0],o,l),clamp:s([i.clamp,!1],o,l),clip:s([i.clip,!1],o,l),color:f,display:e,font:u,lines:r,offset:s([i.offset,0],o,l),opacity:s([i.opacity,1],o,l),origin:y(this._el),padding:h.options.toPadding(s([i.padding,0],o,l)),positioner:(a=this._el,a instanceof t.elements.Arc?c.arc:a instanceof t.elements.Point?c.point:a instanceof t.elements.Rectangle?c.rect:c.fallback),rotation:s([i.rotation,0],o,l)*(Math.PI/180),size:n.textSize(this._ctx,r,u),textAlign:s([i.textAlign,"start"],o,l),textShadowBlur:s([i.textShadowBlur,0],o,l),textShadowColor:s([i.textShadowColor,f],o,l),textStrokeColor:s([i.textStrokeColor,f],o,l),textStrokeWidth:s([i.textStrokeWidth,0],o,l)}},update:function(t){var e,r,i,o=this,a=null,l=null,s=o._index,u=o._config,f=h.options.resolve([u.display,!0],t,s);f&&(e=t.dataset.data[s],r=h.valueOrDefault(h.callback(u.formatter,[e,t]),e),(i=h.isNullOrUndef(r)?[]:n.toTextLines(r)).length&&(l=function(t){var e=t.borderWidth||0,r=t.padding,n=t.size.height,i=t.size.width,o=-i/2,a=-n/2;return{frame:{x:o-r.left-e,y:a-r.top-e,w:i+r.width+2*e,h:n+r.height+2*e},text:{x:o,y:a,w:i,h:n}}}(a=o._modelize(f,i,u,t)))),o._model=a,o._rects=l},geometry:function(){return this._rects?this._rects.frame:{}},rotation:function(){return this._model?this._model.rotation:0},visible:function(){return this._model&&this._model.opacity},model:function(){return this._model},draw:function(t,e){var r,i=t.ctx,o=this._model,a=this._rects;this.visible()&&(i.save(),o.clip&&(r=o.area,i.beginPath(),i.rect(r.left,r.top,r.right-r.left,r.bottom-r.top),i.clip()),i.globalAlpha=n.bound(0,o.opacity,1),i.translate(x(e.x),x(e.y)),i.rotate(o.rotation),function(t,e,r){var n=r.backgroundColor,i=r.borderColor,o=r.borderWidth;(n||i&&o)&&(t.beginPath(),h.canvas.roundedRect(t,x(e.x)+o/2,x(e.y)+o/2,x(e.w)-o,x(e.h)-o,r.borderRadius),t.closePath(),n&&(t.fillStyle=n,t.fill()),i&&o&&(t.strokeStyle=i,t.lineWidth=o,t.lineJoin="miter",t.stroke()))}(i,a.frame,o),function(t,e,r,n){var i,o=n.textAlign,a=n.color,l=!!a,s=n.font,u=e.length,f=n.textStrokeColor,d=n.textStrokeWidth,c=f&&d;if(u&&(l||c))for(r=function(t,e,r){var n=r.lineHeight,i=t.w,o=t.x;return"center"===e?o+=i/2:"end"!==e&&"right"!==e||(o+=i),{h:n,w:i,x:o,y:t.y+n/2}}(r,o,s),t.font=s.string,t.textAlign=o,t.textBaseline="middle",t.shadowBlur=n.textShadowBlur,t.shadowColor=n.textShadowColor,l&&(t.fillStyle=a),c&&(t.lineJoin="round",t.lineWidth=d,t.strokeStyle=f),i=0,u=e.length;i<u;++i)v(t,e[i],{stroked:c,filled:l,w:r.w,x:r.x,y:r.y+r.h*i})}(i,o.lines,a.text,o),i.restore())}});var b=t.helpers,p=Number.MIN_SAFE_INTEGER||-9007199254740991,g=Number.MAX_SAFE_INTEGER||9007199254740991;function m(t,e,r){var n=Math.cos(r),i=Math.sin(r),o=e.x,a=e.y;return{x:o+n*(t.x-o)-i*(t.y-a),y:a+i*(t.x-o)+n*(t.y-a)}}function w(t,e){var r,n,i,o,a,l=g,s=p,u=e.origin;for(r=0;r<t.length;++r)i=(n=t[r]).x-u.x,o=n.y-u.y,a=e.vx*i+e.vy*o,l=Math.min(l,a),s=Math.max(s,a);return{min:l,max:s}}function k(t,e){var r=e.x-t.x,n=e.y-t.y,i=Math.sqrt(r*r+n*n);return{vx:(e.x-t.x)/i,vy:(e.y-t.y)/i,origin:t,ln:i}}var M=function(){this._rotation=0,this._rect={x:0,y:0,w:0,h:0}};function S(t,e,r){var n=e.positioner(t,e),i=n.vx,o=n.vy;if(!i&&!o)return{x:n.x,y:n.y};var a=r.w,l=r.h,s=e.rotation,u=Math.abs(a/2*Math.cos(s))+Math.abs(l/2*Math.sin(s)),f=Math.abs(a/2*Math.sin(s))+Math.abs(l/2*Math.cos(s)),d=1/Math.max(Math.abs(i),Math.abs(o));return u*=i*d,f*=o*d,u+=e.offset*i,f+=e.offset*o,{x:n.x+u,y:n.y+f}}b.extend(M.prototype,{center:function(){var t=this._rect;return{x:t.x+t.w/2,y:t.y+t.h/2}},update:function(t,e,r){this._rotation=r,this._rect={x:e.x+t.x,y:e.y+t.y,w:e.w,h:e.h}},contains:function(t){var e=this._rect;return!((t=m(t,this.center(),-this._rotation)).x<e.x-1||t.y<e.y-1||t.x>e.x+e.w+2||t.y>e.y+e.h+2)},intersects:function(t){var e,r,n,i=this._points(),o=t._points(),a=[k(i[0],i[1]),k(i[0],i[3])];for(this._rotation!==t._rotation&&a.push(k(o[0],o[1]),k(o[0],o[3])),e=0;e<a.length;++e)if(r=w(i,a[e]),n=w(o,a[e]),r.max<n.min||n.max<r.min)return!1;return!0},_points:function(){var t=this._rect,e=this._rotation,r=this.center();return[m({x:t.x,y:t.y},r,e),m({x:t.x+t.w,y:t.y},r,e),m({x:t.x+t.w,y:t.y+t.h},r,e),m({x:t.x,y:t.y+t.h},r,e)]}});var C={prepare:function(t){var e,r,n,i,o,a=[];for(e=0,n=t.length;e<n;++e)for(r=0,i=t[e].length;r<i;++r)o=t[e][r],a.push(o),o.$layout={_box:new M,_hidable:!1,_visible:!0,_set:e,_idx:r};return a.sort(function(t,e){var r=t.$layout,n=e.$layout;return r._idx===n._idx?n._set-r._set:n._idx-r._idx}),this.update(a),a},update:function(t){var e,r,n,i,o,a=!1;for(e=0,r=t.length;e<r;++e)i=(n=t[e]).model(),(o=n.$layout)._hidable=i&&"auto"===i.display,o._visible=n.visible(),a|=o._hidable;a&&function(t){var e,r,n,i,o,a;for(e=0,r=t.length;e<r;++e)(i=(n=t[e]).$layout)._visible&&(o=n.geometry(),a=S(n._el._model,n.model(),o),i._box.update(a,o,n.rotation()));(function(t,e){var r,n,i,o;for(r=t.length-1;r>=0;--r)for(i=t[r].$layout,n=r-1;n>=0&&i._visible;--n)(o=t[n].$layout)._visible&&i._box.intersects(o._box)&&e(i,o)})(t,function(t,e){var r=t._hidable,n=e._hidable;r&&n||n?e._visible=!1:r&&(t._visible=!1)})}(t)},lookup:function(t,e){var r,n;for(r=t.length-1;r>=0;--r)if((n=t[r].$layout)&&n._visible&&n._box.contains(e))return t[r];return null},draw:function(t,e){var r,n,i,o,a,l;for(r=0,n=e.length;r<n;++r)(o=(i=e[r]).$layout)._visible&&(a=i.geometry(),l=S(i._el._view,i.model(),a),o._box.update(l,a,i.rotation()),i.draw(t,l))}},z=t.helpers,$={align:"center",anchor:"center",backgroundColor:null,borderColor:null,borderRadius:0,borderWidth:0,clamp:!1,clip:!1,color:void 0,display:!0,font:{family:void 0,lineHeight:1.2,size:void 0,style:void 0,weight:null},formatter:function(t){if(z.isNullOrUndef(t))return null;var e,r,n,i=t;if(z.isObject(t))if(z.isNullOrUndef(t.label))if(z.isNullOrUndef(t.r))for(i="",n=0,r=(e=Object.keys(t)).length;n<r;++n)i+=(0!==n?", ":"")+e[n]+": "+t[e[n]];else i=t.r;else i=t.label;return""+i},labels:void 0,listeners:{},offset:4,opacity:1,padding:{top:4,right:4,bottom:4,left:4},rotation:0,textAlign:"start",textStrokeColor:void 0,textStrokeWidth:0,textShadowBlur:0,textShadowColor:void 0},O=t.helpers,A="$datalabels",D="$default";function P(t,e,r){if(e){var n,i=r.$context,o=r.$groups;e[o._set]&&(n=e[o._set][o._key])&&!0===O.callback(n,[i])&&(t[A]._dirty=!0,r.update(i))}}function N(t,e){var r,n,i=t[A],o=i._listeners;if(o.enter||o.leave){if("mousemove"===e.type)n=C.lookup(i._labels,e);else if("mouseout"!==e.type)return;r=i._hovered,i._hovered=n,function(t,e,r,n){var i,o;(r||n)&&(r?n?r!==n&&(o=i=!0):o=!0:i=!0,o&&P(t,e.leave,r),i&&P(t,e.enter,n))}(t,o,r,n)}}t.defaults.global.plugins.datalabels=$;var R={id:"datalabels",beforeInit:function(t){t[A]={_actives:[]}},beforeUpdate:function(t){var e=t[A];e._listened=!1,e._listeners={},e._datasets=[],e._labels=[]},afterDatasetUpdate:function(t,e,r){var n,i,o,a,l,s,u,f,d=e.index,c=t[A],h=c._datasets[d]=[],x=t.isDatasetVisible(d),y=t.data.datasets[d],v=function(t,e){var r,n,i,o=t.datalabels,a=[];return!1===o?null:(!0===o&&(o={}),e=O.merge({},[e,o]),n=e.labels||{},i=Object.keys(n),delete e.labels,i.length?i.forEach(function(t){n[t]&&a.push(O.merge({},[e,n[t],{_key:t}]))}):a.push(e),r=a.reduce(function(t,e){return O.each(e.listeners||{},function(r,n){t[n]=t[n]||{},t[n][e._key||D]=r}),delete e.listeners,t},{}),{labels:a,listeners:r})}(y,r),b=e.meta.data||[],p=t.ctx;for(p.save(),n=0,o=b.length;n<o;++n)if((u=b[n])[A]=[],x&&u&&!u.hidden&&!u._model.skip)for(i=0,a=v.labels.length;i<a;++i)s=(l=v.labels[i])._key,(f=new _(l,p,u,n)).$groups={_set:d,_key:s||D},f.$context={active:!1,chart:t,dataIndex:n,dataset:y,datasetIndex:d},f.update(f.$context),u[A].push(f),h.push(f);p.restore(),O.merge(c._listeners,v.listeners,{merger:function(t,r,n){r[t]=r[t]||{},r[t][e.index]=n[t],c._listened=!0}})},afterUpdate:function(t,e){t[A]._labels=C.prepare(t[A]._datasets,e)},afterDatasetsDraw:function(t){C.draw(t,t[A]._labels)},beforeEvent:function(t,e){if(t[A]._listened)switch(e.type){case"mousemove":case"mouseout":N(t,e);break;case"click":!function(t,e){var r=t[A],n=r._listeners.click,i=n&&C.lookup(r._labels,e);i&&P(t,n,i)}(t,e)}},afterEvent:function(e){var r,i,o,a,l,s,u,f=e[A],d=f._actives,c=f._actives=e.lastActive||[],h=n.arrayDiff(d,c);for(r=0,i=h.length;r<i;++r)if((l=h[r])[1])for(o=0,a=(u=l[0][A]||[]).length;o<a;++o)(s=u[o]).$context.active=1===l[1],s.update(s.$context);(f._dirty||h.length)&&(C.update(f._labels),function(e){if(!e.animating){for(var r=t.animationService.animations,n=0,i=r.length;n<i;++n)if(r[n].chart===e)return;e.render({duration:1,lazy:!0})}}(e)),delete f._dirty}};return t.plugins.register(R),R});

```

## File: static\src\xml\survey_breadcrumb_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="survey.survey_breadcrumb_template">
    <ol class="breadcrumb justify-content-end bg-transparent">
        <t t-set="canGoBack" t-value="widget.canGoBack"/>
        <t t-foreach="widget.pages" t-as="page">
            <t t-set="isActivePage" t-value="page.id === widget.currentPageId"/>
            <li t-att-class="'breadcrumb-item' + (isActivePage ? ' active font-weight-bold' : '')"
                t-att-data-page-id="page.id"
                t-att-data-page-title="page.title">
                <t t-if="widget.currentPageId === page.id">
                    <!-- Users can only go back and not forward -->
                    <!-- As soon as we reach the current page, set "can_go_back" to False -->
                    <t t-set="canGoBack" t-value="false" />
                </t>
                <t t-if="canGoBack">
                    <a class="text-primary text-break" href="#">
                        <span t-esc="page.title" />
                    </a>
                </t>
                <t t-else="">
                    <span t-att-class="'text-break ' + (isActivePage ? 'text-black' : 'text-muted')"
                          t-esc="page.title" />
                </t>
            </li>
        </t>
    </ol>
</t>

</templates>

```

## File: static\src\xml\survey_session_text_answer_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<div t-name="survey.survey_session_text_answer" class="o_survey_session_text_answer d-inline-block m-1">
    <div class="o_survey_session_text_answer_container d-inline-block p-2 font-weight-bold"
        t-attf-style="border-color: #{borderColor}">
        <span class="d-inline-block" t-esc="value" />
    </div>
</div>

</templates>

```

## File: views\gamification_badge_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="gamification_badge_form_view_simplified" model="ir.ui.view">
        <field name="name">gamification.badge.form.view.simplified</field>
        <field name="model">gamification.badge</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <form string="Badge">
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Problem Solver"/>
                        </h1>
                    </div>
                    <group>
                        <field name="description" nolabel="1" placeholder="e.g. No one can solve challenges like you do"/>
                    </group>
                    <group string="Rewards for challenges">
                        <field name="level"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
</data></odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="res_partner_action_certifications" model="ir.actions.act_window">
        <field name="name">Certifications Succeeded</field>
        <field name="res_model">survey.user_input</field>
        <field name="view_mode">tree</field>
        <field name="context">{'search_default_scoring_success': 1}</field>
    </record>

    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.survey</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="groups_id" eval="[(4, ref('survey.group_survey_user'))]"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button class="oe_stat_button" type="object"
                    icon="fa-trophy" name="action_view_certifications"
                    attrs="{'invisible': ['|', ('certifications_count', '=', 0), ('is_company', '=', True)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="certifications_count" /></span>
                        <span class="o_stat_text" attrs="{'invisible': [('certifications_count', '&lt;', 2)]}">Certifications</span>
                        <span class="o_stat_text" attrs="{'invisible': [('certifications_count', '&gt;', 1)]}">Certification</span>
                    </div>
                </button>
                <button class="oe_stat_button" type="object"
                    icon="fa-trophy" name="action_view_certifications"
                    attrs="{'invisible': ['|', ('certifications_company_count', '=', 0), ('is_company', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="certifications_company_count" /></span>
                        <span class="o_stat_text" attrs="{'invisible': [('certifications_company_count', '&lt;', 2)]}">Certifications</span>
                        <span class="o_stat_text" attrs="{'invisible': [('certifications_company_count', '&gt;', 1)]}">Certification</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\survey_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <data>
    <!-- Main menu -->
    <menuitem name="Surveys"
      id="menu_surveys"
      sequence="130"
      groups="group_survey_user"
      web_icon="survey,static/description/icon.png"/>

      <!-- Parent menus -->
    <menuitem name="Questions"
      id="survey_menu_questions"
      parent="menu_surveys"
      groups="base.group_no_one"
      sequence="80"/>
    <menuitem name="Participations"
      id="survey_menu_user_inputs"
      parent="menu_surveys"
      sequence="90"/>
  </data>
</odoo>

```

## File: views\survey_question_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- QUESTIONS -->
    <record model="ir.ui.view" id="survey_question_form">
        <field name="name">Form view for survey question</field>
        <field name="model">survey.question</field>
        <field name="arch" type="xml">
            <form string="Survey Question" create="false">
                <field name="is_page" invisible="1"/>
                <field name="page_id" invisible="1" required="False"/>
                <field name="survey_id" invisible="1"/>
                <field name="sequence" invisible="1"/>
                <field name="scoring_type" invisible="1"/>
                <sheet>
                    <div class="oe_title" style="width: 100%;">
                        <label for="title" string="Section" attrs="{'invisible': [('is_page', '=', False)]}"/>
                        <label for="title" string="Question" attrs="{'invisible': [('is_page', '=', True)]}"/>
                        <separator />
                        <field name="title" colspan="4"/>
                        <separator />
                        <field name="questions_selection" invisible="1"/>
                    </div>
                    <group class="o_label_nowrap" attrs="{'invisible': ['|', ('is_page', '=', False), ('questions_selection', '=', 'all')]}">
                        <field name="random_questions_count"/>
                    </group>
                    <group attrs="{'invisible': [('is_page', '=', True)]}">
                        <group>
                            <field name="question_type" widget="radio" attrs="{'required': [('is_page', '=', False)]}" />
                        </group>
                        <group>
                            <div class="col-lg-6 offset-lg-3 o_preview_questions">
                                <!-- Multiple Lines Text Zone -->
                                <div attrs="{'invisible': [('question_type', '!=', 'text_box')]}">
                                        <i class="fa fa-align-justify fa-4x" role="img" aria-label="Multiple lines" title="Multiple Lines"/>
                                </div>
                                <!-- Single Line Text Zone -->
                                <div attrs="{'invisible': [('question_type', '!=', 'char_box')]}">
                                    <i class="fa fa-minus fa-4x" role="img" aria-label="Single Line" title="Single Line"/>
                                </div>
                                <!-- Numerical Value -->
                                <div attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}">
                                    <i class="fa fa-2x" role="img" aria-label="Numeric" title="Numeric">123..</i>
                                </div>
                                <!-- Date -->
                                <div attrs="{'invisible': [('question_type', '!=', 'date')]}">
                                    <p class="o_datetime">YYYY-MM-DD
                                        <i class="fa fa-calendar fa-2x" role="img" aria-label="Calendar" title="Calendar"/>
                                    </p>
                                </div>
                                <!-- Date and Time -->
                                <div attrs="{'invisible': [('question_type', '!=', 'datetime')]}">
                                    <p class="o_datetime">YYYY-MM-DD hh:mm:ss
                                        <i class="fa fa-calendar fa-2x" role="img" aria-label="Calendar" title="Calendar"/>
                                    </p>
                                </div>
                                <!-- Multiple choice: only one answer -->
                                <div attrs="{'invisible': [('question_type', '!=', 'simple_choice')]}" role="img" aria-label="Multiple choice with one answer" title="Multiple choice with one answer">
                                    <div class="row"><i class="fa fa-circle-o  fa-lg"/> answer</div>
                                    <div class="row"><i class="fa fa-dot-circle-o fa-lg"/> answer</div>
                                    <div class="row"><i class="fa fa-circle-o  fa-lg"/> answer</div>
                                </div>
                                <!-- Multiple choice: multiple answers allowed -->
                                <div attrs="{'invisible': [('question_type', '!=', 'multiple_choice')]}" role="img" aria-label="Multiple choice with multiple answers" title="Multiple choice with multiple answers">
                                    <div class="row"><i class="fa fa-square-o fa-lg"/> answer</div>
                                    <div class="row"><i class="fa fa-check-square-o fa-lg"/> answer</div>
                                    <div class="row"><i class="fa fa-square-o fa-lg"/> answer</div>
                                </div>
                                <!-- Matrix -->
                                <div attrs="{'invisible': [('question_type', '!=', 'matrix')]}">
                                    <div class="row o_matrix_head">
                                        <div class="col-3"></div>
                                        <div class="col-3">ans</div>
                                        <div class="col-3">ans</div>
                                        <div class="col-3">ans</div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-3">Row1</div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-3">Row2</div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-3">Row3</div>
                                        <div class="col-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                    </div>
                                </div>
                            </div>
                        </group>
                    </group>
                    <notebook>
                        <page string="Answers" name="answers" attrs="{'invisible': [('is_page', '=', True)]}">
                            <div class="row">
                                <div class="col-6"
                                    attrs="{'invisible': [('question_type', 'not in', ['char_box', 'numerical_box', 'date', 'datetime'])]}">
                                    <div attrs="{'invisible': [('question_type', '!=', 'char_box')]}">
                                        <field name="validation_email" nolabel="1"/>
                                        <label for="validation_email"/><br />

                                        <field name="save_as_email" nolabel="1" attrs="{'invisible': [('validation_email', '=', False)]}"/>
                                        <label for="save_as_email" attrs="{'invisible': [('validation_email', '=', False)]}"/>
                                        <br attrs="{'invisible': [('validation_email', '=', False)]}"/>

                                        <field name="save_as_nickname" nolabel="1"/>
                                        <label for="save_as_nickname"/>
                                    </div>
                                    <div>
                                        <field name="validation_required" nolabel="1"/>
                                        <label for="validation_required"/>
                                        <group attrs="{'invisible': [('validation_required', '=', False)]}">
                                            <field name="validation_length_min" attrs="{'invisible': [('question_type', '!=', 'char_box')]}"/>
                                            <field name="validation_length_max" attrs="{'invisible': [('question_type', '!=', 'char_box')]}"/>
                                            <field name="validation_min_float_value" attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}"/>
                                            <field name="validation_max_float_value" attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}"/>
                                            <field name="validation_min_date" attrs="{'invisible': [('question_type', '!=', 'date')]}"/>
                                            <field name="validation_max_date" attrs="{'invisible': [('question_type', '!=', 'date')]}"/>
                                            <field name="validation_min_datetime" widget="datetime" attrs="{'invisible': [('question_type', '!=', 'datetime')]}"/>
                                            <field name="validation_max_datetime" widget="datetime" attrs="{'invisible': [('question_type', '!=', 'datetime')]}"/>
                                            <field name="validation_error_msg" attrs="{'required': [('validation_required', '=', True)]}"/>
                                        </group>
                                    </div>
                                </div>

                                <div class="col-6" attrs="{'invisible': [
                                        '|', ('scoring_type', '=', 'no_scoring'),
                                        ('question_type', 'not in', ['numerical_box', 'date', 'datetime'])]
                                    }">
                                    <group>
                                        <field name="answer_numerical_box" string="Correct Answer" attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}"/>
                                        <field name="answer_date" string="Correct Answer" attrs="{'invisible': [('question_type', '!=', 'date')]}"/>
                                        <field name="answer_datetime" string="Correct Answer" attrs="{'invisible': [('question_type', '!=', 'datetime')]}"/>
                                        <field name="is_scored_question"/>
                                        <field name="answer_score"/>
                                    </group>
                                </div>
                            </div>
                            <group attrs="{'invisible': [('question_type', 'not in', ['simple_choice', 'multiple_choice', 'matrix'])]}">
                                <field name="suggested_answer_ids" string="Answers" context="{'default_question_id': active_id}">
                                    <tree editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="value" string="Choices"/>
                                        <field name="value_image" options="{'accepted_file_extensions': 'image/*'}"
                                            attrs="{'column_invisible': ['|', ('parent.allow_value_image', '=', False), ('parent.question_type', '=', 'matrix')]}"/>
                                        <field name="is_correct" attrs="{'column_invisible': ['|', ('parent.scoring_type', '=', 'no_scoring'), ('parent.question_type', '=', 'matrix')]}"/>
                                        <field name="answer_score" attrs="{'column_invisible': ['|', ('parent.scoring_type', '=', 'no_scoring'), ('parent.question_type', '=', 'matrix')]}"/>
                                    </tree>
                                </field>
                            </group>
                            <group attrs="{'invisible': [('question_type', '!=', 'matrix')]}">
                                <field name="matrix_row_ids" context="{'default_matrix_question_id': active_id}" attrs="{'invisible': [('question_type', '!=', 'matrix')]}">
                                    <tree editable="bottom">
                                        <field name="sequence" widget="handle"/>
                                        <field name="value" string="Rows"/>
                                    </tree>
                                </field>
                            </group>
                        </page>
                        <page string="Description" name="survey_description">
                            <field name="description" widget="html"/>
                        </page>
                        <page string="Options" name="options" attrs="{'invisible': [('is_page', '=', True)]}">
                            <group string="Constraints">
                                <group>
                                    <field name="constr_mandatory" string="Mandatory Answer"/>
                                    <field name="constr_error_msg" attrs="{'invisible': [('constr_mandatory', '=', False)]}"/>
                                    <field name="matrix_subtype" attrs="{'invisible':[('question_type','not in',['matrix'])],'required':[('question_type','=','matrix')]}"/>
                                </group>
                            </group>
                            <group>
                                <group string="Display" attrs="{'invisible':[('question_type','not in',['simple_choice', 'multiple_choice'])]}">
                                    <field name="column_nb" string="Number of columns" invisible="1"/>
                                    <field name="allow_value_image"/>
                                </group>
                                <group string="Conditional Display" attrs="{'invisible': [('questions_selection', '=', 'random')]}">
                                    <field name="is_conditional"/>
                                    <field name="triggering_question_id"  options="{'no_open': True, 'no_create': True}"
                                           attrs="{'invisible': [('is_conditional','=', False)], 'required': [('is_conditional','=', True)]}"/>
                                    <field name="triggering_answer_id" options="{'no_open': True, 'no_create': True}"
                                           attrs="{'invisible': ['|', ('is_conditional','=', False), ('triggering_question_id','=', False)],
                                                   'required': [('is_conditional','=', True)]}"/>
                                </group>
                            </group>
                            <group string="Allow Comments" attrs="{'invisible':[('question_type','not in',['simple_choice','multiple_choice', 'matrix'])]}">
                                <field name='comments_allowed' />
                                <field name='comments_message' attrs="{'invisible': [('comments_allowed', '=', False)]}"/>
                                <field name='comment_count_as_answer' attrs="{'invisible': ['|', ('comments_allowed', '=', False), ('question_type', 'in', ['matrix'])]}" />
                            </group>
                            <group string="Live Sessions">
                                <label for="is_time_limited" string="Question Time Limit"/>
                                <div>
                                    <field name="is_time_limited" nolabel="1"/>
                                    <field name="time_limit" nolabel="1" class="oe_inline"
                                        attrs="{'invisible': [('is_time_limited', '=', False)]}" />
                                    <span attrs="{'invisible': [('is_time_limited', '=', False)]}"> seconds</span>
                                </div>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record model="ir.ui.view" id="survey_question_tree">
        <field name="name">Tree view for survey question</field>
        <field name="model">survey.question</field>
        <field name="arch" type="xml">
            <tree string="Survey Question" create="false">
                <field name="sequence" widget="handle"/>
                <field name="title"/>
                <field name="survey_id"/>
                <field name="question_type"/>
                <field name="triggering_question_id" invisible="1"/>
                <button disabled="disabled" icon="fa-code-fork" attrs="{'invisible': [('triggering_question_id', '=', False)]}"
                    title="This question depends on another question's answer." class="icon_rotates"/>
            </tree>
        </field>
    </record>
    <record model="ir.ui.view" id="survey_question_search">
        <field name="name">Search view for survey question</field>
        <field name="model">survey.question</field>
        <field name="arch" type="xml">
            <search string="Search Question">
                <field name="title"/>
                <field name="survey_id" string="Survey"/>
                <field name="question_type" string="Type"/>
                <group expand="1" string="Group By">
                    <filter name="group_by_type" string="Type" domain="[]" context="{'group_by':'question_type'}"/>
                    <filter name="group_by_survey" string="Survey" domain="[]" context="{'group_by':'survey_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_survey_question_form">
        <field name="name">Questions</field>
        <field name="res_model">survey.question</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="survey_question_search"/>
        <field name="context">{'search_default_group_by_page': True}</field>
        <field name="domain">[('is_page', '=', False)]</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No questions found
          </p>
        </field>
    </record>

    <!-- LABELS -->
    <record id="survey_question_answer_view_tree" model="ir.ui.view">
        <field name="name">survey.question.answer.view.tree</field>
        <field name="model">survey.question.answer</field>
        <field name="arch" type="xml">
            <tree string="Survey Label" create="false">
                <field name="sequence" widget="handle"/>
                <field name="question_id"/>
                <field name="matrix_question_id"/>
                <field name="value"/>
                <field name="answer_score" groups="base.group_no_one"/>
            </tree>
        </field>
    </record>
    <record id="survey_question_answer_view_search" model="ir.ui.view">
        <field name="name">survey.question.answer.view.search</field>
        <field name="model">survey.question.answer</field>
        <field name="arch" type="xml">
            <search string="Search Label">
                <field name="question_id"/>
                <group expand="1" string="Group By">
                    <filter name="group_by_question" string="Question" domain="[]" context="{'group_by':'question_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="survey_question_answer_action" model="ir.actions.act_window">
        <field name="name">Suggested Values</field>
        <field name="res_model">survey.question.answer</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="survey_question_answer_view_search"/>
        <field name="context">{'search_default_group_by_question': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No survey labels found
          </p>
        </field>
    </record>

    <menuitem name="Questions"
        id="menu_survey_question_form1"
        action="action_survey_question_form"
        parent="survey_menu_questions"
        sequence="2"/>
    <menuitem name="Suggested Values"
        id="menu_survey_label_form1"
        action="survey_question_answer_action"
        parent="survey_menu_questions"
        sequence="3"/>
</data>
</odoo>

```

## File: views\survey_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Paper Format -->
        <record id="paperformat_survey_certification" model="report.paperformat">
            <field name="name">Survey Certification</field>
            <field name="default" eval="True"/>
            <field name="format">A4</field>
            <field name="orientation">Landscape</field>
            <field name="margin_top">0</field>
            <field name="margin_bottom">0</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
            <field name="header_line" eval="False"/>
            <field name="header_spacing">0</field>
            <field name="disable_shrinking" eval="True"/>
            <field name="dpi">96</field>
        </record>
        <!-- QWeb Reports -->
        <record id="certification_report" model="ir.actions.report">
            <field name="name">Certifications</field>
            <field name="model">survey.user_input</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">survey.certification_report_view</field>
            <field name="report_file">survey.certification_report_view</field>
            <field name="print_report_name">'Certification - %s' % (object.survey_id.display_name)</field>
            <field name="attachment">'certification.pdf'</field>
            <field name="binding_model_id" ref="model_survey_user_input"/>
            <field name="binding_type">report</field>
            <field name="paperformat_id" ref="paperformat_survey_certification"/>
        </record>
    </data>
</odoo>

```

## File: views\survey_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="certification_report_view_modern">
            <!-- Style classes to be applyed to '#o_survey_certification': [no class](purple), gold, blue  -->
            <div id="o_survey_certification" t-att-data-oe-model="user_input._name" t-att-data-oe-id="user_input.id" t-att-class="'article certification-wrapper modern %s' % layout_color">
                <div class="certification">
                    <div class="certification-seal" t-if="user_input.scoring_success"/>
                    <div class="certification-top">
                        <h1><b>Certificate</b>
                            <br/><span t-if="user_input.scoring_success">of achievement</span>
                        </h1>
                    </div>

                    <div class="certification-content">
                        <div t-if="user_input.scoring_success">
                            <p>This certificate is presented to
                                <br/>
                                <t t-set="certif_style" t-value="''"/>
                                <t t-set="certified_name" t-value="user_input.partner_id.name or user_input.email or ''"/>
                                <t t-if="certified_name.isupper()">
                                    <t t-set="certif_style" t-value="certif_style + 'font-family: certification-serif;'"/>
                                </t>
                                <t t-if="len(certified_name) > 20">
                                    <t t-set="certif_style" t-value="certif_style + 'font-size: 40px; line-height: 4;'"/>
                                </t>
                                <span t-att-style="certif_style" class="user-name" t-esc="certified_name"/>

                                <br/>by <span class="certification-company" t-field="user_input.env.company.display_name"/> for successfully completing
                                <br/><b><span class="certification-name" t-field="user_input.survey_id.display_name"/></b>
                             </p>
                        </div>
                        <div t-else="" class="certification-failed">
                            <p>Certification Failed</p>
                        </div>
                    </div>

                    <div class="certification-bottom">
                        <div class="certification-date-wrapper">
                            <div class="certification-date" t-field="user_input.create_date" t-options='{"widget": "date"}'/>
                            <span>Date</span>
                        </div>
                        <div class="certification-company">
                            <span class="certification-company-logo" t-field="user_input.env.company.logo" t-options="{'widget': 'image'}" role="img"/>
                        </div>

                    </div>
                    <div class="certification-number" t-if="user_input.scoring_success">
                        Certification n°<t t-esc="str(user_input.id).rjust(10, '0')"/>
                    </div>
                </div>
                <div t-if="user_input.test_entry" class="test-entry"/>
            </div>
        </template>

        <template id="certification_report_view_classic">
            <!-- Style classes to be applyed to '#o_survey_certification': [no class](purple), gold, blue  -->
            <div id="o_survey_certification" t-att-data-oe-model="user_input._name" t-att-data-oe-id="user_input.id" t-att-class="'article certification-wrapper classic %s' % layout_color">
                <div t-if="user_input.test_entry" class="test-entry"/>
                <div class="certification">
                    <div class="certification-top">
                        <h1><b>Certificate</b>
                            <br/><span t-if="user_input.scoring_success">of achievement</span>
                        </h1>
                    </div>

                    <div class="certification-content">
                        <div t-if="user_input.scoring_success">
                            <p>This certificate is presented to
                                 <t t-set="certif_style" t-value="''"/>
                                <t t-set="certified_name" t-value="user_input.partner_id.name or user_input.email or ''"/>
                                <t t-if="certified_name.isupper()">
                                    <t t-set="certif_style" t-value="certif_style + 'font-family: certification-serif;'"/>
                                </t>
                                <t t-if="len(certified_name) > 35">
                                    <t t-set="certif_style" t-value="certif_style + 'font-size: 20px; line-height: 4; font-family: certification-serif; '"/>
                                </t>
                                <t t-elif="len(certified_name) > 20">
                                    <t t-set="certif_style" t-value="certif_style + 'font-size: 30px; line-height: 4;'"/>
                                </t>
                                <br/>
                                <span t-att-style="certif_style" class="user-name" t-esc="certified_name"/>

                                <br/>by <span class="certification-company" t-field="user_input.env.company.display_name"/>
                                for successfully completing
                                <br/><b><span class="certification-name" t-field="user_input.survey_id.display_name"/></b>
                             </p>
                        </div>
                        <div t-else="" class="certification-failed">
                            <p>Certification Failed</p>
                        </div>
                    </div>

                    <div class="certification-bottom">
                            <div class="certification-date-wrapper">
                                <b><div class="certification-date" t-field="user_input.create_date" t-options='{"widget": "date"}'/></b>
                                <span>Date</span>
                            </div>
                            <div class="certification-seal"/>
                            <div class="certification-company">
                                <span class="certification-company-logo" t-field="user_input.env.company.logo" t-options="{'widget': 'image'}" role="img"/>
                            </div>
                        <div class="certification-number" t-if="user_input.scoring_success">
                            Certification n°<t t-esc="str(user_input.id).rjust(10, '0')"/>
                        </div>
                    </div>
                </div>
            </div>
        </template>

        <template id="certification_report_view">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="user_input">
                    <t t-set="layout_values" t-value="user_input.survey_id.certification_report_layout.split('_') if user_input.survey_id.certification_report_layout else ['modern', 'purple']"/>
                    <t t-set="layout_template" t-value="'survey.certification_report_view_%s' % (layout_values[0])"/>
                    <t t-set="layout_color" t-value="layout_values[1]"/>
                    <t t-call="{{layout_template}}"/>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: views\survey_survey_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <record id="survey_form" model="ir.ui.view">
        <field name="name">survey.survey.view.form</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <form string="Survey" class="o_survey_form">
                <field name="id" invisible="1"/>
                <field name="session_state" invisible="1"/>
                <header>
                    <button name="action_send_survey" string="Share" type="object" class="oe_highlight" attrs="{'invisible': [('active', '=', False)]}"/>
                    <button name="action_result_survey" string="See results" type="object" class="oe_highlight"
                      attrs="{'invisible': [('answer_done_count', '&lt;=', 0)]}"/>
                    <button name="action_start_session" string="Create Live Session" type="object"
                        attrs="{'invisible': ['|', ('session_state', '!=', False), '|', ('active', '=', False), ('certification', '=', True)]}" />
                    <button name="action_open_session_manager" string="Open Session Manager" type="object"
                        attrs="{'invisible': [('session_state', '=', False)]}" />
                    <button name="action_end_session" string="Close Live Session" type="object"
                        attrs="{'invisible': [('session_state', 'not in', ['ready', 'in_progress'])]}" />
                    <button name="action_test_survey" string="Test" type="object" attrs="{'invisible': [('active', '=', False)]}"/>
                    <button name="action_print_survey" string="Print" type="object"/>
                    <button name="action_archive" string="Close" type="object" attrs="{'invisible': [('active', '=', False)]}"/>
                    <button name="action_unarchive" string="Reopen" type="object" attrs="{'invisible': [('active', '=', True)]}"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_survey_user_input"
                            type="object"
                            class="oe_stat_button"
                            attrs="{'invisible': [('access_mode', '=', 'public')]}"
                            icon="fa-envelope-o">
                            <field string="Registered" name="answer_count" widget="statinfo"/>
                        </button>
                        <button name="action_survey_user_input_certified"
                            type="object"
                            class="oe_stat_button"
                            attrs="{'invisible': [('certification', '=', False)]}"
                            icon="fa-trophy">
                            <field string="Certified" name="success_count" widget="statinfo"/>
                        </button>
                        <button name="action_survey_user_input_completed"
                            type="object"
                            class="oe_stat_button"
                            icon="fa-pencil-square-o">
                            <field string="Answers" name="answer_done_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <field name="background_image" widget="image" class="oe_avatar"/>
                    <div class="oe_title" style="width: 100%;">
                        <label for="title"/>
                        <h1><field name="title" placeholder="e.g. Satisfaction Survey"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                            <field name="active" invisible="1"/>
                            <field name="has_conditional_questions" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Questions" name="questions">
                            <field name="question_and_page_ids" nolabel="1" widget="question_page_one2many" mode="tree,kanban" context="{'default_survey_id': active_id, 'default_questions_selection': questions_selection}">
                                <tree decoration-bf="is_page" editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="title" widget="survey_description_page"/>
                                    <field name="question_type" />
                                    <field name="is_page" invisible="1"/>
                                    <field name="questions_selection" invisible="1"/>
                                    <field name="survey_id" invisible="1"/>
                                    <field name="triggering_question_id" invisible="1"/>
                                    <field name="random_questions_count" attrs="{'column_invisible': [('parent.questions_selection', '=', 'all')], 'invisible': [('is_page', '=', False)]}" />
                                    <button disabled="disabled" icon="fa-code-fork" attrs="{'invisible': [('triggering_question_id', '=', False)]}"
                                        title="This question depends on another question's answer." class="icon_rotates"/>
                                    <control>
                                        <create name="add_question_control" string="Add a question"/>
                                        <create name="add_section_control" string="Add a section" context="{'default_is_page': True, 'default_questions_selection': 'all'}"/>
                                    </control>
                                </tree>
                            </field>
                        </page>
                        <page string="Description" name="description">
                            <field name="description" nolabel="1"></field>
                        </page>
                        <page string="End Message" name="description_done">
                            <field name="description_done" nolabel="1"></field>
                        </page>
                        <page string="Options" name="options">
                            <group name="options">
                                <group string="Questions" name="questions">
                                    <field name="questions_layout" widget="radio"
                                        attrs="{'readonly': [('session_state', 'in', ['ready', 'in_progress'])]}"/>
                                    <field name="progression_mode" widget="radio" />
                                    <div colspan="2" class="o_checkbox_optional_field">
                                        <label for="is_time_limited" string="Survey Time Limit"/>
                                        <field name="is_time_limited" nolabel="1"/>
                                        <div attrs="{'invisible': [('is_time_limited', '=', False)]}">
                                            <field name="time_limit" widget="float_time" nolabel="1" class="oe_inline"/> minutes
                                        </div>
                                    </div>
                                    <field name="questions_selection" widget="radio" />
                                    <field name="users_can_go_back" string="Back Button" attrs="{'invisible': [('questions_layout', '=', 'one_page')]}"/>
                                </group>
                                <group string="Candidates" name="candidates">
                                    <field name="access_mode"/>
                                    <field name="users_login_required"/>
                                    <div colspan="2" class="o_checkbox_optional_field"
                                        attrs="{'invisible': ['&amp;', ('access_mode', '=', 'public'), ('users_login_required', '=', False)]}">
                                        <label for="is_attempts_limited" string="Attempts Limit"/>
                                        <field name="is_attempts_limited" nolabel="1"/>
                                        <div attrs="{'invisible': [('is_attempts_limited', '=', False)]}">
                                            <field name="attempts_limit" nolabel="1" class="oe_inline"/> attempts
                                        </div>
                                    </div>
                                </group>
                                <group string="Scoring" name="scoring">
                                    <field name="scoring_type" widget="radio" />
                                    <field name="scoring_success_min" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}" />
                                    <field name="certification" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}" />
                                    <field name="certification_mail_template_id" attrs="{'invisible': [('certification', '=', False)]}" />
                                    <label for="certification_report_layout" string="Certification Template" attrs="{'invisible': [('certification', '=', False)]}"/>
                                    <div attrs="{'invisible': [('certification', '=', False)]}" class="d-flex">
                                        <field name="certification_report_layout" class="oe_inline"/>
                                        <button name="action_survey_preview_certification_template"
                                            string="Preview" type="object"
                                            icon="fa-external-link"  target="_blank" class="btn-link pt-0"/>
                                    </div>
                                    <field name="certification_give_badge" attrs="{'invisible': ['|', ('certification', '=', False), ('users_login_required', '=', False)]}" />
                                    <field name="certification_badge_id"
                                           attrs="{'invisible': ['|', ('certification_give_badge', '=', False), ('certification_badge_id', '!=', False)], 'required': [('certification_give_badge', '=', True)]}"
                                           domain="[('survey_id', '=', active_id), ('survey_id', '!=', False)]"
                                           context="{'default_name': title,
                                                   'default_description': 'Congratulations, you have succeeded this certification',
                                                   'default_rule_auth': 'nobody',
                                                   'default_level': None,
                                                   'form_view_ref': 'survey.gamification_badge_form_view_simplified',
                                                   'default_website_published': True}"/>
                                    <field name="certification_badge_id_dummy" attrs="{'invisible': ['|', ('certification_give_badge', '=', False), ('certification_badge_id', '=', False)]}"
                                           options="{'no_create': True}"
                                           context="{'form_view_ref': 'survey.gamification_badge_form_view_simplified'}"/>
                                </group>
                                <group string="Live Session">
                                    <field name="session_speed_rating" />
                                    <div class="o_td_label">
                                        <label for="session_code" string="Session Code" class="oe_edit_only o_form_label" />
                                        <label for="session_link" string="Session Link" class="oe_read_only o_form_label" />
                                    </div>
                                    <div class="d-flex">
                                        <field name="session_code" nolabel="1" class="mr-2 oe_edit_only" />
                                        <field name="session_link" nolabel="1" widget="CopyClipboardChar" class="oe_read_only" />
                                    </div>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>
    <record id="survey_tree" model="ir.ui.view">
        <field name="name">survey.survey.view.tree</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <tree string="Survey" sample="1">
                <field name="active" invisible="1"/>
                <field name="certification" invisible="1"/>
                <field name="title"/>
                <field name="answer_count"/>
                <field name="answer_done_count"/>
                <field name="answer_duration_avg"/>
                <field name="success_count"/>
                <field name="success_ratio"/>
                <field name="answer_score_avg"/>
                <button name="certification" type="button" disabled="disabled"
                    icon="fa-trophy" title="Certification" aria-label="Certification"
                    attrs="{'invisible': [('certification', '=', False)]}"/>
                <!-- Tweak as icons aren't directly supported in xml -->
            </tree>
        </field>
    </record>
    <record id="survey_kanban" model="ir.ui.view">
        <field name="name">survey.survey.view.kanban</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <kanban sample="1">
                <field name="title" />
                <field name="answer_done_count" />
                <field name="certification" />
                <field name="scoring_type" />
                <field name="color" />
                <field name="access_mode"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <field name="success_count"/>
                <field name="success_ratio"/>
                <field name='active'/>
                <templates>
                    <div t-name="kanban-box" 
                        t-attf-class="oe_kanban_color_#{kanban_getcolor(record.color.raw_value)} oe_kanban_card oe_kanban_global_click o_kanban_card_survey 
                            #{record.certification.raw_value ? 'o_kanban_card_survey_successed' : ''}">
                        <div class="o_dropdown_kanban dropdown" t-if="widget.editable">

                            <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                <span class="fa fa-ellipsis-v"/>
                            </a>
                            <div class="dropdown-menu" role="menu">
                                <a role="menuitem" type="edit" class="dropdown-item">Edit Survey</a>
                                <a t-if="record.active.raw_value" role="menuitem" type="object" class="dropdown-item" name="action_send_survey">Share</a>
                                <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                <div role="separator" class="dropdown-divider"/>
                                <div role="separator" class="dropdown-item-text">Color</div>
                                <ul class="oe_kanban_colorpicker" data-field="color"/>
                            </div>
                        </div>
                        <div class="o_kanban_record_top">
                                <h4 class="o_kanban_record_title p-0 mb4"><field name="title" /></h4>
                        </div>
                        <div class="row">
                            <div class="col-10 p-0 pb-1">
                                <div class="container o_kanban_card_content" t-if="record.answer_done_count.raw_value != 0">
                                    <div class="row mt-4 ml-5">
                                        <div class="col-4 p-0">
                                            <a name="action_result_survey" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"><field name="answer_done_count"/></span>
                                                <span class="text-muted">Answers</span>
                                            </a>
                                        </div>
                                        <div class="col-4 p-0 border-left" t-if="record.scoring_type.raw_value != 'no_scoring'" >
                                            <a name="action_survey_user_input_certified" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"><field name="success_count"/></span>
                                                <span class="text-muted" t-if="!record.certification.raw_value">Passed</span>
                                                <span class="text-muted" t-else="">Certified</span>
                                            </a>
                                        </div>
                                        <div class="col-4 p-0 border-left" t-if="record.scoring_type.raw_value != 'no_scoring'" >
                                            <a name="action_survey_user_input_completed" type="object" class="d-flex flex-column align-items-center">
                                                <span class="font-weight-bold"> <t t-esc="Math.round(record.success_ratio.raw_value)"></t> %</span>
                                                <span class="text-muted" >Success</span>
                                            </a> 
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-2 align-self-end">
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left"/>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="activity_ids" widget="kanban_activity"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </templates>
            </kanban>
        </field>
    </record>
    <record id="survey_survey_view_search" model="ir.ui.view">
        <field name="name">survey.survey.search</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <search string="Survey">
                <field string="Survey" name="title"/>
                <filter string="Certification" name="certification" domain="[('certification', '=', True)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Upcoming Activities" name="activities_upcoming_all"
                    domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_survey_form">
        <field name="name">Surveys</field>
        <field name="res_model">survey.survey</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new survey
          </p><p>
            You can create surveys for different purposes: customer opinion, services feedback, recruitment interviews, employee's periodical evaluations, marketing campaigns, etc.
          </p><p>
            Design easily your survey, send invitations and analyze answers.
          </p>
        </field>
    </record>

    <menuitem name="Surveys" id="menu_survey_form" action="action_survey_form" parent="menu_surveys" sequence="1"/>

</data>
</odoo>

```

## File: views\survey_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- Main survey layout -->
    <template id="survey.layout" name="Survey Layout" inherit_id="web.frontend_layout" primary="True">
        <xpath expr="//div[@id='wrapwrap']" position="before">
            <!--TODO DBE Fix me : If one day, there is a survey_livechat bridge module, put this in that module-->
            <t t-set="no_livechat" t-value="True"/>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']" position="attributes">
            <attribute name="t-att-style" add="(('height: 100%; overflow: auto; background: url(' + '/survey/get_background_image/%s/%s' % (survey.access_token, answer.access_token) + ') no-repeat fixed center; box-shadow: inset 0 0 0 10000px rgba(255,255,255,.7); background-size: cover;') if survey and survey.background_image and answer else 'height: 100%; overflow: auto;')"/>
        </xpath>
        <xpath expr="//head/t[@t-call-assets][last()]" position="after">
            <t t-call-assets="survey.survey_assets" lazy_load="True"/>
        </xpath>
        <xpath expr="//header" position="before">
            <t t-set="no_header" t-value="True"/>
            <t t-set="no_footer" t-value="True"/>
        </xpath>
        <xpath expr="//header" position="after">
            <div id="wrap" class="oe_structure oe_empty"/>
        </xpath>
        <xpath expr="//footer" position="after">
            <div class="py-3 m-0 p-0 text-right">
                <div class="o_survey_progress_wrapper d-inline-block pr-5 text-left">
                    <t t-call="survey.survey_progression"
                        t-if="survey and survey.questions_layout != 'one_page' and answer and answer.state == 'in_progress' and (not question or not question.is_page) and not survey_form_readonly">
                        <t t-if="survey.questions_layout == 'page_per_section'">
                            <t t-set="page_ids" t-value="survey.page_ids.ids"/>
                            <t t-set="page_number" t-value="page_ids.index(page.id) + (1 if survey.progression_mode == 'number' else 0)"/>
                        </t>
                        <t t-else="">
                            <t t-if="not answer.is_session_answer and survey.questions_selection == 'random'">
                                <t t-set="page_ids" t-value="answer.predefined_question_ids.ids"/>
                            </t>
                            <t t-else="">
                                <t t-set="page_ids" t-value="survey.question_ids.ids"/>
                            </t>
                            <t t-set="page_number" t-value="page_ids.index(question.id)"/>
                        </t>
                    </t>
                </div>
                <div class="o_survey_brand_message float-right rounded mr-3 border">
                    <div class="px-2 py-2 d-inline-block" t-call="web.brand_promotion_message">
                        <t t-set="_message"></t>
                        <t t-set="_utm_medium" t-valuef="survey"/>
                    </div>
                    <div class="o_survey_navigation_wrapper d-inline-block" t-call="survey.survey_navigation">
                    </div>
                </div>
            </div>
        </xpath>
    </template>

    <!-- Main survey template -->
    <template id="survey_page_fill" name="Survey: main page (take survey)">
        <t t-call="survey.layout">
            <t t-if="answer.test_entry" t-call="survey.survey_button_form_view" />
            <div class="wrap o_survey_wrap d-flex">
                <div class="container o_survey_form d-flex flex-column mb-5">
                    <t t-call="survey.survey_fill_header" />
                    <t t-call="survey.survey_fill_form" />
                </div>
            </div>
        </t>
    </template>

    <template id="survey_fill_header" name="Survey: main page header">
        <div class="o_survey_nav pt16 mb-2">
            <div class="container m-0 p-0">
                <div class="row">
                    <div  class="col-lg-10">
                        <h1 t-if="answer.state == 'new' or survey.questions_layout != 'page_per_question'"
                            t-esc="survey.title" class="o_survey_main_title pt-4"></h1>
                    </div>
                    <div class="o_survey_timer col-lg-2 pt-4">
                        <h1 class="o_survey_timer_container timer text-right">
                        </h1>
                    </div>
                </div>
            </div>
            <div t-att-class="'o_survey_breadcrumb_container mt8' + (' d-none ' if answer.state != 'in_progress' else '')"
                 t-if="not survey.has_conditional_questions and survey.questions_layout == 'page_per_section' and answer.state != 'done'"
                t-att-data-can-go-back="survey.users_can_go_back"
                t-att-data-pages="json.dumps(breadcrumb_pages)" />
        </div>
    </template>

    <template id="survey_fill_form" name="Survey: main page content">
        <t t-set="survey_form_readonly" t-value="answer.state == 'done'"/>
        <form role="form" method="post" t-att-name="survey.id"
                class="d-flex flex-grow-1 align-items-center"
                t-att-data-answer-token="answer.access_token"
                t-att-data-survey-token="survey.access_token"
                t-att-data-users-can-go-back="survey.users_can_go_back and not answer.is_session_answer"
                t-att-data-session-in-progress="answer.is_session_answer"
                t-att-data-is-start-screen="answer.state == 'new'"
                t-att-data-readonly="survey_form_readonly"
                t-att-data-has-answered="bool(has_answered)"
                t-att-data-is-page-description="bool(question and question.is_page and not is_html_empty(question.description))"
                t-att-data-questions-layout="survey.questions_layout"
                t-att-data-triggered-questions-by-answer="json.dumps(triggered_questions_by_answer)"
                t-att-data-selected-answers="json.dumps(selected_answers)">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input type="hidden" name="token" t-att-value="answer.access_token" />
            <div class="o_survey_error alert alert-danger d-none" role="alert">
                <p>There was an error during the validation of the survey.</p>
            </div>

            <div class="o_survey_form_content w-100">
                <t t-if="answer.state == 'new'" t-call="survey.survey_fill_form_start"/>
                <t t-elif="answer.state == 'in_progress'" t-call="survey.survey_fill_form_in_progress" />
                <t t-else="" t-call="survey.survey_fill_form_done"/>
            </div>
        </form>

        <!-- Modal used to display error message, i.c.o. ajax error -->
        <div role="dialog" class="modal fade" id="MasterTabErrorModal" >
            <div class="modal-dialog">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">A problem has occurred</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <main class="modal-body"><p>To take this survey, please close all other tabs on <strong class="text-danger"></strong>.</p></main>
                    <footer class="modal-footer"><button type="button" class="btn btn-primary" data-dismiss="modal">Continue here</button></footer>
                </div>
            </div>
        </div>
    </template>

    <template id="survey_fill_form_start" name="Survey: start form content">
        <div class="wrap o_survey_start">
            <div class='mb32'>
                <div t-field='survey.description' class="oe_no_empty pb-5 text-break"/>
                <t t-if="answer.is_session_answer">
                    <div class="font-weight-bold">
                        The session will begin automatically when the host starts.
                    </div>
                </t>
                <t t-else="">
                    <div t-if="survey.is_time_limited">
                        <p>
                            <span>Time limit for this survey: </span>
                            <span class="font-weight-bold text-danger" t-field="survey.time_limit" t-options="{'widget': 'duration', 'unit': 'minute'}"></span>
                        </p>
                    </div>
                    <button type="submit" value="start" class="btn btn-primary btn-lg">
                        <t t-if="survey.certification">
                            Start Certification
                        </t>
                        <t t-else="">
                            Start Survey
                        </t>
                    </button>
                    <span class="o_survey_enter font-weight-bold text-muted ml-2">or press Enter</span>
                </t>
            </div>
        </div>
    </template>

    <template id="survey_fill_form_in_progress" name="Survey: form with questions">
        <div class="o_survey_form_content_data d-none"
            t-att-data-question-time-limit-reached="answer.question_time_limit_reached"
            t-att-data-has-answered="bool(has_answered)"
            t-att-data-is-page-description="bool(question and question.is_page and not is_html_empty(question.description))"
            t-att-data-server-time="server_time"
            t-att-data-timer="timer_start"
            t-att-data-time-limit-minutes="time_limit_minutes"/>
        <t t-if="survey.questions_layout == 'one_page'">
            <t t-foreach='survey.question_and_page_ids' t-as='question'>
                <h2 t-if="question.is_page" t-field='question.title' class="o_survey_title pb16 text-break" />
                <div t-if="question.is_page" t-field='question.description' class="text-break"/>
                <t t-if="not question.is_page and question in answer.predefined_question_ids" t-call="survey.question_container"/>
            </t>

            <div class="text-center mt16 mb256">
                <button type="submit" value="finish" class="btn btn-primary">
                    Submit</button><span class="font-weight-bold text-muted ml-2 d-none d-md-inline">or press Enter</span>
            </div>
        </t>

        <t t-if="survey.questions_layout == 'page_per_section'">
            <h2 t-field='page.title' class="o_survey_title pb16 text-break" />
            <div t-field='page.description' class="oe_no_empty text-break"/>

            <input type="hidden" name="page_id" t-att-value="page.id" />
            <t t-foreach='page.question_ids' t-as='question'>
                <t t-if="question in answer.predefined_question_ids" t-call="survey.question_container"/>
            </t>

            <div class="row">
                <div class="col-12 text-center mt16">
                    <button type="submit" t-att-value="'next' if not survey_last else 'finish'" class="btn btn-primary">
                        <t t-if="not survey_last">Continue</t>
                        <t t-else="">Submit</t>
                    </button><span class="font-weight-bold text-muted ml-2 d-none d-md-inline"> or press Enter</span>
                </div>
            </div>
        </t>

        <!-- If we have a choice question and less than 6 options, we want minimized display.
        Minimized display means we display the choices vertically (instead of optimized based on screen space).
        An exception is made for options with images, where we always want to optimize screen space.
        Numeric, date and datetime questions are also displayed "minimized", with a smaller screen width.-->
        <t t-set="minimized_display" t-value="survey.questions_layout == 'page_per_question' and len(question.suggested_answer_ids) &lt;= 5 and not any(suggestion.value_image for suggestion in question.suggested_answer_ids)" />
        <div t-if="survey.questions_layout == 'page_per_question'"
            t-attf-class="o_survey_page_per_question o_survey_#{question.question_type} #{'o_survey_minimized_display' if minimized_display else ''}">
            <input type="hidden" name="question_id" t-att-value="question.id" />
            <!-- User has already answered for this session -->
            <t t-if="answer.is_session_answer and (has_answered or answer.question_time_limit_reached)">
                <fieldset disabled="disabled">
                    <t t-set="survey_form_readonly" t-value="True" />
                    <div class="mt-5">
                        <t t-call="survey.question_container" />
                    </div>
                </fieldset>
                <div t-if="answer.question_time_limit_reached and not has_answered" class="font-weight-bold">Sorry, you have not been fast enough.</div>
                <div t-else="" class="font-weight-bold">We have registered your answer! Please wait for the host to go to the next question.</div>
            </t>
            <t t-elif="answer.is_session_answer and question.is_page and not is_html_empty(question.description)">
                <div class="font-weight-bold mt-5">Pay attention to the host screen until the next question.</div>
            </t>
            <t t-else="">
                <div class="mt-5">
                    <t t-call="survey.question_container"/>
                </div>

                <div class="row">
                    <div class="col-12 text-center mt16">
                        <button type="submit" t-att-value="'next' if not survey_last else 'finish'" class="btn btn-primary">
                            <t t-if="answer.is_session_answer or survey_last">Submit</t>
                            <t t-else="">Continue</t>
                        </button>
                        <span class="font-weight-bold text-muted ml-2 d-none d-md-inline">
                            <span t-if="question.question_type == 'text_box'">or press CTRL+Enter</span>
                            <span t-else="">or press Enter</span>
                        </span>
                    </div>
                </div>
            </t>
        </div>
    </template>

    <!-- Finished (taken and finished) survey page -->
    <template id="survey_fill_form_done" name="Survey: finished">
        <div class="wrap">
            <div class="o_survey_finished mt32 mb32">
                <h1>Thank you!</h1>
                <div t-field="survey.description_done" class="oe_no_empty" />
                <div class="row">
                    <div class="col">
                        <t t-if="survey.scoring_type != 'no_scoring' and survey.scoring_success_min">
                            <div>You scored <t t-esc="answer.scoring_percentage" />%</div>
                            <t t-if="answer.scoring_success">
                                <div>Congratulations, you have passed the test!</div>

                                <div t-if="survey.certification" class="mt16 mb16">
                                    <a role="button"
                                        class="btn btn-primary btn-lg"
                                        t-att-href="'/survey/%s/get_certification' % survey.id">
                                        <i class="fa fa-fw fa-trophy" role="img" aria-label="Download certification" title="Download certification"/>
                                        Download certification
                                    </a>
                                </div>
                            </t>
                            <t t-else="">
                                <div>Unfortunately, you have failed the test.</div>
                            </t>
                        </t>
                        <t t-call="survey.survey_button_retake"/>
                        <div t-if="survey.scoring_type != 'scoring_without_answers'">
                            If you wish, you can <a t-att-href="'/survey/print/%s?answer_token=%s&amp;review=True' % (survey.access_token, answer.access_token)">review your answers</a>
                        </div>
                    </div>
                    <div class="col-6 text-center" t-if="survey.certification_give_badge and answer.scoring_success">
                        <img t-att-src="'/web/image/gamification.badge/%s/image_128' % survey.certification_badge_id.id"/>
                        <div>You received the badge <span class="font-weight-bold" t-esc="survey.certification_badge_id.name"/>!</div>
                    </div>
                </div>
            </div>
            <div class="o_survey_result p-4 col-12 row" t-if="graph_data">
                <div t-if="survey.page_ids" class="survey_graph col-lg-6 d-none d-md-block"
                        data-graph-type="by_section"
                        t-att-data-graph-data="graph_data">
                    <canvas id="by_section_chart"></canvas>
                </div>
                <div t-attf-class="survey_graph col-lg-6 #{'offset-lg-3' if not survey.page_ids else ''}"
                        data-graph-type="doughnut"
                        t-att-data-graph-data="graph_data">
                    <canvas id="doughnut_chart"></canvas>
                </div>
            </div>
        </div>
    </template>

    <!-- Question widgets -->
    <template id="question_container" name="Survey: question container">
        <t t-set="display_question"
           t-value="survey.questions_layout == 'page_per_question'
                    or survey.questions_selection == 'random'
                    or (survey.questions_layout == 'one_page' and not question.is_conditional)
                    or (survey.questions_layout == 'page_per_section' and (not question.is_conditional or triggering_answer_by_question[question.id] in selected_answers))"/>

        <t t-set="answer_lines" t-value="answer.user_input_line_ids.filtered(lambda line: line.question_id == question)"/>
        <!--Use Key selection if number of choices is < 26 to keep Z for other choice if any-->
        <t t-set="letters" t-value="'ABCDEFGHIJKLMNOPQRSTUVWXYZ'"/>
        <t t-set="useKeySelection" t-value="len(question.suggested_answer_ids) &lt; len(letters) and survey.questions_layout == 'page_per_question'"/>
        <!-- Extra 'right' margin is added on layouts that are not "page_per_question" to align with choices questions, since all choices have a mr-2 class (pixel perfect yay...) -->
        <t t-set="extra_right_margin" t-value="survey.questions_layout != 'page_per_question' and question.question_type not in ['simple_choice', 'multiple_choice']"/>
        <div t-att-class="'js_question-wrapper pb-4 %s %s' % ('d-none' if not display_question else '', 'mr-2' if extra_right_margin else '')"
             t-att-id="question.id" t-att-data-required="question.constr_mandatory"
             t-att-data-constr-error-msg="question.constr_error_msg"
             t-att-data-validation-error-msg="question.validation_error_msg">
            <div class="mb-4">
                <h3 t-if="not hide_question_title">
                    <span t-field='question.title' class="text-break" />
                    <span t-if="question.constr_mandatory" class="text-danger">*</span>
                </h3>
                <div t-if="not is_html_empty(question.description)" t-field='question.description' class="text-muted oe_no_empty mt-1 text-break"/>
            </div>
            <t t-if="question.question_type == 'text_box'"><t t-call="survey.question_text_box"/></t>
            <t t-if="question.question_type == 'char_box'"><t t-call="survey.question_char_box"/></t>
            <t t-if="question.question_type == 'numerical_box'"><t t-call="survey.question_numerical_box"/></t>
            <t t-if="question.question_type == 'date'"><t t-call="survey.question_date"/></t>
            <t t-if="question.question_type == 'datetime'"><t t-call="survey.question_datetime"/></t>
            <t t-if="question.question_type == 'simple_choice'"><t t-call="survey.question_simple_choice"/></t>
            <t t-if="question.question_type == 'multiple_choice'"><t t-call="survey.question_multiple_choice"/></t>
            <t t-if="question.question_type == 'matrix'"><t t-call="survey.question_matrix"/></t>
            <div class="o_survey_question_error overflow-hidden border-0 py-0 px-3 alert alert-danger" role="alert"></div>
        </div>
    </template>

    <template id="question_text_box" name="Question: free text box">
        <div class="o_survey_comment_container p-0">
            <textarea class="form-control o_survey_question_text_box bg-transparent text-dark rounded-0 p-0" rows="3" t-att-name="question.id"
                      t-att-data-question-type="question.question_type"><t t-if="answer_lines" t-esc="answer_lines[0].value_text_box or None"/></textarea>
        </div>
    </template>

    <template id="question_char_box" name="Question: text box">
        <div class="o_survey_comment_container p-0">
            <input t-att-type="'email' if question.validation_email else 'text'"
               class="form-control o_survey_question_text_box bg-transparent text-dark rounded-0 p-0" t-att-name="question.id"
               t-att-value="answer_lines[0].value_char_box if answer_lines else None"
               t-att-data-question-type="question.question_type"
               t-att-data-validation-length-min="question.validation_length_min if question.validation_required else False"
               t-att-data-validation-length-max="question.validation_length_max if question.validation_required else False"/>
        </div>
    </template>

    <template id="question_numerical_box" name="Question: numerical box">
        <input type="number" step="any" class="form-control o_survey_question_numerical_box bg-transparent text-dark rounded-0 p-0"
               t-att-name="question.id" t-att-value="answer_lines[0].value_numerical_box if answer_lines else None"
               t-att-data-question-type="question.question_type"
               t-att-data-validation-float-min="question.validation_min_float_value if question.validation_required else False"
               t-att-data-validation-float-max="question.validation_max_float_value if question.validation_required else False"/>
    </template>

    <template id="question_date" name="Question: date box">
        <div class="input-group o_survey_form_date" t-attf-id="datetimepicker_#{question.id}" data-target-input="nearest"
                t-att-data-mindate="question.validation_min_date"
                t-att-data-maxdate="question.validation_max_date">
            <input type="text" class="form-control datetimepicker-input o_survey_question_date bg-transparent text-dark rounded-0 p-0"
                   t-attf-data-target="#datetimepicker_#{question.id}" t-att-name="question.id"
                   t-att-value="format_date(answer_lines[0].value_date) if answer_lines else None"
                   t-att-data-question-type="question.question_type"/>
            <div t-if="not survey_form_readonly" class="input-group-append position-absolute" t-attf-data-target="#datetimepicker_#{question.id}" data-toggle="datetimepicker">
                <div class="input-group-text text-primary border-0 bg-transparent p-0"><i class="fa fa-calendar"></i></div>
            </div>
        </div>
    </template>

    <template id="question_datetime" name="Question: datetime box">
        <div class="input-group o_survey_form_date" t-attf-id="datetimepicker_#{question.id}" data-target-input="nearest"
                t-att-data-mindate="question.validation_min_datetime"
                t-att-data-maxdate="question.validation_max_datetime">
            <input type="text" class="form-control datetimepicker-input o_survey_question_datetime bg-transparent text-dark rounded-0 p-0"
                   t-attf-data-target="#datetimepicker_#{question.id}" t-att-name="question.id"
                   t-att-value="format_datetime(answer_lines[0].value_datetime) if answer_lines else None"
                   t-att-data-question-type="question.question_type"/>
            <div t-if="not survey_form_readonly" class="input-group-append position-absolute" t-attf-data-target="#datetimepicker_#{question.id}" data-toggle="datetimepicker">
                <div class="input-group-text text-primary border-0 bg-transparent p-0"><i class="fa fa-calendar"></i></div>
            </div>
        </div>
    </template>

    <template id="question_suggested_value_image" name="Image from the question suggested answer">
        <t t-if="question.allow_value_image and label.value_image">
            <!-- Directly use field or route if the user doesn't have access rights -->
            <div t-if="not env.user.has_group('survey.group_survey_user')"
                class="d-flex my-3 justify-content-center">
                <img t-att-src="'/survey/get_question_image/%s/%s/%s/%s' % (survey.access_token, answer.access_token, question.id, label.id)"/>
            </div>
            <div t-else=""  t-field="label.value_image"
                class="d-flex my-3 justify-content-center"
                t-options="{'widget': 'image', 'alt-field': 'name', 'itemprop': 'image'}"/>
        </t>
    </template>

    <template id="question_simple_choice" name="Question: simple choice">
        <t t-set="answer_line" t-value="answer_lines.filtered(lambda line: line.suggested_answer_id)"/>
        <t t-set="comment_line" t-value="answer_lines.filtered(lambda line: line.value_char_box)"/>
        <div class="row o_survey_form_choice"
             t-att-data-name="question.id"
             data-question-type="simple_choice_radio">
            <t t-set="item_idx" t-value="0"/>
            <div t-attf-class="col-lg-12 d-flex flex-wrap">
                <t t-set="has_correct_answer" t-value="scoring_display_correction and any(label.is_correct for label in question.suggested_answer_ids)"/>
                <t t-foreach='question.suggested_answer_ids' t-as='label'>
                    <t t-set="item_idx" t-value="label_index"/>
                    <t t-set="answer_selected" t-value="answer_line and answer_line.suggested_answer_id.id == label.id"/>
                    <t t-set="is_correct" t-value="label.is_correct"/>

                    <!--Used for print mode with corrections -->
                    <t t-set="answer_class" t-if="not has_correct_answer" t-value="''" />
                    <t t-set="answer_class" t-elif="is_correct" t-value="'bg-success'" />
                    <t t-set="answer_class" t-elif="not is_correct" t-value="'bg-danger'" />

                    <label t-att-for="str(question.id) + '_' + str(label.id)"
                           t-att-class="'o_survey_choice_btn mr-2 mb-2 py-1 px-3 rounded %s %s' % (answer_class, 'o_survey_selected' if answer_selected else '')">
                        <t t-call="survey.survey_selection_key">
                            <t t-set="selection_key_class" t-value="'position-relative o_survey_radio_btn float-left d-flex'"/>
                        </t>
                        <span class="ml-2 text-break" t-field='label.value'/>
                        <input t-att-id="str(question.id) + '_' + str(label.id)" type="radio" t-att-value='label.id' class="o_survey_form_choice_item invisible position-absolute"
                               t-att-name='question.id'
                               t-att-checked="answer_line and answer_line.suggested_answer_id.id == label.id and 'checked' or None"
                               t-att-data-selection-key="letters[item_idx] if useKeySelection else ''"/>
                        <i class="fa fa-check-circle float-right mt-1 position-relative"></i>
                        <i class="fa fa-circle-thin float-right mt-1 position-relative"></i>
                        <t t-call="survey.question_suggested_value_image"/>
                    </label>
                </t>
            </div>
            <div t-if='question.comments_allowed and question.comment_count_as_answer' class="js_comments col-lg-12" >
                <div class="d-flex flex-wrap">
                    <label t-att-class="'o_survey_choice_btn mr-2 py-1 px-3 rounded %s' % ('o_survey_selected' if comment_line else '')">
                        <t t-set="item_idx" t-value="item_idx + 1"/>
                        <t t-call="survey.survey_selection_key">
                            <t t-set="selection_key_class" t-value="'position-relative o_survey_radio_btn float-left d-flex'"/>
                        </t>
                        <input type="radio" class="o_survey_form_choice_item o_survey_js_form_other_comment invisible position-absolute" value="-1"
                               t-att-name='question.id'
                               t-att-checked="comment_line and 'checked' or None"
                               t-att-data-selection-key="letters[item_idx] if useKeySelection else ''"/>
                        <span class="ml-2" t-field="question.comments_message" />
                        <i class="fa fa-check-circle float-right mt-1 position-relative"></i>
                        <i class="fa fa-circle-thin float-right mt-1 position-relative"></i>
                    </label>
                </div>
                <div t-attf-class="o_survey_comment_container mt-3 py-0 px-1  #{'d-none' if not comment_line else ''}">
                    <textarea type="text" class="form-control o_survey_question_text_box bg-transparent text-dark rounded-0 p-0"
                              t-att-disabled="None if comment_line else 'disabled'"><t t-esc="comment_line.value_char_box if comment_line else ''"/></textarea>
                </div>
            </div>
            <div t-if='question.comments_allowed and not question.comment_count_as_answer' class="col-lg-12 o_survey_comment_container mx-1 mt-3 pl-3 pr-4">
                <textarea type="text" class="form-control o_survey_comment o_survey_question_text_box bg-transparent text-dark rounded-0 p-0"
                          t-att-placeholder="question.comments_message if not survey_form_readonly else ''"><t t-esc="comment_line.value_char_box if comment_line else ''"/></textarea>
            </div>
        </div>
    </template>

    <template id="question_multiple_choice" name="Question: multiple choice">
        <t t-set="comment_line" t-value="answer_lines.filtered(lambda line: line.value_char_box)"/>
        <div class="row o_survey_form_choice o_survey_question_multiple_choice"
             t-att-data-name="question.id"
             t-att-data-question-type="question.question_type">
            <t t-set="item_idx" t-value="0"/>
            <div class="d-flex flex-wrap col-lg-12">
                <t t-set="has_correct_answer" t-value="scoring_display_correction and any(label.is_correct for label in question.suggested_answer_ids)"/>
                <t t-foreach='question.suggested_answer_ids' t-as='label'>
                    <t t-set="item_idx" t-value="label_index"/>
                    <t t-set="answer_line" t-value="answer_lines.filtered(lambda line: line.suggested_answer_id == label)"/>
                    <t t-set="answer_selected" t-value="answer_line and answer_line.suggested_answer_id.id == label.id"/>
                    <t t-set="is_correct" t-value="label.is_correct"/>

                    <!--Used for print mode with corrections -->
                    <t t-set="answer_class" t-if="not has_correct_answer" t-value="''" />
                    <t t-set="answer_class" t-elif="is_correct" t-value="'bg-success'" />
                    <t t-set="answer_class" t-elif="not is_correct" t-value="'bg-danger'" />

                    <label t-att-class="'o_survey_choice_btn mr-2 py-1 px-3 rounded %s %s' % (answer_class, 'o_survey_selected' if answer_line else '')">
                        <t t-call="survey.survey_selection_key">
                            <t t-set="selection_key_class" t-value="'position-relative float-left d-flex'"/>
                        </t>
                        <input type="checkbox" t-att-value='label.id' class="o_survey_form_choice_item invisible position-absolute"
                               t-att-name="question.id"
                               t-att-checked="'checked' if answer_line else None"
                               t-att-data-selection-key="letters[item_idx] if useKeySelection else ''"/>
                        <span class="ml-2 text-break" t-field='label.value'/>
                        <i class="fa fa-check-square float-right mt-1 position-relative"></i>
                        <i class="fa fa-square-o float-right mt-1 position-relative"></i>
                        <t t-call="survey.question_suggested_value_image"/>
                    </label>
                </t>
            </div>
            <div t-if='question.comments_allowed and question.comment_count_as_answer' class="js_ck_comments col-lg-12" >
                <div class="d-flex flex-wrap">
                    <label t-att-class="'o_survey_choice_btn mr-2 py-1 px-3 rounded %s' % ('o_survey_selected' if comment_line else '')">
                        <t t-set="item_idx" t-value="item_idx + 1"/>
                        <t t-call="survey.survey_selection_key">
                            <t t-set="selection_key_class" t-value="'position-relative float-left d-flex'"/>
                        </t>
                        <input type="checkbox" class="o_survey_form_choice_item o_survey_js_form_other_comment invisible position-absolute" value="-1"
                               t-att-name="question.id"
                               t-att-checked="comment_line and 'checked' or None"
                               t-att-data-selection-key="letters[item_idx] if useKeySelection else ''"/>
                        <span class="ml-2" t-field="question.comments_message" />
                        <i class="fa fa-check-square float-right mt-1 position-relative"></i>
                        <i class="fa fa-square-o float-right mt-1 position-relative"></i>
                    </label>
                </div>
                <div t-attf-class="o_survey_comment_container mt-3 py-0 px-1 #{'d-none' if not comment_line else ''}">
                    <textarea type="text" class="form-control o_survey_question_text_box bg-transparent text-dark rounded-0 p-0"
                              t-att-disabled="None if comment_line else 'disabled'"><t t-esc="comment_line.value_char_box if comment_line else ''"/></textarea>
                </div>
            </div>
            <div t-if='question.comments_allowed and not question.comment_count_as_answer' class="col-lg-12 o_survey_comment_container mx-1 mt-3 py-0 pl-3 pr-4">
                <textarea type="text" class="form-control o_survey_comment o_survey_question_text_box bg-transparent text-dark rounded-0 p-0"
                          t-att-placeholder="question.comments_message if not survey_form_readonly else ''"><t t-esc="comment_line.value_char_box if comment_line else ''"/></textarea>
            </div>
        </div>
    </template>

    <template id="question_matrix" name="Question: matrix">
        <t t-set="comment_line" t-value="answer_lines.filtered(lambda line: line.value_char_box)"/>
        <table class="table table-hover o_survey_question_matrix text-white text-center mb-0"
               t-att-data-name="question.id"
               t-att-data-question-type="question.question_type"
               t-att-data-sub-questions="question.matrix_row_ids.ids">
            <thead>
                <tr>
                    <th class="border-0"> </th>
                    <th class="border-0 font-weight-normal" t-foreach="question.suggested_answer_ids" t-as="col_label"><span t-field="col_label.value" /></th>
                </tr>
            </thead>
            <tbody>
                <t t-set="item_idx" t-value="0"/>
                <!-- For matrix, we have an extra check because we have rows * columns total options -->
                <t t-set="useKeySelection" t-value="useKeySelection and (len(question.suggested_answer_ids) * len(question.matrix_row_ids)) &lt; len(letters)" />
                <tr class="bg-white text-white" t-foreach="question.matrix_row_ids" t-as="row_label" t-att-id="row_label.id">
                    <th class="border-0 font-weight-normal text-left"><span t-field="row_label.value" /></th>
                    <t t-foreach="question.suggested_answer_ids" t-as="col_label">
                        <t t-set="answer" t-value="answer_lines.filtered(lambda line: line.suggested_answer_id == col_label and line.matrix_row_id == row_label)"/>
                        <td t-att-class="'o_survey_matrix_btn text-primary border-0 position-relative %s'
                        % ('o_survey_selected' if answer else '')">
                            <input t-att-type="'checkbox' if question.matrix_subtype == 'multiple' else 'radio'"
                                   t-att-name="'%s_%s' % (question.id, row_label.id)" t-att-value='col_label.id'
                                   t-att-checked="'checked' if answer else None"
                                   t-att-data-row-id="row_label.id"
                                   t-att-data-selection-key="letters[item_idx] if useKeySelection else ''"
                                   class="o_survey_form_choice_item d-none"/>
                            <i t-att-class="'o_survey_matrix_empty_checkbox fa fa-%s position-relative'
                            % ('square-o' if question.matrix_subtype == 'multiple' else 'circle-thin')"></i>
                            <i t-att-class="'fa fa-%s position-relative'
                            % ('check-square' if question.matrix_subtype == 'multiple' else 'check-circle')"></i>
                            <t t-call="survey.survey_selection_key">
                                <t t-set="selection_key_class"
                                   t-value="'position-absolute float-right font-weight-bold %s' % ('o_survey_radio_btn' if question.matrix_subtype != 'multiple' else '')"/>
                            </t>
                            <t t-set="item_idx" t-value="item_idx + 1"/>
                        </td>
                    </t>
                </tr>
            </tbody>
        </table>
        <div t-if='question.comments_allowed'>
            <textarea type="text" class="form-control o_survey_question_text_box o_survey_comment bg-transparent text-dark rounded-0 p-0 mt-3"
                      t-att-placeholder="question.comments_message if not survey_form_readonly else ''"
                      t-att-name="'%s_%s' % (question.id, 'comment')"><t t-esc="comment_line.value_char_box if comment_line else ''"/></textarea>
        </div>
    </template>

    <template id="survey_selection_key">
        <div t-if="useKeySelection" t-att-class="'o_survey_choice_key bg-white rounded %s' % selection_key_class">
             <span class="o_survey_key text-center position-absolute bg-white rounded-left py-0 pl-2"><span class="text-primary text-center text-center w-100 position-relative">Key</span></span>
             <span class="text-primary text-center w-100 position-relative" t-esc="letters[item_idx]"/>
        </div>
    </template>

    <template id="survey_progression" name="Survey: Progression">
        <t t-if="len(page_ids) > 1 and not survey.has_conditional_questions">
            <t t-set="percentage" t-value="round(100*(page_number/len(page_ids)))"/>
            <t t-if="survey.progression_mode == 'percent'">
                <span class="o_survey_progress_percent" t-esc="percentage"/> % completed
            </t>
            <t t-else="">
                <span class="o_survey_progress_number" t-esc="page_number"/> of <span t-esc="len(page_ids)"/>
                <span t-if="survey.questions_layout == 'page_per_question'">answered</span>
                <span t-else="">pages</span>
            </t>
            <div class="o_survey_progress progress flex-grow-1">
                <div class="progress-bar bg-primary" t-att-style="'width: ' + str(percentage) + '%'"/>
            </div>
        </t>
    </template>
</data>
</odoo>

```

## File: views\survey_templates_management.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <!-- ============================================================ -->
    <!--                Errors / Corner case management               -->
    <!-- ============================================================ -->

    <!-- Forbidden error messages-->
    <template id="survey_403_page" name="Survey: custom 403 page">
        <t t-call="survey.layout">
            <div id="wrap">
                <div class="container">
                    <h1 class="mt32">403: Forbidden</h1>
                    <p>The page you were looking for could not be authorized.</p>
                    <p>Maybe you were looking for
                        <a t-attf-href="/web#view_type=form&amp;model=survey.survey&amp;id=#{survey.id}&amp;action=survey.action_survey_form">this page</a> ?
                    </p>
                </div>
            </div>
        </t>
    </template>

    <!-- Error: void survey -->
    <template id="survey_void_content" name="Survey: void content">
        <t t-call="survey.layout">
            <t t-if="answer.test_entry" t-call="survey.survey_button_form_view" />
            <div class="wrap">
                <div class="container">
                    <div class="jumbotron mt32">
                        <h1><span t-field="survey.title"/> survey is empty</h1>
                        <p t-if="env.user.has_group('survey.group_survey_user')">
                            Please make sure you have at least one question in your survey. You also need at least one section if you chose the "Page per section" layout.<br />
                            <a t-att-href="'/web#view_type=form&amp;model=survey.survey&amp;id=%s&amp;action=survey.action_survey_form' % survey.id"
                                class="btn btn-secondary"
                                groups="survey.group_survey_manager">Edit in backend</a>
                        </p>
                        <p t-else="">
                            No question yet, come back later.
                        </p>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Error: auth required  -->
    <template id="survey_auth_required" name="Survey: login required">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="container">
                    <div class="jumbotron mt32">
                        <h1>Login required</h1>
                        <p>This survey is open only to registered people. Please
                            <a t-att-href="redirect_url">log in</a>.
                        </p>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Expired (closed) survey page -->
    <template id="survey_closed_expired" name="Survey: expired">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="container">
                    <div class="jumbotron mt32">
                        <h1><span t-field="survey.title"/> survey expired</h1>
                        <p>This survey is now closed. Thank you for your interest !</p>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- ============================================================ -->
    <!--                       Tools / Utilities                      -->
    <!-- ============================================================ -->

    <template id="survey_button_form_view" name="Survey: back to form view">
        <div groups="survey.group_survey_manager" t-ignore="true" class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden mb-0">
            <div t-ignore="true" class="text-center">
                <a t-attf-href="/web#view_type=form&amp;model=survey.survey&amp;id=#{survey.id}&amp;action=survey.action_survey_form"><i class="fa fa-fw fa-arrow-right"/><span t-if="answer and answer.test_entry">This is a test survey. </span>Edit Survey</a>
            </div>
            <button type="button" class="close" data-dismiss="alert" aria-label="Close"> &#215; </button>
        </div>
    </template>

    <template id="survey_button_retake" name="Survey: retake button">
        <div>
            <t t-if="not answer.scoring_success and not answer.is_session_answer">
                <t t-if="survey.is_attempts_limited">
                    <t t-set="attempts_left" t-value="survey._get_number_of_attempts_lefts(answer.partner_id, answer.email, answer.invite_token)" />
                    <t t-if="attempts_left > 0">
                        <p><span>Number of attemps left</span>: <span t-esc="attempts_left"></span></p>
                        <p><a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/retry/%s/%s' % (survey.access_token, answer.access_token)">
                        Retry</a></p>
                    </t>
                </t>
                <t t-else="">
                    <p><a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/retry/%s/%s' % (survey.access_token, answer.access_token)">
                        Retry</a></p>
                </t>
            </t>
        </div>
    </template>

    <!-- Survey Home page - Session Code
    Used in 'session mode' to give an easy access to the survey through the '/s' route. -->
    <template id="survey_session_code" name="Survey: Access Code page">
        <t t-call="survey.layout">
            <div class="wrap o_survey_wrap pb16 d-flex">
                <div class="container o_survey_quick_access d-flex flex-column">
                    <div class="d-flex flex-grow-1 align-items-center">
                        <div class="w-100 px-4 px-md-0">
                            <div class="text-center mb32">
                                <h3>Enter Session Code</h3>
                            </div>
                            <div class="row">
                                <div class="col-12 col-md-4 offset-md-4 text-center">
                                    <input id="session_code" type="text" placeholder="4812"
                                           class="form-control o_survey_question_text_box font-weight-bold bg-transparent text-primary text-center rounded-0 p-2 w-100"/>
                                </div>
                                <div class="col-12 col-md-4 offset-md-4 text-center o_survey_error text-danger pt-2" role="alert">
                                    <span class="o_survey_session_error_invalid_code d-none">Code is incorrect.</span>
                                    <span class="o_survey_session_error_closed d-none">Session is finished.</span>
                                </div>
                            </div>
                            <div class="text-center mt32 p-2">
                                <button type="submit" class="btn btn-primary">Join Session</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="survey_navigation" name="Survey: Navigation">
        <!-- Keep this enclosing div so that the template always returns valid HTML content, no matter the use case -->
        <div class="d-inline-block">
            <button t-if="can_go_back"
                type="submit" class="btn p-0 shadow-none o_survey_navigation_submit" name="button_submit" value="previous" t-att-data-previous-page-id="previous_page_id">
                <i class="border-left fa fa-chevron-left p-2" />
            </button>
            <button t-if="survey and survey.questions_layout in ['page_per_question', 'page_per_section'] and answer and answer.state != 'done' and not answer.is_session_answer"
                type="submit" class="btn p-0 shadow-none o_survey_navigation_submit" t-att-value="'next' if not survey_last else 'finish'">
                <i class="border-left fa fa-chevron-right p-2" />
            </button>
        </div>
    </template>

</data>
</odoo>

```

## File: views\survey_templates_print.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- Survey: printable page view (all pages) -->
    <template id="survey_page_print" name="Survey: print page">
        <t t-call="survey.layout">
            <t t-set="survey_form_readonly" t-value="true"/>
            <t t-if="answer.test_entry" t-call="survey.survey_button_form_view" />
            <div class="wrap">
                <div class="o_survey_print container">
                    <div class='py-5 mt32'>
                        <h1><span t-field='survey.title' class="text-break"/></h1>
                        <t t-if="survey.description"><div t-field='survey.description' class="oe_no_empty text-break"/></t>
                        <t t-if="review" t-call="survey.survey_button_retake"/>
                    </div>
                    <div role="form">
                        <fieldset disabled="disabled">
                            <t t-foreach='survey.question_and_page_ids' t-as='question'>
                                <t t-if="question.is_page and
                                            (any(q in questions_to_display for q in question.question_ids)
                                            or not is_html_empty(question.description))">
                                    <hr t-if="question != survey.page_ids[0]" />
                                    <div class="o_page_header">
                                        <h1 t-field='question.title' class="text-break" />
                                        <div t-if="question.description" t-field='question.description' class="oe_no_empty text-break"/>
                                    </div>
                                </t>
                                <t t-if="not question.is_page and not answer or (question in answer.predefined_question_ids &amp; questions_to_display)" >
                                    <t t-set="answer_lines" t-value="answer.user_input_line_ids.filtered(lambda line: line.question_id == question)"/>
                                    <div class="js_question-wrapper" t-att-id="question.id">
                                        <h2>
                                            <span t-field='question.title' class="text-break"/>
                                            <span t-if="question.constr_mandatory" class="text-danger">*</span>
                                            <span t-if="scoring_display_correction" class="badge badge-pill" t-att-data-score-question="question.id"></span>
                                        </h2>
                                        <t t-if="question.description"><div class="text-muted oe_no_empty text-break" t-field='question.description'/></t>
                                        <t t-if="question.question_type == 'text_box'"><t t-call="survey.question_text_box"/></t>
                                        <t t-if="question.question_type == 'char_box'"><t t-call="survey.question_char_box"/></t>
                                        <t t-if="question.question_type == 'numerical_box'"><t t-call="survey.question_numerical_box"/></t>
                                        <t t-if="question.question_type == 'date'"><t t-call="survey.question_date"/></t>
                                        <t t-if="question.question_type == 'datetime'"><t t-call="survey.question_datetime"/></t>
                                        <t t-if="question.question_type == 'simple_choice'"><t t-call="survey.question_simple_choice"/></t>
                                        <t t-if="question.question_type == 'multiple_choice'"><t t-call="survey.question_multiple_choice"/></t>
                                        <t t-if="question.question_type == 'matrix'"><t t-call="survey.question_matrix"/></t>
                                        <div class="o_survey_question_error overflow-hidden border-0 py-0 px-3 alert alert-danger" role="alert"></div>
                                    </div>
                                </t>
                            </t>
                        </fieldset>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- simple template with no assets, to show title on the certification preview -->
    <template id="certification_preview">
        <html>
            <head>
                <title t-esc="'%s Preview' % page_title"/>
                <link rel="shortcut icon" href="/web/static/img/favicon.ico" type="image/x-icon"/>
            </head>
            <body style="margin:0;">
                <iframe type="application/pdf" t-att-src="preview_url" frameBorder="0" width="100%" height="100%"/>
            </body>
        </html>
    </template>
</data>
</odoo>

```

## File: views\survey_templates_statistics.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <template id="survey_page_statistics" name="Survey: result statistics page">
        <t t-call="survey.layout">
            <t t-call="survey.survey_button_form_view" />
            <t t-set="page_record_limit" t-value="10"/><!-- Change this record_limit to change number of record  per page-->
            <div class="container o_survey_result">
                <t t-call="survey.survey_page_statistics_header" />
                <t t-call="survey.survey_page_statistics_inner" />
            </div>
        </t>
    </template>

    <template id="survey_page_statistics_header" name="Survey: result statistics header">
        <div class="py-5 mt32">
            <h1>
                <span t-field="survey.title"/>
                <span style="font-size:1.5em;"
                    t-attf-class="fa fa-bar-chart-o #{'fa-bar-chart-o' if survey.scoring_type == 'no_scoring' else 'fa-trophy' if survey.certification else 'fa-question-circle-o'} float-right " role="img" aria-label="Chart" title="Chart"/>
            </h1>
            <div t-field="survey.description" class="oe_no_empty"/>
            <h2 t-if="not question_and_page_data">
                Sorry, no one answered this survey yet.
            </h2>
        </div>
    </template>

    <template id="survey_page_statistics_inner" name="Survey: result statistics content">
        <div t-if="survey.session_show_leaderboard" class="o_survey_session_leaderboard mb-5 mt-1">
            <h2 class="mt16 text-uppercase text-muted">Leaderboard</h2>
            <t t-call="survey.user_input_session_leaderboard" />
        </div>
        <div t-if="question_and_page_data" class="card d-print-none">
            <div class="card-header"><span class="fa fa-filter"></span>  Filters <span t-if="search_filters" class="float-right text-primary clear_survey_filter"><i class="fa fa-times"></i> Clear All Filters</span></div>
            <div class="card-body">
                <span t-att-class="'badge only_left_radius filter-all %s' % 'badge-secondary' if search_finished else 'badge-primary o_active_filter'">All surveys</span>
                <span t-att-class="'badge only_right_radius filter-finished %s' % 'badge-secondary' if not search_finished else 'badge-primary o_active_filter'">Finished surveys</span>
                <span t-foreach="search_filters" t-as="filter_data">
                    <span class="badge badge-primary only_left_radius"><i class="fa fa-filter" role="img" aria-label="Filter" title="Filter"></i></span><span class="badge badge-primary no_radius" t-esc="filter_data['question']"></span><span class="badge badge-success only_right_radius" t-esc="filter_data['answers']"></span>
                </span>
            </div>
        </div>
        <div t-if="survey.scoring_type in ['scoring_with_answers', 'scoring_without_answers']">
            <h2 class="mt16 text-uppercase text-muted">Results Overview</h2>
            <div>Success rate: <mark class="font-weight-bold"><t t-esc="survey_data['global_success_rate']"></t>%</mark></div>
            <div class="survey_graph" data-graph-type="pie" t-att-data-graph-data="survey_data['global_success_graph']">
                <!-- canvas element for drawing pie chart -->
                <canvas/>
            </div>
            <hr/>
        </div>

        <div t-foreach="question_and_page_data" t-as='question_data'>
            <t t-set="question" t-value="question_data['question']"/>
            <t t-if="question_data['is_page']">
                <h2 class="mt16 text-uppercase text-muted" t-field="question.title"/>
                <div t-field="question.description" class="oe_no_empty" />
                <hr class="mt-2 pb-1 mb-1"/>
            </t>
            <div t-else="" class="ml-4 mt-4">
                <t t-call="survey.survey_page_statistics_question" />
            </div>
        </div>
    </template>

    <template id="survey_page_statistics_question" name="Question: result statistics">
        <t t-set="question_scoring" t-value="survey.scoring_type in ['scoring_with_answers', 'scoring_without_answers']"/>
        <t t-set="comment_lines" t-value="question_data['comment_line_ids']"/>
        <t t-set="graph_data" t-value="question_data['graph_data']"/>
        <t t-set="table_data" t-value="question_data['table_data']"/>

        <div>
        <div class="d-flex align-items-start mb-2">
            <div class="mr-auto">
                <h5 t-field="question.title" class="mb-1"/>
                <div t-if="question_scoring">
                    <t t-if="question.question_type in ['simple_choice', 'multiple_choice']">
                        <t t-if="len(question_data['right_answers']) >= 1">
                            <span t-if="len(question_data['right_answers']) == 1">Right answer: </span>
                            <span t-if="len(question_data['right_answers']) > 1">Right answers: </span>
                            <strong>
                                <t t-foreach="question_data['right_answers']" t-as="answer">
                                    <span t-if="answer_index != 0">, </span><span t-esc="answer.value"/>
                                </t>
                            </strong>
                        </t>
                    </t>
                    <t t-elif="question.is_scored_question">
                        <span>Right answer:</span>
                        <strong>
                            <t t-if="question.question_type == 'numerical_box'">
                                <span t-esc="question.answer_numerical_box"/>
                            </t>
                            <t t-if="question.question_type == 'date'">
                                <span t-field="question.answer_date"/>
                            </t>
                            <t t-if="question.question_type == 'datetime'">
                                <span t-field="question.answer_datetime"/>
                            </t>
                        </strong><br/>
                        <span>Score:</span>
                        <strong>
                            <span t-esc="question.answer_score"/>
                        </strong>
                    </t>
                </div>
            </div>
            <!-- Question info -->
            <span class="badge badge-info" t-field='question.question_type'/>
            <t t-if="question.question_type == 'matrix'">
                <span class="badge badge-info ml-2" t-field='question.matrix_subtype'/>
            </t>
            <!-- Scoring info -->
            <t t-if="question_scoring">
                <t t-if="question.question_type in ['simple_choice', 'multiple_choice'] or question.is_scored_question">
                    <span class="badge badge-success ml-3"><span t-esc="question_data['right_inputs_count']"></span> Correct</span>
                </t>
                <t t-if="question.question_type in ['simple_choice', 'multiple_choice']">
                    <span class="badge badge-warning ml-1" t-if="question.question_type == 'multiple_choice'">
                        <span t-esc="question_data['partial_inputs_count']"></span> Partial
                    </span>
                </t>
            </t>
            <!-- Inputs info -->
            <span class="badge badge-info ml-3"><span t-esc="len(question_data['answer_input_done_ids'])"></span> Answered</span>
            <span class="badge badge-info ml-1"><span t-esc="len(question_data['answer_input_skipped_ids'])"></span> Skipped</span>
        </div>

        <!-- Question Description -->
        <div class="ml-3 text-muted" t-field="question.description"/>
            <t t-if="question.question_type in ['text_box', 'char_box', 'datetime']">
                <t t-call="survey.question_result_text"/>
            </t>
            <t t-elif="question.question_type in ['numerical_box', 'date']">
                <t t-call="survey.question_result_number_or_date"></t>
            </t>
            <t t-elif="question.question_type in ['simple_choice', 'multiple_choice']">
                <t t-call="survey.question_result_choice"></t>
            </t>
            <t t-elif="question.question_type in ['matrix']">
                <t t-call="survey.question_result_matrix"></t>
            </t>
        </div>
    </template>

    <template id="question_result_text" name="Question: text result (text_box, char_box, datetime)">
        <table class="table table-hover table-sm" t-att-id="'survey_table_question_%d' % question.id">
            <thead>
                <tr>
                    <th>#</th>
                    <th>User Responses</th>
                </tr>
            </thead>
            <tbody>
                <t t-foreach="table_data" t-as="input_line">
                    <tr t-if="not input_line.skipped" t-att-class="'d-none' if input_line_index >= page_record_limit else ''">
                        <td>
                            <t t-if="no_print_url"><t t-esc="input_line_index + 1"></t></t>
                            <t t-else="">
                                <a t-att-href="input_line.user_input_id.get_print_url()">
                                    <t t-esc="input_line_index + 1"></t>
                                </a>
                            </t>
                        </td>
                        <td>
                            <t t-if="question.question_type == 'text_box'">
                                <span t-if="no_print_url" t-field="input_line.value_text_box" />
                                <a t-else="" t-att-href="input_line.user_input_id.get_print_url()" t-field="input_line.value_text_box"></a>
                            </t>
                            <t t-if="question.question_type == 'char_box'">
                                <span t-if="no_print_url" t-field="input_line.value_char_box" />
                                <a t-else="" t-att-href="input_line.user_input_id.get_print_url()" t-field="input_line.value_char_box"></a>
                            </t>
                            <t t-if="question.question_type == 'datetime'">
                                <span t-if="no_print_url" t-field="input_line.value_datetime" />
                                <a t-else="" t-att-href="input_line.user_input_id.get_print_url()" t-field="input_line.value_datetime"></a>
                            </t>
                        </td>
                    </tr>
                </t>
            </tbody>
        </table>
        <t t-call="survey.question_table_pagination"/>
    </template>

    <template id="question_result_number_or_date" name="Question: number or date result (numerical_box or date)">
        <t t-if="question.question_type == 'numerical_box'">
            <span class="float-right mt8">
                <span class="badge badge-secondary only_left_radius">Maximum </span> <span class="badge badge-success only_right_radius" t-esc="question_data['numerical_max']"></span>
                <span class="badge badge-secondary only_left_radius">Minimum </span> <span class="badge badge-danger only_right_radius" t-esc="question_data['numerical_min']"></span>
                <span class="badge badge-secondary only_left_radius">Average </span> <span class="badge badge-warning only_right_radius" t-esc="question_data['numerical_average']"></span>
            </span>
        </t>

        <ul class="nav nav-tabs d-print-none" role="tablist">
            <li class="nav-item">
                <a t-att-href="'#survey_stats_question_%d' % question.id"
                    t-att-aria-controls="'survey_stats_question_%d' % question.id"
                    class="nav-link active" data-toggle="tab" role="tab">
                    <i class="fa fa-list-ol"></i> Most Common
                </a>
            </li>
            <li class="nav-item">
                <a t-att-href="'#survey_data_question_%d' % question.id"
                    t-att-aria-controls="'survey_data_question_%d' % question.id"
                    class="nav-link" data-toggle="tab" role="tab">
                    <i class="fa fa-list-alt"></i> All Data
                </a>
            </li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" class="tab-pane active with-3d-shadow with-transitions" t-att-id="'survey_stats_question_%d' % question.id">
                <table class="table table-hover table-sm">
                    <thead>
                        <tr>
                            <th>User Responses</th>
                            <th>Occurrence</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="question_data['common_lines']" t-as="common_line">
                            <td>
                                <span t-if="question.question_type == 'numerical_box'" t-esc="common_line[0]"/>
                                <span t-if="question.question_type == 'date'" t-esc="common_line[0]" t-options='{"widget": "date"}'/>
                            </td>
                            <td><span t-esc="common_line[1]"></span></td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div role="tabpanel" class="tab-pane" t-att-id="'survey_data_question_%d' % question.id">
                <table class="table table-hover table-sm" t-att-id="'survey_table_question_%d' % question.id">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>User Responses</th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="table_data" t-as="input_line">
                            <tr t-if="not input_line.skipped" t-att-class="'d-none' if input_line_index >= page_record_limit else ''">
                                <td>
                                    <t t-if="no_print_url"><t t-esc="input_line_index + 1"></t></t>
                                    <t t-else="">
                                        <a t-att-href="input_line.user_input_id.get_print_url()">
                                            <t t-esc="input_line_index + 1"></t>
                                        </a>
                                    </t>
                                </td>
                                <td>
                                    <a t-if="question.question_type == 'numerical_box'" t-att-href="input_line.user_input_id.get_print_url()" t-field="input_line.value_numerical_box"/>
                                    <a t-if="question.question_type == 'date'" t-att-href="input_line.user_input_id.get_print_url()" t-field="input_line.value_date"/>
                                </td>
                            </tr>
                        </t>
                    </tbody>
                </table>
               <t t-call="survey.question_table_pagination"/>
            </div>
        </div>
    </template>

    <template id="question_result_choice" name="Question: choice result (simple_choice, multiple_choice)">
        <ul class="nav nav-tabs d-print-none" role="tablist">
            <li class="nav-item">
                <a t-att-href="'#survey_graph_question_%d' % question.id"
                    t-att-aria-controls="'survey_graph_question_%d' % question.id"
                    class="nav-link active" data-toggle="tab" role="tab">
                    <i class="fa fa-bar-chart-o"></i>
                    <span t-if="question.question_type == 'simple_choice'">Pie Graph</span>
                    <span t-else="">Graph</span>
                </a>
            </li>
            <li class="nav-item">
                <a t-att-href="'#survey_data_question_%d' % question.id"
                    t-att-aria-controls="'survey_data_question_%d' % question.id"
                    class="nav-link" data-toggle="tab" role="tab">
                    <i class="fa fa-list-alt"></i> Data
                </a>
            </li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" class="tab-pane active survey_graph"
                t-att-id="'survey_graph_question_%d' % question.id"
                t-att-data-question_id="question.id" 
                t-att-data-graph-type="'pie' if question.question_type == 'simple_choice' else 'bar'"
                t-att-data-graph-data="graph_data">
                <!-- canvas element for drawing bar chart -->
                <canvas/>
            </div>
            <div role="tabpanel" class="tab-pane" t-att-id="'survey_data_question_%d' % question.id">
                <table class="table table-hover table-sm">
                    <thead>
                        <tr>
                            <th>Answer</th>
                            <th>User Choice</th>
                            <th t-if="question_scoring">Score</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="table_data" t-as="choice_data">
                            <td>
                                <p t-esc="choice_data['value']"></p>
                            </td>
                            <td class="survey_answer">
                                <span t-esc="round(choice_data['count'] * 100.0/ (len(question_data['answer_line_done_ids']) or 1), 2)"></span> %
                                <span t-esc="'%s Votes' % choice_data['count']" class="badge badge-primary"/>
                                <i t-if="choice_data['suggested_answer'].id"
                                    class="fa fa-filter text-primary survey_filter"
                                    t-att-data-question_id="question.id"
                                    t-att-data-answer_id="choice_data['suggested_answer'].id"
                                    role="img" aria-label="Filter question" title="Filter question"/>
                            </td>
                            <td t-if="question_scoring" t-esc="choice_data['suggested_answer'].answer_score"/>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
        <div t-if="comment_lines" t-call="survey.question_result_comments" />
    </template>

    <template id="question_result_matrix" name="Question: matrix result (matrix)">
        <t t-set="graph_data" t-value="question_data['graph_data']"/>
        <t t-set="table_data" t-value="question_data['table_data']"/>
        <ul class="nav nav-tabs d-print-none" role="tablist">
            <li class="nav-item">
                <a t-att-href="'#survey_graph_question_%d' % question.id"
                    t-att-aria-controls="'survey_graph_question_%d' % question.id"
                    class="nav-link active" data-toggle="tab" role="tab">
                    <i class="fa fa-bar-chart"></i> Graph
                </a>
            </li>
            <li class="nav-item">
                <a t-att-href="'#survey_data_question_%d' % question.id"
                    t-att-aria-controls="'survey_data_question_%d' % question.id"
                    class="nav-link" data-toggle="tab" role="tab">
                    <i class="fa fa-list-alt"></i> Data
                </a>
            </li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" class="tab-pane active with-3d-shadow with-transitions survey_graph"
                t-att-id="'survey_graph_question_%d' % question.id"
                t-att-data-question_id= "question.id"
                data-graph-type= "multi_bar"
                t-att-data-graph-data="graph_data">
                <!-- canvas element for drawing Multibar chart -->
                <canvas/>
            </div>
            <div role="tabpanel" class="tab-pane" t-att-id="'survey_data_question_%d' % question.id">
                <table class="table table-hover table-sm text-right">
                    <thead t-if="table_data">
                        <tr>
                            <th></th>
                            <th class="text-right" t-foreach="table_data[0]['columns']" t-as="column_data">
                                <span t-esc="column_data['suggested_answer'].value"></span>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="table_data" t-as="choice_data">
                            <td>
                                <span t-esc="choice_data['row'].value"></span>
                            </td>
                            <td class="survey_answer" t-foreach="choice_data['columns']" t-as="column_data">
                                <span t-esc="round(column_data['count'] * 100.0/ (len(question_data['answer_input_done_ids']) or 1), 2)"></span> %
                                <span class="badge badge-primary" t-esc="column_data['count']"></span>
                                <i class="fa fa-filter text-primary survey_filter"
                                    t-att-data-question_id="question.id"
                                    t-att-data-row_id="choice_data['row'].id"
                                    t-att-data-answer_id="column_data['suggested_answer'].id" role="img" aria-label="Survey filter" title="Survey filter"></i>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        <div t-if="comment_lines" t-call="survey.question_result_comments" />
        </div>
    </template>

    <template id="question_result_comments" name="Question: comments">
        <table class="table table-hover table-sm" t-att-id="'survey_table_question_%d' % question.id">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Comment</th>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="comment_lines" t-as="input_line">
                    <td>
                        <t t-if="no_print_url"><t t-esc="input_line_index + 1"></t></t>
                        <t t-else="">
                            <a t-att-href="input_line.user_input_id.get_print_url()">
                                <t t-esc="input_line_index + 1"></t>
                            </a>
                        </t>
                    </td>
                    <td>
                        <span t-field="input_line.value_char_box"></span><br/>
                    </td>
                </tr>
            </tbody>
        </table>
    </template>

    <template id="question_table_pagination" name="Survey: statistics table pagination">
        <ul t-att-id="'pagination_%d' % question.id" class="pagination" t-att-data-question_id="question.id" t-att-data-record_limit="page_record_limit">
            <t t-if="len(table_data) > page_record_limit">
                <t t-set="total" t-value="ceil(len(table_data) / page_record_limit) + 1"/>
                <li t-foreach="range(1, total)" t-as="num"
                    t-att-class="'page-item o_survey_js_results_pagination %s' % ('active' if num == 1 else '')">
                    <a href="#" class="page-link" t-esc="num"></a>
                </li>
            </t>
        </ul>
    </template>
</data>
</odoo>

```

## File: views\survey_templates_user_input_session.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <template id="user_input_session" name="Survey User Input Session" inherit_id="web.frontend_layout" primary="True">
        <xpath expr="//div[@id='wrapwrap']" position="before">
            <!--TODO DBE Fix me : If one day, there is a survey_livechat bridge module, put this in that module-->
            <t t-set="no_livechat" t-value="True"/>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']" position="attributes">
            <attribute name="t-att-style" add="('height: 100%; overflow: auto; background: url(' + '/web/image/survey.survey/%s/background_image' % survey.id + ') no-repeat fixed center; box-shadow: inset 0 0 0 10000px rgba(255,255,255,.7); background-size: cover;') if survey and survey.background_image else 'height: 100%; overflow: auto;'"/>
        </xpath>
        <xpath expr="//head/t[@t-call-assets][last()]" position="after">
            <t t-call-assets="survey.survey_assets" lazy_load="True"/>
            <t t-call-assets="survey.survey_user_input_session_assets" lazy_load="True"/>
        </xpath>
        <xpath expr="//header" position="before">
            <t t-set="no_header" t-value="True"/>
            <t t-set="no_footer" t-value="True"/>
        </xpath>
        <xpath expr="//header" position="after">
            <div id="wrap" class="oe_structure oe_empty"/>
        </xpath>
    </template>

    <template id="user_input_session_open" name="Survey: Open Session">
        <t t-call="survey.user_input_session">
            <div class="wrap py-3 min-vh-100 align-items-center justify-content-center d-flex o_survey_session_open o_survey_session_manage"
                t-att-data-survey-access-token="survey.access_token"
                t-att-data-survey-id="survey.id"
                t-att-data-is-start-screen="True">
                <div class="w-75 p-4">
                    <div class="text-center">
                        <h1 class="mb-4" t-field="survey.title" />
                        <h2 class="mb-5 font-weight-normal">
                            <span>Go to <a t-att-href="survey.session_link" t-esc="survey.session_link" target="_blank" /></span>
                            <i class="fa fa-copy font-weight-normal ml-3 o_survey_session_copy" />
                            <input class="o_survey_session_copy_url d-none" type="text" t-att-value="survey.session_link" />
                        </h2>
                        <h2 class="font-weight-normal"><span>Waiting for attendees...</span>
                            <span class="ml-1 font-weight-bold o_survey_session_attendees_count" t-esc="survey.session_answer_count" />
                        </h2>
                    </div>
                    <a role="button"
                        class="font-weight-bold fa fa-chevron-right o_survey_session_navigation o_survey_session_navigation_next p-3" />
                </div>
            </div>
        </t>
    </template>

    <template id="user_input_session_manage" name="Survey: Manage Session">
        <t t-call="survey.user_input_session">
            <t t-call="survey.user_input_session_manage_content" />
        </t>
    </template>

    <template id="user_input_session_manage_content" name="Survey User Input Session Manage">
        <t t-set="question" t-value="survey.session_question_id" />
        <t t-set="is_scored_question" t-value="any(answer.answer_score for answer in question.suggested_answer_ids)" />
        <t t-set="show_bar_chart" t-value="question.question_type in ['simple_choice', 'multiple_choice']" />
        <t t-set="show_text_answers" t-value="question.question_type in ['char_box', 'date', 'datetime'] and not question.save_as_email and not question.save_as_nickname" />
        <div class="wrap min-vh-100 align-items-center justify-content-center d-flex flex-column o_survey_session_manage"
            t-att-style="'display: none;' if is_rpc_call else ''"
            t-att-data-is-rpc-call="is_rpc_call"
            t-att-data-survey-id="survey.id"
            t-att-data-attendees-count="survey.session_answer_count"
            t-att-data-survey-access-token="survey.access_token"
            t-att-data-timer="survey.session_question_start_time.isoformat()"
            t-att-data-time-limit-minutes="question.time_limit / 60"
            t-att-data-is-scored-question="is_scored_question"
            t-att-data-session-show-leaderboard="survey.session_show_leaderboard"
            t-att-data-question-statistics="question_statistics_graph"
            t-att-data-question-type="question.question_type"
            t-att-data-has-correct-answers="any(answer.is_correct for answer in question.suggested_answer_ids)"
            t-att-data-answers-validity="answers_validity"
            t-att-data-is-first-question="is_first_question"
            t-att-data-is-last-question="is_last_question"
            t-att-data-current-screen="'question' if is_scored_question else 'userInputs'"
            t-att-data-show-bar-chart="show_bar_chart"
            t-att-data-show-text-answers="show_text_answers">
            <div class="o_survey_question_header flex-wrap px-3 w-100 d-flex justify-content-between align-items-center position-absolute">
                <h3>
                    <span>Go to <a t-att-href="survey.session_link" t-esc="survey.session_link" target="_blank" /></span>
                    <i class="fa fa-copy font-weight-normal ml-3 mr-1 o_survey_session_copy" />
                    <input class="o_survey_session_copy_url d-none" type="text" t-att-value="survey.session_link" />
                </h3>
                <h1 t-if="question.is_time_limited" class="o_survey_timer_container">
                    <span class="o_survey_timer d-inline-block"/>
                </h1>
                <div class="text-right d-flex flex-column justify-content-center">
                    <div t-if="show_bar_chart or show_text_answers">
                        <div class="progress" title="Attendees are answering the question...">
                            <div class="progress-bar o_survey_session_progress_small font-weight-bold"
                                role="progressbar" aria-valuenow="0" aria-valuemin="0" aria-valuemax="100">
                                <span class="px-2">0 / <t t-esc="survey.session_answer_count" /></span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="container px-4 pb-3 pt96 d-flex flex-column o_survey_session_manage_container">
                <a role="button"
                    class="font-weight-bold fa fa-chevron-right o_survey_session_navigation o_survey_session_navigation_next p-3" />
                <a role="button"
                    class="font-weight-bold fa fa-chevron-left o_survey_session_navigation o_survey_session_navigation_previous p-3" />
                <div class="o_survey_session_results flex-column flex-grow-1">
                    <div class="row">
                        <div class="col-lg-12"><h1 t-esc="question.title"></h1></div>
                    </div>
                    <div t-attf-class="d-flex flex-column flex-grow-1 #{'justify-content-center' if not show_text_answers else ''} #{'align-items-center' if show_bar_chart else ''}">
                        <!-- Has to stay in 'style' attribute for Chartjs -->
                        <div t-if="show_bar_chart" class="p-2 o_survey_session_chart"
                            style="position: relative; width: 75vw; height: 70vh;">
                            <!-- canvas element for drawing bar chart -->
                            <canvas />
                        </div>
                        <div t-elif="show_text_answers" class="p-2 pt-4 o_survey_session_text_answers_container">
                        </div>
                        <div t-elif="question.is_page and not is_html_empty(question.description)" class="mb-6 o_survey_manage_fontsize_14" t-field="question.description" />
                        <div t-else="" class="mb-6">
                            <h2 class="font-weight-normal mb-3">
                                <span>Waiting for attendees...</span>
                                <span>
                                    <span class="o_survey_session_answer_count">0</span>
                                     / 
                                    <span t-esc="survey.session_answer_count" />
                                </span>
                            </h2>
                            <div class="progress">
                                <div class="progress-bar font-weight-bold" role="progressbar" aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
                            </div>
                            <fieldset disabled="disabled" class="mt-5" t-if="question.question_type == 'matrix'" t-call="survey.question_container">
                                <t t-set="hide_question_title" t-value="true" />
                                <t t-set="answer" t-value="env['survey.user_input']" />
                                <t t-set="survey_form_readonly" t-value="True"/>
                            </fieldset>
                        </div>
                    </div>
                </div>
                <div class="o_survey_session_leaderboard w-100 flex-column flex-grow-1" style="display: none;">
                    <div class="d-flex">
                        <h1 class="o_survey_session_leaderboard_title flex-grow-1">
                            <span t-if="is_last_question">Final Leaderboard</span>
                            <span t-else="">Leaderboard</span>
                        </h1>
                        <div t-att-class="'o_survey_leaderboard_buttons font-weight-bold %s' % 'd-none' if not is_last_question else ''">
                            <a href="#" role="button" class="o_survey_session_close btn btn-primary mr-4"><i class="fa fa-close"/> Close</a>
                            <a href="#" role="button" class="o_survey_session_close btn btn-primary" t-att-data-show-results="True"><i class="fa fa-bar-chart"/> Results</a>
                        </div>
                    </div>
                    <div class="justify-content-center d-flex flex-column flex-grow-1 mt-5 mb-5 pb-5 o_survey_session_leaderboard_container"/>
                </div>
            </div>
        </div>
    </template>

    <template id="user_input_session_leaderboard" name="Survey User Input Leaderboard">
        <div t-if="leaderboard" class="position-relative mb-5" t-attf-style="height: calc(3.8rem * #{len(leaderboard)});">
            <t t-set="max_score" t-value="max(score.get('scoring_total', 1) for score in leaderboard) or 1" />
            <t t-set="max_updated_score" t-value="max(score.get('updated_score', 1) for score in leaderboard)" />
            <t t-foreach="leaderboard" t-as="score">
                <div class="o_survey_session_leaderboard_item ml-2 d-flex position-absolute"
                    t-attf-style="top: calc(#{score_index} * 3.8rem);"
                    t-att-data-current-position="str(score_index)"
                    t-att-data-new-position="str(score.get('leaderboard_position', score_index))"
                    t-att-data-question-score="str(round(score.get('question_score', 0)))"
                    t-att-data-current-score="str(round(score.get('scoring_total', 0)))"
                    t-att-data-updated-score="str(round(score.get('updated_score', 0)))"
                    t-att-data-max-question-score="round(score.get('max_question_score', 1))"
                    t-att-data-max-updated-score="round(max_updated_score)">
                    <div class="d-inline-block font-weight-bold align-top">
                        <div class="d-inline-block mr-2 o_survey_session_leaderboard_score" t-esc="'%.0f p' % score['scoring_total']" />
                    </div>
                    <!-- We keep "18rem" of space to display the points / nickname.
                    Then, the length of the bar is a percentage of the attendee's score compared to the max_score. -->
                    <t t-set="width_ratio" t-value="round(round(score['scoring_total']) / round(max_score), 3)"/>
                    <t t-set="width_ratio_question" t-value="str(round(round(score.get('question_score', 0)) / round(score.get('max_question_score', 1)), 3))"/>
                    <div class="o_survey_session_leaderboard_bar ml-2 align-top d-inline-block text-right font-weight-bold"
                        t-att-style="'width: calc(calc(%s - 18rem) * %s)' % ('100%', width_ratio)"
                        t-att-data-width-ratio="width_ratio">
                    </div>
                    <div class="o_survey_session_leaderboard_bar_question mr-2 align-top d-inline-block text-right font-weight-bold position-relative"
                        style="width: 0px;"
                        t-att-data-width-ratio="width_ratio_question"
                        t-att-data-max-question-score="str(score.get('max_question_score', 1))"
                        t-att-data-question-score="str(score.get('question_score', 0))">
                        <div class="o_survey_session_leaderboard_bar_question_score position-absolute"></div>
                    </div>
                    <div class="o_survey_session_leaderboard_name d-inline-block">
                        <span t-if="score.get('nickname')" t-esc="score['nickname']" />
                        <span t-else="">Anonymous</span>
                    </div>
                </div>
            </t>
        </div>
    </template>
</data>
</odoo>

```

## File: views\survey_user_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- USER INPUTS -->
    <record id="survey_user_input_view_search" model="ir.ui.view">
        <field name="name">survey.user_input.view.search</field>
        <field name="model">survey.user_input</field>
        <field name="arch" type="xml">
            <search string="Search Survey">
                <field name="survey_id"/>
                <field name="email"/>
                <field name="partner_id"/>
                <filter name="completed" string="Completed" domain="[('state', '=', 'done')]"/>
                <filter string="In Progress" name="in_progress" domain="[('state', '=', 'in_progress')]"/>
                <filter string="New" name="new" domain="[('state', '=', 'new')]"/>
                <separator/>
                <filter string="Quizz passed" name="scoring_success" domain="[('scoring_success','=', True)]"/>
                <separator/>
                <filter string="Test Entries" name="test" domain="[('test_entry','=', True)]"/>
                <filter string="Except Test Entries" name="not_test" domain="[('test_entry','=', False)]" invisible="1"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_survey" string="Survey" domain="[]" context="{'group_by': 'survey_id'}"/>
                    <filter string="Email" name="group_by_email" domain="[]" context="{'group_by': 'email'}"/>
                    <filter string="Partner" name="group_by_partner" domain="[]" context="{'group_by': 'partner_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="survey_user_input_view_form" model="ir.ui.view">
        <field name="name">survey.user_input.view.form</field>
        <field name="model">survey.user_input</field>
        <field name="arch" type="xml">
            <form string="Survey User inputs" create="false">
                <header>
                    <button name="action_resend" string="Resend Invitation" type="object" class="oe_highlight"
                        attrs="{'invisible': ['|', ('state', '=', 'done'), '&amp;', ('partner_id', '=', False), ('email', '=', False)]}"/>
                    <button name="action_print_answers" states="done" string="Print" type="object"  class="oe_highlight"/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <group col="2">
                        <group>
                            <field name="survey_id"/>
                            <field name="create_date"/>
                            <field name="is_attempts_limited" invisible="1"/>
                            <label for="attempts_number" string="Attempt n°" attrs="{'invisible': ['|', ('is_attempts_limited', '=', False), '|', ('test_entry', '=', True), ('state', '!=', 'done')]}"/>
                            <div attrs="{'invisible': ['|', ('is_attempts_limited', '=', False), '|', ('test_entry', '=', True), ('state', '!=', 'done')]}">
                                <field name="attempts_number" nolabel="1"/>
                                 / 
                                <field name="attempts_limit" nolabel="1" />
                            </div>
                            <field name="access_token" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="deadline"/>
                            <field name="partner_id"/>
                            <field name="email" widget="email"/>
                            <field name="test_entry" groups="base.group_no_one"/>
                            <field name="scoring_type" invisible="1"/>
                            <field name="scoring_percentage" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}"/>
                            <field name="scoring_success" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}"/>
                        </group>
                    </group>
                    <field name="user_input_line_ids" mode="tree" attrs="{'readonly': False}">
                        <tree>
                            <field name="question_sequence" invisible="1"/>
                            <field name="question_id"/>
                            <field name="page_id"/>
                            <field name="answer_type"/>
                            <field name="skipped"/>
                            <field name="create_date"/>
                            <field name="answer_is_correct"/>
                            <field name="answer_score"/>
                        </tree>
                    </field>
                </sheet>
            </form>
        </field>
    </record>

    <record id="survey_user_input_view_tree" model="ir.ui.view">
        <field name="name">survey.user_input.view.tree</field>
        <field name="model">survey.user_input</field>
        <field name="arch" type="xml">
            <tree string="Survey User inputs" decoration-muted="test_entry == True" create="false">
                <field name="survey_id"/>
                <field name="create_date"/>
                <field name="deadline"/>
                <field name="partner_id"/>
                <field name="email"/>
                <field name="attempts_number"/>
                <field name="state"/>
                <field name="test_entry" invisible="True"/>
                <field name="scoring_success"/>
                <field name="scoring_percentage"/>
            </tree>
        </field>
    </record>

    <record id="survey_user_input_viuew_kanban" model="ir.ui.view">
        <field name="name">survey.user_input.view.kanban</field>
        <field name="model">survey.user_input</field>
        <field name="arch" type="xml">
            <kanban create="false">
                <field name="survey_id"/>
                <field name="create_date"/>
                <field name="partner_id"/>
                <field name="email"/>
                <field name="state"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title"><t t-esc="record.survey_id.value"/></strong>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left">
                                    <field name="create_date"/>
                                </div>
                                <div class="oe_kanban_bottom_right mr4">
                                    <field name="state" widget="label_selection" options="{'classes': {'new': 'default', 'done': 'success', 'in_progress':'warning'}}"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_survey_user_input">
        <field name="name">Participations</field>
        <field name="res_model">survey.user_input</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="view_id" ref="survey_user_input_view_tree"></field>
        <field name="search_view_id" ref="survey_user_input_view_search"/>
        <field name="context">{'search_default_group_by_survey': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            Nobody has replied to your surveys yet
          </p>
        </field>
    </record>

    <!-- USER INPUT LINES
        .. note:: these views are useful mainly for technical users/administrators -->
    <record id="survey_user_input_line_view_form" model="ir.ui.view">
        <field name="name">survey.user_input.line.view.form</field>
        <field name="model">survey.user_input.line</field>
        <field name="arch" type="xml">
            <form string="User input line details" create="false">
                <sheet>
                    <group col="4">
                        <field name="question_id"/>
                        <field name="create_date"/>
                        <field name="answer_type"/>
                        <field name="skipped" />
                        <field name="answer_score" groups="base.group_no_one"/>
                    </group>
                    <group>
                        <field name="value_char_box" colspan='2' attrs="{'invisible': [('answer_type','!=','char_box')]}"/>
                        <field name="value_numerical_box" colspan='2' attrs="{'invisible': [('answer_type','!=','numerical_box')]}"/>
                        <field name="value_date" colspan='2' attrs="{'invisible': [('answer_type','!=','date')]}"/>
                        <field name="value_datetime" colspan='2' attrs="{'invisible': [('answer_type','!=','datetime')]}"/>
                        <field name="value_text_box" colspan='2' attrs="{'invisible': [('answer_type','!=','text_box')]}"/>
                        <field name="matrix_row_id" colspan='2' />
                        <field name="suggested_answer_id" colspan='2' attrs="{'invisible': [('answer_type','!=','suggestion')]}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="survey_response_line_view_tree" model="ir.ui.view">
        <field name="name">survey.user_input.line.view.tree</field>
        <field name="model">survey.user_input.line</field>
        <field name="arch" type="xml">
            <tree string="Survey Answer Line" create="false">
                <field name="survey_id"/>
                <field name="user_input_id"/>
                <field name="question_id"/>
                <field name="create_date"/>
                <field name="answer_type"/>
                <field name="skipped"/>
                <field name="answer_score" groups="base.group_no_one"/>
            </tree>
        </field>
    </record>
    <record id="survey_user_input_line_view_search" model="ir.ui.view">
        <field name="name">survey.user_input.line.view.search</field>
        <field name="model">survey.user_input.line</field>
        <field name="arch" type="xml">
            <search string="Search User input lines">
                <field name="user_input_id"/>
                <field name="survey_id"/>
                <group expand="1" string="Group By">
                    <filter name="group_by_survey" string="Survey" domain="[]"  context="{'group_by':'survey_id'}"/>
                    <filter name="group_by_user_input" string="User Input" domain="[]"  context="{'group_by':'user_input_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="survey_user_input_line_action" model="ir.actions.act_window">
        <field name="name">Detailed Answers</field>
        <field name="res_model">survey.user_input.line</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="survey_user_input_line_view_search"/>
        <field name="context">{'search_default_group_by_survey': True, 'search_default_group_by_user_input': True}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No user input lines found
          </p>
        </field>
    </record>

    <menuitem name="Participations"
        id="menu_survey_type_form1"
        action="action_survey_user_input"
        parent="survey_menu_user_inputs"
        sequence="1"/>
    <menuitem name="Detailed Answers"
        id="menu_survey_response_line_form"
        action="survey_user_input_line_action"
        parent="survey_menu_user_inputs"
        sequence="4"
        groups="base.group_no_one"/>
</data>
</odoo>

```

## File: wizard\survey_invite.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re
import werkzeug

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

emails_split = re.compile(r"[;,\n\r]+")


class SurveyInvite(models.TransientModel):
    _name = 'survey.invite'
    _inherit = 'mail.composer.mixin'
    _description = 'Survey Invitation Wizard'

    @api.model
    def _get_default_author(self):
        return self.env.user.partner_id

    # composer content
    attachment_ids = fields.Many2many(
        'ir.attachment', 'survey_mail_compose_message_ir_attachments_rel', 'wizard_id', 'attachment_id',
        string='Attachments')
    # origin
    email_from = fields.Char(
        'From', compute='_compute_email_from', readonly=False, store=True,
        help="Email address of the sender.")
    author_id = fields.Many2one(
        'res.partner', 'Author', index=True,
        ondelete='set null', default=_get_default_author,
        help="Author of the message.")
    # recipients
    partner_ids = fields.Many2many(
        'res.partner', 'survey_invite_partner_ids', 'invite_id', 'partner_id', string='Recipients',
        domain="""[
            '|', (survey_users_can_signup, '=', 1),
            '|', (not survey_users_login_required, '=', 1),
                 ('user_ids', '!=', False),
        ]"""
    )
    existing_partner_ids = fields.Many2many(
        'res.partner', compute='_compute_existing_partner_ids', readonly=True, store=False)
    emails = fields.Text(string='Additional emails', help="This list of emails of recipients will not be converted in contacts.\
        Emails must be separated by commas, semicolons or newline.")
    existing_emails = fields.Text(
        'Existing emails', compute='_compute_existing_emails',
        readonly=True, store=False)
    existing_mode = fields.Selection([
        ('new', 'New invite'), ('resend', 'Resend invite')],
        string='Handle existing', default='resend', required=True)
    existing_text = fields.Text('Resend Comment', compute='_compute_existing_text', readonly=True, store=False)
    # technical info
    mail_server_id = fields.Many2one('ir.mail_server', 'Outgoing mail server')
    # survey
    survey_id = fields.Many2one('survey.survey', string='Survey', required=True)
    survey_start_url = fields.Char('Survey URL', compute='_compute_survey_start_url')
    survey_access_mode = fields.Selection(related="survey_id.access_mode", readonly=True)
    survey_users_login_required = fields.Boolean(related="survey_id.users_login_required", readonly=True)
    survey_users_can_signup = fields.Boolean(related='survey_id.users_can_signup')
    deadline = fields.Datetime(string="Answer deadline")

    @api.depends('partner_ids', 'survey_id')
    def _compute_existing_partner_ids(self):
        self.existing_partner_ids = list(set(self.survey_id.user_input_ids.partner_id.ids) & set(self.partner_ids.ids))

    @api.depends('template_id.email_from')
    def _compute_email_from(self):
        if self.template_id.email_from:
            self.email_from = self.template_id.email_from
        else:
            self.email_from = self.env.user.email_formatted

    @api.depends('emails', 'survey_id')
    def _compute_existing_emails(self):
        emails = list(set(emails_split.split(self.emails or "")))
        existing_emails = self.survey_id.mapped('user_input_ids.email')
        self.existing_emails = '\n'.join(email for email in emails if email in existing_emails)

    @api.depends('existing_partner_ids', 'existing_emails')
    def _compute_existing_text(self):
        existing_text = False
        if self.existing_partner_ids:
            existing_text = '%s: %s.' % (
                _('The following customers have already received an invite'),
                ', '.join(self.mapped('existing_partner_ids.name'))
            )
        if self.existing_emails:
            existing_text = '%s\n' % existing_text if existing_text else ''
            existing_text += '%s: %s.' % (
                _('The following emails have already received an invite'),
                self.existing_emails
            )

        self.existing_text = existing_text

    @api.depends('survey_id.access_token')
    def _compute_survey_start_url(self):
        for invite in self:
            invite.survey_start_url = werkzeug.urls.url_join(invite.survey_id.get_base_url(), invite.survey_id.get_start_url()) if invite.survey_id else False

    # Overrides of mail.composer.mixin
    @api.depends('survey_id')  # fake trigger otherwise not computed in new mode
    def _compute_render_model(self):
        self.render_model = 'survey.user_input'

    @api.onchange('emails')
    def _onchange_emails(self):
        if self.emails and (self.survey_users_login_required and not self.survey_id.users_can_signup):
            raise UserError(_('This survey does not allow external people to participate. You should create user accounts or update survey access mode accordingly.'))
        if not self.emails:
            return
        valid, error = [], []
        emails = list(set(emails_split.split(self.emails or "")))
        for email in emails:
            email_check = tools.email_split_and_format(email)
            if not email_check:
                error.append(email)
            else:
                valid.extend(email_check)
        if error:
            raise UserError(_("Some emails you just entered are incorrect: %s") % (', '.join(error)))
        self.emails = '\n'.join(valid)

    @api.onchange('partner_ids')
    def _onchange_partner_ids(self):
        if self.survey_users_login_required and self.partner_ids:
            if not self.survey_id.users_can_signup:
                invalid_partners = self.env['res.partner'].search([
                    ('user_ids', '=', False),
                    ('id', 'in', self.partner_ids.ids)
                ])
                if invalid_partners:
                    raise UserError(_(
                        'The following recipients have no user account: %s. You should create user accounts for them or allow external signup in configuration.',
                        ', '.join(invalid_partners.mapped('name'))
                    ))

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('template_id') and not (values.get('body') or values.get('subject')):
                template = self.env['mail.template'].browse(values['template_id'])
                if not values.get('subject'):
                    values['subject'] = template.subject
                if not values.get('body'):
                    values['body'] = template.body_html
        return super().create(vals_list)

    @api.depends('template_id', 'partner_ids')
    def _compute_subject(self):
        for invite in self:
            langs = set(invite.partner_ids.mapped('lang')) - {False}
            if len(langs) == 1:
                invite = invite.with_context(lang=langs.pop())
            super(SurveyInvite, invite)._compute_subject()

    @api.depends('template_id', 'partner_ids')
    def _compute_body(self):
        for invite in self:
            langs = set(invite.partner_ids.mapped('lang')) - {False}
            if len(langs) == 1:
                invite = invite.with_context(lang=langs.pop())
            super(SurveyInvite, invite)._compute_body()

    # ------------------------------------------------------
    # Wizard validation and send
    # ------------------------------------------------------

    def _prepare_answers(self, partners, emails):
        answers = self.env['survey.user_input']
        existing_answers = self.env['survey.user_input'].search([
            '&', ('survey_id', '=', self.survey_id.id),
            '|',
            ('partner_id', 'in', partners.ids),
            ('email', 'in', emails)
        ])
        partners_done = self.env['res.partner']
        emails_done = []
        if existing_answers:
            if self.existing_mode == 'resend':
                partners_done = existing_answers.mapped('partner_id')
                emails_done = existing_answers.mapped('email')

                # only add the last answer for each user of each type (partner_id & email)
                # to have only one mail sent per user
                for partner_done in partners_done:
                    answers |= next(existing_answer for existing_answer in
                        existing_answers.sorted(lambda answer: answer.create_date, reverse=True)
                        if existing_answer.partner_id == partner_done)

                for email_done in emails_done:
                    answers |= next(existing_answer for existing_answer in
                        existing_answers.sorted(lambda answer: answer.create_date, reverse=True)
                        if existing_answer.email == email_done)

        for new_partner in partners - partners_done:
            answers |= self.survey_id._create_answer(partner=new_partner, check_attempts=False, **self._get_answers_values())
        for new_email in [email for email in emails if email not in emails_done]:
            answers |= self.survey_id._create_answer(email=new_email, check_attempts=False, **self._get_answers_values())

        return answers

    def _get_answers_values(self):
        return {
            'deadline': self.deadline,
        }

    def _send_mail(self, answer):
        """ Create mail specific for recipient containing notably its access token """
        email_from = self._render_field('email_from', answer.ids)[answer.id]
        if not email_from:
            raise UserError(_("Unable to post message, please configure the sender's email address."))
        subject = self._render_field('subject', answer.ids, options={'render_safe': True})[answer.id]
        body = self._render_field('body', answer.ids, post_process=True)[answer.id]
        # post the message
        mail_values = {
            'email_from': email_from,
            'author_id': self.author_id.id,
            'model': None,
            'res_id': None,
            'subject': subject,
            'body_html': body,
            'attachment_ids': [(4, att.id) for att in self.attachment_ids],
            'auto_delete': True,
        }
        if answer.partner_id:
            mail_values['recipient_ids'] = [(4, answer.partner_id.id)]
        else:
            mail_values['email_to'] = answer.email

        # optional support of notif_layout in context
        notif_layout = self.env.context.get('notif_layout', self.env.context.get('custom_layout'))
        if notif_layout:
            try:
                template = self.env.ref(notif_layout, raise_if_not_found=True)
            except ValueError:
                _logger.warning('QWeb template %s not found when sending survey mails. Sending without layouting.' % (notif_layout))
            else:
                template_ctx = {
                    'message': self.env['mail.message'].sudo().new(dict(body=mail_values['body_html'], record_name=self.survey_id.title)),
                    'model_description': self.env['ir.model']._get('survey.survey').display_name,
                    'company': self.env.company,
                }
                body = template._render(template_ctx, engine='ir.qweb', minimal_qcontext=True)
                mail_values['body_html'] = self.env['mail.render.mixin']._replace_local_links(body)

        return self.env['mail.mail'].sudo().create(mail_values)

    def action_invite(self):
        """ Process the wizard content and proceed with sending the related
            email(s), rendering any template patterns on the fly if needed """
        self.ensure_one()
        Partner = self.env['res.partner']

        # compute partners and emails, try to find partners for given emails
        valid_partners = self.partner_ids
        langs = set(valid_partners.mapped('lang')) - {False}
        if len(langs) == 1:
            self = self.with_context(lang=langs.pop())
        valid_emails = []
        for email in emails_split.split(self.emails or ''):
            partner = False
            email_normalized = tools.email_normalize(email)
            if email_normalized:
                limit = None if self.survey_users_login_required else 1
                partner = Partner.search([('email_normalized', '=', email_normalized)], limit=limit)
            if partner:
                valid_partners |= partner
            else:
                email_formatted = tools.email_split_and_format(email)
                if email_formatted:
                    valid_emails.extend(email_formatted)

        if not valid_partners and not valid_emails:
            raise UserError(_("Please enter at least one valid recipient."))

        answers = self._prepare_answers(valid_partners, valid_emails)
        for answer in answers:
            self._send_mail(answer)

        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\survey_invite_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.ui.view" id="survey_invite_view_form">
            <field name="name">survey.invite.view.form</field>
            <field name="model">survey.invite</field>
            <field name="arch" type="xml">
                <form string="Compose Email" class="o_mail_composer_form">
                    <group col="1">
                        <group col="2">
                            <field name="survey_access_mode" invisible="1"/>
                            <field name="survey_users_login_required" invisible="1"/>
                            <field name="survey_users_can_signup" invisible="1"/>
                            <field name="survey_id" readonly="context.get('default_survey_id')"/>
                            <field name="existing_mode" widget="radio" invisible="1" />
                            <field name="render_model" invisible="1"/>
                            <field name="survey_start_url" label="Public share URL" readonly="1" widget="CopyClipboardChar"
                                 attrs="{'invisible':[('survey_access_mode', '!=', 'public')]}"
                                 class="mb16"/>
                            <field name="partner_ids"
                                widget="many2many_tags_email"
                                placeholder="Add existing contacts..."
                                context="{'force_email':True, 'show_email':True, 'no_create_edit': True}"/>
                            <field name="emails"
                                attrs="{
                                    'invisible': [('survey_users_login_required', '=', True)],
                                }"
                                placeholder="Add a list of email of recipients (will not be converted into contacts). Separated by commas, semicolons or newline..."/>
                        </group>
                        <div col="2" class="alert alert-warning" role="alert"
                            attrs="{'invisible': ['|', ('survey_access_mode', '=', 'public'), ('existing_text', '=', False)]}">
                            <field name="existing_text"/>
                            <group col="2">
                                <label for="existing_mode" string="Handle existing"/>
                                <div>
                                    <field name="existing_mode" nolabel="1"/>
                                    <p attrs="{'invisible': [('existing_mode', '!=', 'resend')]}">Customers will receive the same token.</p>
                                    <p attrs="{'invisible': [('existing_mode', '!=', 'new')]}">Customers will receive a new token and be able to completely retake the survey.</p>
                                </div>
                            </group>
                            <field name="existing_partner_ids" invisible="1"/>
                            <field name="existing_emails" invisible="1"/>
                        </div>
                        <group col="2">
                            <field name="subject" placeholder="Subject..."/>
                        </group>
                        <field name="can_edit_body" invisible="1"/>
                        <field name="body" class="oe-bordered-editor" options="{'style-inline': true, 'height': 380}" attrs="{'readonly': [('can_edit_body', '=', False)]}" force_save="1"/>
                        <group>
                            <group>
                                <field name="attachment_ids" widget="many2many_binary"/>
                            </group>
                            <group>
                                <field name="deadline"/>
                                <field name="template_id" label="Use template"/>
                            </group>
                        </group>
                    </group>
                    <footer>
                        <button string="Send" name="action_invite" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import survey_invite

```

