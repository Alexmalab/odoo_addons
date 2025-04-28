# Odoo Module: survey

Category: Marketing/Survey

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
    'version': '3.0',
    'category': 'Marketing/Survey',
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
    'website': 'https://www.odoo.com/page/survey',
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
        'views/assets.xml',
        'views/survey_menus.xml',
        'views/survey_survey_views.xml',
        'views/survey_user_views.xml',
        'views/survey_question_views.xml',
        'views/survey_templates.xml',
        'views/gamification_badge_views.xml',
        'wizard/survey_invite_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/survey_demo_user.xml',
        'data/survey_demo_feedback.xml',
        'data/survey_demo_certification.xml',
        'data/survey.user_input_line.csv'
    ],
    'installable': True,
    'auto_install': False,
    'application': True,
    'sequence': 105,
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

from datetime import datetime
from dateutil.relativedelta import relativedelta
from math import ceil

from odoo import fields, http, _
from odoo.addons.base.models.ir_ui_view import keep_query
from odoo.exceptions import UserError
from odoo.http import request, content_disposition
from odoo.tools import ustr

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
                ('token', '=', answer_token)
            ], limit=1)
        return survey_sudo, answer_sudo

    def _check_validity(self, survey_token, answer_token, ensure_token=True):
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
         * answer_done: token linked to a finished answer;
         * answer_deadline: token linked to an expired answer;

        :param ensure_token: whether user input existence based on given access token
          should be enforced or not, depending on the route requesting a token or
          allowing external world calls;
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

        if (survey_sudo.state == 'closed' or survey_sudo.state == 'draft' or not survey_sudo.active) and (not answer_sudo or not answer_sudo.test_entry):
            return 'survey_closed'

        if (not survey_sudo.page_ids and survey_sudo.questions_layout == 'page_per_section') or not survey_sudo.question_ids:
            return 'survey_void'

        if answer_sudo and answer_sudo.state == 'done':
            return 'answer_done'

        if answer_sudo and answer_sudo.deadline and answer_sudo.deadline < datetime.now():
            return 'answer_deadline'

        return True

    def _get_access_data(self, survey_token, answer_token, ensure_token=True):
        """ Get back data related to survey and user input, given the ID and access
        token provided by the route.

         : param ensure_token: whether user input existence should be enforced or not(see ``_check_validity``)
        """
        survey_sudo, answer_sudo = request.env['survey.survey'].sudo(), request.env['survey.user_input'].sudo()
        has_survey_access, can_answer = False, False

        validity_code = self._check_validity(survey_token, answer_token, ensure_token=ensure_token)
        if validity_code != 'survey_wrong':
            survey_sudo, answer_sudo = self._fetch_from_access_token(survey_token, answer_token)
            try:
                survey_user = survey_sudo.with_user(request.env.user)
                survey_user.check_access_rights(self, 'read', raise_exception=True)
                survey_user.check_access_rule(self, 'read')
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
            return request.render("survey.survey_void", {'survey': survey_sudo, 'answer': answer_sudo})
        elif error_key == 'survey_closed' and access_data['can_answer']:
            return request.render("survey.survey_expired", {'survey': survey_sudo})
        elif error_key == 'survey_auth':
            if not answer_sudo:  # survey is not even started
                redirect_url = '/web/login?redirect=/survey/start/%s' % survey_sudo.access_token
            elif answer_sudo.token:  # survey is started but user is not logged in anymore.
                if answer_sudo.partner_id and (answer_sudo.partner_id.user_ids or survey_sudo.users_can_signup):
                    if answer_sudo.partner_id.user_ids:
                        answer_sudo.partner_id.signup_cancel()
                    else:
                        answer_sudo.partner_id.signup_prepare(expiration=fields.Datetime.now() + relativedelta(days=1))
                    redirect_url = answer_sudo.partner_id._get_signup_url_for_action(url='/survey/start/%s?answer_token=%s' % (survey_sudo.access_token, answer_sudo.token))[answer_sudo.partner_id.id]
                else:
                    redirect_url = '/web/login?redirect=%s' % ('/survey/start/%s?answer_token=%s' % (survey_sudo.access_token, answer_sudo.token))
            return request.render("survey.auth_required", {'survey': survey_sudo, 'redirect_url': redirect_url})
        elif error_key == 'answer_deadline' and answer_sudo.token:
            return request.render("survey.survey_expired", {'survey': survey_sudo})
        elif error_key == 'answer_done' and answer_sudo.token:
            return request.render("survey.sfinished", self._prepare_survey_finished_values(survey_sudo, answer_sudo, token=answer_sudo.token))

        return werkzeug.utils.redirect("/")

    @http.route('/survey/test/<string:survey_token>', type='http', auth='user', website=True)
    def survey_test(self, survey_token, **kwargs):
        """ Test mode for surveys: create a test answer, only for managers or officers
        testing their surveys """
        survey_sudo, dummy = self._fetch_from_access_token(survey_token, False)
        try:
            answer_sudo = survey_sudo._create_answer(user=request.env.user, test_entry=True)
        except:
            return werkzeug.utils.redirect('/')
        return request.redirect('/survey/start/%s?%s' % (survey_sudo.access_token, keep_query('*', answer_token=answer_sudo.token)))

    @http.route('/survey/retry/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_retry(self, survey_token, answer_token, **post):
        """ This route is called whenever the user has attempts left and hits the 'Retry' button
        after failing the survey."""
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True and access_data['validity_code'] != 'answer_done':
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        if not answer_sudo:
            # attempts to 'retry' without having tried first
            return werkzeug.utils.redirect("/")

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
            return werkzeug.utils.redirect("/")
        return request.redirect('/survey/start/%s?%s' % (survey_sudo.access_token, keep_query('*', answer_token=retry_answer_sudo.token)))

    def _prepare_retry_additional_values(self, answer):
        return {
            'input_type': answer.input_type,
            'deadline': answer.deadline,
        }

    # ------------------------------------------------------------
    # TAKING SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/survey/start/<string:survey_token>', type='http', auth='public', website=True)
    def survey_start(self, survey_token, answer_token=None, email=False, **post):
        """ Start a survey by providing
         * a token linked to a survey;
         * a token linked to an answer or generate a new token if access is allowed;
        """
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=False)
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
                return werkzeug.utils.redirect("/")
            else:
                return request.render("survey.403", {'survey': survey_sudo})

        # Select the right page
        if answer_sudo.state == 'new':  # Intro page
            data = {'survey': survey_sudo, 'answer': answer_sudo, 'page': 0}
            return request.render('survey.survey_init', data)
        else:
            return request.redirect('/survey/fill/%s/%s' % (survey_sudo.access_token, answer_sudo.token))

    # Survey direct link to a specific page
    @http.route('/survey/page/<string:survey_token>/<string:answer_token>/<int:page_id>',
                type='http', auth='public', website=True)
    def survey_change_page(self, survey_token, answer_token, page_id, **post):
        """ Method called when the user switches from one page to another using the breadcrumbs links
        in the survey layout.
        TODO: Right now, the answers that are not submitted are LOST when changing from one page to another
        using this method.

        The survey "submit" mechanism needs to be refactored entirely to make this more user-friendly."""
        # Controls if the survey can be displayed
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=False)
        if access_data['validity_code'] is not True:
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        return request.render('survey.survey', {
            'survey': survey_sudo,
            'page': request.env['survey.question'].sudo().browse(page_id),
            'answer': answer_sudo
        })

    @http.route('/survey/fill/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_display_page(self, survey_token, answer_token, prev=None, **post):
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        if survey_sudo.is_time_limited and not answer_sudo.start_datetime:
            # init start date when user starts filling in the survey
            answer_sudo.write({
                'start_datetime': fields.Datetime.now()
            })

        page_or_question_key = 'question' if survey_sudo.questions_layout == 'page_per_question' else 'page'
        # Select the right page
        if answer_sudo.state == 'new':  # First page
            page_or_question_id, last = survey_sudo.next_page_or_question(answer_sudo, 0, go_back=False)
            data = {
                'survey': survey_sudo,
                page_or_question_key: page_or_question_id,
                'answer': answer_sudo
            }
            if last:
                data.update({'last': True})
            return request.render('survey.survey', data)
        elif answer_sudo.state == 'done':  # Display success message
            return request.render('survey.sfinished', self._prepare_survey_finished_values(survey_sudo, answer_sudo))
        elif answer_sudo.state == 'skip':
            flag = (True if prev and prev == 'prev' else False)
            page_or_question_id, last = survey_sudo.next_page_or_question(answer_sudo, answer_sudo.last_displayed_page_id.id, go_back=flag)

            #special case if you click "previous" from the last page, then leave the survey, then reopen it from the URL, avoid crash
            if not page_or_question_id:
                page_or_question_id, last = survey_sudo.next_page_or_question(answer_sudo, answer_sudo.last_displayed_page_id.id, go_back=True)

            data = {
                'survey': survey_sudo,
                page_or_question_key: page_or_question_id,
                'answer': answer_sudo
            }
            if last:
                data.update({'last': True})

            return request.render('survey.survey', data)
        else:
            return request.render("survey.403", {'survey': survey_sudo})

    @http.route('/survey/prefill/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_get_answers(self, survey_token, answer_token, page_or_question_id=None, **post):
        """ TDE NOTE: original comment: # AJAX prefilling of a survey -> AJAX / http ?? """
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True and access_data['validity_code'] != 'answer_done':
            return {}

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        try:
            page_or_question_id = int(page_or_question_id)
        except:
            page_or_question_id = None

        # Fetch previous answers
        if survey_sudo.questions_layout == 'one_page' or not page_or_question_id:
            previous_answers = answer_sudo.user_input_line_ids
        elif survey_sudo.questions_layout == 'page_per_section':
            previous_answers = answer_sudo.user_input_line_ids.filtered(lambda line: line.page_id.id == page_or_question_id)
        else:
            previous_answers = answer_sudo.user_input_line_ids.filtered(lambda line: line.question_id.id == page_or_question_id)

        # Return non empty answers in a JSON compatible format
        ret = {}
        for answer in previous_answers:
            if not answer.skipped:
                answer_tag = '%s_%s' % (answer.survey_id.id, answer.question_id.id)
                answer_value = None
                if answer.answer_type == 'free_text':
                    answer_value = answer.value_free_text
                elif answer.answer_type == 'text' and answer.question_id.question_type == 'textbox':
                    answer_value = answer.value_text
                elif answer.answer_type == 'text' and answer.question_id.question_type != 'textbox':
                    # here come comment answers for matrices, simple choice and multiple choice
                    answer_tag = "%s_%s" % (answer_tag, 'comment')
                    answer_value = answer.value_text
                elif answer.answer_type == 'number':
                    answer_value = str(answer.value_number)
                elif answer.answer_type == 'date':
                    answer_value = fields.Datetime.to_string(answer.value_date)
                elif answer.answer_type == 'datetime':
                    answer_value = fields.Datetime.to_string(answer.value_datetime)
                elif answer.answer_type == 'suggestion' and not answer.value_suggested_row:
                    answer_value = answer.value_suggested.id
                elif answer.answer_type == 'suggestion' and answer.value_suggested_row:
                    answer_tag = "%s_%s" % (answer_tag, answer.value_suggested_row.id)
                    answer_value = answer.value_suggested.id
                if answer_value:
                    ret.setdefault(answer_tag, []).append(answer_value)
                else:
                    _logger.warning("[survey] No answer has been found for question %s marked as non skipped" % answer_tag)
        return json.dumps(ret, default=str)

    @http.route('/survey/scores/<string:survey_token>/<string:answer_token>', type='http', auth='public', website=True)
    def survey_get_scores(self, survey_id, answer_token, page_id=None, **post):
        """ TDE NOTE: original comment: # AJAX scores loading for quiz correction mode -> AJAX / http ?? """
        access_data = self._get_access_data(survey_id, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return {}

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        # Compute score for each question
        ret = {}
        for answer in answer_sudo.user_input_line_ids:
            tmp_score = ret.get(answer.question_id.id, 0.0)
            ret.update({answer.question_id.id: tmp_score + answer.answer_score})
        return json.dumps(ret)

    @http.route('/survey/submit/<string:survey_token>/<string:answer_token>', type='http', methods=['POST'], auth='public', website=True)
    def survey_submit(self, survey_token, answer_token, **post):
        """ Submit a page from the survey.
        This will take into account the validation errors and store the answers to the questions.
        If the time limit is reached, errors will be skipped, answers wil be ignored and
        survey state will be forced to 'done'

        TDE NOTE: original comment: # AJAX submission of a page -> AJAX / http ?? """
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=True)
        if access_data['validity_code'] is not True:
            return {}

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']
        if not answer_sudo.test_entry and not survey_sudo._has_attempts_left(answer_sudo.partner_id, answer_sudo.email, answer_sudo.invite_token):
            # prevent cheating with users creating multiple 'user_input' before their last attempt
            return {}

        if survey_sudo.questions_layout == 'page_per_section':
            page_id = int(post['page_id'])
            questions = request.env['survey.question'].sudo().search([('survey_id', '=', survey_sudo.id), ('page_id', '=', page_id)])
            # we need the intersection of the questions of this page AND the questions prepared for that user_input
            # (because randomized surveys do not use all the questions of every page)
            questions = questions & answer_sudo.question_ids
            page_or_question_id = page_id
        elif survey_sudo.questions_layout == 'page_per_question':
            question_id = int(post['question_id'])
            questions = request.env['survey.question'].sudo().browse(question_id)
            page_or_question_id = question_id
        else:
            questions = survey_sudo.question_ids
            questions = questions & answer_sudo.question_ids

        errors = {}
        # Answer validation
        if not answer_sudo.is_time_limit_reached:
            for question in questions:
                answer_tag = "%s_%s" % (survey_sudo.id, question.id)
                errors.update(question.validate_question(post, answer_tag))

        ret = {}
        if len(errors):
            # Return errors messages to webpage
            ret['errors'] = errors
        else:
            if not answer_sudo.is_time_limit_reached:
                for question in questions:
                    answer_tag = "%s_%s" % (survey_sudo.id, question.id)
                    request.env['survey.user_input_line'].sudo().save_lines(answer_sudo.id, question, post, answer_tag)

            go_back = False
            vals = {}
            if answer_sudo.is_time_limit_reached or survey_sudo.questions_layout == 'one_page':
                answer_sudo._mark_done()
            elif 'button_submit' in post:
                go_back = post['button_submit'] == 'previous'
                next_page, last = request.env['survey.survey'].next_page_or_question(answer_sudo, page_or_question_id, go_back=go_back)
                vals = {'last_displayed_page_id': page_or_question_id}

                if next_page is None and not go_back:
                    answer_sudo._mark_done()
                else:
                    vals.update({'state': 'skip'})

            if 'breadcrumb_redirect' in post:
                ret['redirect'] = post['breadcrumb_redirect']
            else:
                if vals:
                    answer_sudo.write(vals)

                ret['redirect'] = '/survey/fill/%s/%s' % (survey_sudo.access_token, answer_token)
                if go_back:
                    ret['redirect'] += '?prev=prev'

        return json.dumps(ret)

    # ------------------------------------------------------------
    # COMPLETED SURVEY ROUTES
    # ------------------------------------------------------------

    @http.route('/survey/print/<string:survey_token>', type='http', auth='public', website=True, sitemap=False)
    def survey_print(self, survey_token, review=False, answer_token=None, **post):
        '''Display an survey in printable view; if <answer_token> is set, it will
        grab the answers of the user_input_id that has <answer_token>.'''
        access_data = self._get_access_data(survey_token, answer_token, ensure_token=False)
        if access_data['validity_code'] is not True and (
                access_data['has_survey_access'] or
                access_data['validity_code'] not in ['token_required', 'survey_closed', 'survey_void', 'answer_done']):
            return self._redirect_with_error(access_data, access_data['validity_code'])

        survey_sudo, answer_sudo = access_data['survey_sudo'], access_data['answer_sudo']

        return request.render('survey.survey_print', {
            'review': review,
            'survey': survey_sudo,
            'answer': answer_sudo if survey_sudo.scoring_type != 'scoring_without_answers' else answer_sudo.browse(),
            'page_nr': 0,
            'quizz_correction': survey_sudo.scoring_type != 'scoring_without_answers' and answer_sudo})

    @http.route('/survey/results/<model("survey.survey"):survey>', type='http', auth='user', website=True)
    def survey_report(self, survey, answer_token=None, **post):
        '''Display survey Results & Statistics for given survey.'''
        result_template = 'survey.result'
        current_filters = []
        filter_display_data = []
        filter_finish = False

        answers = survey.user_input_ids.filtered(lambda answer: answer.state != 'new' and not answer.test_entry)
        if 'finished' in post:
            post.pop('finished')
            filter_finish = True
        if post or filter_finish:
            filter_data = self._get_filter_data(post)
            current_filters = survey.filter_input_ids(filter_data, filter_finish)
            filter_display_data = survey.get_filter_display_data(filter_data)
        return request.render(result_template,
                                      {'survey': survey,
                                       'answers': answers,
                                       'survey_dict': self._prepare_result_dict(survey, current_filters),
                                       'page_range': self.page_range,
                                       'current_filters': current_filters,
                                       'filter_display_data': filter_display_data,
                                       'filter_finish': filter_finish
                                       })
        # Quick retroengineering of what is injected into the template for now:
        # (TODO: flatten and simplify this)
        #
        #     survey: a browse record of the survey
        #     survey_dict: very messy dict containing all the info to display answers
        #         {'page_ids': [
        #
        #             ...
        #
        #                 {'page': browse record of the page,
        #                  'question_ids': [
        #
        #                     ...
        #
        #                     {'graph_data': data to be displayed on the graph
        #                      'input_summary': number of answered, skipped...
        #                      'prepare_result': {
        #                                         answers displayed in the tables
        #                                         }
        #                      'question': browse record of the question_ids
        #                     }
        #
        #                     ...
        #
        #                     ]
        #                 }
        #
        #             ...
        #
        #             ]
        #         }
        #
        #     page_range: pager helper function
        #     current_filters: a list of ids
        #     filter_display_data: [{'labels': ['a', 'b'], question_text} ...  ]
        #     filter_finish: boolean => only finished surveys or not
        #

    @http.route(['/survey/<int:survey_id>/get_certification'], type='http', auth='user', methods=['GET'], website=True)
    def survey_get_certification(self, survey_id, **kwargs):
        """ The certification document can be downloaded as long as the user has succeeded the certification """
        survey = request.env['survey.survey'].sudo().search([
            ('id', '=', survey_id),
            ('certificate', '=', True)
        ])

        if not survey:
            # no certification found
            return werkzeug.utils.redirect("/")

        succeeded_attempt = request.env['survey.user_input'].sudo().search([
            ('partner_id', '=', request.env.user.partner_id.id),
            ('survey_id', '=', survey_id),
            ('quizz_passed', '=', True)
        ], limit=1)

        if not succeeded_attempt:
            raise UserError(_("The user has not succeeded the certification"))

        report_sudo = request.env.ref('survey.certification_report').sudo()

        report = report_sudo.render_qweb_pdf([succeeded_attempt.id], data={'report_type': 'pdf'})[0]
        reporthttpheaders = [
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(report)),
        ]
        reporthttpheaders.append(('Content-Disposition', content_disposition('Certification.pdf')))
        return request.make_response(report, headers=reporthttpheaders)

    def _prepare_result_dict(self, survey, current_filters=None):
        """Returns dictionary having values for rendering template"""
        current_filters = current_filters if current_filters else []
        result = {'page_ids': []}
        
        # First append questions without page
        questions_without_page = [self._prepare_question_values(question,current_filters) for question in survey.question_ids if not question.page_id]
        if questions_without_page:
            result['page_ids'].append({'page': request.env['survey.question'], 'question_ids': questions_without_page})

        # Then, questions in sections
        for page in survey.page_ids:
            page_dict = {'page': page, 'question_ids': [self._prepare_question_values(question,current_filters) for question in page.question_ids]}
            result['page_ids'].append(page_dict)

        if survey.scoring_type in ['scoring_with_answers', 'scoring_without_answers']:
            scoring_data = self._get_scoring_data(survey)
            result['success_rate'] = scoring_data['success_rate']
            result['scoring_graph_data'] = json.dumps(scoring_data['graph_data'])

        return result

    def _prepare_question_values(self, question, current_filters):
        Survey = request.env['survey.survey']
        return {
            'question': question,
            'input_summary': Survey.get_input_summary(question, current_filters),
            'prepare_result': Survey.prepare_result(question, current_filters),
            'graph_data': self._get_graph_data(question, current_filters),
        }

    def _get_filter_data(self, post):
        """Returns data used for filtering the result"""
        filters = []
        for ids in post:
            #if user add some random data in query URI, ignore it
            try:
                row_id, answer_id = ids.split(',')
                filters.append({'row_id': int(row_id), 'answer_id': int(answer_id)})
            except:
                return filters
        return filters

    def page_range(self, total_record, limit):
        '''Returns number of pages required for pagination'''
        total = ceil(total_record / float(limit))
        return range(1, int(total + 1))

    def _get_graph_data(self, question, current_filters=None):
        '''Returns formatted data required by graph library on basis of filter'''
        # TODO refactor this terrible method and merge it with _prepare_result_dict
        current_filters = current_filters if current_filters else []
        Survey = request.env['survey.survey']
        result = []
        if question.question_type == 'multiple_choice':
            result.append({'key': ustr(question.question),
                           'values': Survey.prepare_result(question, current_filters)['answers']
                           })
        if question.question_type == 'simple_choice':
            result = Survey.prepare_result(question, current_filters)['answers']
        if question.question_type == 'matrix':
            data = Survey.prepare_result(question, current_filters)
            for answer in data['answers']:
                values = []
                for row in data['rows']:
                    values.append({'text': data['rows'].get(row), 'count': data['result'].get((row, answer))})
                result.append({'key': data['answers'].get(answer), 'values': values})
        return json.dumps(result)

    def _get_scoring_data(self, survey):
        """Performs a read_group to fetch the count of failed/passed tests in a single query."""

        count_data = request.env['survey.user_input'].read_group(
            [('survey_id', '=', survey.id), ('state', '=', 'done'), ('test_entry', '=', False)],
            ['quizz_passed', 'id:count_distinct'],
            ['quizz_passed']
        )

        quizz_passed_count = 0
        quizz_failed_count = 0
        for count_data_item in count_data:
            if count_data_item['quizz_passed']:
                quizz_passed_count = count_data_item['quizz_passed_count']
            else:
                quizz_failed_count = count_data_item['quizz_passed_count']

        graph_data = [{
            'text': _('Passed'),
            'count': quizz_passed_count,
            'color': '#2E7D32'
        }, {
            'text': _('Missed'),
            'count': quizz_failed_count,
            'color': '#C62828'
        }]

        total_quizz_passed = quizz_passed_count + quizz_failed_count
        return {
            'success_rate': round((quizz_passed_count / total_quizz_passed) * 100, 1) if total_quizz_passed > 0 else 0,
            'graph_data': graph_data
        }

    def _prepare_survey_finished_values(self, survey, answer, token=False):
        values = {'survey': survey, 'answer': answer}
        if token:
            values['token'] = token
        if survey.scoring_type != 'no_scoring' and survey.certificate:
            answer_perf = survey._get_answers_correctness(answer)[answer]
            values['graph_data'] = json.dumps([
                {"text": "Correct", "count": answer_perf['correct']},
                {"text": "Partially", "count": answer_perf['partial']},
                {"text": "Incorrect", "count": answer_perf['incorrect']},
                {"text": "Unanswered", "count": answer_perf['skipped']}
            ])
        return values

```

## File: controllers\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

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
            <field name="subject">Participate to ${object.survey_id.title} survey</field>
            <field name="email_to">${(object.partner_id.email_formatted or object.email) |safe}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear ${object.partner_id.name or 'participant'}<br/><br/>
        % if object.survey_id.certificate:
            You have been invited to take a new certification.
        % else:
            We are conducting a survey and your response would be appreciated.
        % endif
        <div style="margin: 16px 0px 16px 0px;">
            <a href="${('%s?answer_token=%s' % (object.survey_id.public_url, object.token)) | safe}"
                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                % if object.survey_id.certificate:
                    Start Certification
                % else:
                    Start Survey
                % endif
            </a>
        </div>
        % if object.deadline:
            Please answer the survey for ${format_date(object.deadline)}.<br/><br/>
        % endif
        Thank you for your participation.
    </p>
</div>
            </field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>

        <!-- Certification Email template -->
        <record id="mail_template_certification" model="mail.template">
            <field name="name">Survey: Send certification by email</field>
            <field name="model_id" ref="survey.model_survey_user_input"/>
            <field name="subject">Certification: ${object.survey_id.display_name}</field>
            <field name="email_from">${(object.survey_id.create_uid.email_formatted or user.email_formatted or user.company_id.catchall) |safe}</field>
            <field name="email_to">${(object.partner_id.email_formatted or object.email) |safe}</field>
            <field name="body_html" type="xml">
                <div style="background:#F0F0F0;color:#515166;padding:10px 0px;font-family:Arial,Helvetica,sans-serif;font-size:14px;">
                    <table style="width:600px;margin:5px auto;">
                        <tbody>
                            <tr><td>
                                <!-- We use the logo of the company that created the survey (to handle multi company cases) -->
                                <a href="/"><img src="/logo.png?company=${object.survey_id.create_uid.company_id.id}" style="vertical-align:baseline;max-width:100px;" /></a>
                            </td><td style="text-align:right;vertical-align:middle;">
                                    Certification: ${object.survey_id.display_name}
                            </td></tr>
                        </tbody>
                    </table>
                    <table style="width:600px;margin:0px auto;background:white;border:1px solid #e1e1e1;">
                        <tbody>
                            <tr><td style="padding:15px 20px 10px 20px;">
                                <p>Dear <span>${object.partner_id.name or 'participant'}</span></p>
                                <p>
                                    Here is, in attachment, your certification document for
                                        <strong>${object.survey_id.display_name}</strong>
                                </p>
                                <p>Congratulations for succeeding the test!</p>
                            </td></tr>
                        </tbody>
                    </table>
                </div>
            </field>
            <field name="report_template" ref="certification_report"/>
            <field name="report_name">Certification Document</field>
            <field name="lang">${object.partner_id.lang}</field>
            <field name="auto_delete" eval="True"/>
            <field name="user_signature" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\survey.user_input_line.csv

```csv
id,user_input_id:id,question_id:id,skipped,answer_type,value_text,value_number,value_date,value_free_text,value_suggested:id,value_suggested_row:id
survey_answer_1_p1_q1,survey_answer_1,survey_feedback_p1_q1,False,text,Brussels,,,,,
survey_answer_1_p1_q2,survey_answer_1,survey_feedback_p1_q2,False,date,,,1980-01-11,,,
survey_answer_1_p1_q3,survey_answer_1,survey_feedback_p1_q3,False,suggestion,,,,,survey_feedback_p1_q3_sug3,
survey_answer_1_p1_q4,survey_answer_1,survey_feedback_p1_q4,False,number,,4,,,,
survey_answer_1_p2_q1_1,survey_answer_1,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug1,
survey_answer_1_p2_q1_2,survey_answer_1,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug2,
survey_answer_1_p2_q2_1,survey_answer_1,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row1
survey_answer_1_p2_q2_2,survey_answer_1,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row2
survey_answer_1_p2_q2_3,survey_answer_1,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col2,survey_feedback_p2_q2_row3
survey_answer_1_p2_q2_4,survey_answer_1,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col4,survey_feedback_p2_q2_row4
survey_answer_1_p2_q2_5,survey_answer_1,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col2,survey_feedback_p2_q2_row5
survey_answer_1_p2_q3,survey_answer_1,survey_feedback_p2_q3,False,free_text,,,,Thanks for the good quality of your products,,
survey_answer_2_p1_q1,survey_answer_2,survey_feedback_p1_q1,False,text,Paris,,,,,
survey_answer_2_p1_q2,survey_answer_2,survey_feedback_p1_q2,True,,,,,,,
survey_answer_2_p1_q3,survey_answer_2,survey_feedback_p1_q3,False,suggestion,,,,,survey_feedback_p1_q3_sug2,
survey_answer_2_p1_q4,survey_answer_2,survey_feedback_p1_q4,False,number,,10,,,,
survey_answer_2_p2_q1_1,survey_answer_2,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug2,
survey_answer_2_p2_q1_2,survey_answer_2,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug3,
survey_answer_2_p2_q1_3,survey_answer_2,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug4,
survey_answer_2_p2_q2_1,survey_answer_2,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col4,survey_feedback_p2_q2_row1
survey_answer_2_p2_q2_2,survey_answer_2,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row2
survey_answer_2_p2_q2_3,survey_answer_2,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row3
survey_answer_2_p2_q2_4,survey_answer_2,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col4,survey_feedback_p2_q2_row4
survey_answer_2_p2_q2_5,survey_answer_2,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row5
survey_answer_2_p2_q3,survey_answer_2,survey_feedback_p2_q3,False,free_text,,,,I really appreciate your products. They are awesome !!,,
survey_answer_3_p1_q1,survey_answer_3,survey_feedback_p1_q1,False,text,New York,,,,,
survey_answer_3_p1_q2,survey_answer_3,survey_feedback_p1_q2,False,date,,,1966-06-15,,,
survey_answer_3_p1_q3,survey_answer_3,survey_feedback_p1_q3,False,suggestion,,,,,survey_feedback_p1_q3_sug4,
survey_answer_3_p1_q4,survey_answer_3,survey_feedback_p1_q4,False,number,,1,,,,
survey_answer_3_p2_q1_1,survey_answer_3,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug7,
survey_answer_3_p2_q1_2,survey_answer_3,survey_feedback_p2_q1,False,suggestion,,,,,survey_feedback_p2_q1_sug8,
survey_answer_3_p2_q2_1,survey_answer_3,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col1,survey_feedback_p2_q2_row1
survey_answer_3_p2_q2_2,survey_answer_3,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col2,survey_feedback_p2_q2_row2
survey_answer_3_p2_q2_3,survey_answer_3,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col1,survey_feedback_p2_q2_row3
survey_answer_3_p2_q2_4,survey_answer_3,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col3,survey_feedback_p2_q2_row4
survey_answer_3_p2_q2_5,survey_answer_3,survey_feedback_p2_q2,False,suggestion,,,,,survey_feedback_p2_q2_col1,survey_feedback_p2_q2_row5
survey_answer_3_p2_q3,survey_answer_3,survey_feedback_p2_q3,False,free_text,,,,The customizable desk received is not the one I ordered on your website and the quality is very poor ! Really disappointed.,,
survey_vendor_certification_answer_1_p1_q1,survey_vendor_certification_answer_1,vendor_certification_page_1_question_1,False,suggestion,,,,,vendor_certification_page_1_question_1_choice_2,
survey_vendor_certification_answer_1_p1_q2_1,survey_vendor_certification_answer_1,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_1,
survey_vendor_certification_answer_1_p1_q2_2,survey_vendor_certification_answer_1,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_3,
survey_vendor_certification_answer_1_p1_q2_3,survey_vendor_certification_answer_1,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_4,
survey_vendor_certification_answer_1_p1_q3_1,survey_vendor_certification_answer_1,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_1,
survey_vendor_certification_answer_1_p1_q3_2,survey_vendor_certification_answer_1,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_3,
survey_vendor_certification_answer_1_p1_q3_3,survey_vendor_certification_answer_1,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_4,
survey_vendor_certification_answer_1_p1_q4,survey_vendor_certification_answer_1,vendor_certification_page_1_question_4,False,suggestion,,,,,vendor_certification_page_1_question_4_choice_2,
survey_vendor_certification_answer_1_p1_q5,survey_vendor_certification_answer_1,vendor_certification_page_1_question_5,False,free_text,,,,I think it misses a product but I don't know what,,
survey_vendor_certification_answer_1_p2_q1,survey_vendor_certification_answer_1,vendor_certification_page_2_question_1,False,suggestion,,,,,vendor_certification_page_2_question_1_choice_4,
survey_vendor_certification_answer_1_p2_q2_1,survey_vendor_certification_answer_1,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_1,
survey_vendor_certification_answer_1_p2_q2_2,survey_vendor_certification_answer_1,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_2,
survey_vendor_certification_answer_1_p2_q2_3,survey_vendor_certification_answer_1,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_4,
survey_vendor_certification_answer_1_p2_q3,survey_vendor_certification_answer_1,vendor_certification_page_2_question_3,False,suggestion,,,,,vendor_certification_page_2_question_3_choice_3,
survey_vendor_certification_answer_2_p1_q1,survey_vendor_certification_answer_2,vendor_certification_page_1_question_1,False,suggestion,,,,,vendor_certification_page_1_question_1_choice_2,
survey_vendor_certification_answer_2_p1_q2_1,survey_vendor_certification_answer_2,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_1,
survey_vendor_certification_answer_2_p1_q2_2,survey_vendor_certification_answer_2,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_3,
survey_vendor_certification_answer_2_p1_q3_1,survey_vendor_certification_answer_2,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_1,
survey_vendor_certification_answer_2_p1_q3_2,survey_vendor_certification_answer_2,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_3,
survey_vendor_certification_answer_2_p1_q4,survey_vendor_certification_answer_2,vendor_certification_page_1_question_4,False,suggestion,,,,,vendor_certification_page_1_question_4_choice_2,
survey_vendor_certification_answer_2_p1_q5,survey_vendor_certification_answer_2,vendor_certification_page_1_question_5,True,,,,,,,
survey_vendor_certification_answer_2_p2_q1,survey_vendor_certification_answer_2,vendor_certification_page_2_question_1,False,suggestion,,,,,vendor_certification_page_2_question_1_choice_4,
survey_vendor_certification_answer_2_p2_q2_1,survey_vendor_certification_answer_2,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_1,
survey_vendor_certification_answer_2_p2_q2_2,survey_vendor_certification_answer_2,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_2,
survey_vendor_certification_answer_2_p2_q2_3,survey_vendor_certification_answer_2,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_4,
survey_vendor_certification_answer_2_p2_q3,survey_vendor_certification_answer_2,vendor_certification_page_2_question_3,False,suggestion,,,,,vendor_certification_page_2_question_3_choice_4,
survey_vendor_certification_answer_3_p1_q1,survey_vendor_certification_answer_3,vendor_certification_page_1_question_1,False,suggestion,,,,,vendor_certification_page_1_question_1_choice_2,
survey_vendor_certification_answer_3_p1_q2_1,survey_vendor_certification_answer_3,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_1,
survey_vendor_certification_answer_3_p1_q2_2,survey_vendor_certification_answer_3,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_4,
survey_vendor_certification_answer_3_p1_q3_1,survey_vendor_certification_answer_3,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_1,
survey_vendor_certification_answer_3_p1_q3_2,survey_vendor_certification_answer_3,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_4,
survey_vendor_certification_answer_3_p1_q4,survey_vendor_certification_answer_3,vendor_certification_page_1_question_4,False,suggestion,,,,,vendor_certification_page_1_question_4_choice_2,
survey_vendor_certification_answer_3_p1_q5,survey_vendor_certification_answer_3,vendor_certification_page_1_question_5,True,,,,,,,
survey_vendor_certification_answer_3_p2_q1,survey_vendor_certification_answer_3,vendor_certification_page_2_question_1,False,suggestion,,,,,vendor_certification_page_2_question_1_choice_4,
survey_vendor_certification_answer_3_p2_q2_2,survey_vendor_certification_answer_3,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_1,
survey_vendor_certification_answer_3_p1_q2_3,survey_vendor_certification_answer_3,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_4,
survey_vendor_certification_answer_3_p2_q3,survey_vendor_certification_answer_3,vendor_certification_page_2_question_3,False,suggestion,,,,,vendor_certification_page_2_question_3_choice_2,
survey_vendor_certification_answer_4_p1_q1,survey_vendor_certification_answer_4,vendor_certification_page_1_question_1,False,suggestion,,,,,vendor_certification_page_1_question_1_choice_1,
survey_vendor_certification_answer_4_p1_q2,survey_vendor_certification_answer_4,vendor_certification_page_1_question_2,False,suggestion,,,,,vendor_certification_page_1_question_2_choice_3,
survey_vendor_certification_answer_4_p1_q3,survey_vendor_certification_answer_4,vendor_certification_page_1_question_3,False,suggestion,,,,,vendor_certification_page_1_question_3_choice_2,
survey_vendor_certification_answer_4_p1_q4,survey_vendor_certification_answer_4,vendor_certification_page_1_question_4,False,suggestion,,,,,vendor_certification_page_1_question_4_choice_4,
survey_vendor_certification_answer_4_p1_q5,survey_vendor_certification_answer_4,vendor_certification_page_1_question_5,True,,,,,,,
survey_vendor_certification_answer_4_p2_q1,survey_vendor_certification_answer_4,vendor_certification_page_2_question_1,False,suggestion,,,,,vendor_certification_page_2_question_1_choice_2,
survey_vendor_certification_answer_4_p2_q2,survey_vendor_certification_answer_4,vendor_certification_page_2_question_2,False,suggestion,,,,,vendor_certification_page_2_question_2_choice_4,
survey_vendor_certification_answer_4_p2_q3,survey_vendor_certification_answer_4,vendor_certification_page_2_question_3,False,suggestion,,,,,vendor_certification_page_2_question_3_choice_5,
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
            <field name="state">open</field>
            <field name="access_mode">public</field>
            <field name="users_can_go_back" eval="True" />
            <field name="users_login_required" eval="True" />
            <field name="scoring_type" >scoring_with_answers</field>
            <field name="certificate" eval="True"></field>
            <field name="certification_mail_template_id" ref="mail_template_certification"></field>
            <field name="is_time_limited" >limited</field>
            <field name="time_limit" >10.0</field>
            <field name="is_attempts_limited" eval="True" />
            <field name="attempts_limit">2</field>
            <field name="description">&lt;p&gt;Test your vendor skills!&lt;/p&gt;</field>
            <field name="thank_you_message">&lt;p&gt;&lt;/p&gt;</field>
        </record>
        <!-- Page 1 -->
        <record model="survey.question" id="vendor_certification_page_1">
            <field name="title">Products</field>
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">1</field>
            <field name="is_page" eval="True"/>
            <field name="question_type" eval="False" />
            <field name="description">&lt;p&gt;Test your knowledge of your products!&lt;/p&gt;</field>
        </record>
        <!-- Question and predefined answer 1 -->
        <record model="survey.question" id="vendor_certification_page_1_question_1">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">2</field>
            <field name="title">Do we sell Acoustic Bloc Screens?</field>
            <field name="question_type">simple_choice</field>
            <field name="display_mode">dropdown</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_1_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_1"/>
            <field name="sequence">1</field>
            <field name="value">No</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_1_choice_2">
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
        <record model="survey.label" id="vendor_certification_page_1_question_2_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Chair floor protection</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_2_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Fanta</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_2_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Conference chair</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_2_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Drawer</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_2_choice_5">
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
        <record model="survey.label" id="vendor_certification_page_1_question_3_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">1</field>
            <field name="value">Color</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_3_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">2</field>
            <field name="value">Height</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_3_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">3</field>
            <field name="value">Width</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_3_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_3"/>
            <field name="sequence">4</field>
            <field name="value">Legs</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_3_choice_5">
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
            <field name="display_mode">dropdown</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_4_choice_1">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">1</field>
            <field name="value">1</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_4_choice_2">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">2</field>
            <field name="value">2</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_4_choice_3">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">3</field>
            <field name="value">3</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_1_question_4_choice_4">
            <field name="question_id" ref="vendor_certification_page_1_question_4"/>
            <field name="sequence">4</field>
            <field name="value">4</field>
        </record>
        <!-- Question and predefined answer 5 -->
        <record model="survey.question" id="vendor_certification_page_1_question_5">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">6</field>
            <field name="title">Do you think we have missing products in our catalog? (not rated)</field>
            <field name="question_type">free_text</field>
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
            <field name="display_mode">dropdown</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">1</field>
            <field name="value">20$</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">2</field>
            <field name="value">50$</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">3</field>
            <field name="value">80$</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">4</field>
            <field name="value">100$</field>
            <field name="is_correct" eval="True" />
            <field name="answer_score">2.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">5</field>
            <field name="value">200$</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_1_choice_6">
            <field name="question_id" ref="vendor_certification_page_2_question_1"/>
            <field name="sequence">6</field>
            <field name="value">300$</field>
        </record>
        <!-- Question and predefined answer 7 -->
        <record model="survey.question" id="vendor_certification_page_2_question_2">
            <field name="survey_id" ref="vendor_certification" />
            <field name="sequence">9</field>
            <field name="title">Select all the the products that sell for 100$ or more</field>
            <field name="question_type">multiple_choice</field>
            <field name="column_nb">2</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">1</field>
            <field name="value">Corner Desk Right Sit</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">2</field>
            <field name="value">Desk Combination</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">3</field>
            <field name="value">Cabinet with Doors</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">4</field>
            <field name="value">Large Desk</field>
            <field name="answer_score">1.0</field>
            <field name="is_correct" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_2"/>
            <field name="sequence">5</field>
            <field name="value">Letter Tray</field>
            <field name="answer_score">-1.0</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_2_choice_5">
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
            <field name="display_mode">dropdown</field>
            <field name="constr_mandatory" eval="True" />
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_3_choice_1">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">1</field>
            <field name="value">Very underpriced</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_3_choice_2">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">2</field>
            <field name="value">Underpriced</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_3_choice_3">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">3</field>
            <field name="value">Correctly priced</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_3_choice_4">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">4</field>
            <field name="value">A little bit overpriced</field>
        </record>
        <record model="survey.label" id="vendor_certification_page_2_question_3_choice_5">
            <field name="question_id" ref="vendor_certification_page_2_question_3"/>
            <field name="sequence">5</field>
            <field name="value">A lot overpriced</field>
        </record>

        <record model="survey.user_input" id="survey_vendor_certification_answer_1">
            <field name="survey_id" ref="survey.vendor_certification" />
            <field name="input_type">manually</field>
            <field name="partner_id" ref="base.res_partner_address_3"/>
            <field name="email">douglas.fletcher51@example.com</field>
            <field name="state">done</field>
            <field name="question_ids" eval="[
                    ref('vendor_certification_page_1_question_1'),
                    ref('vendor_certification_page_1_question_2'),
                    ref('vendor_certification_page_1_question_3'),
                    ref('vendor_certification_page_1_question_4'),
                    ref('vendor_certification_page_1_question_5'),
                    ref('vendor_certification_page_2_question_1'),
                    ref('vendor_certification_page_2_question_2'),
                    ref('vendor_certification_page_2_question_3'),
                ]"/>
        </record>
        <record model="survey.user_input" id="survey_vendor_certification_answer_2">
            <field name="survey_id" ref="survey.vendor_certification" />
            <field name="input_type">manually</field>
            <field name="partner_id" ref="base.res_partner_address_7"/>
            <field name="email">billy.fox45@example.com</field>
            <field name="state">done</field>
            <field name="question_ids" eval="[
                    ref('vendor_certification_page_1_question_1'),
                    ref('vendor_certification_page_1_question_2'),
                    ref('vendor_certification_page_1_question_3'),
                    ref('vendor_certification_page_1_question_4'),
                    ref('vendor_certification_page_1_question_5'),
                    ref('vendor_certification_page_2_question_1'),
                    ref('vendor_certification_page_2_question_2'),
                    ref('vendor_certification_page_2_question_3'),
                ]"/>
        </record>
        <record model="survey.user_input" id="survey_vendor_certification_answer_3">
            <field name="survey_id" ref="survey.vendor_certification" />
            <field name="input_type">manually</field>
            <field name="partner_id" ref="base.res_partner_address_15"/>
            <field name="email">brandon.freeman55@example.com</field>
            <field name="state">done</field>
            <field name="question_ids" eval="[
                    ref('vendor_certification_page_1_question_1'),
                    ref('vendor_certification_page_1_question_2'),
                    ref('vendor_certification_page_1_question_3'),
                    ref('vendor_certification_page_1_question_4'),
                    ref('vendor_certification_page_1_question_5'),
                    ref('vendor_certification_page_2_question_1'),
                    ref('vendor_certification_page_2_question_2'),
                    ref('vendor_certification_page_2_question_3'),
                ]"/>
        </record>
        <record model="survey.user_input" id="survey_vendor_certification_answer_4">
            <field name="survey_id" ref="survey.vendor_certification" />
            <field name="input_type">manually</field>
            <field name="partner_id" ref="base.res_partner_address_25"/>
            <field name="email">oscar.morgan11@example.com</field>
            <field name="state">done</field>
            <field name="question_ids" eval="[
                    ref('vendor_certification_page_1_question_1'),
                    ref('vendor_certification_page_1_question_2'),
                    ref('vendor_certification_page_1_question_3'),
                    ref('vendor_certification_page_1_question_4'),
                    ref('vendor_certification_page_1_question_5'),
                    ref('vendor_certification_page_2_question_1'),
                    ref('vendor_certification_page_2_question_2'),
                    ref('vendor_certification_page_2_question_3'),
                ]"/>
        </record>
    </data>
</odoo>

```

## File: data\survey_demo_feedback.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record model="survey.survey" id="survey_feedback">
        <field name="title">User Feedback Form</field>
        <field name="access_token">b137640d-14d4-4748-9ef6-344ca256531e</field>
        <field name="state">open</field>
        <field name="access_mode">public</field>
        <field name="users_can_go_back" eval="True" />
        <field name="questions_layout">page_per_section</field>
        <field name="description" type="html">
<p>This survey allows you to give a feedback about your experience with our eCommerce solution.
    Filling it helps us improving your experience.</p></field>
        <field name="thank_you_message">&lt;p&gt;&lt;/p&gt;</field>
    </record>

    <!-- Page1: general information -->
    <record model="survey.question" id="survey_feedback_p1">
        <field name="title">General information</field>
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">1</field>
        <field name="question_type" eval="False" />
        <field name="is_page" eval="True" />
        <field name="description" type="html">
<p>This section is about general informations about you. Answering them helps qualifying your answers.</p></field>
    </record>
    <record model="survey.question" id="survey_feedback_p1_q1">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">2</field>
        <field name="title">Where do you live ?</field>
        <field name="question_type">textbox</field>
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
        <field name="display_mode">dropdown</field>
        <field name="comments_allowed" eval="True"/>
        <field name="comment_count_as_answer" eval="True"/>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record model="survey.label" id="survey_feedback_p1_q3_sug1">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">1</field>
        <field name="value">Once a day</field>
    </record>
    <record model="survey.label" id="survey_feedback_p1_q3_sug2">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">2</field>
        <field name="value">Once a week</field>
    </record>
    <record model="survey.label" id="survey_feedback_p1_q3_sug3">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">3</field>
        <field name="value">Once a month</field>
    </record>
    <record model="survey.label" id="survey_feedback_p1_q3_sug4">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">4</field>
        <field name="value">Once a year</field>
    </record>
    <record model="survey.label" id="survey_feedback_p1_q3_sug5">
        <field name="question_id" ref="survey_feedback_p1_q3"/>
        <field name="sequence">5</field>
        <field name="value">Other (answer in comment)</field>
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
    <record model="survey.label" id="survey_feedback_p2_q1_sug1">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">1</field>
        <field name="value">High quality</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug2">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">2</field>
        <field name="value">Useful</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug3">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">3</field>
        <field name="value">Unique</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug4">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">4</field>
        <field name="value">Good value for money</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug5">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">5</field>
        <field name="value">Overpriced</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug6">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">6</field>
        <field name="value">Impractical</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug7">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">7</field>
        <field name="value">Ineffective</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug8">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">8</field>
        <field name="value">Poor quality</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q1_sug9">
        <field name="question_id" ref="survey_feedback_p2_q1"/>
        <field name="sequence">9</field>
        <field name="value">Other</field>
    </record>
    <record model="survey.question" id="survey_feedback_p2_q2">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">8</field>
        <field name="title">What do your think about our new eCommerce ?</field>
        <field name="question_type">matrix</field>
        <field name="matrix_subtype">multiple</field>
        <field name="constr_mandatory" eval="True"/>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_col1">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">1</field>
        <field name="value">Totally disagree</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_col2">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">2</field>
        <field name="value">Disagree</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_col3">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">3</field>
        <field name="value">Agree</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_col4">
        <field name="question_id" ref="survey_feedback_p2_q2"/>
        <field name="sequence">4</field>
        <field name="value">Totally agree</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_row1">
        <field name="question_id_2" ref="survey_feedback_p2_q2"/>
        <field name="sequence">1</field>
        <field name="value">The new layout and design is fresh and up-to-date</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_row2">
        <field name="question_id_2" ref="survey_feedback_p2_q2"/>
        <field name="sequence">2</field>
        <field name="value">It is easy to find the product that I want</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_row3">
        <field name="question_id_2" ref="survey_feedback_p2_q2"/>
        <field name="sequence">3</field>
        <field name="value">The tool to compare the products is useful to make a choice</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_row4">
        <field name="question_id_2" ref="survey_feedback_p2_q2"/>
        <field name="sequence">4</field>
        <field name="value">The checkout process is clear and secure</field>
    </record>
    <record model="survey.label" id="survey_feedback_p2_q2_row5">
        <field name="question_id_2" ref="survey_feedback_p2_q2"/>
        <field name="sequence">5</field>
        <field name="value">I have added products to my wishlist</field>
    </record>
    <record model="survey.question" id="survey_feedback_p2_q3">
        <field name="survey_id" ref="survey_feedback" />
        <field name="sequence">9</field>
        <field name="title">Do you have any other comments, questions, or concerns ?</field>
        <field name="question_type">free_text</field>
        <field name="constr_mandatory" eval="False"/>
    </record>

    <record model="survey.user_input" id="survey_answer_1">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="input_type">manually</field>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="email">mark.brown23@example.com</field>
        <field name="state">done</field>
    </record>
    <record model="survey.user_input" id="survey_answer_2">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="input_type">manually</field>
        <field name="partner_id" ref="base.res_partner_address_7"/>
        <field name="email">billy.fox45@example.com</field>
        <field name="state">done</field>
    </record>
    <record model="survey.user_input" id="survey_answer_3">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="input_type">manually</field>
        <field name="partner_id" eval="False"/>
        <field name="email">Evelyne Gargouillis &lt;evelyne@example.com&gt;</field>
        <field name="state">done</field>
    </record>
    <record model="survey.user_input" id="survey_answer_4">
        <field name="survey_id" ref="survey.survey_feedback" />
        <field name="input_type">manually</field>
        <field name="partner_id" eval="False"/>
        <field name="email">Martin Tamarre &lt;martin@example.com&gt;</field>
        <field name="state">skip</field>
    </record>

</data></odoo>

```

## File: data\survey_demo_user.xml

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

    category = fields.Selection(selection_add=[('certification', 'Certifications')])

```

## File: models\ir_autovacuum.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AutoVacuum(models.AbstractModel):
    _inherit = 'ir.autovacuum'

    @api.model
    def power_on(self, *args, **kwargs):
        self.env['survey.user_input'].do_clean_emptys()
        return super(AutoVacuum, self).power_on(*args, **kwargs)

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
            [('partner_id', 'in', self.ids), ('quizz_passed', '=', True)],
            ['partner_id'], 'partner_id'
        )
        data = dict((res['partner_id'][0], res['partner_id_count']) for res in read_group_res)
        for partner in self:
            partner.certifications_count = data.get(partner.id, 0)

    @api.depends('is_company', 'child_ids.certifications_count')
    def _compute_certifications_company_count(self):
        self.certifications_company_count = sum(child.certifications_count for child in self.child_ids)

    def action_view_certifications(self):
        action = self.env.ref('survey.res_partner_action_certifications').read()[0]
        action['view_mode'] = 'tree'
        action['domain'] = ['|', ('partner_id', 'in', self.ids), ('partner_id', 'in', self.child_ids.ids)]

        return action

```

## File: models\survey_question.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re
import datetime

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError

email_validator = re.compile(r"[^@]+@[^@]+\.[^@]+")
_logger = logging.getLogger(__name__)


def dict_keys_startswith(dictionary, string):
    """Returns a dictionary containing the elements of <dict> whose keys start with <string>.
        .. note::
            This function uses dictionary comprehensions (Python >= 2.7)
    """
    return {k: v for k, v in dictionary.items() if k.startswith(string)}


class SurveyQuestion(models.Model):
    """ Questions that will be asked in a survey.

        Each question can have one of more suggested answers (eg. in case of
        dropdown choices, multi-answer checkboxes, radio buttons...).

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
    _rec_name = 'question'
    _order = 'sequence,id'

    @api.model
    def default_get(self, fields):
        defaults = super(SurveyQuestion, self).default_get(fields)
        if (not fields or 'question_type' in fields):
            defaults['question_type'] = False if defaults.get('is_page') == True else 'free_text'
        return defaults

    # Question metadata
    survey_id = fields.Many2one('survey.survey', string='Survey', ondelete='cascade')
    page_id = fields.Many2one('survey.question', string='Page', compute="_compute_page_id", store=True)
    question_ids = fields.One2many('survey.question', string='Questions', compute="_compute_question_ids")
    scoring_type = fields.Selection(related='survey_id.scoring_type', string='Scoring Type', readonly=True)
    sequence = fields.Integer('Sequence', default=10)
    # Question
    is_page = fields.Boolean('Is a page?')
    questions_selection = fields.Selection(
        related='survey_id.questions_selection', readonly=True,
        help="If randomized is selected, add the number of random questions next to the section.")
    random_questions_count = fields.Integer(
        'Random questions count', default=1,
        help="Used on randomized sections to take X random questions from all the questions of that section.")
    title = fields.Char('Title', required=True, translate=True)
    question = fields.Char('Question', related="title")
    description = fields.Html('Description', help="Use this field to add additional explanations about your question", translate=True)
    question_type = fields.Selection([
        ('free_text', 'Multiple Lines Text Box'),
        ('textbox', 'Single Line Text Box'),
        ('numerical_box', 'Numerical Value'),
        ('date', 'Date'),
        ('datetime', 'Datetime'),
        ('simple_choice', 'Multiple choice: only one answer'),
        ('multiple_choice', 'Multiple choice: multiple answers allowed'),
        ('matrix', 'Matrix')], string='Question Type')
    # simple choice / multiple choice / matrix
    labels_ids = fields.One2many(
        'survey.label', 'question_id', string='Types of answers', copy=True,
        help='Labels used for proposed choices: simple choice, multiple choice and columns of matrix')
    # matrix
    matrix_subtype = fields.Selection([
        ('simple', 'One choice per row'),
        ('multiple', 'Multiple choices per row')], string='Matrix Type', default='simple')
    labels_ids_2 = fields.One2many(
        'survey.label', 'question_id_2', string='Rows of the Matrix', copy=True,
        help='Labels used for proposed choices: rows of matrix')
    # Display options
    column_nb = fields.Selection([
        ('12', '1'), ('6', '2'), ('4', '3'), ('3', '4'), ('2', '6')],
        string='Number of columns', default='12',
        help='These options refer to col-xx-[12|6|4|3|2] classes in Bootstrap for dropdown-based simple and multiple choice questions.')
    display_mode = fields.Selection(
        [('columns', 'Radio Buttons'), ('dropdown', 'Selection Box')],
        string='Display Mode', default='columns', help='Display mode of simple choice questions.')
    # Comments
    comments_allowed = fields.Boolean('Show Comments Field')
    comments_message = fields.Char('Comment Message', translate=True, default=lambda self: _("If other, please specify:"))
    comment_count_as_answer = fields.Boolean('Comment Field is an Answer Choice')
    # Validation
    validation_required = fields.Boolean('Validate entry')
    validation_email = fields.Boolean('Input must be an email')
    validation_length_min = fields.Integer('Minimum Text Length')
    validation_length_max = fields.Integer('Maximum Text Length')
    validation_min_float_value = fields.Float('Minimum value')
    validation_max_float_value = fields.Float('Maximum value')
    validation_min_date = fields.Date('Minimum Date')
    validation_max_date = fields.Date('Maximum Date')
    validation_min_datetime = fields.Datetime('Minimum Datetime')
    validation_max_datetime = fields.Datetime('Maximum Datetime')
    validation_error_msg = fields.Char('Validation Error message', translate=True, default=lambda self: _("The answer you entered is not valid."))
    # Constraints on number of answers (matrices)
    constr_mandatory = fields.Boolean('Mandatory Answer')
    constr_error_msg = fields.Char('Error message', translate=True, default=lambda self: _("This question requires an answer."))
    # Answer
    user_input_line_ids = fields.One2many(
        'survey.user_input_line', 'question_id', string='Answers',
        domain=[('skipped', '=', False)], groups='survey.group_survey_user')

    _sql_constraints = [
        ('positive_len_min', 'CHECK (validation_length_min >= 0)', 'A length must be positive!'),
        ('positive_len_max', 'CHECK (validation_length_max >= 0)', 'A length must be positive!'),
        ('validation_length', 'CHECK (validation_length_min <= validation_length_max)', 'Max length cannot be smaller than min length!'),
        ('validation_float', 'CHECK (validation_min_float_value <= validation_max_float_value)', 'Max value cannot be smaller than min value!'),
        ('validation_date', 'CHECK (validation_min_date <= validation_max_date)', 'Max date cannot be smaller than min date!'),
        ('validation_datetime', 'CHECK (validation_min_datetime <= validation_max_datetime)','Max datetime cannot be smaller than min datetime!')
    ]

    @api.onchange('validation_email')
    def _onchange_validation_email(self):
        if self.validation_email:
            self.validation_required = False

    @api.onchange('is_page')
    def _onchange_is_page(self):
        if self.is_page:
            self.question_type = False

    # Validation methods

    def validate_question(self, post, answer_tag):
        """ Validate question, depending on question type and parameters """
        self.ensure_one()
        try:
            checker = getattr(self, 'validate_' + self.question_type)
        except AttributeError:
            _logger.warning(self.question_type + ": This type of question has no validation method")
            return {}
        else:
            return checker(post, answer_tag)

    def validate_free_text(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        answer = post[answer_tag].strip()
        # Empty answer to mandatory question
        if self.constr_mandatory and not answer:
            errors.update({answer_tag: self.constr_error_msg})
        return errors

    def validate_textbox(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        answer = post[answer_tag].strip()
        # Empty answer to mandatory question
        if self.constr_mandatory and not answer:
            errors.update({answer_tag: self.constr_error_msg})
        # Email format validation
        # Note: this validation is very basic:
        #     all the strings of the form
        #     <something>@<anything>.<extension>
        #     will be accepted
        if answer and self.validation_email:
            if not email_validator.match(answer):
                errors.update({answer_tag: _('This answer must be an email address')})
        # Answer validation (if properly defined)
        # Length of the answer must be in a range
        if answer and self.validation_required:
            if not (self.validation_length_min <= len(answer) <= self.validation_length_max):
                errors.update({answer_tag: self.validation_error_msg})
        return errors

    def validate_numerical_box(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        answer = post[answer_tag].strip()
        # Empty answer to mandatory question
        if self.constr_mandatory and not answer:
            errors.update({answer_tag: self.constr_error_msg})
        # Checks if user input is a number
        if answer:
            try:
                floatanswer = float(answer)
            except ValueError:
                errors.update({answer_tag: _('This is not a number')})
        # Answer validation (if properly defined)
        if answer and self.validation_required:
            # Answer is not in the right range
            with tools.ignore(Exception):
                floatanswer = float(answer)  # check that it is a float has been done hereunder
                if not (self.validation_min_float_value <= floatanswer <= self.validation_max_float_value):
                    errors.update({answer_tag: self.validation_error_msg})
        return errors

    def date_validation(self, date_type, post, answer_tag, min_value, max_value):
        self.ensure_one()
        errors = {}
        if date_type not in ('date', 'datetime'):
            raise ValueError("Unexpected date type value")
        answer = post[answer_tag].strip()
        # Empty answer to mandatory question
        if self.constr_mandatory and not answer:
            errors.update({answer_tag: self.constr_error_msg})
        # Checks if user input is a date
        if answer:
            try:
                if date_type == 'datetime':
                    dateanswer = fields.Datetime.from_string(answer)
                else:
                    dateanswer = fields.Date.from_string(answer)
            except ValueError:
                errors.update({answer_tag: _('This is not a date')})
                return errors
        # Answer validation (if properly defined)
        if answer and self.validation_required:
            # Answer is not in the right range
            try:
                if date_type == 'datetime':
                    date_from_string = fields.Datetime.from_string
                else:
                    date_from_string = fields.Date.from_string
                dateanswer = date_from_string(answer)
                min_date = date_from_string(min_value)
                max_date = date_from_string(max_value)

                if min_date and max_date and not (min_date <= dateanswer <= max_date):
                    # If Minimum and Maximum Date are entered
                    errors.update({answer_tag: self.validation_error_msg})
                elif min_date and not min_date <= dateanswer:
                    # If only Minimum Date is entered and not Define Maximum Date
                    errors.update({answer_tag: self.validation_error_msg})
                elif max_date and not dateanswer <= max_date:
                    # If only Maximum Date is entered and not Define Minimum Date
                    errors.update({answer_tag: self.validation_error_msg})
            except ValueError:  # check that it is a date has been done hereunder
                pass
        return errors

    def validate_date(self, post, answer_tag):
        return self.date_validation('date', post, answer_tag, self.validation_min_date, self.validation_max_date)

    def validate_datetime(self, post, answer_tag):
        return self.date_validation('datetime', post, answer_tag, self.validation_min_datetime, self.validation_max_datetime)

    def validate_simple_choice(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        if self.comments_allowed:
            comment_tag = "%s_%s" % (answer_tag, 'comment')
        # Empty answer to mandatory self
        if self.constr_mandatory and answer_tag not in post:
            errors.update({answer_tag: self.constr_error_msg})
        if self.constr_mandatory and answer_tag in post and not post[answer_tag].strip():
            errors.update({answer_tag: self.constr_error_msg})
        # Answer is a comment and is empty
        if self.constr_mandatory and answer_tag in post and post[answer_tag] == "-1" and self.comment_count_as_answer and comment_tag in post and not post[comment_tag].strip():
            errors.update({answer_tag: self.constr_error_msg})
        return errors

    def validate_multiple_choice(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        if self.constr_mandatory:
            answer_candidates = dict_keys_startswith(post, answer_tag)
            comment_flag = answer_candidates.pop(("%s_%s" % (answer_tag, -1)), None)
            if self.comments_allowed:
                comment_answer = answer_candidates.pop(("%s_%s" % (answer_tag, 'comment')), '').strip()
            # Preventing answers with blank value
            if all(not answer.strip() for answer in answer_candidates.values()) and answer_candidates:
                errors.update({answer_tag: self.constr_error_msg})
            # There is no answer neither comments (if comments count as answer)
            if not answer_candidates and self.comment_count_as_answer and (not comment_flag or not comment_answer):
                errors.update({answer_tag: self.constr_error_msg})
            # There is no answer at all
            if not answer_candidates and not self.comment_count_as_answer:
                errors.update({answer_tag: self.constr_error_msg})
        return errors

    def validate_matrix(self, post, answer_tag):
        self.ensure_one()
        errors = {}
        if self.constr_mandatory:
            lines_number = len(self.labels_ids_2)
            answer_candidates = dict_keys_startswith(post, answer_tag)
            answer_candidates.pop(("%s_%s" % (answer_tag, 'comment')), '').strip()
            # Number of lines that have been answered
            if self.matrix_subtype == 'simple':
                answer_number = len(answer_candidates)
            elif self.matrix_subtype == 'multiple':
                answer_number = len({sk.rsplit('_', 1)[0] for sk in answer_candidates})
            else:
                raise RuntimeError("Invalid matrix subtype")
            # Validate that each line has been answered
            if answer_number != lines_number:
                errors.update({answer_tag: self.constr_error_msg})
        return errors

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

                question.question_ids = question.survey_id.question_ids.filtered(lambda q:
                    q._index() > question._index() and (not next_page_index or q._index() < next_page_index))
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

    def _index(self):
        """We would normally just use the 'sequence' field of questions BUT, if the pages and questions are
        created without ever moving records around, the sequence field can be set to 0 for all the questions.

        However, the order of the recordset is always correct so we can rely on the index method."""
        self.ensure_one()
        return list(self.survey_id.question_and_page_ids).index(self)

    def get_correct_answer_ids(self):
        self.ensure_one()

        return self.labels_ids.filtered(lambda label: label.is_correct)

class SurveyLabel(models.Model):
    """ A suggested answer for a question """
    _name = 'survey.label'
    _rec_name = 'value'
    _order = 'sequence,id'
    _description = 'Survey Label'

    question_id = fields.Many2one('survey.question', string='Question', ondelete='cascade')
    question_id_2 = fields.Many2one('survey.question', string='Question 2', ondelete='cascade')
    sequence = fields.Integer('Label Sequence order', default=10)
    value = fields.Char('Suggested value', translate=True, required=True)
    is_correct = fields.Boolean('Is a correct answer')
    answer_score = fields.Float('Score for this choice',
    help="A positive score indicates a correct choice; a negative or null score indicates a wrong answer")

    @api.constrains('question_id', 'question_id_2')
    def _check_question_not_empty(self):
        """Ensure that field question_id XOR field question_id_2 is not null"""
        for label in self:
            if not bool(label.question_id) != bool(label.question_id_2):
                raise ValidationError(_("A label must be attached to only one question."))

```

## File: models\survey_survey.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import uuid

from collections import Counter, OrderedDict
from itertools import product
from werkzeug import urls
import random

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class Survey(models.Model):
    """ Settings for a multi-page/multi-question survey. Each survey can have one or more attached pages
    and each page can display one or more questions. """
    _name = 'survey.survey'
    _description = 'Survey'
    _rec_name = 'title'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    def _get_default_access_token(self):
        return str(uuid.uuid4())

    # description
    title = fields.Char('Survey Title', required=True, translate=True)
    description = fields.Html("Description", translate=True,
        help="The description will be displayed on the home page of the survey. You can use this to give the purpose and guidelines to your candidates before they start it.")
    color = fields.Integer('Color Index', default=0)
    thank_you_message = fields.Html("Thanks Message", translate=True, help="This message will be displayed when survey is completed")
    active = fields.Boolean("Active", default=True)
    question_and_page_ids = fields.One2many('survey.question', 'survey_id', string='Sections and Questions', copy=True)
    page_ids = fields.One2many('survey.question', string='Pages', compute="_compute_page_and_question_ids")
    question_ids = fields.One2many('survey.question', string='Questions', compute="_compute_page_and_question_ids")
    state = fields.Selection(
        string="Survey Stage",
        selection=[
                ('draft', 'Draft'),
                ('open', 'In Progress'),
                ('closed', 'Closed'),
        ], default='draft', required=True,
        group_expand='_read_group_states'
    )
    questions_layout = fields.Selection([
        ('one_page', 'One page with all the questions'),
        ('page_per_section', 'One page per section'),
        ('page_per_question', 'One page per question')],
        string="Layout", required=True, default='one_page')
    questions_selection = fields.Selection([
        ('all', 'All questions'),
        ('random', 'Randomized per section')],
        string="Selection", required=True, default='all',
        help="If randomized is selected, add the number of random questions next to the section.")

    category = fields.Selection([
        ('default', 'Generic Survey')], string='Category',
        default='default', required=True,
        help='Category is used to know in which context the survey is used. Various apps may define their own categories when they use survey like jobs recruitment or employee appraisal surveys.')
    # content
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
    public_url = fields.Char("Public link", compute="_compute_survey_url")
    # statistics
    answer_count = fields.Integer("Registered", compute="_compute_survey_statistic")
    answer_done_count = fields.Integer("Attempts", compute="_compute_survey_statistic")
    answer_score_avg = fields.Float("Avg Score %", compute="_compute_survey_statistic")
    success_count = fields.Integer("Success", compute="_compute_survey_statistic")
    success_ratio = fields.Integer("Success Ratio", compute="_compute_survey_statistic")
    # scoring and certification fields
    scoring_type = fields.Selection([
        ('no_scoring', 'No scoring'),
        ('scoring_with_answers', 'Scoring with answers at the end'),
        ('scoring_without_answers', 'Scoring without answers at the end')],
        string="Scoring", required=True, default='no_scoring')
    passing_score = fields.Float('Passing score (%)', required=True, default=80.0)
    is_attempts_limited = fields.Boolean('Limited number of attempts',
        help="Check this option if you want to limit the number of attempts per user")
    attempts_limit = fields.Integer('Number of attempts', default=1)
    is_time_limited = fields.Boolean('The survey is limited in time')
    time_limit = fields.Float("Time limit (minutes)")
    certificate = fields.Boolean('Certificate')
    certification_mail_template_id = fields.Many2one(
        'mail.template', 'Email Template',
        domain="[('model', '=', 'survey.user_input')]",
        help="Automated email sent to the user when he succeeds the certification, containing his certification document.")

    # Certification badge
    #   certification_badge_id_dummy is used to have two different behaviours in the form view :
    #   - If the certification badge is not set, show certification_badge_id and only display create option in the m2o
    #   - If the certification badge is set, show certification_badge_id_dummy in 'no create' mode.
    #       So it can be edited but not removed or replaced.
    certification_give_badge = fields.Boolean('Give Badge')
    certification_badge_id = fields.Many2one('gamification.badge', 'Certification Badge')
    certification_badge_id_dummy = fields.Many2one(related='certification_badge_id', string='Certification Badge ')

    _sql_constraints = [
        ('access_token_unique', 'unique(access_token)', 'Access token should be unique'),
        ('certificate_check', "CHECK( scoring_type!='no_scoring' OR certificate=False )",
            'You can only create certifications for surveys that have a scoring mechanism.'),
        ('time_limit_check', "CHECK( (is_time_limited=False) OR (time_limit is not null AND time_limit > 0) )",
            'The time limit needs to be a positive number if the survey is time limited.'),
        ('attempts_limit_check', "CHECK( (is_attempts_limited=False) OR (attempts_limit is not null AND attempts_limit > 0) )",
            'The attempts limit needs to be a positive number if the survey has a limited number of attempts.'),
        ('badge_uniq', 'unique (certification_badge_id)', "The badge for each survey should be unique!"),
        ('give_badge_check', "CHECK(certification_give_badge=False OR (certification_give_badge=True AND certification_badge_id is not null))",
            'Certification badge must be configured if Give Badge is set.'),
    ]

    def _compute_users_can_signup(self):
        signup_allowed = self.env['res.users'].sudo()._get_signup_invitation_scope() == 'b2c'
        for survey in self:
            survey.users_can_signup = signup_allowed

    @api.depends('user_input_ids.state', 'user_input_ids.test_entry', 'user_input_ids.quizz_score', 'user_input_ids.quizz_passed')
    def _compute_survey_statistic(self):
        default_vals = {
            'answer_count': 0, 'answer_done_count': 0, 'success_count': 0,
            'answer_score_avg': 0.0, 'success_ratio': 0.0
        }
        stat = dict((cid, dict(default_vals, answer_score_avg_total=0.0)) for cid in self.ids)
        UserInput = self.env['survey.user_input']
        base_domain = ['&', ('survey_id', 'in', self.ids), ('test_entry', '!=', True)]

        read_group_res = UserInput.read_group(base_domain, ['survey_id', 'state'], ['survey_id', 'state', 'quizz_score', 'quizz_passed'], lazy=False)
        for item in read_group_res:
            stat[item['survey_id'][0]]['answer_count'] += item['__count']
            stat[item['survey_id'][0]]['answer_score_avg_total'] += item['quizz_score']
            if item['state'] == 'done':
                stat[item['survey_id'][0]]['answer_done_count'] += item['__count']
            if item['quizz_passed']:
                stat[item['survey_id'][0]]['success_count'] += item['__count']

        for survey_id, values in stat.items():
            avg_total = stat[survey_id].pop('answer_score_avg_total')
            stat[survey_id]['answer_score_avg'] = avg_total / (stat[survey_id]['answer_done_count'] or 1)
            stat[survey_id]['success_ratio'] = (stat[survey_id]['success_count'] / (stat[survey_id]['answer_done_count'] or 1.0))*100

        for survey in self:
            survey.update(stat.get(survey._origin.id, default_vals))

    def _compute_survey_url(self):
        """ Computes a public URL for the survey """
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        for survey in self:
            survey.public_url = urls.url_join(base_url, "survey/start/%s" % (survey.access_token))

    @api.depends('question_and_page_ids')
    def _compute_page_and_question_ids(self):
        for survey in self:
            survey.page_ids = survey.question_and_page_ids.filtered(lambda question: question.is_page)
            survey.question_ids = survey.question_and_page_ids - survey.page_ids

    @api.onchange('passing_score')
    def _onchange_passing_score(self):
        if self.passing_score < 0 or self.passing_score > 100:
            self.passing_score = 80.0

    @api.onchange('scoring_type')
    def _onchange_scoring_type(self):
        if self.scoring_type == 'no_scoring':
            self.certificate = False

    @api.onchange('users_login_required', 'access_mode')
    def _onchange_access_mode(self):
        if self.access_mode == 'public' and not self.users_login_required:
            self.is_attempts_limited = False

    @api.onchange('attempts_limit')
    def _onchange_attempts_limit(self):
        if self.attempts_limit <= 0:
            self.attempts_limit = 1

    @api.onchange('is_time_limited', 'time_limit')
    def _onchange_time_limit(self):
        if self.is_time_limited and (not self.time_limit or self.time_limit <= 0):
            self.time_limit = 10

    def _read_group_states(self, values, domain, order):
        selection = self.env['survey.survey'].fields_get(allfields=['state'])['state']['selection']
        return [s[0] for s in selection]

    @api.onchange('users_login_required', 'certificate')
    def _onchange_set_certification_give_badge(self):
        if not self.users_login_required or not self.certificate:
            self.certification_give_badge = False

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
            return self.sudo()._handle_certification_badges(vals)
        return result

    def copy_data(self, default=None):
        title = _("%s (copy)") % (self.title)
        default = dict(default or {}, title=title)
        return super(Survey, self).copy_data(default)

    # ------------------------------------------------------------
    # TECHNICAL
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

        answers = self.env['survey.user_input']
        for survey in self:
            if partner and not user and partner.user_ids:
                user = partner.user_ids[0]

            invite_token = additional_vals.pop('invite_token', False)
            survey._check_answer_creation(user, partner, email, test_entry=test_entry, check_attempts=check_attempts, invite_token=invite_token)
            answer_vals = {
                'survey_id': survey.id,
                'test_entry': test_entry,
                'question_ids': [(6, 0, survey._prepare_answer_questions().ids)]
            }
            if user and not user._is_public():
                answer_vals['partner_id'] = user.partner_id.id
                answer_vals['email'] = user.email
            elif partner:
                answer_vals['partner_id'] = partner.id
                answer_vals['email'] = partner.email
            else:
                answer_vals['email'] = email

            if invite_token:
                answer_vals['invite_token'] = invite_token
            elif survey.is_attempts_limited and survey.access_mode != 'public':
                # attempts limited: create a new invite_token
                # exception made for 'public' access_mode since the attempts pool is global because answers are
                # created every time the user lands on '/start'
                answer_vals['invite_token'] = self.env['survey.user_input']._generate_invite_token()

            answer_vals.update(additional_vals)
            answers += answers.create(answer_vals)

        return answers

    def _check_answer_creation(self, user, partner, email, test_entry=False, check_attempts=True, invite_token=False):
        """ Ensure conditions to create new tokens are met. """
        self.ensure_one()
        if test_entry:
            # the current user must have the access rights to survey
            if not user.has_group('survey.group_survey_user'):
                raise UserError(_('Creating test token is not allowed for you.'))
        else:
            if not self.active:
                raise UserError(_('Creating token for archived surveys is not allowed.'))
            elif self.state == 'closed':
                raise UserError(_('Creating token for closed surveys is not allowed.'))
            if self.access_mode == 'authentication':
                # signup possible -> should have at least a partner to create an account
                if self.users_can_signup and not user and not partner:
                    raise UserError(_('Creating token for external people is not allowed for surveys requesting authentication.'))
                # no signup possible -> should be a not public user (employee or portal users)
                if not self.users_can_signup and (not user or user._is_public()):
                    raise UserError(_('Creating token for external people is not allowed for surveys requesting authentication.'))
            if self.access_mode == 'internal' and (not user or not user.has_group('base.group_user')):
                raise UserError(_('Creating token for anybody else than employees is not allowed for internal surveys.'))
            if check_attempts and not self._has_attempts_left(partner or (user and user.partner_id), email, invite_token):
                raise UserError(_('No attempts left.'))

    def _prepare_answer_questions(self):
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
    # ACTIONS
    # ------------------------------------------------------------

    @api.model
    def next_page_or_question(self, user_input, page_or_question_id, go_back=False):
        """ The next page to display to the user, knowing that page_id is the id
            of the last displayed page.

            If page_id == 0, it will always return the first page of the survey.

            If all the pages have been displayed and go_back == False, it will
            return None

            If go_back == True, it will return the *previous* page instead of the
            next page.

            .. note::
                It is assumed here that a careful user will not try to set go_back
                to True if she knows that the page to display is the first one!
                (doing this will probably cause a giant worm to eat her house)
        """
        survey = user_input.survey_id

        if survey.questions_layout == 'one_page':
            return (None, False)
        elif survey.questions_layout == 'page_per_question' and survey.questions_selection == 'random':
            pages_or_questions = list(enumerate(
                user_input.question_ids
            ))
        else:
            pages_or_questions = list(enumerate(
                survey.question_ids if survey.questions_layout == 'page_per_question' else survey.page_ids
            ))

        # First page
        if page_or_question_id == 0:
            return (pages_or_questions[0][1], len(pages_or_questions) == 1)

        current_page_index = pages_or_questions.index(next(p for p in pages_or_questions if p[1].id == page_or_question_id))

        # All the pages have been displayed
        if current_page_index == len(pages_or_questions) - 1 and not go_back:
            return (None, False)
        # Let's get back, baby!
        elif go_back and survey.users_can_go_back:
            return (pages_or_questions[current_page_index - 1][1], False)
        else:
            # This will show the last page
            if current_page_index == len(pages_or_questions) - 2:
                return (pages_or_questions[current_page_index + 1][1], True)
            # This will show a regular page
            else:
                return (pages_or_questions[current_page_index + 1][1], False)

    def action_draft(self):
        self.write({'state': 'draft'})

    def action_open(self):
        self.write({'state': 'open'})

    def action_close(self):
        self.write({'state': 'closed'})

    def action_start_survey(self):
        """ Open the website page with the survey form """
        self.ensure_one()
        token = self.env.context.get('survey_token')
        trail = "?answer_token=%s" % token if token else ""
        return {
            'type': 'ir.actions.act_url',
            'name': "Start Survey",
            'target': 'self',
            'url': self.public_url + trail
        }

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

        if self.state == 'closed':
            raise UserError(_("You cannot send invitations for closed surveys."))

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

    def action_print_survey(self):
        """ Open the website page with the survey printable view """
        self.ensure_one()
        token = self.env.context.get('survey_token')
        trail = "?answer_token=%s" % token if token else ""
        return {
            'type': 'ir.actions.act_url',
            'name': "Print Survey",
            'target': 'self',
            'url': '/survey/print/%s%s' % (self.access_token, trail)
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
            'target': 'self',
            'url': '/survey/test/%s' % self.access_token,
        }

    def action_survey_user_input_completed(self):
        action_rec = self.env.ref('survey.action_survey_user_input')
        action = action_rec.read()[0]
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_completed': 1,
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    def action_survey_user_input_certified(self):
        action_rec = self.env.ref('survey.action_survey_user_input')
        action = action_rec.read()[0]
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_quizz_passed': 1,
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    def action_survey_user_input(self):
        action_rec = self.env.ref('survey.action_survey_user_input')
        action = action_rec.read()[0]
        ctx = dict(self.env.context)
        ctx.update({'search_default_survey_id': self.ids[0],
                    'search_default_not_test': 1})
        action['context'] = ctx
        return action

    # ------------------------------------------------------------
    # GRAPH / RESULTS
    # ------------------------------------------------------------

    def filter_input_ids(self, filters, finished=False):
        """If user applies any filters, then this function returns list of
           filtered user_input_id and label's strings for display data in web.
           :param filters: list of dictionary (having: row_id, ansewr_id)
           :param finished: True for completely filled survey,Falser otherwise.
           :returns list of filtered user_input_ids.
        """
        self.ensure_one()
        if filters:
            domain_filter, choice = [], []
            for current_filter in filters:
                row_id, answer_id = current_filter['row_id'], current_filter['answer_id']
                if row_id == 0:
                    choice.append(answer_id)
                else:
                    domain_filter.extend(['|', ('value_suggested_row.id', '=', row_id), ('value_suggested.id', '=', answer_id)])
            if choice:
                domain_filter.insert(0, ('value_suggested.id', 'in', choice))
            else:
                domain_filter = domain_filter[1:]
            input_lines = self.env['survey.user_input_line'].search(domain_filter)
            filtered_input_ids = [input_line.user_input_id.id for input_line in input_lines]
        else:
            filtered_input_ids = []
        if finished:
            UserInput = self.env['survey.user_input']
            if not filtered_input_ids:
                user_inputs = UserInput.search([('survey_id', '=', self.id)])
            else:
                user_inputs = UserInput.browse(filtered_input_ids)
            return user_inputs.filtered(lambda input_item: input_item.state == 'done').ids
        return filtered_input_ids

    @api.model
    def get_filter_display_data(self, filters):
        """Returns data to display current filters
            :param filters: list of dictionary (having: row_id, answer_id)
            :returns list of dict having data to display filters.
        """
        filter_display_data = []
        if filters:
            Label = self.env['survey.label']
            for current_filter in filters:
                row_id, answer_id = current_filter['row_id'], current_filter['answer_id']
                label = Label.browse(answer_id)
                question = label.question_id
                if row_id == 0:
                    labels = label
                else:
                    labels = Label.browse([row_id, answer_id])
                filter_display_data.append({'question_text': question.question,
                                            'labels': labels.mapped('value')})
        return filter_display_data

    @api.model
    def prepare_result(self, question, current_filters=None):
        """ Compute statistical data for questions by counting number of vote per choice on basis of filter """
        current_filters = current_filters if current_filters else []
        result_summary = {}
        input_lines = question.user_input_line_ids.filtered(lambda line: not line.user_input_id.test_entry)

        # Calculate and return statistics for choice
        if question.question_type in ['simple_choice', 'multiple_choice']:
            comments = []
            answers = OrderedDict((label.id, {'text': label.value, 'count': 0, 'answer_id': label.id, 'answer_score': label.answer_score}) for label in question.labels_ids)
            for input_line in input_lines:
                if input_line.answer_type == 'suggestion' and answers.get(input_line.value_suggested.id) and (not(current_filters) or input_line.user_input_id.id in current_filters):
                    answers[input_line.value_suggested.id]['count'] += 1
                if input_line.answer_type == 'text' and (not(current_filters) or input_line.user_input_id.id in current_filters):
                    comments.append(input_line)
            result_summary = {'answers': list(answers.values()), 'comments': comments}

        # Calculate and return statistics for matrix
        if question.question_type == 'matrix':
            rows = OrderedDict()
            answers = OrderedDict()
            res = dict()
            comments = []
            [rows.update({label.id: label.value}) for label in question.labels_ids_2]
            [answers.update({label.id: label.value}) for label in question.labels_ids]
            for cell in product(rows, answers):
                res[cell] = 0
            for input_line in input_lines:
                if input_line.answer_type == 'suggestion' and (not(current_filters) or input_line.user_input_id.id in current_filters) and input_line.value_suggested_row:
                    res[(input_line.value_suggested_row.id, input_line.value_suggested.id)] += 1
                if input_line.answer_type == 'text' and (not(current_filters) or input_line.user_input_id.id in current_filters):
                    comments.append(input_line)
            result_summary = {'answers': answers, 'rows': rows, 'result': res, 'comments': comments}

        # Calculate and return statistics for free_text, textbox, date
        if question.question_type in ['free_text', 'textbox', 'date', 'datetime']:
            result_summary = []
            for input_line in input_lines:
                if not(current_filters) or input_line.user_input_id.id in current_filters:
                    result_summary.append(input_line)

        # Calculate and return statistics for numerical_box
        if question.question_type == 'numerical_box':
            result_summary = {'input_lines': []}
            all_inputs = []
            for input_line in input_lines:
                if not(current_filters) or input_line.user_input_id.id in current_filters:
                    all_inputs.append(input_line.value_number)
                    result_summary['input_lines'].append(input_line)
            if all_inputs:
                result_summary.update({'average': round(sum(all_inputs) / len(all_inputs), 2),
                                       'max': round(max(all_inputs), 2),
                                       'min': round(min(all_inputs), 2),
                                       'sum': sum(all_inputs),
                                       'most_common': Counter(all_inputs).most_common(5)})
        return result_summary

    @api.model
    def get_input_summary(self, question, current_filters=None):
        """ Returns overall summary of question e.g. answered, skipped, total_inputs on basis of filter """
        domain = [
            ('user_input_id.test_entry', '=', False),
            ('user_input_id.state', '!=', 'new'),
            ('question_id', '=', question.id)
        ]
        if current_filters:
            domain = expression.AND([[('id', 'in', current_filters)], domain])

        line_ids = self.env["survey.user_input_line"].search(domain)
        return {
            'answered': len(line_ids.filtered(lambda line: not line.skipped).mapped('user_input_id')),
            'skipped': len(line_ids.filtered(lambda line: line.skipped).mapped('user_input_id'))
        }

    def _get_answers_correctness(self, user_answers):
        if not user_answers.mapped('survey_id') == self:
            raise UserError(_('Invalid performance computation'))

        res = dict((user_answer, {
            'correct': 0,
            'incorrect': 0,
            'partial': 0,
            'skipped': 0,
        }) for user_answer in user_answers)

        scored_questions = self.question_ids.filtered(
            lambda question: question.question_type in ['simple_choice', 'multiple_choice']
        )

        for question in scored_questions:
            question_answer_correct = question.labels_ids.filtered(lambda answer: answer.is_correct)
            for user_answer in user_answers:
                if question not in user_answer.question_ids:
                    # the question may be in the survey, but not be selected by the random selection
                    continue

                user_answer_lines_question = user_answer.user_input_line_ids.filtered(lambda line: line.question_id == question)
                user_answer_correct = user_answer_lines_question.filtered(lambda line: line.answer_is_correct and not line.skipped).mapped('value_suggested')
                user_answer_incorrect = user_answer_lines_question.filtered(lambda line: not line.answer_is_correct and not line.skipped)

                if question_answer_correct and user_answer_correct == question_answer_correct:
                    res[user_answer]['correct'] += 1
                elif user_answer_correct and user_answer_correct < question_answer_correct:
                    res[user_answer]['partial'] += 1
                if not user_answer_correct and user_answer_incorrect:
                    res[user_answer]['incorrect'] += 1
                if not user_answer_correct and not user_answer_incorrect:
                    res[user_answer]['skipped'] += 1

        return res

    # ------------------------------------------------------------
    # GAMIFICATION / BADGES
    # ------------------------------------------------------------

    def _create_certification_badge_trigger(self):
        self.ensure_one()
        goal = self.env['gamification.goal.definition'].create({
            'name': self.title,
            'description': "%s certification passed" % self.title,
            'domain': "['&', ('survey_id', '=', %s), ('quizz_passed', '=', True)]" % self.id,
            'computation_mode': 'count',
            'display_mode': 'boolean',
            'model_id': self.env.ref('survey.model_survey_user_input').id,
            'condition': 'higher',
            'batch_mode': True,
            'batch_distinctive_field': self.env.ref('survey.field_survey_user_input__partner_id').id,
            'batch_user_expression': 'user.partner_id.id'
        })
        challenge = self.env['gamification.challenge'].create({
            'name': _('%s challenge certificate' % self.title),
            'reward_id': self.certification_badge_id.id,
            'state': 'inprogress',
            'period': 'once',
            'category': 'certification',
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
            surveys_with_badge = self.filtered(lambda survey: survey.certification_badge_id
                                                                 and not survey.certification_badge_id.active)
            surveys_with_badge.mapped('certification_badge_id').write({'active': True})
            # (re-)create challenge and goal
            for survey in self:
                survey._create_certification_badge_trigger()
        else:
            # if badge with owner : archive them, else delete everything (badge, challenge, goal)
            badges = self.mapped('certification_badge_id')
            challenges_to_delete = self.env['gamification.challenge'].search([('reward_id', 'in', badges.ids)])
            goals_to_delete = challenges_to_delete.mapped('line_ids').mapped('definition_id')
            badges.write({'active': False})
            # delete all challenges and goals because not needed anymore (challenge lines are deleted in cascade)
            challenges_to_delete.unlink()
            goals_to_delete.unlink()

```

## File: models\survey_user.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
import re
import uuid

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

from dateutil.relativedelta import relativedelta

email_validator = re.compile(r"[^@]+@[^@]+\.[^@]+")
_logger = logging.getLogger(__name__)


def dict_keys_startswith(dictionary, string):
    """Returns a dictionary containing the elements of <dict> whose keys start with <string>.
        .. note::
            This function uses dictionary comprehensions (Python >= 2.7)
    """
    return {k: v for k, v in dictionary.items() if k.startswith(string)}


class SurveyUserInput(models.Model):
    """ Metadata for a set of one user's answers to a particular survey """

    _name = "survey.user_input"
    _rec_name = 'survey_id'
    _description = 'Survey User Input'

    # description
    survey_id = fields.Many2one('survey.survey', string='Survey', required=True, readonly=True, ondelete='cascade')
    scoring_type = fields.Selection(string="Scoring", related="survey_id.scoring_type")
    is_attempts_limited = fields.Boolean("Limited number of attempts", related='survey_id.is_attempts_limited')
    attempts_limit = fields.Integer("Number of attempts", related='survey_id.attempts_limit')
    start_datetime = fields.Datetime('Start date and time', readonly=True)
    is_time_limit_reached = fields.Boolean("Is time limit reached?", compute='_compute_is_time_limit_reached')
    input_type = fields.Selection([
        ('manually', 'Manual'), ('link', 'Invitation')],
        string='Answer Type', default='manually', required=True, readonly=True)
    state = fields.Selection([
        ('new', 'Not started yet'),
        ('skip', 'Partially completed'),
        ('done', 'Completed')], string='Status', default='new', readonly=True)
    test_entry = fields.Boolean(readonly=True)
    # identification and access
    token = fields.Char('Identification token', default=lambda self: str(uuid.uuid4()), readonly=True, required=True, copy=False)
    # no unique constraint, as it identifies a pool of attempts
    invite_token = fields.Char('Invite token', readonly=True, copy=False)
    partner_id = fields.Many2one('res.partner', string='Partner', readonly=True)
    email = fields.Char('E-mail', readonly=True)
    attempt_number = fields.Integer("Attempt n°", compute='_compute_attempt_number')

    # Displaying data
    last_displayed_page_id = fields.Many2one('survey.question', string='Last displayed question/page')
    # answers
    user_input_line_ids = fields.One2many('survey.user_input_line', 'user_input_id', string='Answers', copy=True)
    # Pre-defined questions
    question_ids = fields.Many2many('survey.question', string='Predefined Questions', readonly=True)
    deadline = fields.Datetime('Deadline', help="Datetime until customer can open the survey and submit answers")
    # Stored for performance reasons while displaying results page
    quizz_score = fields.Float("Score (%)", compute="_compute_quizz_score", store=True, compute_sudo=True)
    quizz_passed = fields.Boolean('Quizz Passed', compute='_compute_quizz_passed', store=True, compute_sudo=True)

    @api.depends('user_input_line_ids.answer_score', 'user_input_line_ids.question_id')
    def _compute_quizz_score(self):
        for user_input in self:
            total_possible_score = sum([
                answer_score if answer_score > 0 else 0
                for answer_score in user_input.question_ids.mapped('labels_ids.answer_score')
            ])

            if total_possible_score == 0:
                user_input.quizz_score = 0
            else:
                score = (sum(user_input.user_input_line_ids.mapped('answer_score')) / total_possible_score) * 100
                user_input.quizz_score = round(score, 2) if score > 0 else 0

    @api.depends('quizz_score', 'survey_id')
    def _compute_quizz_passed(self):
        for user_input in self:
            user_input.quizz_passed = user_input.quizz_score >= user_input.survey_id.passing_score

    _sql_constraints = [
        ('unique_token', 'UNIQUE (token)', 'A token must be unique!'),
    ]

    @api.model
    def do_clean_emptys(self):
        """ Remove empty user inputs that have been created manually
            (used as a cronjob declared in data/survey_cron.xml)
        """
        an_hour_ago = fields.Datetime.to_string(datetime.datetime.now() - datetime.timedelta(hours=1))
        self.search([('input_type', '=', 'manually'),
                     ('state', '=', 'new'),
                     ('create_date', '<', an_hour_ago)]).unlink()

    @api.model
    def _generate_invite_token(self):
        return str(uuid.uuid4())

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
            'url': '/survey/print/%s?answer_token=%s' % (self.survey_id.access_token, self.token)
        }

    @api.depends('start_datetime', 'survey_id.is_time_limited', 'survey_id.time_limit')
    def _compute_is_time_limit_reached(self):
        """ Checks that the user_input is not exceeding the survey's time limit. """
        for user_input in self:
            user_input.is_time_limit_reached = user_input.survey_id.is_time_limited and fields.Datetime.now() \
                > user_input.start_datetime + relativedelta(minutes=user_input.survey_id.time_limit)

    @api.depends('state', 'test_entry', 'survey_id.is_attempts_limited', 'partner_id', 'email', 'invite_token')
    def _compute_attempt_number(self):
        attempts_to_compute = self.filtered(
            lambda user_input: user_input.state == 'done' and not user_input.test_entry and user_input.survey_id.is_attempts_limited
        )

        for user_input in (self - attempts_to_compute):
            user_input.attempt_number = 1

        if attempts_to_compute:
            self.env.cr.execute("""SELECT user_input.id, (COUNT(previous_user_input.id) + 1) AS attempt_number
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
                attempt_number = 1
                for attempts_count_result in attempts_count_results:
                    if attempts_count_result['id'] == user_input.id:
                        attempt_number = attempts_count_result['attempt_number']
                        break

                user_input.attempt_number = attempt_number

    def _mark_done(self):
        """ This method will:
        1. mark the state as 'done'
        2. send the certification email with attached document if
        - The survey is a certification
        - It has a certification_mail_template_id set
        - The user succeeded the test
        Will also run challenge Cron to give the certification badge if any."""
        self.write({'state': 'done'})
        Challenge = self.env['gamification.challenge'].sudo()
        badge_ids = []
        for user_input in self:
            if user_input.survey_id.certificate and user_input.quizz_passed:
                if user_input.survey_id.certification_mail_template_id and not user_input.test_entry:
                    user_input.survey_id.certification_mail_template_id.send_mail(user_input.id, notif_layout="mail.mail_notification_light")
                if user_input.survey_id.certification_give_badge:
                    badge_ids.append(user_input.survey_id.certification_badge_id.id)

        if badge_ids:
            challenges = Challenge.search([('reward_id', 'in', badge_ids)])
            if challenges:
                Challenge._cron_update(ids=challenges.ids, commit=False)

    def _get_survey_url(self):
        self.ensure_one()
        return '/survey/start/%s?answer_token=%s' % (self.survey_id.access_token, self.token)


class SurveyUserInputLine(models.Model):
    _name = 'survey.user_input_line'
    _description = 'Survey User Input Line'
    _rec_name = 'user_input_id'
    _order = 'question_sequence,id'

    # survey data
    user_input_id = fields.Many2one('survey.user_input', string='User Input', ondelete='cascade', required=True)
    survey_id = fields.Many2one(related='user_input_id.survey_id', string='Survey', store=True, readonly=False)
    question_id = fields.Many2one('survey.question', string='Question', ondelete='cascade', required=True)
    page_id = fields.Many2one(related='question_id.page_id', string="Section", readonly=False)
    question_sequence = fields.Integer('Sequence', related='question_id.sequence', store=True)
    # answer
    skipped = fields.Boolean('Skipped')
    answer_type = fields.Selection([
        ('text', 'Text'),
        ('number', 'Number'),
        ('date', 'Date'),
        ('datetime', 'Datetime'),
        ('free_text', 'Free Text'),
        ('suggestion', 'Suggestion')], string='Answer Type')
    value_text = fields.Char('Text answer')
    value_number = fields.Float('Numerical answer')
    value_date = fields.Date('Date answer')
    value_datetime = fields.Datetime('Datetime answer')
    value_free_text = fields.Text('Free Text answer')
    value_suggested = fields.Many2one('survey.label', string="Suggested answer")
    value_suggested_row = fields.Many2one('survey.label', string="Row answer")
    answer_score = fields.Float('Score')
    answer_is_correct = fields.Boolean('Correct', compute='_compute_answer_is_correct')

    @api.depends('value_suggested', 'question_id')
    def _compute_answer_is_correct(self):
        for answer in self:
            if answer.value_suggested and answer.question_id.question_type in ['simple_choice', 'multiple_choice']:
                answer.answer_is_correct = answer.value_suggested.is_correct
            else:
                answer.answer_is_correct = False

    @api.constrains('skipped', 'answer_type')
    def _answered_or_skipped(self):
        for uil in self:
            if not uil.skipped != bool(uil.answer_type):
                raise ValidationError(_('This question cannot be unanswered or skipped.'))

    @api.constrains('answer_type')
    def _check_answer_type(self):
        for uil in self:
            fields_type = {
                'text': bool(uil.value_text),
                'number': (bool(uil.value_number) or uil.value_number == 0),
                'date': bool(uil.value_date),
                'free_text': bool(uil.value_free_text),
                'suggestion': bool(uil.value_suggested)
            }
            if not fields_type.get(uil.answer_type, True):
                raise ValidationError(_('The answer must be in the right type'))

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            value_suggested = vals.get('value_suggested')
            if value_suggested:
                vals.update({'answer_score': self.env['survey.label'].browse(int(value_suggested)).answer_score})
        return super(SurveyUserInputLine, self).create(vals_list)

    def write(self, vals):
        value_suggested = vals.get('value_suggested')
        if value_suggested:
            vals.update({'answer_score': self.env['survey.label'].browse(int(value_suggested)).answer_score})
        return super(SurveyUserInputLine, self).write(vals)

    @api.model
    def save_lines(self, user_input_id, question, post, answer_tag):
        """ Save answers to questions, depending on question type

            If an answer already exists for question and user_input_id, it will be
            overwritten (in order to maintain data consistency).
        """
        try:
            saver = getattr(self, 'save_line_' + question.question_type)
        except AttributeError:
            _logger.error(question.question_type + ": This type of question has no saving function")
            return False
        else:
            saver(user_input_id, question, post, answer_tag)

    @api.model
    def save_line_free_text(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False,
        }
        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'free_text', 'value_free_text': post[answer_tag]})
        else:
            vals.update({'answer_type': None, 'skipped': True})
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        if old_uil:
            old_uil.write(vals)
        else:
            old_uil.create(vals)
        return True

    @api.model
    def save_line_textbox(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'text', 'value_text': post[answer_tag]})
        else:
            vals.update({'answer_type': None, 'skipped': True})
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        if old_uil:
            old_uil.write(vals)
        else:
            old_uil.create(vals)
        return True

    @api.model
    def save_line_numerical_box(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'number', 'value_number': float(post[answer_tag])})
        else:
            vals.update({'answer_type': None, 'skipped': True})
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        if old_uil:
            old_uil.write(vals)
        else:
            old_uil.create(vals)
        return True

    @api.model
    def save_line_date(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'date', 'value_date': post[answer_tag]})
        else:
            vals.update({'answer_type': None, 'skipped': True})
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        if old_uil:
            old_uil.write(vals)
        else:
            old_uil.create(vals)
        return True

    @api.model
    def save_line_datetime(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'datetime', 'value_datetime': post[answer_tag]})
        else:
            vals.update({'answer_type': None, 'skipped': True})
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        if old_uil:
            old_uil.write(vals)
        else:
            old_uil.create(vals)
        return True

    @api.model
    def save_line_simple_choice(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        old_uil.sudo().unlink()

        if answer_tag in post and post[answer_tag].strip():
            vals.update({'answer_type': 'suggestion', 'value_suggested': int(post[answer_tag])})
        else:
            vals.update({'answer_type': None, 'skipped': True})

        # '-1' indicates 'comment count as an answer' so do not need to record it
        if post.get(answer_tag) and post.get(answer_tag) != '-1':
            self.create(vals)

        comment_answer = post.pop(("%s_%s" % (answer_tag, 'comment')), '').strip()
        if comment_answer:
            vals.pop('answer_score', False)
            vals.update({'answer_type': 'text', 'value_text': comment_answer, 'skipped': False, 'value_suggested': False})
            self.create(vals)

        return True

    @api.model
    def save_line_multiple_choice(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        old_uil.sudo().unlink()

        ca_dict = dict_keys_startswith(post, answer_tag + '_')
        comment_answer = ca_dict.pop(("%s_%s" % (answer_tag, 'comment')), '').strip()
        if len(ca_dict) > 0:
            for key in ca_dict:
                # '-1' indicates 'comment count as an answer' so do not need to record it
                if key != ('%s_%s' % (answer_tag, '-1')):
                    val = ca_dict[key]
                    vals.update({'answer_type': 'suggestion', 'value_suggested': bool(val) and int(val)})
                    self.create(vals)
        if comment_answer:
            vals.update({'answer_type': 'text', 'value_text': comment_answer, 'value_suggested': False})
            self.create(vals)
        if not ca_dict and not comment_answer:
            vals.update({'answer_type': None, 'skipped': True})
            self.create(vals)
        return True

    @api.model
    def save_line_matrix(self, user_input_id, question, post, answer_tag):
        vals = {
            'user_input_id': user_input_id,
            'question_id': question.id,
            'survey_id': question.survey_id.id,
            'skipped': False
        }
        old_uil = self.search([
            ('user_input_id', '=', user_input_id),
            ('survey_id', '=', question.survey_id.id),
            ('question_id', '=', question.id)
        ])
        old_uil.sudo().unlink()

        no_answers = True
        ca_dict = dict_keys_startswith(post, answer_tag + '_')

        comment_answer = ca_dict.pop(("%s_%s" % (answer_tag, 'comment')), '').strip()
        if comment_answer:
            vals.update({'answer_type': 'text', 'value_text': comment_answer})
            self.create(vals)
            no_answers = False

        if question.matrix_subtype == 'simple':
            for row in question.labels_ids_2:
                a_tag = "%s_%s" % (answer_tag, row.id)
                if a_tag in ca_dict:
                    no_answers = False
                    vals.update({'answer_type': 'suggestion', 'value_suggested': ca_dict[a_tag], 'value_suggested_row': row.id})
                    self.create(vals)

        elif question.matrix_subtype == 'multiple':
            for col in question.labels_ids:
                for row in question.labels_ids_2:
                    a_tag = "%s_%s_%s" % (answer_tag, row.id, col.id)
                    if a_tag in ca_dict:
                        no_answers = False
                        vals.update({'answer_type': 'suggestion', 'value_suggested': col.id, 'value_suggested_row': row.id})
                        self.create(vals)
        if no_answers:
            vals.update({'answer_type': None, 'skipped': True})
            self.create(vals)
        return True

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_autovacuum
from . import survey_survey
from . import survey_question
from . import survey_user
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
access_survey_label_all,survey.label.all,model_survey_label,,0,0,0,0
access_survey_label_user,survey.label.user,model_survey_label,base.group_user,0,0,0,0
access_survey_label_survey_user,survey.label.survey.user,model_survey_label,group_survey_user,1,1,1,1
access_survey_label_survey_manager,survey.label.survey.manager,model_survey_label,group_survey_manager,1,1,1,1
access_survey_user_input_all,survey.user_input.all,model_survey_user_input,,0,0,0,0
access_survey_user_input_user,survey.user_input.user,model_survey_user_input,base.group_user,0,0,0,0
access_survey_user_input_survey_user,survey.user_input.survey.user,model_survey_user_input,group_survey_user,1,1,1,1
access_survey_user_input_survey_manager,survey.user_input.survey.manager,model_survey_user_input,group_survey_manager,1,1,1,1
access_survey_user_input_line_all,survey.user_input_line.all,model_survey_user_input_line,,0,0,0,0
access_survey_user_input_line_user,survey.user_input_line.user,model_survey_user_input_line,base.group_user,0,0,0,0
access_survey_user_input_line_survey_user,survey.user_input_line.survey.user,model_survey_user_input_line,group_survey_user,1,1,1,1
access_survey_user_input_line_survey_manager,survey.user_input_line.survey.manager,model_survey_user_input_line,group_survey_manager,1,1,1,1
access_gamification_badge_survey_user,gamification.badge.survey.user,model_gamification_badge,group_survey_user,1,1,1,1

```

## File: security\survey_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.module.category" id="base.module_category_marketing_survey">
            <field name="description">Helps you manage your survey for review of different-different users.</field>
            <field name="sequence">20</field>
        </record>

        <!-- Survey users -->
        <record model="res.groups" id="group_survey_user">
            <field name="name">User</field>
            <field name="category_id" ref="base.module_category_marketing_survey"/>
        </record>

        <!-- Survey managers -->
        <record model="res.groups" id="group_survey_manager">
            <field name="name">Administrator</field>
            <field name="category_id" ref="base.module_category_marketing_survey"/>
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

        <record id="survey_label_rule_survey_manager" model="ir.rule">
            <field name="name">Survey label: manager: all</field>
            <field name="model_id" ref="survey.model_survey_label"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_manager'))]"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
        </record>
        <record id="survey_label_rule_survey_user_read" model="ir.rule">
            <field name="name">Survey label: officer: read all</field>
            <field name="model_id" ref="survey.model_survey_label"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[(4, ref('group_survey_user'))]"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="0"/>
        </record>
        <record id="survey_label_rule_survey_user_cw" model="ir.rule">
            <field name="name">Survey label: officer: create/write/unlink linked to own survey only</field>
            <field name="model_id" ref="survey.model_survey_question"/>
            <field name="domain_force">[('survey_id.create_uid', '=', user.id)]</field>
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

## File: static\src\img\trophy-solid.svg

```svg
<svg aria-hidden="true" focusable="false" data-prefix="fas" data-icon="trophy" class="svg-inline--fa fa-trophy fa-w-18" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512"><path fill="currentColor" d="M552 64H448V24c0-13.3-10.7-24-24-24H152c-13.3 0-24 10.7-24 24v40H24C10.7 64 0 74.7 0 88v56c0 35.7 22.5 72.4 61.9 100.7 31.5 22.7 69.8 37.1 110 41.7C203.3 338.5 240 360 240 360v72h-48c-35.3 0-64 20.7-64 56v12c0 6.6 5.4 12 12 12h296c6.6 0 12-5.4 12-12v-12c0-35.3-28.7-56-64-56h-48v-72s36.7-21.5 68.1-73.6c40.3-4.6 78.6-19 110-41.7 39.3-28.3 61.9-65 61.9-100.7V88c0-13.3-10.7-24-24-24zM99.3 192.8C74.9 175.2 64 155.6 64 144v-16h64.2c1 32.6 5.8 61.2 12.8 86.2-15.1-5.2-29.2-12.4-41.7-21.4zM512 144c0 16.1-17.7 36.1-35.3 48.8-12.5 9-26.7 16.2-41.8 21.4 7-25 11.8-53.6 12.8-86.2H512v16z"></path></svg>
```

## File: static\src\js\fields_section_one2many.js

```javascript
odoo.define('survey.question_page_one2many', function (require){
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
    _checkIfRecordIsSection: function (id){
        var record = this._findRecordById(id);
        return record && record.data[this.sectionFieldName];
    },
    _findRecordById: function (id){
        return _.find(this.state.data, function (record){
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
    _renderBodyCell: function (record, node, index, options){
        var $cell = this._super.apply(this, arguments);

        var isSection = record.data[this.sectionFieldName];

        if (isSection){
            if (node.attrs.widget === "handle" || node.attrs.name === "random_questions_count"){
                return $cell;
            } else if (node.attrs.name === "title"){
                var nbrColumns = this._getNumberOfCols();
                if (this.handleField){
                    nbrColumns--;
                }
                if (this.addTrashIcon){
                    nbrColumns--;
                }
                if (record.data.questions_selection === "random"){
                    nbrColumns--;
                }
                $cell.attr('colspan', nbrColumns);
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
    _renderRow: function (record, index){
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
    _renderView: function (){
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
    _onRowClicked: function (ev){
        var parent = this.getParent();
        var recordId = $(ev.currentTarget).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit"){
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
    _onCellClick: function (ev){
        var parent = this.getParent();
        var recordId = $(ev.currentTarget.parentElement).data('id');
        var is_section = this._checkIfRecordIsSection(recordId);
        if (is_section && parent.mode === "edit"){
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
    _onNavigationMove: function (ev){
        this.unselectRow();
    },
});

var SectionFieldOne2Many = FieldOne2Many.extend({
    init: function (parent, name, record, options){
        this._super.apply(this, arguments);
        this.sectionFieldName = "is_page";
        this.rendered = false;
    },
    /**
     * Overridden to use our custom renderer
     * @private
     * @override
     */
    _getRenderer: function (){
        if (this.view.arch.tag === 'tree'){
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
        if (!config.device.isMobile){
            var context_str = ev.data.context && ev.data.context[0];
            var context = new Context(context_str).eval();
            if (context['default_' + this.sectionFieldName]){
                this.editable = "bottom";
            }
        }
        this._super.apply(this, arguments);
    },
});

FieldRegistry.add('question_page_one2many', SectionFieldOne2Many);
});
```

## File: static\src\js\survey.js

```javascript
odoo.define('survey.survey', function (require) {
'use strict';

require('web.dom_ready');
var core = require('web.core');
var session = require('web.session');
var time = require('web.time');
var field_utils = require('web.field_utils');
var Dialog = require('web.Dialog');

var _t = core._t;
/*
 * This file is intended to add interactivity to survey forms
 */

var the_form = $('.js_surveyform');

if(!the_form.length) {
    return Promise.reject("DOM doesn't contain '.js_surveyform'");
}

    var prefill_controller = the_form.attr("data-prefill");
    var submit_controller = the_form.attr("data-submit");
    var scores_controller = the_form.attr("data-scores");
    var print_mode = false;
    var quiz_correction_mode = false;

    // Printing mode: will disable all the controls in the form
    if (_.isUndefined(submit_controller)) {
        $(".js_surveyform .input-group-text span.fa-calendar").css("pointer-events", "none");
        $('.js_surveyform :input').prop('disabled', true);
        print_mode = true;
    }

    // Quiz correction mode
    if (! _.isUndefined(scores_controller)) {
        quiz_correction_mode = true;
    }


    // Custom code for right behavior of radio buttons with comments box
    $('.js_comments>input[type="text"]').focusin(function(){
        $(this).prev().find('>input').attr("checked","checked");
    });
    $('.js_radio input[type="radio"][data-oe-survey-otherr!="1"]').click(function(){
        $(this).closest('.js_radio').find('.js_comments>input[type="text"]').val("");
    });
    $('.js_comments input[type="radio"]').click(function(){
        $(this).closest('.js_comments').find('>input[data-oe-survey-othert="1"]').focus();
    });
    // Custom code for right behavior of dropdown menu with comments
    $('.js_drop input[data-oe-survey-othert="1"]').hide();
    $('.js_drop select').change(function(){
        var other_val = $(this).find('.js_other_option').val();
        if($(this).val() === other_val){
            $(this).parent().removeClass('col-lg-12').addClass('col-lg-6');
            $(this).closest('.js_drop').find('input[data-oe-survey-othert="1"]').show().focus();
        }
        else{
            $(this).parent().removeClass('col-lg-6').addClass('col-lg-12');
            $(this).closest('.js_drop').find('input[data-oe-survey-othert="1"]').val("").hide();
        }
    });
    // Custom code for right behavior of checkboxes with comments box
    $('.js_ck_comments>input[type="text"]').focusin(function(){
        $(this).prev().find('>input').attr("checked","checked");
    });
    $('.js_ck_comments input[type="checkbox"]').change(function(){
        if (! $(this).prop("checked")){
            $(this).closest('.js_ck_comments').find('input[type="text"]').val("");
        }
    });

    // Pre-filling of the form with previous answers
    function prefill(){
        if (! _.isUndefined(prefill_controller)) {
            var prefill_def = $.ajax(prefill_controller, {dataType: "json"})
                .done(function(json_data){
                    _.each(json_data, function(value, key){

                        // prefill of text/number/date boxes
                        var input = the_form.find(".form-control[name=" + key + "]");
                        if (input.hasClass('datetimepicker-input')) {
                            // display dates in user timezone
                            var moment_date = field_utils.parse.date(value[0]);
                            if ($(input).closest('.input-group.date').data('questiontype') === 'datetime') {
                                value = field_utils.format.datetime(moment_date, null, {timezone: true});
                            }
                            else {
                                value = field_utils.format.date(moment_date, null, {timezone: true});
                            }
                        }
                        input.val(value);

                        // special case for comments under multiple suggestions questions
                        if (_.string.endsWith(key, "_comment") &&
                            (input.parent().hasClass("js_comments") || input.parent().hasClass("js_ck_comments"))) {
                            input.siblings().find('>input').attr("checked","checked");
                        }

                        // checkboxes and radios
                        the_form.find("input[name^='" + key + "_'][type='checkbox']").each(function(){
                            $(this).val(value);
                        });
                        the_form.find("input[name=" + key + "][type!='text']").each(function(){
                            $(this).val(value);
                        });
                    });
                })
                .fail(function(){
                    console.warn("[survey] Unable to load prefill data");
                });
            return prefill_def;
        }
    }

    // Display score if quiz correction mode
    function display_scores(){
        if (! _.isUndefined(scores_controller)) {
            var score_def = $.ajax(scores_controller, {dataType: "json"})
                .done(function(json_data){
                    _.each(json_data, function(value, key){
                        the_form.find("span[data-score-question=" + key + "]").text("Your score: " + value);
                    });
                })
                .fail(function(){
                    console.warn("[survey] Unable to load score data");
                });
            return score_def;
        }
    }

    $('.o_survey_header .breadcrumb-item a').each(function() {
        var $valObj = $(this);
        $valObj.click(function(event) {
            event.preventDefault();
            $('<input>').attr({type: 'hidden', name: 'breadcrumb_redirect', value: $valObj.attr('href')}).appendTo('.js_surveyform');
            $('.js_surveyform').submit();
        });
    });

    // Parameters for form submission
    $('.js_surveyform').ajaxForm({
        url: submit_controller,
        type: 'POST',                       // submission type
        dataType: 'json',                   // answer expected type
        beforeSubmit: function(formData, $form, options){           // hide previous errmsg before resubmitting
            var date_fields = $form.find('div.date > input.form-control');
            for (var i=0; i < date_fields.length; i++) {
                var el = date_fields[i];
                var questiontype = $(el).closest('.input-group.date').data('questiontype');
                var moment_date = questiontype === 'datetime' ? field_utils.parse.datetime(el.value, null, {timezone: true}) : field_utils.parse.date(el.value);

                var field_obj = _.findWhere(formData, {'name': el.name});
                field_obj.value = moment_date ? moment_date.toJSON() : '';
            }
            $('.js_errzone').html("").hide();
        },
        success: function(response, status, xhr, wfe){ // submission attempt
            if(_.has(response, 'errors')){  // some questions have errors
                _.each(_.keys(response.errors), function(key){
                    $("#" + key + '>.js_errzone').append($('<p>', {'text': response.errors[key]})).show();
                    if (_.keys(response.errors)[_.keys(response.errors).length - 1] === key) {
                         $('html, body').animate({
                            scrollTop: $('.js_errzone:visible:first').closest('.js_question-wrapper').offset().top - $('.o_main_navbar').height()
                        }, 500);
                    }
                });
                return false;
            }
            else if (_.has(response, 'redirect')){      // form is ok
                window.location.replace(response.redirect);
                return true;
            }
            else {                                      // server sends bad data
                console.error("Incorrect answer sent by server");
                return false;
            }
        },
        timeout: 5000,
        error: function(jqXHR, textStatus, errorThrown){ // failure of AJAX request
            $('#AJAXErrorModal').modal('show');
        }
    });

    // datetimepicker use moment locale to display date format according to language
    // frontend does not load moment locale at all.
    // so wait until DOM ready with locale then init datetimepicker
    session.load_translations().then(function(){
        _.each($('.input-group.date'), function(date_field){
            var disabledDates = [];
            var minDate, maxDate;

            // Set the datetimepicker format depending on the question type
            var datetimepickerFormat = $(date_field).data('questiontype') === 'datetime' ? time.getLangDatetimeFormat() : time.getLangDateFormat();

            // Retrieving min date & format it for datetimepicker compatibility
            if ($(date_field).data('mindate')) {
                minDate = moment(field_utils.format.datetime(moment($(date_field).data('mindate')), null, {timezone: true}), datetimepickerFormat);
            }

            // Retrieving max date & format it for datetimepicker compatibility
            if ($(date_field).data('maxdate')) {
                maxDate = moment(field_utils.format.datetime(moment($(date_field).data('maxdate')), null, {timezone: true}), datetimepickerFormat);
            }

            // Fallback in case dates are invalid or empty
            minDate = minDate ? minDate : moment({ y: 1900 });
            maxDate = maxDate ? maxDate : moment().add(200, "y");

            // Setting up maxDate & disabledDates for date-only questions
            if ($(date_field).data('questiontype') === 'date') {
                maxDate = maxDate.add(1, "d");
                disabledDates = [maxDate];
            }

            $('#' + date_field.id).datetimepicker({
                format: datetimepickerFormat,
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
                keyBinds: null,
            });
            $('#' + date_field.id).on('error.datetimepicker', function (err) {
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
        });
        // Launch prefilling
        prefill();
        if(quiz_correction_mode === true){
            display_scores();
        }
    });
});

```

## File: static\src\js\survey_result.js

```javascript
odoo.define('survey.result', function (require) {
'use strict';

require('web.dom_ready');
var _t = require('web.core')._t;

if(!$('.js_surveyresult').length) {
    return Promise.reject("DOM doesn't contain '.js_surveyresult'");
}

    // The given colors are the same as those used by D3
    var D3_COLORS = ["#1f77b4","#ff7f0e","#aec7e8","#ffbb78","#2ca02c","#98df8a","#d62728",
                        "#ff9896","#9467bd","#c5b0d5","#8c564b","#c49c94","#e377c2","#f7b6d2",
                        "#7f7f7f","#c7c7c7","#bcbd22","#dbdb8d","#17becf","#9edae5"];

    //Script For Pagination
    var survey_pagination = $('.pagination');
    $.each(survey_pagination, function(index, pagination){
        var question_id = $(pagination).attr("data-question_id");
        var limit = $(pagination).attr("data-record_limit"); //Number of Record Par Page. If you want to change number of record per page, change record_limit in pagination template.
        $('#table_question_'+ question_id +' tbody tr:lt('+limit+')').removeClass('d-none');
        $('#pagination_'+question_id+' li a').click(function(event){
            event.preventDefault();
            $('#pagination_'+question_id+' li').removeClass('active');
            $(this).parent('li').addClass('active');
            $('#table_question_'+ question_id +' tbody tr').addClass('d-none');
            var num = $(this).text();
            var min = (limit * (num-1))-1;
            if (min == -1){
                $('#table_question_'+ question_id +' tbody tr:lt('+ limit * num +')').removeClass('d-none');
            }
            else{
                $('#table_question_'+question_id+' tbody tr:lt('+ limit * num +'):gt('+min+')').removeClass('d-none');
            }
        });
        $('#pagination_'+question_id+' li:first').addClass('active').find('a').click();
    });

    // Custom Tick fuction for replacing long text with '...'
    var customtick_function = function (tick_limit) {
        return function(label) {
            if (label.length <= tick_limit) {
                return label;
            }
            else {
                return label.slice(0, tick_limit) + '...';
            }
        };
    };

    //initialize MultiBar Chart
    function init_multibar_chart (graph_data) {
        var chartConfig = {
            type: 'bar',
            data: {
                labels: graph_data[0].values.map(function (value) {
                    return value.text;
                }),
                datasets: graph_data.map(function (group, index) {
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
                            callback: customtick_function(25),
                        },
                    }],
                    yAxes: [{
                        ticks: {
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
        return chartConfig;
    }

    //initialize discreteBar Chart
    function init_bar_chart(graph_data){
        var chartConfig = {
            type: 'bar',
            data: {
                labels: graph_data[0].values.map(function (value) {
                    return value.text;
                }),
                datasets: graph_data.map(function (group) {
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
                            callback: customtick_function(35),
                        },
                    }],
                    yAxes: [{
                        ticks: {
                                precision: 0,
                        },
                    }],
                },
                tooltips: {
                    enabled: false,
                }
            },
        };
        return chartConfig;
    }

    //initialize Pie Chart
    function init_pie_chart(graph_data){
        var data = graph_data.map(function (point) {
            return point.count;
        });
        var chartConfig = {
            type: 'pie',
            data: {
                labels: graph_data.map(function (point) {
                    return point.text;
                }),
                datasets: [{
                    label: '',
                    data: data,
                    backgroundColor: data.map(function (val, index) {
                        return D3_COLORS[index % 20];
                    }),
                }]
        }
        };
        return chartConfig;
    }

    //initialize doughnut Chart
    function init_doughnut_chart(graph_data, quizz_score){
        var data = graph_data.map(function (point) {
            return point.count;
        });
        var chartConfig = {
            type: 'doughnut',
            data: {
                labels: graph_data.map(function (point) {
                    return point.text;
                }),
                datasets: [{
                    label: '',
                    data: data,
                    backgroundColor: data.map(function (val, index) {
                        return D3_COLORS[index % 20];
                    }),
                }]
            },
            options: {
                title: {
                    display: true,
                    text: _.str.sprintf(_t("Overall Performance %.2f%s"), parseFloat(quizz_score), '%'),
                },
            }
        };
        return chartConfig;
    }

    //load chart to svg element chart:initialized chart, response:AJAX response, quistion_id:if of survey question, tick_limit:text length limit
    function load_chart(chartConfig, containerSelector){
        var $container = $(containerSelector).css({position: 'relative'});
        var $canvas = $container.find('canvas');
        var ctx = $canvas.get(0).getContext('2d');
        return new Chart(ctx, chartConfig);
    }

    //Script For Graph
    var survey_graphs = $('.survey_graph');
    $.each(survey_graphs, function(index, graph){
        var question_id = $(graph).attr("data-question_id");
        var graph_type = $(graph).attr("data-graph_type");
        var graph_data = JSON.parse($(graph).attr("graph-data"));
        var containerSelector = '#graph_question_' + question_id;
        var chartConfig;
        if(graph_type == 'multi_bar'){
            chartConfig = init_multibar_chart(graph_data);
            load_chart(chartConfig, containerSelector);
        }
        else if(graph_type == 'bar'){
            chartConfig = init_bar_chart(graph_data);
            load_chart(chartConfig, containerSelector);
        }
        else if(graph_type == 'pie'){
            chartConfig = init_pie_chart(graph_data);
            load_chart(chartConfig, containerSelector);
        }
        else if (graph_type === 'doughnut') {
            var quizz_score = $(graph).attr("quizz-score") || 0.0;
            chartConfig = init_doughnut_chart(graph_data, quizz_score);
            return load_chart(chartConfig, containerSelector);
        }
    });

    var $scoringResultsChart = $('#scoring_results_chart');
    if ($scoringResultsChart.length > 0) {
        var chartConfig = init_pie_chart($scoringResultsChart.data('graph_data'));
        load_chart(chartConfig, '#scoring_results_chart');
    }

    // Script for filter
    $('td.survey_answer').hover(function(){
        $(this).find('i.fa-filter').removeClass('invisible');
    }, function(){
        $(this).find('i.fa-filter').addClass('invisible');
    });
    $('td.survey_answer i.fa-filter').click(function(){
        var cell = $(this);
        var row_id = cell.attr('data-row_id') | 0;
        var answer_id = cell.attr('data-answer_id');
        if(document.URL.indexOf("?") == -1){
            window.location.href = document.URL + '?' + encodeURI(row_id + ',' + answer_id);
        }
        else {
            window.location.href = document.URL + '&' + encodeURI(row_id + ',' + answer_id);
        }
    });

    // for clear all filters
    $('.clear_survey_filter').click(function(){
        window.location.href = document.URL.substring(0,document.URL.indexOf("?"));
    });
    $('span.filter-all').click(function(){
        event.preventDefault();
        if(document.URL.indexOf("finished") != -1){
            window.location.href = document.URL.replace('?finished&','?').replace('&finished&','&').replace('?finished','').replace('&finished','');
        }
    }).hover(function(){
        if(document.URL.indexOf("finished") == -1){
            $(this)[0].style.cursor = 'default';
        }
    });
    // toggle finished/all surveys filter
    $('span.filter-finished').click(function(){
        event.preventDefault();
        if(document.URL.indexOf("?") == -1){
            window.location.href = document.URL + '?' + encodeURI('finished');
        }
        else if(document.URL.indexOf("finished") == -1){
            window.location.href = document.URL + '&' + encodeURI('finished');
        }
    }).hover(function(){
        if(document.URL.indexOf("finished") != -1){
            $(this)[0].style.cursor = 'default';
        }
    });

});

```

## File: static\src\js\survey_timer.js

```javascript
odoo.define('survey.timer', function (require) {
'use strict';

require('web.dom_ready');

if (!$('.js_survey_timer').length) {
    return Promise.reject("DOM doesn't contain '.js_survey_timer'");
}

var $parent = $('.js_survey_timer');
var timeLimitMinutes = parseInt($parent.data('time_limit_minutes'));

if (timeLimitMinutes <= 0) {
    return Promise.reject("Timer is not positive");
}

var countDownDate = moment.utc($parent.data('timer')).add(timeLimitMinutes, 'minutes');

if (countDownDate.diff(moment.utc(), 'seconds') < 0) {
    return Promise.reject("Timer is already finished");
}

var formatTime = function (time) {
    return time > 9 ? time : '0' + time;
};

var $timer = $parent.find('.timer');
var interval = null;
var updateTimer = function () {
    var timeLeft = countDownDate.diff(moment.utc(), 'seconds');

    if (timeLeft >= 0) {
        var timeLeftMinutes = parseInt(timeLeft / 60);
        var timeLeftSeconds = timeLeft - (timeLeftMinutes * 60);
        $timer.html(formatTime(timeLeftMinutes) + ':' + formatTime(timeLeftSeconds));
    } else {
        clearInterval(interval);
        $('.js_surveyform').submit();
    }
};

updateTimer();
interval = setInterval(updateTimer, 1000);

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- survey assets  -->
    <template id="survey_assets" name="Survey Results assets">
        <script src="/web/static/lib/Chart/Chart.js"></script>

        <script type="text/javascript" src="/web/static/src/js/fields/field_utils.js"></script>

        <script type="text/javascript" src="/survey/static/src/js/survey_result.js" />
        <script type="text/javascript" src="/survey/static/src/js/survey.js" />
        <script type="text/javascript" src="/survey/static/src/js/survey_timer.js" />

        <link href="/survey/static/src/css/survey_print.css" rel="stylesheet" type="text/css"/>
        <link href="/survey/static/src/css/survey_result.css" rel="stylesheet" type="text/css"></link>
        <link rel="stylesheet" type="text/scss" href="/survey/static/src/scss/survey_templates.scss"/>
    </template>

    <template id="survey_report_assets_pdf" inherit_id="web.report_assets_pdf">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/survey/static/src/scss/survey_reports.scss"/>
        </xpath>
    </template>

    <template id="assets_backend" name="survey assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/survey/static/src/css/survey_result.css"/>
            <script type="text/javascript" src="/survey/static/src/js/fields_section_one2many.js"/>
        </xpath>
    </template>

    <template id="assets_backend_inherit_survey" inherit_id="web.assets_backend" name="Survey backend assets">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/survey/static/src/scss/survey_views.scss"/>
        </xpath>
    </template>

    <template id="assets_tests" name="Survey Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/survey/static/tests/tours/certification_failure.js"></script>
            <script type="text/javascript" src="/survey/static/tests/tours/certification_success.js"></script>
            <script type="text/javascript" src="/survey/static/tests/tours/survey.js"></script>
            <script type="text/javascript" src="/survey/static/tests/tours/survey_prefill.js"></script>
        </xpath>
    </template>
</data>
</odoo>

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
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="Badge Name"/>
                        </h1>
                    </div>
                    <group>
                        <field name="description" nolabel="1" placeholder="Badge Description"/>
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
        <field name="context">{'search_default_quizz_passed': 1}</field>
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
    	sequence="70"
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
                <field name="survey_id" invisible="1"/>
                <field name="sequence" invisible="1"/>
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
                                <div attrs="{'invisible': [('question_type', '!=', 'free_text')]}">
                                        <i class="fa fa-align-justify fa-4x" role="img" aria-label="Multiple lines" title="Multiple Lines"/>
                                </div>
                                <!-- Single Line Text Zone -->
                                <div attrs="{'invisible': [('question_type', '!=', 'textbox')]}">
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
                                        <div class="col-lg-3"></div>
                                        <div class="col-lg-3">ans</div>
                                        <div class="col-lg-3">ans</div>
                                        <div class="col-lg-3">ans</div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-lg-3">Row1</div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-lg-3">Row2</div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                    </div>
                                    <div class="row o_matrix_row">
                                        <div class="col-lg-3">Row3</div>
                                        <div class="col-lg-3"><i class="fa fa-dot-circle-o fa-lg" role="img" aria-label="Checked" title="Checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                        <div class="col-lg-3"><i class="fa fa-circle-o fa-lg" role="img" aria-label="Not checked" title="Not checked"/></div>
                                    </div>
                                </div>
                            </div>
                        </group>
                    </group>
                    <notebook attrs="{'invisible': [('is_page', '=', True)]}">
                        <page string="Answers">
                            <field name="validation_email" attrs="{'invisible': [('question_type', '!=', 'textbox')]}"/>
                            <label for="validation_email" attrs="{'invisible': [('question_type', '!=', 'textbox')]}"/>
                            <field name="page_id" invisible="1" required="0"/>
                            <field name="survey_id" invisible="1" readonly="1"/>
                            <field name="scoring_type" invisible="1"/>
                            <separator />
                            <label for="labels_ids" string="Columns of the Matrix" attrs="{'invisible': [('question_type', '!=', 'matrix')]}" />
                            <field name="labels_ids" string="Type of answers" context="{'default_question_id': active_id}" attrs="{'invisible': [('question_type', 'not in', ['simple_choice', 'multiple_choice', 'matrix'])]}">
                                <tree editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="value" string="Choices"/>
                                    <field name="is_correct" attrs="{'column_invisible': ['|', ('parent.scoring_type', '=', 'no_scoring'), ('parent.question_type', '=', 'matrix')]}"/>
                                    <field name="answer_score" attrs="{'column_invisible': ['|', ('parent.scoring_type', '=', 'no_scoring'), ('parent.question_type', '=', 'matrix')]}"/>
                                </tree>
                            </field>
                            <separator />
                            <label for="labels_ids_2" attrs="{'invisible': [('question_type', '!=', 'matrix')]}" />
                            <field name="labels_ids_2" context="{'default_question_id_2': active_id}" attrs="{'invisible': [('question_type', '!=', 'matrix')]}">
                                <tree editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="value" string="Rows"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Options">
                            <group string="Constraints">
                                <group colspan="2" col="4">
                                    <field name="constr_mandatory" string="Mandatory Answer"/>
                                    <field name="constr_error_msg" attrs="{'invisible': [('constr_mandatory', '=', False)]}"/>
                                </group>
                                <div colspan="2" attrs="{'invisible': [('question_type', 'not in', ['textbox', 'numerical_box', 'date', 'datetime'])]}">
                                    <group>
                                        <field name="validation_required" attrs="{'invisible': [('validation_email', '=', True), ('question_type', '=', 'textbox')]}"/>
                                    </group>
                                    <group col="4" attrs="{'invisible': [('validation_required', '=', False)]}">
                                        <field name="validation_length_min" attrs="{'invisible': [('question_type', '!=', 'textbox')]}"/>
                                        <field name="validation_length_max" attrs="{'invisible': [('question_type', '!=', 'textbox')]}"/>
                                        <field name="validation_min_float_value" attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}"/>
                                        <field name="validation_max_float_value" attrs="{'invisible': [('question_type', '!=', 'numerical_box')]}"/>
                                        <field name="validation_min_date" attrs="{'invisible': [('question_type', '!=', 'date')]}"/>
                                        <field name="validation_max_date" attrs="{'invisible': [('question_type', '!=', 'date')]}"/>
                                        <field name="validation_min_datetime" widget="datetime" attrs="{'invisible': [('question_type', '!=', 'datetime')]}"/>
                                        <field name="validation_max_datetime" widget="datetime" attrs="{'invisible': [('question_type', '!=', 'datetime')]}"/>
                                        <field name="validation_error_msg" colspan="4"/>
                                    </group>
                                </div>
                                <group>
                                    <field name="matrix_subtype" attrs="{'invisible':[('question_type','not in',['matrix'])],'required':[('question_type','=','matrix')]}"/>
                                </group>
                            </group>
                            <group string="Display mode" attrs="{'invisible':[('question_type','not in',['simple_choice', 'multiple_choice'])]}">
                                <field name="display_mode" string="Format" attrs="{'invisible':[('question_type','not in',['simple_choice'])],'required':[('question_type','=','simple_choice')]}"/>
                                <field name="column_nb" string="Number of columns" attrs="{'invisible':[('display_mode','=','dropdown'), ('question_type','=','simple_choice')]}"/>
                            </group>
                            <group string="Allow Comments" attrs="{'invisible':[('question_type','not in',['simple_choice','multiple_choice', 'matrix'])]}">
                                <field name='comments_allowed' />
                                <field name='comments_message' attrs="{'invisible': [('comments_allowed', '=', False)]}"/>
                                <field name='comment_count_as_answer' attrs="{'invisible': ['|', ('comments_allowed', '=', False), ('question_type', 'in', ['matrix'])]}" />
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
                <field name="question"/>
                <field name="survey_id"/>
                <field name="question_type"/>
            </tree>
        </field>
    </record>
    <record model="ir.ui.view" id="survey_question_search">
        <field name="name">Search view for survey question</field>
        <field name="model">survey.question</field>
        <field name="arch" type="xml">
            <search string="Search Question">
                <field name="question" string="Question"/>
                <field name="survey_id" string="Survey"/>
                <field name="question_type" string="Type"/>
                <group expand="1" string="Group By">
                    <filter name="group_by_type" string="Type" domain="[]"  context="{'group_by':'question_type'}"/>
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
    <record model="ir.ui.view" id="survey_label_tree">
        <field name="name">survey_label_tree</field>
        <field name="model">survey.label</field>
        <field name="arch" type="xml">
            <tree string="Survey Label" create="false">
                <field name="sequence" widget="handle"/>
                <field name="question_id"/>
                <field name="question_id_2"/>
                <field name="value"/>
                <field name="answer_score" groups="base.group_no_one"/>
            </tree>
        </field>
    </record>
    <record id="survey_label_search" model="ir.ui.view">
        <field name="name">survey_label_search</field>
        <field name="model">survey.label</field>
        <field name="arch" type="xml">
            <search string="Search Label">
                <field name="question_id" string="Question"/>
                <filter name="group_by_question" string="Question" domain="[]" context="{'group_by':'question_id'}"/>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_survey_label_form">
        <field name="name">Suggested Values</field>
        <field name="res_model">survey.label</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="survey_label_search"/>
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
        action="action_survey_label_form"
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
        <!-- QWeb Reports -->
        <report
            id="certification_report"
            model="survey.user_input"
            string="Certifications"
            report_type="qweb-pdf"
            name="survey.certification_report_view"
            file="survey.certification_report_view"
            attachment="'certification.pdf'"
            print_report_name="'Certification - %s' % (object.survey_id.display_name)"
        />
    </data>
</odoo>

```

## File: views\survey_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <template id="certification_report_view">
        <t t-set="data_report_landscape" t-value="True"/>
        <t t-set="full_width" t-value="True"/>
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="user_input">
                <div t-att-data-oe-model="user_input._name" t-att-data-oe-id="user_input.id" class="article certification-wrapper">
                    <div t-attf-class="certification #{'test-entry' if user_input.test_entry else ''}">
                        <h2>
                            <span t-field="user_input.survey_id.create_uid.company_id.display_name"/>
                        </h2>
                        <p>Certifies that</p>
                        <h2>
                            <span t-esc="user_input.partner_id.name or user_input.email"/>
                        </h2>
                        <!-- You should not print a certificate if failed :) -->
                        <p t-if="user_input.quizz_passed">Successfully achieved</p>
                        <p t-if="not user_input.quizz_passed">Successfully failed</p>
                        <h3>
                            <span t-field="user_input.survey_id.display_name"/>
                        </h3>
                        <p>
                            <strong t-if="user_input.quizz_passed">Date of Certification:</strong>
                            <strong t-if="not user_input.quizz_passed">Date of Failure:</strong>
                            <span t-field="user_input.create_date"/>
                        </p>
                    </div>
                </div>
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
    <record model="ir.ui.view" id="survey_form">
        <field name="name">Form view for survey</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <form string="Survey" class="o_survey_form">
                <field name="id" invisible="1"/>
                <header>
                    <button name="action_open" string="Start Survey" type="object" class="oe_highlight" attrs="{'invisible': ['|', ('state', '!=', 'draft'), ('id', '=', False)]}"/>
                    <button name="action_send_survey" string="Share" type="object" class="oe_highlight" states="open"/>
                    <button name="action_result_survey" string="See results" type="object" class="oe_highlight"
                      attrs="{'invisible': ['|', ('state', '=', 'draft'), ('answer_done_count', '&lt;=', 0)]}"/>
                    <button name="action_draft" string="Set to draft" type="object" states="closed"/>
                    <button name="action_test_survey" string="Test" type="object" attrs="{'invisible': ['|', ('state', '=', 'closed'), ('id', '=', False)]}"/>
                    <button name="action_print_survey" string="Print" type="object" attrs="{'invisible': [('id', '=', False)]}"/>
                    <button name="action_close" string="Close" type="object" states="open"/>
                    <field name="state" widget="statusbar"/>
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
                            attrs="{'invisible': [('certificate', '=', False)]}"
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
                    <div class="oe_title" style="width: 100%;">
                        <label for="title" class="oe_edit_only"/>
                        <h1><field name="title" placeholder="e.g. Satisfaction Survey"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="category" invisible="1"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Questions">
                            <field name="question_and_page_ids" nolabel="1" widget="question_page_one2many" mode="tree,kanban" context="{'default_survey_id': active_id, 'default_questions_selection': questions_selection}">
                                <tree decoration-bf="is_page" editable="bottom">
                                    <field name="sequence" widget="handle"/>
                                    <field name="title"/>
                                    <field name="question_type" />
                                    <field name="is_page" invisible="1"/>
                                    <field name="questions_selection" invisible="1"/>
                                    <field name="random_questions_count" attrs="{'column_invisible': [('parent.questions_selection', '=', 'all')], 'invisible': [('is_page', '=', False)]}" />
                                    <control>
                                        <create name="add_section_control" string="Add a section" context="{'default_is_page': True, 'default_questions_selection': 'all'}"/>
                                        <create name="add_question_control" string="Add a question"/>
                                    </control>
                                </tree>
                            </field>
                        </page>
                        <page string="Description">
                            <field name="description" nolabel="1"></field>
                        </page>
                        <page string="Options">
                            <group name="options">
                                <group string="Questions" name="questions">
                                    <field name="questions_layout" widget="radio" />
                                    <label for="is_time_limited" string="Time Limit"/>
                                    <div>
                                        <field name="is_time_limited" nolabel="1"/>
                                        <field name="time_limit" widget="float_time" attrs="{'invisible': [('is_time_limited', '=', False)]}" nolabel="1" class="oe_inline" /> <span attrs="{'invisible': [('is_time_limited', '=', False)]}"> minutes</span>
                                    </div>
                                    <field name="questions_selection" widget="radio" />
                                    <field name="users_can_go_back" string="Back Button" attrs="{'invisible': [('questions_layout', '=', 'one_page')]}"/>
                                </group>
                                <group string="Candidates" name="candidates">
                                    <field name="access_mode"/>
                                    <field name="users_login_required"/>
                                    <label for="is_attempts_limited" string="Attempts Limit" attrs="{'invisible': [('access_mode', '=', 'public'), ('users_login_required', '=', False)]}"/>
                                    <div attrs="{'invisible': [('access_mode', '=', 'public'), ('users_login_required', '=', False)]}">
                                        <field name="is_attempts_limited" nolabel="1"/>
                                        <field name="attempts_limit" attrs="{'invisible': [('is_attempts_limited', '=', False)]}" nolabel="1" class="oe_inline" /> <span attrs="{'invisible': [('is_attempts_limited', '=', False)]}"> attempts</span>
                                    </div>
                                </group>
                                <group string="Scoring" name="scoring">
                                    <field name="scoring_type" widget="radio" />
                                    <field name="passing_score" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}" />
                                    <field name="certificate" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}" />
                                    <field name="certification_mail_template_id" attrs="{'invisible': [('certificate', '=', False)]}" />
                                    <field name="certification_give_badge" attrs="{'invisible': ['|', ('certificate', '=', False), ('users_login_required', '=', False)]}" />
                                    <field name="certification_badge_id" attrs="{'invisible': ['|', ('certification_give_badge', '=', False), ('certification_badge_id', '!=', False)]}"
                                           domain="[('survey_id', '=', active_id), ('survey_id', '!=', False)]"
                                           context="{'default_name': title,
                                                   'default_description': 'Congratulation, you succeeded this certification',
                                                   'default_rule_auth': 'nobody',
                                                   'default_level': None,
                                                   'form_view_ref': 'survey.gamification_badge_form_view_simplified',
                                                   'default_website_published': True}"/>
                                    <field name="certification_badge_id_dummy" attrs="{'invisible': ['|', ('certification_give_badge', '=', False), ('certification_badge_id', '=', False)]}"
                                           options="{'no_create': True}"
                                           context="{'form_view_ref': 'survey.gamification_badge_form_view_simplified'}"/>
                                </group>
                            </group>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="activity_ids" widget="mail_activity"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
            </form>
        </field>
    </record>
    <record model="ir.ui.view" id= "survey_tree">
        <field name="name">Tree view for survey</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <tree string="Survey">
                <field name="active" invisible="1"/>
                <field name="certificate" invisible="1"/>
                <field name="title"/>
                <field name="state"/>
                <field name="answer_count"/>
                <field name="answer_done_count"/>
                <field name="success_count"/>
                <field name="success_ratio"/>
                <field name="answer_score_avg"/>
                <button name="certificate" type="button" disabled="disabled" icon="fa-trophy" title="Certificate" aria-label="Certificate" attrs="{'invisible': [('certificate', '=', False)]}"/>
                <!-- Tweak as icons aren't directly supported in xml -->
            </tree>
        </field>
    </record>
    <record model="ir.ui.view" id="survey_kanban">
        <field name="name">Kanban view for survey</field>
        <field name="model">survey.survey</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="state" />
                <field name="title" />
                <field name="answer_done_count" />
                <field name="certificate" />
                <field name="scoring_type" />
                <field name="color" />
                <field name="access_mode"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <field name="success_count"/>
                <field name="success_ratio"/>
                <templates>
                    <div t-name="kanban-box" 
                        t-attf-class="oe_kanban_color_#{kanban_getcolor(record.color.raw_value)} oe_kanban_card oe_kanban_global_click o_kanban_card_survey 
                            #{record.certificate.raw_value ? 'o_kanban_card_survey_successed' : ''}">
                        <div class="o_dropdown_kanban dropdown" t-if="widget.editable">

                            <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" data-display="static" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                <span class="fa fa-ellipsis-v"/>
                            </a>
                            <div class="dropdown-menu" role="menu">
                                <a role="menuitem" type="edit" class="dropdown-item">Edit Survey</a>
                                <a t-if="record.state.raw_value != 'closed'" role="menuitem" type="object" class="dropdown-item" name="action_send_survey">Share</a>
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
                                <div class="container o_kanban_card_content" t-if="record.answer_done_count.raw_value != 0 or record.state.raw_value != 'draft'">
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
                                                <span class="text-muted" t-if="!record.certificate.raw_value">Passed</span>
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
                <filter string="Certification" name="certificate" domain="[('certificate', '=', True)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <separator/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('activity_ids.date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('activity_ids.date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Upcoming Activities" name="activities_upcoming_all"
                    domain="[('activity_ids.date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Category" name="groupby_category" context="{'group_by': 'category'}"/>
                    <filter string="State" name="groupby_state" context="{'group_by': 'state'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_survey_form">
        <field name="name">Surveys</field>
        <field name="res_model">survey.survey</field>
        <field name="view_mode">kanban,tree,form,activity</field>
        <field name="context">{'search_default_groupby_state': 1}</field>
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
    <!-- Forbidden error messages-->
    <template id="403">
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

    <!-- "Thank you" message when the survey is completed -->
    <template id="sfinished" name="Survey Finished">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="container">
                    <t t-if="answer.test_entry" t-call="survey.back" />
                    <div class="jumbotron mt32">
                        <h1>Thank you!</h1>
                        <div t-field="survey.thank_you_message" class="oe_no_empty" />
                        <div class="row">
                            <div class="col">
                                <t t-if="survey.scoring_type != 'no_scoring'">
                                    <div>You scored <t t-esc="answer.quizz_score" />%</div>
                                    <t t-if="answer.quizz_passed">
                                        <div>Congratulations, you have passed the test!</div>

                                        <div t-if="survey.certificate" class="mt16 mb16">
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
                                <t t-call="survey.retake_survey_button"/>
                                <div t-if="survey.scoring_type != 'scoring_without_answers'">
                                    If you wish, you can <a t-att-href="'/survey/print/%s?answer_token=%s&amp;review=True' % (survey.access_token, answer.token)">review your answers</a>
                                </div>
                            </div>
                            <div class="col-6 text-center" t-if="survey.certification_give_badge and answer.quizz_passed">
                                <img t-att-src="'/web/image/gamification.badge/%s/image_128' % survey.certification_badge_id.id"/>
                                <div>You received the badge <span class="font-weight-bold" t-esc="survey.certification_badge_id.name"/>!</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="container js_surveyresult p-4" t-if="graph_data">
                    <div class="tab-content">
                        <div role="tabpanel" class="tab-pane active survey_graph" t-att-quizz-score="answer.quizz_score" t-att-id="'graph_question_%d' % answer.id" t-att-data-question_id="answer.id" data-graph_type="doughnut" t-att-graph-data="graph_data">
                            <canvas id="doughnut_chart"></canvas>
                            <span class="o_overall_performance"></span>
                        </div>
                    </div>
                </div>
                <div class="oe_structure"/>
            </div>
        </t>
    </template>

    <template id="retake_survey_button" name="Retake Survey Button">
        <div>
            <t t-if="not answer.quizz_passed">
                <t t-if="survey.is_attempts_limited">
                    <t t-set="attempts_left" t-value="survey._get_number_of_attempts_lefts(answer.partner_id, answer.email, answer.invite_token)" />
                    <t t-if="attempts_left > 0">
                        <p><span>Number of attemps left</span>: <span t-esc="attempts_left"></span></p>
                        <p><a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/retry/%s/%s' % (survey.access_token, answer.token)">
                        Retry</a></p>
                    </t>
                </t>
                <t t-else="">
                    <p><a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/retry/%s/%s' % (survey.access_token, answer.token)">
                        Retry</a></p>
                </t>
            </t>
        </div>
    </template>

    <!-- Message when the survey is not open  -->
    <template id="survey_expired" name="Survey Expired">
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

    <!-- Message when a login is required  -->
    <template id="auth_required" name="Login required for this survey">
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

    <!-- Message when the survey has no pages  -->
    <template id="survey_void" name="Survey has no pages">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="container">
                    <t t-if="answer.test_entry" t-call="survey.back" />
                    <div class="jumbotron mt32">
                        <h1><span t-field="survey.title"/> survey is void</h1>
                        <p>Please make sure you have at least one question in your survey. You also need at least one section if you chose the "Page per section" layout.<br />
                            <a t-att-href="'/web#view_type=form&amp;model=survey.survey&amp;id=%s&amp;action=survey.action_survey_form' % survey.id"
                                class="btn btn-secondary"
                                groups="survey.group_survey_manager">Edit in backend</a>
                        </p>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Back Button to redirect in form view of survey -->
    <template id="back" name="Back">
        <div groups="survey.group_survey_manager" t-ignore="true" class="alert alert-info alert-dismissible rounded-0 mt16 fade show d-print-none css_editable_mode_hidden">
            <div t-ignore="true" class="text-center">
                <a t-attf-href="/web#view_type=form&amp;model=survey.survey&amp;id=#{survey.id}&amp;action=survey.action_survey_form"><i class="fa fa-fw fa-arrow-right"/><span t-if="answer and answer.test_entry">This is a test survey. </span>Edit Survey.</a>
            </div>
            <button type="button" class="close" data-dismiss="alert" aria-label="Close"> &#215; </button>
        </div>
    </template>

    <!-- new layout for survey -->
    <template id="survey.layout" name="Survey Layout" inherit_id="web.frontend_layout" primary="True">
        <xpath expr="//head/t[@t-call-assets][last()]" position="after">
            <t t-call-assets="survey.survey_assets" lazy_load="True"/>
        </xpath>
        <xpath expr="//header" position="before">
            <t t-set="no_header" t-value="no_header or survey.certificate"/>
            <t t-set="no_footer" t-value="no_footer or survey.certificate"/>
        </xpath>
        <xpath expr="//header" position="after">
            <div id="wrap" class="oe_structure oe_empty"/>
        </xpath>
    </template>

    <!-- First page of a survey -->
    <template id="survey_init" name="Survey">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="oe_structure" id="oe_structure_survey_init_1"/>
                <div class="container">
                    <t t-if="answer.test_entry" t-call="survey.back" />
                    <div class='jumbotron mt32'>
                        <h1 t-field='survey.title' />
                        <div t-field='survey.description' class="oe_no_empty"/>
                        <div t-if="survey.is_time_limited">
                            <p>
                                <span>Time limit for this survey: </span>
                                <span class="font-weight-bold text-danger" t-field="survey.time_limit" t-options="{'widget': 'duration', 'unit': 'minute'}"></span>
                            </p>
                        </div>
                        <a role="button" class="btn btn-primary btn-lg" t-att-href="'/survey/fill/%s/%s' % (survey.access_token, answer.token)">
                            <t t-if="survey.certificate">
                                Start Certification
                            </t>
                            <t t-else="">
                                Start Survey
                            </t>
                        </a>
                    </div>
                </div>
                <div class="oe_structure" id="oe_structure_survey_init_2"/>
            </div>
        </t>
    </template>

    <!-- A survey -->
    <template id="survey" name="Survey">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="oe_structure" id="oe_structure_survey_survey_1"/>
                <div class="container">
                    <t t-if="answer.test_entry" t-call="survey.back" />
                    <t t-call="survey.survey_header" />
                    <t t-call="survey.page" />
                </div>
                <div class="oe_structure" id="oe_structure_survey_survey_2"/>
            </div>
        </t>
    </template>

    <!-- The survey header -->
    <template id="survey_header" name="Header">
        <div class="o_page_header o_survey_header">
            <div class="container m-0 p-0">
                <div class="row">
                    <div class="col-lg-8"><h1 t-esc="survey.title"></h1></div>
                    <div
                        t-if="survey.is_time_limited"
                        class="js_survey_timer col-lg-4"
                        t-att-data-timer="answer.start_datetime.isoformat()"
                        t-att-data-time_limit_minutes="survey.time_limit">
                        <h1 class="timer text-right">00:00</h1>
                    </div>
                </div>
            </div>
            <ol t-if="survey.questions_layout == 'page_per_section'" class="breadcrumb mt8 justify-content-end">
                <t t-set="can_go_back" t-value="survey.users_can_go_back" />
                <t t-foreach='survey.page_ids' t-as='breadcrumb_page'>
                    <li t-attf-class="breadcrumb-item #{'active' if breadcrumb_page == page else ''}">
                        <t t-if="page == breadcrumb_page">
                            <!-- Users can only go back and not forward -->
                            <!-- As soon as we reach the current page, set "can_go_back" to False -->
                            <t t-set="can_go_back" t-value="False" />
                        </t>
                        <t t-if="can_go_back">
                            <a t-att-href="'/survey/page/%s/%s/%s' % (survey.access_token, answer.token, breadcrumb_page.id)">
                                <span t-field='breadcrumb_page.title' />
                            </a>
                        </t>
                        <t t-else="">
                            <span t-field='breadcrumb_page.title' />
                        </t>
                    </li>
                </t>
            </ol>
        </div>
    </template>

    <!-- A page -->
    <template id="page" name="Page">
        <t t-if="survey.questions_layout == 'one_page'" t-set="page_or_question_id" t-value="None" />
        <t t-if="survey.questions_layout == 'page_per_section'" t-set="page_or_question_id" t-value="page.id" />
        <t t-if="survey.questions_layout == 'page_per_question'" t-set="page_or_question_id" t-value="question.id" />

        <form role="form" method="post" class="js_surveyform" t-att-name="survey.id"
                t-att-action="'/survey/fill/%s/%s' % (survey.access_token, answer.token)"
                t-att-data-prefill="'/survey/prefill/%s/%s?page_or_question_id=%s' % (survey.access_token, answer.token, page_or_question_id)"
                t-att-data-validate="'/survey/validate/%s/%s' % (survey.access_token, answer.token)"
                t-att-data-submit="'/survey/submit/%s/%s' % (survey.access_token, answer.token)">
            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
            <input type="hidden" name="token" t-att-value="answer.token" />

            <t t-if="survey.questions_layout == 'one_page'">
                <t t-foreach='survey.question_and_page_ids' t-as='question'>
                    <h2 t-if="question.is_page" t-field='question.title' class="o_survey_title" />
                    <t t-if="not question.is_page and question in answer.question_ids" t-call="survey.question" />
                </t>

                <div class="text-center mt16 mb256">
                    <button type="submit" class="btn btn-primary" name="button_submit" value="finish">Submit Survey</button>
                </div>
            </t>

            <t t-if="survey.questions_layout == 'page_per_section'">
                <h2 t-field='page.title' class="o_survey_title" />
                <div t-field='page.description' class="oe_no_empty"/>

                <input type="hidden" name="page_id" t-att-value="page.id" />
                <t t-foreach='page.question_ids' t-as='question'>
                    <t t-if="question in answer.question_ids" t-call="survey.question" />
                </t>

                <div class="text-center mt16 mb256">
                    <button t-if="survey.users_can_go_back and page != survey.page_ids[0]" type="submit" class="btn btn-secondary" name="button_submit" value="previous">Previous page</button>
                    <button t-if="not last" type="submit" class="btn btn-primary" name="button_submit" value="next">Next page</button>
                    <button t-if="last" type="submit" class="btn btn-primary" name="button_submit" value="finish">Submit Survey</button>
                </div>
            </t>

            <t t-if="survey.questions_layout == 'page_per_question'">
                <input type="hidden" name="question_id" t-att-value="question.id" />
                <t t-call="survey.question" />

                <div class="text-center mt16 mb256">
                    <button t-if="survey.users_can_go_back and question != answer.question_ids[0]" type="submit" class="btn btn-secondary" name="button_submit" value="previous">Previous question</button>
                    <button t-if="not last" type="submit" class="btn btn-primary" name="button_submit" value="next">Next question</button>
                    <button t-if="last" type="submit" class="btn btn-primary" name="button_submit" value="finish">Submit Survey</button>
                </div>
            </t>
        </form>

        <!-- Modal used to display error message, i.c.o. ajax error -->
        <div role="dialog" class="modal fade" id="AJAXErrorModal" >
            <div class="modal-dialog">
                <div class="modal-content">
                    <header class="modal-header">
                        <h4 class="modal-title">A problem has occured</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">&amp;times;</button>
                    </header>
                    <main class="modal-body"><p>Something went wrong while contacting survey server. <strong class="text-danger">Your answers have probably not been recorded.</strong> Try refreshing.</p></main>
                    <footer class="modal-footer"><button type="button" class="btn btn-primary" data-dismiss="modal">Close</button></footer>
                </div>
            </div>
        </div>
    </template>

    <template id="question" name="Question">
        <t t-set="prefix" t-value="'%s_%s' % (survey.id, question.id)" />
        <div class="js_question-wrapper" t-att-id="prefix">
            <h4>
                <span t-field='question.question' />
                <span t-if="question.constr_mandatory" class="text-danger">*</span>
            </h4>
            <div t-field='question.description' class="text-muted oe_no_empty"/>
            <t t-if="question.question_type == 'free_text'"><t t-call="survey.free_text"/></t>
            <t t-if="question.question_type == 'textbox'"><t t-call="survey.textbox"/></t>
            <t t-if="question.question_type == 'numerical_box'"><t t-call="survey.numerical_box"/></t>
            <t t-if="question.question_type == 'date'"><t t-call="survey.date"/></t>
            <t t-if="question.question_type == 'datetime'"><t t-call="survey.datetime"/></t>
            <t t-if="question.question_type == 'simple_choice'"><t t-call="survey.simple_choice"/></t>
            <t t-if="question.question_type == 'multiple_choice'"><t t-call="survey.multiple_choice"/></t>
            <t t-if="question.question_type == 'matrix'"><t t-call="survey.matrix"/></t>
            <div class="js_errzone alert alert-danger" style="display:none;" role="alert"></div>
        </div>
    </template>

    <!-- Question widgets -->
    <template id="free_text" name="Free text box">
        <textarea class="form-control" rows="3" t-att-name="prefix"></textarea>
    </template>

    <template id="textbox" name="Text box">
        <input type="text" class="form-control" t-att-name="prefix"/>
    </template>

    <template id="numerical_box" name="Numerical box">
        <input type="number" step="any" class="form-control" t-att-name="prefix"/>
    </template>

    <template id="date" name="Date box">
        <div class="form-group">
            <div class="input-group date" t-attf-id="datetimepicker_#{question.id}" data-target-input="nearest"
                    t-att-data-mindate="question.validation_min_date"
                    t-att-data-maxdate="question.validation_max_date">
                <input type="text" class="form-control datetimepicker-input" t-attf-data-target="#datetimepicker_#{question.id}" t-att-name="prefix"/>
                <div class="input-group-append" t-attf-data-target="#datetimepicker_#{question.id}" data-toggle="datetimepicker">
                    <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                </div>
            </div>
        </div>
    </template>

    <template id="datetime" name="DateTime box">
        <div class="form-group">
            <div class="input-group date" t-attf-id="datetimepicker_#{question.id}" data-target-input="nearest"
                    t-att-data-mindate="question.validation_min_datetime"
                    t-att-data-maxdate="question.validation_max_datetime"
                    t-att-data-questiontype="question.question_type">
                <input type="text" class="form-control datetimepicker-input" t-attf-data-target="#datetimepicker_#{question.id}" t-att-name="prefix"/>
                <div class="input-group-append" t-attf-data-target="#datetimepicker_#{question.id}" data-toggle="datetimepicker">
                    <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                </div>
            </div>
        </div>
    </template>

    <template id="simple_choice" name="Simple choice">
        <div t-if="question.display_mode == 'dropdown' and answer.token" class="js_drop row">
            <div class="col-lg-12">
                <select class="form-control" t-att-name="prefix">
                    <option disabled="1" selected="1" value="">Choose...</option>
                    <t t-foreach='question.labels_ids' t-as='label'>
                        <option t-att-value='label.id'><t t-esc='label.value'/></option>
                    </t>
                    <t t-if='question.comments_allowed and question.comment_count_as_answer'>
                        <option class="js_other_option" value="-1"><span t-esc="question.comments_message" /></option>
                    </t>
                </select>
            </div>
            <div t-if='question.comments_allowed and question.comment_count_as_answer' class="col-lg-6">
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')" data-oe-survey-othert="1"/>
            </div>
            <div t-if='question.comments_allowed and not question.comment_count_as_answer' class="col-lg-12 mt16">
                <span t-field="question.comments_message"/>
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')"/>
            </div>
        </div>
        <div t-if="question.display_mode == 'columns' or not answer.token" class="row js_radio">
            <div t-foreach='question.labels_ids' t-as='label' t-attf-class="col-lg-#{question.column_nb}">
                <label t-att-class="' bg-success ' if quizz_correction and label.answer_score > 0.0 else None">
                    <input type="radio" t-att-name="prefix" t-att-value='label.id' />
                    <span t-field='label.value'/>
                </label>
            </div>
            <div t-if='question.comments_allowed and question.comment_count_as_answer' class="js_comments col-lg-12" >
                <label>
                    <input type="radio" t-att-name="prefix" value="-1"/>
                    <span t-field="question.comments_message" />
                </label>
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')"/>
            </div>
            <div t-if='question.comments_allowed and not question.comment_count_as_answer' class="col-lg-12">
                <span t-field="question.comments_message"/>
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')" data-oe-survey-othert="1"/>
            </div>
        </div>
    </template>

    <template id="multiple_choice" name="Multiple choice">
        <div class="row">
            <div t-foreach='question.labels_ids' t-as='label' t-attf-class="col-lg-#{question.column_nb}">
                <label t-att-class="' bg-success ' if quizz_correction and label.answer_score > 0.0 else None">
                    <input type="checkbox" t-att-name="'%s_%s' % (prefix, label.id)" t-att-value='label.id' />
                    <span t-field='label.value'/>
                </label>
            </div>
            <div t-if='question.comments_allowed and question.comment_count_as_answer' class="js_ck_comments col-lg-12" >
                <label>
                    <input type="checkbox" t-att-name="'%s_%s' % (prefix, -1)" value="-1" />
                    <span t-field="question.comments_message" />
                </label>
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')"/>
            </div>
            <div t-if='question.comments_allowed and not question.comment_count_as_answer' class="col-lg-12">
                <span t-field="question.comments_message"/>
                <textarea type="text" class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')" data-oe-survey-othert="1"/>
            </div>
        </div>
    </template>

    <template id="matrix" name="Matrix">
        <table class="table table-hover">
            <thead>
                <tr>
                    <th> </th>
                    <th t-foreach="question.labels_ids" t-as="col_label"><span t-field="col_label.value" /></th>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="question.labels_ids_2" t-as="row_label">
                    <th><span t-field="row_label.value" /></th>
                    <td t-foreach="question.labels_ids" t-as="col_label">
                        <input t-if="question.matrix_subtype == 'simple'" type="radio" t-att-name="'%s_%s' % (prefix, row_label.id)" t-att-value='col_label.id' />
                        <input t-if="question.matrix_subtype == 'multiple'" type="checkbox" t-att-name="'%s_%s_%s' % (prefix, row_label.id, col_label.id)" t-att-value='col_label.id' />
                    </td>
                </tr>
            </tbody>
        </table>
        <div t-if='question.comments_allowed'>
            <span t-field="question.comments_message"/>
            <textarea class="form-control" rows="1" t-att-name="'%s_%s' % (prefix, 'comment')" />
        </div>
    </template>

    <!-- Printable view of a survey (all pages) -->
    <template id="survey_print" name="Survey">
        <t t-call="survey.layout">
            <div class="wrap">
                <div class="container">
                    <t t-if="answer.test_entry" t-call="survey.back" />
                    <div class='jumbotron mt32'>
                        <h1><span t-field='survey.title'/></h1>
                        <t t-if="survey.description"><div t-field='survey.description' class="oe_no_empty"/></t>
                        <t t-if="review" t-call="survey.retake_survey_button"/>
                    </div>
                    <div role="form" class="js_surveyform" t-att-name="'%s' % (survey.id)" t-att-data-prefill="'/survey/prefill/%s/%s' % (survey.access_token, answer.token)">
                        <t t-foreach='survey.question_and_page_ids' t-as='question'>
                            <hr t-if="question.is_page and question != survey.page_ids[0]" />
                            <div t-if="question.is_page" class="o_page_header">
                                <h1 t-field='question.title' />
                                <div t-if="question.description" t-field='question.description' class="oe_no_empty"/>
                            </div>
                            <t t-if="not question.is_page and not answer or (question in answer.question_ids)" >
                                <t t-set="prefix" t-value="'%s_%s' % (survey.id, question.id)" />
                                <div class="js_question-wrapper" t-att-id="prefix">
                                    <h2>
                                        <span t-field='question.question' />
                                        <span t-if="question.constr_mandatory" class="text-danger">*</span>
                                        <span t-if="quizz_correction" class="badge badge-pill" t-att-data-score-question="question.id"></span>
                                    </h2>
                                    <t t-if="question.description"><div class="text-muted oe_no_empty" t-field='question.description'/></t>
                                    <t t-if="question.question_type == 'free_text'"><t t-call="survey.free_text"/></t>
                                    <t t-if="question.question_type == 'textbox'"><t t-call="survey.textbox"/></t>
                                    <t t-if="question.question_type == 'numerical_box'"><t t-call="survey.numerical_box"/></t>
                                    <t t-if="question.question_type == 'date'"><t t-call="survey.date"/></t>
                                    <t t-if="question.question_type == 'datetime'"><t t-call="survey.datetime"/></t>
                                    <t t-if="question.question_type == 'simple_choice'"><t t-call="survey.simple_choice"/></t>
                                    <t t-if="question.question_type == 'multiple_choice'"><t t-call="survey.multiple_choice"/></t>
                                    <t t-if="question.question_type == 'matrix'"><t t-call="survey.matrix"/></t>
                                    <div class="js_errzone alert alert-danger" style="display:none;" role="alert"></div>
                                </div>
                            </t>
                        </t>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="result" name="Survey Result">
        <t t-call="survey.layout">
            <div class="oe_structure" id="oe_structure_survey_result_1"/>
            <div class="container js_surveyresult">
                <t t-call="survey.back" />
                <div class="jumbotron mt32">
                    <h1><span t-field="survey.title" />
                    <span style="font-size:1.5em;" t-attf-class="fa fa-bar-chart-o #{'fa-bar-chart-o' if survey.scoring_type == 'no_scoring' else 'fa-trophy' if survey.certificate else 'fa-question-circle-o'} float-right " role="img" aria-label="Chart" title="Chart"/></h1>
                    <div t-field="survey.description" class="oe_no_empty" />
                    <h2 t-if="not answers">
                        Sorry, no one answered this survey yet.
                    </h2>
                </div>
                <div t-if="answers" class="card d-print-none">
                    <div class="card-header"><span class="fa fa-filter"></span>  Filters <span t-if="filter_display_data" class="float-right text-primary clear_survey_filter"><i class="fa fa-times"></i> Clear All Filters</span></div>
                    <div class="card-body">
                        <span t-if="filter_finish == True">
                            <span class="badge badge-secondary only_left_radius filter-all">All surveys</span><span class="badge badge-primary only_right_radius filter-finished">Finished surveys</span>
                        </span>
                        <span t-if="filter_finish == False">
                            <span class="badge badge-primary only_left_radius filter-all">All surveys</span><span class="badge badge-secondary only_right_radius filter-finished">Finished surveys</span>
                        </span>
                        <span t-foreach="filter_display_data" t-as="filter_data">
                            <span class="badge badge-primary only_left_radius"><i class="fa fa-filter" role="img" aria-label="Filter" title="Filter"></i></span><span class="badge badge-primary no_radius" t-esc="filter_data['question_text']"></span><span class="badge badge-success only_right_radius" t-esc="' > '.join(filter_data['labels'])"></span>
                        </span>
                    </div>
                </div>
                <div t-if="survey.scoring_type in ['scoring_with_answers', 'scoring_without_answers']">
                    <h1 class="mt16">Results Overview</h1>
                    <div>Success rate: <mark class="font-weight-bold"><t t-esc="survey_dict['success_rate']"></t>%</mark></div>
                    <div id="scoring_results_chart" t-att-data-graph_data="survey_dict['scoring_graph_data']">
                        <!-- canvas element for drawing pie chart -->
                        <canvas/>
                    </div>
                    <hr/>
                </div>
                <div t-if="answers" t-foreach="survey_dict['page_ids']" t-as='page_ids'>
                    <t t-set="page" t-value="page_ids['page']"/>
                    <h1 class="mt16" t-field='page.title'></h1>
                    <div t-field="page.description" class="oe_no_empty" />
                    <hr/>
                    <div t-foreach="page_ids['question_ids']" t-as='question_ids' class="mt16">
                        <t t-set="input_summary" t-value="question_ids['input_summary']"/>
                        <t t-set="question" t-value="question_ids['question']"/>
                        <t t-set="graph_data" t-value="question_ids['graph_data']"/>
                        <t t-set="prepare_result" t-value="question_ids['prepare_result']"/>
                        <t t-set="question_with_correct_answers" t-value="survey.scoring_type in ['scoring_with_answers', 'scoring_without_answers'] and question.question_type in ['simple_choice', 'multiple_choice']" />
                        <h4>
                            <b>Question </b>
                            <span t-field='question.question'></span>
                            <t t-if="question.question_type == 'matrix'">
                                <small><span class="badge badge-secondary">Matrix: <span t-field='question.matrix_subtype'/></span></small>
                            </t>
                            <t t-if="question.question_type in ['simple_choice', 'multiple_choice']">
                                <small><span t-field='question.question_type' class="badge badge-secondary"></span></small>
                            </t>
                            <span class="float-right">
                                <span class="badge badge-success"><span t-esc="input_summary['answered']"></span> Answered</span>
                                <span class="badge badge-warning"><span t-esc="input_summary['skipped']"></span> Skipped</span>
                            </span>
                        </h4>
                        <t t-if="question_with_correct_answers">
                            <t t-set="correct_answers" t-value="question.get_correct_answer_ids()" />
                            <div t-if="correct_answers">
                                <t t-if="question.question_type == 'simple_choice'">
                                    <p><span>Correct answer</span>: <span class="font-weight-bold" t-esc="correct_answers[0].value"></span></p>
                                </t>
                                <t t-if="question.question_type == 'multiple_choice'">
                                    <span>Correct answers</span>:
                                    <ul>
                                        <li t-foreach="correct_answers" t-as="correct_answer">
                                            <t t-esc="correct_answer.value" />
                                        </li>
                                    </ul>
                                </t>
                            </div>
                        </t>
                        <t t-if="input_summary['answered'] != 0">
                            <t t-if="question.description">
                                <div class="text-muted oe_no_empty" t-field="question.description" />
                            </t>
                            <t t-if="question.question_type in ['textbox', 'free_text', 'date', 'datetime']">
                                <t t-call="survey.result_text"></t>
                            </t>
                            <t t-if="question.question_type in ['simple_choice', 'multiple_choice']">
                                <t t-call="survey.result_choice"></t>
                            </t>
                            <t t-if="question.question_type == 'matrix'">
                                <t t-call="survey.result_matrix"></t>
                            </t>
                            <t t-if="question.question_type == 'numerical_box'">
                                <t t-call="survey.result_number"></t>
                            </t>
                        </t>
                        <t t-if="input_summary['answered'] == 0">
                            <h2 style="padding-top:30px;padding-bottom:30px;text-align:center;" class="text-muted">Sorry, No one answered this question.</h2>
                        </t>
                    </div>
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_survey_result_2"/>
        </t>
    </template>

    <!-- Result for free_text,textbox and date -->
    <template id="result_text" name="Text Result">
        <table class="table table-hover table-sm" t-att-id="'table_question_%d' % question.id">
            <thead>
                <tr>
                    <th>#</th>
                    <th>User Responses</th>
                </tr>
            </thead>
            <tbody>
                <t t-set="text_result" t-value="prepare_result"/>
                <tr t-foreach="text_result" t-as="user_input">
                    <td><a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)"><t t-esc="user_input_index + 1"></t></a></td>
                    <t t-if="question.question_type == 'free_text'">
                        <td>
                            <a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)" t-field="user_input.value_free_text"></a><br/>
                        </td>
                    </t>
                    <t t-if="question.question_type == 'textbox'">
                        <td>
                            <a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)" t-field="user_input.value_text"></a><br/>
                        </td>
                    </t>
                    <t t-if="question.question_type == 'date'">
                        <td>
                            <a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)" t-field="user_input.value_date"></a><br/>
                        </td>
                    </t>
                    <t t-if="question.question_type == 'datetime'">
                        <td>
                            <a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)" t-field="user_input.value_datetime"></a><br/>
                        </td>
                    </t>
                </tr>
            </tbody>
        </table>
        <t t-call="survey.pagination" />
    </template>

    <!-- Result for comments -->
    <template id="result_comments" name="Text Result">
        <!-- a 'comments' variable must be set an must contain a list of browse records of user input lines -->
        <table class="table table-hover table-sm" t-att-id="'table_question_%d' % question.id">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Comment</th>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="comments" t-as="user_input">
                    <td><a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)"><t t-esc="user_input_index + 1"></t></a></td>
                        <td>
                            <span t-field="user_input.value_text"></span><br/>
                        </td>
                </tr>
            </tbody>
        </table>
    </template>

    <!-- Result for simple_choice and multiple_choice -->
    <template id="result_choice" name="Choice Result">
        <div>
            <!-- Tabs -->
            <ul class="nav nav-tabs d-print-none" role="tablist">
                <li class="nav-item" t-if="question.question_type != 'simple_choice'">
                    <a t-att-href="'#graph_question_%d' % question.id" t-att-aria-controls="'graph_question_%d' % question.id" class="nav-link active" data-toggle="tab" role="tab">
                        <i class="fa fa-bar-chart-o"></i> Graph
                    </a>
                </li>
                <li class="nav-item" t-if="question.question_type == 'simple_choice'">
                    <a t-att-href="'#graph_question_%d' % question.id" t-att-aria-controls="'graph_question_%d' % question.id" class="nav-link active" data-toggle="tab" role="tab">
                        <i class="fa fa-bar-chart-o"></i> Pie Chart
                    </a>
                </li>
                <li class="nav-item">
                    <a t-att-href="'#data_question_%d' % question.id" t-att-aria-controls="'data_question_%d' % question.id" class="nav-link" data-toggle="tab" role="tab">
                        <i class="fa fa-list-alt"></i> Data
                    </a>
                </li>
            </ul>
            <div class="tab-content">
                <div role="tabpanel" class="tab-pane active survey_graph" t-if="question.question_type != 'simple_choice'" t-att-id="'graph_question_%d' % question.id" t-att-data-question_id="question.id" data-graph_type="bar" t-att-graph-data="graph_data">
                    <!-- canvas element for drawing bar chart -->
                    <canvas/>
                </div>
                <div role="tabpanel" class="tab-pane active survey_graph" t-if="question.question_type == 'simple_choice'" t-att-id="'graph_question_%d' % question.id" t-att-data-question_id="question.id" data-graph_type="pie" t-att-graph-data="graph_data">
                    <!-- canvas element for drawing pie chart -->
                    <canvas/>
                </div>
                <div role="tabpanel" class="tab-pane" t-att-id="'data_question_%d' % question.id">
                    <table class="table table-hover table-sm">
                        <thead>
                            <tr>
                                <th>Answer Choices</th>
                                <th>User Responses</th>
                                <th t-if="question_with_correct_answers">Answer Score</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="prepare_result['answers']" t-as="user_input">
                                <td>
                                    <p t-esc="user_input['text']"></p>
                                </td>
                                <td class="survey_answer">
                                    <span t-esc="round(user_input['count']*100.0/(input_summary['answered'] or 1),2)"></span> %
                                    <span t-esc="user_input['count']" class="badge badge-primary">Vote</span>
                                    <i class="fa fa-filter text-primary invisible survey_filter" t-att-data-question_id="question.id" t-att-data-answer_id="user_input['answer_id']" role="img" aria-label="Filter question" title="Filter question"/>
                                </td>
                                <td t-if="question_with_correct_answers" t-esc="user_input['answer_score']">
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
            <!-- handle comments -->
            <div>
                <t t-set="comments" t-value="prepare_result['comments']" />
                <t t-if="comments">
                    <t t-call="survey.result_comments" />
                </t>
            </div>

        </div>
    </template>

    <!-- Result for matrix -->
    <template id="result_matrix" name="Matrix Result">
        <t t-set="matrix_result" t-value="prepare_result"/>
        <!-- Tabs -->
        <ul class="nav nav-tabs d-print-none" role="tablist">
            <li class="nav-item">
                <a t-att-href="'#graph_question_%d' % question.id" t-att-aria-controls="'graph_question_%d' % question.id" class="nav-link active" data-toggle="tab" role="tab">
                    <i class="fa fa-bar-chart"></i>
                    Graph
                </a>
            </li>
            <li class="nav-item">
                <a t-att-href="'#data_question_%d' % question.id" t-att-aria-controls="'data_question_%d' % question.id" class="nav-link" data-toggle="tab" role="tab">
                    <i class="fa fa-list-alt"></i>
                    Data
                </a>
            </li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" class="tab-pane active with-3d-shadow with-transitions survey_graph" t-att-id="'graph_question_%d' % question.id" t-att-data-question_id= "question.id" data-graph_type= "multi_bar" t-att-graph-data="graph_data">
                <!-- canvas element for drawing Multibar chart -->
                <canvas/>
            </div>
            <div role="tabpanel" class="tab-pane" t-att-id="'data_question_%d' % question.id">
                <table class="table table-hover table-sm text-right">
                    <thead>
                        <tr>
                            <th></th>
                            <th class="text-right" t-foreach="matrix_result['answers']" t-as="answer_id">
                                <span t-esc="matrix_result['answers'][answer_id]"></span>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="matrix_result['rows']" t-as="row_id">
                            <td>
                                <span t-esc="matrix_result['rows'][row_id]"></span>
                            </td>
                            <td class="survey_answer" t-foreach="matrix_result['answers']" t-as="answer_id">
                                <span t-esc="round(matrix_result['result'][(row_id,answer_id)]*100.0/(input_summary['answered'] or 1),2)"></span> %
                                <span class="badge badge-primary" t-esc="matrix_result['result'][(row_id,answer_id)]"></span><i class="fa fa-filter text-primary invisible survey_filter" t-att-data-question_id="question.id" t-att-data-row_id="row_id" t-att-data-answer_id="answer_id" role="img" aria-label="Survey filter" title="Survey filter"></i>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <!-- handle comments to matrix -->
            <div>
                <t t-set="comments" t-value="matrix_result['comments']" />
                <t t-if="comments">
                    <t t-call="survey.result_comments" />
                </t>
            </div>
        </div>
    </template>

    <!-- Result for Numeric Data -->
    <template id="result_number" name="Number Result">
        <t t-set="number_result" t-value="prepare_result"/>
        <t t-set="text_result" t-value="number_result['input_lines']" />
        <span class="float-right mt8">
            <span class="badge badge-secondary only_left_radius">Sum </span> <span class="badge badge-info only_right_radius" t-esc="number_result['sum']"></span>
            <span class="badge badge-secondary only_left_radius">Maximum </span> <span class="badge badge-success only_right_radius" t-esc="number_result['max']"></span>
            <span class="badge badge-secondary only_left_radius">Minimum </span> <span class="badge badge-danger only_right_radius" t-esc="number_result['min']"></span>
            <span class="badge badge-secondary only_left_radius">Average </span> <span class="badge badge-warning only_right_radius" t-esc="number_result['average']"></span>
        </span>
        <ul class="nav nav-tabs d-print-none" role="tablist">
            <li class="nav-item">
                <a t-att-href="'#most_common_%d' % question.id" t-att-aria-controls="'most_common_%d' % question.id" class="nav-link active" data-toggle="tab" role="tab">
                    <i class="fa fa-list-ol"></i>
                    Most Common <span t-esc="len(number_result['most_common'])"></span>
                </a>
            </li>
            <li class="nav-item">
                <a t-att-href="'#data_question_%d' % question.id" t-att-aria-controls="'data_question_%d' % question.id" class="nav-link" data-toggle="tab" role="tab">
                    <i class="fa fa-list-alt"></i>
                    All Data
                </a>
            </li>
        </ul>
        <div class="tab-content">
            <div role="tabpanel" class="tab-pane active with-3d-shadow with-transitions" t-att-id="'most_common_%d' % question.id">
                <table class="table table-hover table-sm">
                     <thead>
                         <tr>
                             <th>User Responses</th>
                             <th>Occurence</th>
                         </tr>
                     </thead>
                     <tbody>
                         <tr t-foreach="number_result['most_common']" t-as="row">
                             <td>
                                 <span t-esc="row[0]"></span>
                             </td>
                             <td>
                                 <span t-esc="row[1]"></span>
                             </td>
                         </tr>
                     </tbody>
                </table>
            </div>
            <div role="tabpanel" class="tab-pane" t-att-id="'data_question_%d' % question.id">
                <table class="table table-hover table-sm" t-att-id="'table_question_%d' % question.id">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>User Responses</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr class="d-none" t-foreach="number_result['input_lines']" t-as="user_input">
                            <td><a t-att-href="'/survey/print/%s?answer_token=%s' % (user_input.survey_id.access_token, user_input.user_input_id.token)"><t t-esc="user_input_index + 1"></t></a></td>
                            <td><span t-field="user_input.value_number"></span><br/></td>
                        </tr>
                    </tbody>
                </table>
               <t t-call="survey.pagination"/>
            </div>
        </div>
    </template>
    <!-- Pagination Element -->
    <template id="pagination" name="Survey Result">
        <t t-set="record_limit" t-value="10"/><!-- Change This record_limit to change number of record  per page-->
        <ul t-att-id="'pagination_%d' % question.id" class="pagination" t-att-data-question_id="question.id" t-att-data-record_limit="record_limit">
            <t t-if="len(text_result) > record_limit">
                <li t-foreach="page_range(len(text_result), record_limit)" t-as="num" class="page-item">
                    <a href="#" class="page-link" t-esc="num"></a>
                </li>
            </t>
        </ul>
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
                <filter string="Partially Completed" name="partially_completed" domain="[('state', '=', 'skip')]"/>
                <filter string="New" name="new" domain="[('state', '=', 'new')]"/>
                <separator/>
                <filter string="Invite" name="invite" domain="[('input_type', '=', 'link')]"/>
                <separator/>
                <filter string="Quizz passed" name="quizz_passed" domain="[('quizz_passed','=', True)]"/>
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
                            <field name="input_type"/>
                            <field name="is_attempts_limited" invisible="1"/>
                            <label for="attempt_number" string="Attempt n°" attrs="{'invisible': ['|', ('is_attempts_limited', '=', False), '|', ('test_entry', '=', True), ('state', '!=', 'done')]}"/>
                            <div attrs="{'invisible': ['|', ('is_attempts_limited', '=', False), '|', ('test_entry', '=', True), ('state', '!=', 'done')]}">
                                <field name="attempt_number" nolabel="1"/>
                                 / 
                                <field name="attempts_limit" nolabel="1" />
                            </div>
                            <field name="token" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="deadline"/>
                            <field name="partner_id"/>
                            <field name="email" widget="email"/>
                            <field name="test_entry" groups="base.group_no_one"/>
                            <field name="scoring_type" invisible="1"/>
                            <field name="quizz_score" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}"/>
                            <field name="quizz_passed" attrs="{'invisible': [('scoring_type', '=', 'no_scoring')]}"/>
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
                <field name="input_type" groups="base.group_no_one"/>
                <field name="attempt_number" />
                <field name="state"/>
                <field name="test_entry" invisible="True"/>
                <field name="quizz_passed"/>
                <field name="quizz_score"/>
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
                <field name="input_type"/>
                <field name="state"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title"><t t-esc="record.survey_id.value"/></strong>
                                </div>
                                <span class="badge badge-pill"><t t-esc="record.input_type.value"/></span>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left">
                                    <field name="create_date"/>
                                </div>
                                <div class="oe_kanban_bottom_right mr4">
                                    <field name="state" widget="label_selection" options="{'classes': {'new': 'default', 'done': 'success', 'skip':'warning'}}"/>
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
    <record model="ir.ui.view" id="survey_user_input_line_form">
        <field name="name">survey_user_input_line_form</field>
        <field name="model">survey.user_input_line</field>
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
                        <field name="value_text" colspan='2' attrs="{'invisible': [('answer_type','!=','text')]}"/>
                        <field name="value_number" colspan='2' attrs="{'invisible': [('answer_type','!=','number')]}"/>
                        <field name="value_date" colspan='2' attrs="{'invisible': [('answer_type','!=','date')]}"/>
                        <field name="value_datetime" colspan='2' attrs="{'invisible': [('answer_type','!=','datetime')]}"/>
                        <field name="value_free_text" colspan='2' attrs="{'invisible': [('answer_type','!=','free_text')]}"/>
                        <field name="value_suggested_row" colspan='2' />
                        <field name="value_suggested" colspan='2' attrs="{'invisible': [('answer_type','!=','suggestion')]}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record model="ir.ui.view" id="survey_response_line_tree">
        <field name="name">survey_response_line_tree</field>
        <field name="model">survey.user_input_line</field>
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
    <record id="survey_response_line_search" model="ir.ui.view">
        <field name="name">survey_response_line_search</field>
        <field name="model">survey.user_input_line</field>
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

    <record model="ir.actions.act_window" id="action_survey_user_input_line">
        <field name="name">Detailed Answers</field>
        <field name="res_model">survey.user_input_line</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="survey_response_line_search"/>
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
        action="action_survey_user_input_line"
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

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

emails_split = re.compile(r"[;,\n\r]+")


class SurveyInvite(models.TransientModel):
    _name = 'survey.invite'
    _description = 'Survey Invitation Wizard'

    @api.model
    def _get_default_from(self):
        if self.env.user.email:
            return tools.formataddr((self.env.user.name, self.env.user.email))
        raise UserError(_("Unable to post message, please configure the sender's email address."))

    @api.model
    def _get_default_author(self):
        return self.env.user.partner_id

    # composer content
    subject = fields.Char('Subject')
    body = fields.Html('Contents', default='', sanitize_style=True)
    attachment_ids = fields.Many2many(
        'ir.attachment', 'survey_mail_compose_message_ir_attachments_rel', 'wizard_id', 'attachment_id',
        string='Attachments')
    template_id = fields.Many2one(
        'mail.template', 'Use template', index=True,
        domain="[('model', '=', 'survey.user_input')]")
    # origin
    email_from = fields.Char('From', default=_get_default_from, help="Email address of the sender.")
    author_id = fields.Many2one(
        'res.partner', 'Author', index=True,
        ondelete='set null', default=_get_default_author,
        help="Author of the message.")
    # recipients
    partner_ids = fields.Many2many(
        'res.partner', 'survey_invite_partner_ids', 'invite_id', 'partner_id', string='Recipients')
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
    survey_url = fields.Char(related="survey_id.public_url", readonly=True)
    survey_access_mode = fields.Selection(related="survey_id.access_mode", readonly=True)
    survey_users_login_required = fields.Boolean(related="survey_id.users_login_required", readonly=True)
    deadline = fields.Datetime(string="Answer deadline")

    @api.depends('partner_ids', 'survey_id')
    def _compute_existing_partner_ids(self):
        existing_answers = self.survey_id.user_input_ids
        self.existing_partner_ids = existing_answers.mapped('partner_id') & self.partner_ids

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

    @api.onchange('survey_users_login_required')
    def _onchange_survey_users_login_required(self):
        if self.survey_users_login_required and not self.survey_id.users_can_signup:
            return {'domain': {
                    'partner_ids': [('user_ids', '!=', False)]
                    }}
        return {'domain': {
                'partner_ids': []
                }}

    @api.onchange('partner_ids')
    def _onchange_partner_ids(self):
        if self.survey_users_login_required and self.partner_ids:
            if not self.survey_id.users_can_signup:
                invalid_partners = self.env['res.partner'].search([
                    ('user_ids', '=', False),
                    ('id', 'in', self.partner_ids.ids)
                ])
                if invalid_partners:
                    raise UserError(
                        _('The following recipients have no user account: %s. You should create user accounts for them or allow external signup in configuration.' %
                            (','.join(invalid_partners.mapped('name')))))

    @api.onchange('template_id')
    def _onchange_template_id(self):
        """ UPDATE ME """
        if self.template_id:
            self.subject = self.template_id.subject
            self.body = self.template_id.body_html

    @api.model
    def create(self, values):
        if values.get('template_id') and not (values.get('body') or values.get('subject')):
            template = self.env['mail.template'].browse(values['template_id'])
            if not values.get('subject'):
                values['subject'] = template.subject
            if not values.get('body'):
                values['body'] = template.body_html
        return super(SurveyInvite, self).create(values)

    #------------------------------------------------------
    # Wizard validation and send
    #------------------------------------------------------

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
            'input_type': 'link',
            'deadline': self.deadline,
        }

    def _send_mail(self, answer):
        """ Create mail specific for recipient containing notably its access token """
        subject = self.env['mail.template'].with_context(safe=True)._render_template(self.subject, 'survey.user_input', answer.id, post_process=True)
        body = self.env['mail.template']._render_template(self.body, 'survey.user_input', answer.id, post_process=True)
        # post the message
        mail_values = {
            'email_from': self.email_from,
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
                body = template.render(template_ctx, engine='ir.qweb', minimal_qcontext=True)
                mail_values['body_html'] = self.env['mail.thread']._replace_local_links(body)

        return self.env['mail.mail'].sudo().create(mail_values)

    def action_invite(self):
        """ Process the wizard content and proceed with sending the related
            email(s), rendering any template patterns on the fly if needed """
        self.ensure_one()
        Partner = self.env['res.partner']

        # compute partners and emails, try to find partners for given emails
        valid_partners = self.partner_ids
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
                <form string="Compose Email">
                    <group col="1">
                        <group col="2">
                            <field name="survey_access_mode" invisible="1"/>
                            <field name="survey_users_login_required" invisible="1"/>
                            <field name="survey_id" readonly="context.get('default_survey_id')"/>
                            <field name="existing_mode" widget="radio" invisible="1" />
                            <field name="survey_url" label="Public share URL" readonly="1" widget="CopyClipboardChar"
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
                        <field name="body" options="{'style-inline': true}"/>
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
                        <button string="Send" name="action_invite" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
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

