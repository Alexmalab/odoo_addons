# Odoo Module: rating

Category: Productivity

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
    'name': 'Customer Rating',
    'version': '1.1',
    'category': 'Productivity',
    'description': """
This module allows a customer to give rating.
""",
    'depends': [
        'mail',
    ],
    'data': [
        'views/rating_rating_views.xml',
        'views/rating_templates.xml',
        'views/mail_message_views.xml',
        'security/ir.model.access.csv'
    ],
    'installable': True,
    'assets': {
        'mail.assets_messaging': [
            'rating/static/src/models/*.js',
        ],
        'web.assets_backend': [
            'rating/static/src/scss/rating_rating_views.scss',
            'rating/static/src/components/*/*.scss',
            'rating/static/src/components/*/*.xml',
        ],
        'web.assets_frontend': [
            'rating/static/src/scss/rating_templates.scss',
        ],
        'web.tests_assets': [
            'rating/static/tests/helpers/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import werkzeug

from odoo import http
from odoo.http import request
from odoo.tools.translate import _
from odoo.tools.misc import get_lang

_logger = logging.getLogger(__name__)

MAPPED_RATES = {
    1: 1,
    5: 3,
    10: 5,
}

class Rating(http.Controller):

    @http.route('/rate/<string:token>/<int:rate>', type='http', auth="public", website=True)
    def action_open_rating(self, token, rate, **kwargs):
        if rate not in (1, 3, 5):
            raise ValueError(_("Incorrect rating: should be 1, 3 or 5 (received %d)"), rate)

        # This route used to allow sending a rating with a GET, the
        # feature proved incompatible with various email provider URL crawlers and
        # has been removed.
        rating, _record_sudo = self._get_rating_and_record(token)

        lang = rating.partner_id.lang or get_lang(request.env).code
        return request.env['ir.ui.view'].with_context(lang=lang)._render_template('rating.rating_external_page_submit', {
            'rating': rating,
            'token': token,
            'rate_names': {
                5: _("Satisfied"),
                3: _("Okay"),
                1: _("Dissatisfied"),
            },
            'rate': rate,
        })

    @http.route(['/rate/<string:token>/submit_feedback'], type="http", auth="public", methods=['post', 'get'], website=True)
    def action_submit_rating(self, token, rate=0, **kwargs):

        rating, record_sudo = self._get_rating_and_record(token)
        if request.httprequest.method == "POST":
            rate = int(rate)
            if rate not in (1, 3, 5):
                raise ValueError(_("Incorrect rating: should be 1, 3 or 5 (received %d)"), rate)
            record_sudo.rating_apply(
                rate,
                rating=rating,
                feedback=kwargs.get('feedback'),
                subtype_xmlid=None,  # force default subtype choice
            )

        lang = rating.partner_id.lang or get_lang(request.env).code
        return request.env['ir.ui.view'].with_context(lang=lang)._render_template('rating.rating_external_page_view', {
            'web_base_url': rating.get_base_url(),
            'rating': rating,
        })

    def _get_rating_and_record(self, token):
        rating_sudo = request.env['rating.rating'].sudo().search([('access_token', '=', token)])
        if not rating_sudo:
            raise werkzeug.exceptions.NotFound()

        record_sudo = request.env[rating_sudo.res_model].sudo().browse(rating_sudo.res_id)
        if not record_sudo.exists():
            raise werkzeug.exceptions.NotFound()
        return rating_sudo, record_sudo

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailMessage(models.Model):
    _inherit = 'mail.message'

    rating_ids = fields.One2many('rating.rating', 'message_id', groups='base.group_user', string='Related ratings')
    rating_value = fields.Float(
        'Rating Value', compute='_compute_rating_value', compute_sudo=True,
        store=False, search='_search_rating_value')

    @api.depends('rating_ids', 'rating_ids.rating')
    def _compute_rating_value(self):
        ratings = self.env['rating.rating'].search([('message_id', 'in', self.ids), ('consumed', '=', True)], order='create_date DESC')
        mapping = dict((r.message_id.id, r.rating) for r in ratings)
        for message in self:
            message.rating_value = mapping.get(message.id, 0.0)

    def _search_rating_value(self, operator, operand):
        ratings = self.env['rating.rating'].sudo().search([
            ('rating', operator, operand),
            ('message_id', '!=', False)
        ])
        return [('id', 'in', ratings.mapped('message_id').ids)]

    def message_format(self, format_reply=True):
        message_values = super().message_format(format_reply=format_reply)
        rating_mixin_messages = self.filtered(lambda message:
            message.model
            and message.res_id
            and issubclass(self.pool[message.model], self.pool['rating.mixin'])
        )
        if rating_mixin_messages:
            ratings = self.env['rating.rating'].sudo().search([('message_id', 'in', rating_mixin_messages.ids), ('consumed', '=', True)])
            rating_by_message_id = dict((r.message_id.id, r) for r in ratings)
            for vals in message_values:
                if vals['id'] in rating_by_message_id:
                    rating = rating_by_message_id[vals['id']]
                    vals['rating'] = {
                        'id': rating.id,
                        'ratingImageUrl': rating.rating_image_url,
                        'ratingText': rating.rating_text,
                    }
        return message_values

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        rating_id = kwargs.pop('rating_id', False)
        rating_value = kwargs.pop('rating_value', False)
        rating_feedback = kwargs.pop('rating_feedback', False)
        message = super(MailThread, self).message_post(**kwargs)

        # create rating.rating record linked to given rating_value. Using sudo as portal users may have
        # rights to create messages and therefore ratings (security should be checked beforehand)
        if rating_value:
            self.env['rating.rating'].sudo().create({
                'rating': float(rating_value) if rating_value is not None else False,
                'feedback': rating_feedback,
                'res_model_id': self.env['ir.model']._get_id(self._name),
                'res_id': self.id,
                'message_id': message.id,
                'consumed': True,
                'partner_id': self.env.user.partner_id.id,
            })
        elif rating_id:
            self.env['rating.rating'].browse(rating_id).write({'message_id': message.id})

        return message

    def _message_create(self, values_list):
        """ Force usage of rating-specific methods and API allowing to delegate
        computation to records. Keep methods optimized and skip rating_ids
        support to simplify MailThrad main API. """
        if not isinstance(values_list, (list)):
            values_list = [values_list]
        if any(values.get('rating_ids') for values in values_list):
            raise ValueError(_("Posting a rating should be done using message post API."))
        return super()._message_create(values_list)

```

## File: models\rating.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import uuid

from odoo import api, fields, models
from odoo.addons.rating.models import rating_data
from odoo.modules.module import get_resource_path


class Rating(models.Model):
    _name = "rating.rating"
    _description = "Rating"
    _order = 'write_date desc, id desc'
    _rec_name = 'res_name'

    @api.model
    def _default_access_token(self):
        return uuid.uuid4().hex

    @api.model
    def _selection_target_model(self):
        return [(model.model, model.name) for model in self.env['ir.model'].sudo().search([])]

    create_date = fields.Datetime(string="Submitted on")
    res_name = fields.Char(string='Resource name', compute='_compute_res_name', store=True)
    res_model_id = fields.Many2one('ir.model', 'Related Document Model', index=True, ondelete='cascade')
    res_model = fields.Char(string='Document Model', related='res_model_id.model', store=True, index=True, readonly=True)
    res_id = fields.Many2oneReference(string='Document', model_field='res_model', required=True, index=True)
    resource_ref = fields.Reference(
        string='Resource Ref', selection='_selection_target_model',
        compute='_compute_resource_ref', readonly=True)
    parent_res_name = fields.Char('Parent Document Name', compute='_compute_parent_res_name', store=True)
    parent_res_model_id = fields.Many2one('ir.model', 'Parent Related Document Model', index=True, ondelete='cascade')
    parent_res_model = fields.Char('Parent Document Model', store=True, related='parent_res_model_id.model', index=True, readonly=False)
    parent_res_id = fields.Integer('Parent Document', index=True)
    parent_ref = fields.Reference(
        string='Parent Ref', selection='_selection_target_model',
        compute='_compute_parent_ref', readonly=True)
    rated_partner_id = fields.Many2one('res.partner', string="Rated Operator")
    rated_partner_name = fields.Char(related="rated_partner_id.name")
    partner_id = fields.Many2one('res.partner', string='Customer')
    rating = fields.Float(string="Rating Value", group_operator="avg", default=0)
    rating_image = fields.Binary('Image', compute='_compute_rating_image')
    rating_image_url = fields.Char('Image URL', compute='_compute_rating_image')
    rating_text = fields.Selection(rating_data.RATING_TEXT, string='Rating', store=True, compute='_compute_rating_text', readonly=True)
    feedback = fields.Text('Comment')
    message_id = fields.Many2one(
        'mail.message', string="Message",
        index=True, ondelete='cascade')
    is_internal = fields.Boolean('Visible Internally Only', readonly=False, related='message_id.is_internal', store=True)
    access_token = fields.Char('Security Token', default=_default_access_token)
    consumed = fields.Boolean(string="Filled Rating")

    _sql_constraints = [
        ('rating_range', 'check(rating >= 0 and rating <= 5)', 'Rating should be between 0 and 5'),
    ]

    @api.depends('res_model', 'res_id')
    def _compute_res_name(self):
        for rating in self:
            name = self.env[rating.res_model].sudo().browse(rating.res_id).name_get()
            rating.res_name = name and name[0][1] or ('%s/%s') % (rating.res_model, rating.res_id)

    @api.depends('res_model', 'res_id')
    def _compute_resource_ref(self):
        for rating in self:
            if rating.res_model and rating.res_model in self.env:
                rating.resource_ref = '%s,%s' % (rating.res_model, rating.res_id or 0)
            else:
                rating.resource_ref = None

    @api.depends('parent_res_model', 'parent_res_id')
    def _compute_parent_ref(self):
        for rating in self:
            if rating.parent_res_model and rating.parent_res_model in self.env:
                rating.parent_ref = '%s,%s' % (rating.parent_res_model, rating.parent_res_id or 0)
            else:
                rating.parent_ref = None

    @api.depends('parent_res_model', 'parent_res_id')
    def _compute_parent_res_name(self):
        for rating in self:
            name = False
            if rating.parent_res_model and rating.parent_res_id:
                name = self.env[rating.parent_res_model].sudo().browse(rating.parent_res_id).name_get()
                name = name and name[0][1] or ('%s/%s') % (rating.parent_res_model, rating.parent_res_id)
            rating.parent_res_name = name

    def _get_rating_image_filename(self):
        self.ensure_one()
        return 'rating_%s.png' % rating_data._rating_to_threshold(self.rating)

    @api.depends('rating')
    def _compute_rating_image(self):
        self.rating_image_url = False
        self.rating_image = False
        for rating in self:
            try:
                image_path = get_resource_path('rating', 'static/src/img', rating._get_rating_image_filename())
                rating.rating_image_url = '/rating/static/src/img/%s' % rating._get_rating_image_filename()
                rating.rating_image = base64.b64encode(open(image_path, 'rb').read()) if image_path else False
            except (IOError, OSError):
                pass

    @api.depends('rating')
    def _compute_rating_text(self):
        for rating in self:
            rating.rating_text = rating_data._rating_to_text(rating.rating)

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if values.get('res_model_id') and values.get('res_id'):
                values.update(self._find_parent_data(values))
        return super().create(vals_list)

    def write(self, values):
        if values.get('res_model_id') and values.get('res_id'):
            values.update(self._find_parent_data(values))
        return super(Rating, self).write(values)

    def unlink(self):
        # OPW-2181568: Delete the chatter message too
        self.env['mail.message'].search([('rating_ids', 'in', self.ids)]).unlink()
        return super(Rating, self).unlink()

    def _find_parent_data(self, values):
        """ Determine the parent res_model/res_id, based on the values to create or write """
        current_model_name = self.env['ir.model'].sudo().browse(values['res_model_id']).model
        current_record = self.env[current_model_name].browse(values['res_id'])
        data = {
            'parent_res_model_id': False,
            'parent_res_id': False,
        }
        if hasattr(current_record, '_rating_get_parent_field_name'):
            current_record_parent = current_record._rating_get_parent_field_name()
            if current_record_parent:
                parent_res_model = getattr(current_record, current_record_parent)
                data['parent_res_model_id'] = self.env['ir.model']._get(parent_res_model._name).id
                data['parent_res_id'] = parent_res_model.id
        return data

    def reset(self):
        for record in self:
            record.write({
                'rating': 0,
                'access_token': record._default_access_token(),
                'feedback': False,
                'consumed': False,
            })

    def action_open_rated_object(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'res_model': self.res_model,
            'res_id': self.res_id,
            'views': [[False, 'form']]
        }

```

## File: models\rating_data.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import operator

from odoo.tools.float_utils import float_compare

RATING_AVG_TOP = 3.66
RATING_AVG_OK = 2.33
RATING_AVG_MIN = 1

RATING_LIMIT_SATISFIED = 4
RATING_LIMIT_OK = 3
RATING_LIMIT_MIN = 1
RATING_TEXT = [
    ('top', 'Satisfied'),
    ('ok', 'Okay'),
    ('ko', 'Dissatisfied'),
    ('none', 'No Rating yet'),
]

OPERATOR_MAPPING = {
    '=': operator.eq,
    '!=': operator.ne,
    '<': operator.lt,
    '<=': operator.le,
    '>': operator.gt,
    '>=': operator.ge,
}

def _rating_avg_to_text(rating_avg):
    if float_compare(rating_avg, RATING_AVG_TOP, 2) >= 0:
        return 'top'
    if float_compare(rating_avg, RATING_AVG_OK, 2) >= 0:
        return 'ok'
    if float_compare(rating_avg, RATING_AVG_MIN, 2) >= 0:
        return 'ko'
    return 'none'

def _rating_assert_value(rating_value):
    assert 0 <= rating_value <= 5

def _rating_to_grade(rating_value):
    """ From a rating value give a text-based mean value. """
    _rating_assert_value(rating_value)
    if rating_value >= RATING_LIMIT_SATISFIED:
        return 'great'
    if rating_value >= RATING_LIMIT_OK:
        return 'okay'
    return 'bad'

def _rating_to_text(rating_value):
    """ From a rating value give a text-based mean value. """
    _rating_assert_value(rating_value)
    if rating_value >= RATING_LIMIT_SATISFIED:
        return 'top'
    if rating_value >= RATING_LIMIT_OK:
        return 'ok'
    if rating_value >= RATING_LIMIT_MIN:
        return 'ko'
    return 'none'

def _rating_to_threshold(rating_value):
    """ From a rating value, return the thresholds in form of 0-1-3-5 used
    notably for images. """
    _rating_assert_value(rating_value)
    if rating_value >= RATING_LIMIT_SATISFIED:
        return 5
    if rating_value >= RATING_LIMIT_OK:
        return 3
    if rating_value >= RATING_LIMIT_MIN:
        return 1
    return 0

```

## File: models\rating_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import markupsafe

from odoo import api, fields, models, tools
from odoo.addons.rating.models import rating_data
from odoo.osv import expression
from odoo.tools.float_utils import float_compare, float_round


class RatingMixin(models.AbstractModel):
    _name = 'rating.mixin'
    _description = "Rating Mixin"

    rating_ids = fields.One2many('rating.rating', 'res_id', string='Rating', groups='base.group_user', domain=lambda self: [('res_model', '=', self._name)], auto_join=True)
    rating_last_value = fields.Float('Rating Last Value', groups='base.group_user', compute='_compute_rating_last_value', compute_sudo=True, store=True)
    rating_last_feedback = fields.Text('Rating Last Feedback', groups='base.group_user', related='rating_ids.feedback')
    rating_last_image = fields.Binary('Rating Last Image', groups='base.group_user', related='rating_ids.rating_image')
    rating_count = fields.Integer('Rating count', compute="_compute_rating_stats", compute_sudo=True)
    rating_avg = fields.Float("Average Rating", groups='base.group_user',
        compute='_compute_rating_stats', compute_sudo=True, search='_search_rating_avg')
    rating_avg_text = fields.Selection(rating_data.RATING_TEXT, groups='base.group_user',
        compute='_compute_rating_avg_text', compute_sudo=True)
    rating_percentage_satisfaction = fields.Float("Rating Satisfaction", compute='_compute_rating_satisfaction', compute_sudo=True)
    rating_last_text = fields.Selection(string="Rating Text", groups='base.group_user', related="rating_ids.rating_text")

    @api.depends('rating_ids', 'rating_ids.rating', 'rating_ids.consumed')
    def _compute_rating_last_value(self):
        # Pure SQL instead of calling read_group to allow ordering array_agg
        self.flush_model(['rating_ids'])
        self.env['rating.rating'].flush_model(['consumed', 'rating'])
        if not self.ids:
            self.rating_last_value = 0
            return
        self.env.cr.execute("""
            SELECT
                array_agg(rating ORDER BY write_date DESC, id DESC) AS "ratings",
                res_id as res_id
            FROM "rating_rating"
            WHERE
                res_model = %s
            AND res_id in %s
            AND consumed = true
            GROUP BY res_id""", [self._name, tuple(self.ids)])
        read_group_raw = self.env.cr.dictfetchall()
        rating_by_res_id = {e['res_id']: e['ratings'][0] for e in read_group_raw}
        for record in self:
            record.rating_last_value = rating_by_res_id.get(record.id, 0)

    @api.depends('rating_ids.res_id', 'rating_ids.rating')
    def _compute_rating_stats(self):
        """ Compute avg and count in one query, as thoses fields will be used together most of the time. """
        domain = expression.AND([self._rating_domain(), [('rating', '>=', rating_data.RATING_LIMIT_MIN)]])
        read_group_res = self.env['rating.rating'].read_group(domain, ['rating:avg'], groupby=['res_id'], lazy=False)  # force average on rating column
        mapping = {item['res_id']: {'rating_count': item['__count'], 'rating_avg': item['rating']} for item in read_group_res}
        for record in self:
            record.rating_count = mapping.get(record.id, {}).get('rating_count', 0)
            record.rating_avg = mapping.get(record.id, {}).get('rating_avg', 0)

    def _search_rating_avg(self, operator, value):
        if operator not in rating_data.OPERATOR_MAPPING:
            raise NotImplementedError('This operator %s is not supported in this search method.' % operator)
        rating_read_group = self.env['rating.rating'].sudo().read_group(
            [('res_model', '=', self._name), ('consumed', '=', True), ('rating', '>=', rating_data.RATING_LIMIT_MIN)],
            ['res_id', 'rating_avg:avg(rating)'], ['res_id'])
        res_ids = [
            res['res_id']
            for res in rating_read_group
            if rating_data.OPERATOR_MAPPING[operator](float_compare(res['rating_avg'], value, 2), 0)
        ]
        return [('id', 'in', res_ids)]

    @api.depends('rating_avg')
    def _compute_rating_avg_text(self):
        for record in self:
            record.rating_avg_text = rating_data._rating_avg_to_text(record.rating_avg)

    @api.depends('rating_ids.res_id', 'rating_ids.rating')
    def _compute_rating_satisfaction(self):
        """ Compute the rating satisfaction percentage, this is done separately from rating_count and rating_avg
            since the query is different, to avoid computing if it is not necessary"""
        domain = expression.AND([self._rating_domain(), [('rating', '>=', rating_data.RATING_LIMIT_MIN)]])
        # See `_compute_rating_percentage_satisfaction` above
        read_group_res = self.env['rating.rating']._read_group(domain, ['res_id', 'rating'], groupby=['res_id', 'rating'], lazy=False)
        default_grades = {'great': 0, 'okay': 0, 'bad': 0}
        grades_per_record = {record_id: default_grades.copy() for record_id in self.ids}

        for group in read_group_res:
            record_id = group['res_id']
            grade = rating_data._rating_to_grade(group['rating'])
            grades_per_record[record_id][grade] += group['__count']

        for record in self:
            grade_repartition = grades_per_record.get(record.id, default_grades)
            grade_count = sum(grade_repartition.values())
            record.rating_percentage_satisfaction = grade_repartition['great'] * 100 / grade_count if grade_count else -1

    def write(self, values):
        """ If the rated ressource name is modified, we should update the rating res_name too.
            If the rated ressource parent is changed we should update the parent_res_id too"""
        with self.env.norecompute():
            result = super(RatingMixin, self).write(values)
            for record in self:
                if record._rec_name in values:  # set the res_name of ratings to be recomputed
                    res_name_field = self.env['rating.rating']._fields['res_name']
                    self.env.add_to_compute(res_name_field, record.rating_ids)
                if record._rating_get_parent_field_name() in values:
                    record.rating_ids.sudo().write({'parent_res_id': record[record._rating_get_parent_field_name()].id})

        return result

    def unlink(self):
        """ When removing a record, its rating should be deleted too. """
        record_ids = self.ids
        result = super(RatingMixin, self).unlink()
        self.env['rating.rating'].sudo().search([('res_model', '=', self._name), ('res_id', 'in', record_ids)]).unlink()
        return result

    def _rating_get_parent_field_name(self):
        """Return the parent relation field name
           Should return a Many2One"""
        return None

    def _rating_domain(self):
        """ Returns a normalized domain on rating.rating to select the records to
            include in count, avg, ... computation of current model.
        """
        return ['&', '&', ('res_model', '=', self._name), ('res_id', 'in', self.ids), ('consumed', '=', True)]

    def _rating_get_partner(self):
        """ Return the customer (partner) that performs the rating.

        :return record: res.partner singleton
        """
        if hasattr(self, 'partner_id') and self.partner_id:
            return self.partner_id
        return self.env['res.partner']

    def _rating_get_operator(self):
        """ Return the operator (partner) that is the person who is rated.

        :return record: res.partner singleton
        """
        if hasattr(self, 'user_id') and self.user_id.partner_id:
            return self.user_id.partner_id
        return self.env['res.partner']

    def _rating_get_access_token(self, partner=None):
        """ Return access token linked to existing ratings, or create a new rating
        that will create the asked token. An explicit call to access rights is
        performed as sudo is used afterwards as this method could be used from
        different sources, notably templates. """
        self.check_access_rights('read')
        self.check_access_rule('read')
        if not partner:
            partner = self._rating_get_partner()
        rated_partner = self._rating_get_operator()
        ratings = self.rating_ids.sudo().filtered(lambda x: x.partner_id.id == partner.id and not x.consumed)
        if not ratings:
            rating = self.env['rating.rating'].sudo().create({
                'partner_id': partner.id,
                'rated_partner_id': rated_partner.id,
                'res_model_id': self.env['ir.model']._get_id(self._name),
                'res_id': self.id,
                'is_internal': False,
            })
        else:
            rating = ratings[0]
        return rating.access_token

    def rating_send_request(self, template, lang=False, subtype_id=False, force_send=True, composition_mode='comment',
                            email_layout_xmlid=None):
        """ This method send rating request by email, using a template given
        in parameter.

         :param record template: a mail.template record used to compute the message body;
         :param str lang: optional lang; it can also be specified directly on the template
           itself in the lang field;
         :param int subtype_id: optional subtype to use when creating the message; is
           a note by default to avoid spamming followers;
         :param bool force_send: whether to send the request directly or use the mail
           queue cron (preferred option);
         :param str composition_mode: comment (message_post) or mass_mail (template.send_mail);
         :param str email_layout_xmlid: layout used to encapsulate the content when sending email;
        """
        if lang:
            template = template.with_context(lang=lang)
        if subtype_id is False:
            subtype_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note')
        if force_send:
            self = self.with_context(mail_notify_force_send=True)  # default value is True, should be set to false if not?
        for record in self:
            record.message_post_with_template(
                template.id,
                composition_mode=composition_mode,
                email_layout_xmlid=email_layout_xmlid if email_layout_xmlid is not None else 'mail.mail_notification_light',
                subtype_id=subtype_id
            )

    def rating_apply(self, rate, token=None, rating=None, feedback=None,
                     subtype_xmlid=None, notify_delay_send=False):
        """ Apply a rating to the record. This rating can either be linked to a
        token (customer flow) or directly a rating record (code flow).

        If the current model inherits from mail.thread mixin a message is posted
        on its chatter. User going through this method should have at least
        employee rights as well as rights on the current record because of rating
        manipulation and chatter post (either employee, either sudo-ed in public
        controllers after security check granting access).

        :param float rate: the rating value to apply (from 0 to 5);
        :param string token: access token to fetch the rating to apply (optional);
        :param record rating: rating.rating to apply (if no token);
        :param string feedback: additional feedback (plaintext);
        :param string subtype_xmlid: xml id of a valid mail.message.subtype used
          to post the message (if it applies). If not given a classic comment is
          posted;
        :param notify_delay_send: Delay the sending by 2 hours of the email so the user
            can still change his feedback. If False, the email will be sent immediately.

        :returns rating: rating.rating record
        """
        if rate < 0 or rate > 5:
            raise ValueError('Wrong rating value. A rate should be between 0 and 5 (received %d).' % rate)
        if token:
            rating = self.env['rating.rating'].search([('access_token', '=', token)], limit=1)
        if not rating:
            raise ValueError('Invalid token or rating.')

        rating.write({'rating': rate, 'feedback': feedback, 'consumed': True})
        if isinstance(self, self.env.registry['mail.thread']):
            if subtype_xmlid is None:
                subtype_id = self._rating_apply_get_default_subtype_id()
            else:
                subtype_id = self.env['ir.model.data']._xmlid_to_res_id(subtype_xmlid)
            feedback = tools.plaintext2html(feedback or '')

            scheduled_datetime = (
                fields.Datetime.now() + datetime.timedelta(hours=2)
                if notify_delay_send else None
            )
            rating_body = (
                markupsafe.Markup(
                    "<img src='%s' alt=':%s/5' style='width:18px;height:18px;float:left;margin-right: 5px;'/>%s"
                ) % (rating.rating_image_url, rate, feedback)
            )

            if rating.message_id:
                self._message_update_content(
                    rating.message_id, rating_body,
                    scheduled_date=scheduled_datetime,
                    strict=False
                )
            else:
                self.message_post(
                    author_id=rating.partner_id.id or None,  # None will set the default author in mail_thread.py
                    body=rating_body,
                    rating_id=rating.id,
                    scheduled_date=scheduled_datetime,
                    subtype_id=subtype_id,
                )
        return rating

    def _rating_apply_get_default_subtype_id(self):
        return self.env['ir.model.data']._xmlid_to_res_id("mail.mt_comment")

    def _rating_get_repartition(self, add_stats=False, domain=None):
        """ get the repatition of rating grade for the given res_ids.
            :param add_stats : flag to add stat to the result
            :type add_stats : boolean
            :param domain : optional extra domain of the rating to include/exclude in repartition
            :return dictionnary
                if not add_stats, the dict is like
                    - key is the rating value (integer)
                    - value is the number of object (res_model, res_id) having the value
                otherwise, key is the value of the information (string) : either stat name (avg, total, ...) or 'repartition'
                containing the same dict if add_stats was False.
        """
        base_domain = expression.AND([self._rating_domain(), [('rating', '>=', 1)]])
        if domain:
            base_domain += domain
        rg_data = self.env['rating.rating'].read_group(base_domain, ['rating'], ['rating', 'res_id'])
        # init dict with all posible rate value, except 0 (no value for the rating)
        values = dict.fromkeys(range(1, 6), 0)
        for rating_rg in rg_data:
            rating_val_round = float_round(rating_rg['rating'], precision_digits=1)
            values[rating_val_round] = values.get(rating_val_round, 0) + rating_rg['rating_count']
        # add other stats
        if add_stats:
            rating_number = sum(values.values())
            result = {
                'repartition': values,
                'avg': sum(float(key * values[key]) for key in values) / rating_number if rating_number > 0 else 0,
                'total': sum(it['rating_count'] for it in rg_data),
            }
            return result
        return values

    def rating_get_grades(self, domain=None):
        """ get the repatition of rating grade for the given res_ids.
            :param domain : optional domain of the rating to include/exclude in grades computation
            :return dictionnary where the key is the grade (great, okay, bad), and the value, the number of object (res_model, res_id) having the grade
                    the grade are compute as    0-30% : Bad
                                                31-69%: Okay
                                                70-100%: Great
        """
        data = self._rating_get_repartition(domain=domain)
        res = dict.fromkeys(['great', 'okay', 'bad'], 0)
        for key in data:
            grade = rating_data._rating_to_grade(key)
            res[grade] += data[key]
        return res

    def rating_get_stats(self, domain=None):
        """ get the statistics of the rating repatition
            :param domain : optional domain of the rating to include/exclude in statistic computation
            :return dictionnary where
                - key is the name of the information (stat name)
                - value is statistic value : 'percent' contains the repartition in percentage, 'avg' is the average rate
                  and 'total' is the number of rating
        """
        data = self._rating_get_repartition(domain=domain, add_stats=True)
        result = {
            'avg': data['avg'],
            'total': data['total'],
            'percent': dict.fromkeys(range(1, 6), 0),
        }
        for rate in data['repartition']:
            result['percent'][rate] = (data['repartition'][rate] * 100) / data['total'] if data['total'] > 0 else 0
        return result

```

## File: models\rating_parent_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from datetime import timedelta

from odoo import api, fields, models
from odoo.addons.rating.models import rating_data
from odoo.osv import expression
from odoo.tools.float_utils import float_compare


class RatingParentMixin(models.AbstractModel):
    _name = 'rating.parent.mixin'
    _description = "Rating Parent Mixin"
    _rating_satisfaction_days = False  # Number of last days used to compute parent satisfaction. Set to False to include all existing rating.

    rating_ids = fields.One2many(
        'rating.rating', 'parent_res_id', string='Ratings',
        auto_join=True, groups='base.group_user',
        domain=lambda self: [('parent_res_model', '=', self._name)])
    rating_percentage_satisfaction = fields.Integer(
        "Rating Satisfaction",
        compute="_compute_rating_percentage_satisfaction", compute_sudo=True,
        store=False, help="Percentage of happy ratings")
    rating_count = fields.Integer(string='# Ratings', compute="_compute_rating_percentage_satisfaction", compute_sudo=True)
    rating_avg = fields.Float('Average Rating', groups='base.group_user',
        compute='_compute_rating_percentage_satisfaction', compute_sudo=True, search='_search_rating_avg')
    rating_avg_percentage = fields.Float('Average Rating (%)', groups='base.group_user',
        compute='_compute_rating_percentage_satisfaction', compute_sudo=True)

    @api.depends('rating_ids.rating', 'rating_ids.consumed')
    def _compute_rating_percentage_satisfaction(self):
        # build domain and fetch data
        domain = [('parent_res_model', '=', self._name), ('parent_res_id', 'in', self.ids), ('rating', '>=', rating_data.RATING_LIMIT_MIN), ('consumed', '=', True)]
        if self._rating_satisfaction_days:
            domain += [('write_date', '>=', fields.Datetime.to_string(fields.datetime.now() - timedelta(days=self._rating_satisfaction_days)))]
        data = self.env['rating.rating'].read_group(domain, ['parent_res_id', 'rating'], ['parent_res_id', 'rating'], lazy=False)

        # get repartition of grades per parent id
        default_grades = {'great': 0, 'okay': 0, 'bad': 0}
        grades_per_parent = dict((parent_id, dict(default_grades)) for parent_id in self.ids)  # map: {parent_id: {'great': 0, 'bad': 0, 'ok': 0}}
        rating_scores_per_parent = defaultdict(int)  # contains the total of the rating values per record
        for item in data:
            parent_id = item['parent_res_id']
            grade = rating_data._rating_to_grade(item['rating'])
            grades_per_parent[parent_id][grade] += item['__count']
            rating_scores_per_parent[parent_id] += item['rating'] * item['__count']

        # compute percentage per parent
        for record in self:
            repartition = grades_per_parent.get(record.id, default_grades)
            rating_count = sum(repartition.values())
            record.rating_count = rating_count
            record.rating_percentage_satisfaction = repartition['great'] * 100 / rating_count if rating_count else -1
            record.rating_avg = rating_scores_per_parent[record.id] / rating_count if rating_count else 0
            record.rating_avg_percentage = record.rating_avg / 5

    def _search_rating_avg(self, operator, value):
        if operator not in rating_data.OPERATOR_MAPPING:
            raise NotImplementedError('This operator %s is not supported in this search method.' % operator)
        domain = [('parent_res_model', '=', self._name), ('consumed', '=', True), ('rating', '>=', rating_data.RATING_LIMIT_MIN)]
        if self._rating_satisfaction_days:
            min_date = fields.datetime.now() - timedelta(days=self._rating_satisfaction_days)
            domain = expression.AND([domain, [('write_date', '>=', fields.Datetime.to_string(min_date))]])
        rating_read_group = self.env['rating.rating'].sudo().read_group(domain, ['parent_res_id', 'rating_avg:avg(rating)'], ['parent_res_id'])
        parent_res_ids = [
            res['parent_res_id']
            for res in rating_read_group
            if rating_data.OPERATOR_MAPPING[operator](float_compare(res['rating_avg'], value, 2), 0)
        ]
        return [('id', 'in', parent_res_ids)]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import rating
from . import rating_data
from . import rating_mixin
from . import rating_parent_mixin
from . import mail_thread
from . import mail_message

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_rating_user,rating.rating.user,rating.model_rating_rating,base.group_user,1,1,1,0
access_rating_public,rating.rating.public,rating.model_rating_rating,base.group_public,0,0,0,0
access_rating_portal,rating.rating.portal,rating.model_rating_rating,base.group_portal,0,0,0,0
rating_rating_access_system,rating.rating.access.system,rating.model_rating_rating,base.group_system,1,1,1,1

```

## File: static\src\components\channel_preview_view\channel_preview_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ChannelPreviewView" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_ChannelPreviewView_messageBody')]" position="replace">
            <t t-if="channelPreviewView.isRating">
                <span class="o_ChannelPreviewView_ratingText">Rating:</span>
                <img class="o_ChannelPreviewView_ratingImage ms-2" t-att-src="channelPreviewView.thread.lastMessage.rating.ratingImageUrl" t-att-alt="channelPreviewView.thread.lastMessage.rating.ratingText"/>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_needaction_preview\thread_needaction_preview.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadNeedactionPreview" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_ThreadNeedactionPreview_messageBody')]" position="replace">
            <t t-if="threadNeedactionPreviewView.isRating">
                <span class="o_ThreadNeedactionPreview_ratingText">Rating:</span>
                <img class="o_ThreadNeedactionPreview_ratingImage ms-2" t-att-src="threadNeedactionPreviewView.thread.lastNeedactionMessageAsOriginThread.rating.ratingImageUrl" t-att-alt="threadNeedactionPreviewView.thread.lastNeedactionMessageAsOriginThread.rating.ratingText"/>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\img\rating_1.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 68 68"><path d="M34,68A34,34,0,1,1,68,34,34,34,0,0,1,34,68ZM34,4A30,30,0,1,0,64,34,30,30,0,0,0,34,4Z" style="fill:#d23f3a"/><path d="M49.55,51.18a2,2,0,0,1-1.63-.84,17.2,17.2,0,0,0-27.84,0A2,2,0,0,1,16.82,48a21.21,21.21,0,0,1,34.36,0,2,2,0,0,1-1.63,3.2Z" style="fill:#d23f3a"/><circle cx="20.9" cy="30.19" r="4.79" style="fill:#d23f3a"/><circle cx="47" cy="30.19" r="4.79" style="fill:#d23f3a"/></svg>
```

## File: static\src\img\rating_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 68 68"><path d="M34,68A34,34,0,1,1,68,34,34,34,0,0,1,34,68ZM34,4A30,30,0,1,0,64,34,30,30,0,0,0,34,4Z" style="fill:#f7931e"/><path d="M18.45,47.15a2,2,0,0,1,0-4h31.1a2,2,0,0,1,0,4Z" style="fill:#f7931e"/><circle cx="20.9" cy="30.19" r="4.79" style="fill:#f7931e"/><circle cx="47" cy="30.19" r="4.79" style="fill:#f7931e"/></svg>
```

## File: static\src\img\rating_5.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 68 68"><path d="M34,68A34,34,0,1,1,68,34,34,34,0,0,1,34,68ZM34,4A30,30,0,1,0,64,34,30,30,0,0,0,34,4Z" style="fill:#008818"/><path d="M34,55.21a21.26,21.26,0,0,1-17.18-8.78,2,2,0,1,1,3.26-2.36,17.18,17.18,0,0,0,27.84,0,2,2,0,0,1,3.26,2.36A21.26,21.26,0,0,1,34,55.21Z" style="fill:#008818"/><circle cx="20.9" cy="30.19" r="4.79" style="fill:#008818"/><circle cx="47" cy="30.19" r="4.79" style="fill:#008818"/></svg>
```

## File: static\src\models\channel_preview_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerPatch({
    name: 'ChannelPreviewView',
    fields: {
        isEmpty: {
            compute() {
                return this.isRating || this._super();
            },
        },
        isRating: attr({
            compute() {
                return Boolean(this.thread.lastMessage && this.thread.lastMessage.rating);
            },
        }),
    },
});

```

## File: static\src\models\message.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { one } from '@mail/model/model_field';

registerPatch({
    name: 'Message',
    modelMethods: {
        /**
         * @override
         */
        convertData(data) {
            const data2 = this._super(data);
            if ('rating' in data) {
                data2.rating = data.rating;
            }
            return data2;
        },
    },
    fields: {
        rating: one('Rating', {
            isCausal: true,
        }),
    },
});

```

## File: static\src\models\rating.js

```javascript
/** @odoo-module **/

import { registerModel } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerModel({
    name: 'Rating',
    fields: {
        id: attr({
            identifying: true,
        }),
        ratingImageUrl: attr({
            readonly: true,
        }),
        ratingText: attr({
            readonly: true,
        }),
    },
});

```

## File: static\src\models\thread_needaction_preview_view.js

```javascript
/** @odoo-module **/

import { registerPatch } from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerPatch({
    name: 'ThreadNeedactionPreviewView',
    fields: {
        isEmpty: {
            compute() {
                return this.isRating || this._super();
            },
        },
        isRating: attr({
            compute() {
                return Boolean(this.thread.lastNeedactionMessageAsOriginThread && this.thread.lastNeedactionMessageAsOriginThread.rating);
            },
        }),
    },
});

```

## File: views\mail_message_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="mail_message_view_form" model="ir.ui.view">
        <field name="name">mail.message.view.form.inherit.rating</field>
        <field name="model">mail.message</field>
        <field name="inherit_id" ref="mail.mail_message_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='page_tracking']" position="after">
                <page string="Ratings" name="page_rating">
                    <field name="rating_ids"/>
                </page>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="rating_rating_view_tree" model="ir.ui.view">
            <field name="name">rating.rating.tree</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <tree string="Ratings" create="false" edit="false" sample="1">
                    <field name="create_date"/>
                    <field name="rated_partner_id" optional="show"/>
                    <field name="partner_id" optional="show"/>
                    <field name="parent_res_name" optional="show"/>
                    <field name="res_name" optional="show"/>
                    <field name="feedback" optional="hide"/>
                    <field name="rating_text" decoration-danger="rating_text == 'ko'" decoration-warning="rating_text == 'ok'" decoration-success="rating_text == 'top'" class="fw-bold" widget="badge"/>
                </tree>
            </field>
        </record>

        <record id="rating_rating_view_form" model="ir.ui.view">
            <field name="name">rating.rating.form</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <form string="Ratings" create="false">
                    <sheet>
                        <group>
                            <group>
                                <field name="resource_ref" string="Document"/>
                                <field name="res_name" string="Document" invisible="1"/>
                                <field name="parent_ref" string="Parent Holder"/>
                                <field name="parent_res_name" string="Parent Holder" invisible="1"/>
                                <field name="rated_partner_id" widget="many2one_avatar"/>
                                <field name="rating" invisible="1"/>
                                <field name="is_internal"/>
                            </group>
                            <group>
                                <field name="partner_id"/>
                                <div colspan="2" class="text-center" name="rating_image_container">
                                    <field name="rating_image" widget='image'/>
                                    <div class="mt4">
                                        <strong><field name="rating_text"/></strong>
                                    </div>
                                </div>
                                <field name="create_date"/>
                                <field name="feedback" attrs="{'invisible': [('feedback','=',False)]}"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="rating_rating_view_form_text" model="ir.ui.view">
            <field name="name">rating.rating.view.form.text</field>
            <field name="model">rating.rating</field>
            <field name="inherit_id" ref="rating.rating_rating_view_form"/>
            <field name="priority">32</field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='rating_image_container']" position="replace">
                    <field name="rating_text" string="Rating"
                        decoration-danger="rating_text == 'ko'"
                        decoration-warning="rating_text == 'ok'"
                        decoration-success="rating_text == 'top'"
                        widget='badge'/>
                </xpath>
            </field>
        </record>

        <record id="rating_rating_view_form_complete" model="ir.ui.view">
            <field name="name">rating.rating.view.form.complete</field>
            <field name="model">rating.rating</field>
            <field name="inherit_id" ref="rating.rating_rating_view_form"/>
            <field name="priority">48</field>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='is_internal']" position="after">
                    <field name="consumed" groups="base.group_no_one"/>
                </xpath>
            </field>
        </record>

        <record id="rating_rating_view_kanban" model="ir.ui.view">
            <field name="name">rating.rating.kanban</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <kanban create="false" sample="1">
                    <field name="rating"/>
                    <field name="res_name"/>
                    <field name="feedback"/>
                    <field name="partner_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div class="oe_kanban_global_click d-flex align-items-center justify-content-center">
                                <div class="row oe_kanban_details">
                                    <div class="col-4 my-auto">
                                        <field name="rating_image" widget="image" class="bg-view" />
                                    </div>
                                    <div class="col-8 ps-1">
                                        <strong>
                                            <field name="rated_partner_name"/>
                                        </strong>
                                        <ul>
                                            <li t-if="record.partner_id.value">
                                                <span class="o_text_overflow">
                                                    by
                                                    <span t-att-title="record.partner_id.value">
                                                        <field name="partner_id" />
                                                    </span>
                                                </span>
                                            </li>
                                            <li>
                                                <span class="o_text_overflow">
                                                    for
                                                    <a type="object" name="action_open_rated_object" t-att-title="record.res_name.raw_value">
                                                        <field name="res_name" />
                                                    </a>
                                                </span>
                                            </li>
                                            <li>
                                                on <field name="create_date" />
                                            </li>
                                            <li t-if="record.feedback.raw_value" class="o_text_overflow" t-att-title="record.feedback.raw_value">
                                                <field name="feedback"/>
                                            </li>
                                        </ul>
                                </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="rating_rating_view_kanban_stars" model="ir.ui.view">
            <field name="name">rating.rating.view.kanban.stars</field>
            <field name="model">rating.rating</field>
            <field name="priority">20</field>
            <field name="arch" type="xml">
                <kanban create="false" class="o_rating_rating_kanban">
                    <field name="rating"/>
                    <field name="res_name"/>
                    <field name="feedback"/>
                    <field name="partner_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <t t-set="val_stars" t-value="Math.round(record.rating.raw_value * 10) / 10"/>
                            <t t-set="val_integer" t-value="Math.floor(val_stars)"/>
                            <t t-set="val_decimal" t-value="val_stars - val_integer"/>
                            <t t-set="empty_star" t-value="5 - (val_integer + Math.ceil(val_decimal))"/>
                            <div class="oe_kanban_card oe_kanban_global_click">
                                <div class="d-flex flex-row">
                                    <div class="o_rating_kanban_left me-3">
                                        <h1 class="o_rating_value text-center text-primary" t-esc="val_stars"/>
                                        <i t-foreach="[...Array(val_integer).keys()]" t-as="num"  t-key="num"
                                           class="fa fa-star"
                                           aria-label="A star"
                                           role="img"/>
                                        <i t-if="val_decimal"
                                           class="fa fa-star-half-o"
                                           aria-label="Half a star"
                                           role="img"/>
                                        <i t-foreach="[...Array(empty_star).keys()]" t-as="num" t-key="num"
                                           class="fa fa-star text-black-25"
                                           aria-label="A star"
                                           role="img"/>
                                    </div>
                                    <div>
                                        <div class="o_kanban_card_header">
                                            <div class="o_kanban_card_header_title">
                                                <span class="fw-bold"><field name="partner_id"/></span>
                                            </div>
                                        </div>
                                        <div class="o_kanban_card_content mt0 d-flex flex-column">
                                            <span>
                                                <i class="fa fa-folder me-2" aria-label="Open folder"></i>
                                                <a type="object" name="action_open_rated_object" t-att-title="record.res_name.raw_value">
                                                    <field name="res_name" />
                                                </a>
                                            </span>
                                            <span><i class="fa fa-clock-o me-2" aria-label="Create date"/> <field name="create_date" /></span>
                                            <div class="d-flex mt-2">
                                                <span t-esc="record.feedback.raw_value"/>
                                            </div>
                                        </div>
                                    </div>
                                 </div>
                             </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="rating_rating_view_pivot" model="ir.ui.view">
            <field name="name">rating.rating.pivot</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <pivot string="Ratings" display_quantity="1" sample="1">
                    <field name="rated_partner_id" type="row"/>
                    <field name="create_date" type="col"/>
                    <field name="rating" type="measure" string="Rating Value (/5)"/>
                    <field name="parent_res_id" invisible="1"/>
                    <field name="res_id" invisible="1"/>
                </pivot>
            </field>
        </record>

        <record id="rating_rating_view_graph" model="ir.ui.view">
           <field name="name">rating.rating.graph</field>
           <field name="model">rating.rating</field>
           <field name="arch" type="xml">
                <graph string="Ratings" sample="1">
                    <field name="create_date"/>
                    <field name="rating" type="measure" string="Rating Value (/5)"/>
                    <field name="parent_res_id" invisible="1"/>
                    <field name="res_id" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="rating_rating_view_search" model="ir.ui.view">
            <field name="name">rating.rating.search</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <search string="Ratings">
                    <field name="rated_partner_id"/>
                    <field name="rating"/>
                    <field name="partner_id"/>
                    <field name="res_name" filter_domain="[('res_name','ilike',self)]"/>
                    <field name="res_id"/>
                    <field name="parent_res_name" filter_domain="[('parent_res_name','ilike',self)]"/>
                    <filter string="My Ratings" name="my_ratings" domain="[('rated_partner_id.user_ids', 'in', [uid])]"/>
                    <separator/>
                    <filter string="Satisfied" name="rating_happy" domain="[('rating_text', '=', 'top')]"/>
                    <filter string="Okay" name="rating_okay" domain="[('rating_text', '=', 'ok')]"/>
                    <filter string="Dissatisfied" name="rating_unhappy" domain="[('rating_text', '=', 'ko')]"/>
                    <separator/>
                    <filter string="Today" name="today" domain="[('create_date', '&gt;', (context_today() - datetime.timedelta(days=1)).strftime('%Y-%m-%d'))]"/>
                    <filter string="Last 7 days" name="last_7days" domain="[('create_date','&gt;', (context_today() - datetime.timedelta(days=7)).strftime('%Y-%m-%d'))]"/>
                    <filter string="Last 30 days" name="last_month" domain="[('create_date','&gt;', (context_today() - datetime.timedelta(days=30)).strftime('%Y-%m-%d'))]"/>
                    <separator/>
                    <filter name="filter_create_date" date="create_date"/>
                    <group expand="0" string="Group By">
                        <filter string="Rated Operator" name="responsible" context="{'group_by':'rated_partner_id'}"/>
                        <filter string="Customer" name="customer" context="{'group_by':'partner_id'}"/>
                        <filter string="Rating" name="rating_text" context="{'group_by':'rating_text'}"/>
                        <filter string="Resource" name="resource" context="{'group_by':'res_name'}"/>
                        <filter string="Date" name="month" context="{'group_by':'create_date:month'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="rating_rating_action" model="ir.actions.act_window">
            <field name="name">Ratings</field>
            <field name="res_model">rating.rating</field>
            <field name="view_mode">kanban,tree,graph,pivot,form</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    No rating yet
                </p><p>
                    There is no rating for this object at the moment.
                </p>
            </field>
        </record>
        <record id="rating_rating_action_view_kanban" model="ir.actions.act_window.view">
            <field name="act_window_id" ref="rating_rating_action"/>
            <field name="sequence">1</field>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="rating.rating_rating_view_kanban"/>
        </record>
        <record id="rating_rating_action_view_form" model="ir.actions.act_window.view">
            <field name="act_window_id" ref="rating_rating_action"/>
            <field name="sequence">5</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="rating_rating_view_form_complete"/>
        </record>

        <!-- Add menu entry in Technical/Discuss -->
        <menuitem name="Ratings"
            id="rating_rating_menu_technical"
            parent="mail.mail_menu_technical"
            action="rating_rating_action"
            sequence="30"/>

</odoo>

```

## File: views\rating_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
        <!-- External page : thanks message -->
        <template id="rating_external_page_view" name="Rating Page Done">
            <t t-call="web.frontend_layout">
                <div class="container pt-5">
                    <div class="oe_structure" id="oe_structure_rating_header_thanks_you"/>
                    <div class="text-center mt-5 pt-5">
                        <h3 class="d-inline">Thank you, we appreciate your feedback!</h3>
                    </div>
                    <div class="text-center">
                        <a role="button" t-att-href="web_base_url" class="btn btn-primary my-5">Go back to the Homepage</a>
                    </div>
                    <div class="oe_structure" id="oe_structure_rating_footer_thanks_you"/>
                </div>
            </t>
        </template>

        <!-- External page: rate and submit feedback -->
        <template id="rating_external_page_submit" name="Rate our Services">
            <t t-call="web.frontend_layout">
                <div class="container mb-5 mt-4 o_rating_page_submit">
                    <div class="row text-center justify-content-center">
                        <h1 class="col-12 mt-5">Thank you for rating our services!</h1>
                        <form class="col-md-6" t-attf-action="/rate/#{token}/submit_feedback" method="post">
                            <div class="btn-group row flex-nowrap justify-content-center w-100 mt-5"
                                 role="group"
                                 data-bs-toggle="buttons">
                                <t t-foreach="rate_names" t-as="rate_name">
                                    <input type="radio" name="rate"
                                           class="btn-check"
                                           t-attf-id="rate_{{rate_name}}"
                                           t-att-value="rate_name"
                                           t-att-checked="rate == rate_name"/>
                                    <label t-attf-class="col p-4 btn o_rating_label shadow-none transition-base {{rate == rate_name and 'active' or ''}}"
                                           t-att-for="'rate_%s' % (rate_name)">
                                        <img t-attf-src='/rating/static/src/img/rating_#{rate_name}.svg'
                                             t-att-alt="rate_name_value"
                                             t-att-title="rate_name_value"
                                             t-attf-class="o_{{rate_name_value.lower()}}"/>
                                    </label>
                                </t>
                            </div>
                            <p class="mt-5">
                                Feel free to write a feedback on your experience:
                            </p>
                            <input type="hidden" name="csrf_token"
                                   t-att-value="request.csrf_token()"/>
                            <textarea class="form-control" name="feedback" rows="8"
                                      t-att-value="rating.feedback"></textarea>
                            <button type="submit" class="btn btn-primary mt-4"
                                    style="margin-top:8px;">Send Feedback</button>
                        </form>
                    </div>
                </div>
            </t>
        </template>
</odoo>

```

