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
{
    'name': 'Customer Rating',
    'version': '1.0',
    'category': 'Productivity',
    'description': """
This module allows a customer to give rating.
""",
    'depends': [
        'mail',
    ],
    'data': [
        'views/rating_rating_views.xml',
        'views/rating_template.xml',
        'views/mail_message_views.xml',
        'views/assets.xml',
        'security/ir.model.access.csv'
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

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

    @http.route('/rating/<string:token>/<int:rate>', type='http', auth="public", website=True)
    def open_rating(self, token, rate, **kwargs):
        _logger.warning('/rating is deprecated, use /rate instead')
        assert rate in (1, 5, 10), "Incorrect rating"
        return self.action_open_rating(token, MAPPED_RATES.get(rate), **kwargs)

    @http.route(['/rating/<string:token>/submit_feedback'], type="http", auth="public", methods=['post'], website=True)
    def submit_rating(self, token, **kwargs):
        _logger.warning('/rating is deprecated, use /rate instead')
        rate = int(kwargs.get('rate'))
        assert rate in (1, 5, 10), "Incorrect rating"
        kwargs['rate'] = MAPPED_RATES.gate(rate)
        return self.action_submit_rating(token, **kwargs)

    @http.route('/rate/<string:token>/<int:rate>', type='http', auth="public", website=True)
    def action_open_rating(self, token, rate, **kwargs):
        assert rate in (1, 3, 5), "Incorrect rating"
        rating = request.env['rating.rating'].sudo().search([('access_token', '=', token)])
        if not rating:
            return request.not_found()
        rate_names = {
            5: _("satisfied"),
            3: _("not satisfied"),
            1: _("highly dissatisfied")
        }
        rating.write({'rating': rate, 'consumed': True})
        lang = rating.partner_id.lang or get_lang(request.env).code
        return request.env['ir.ui.view'].with_context(lang=lang)._render_template('rating.rating_external_page_submit', {
            'rating': rating, 'token': token,
            'rate_names': rate_names, 'rate': rate
        })

    @http.route(['/rate/<string:token>/submit_feedback'], type="http", auth="public", methods=['post', 'get'], website=True)
    def action_submit_rating(self, token, **kwargs):
        rating = request.env['rating.rating'].sudo().search([('access_token', '=', token)])
        if not rating:
            return request.not_found()
        if request.httprequest.method == "POST":
            rate = int(kwargs.get('rate'))
            assert rate in (1, 3, 5), "Incorrect rating"
            record_sudo = request.env[rating.res_model].sudo().browse(rating.res_id)
            record_sudo.rating_apply(rate, token=token, feedback=kwargs.get('feedback'))
        lang = rating.partner_id.lang or get_lang(request.env).code
        return request.env['ir.ui.view'].with_context(lang=lang)._render_template('rating.rating_external_page_view', {
            'web_base_url': request.env['ir.config_parameter'].sudo().get_param('web.base.url'),
            'rating': rating,
        })

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

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        rating_value = kwargs.pop('rating_value', False)
        rating_feedback = kwargs.pop('rating_feedback', False)
        message = super(MailThread, self).message_post(**kwargs)

        # create rating.rating record linked to given rating_value. Using sudo as portal users may have
        # rights to create messages and therefore ratings (security should be checked beforehand)
        if rating_value:
            ir_model = self.env['ir.model'].sudo().search([('model', '=', self._name)])
            self.env['rating.rating'].sudo().create({
                'rating': float(rating_value) if rating_value is not None else False,
                'feedback': rating_feedback,
                'res_model_id': ir_model.id,
                'res_id': self.id,
                'message_id': message.id,
                'consumed': True,
                'partner_id': self.env.user.partner_id.id,
            })
        return message

```

## File: models\rating.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import uuid

from odoo import api, fields, models

from odoo.modules.module import get_resource_path

RATING_LIMIT_SATISFIED = 5
RATING_LIMIT_OK = 3
RATING_LIMIT_MIN = 1


class Rating(models.Model):
    _name = "rating.rating"
    _description = "Rating"
    _order = 'write_date desc'
    _rec_name = 'res_name'
    _sql_constraints = [
        ('rating_range', 'check(rating >= 0 and rating <= 5)', 'Rating should be between 0 and 5'),
    ]

    @api.depends('res_model', 'res_id')
    def _compute_res_name(self):
        for rating in self:
            name = self.env[rating.res_model].sudo().browse(rating.res_id).name_get()
            rating.res_name = name and name[0][1] or ('%s/%s') % (rating.res_model, rating.res_id)

    @api.model
    def _default_access_token(self):
        return uuid.uuid4().hex

    @api.model
    def _selection_target_model(self):
        return [(model.model, model.name) for model in self.env['ir.model'].search([])]

    create_date = fields.Datetime(string="Submitted on")
    res_name = fields.Char(string='Resource name', compute='_compute_res_name', store=True, help="The name of the rated resource.")
    res_model_id = fields.Many2one('ir.model', 'Related Document Model', index=True, ondelete='cascade', help='Model of the followed resource')
    res_model = fields.Char(string='Document Model', related='res_model_id.model', store=True, index=True, readonly=True)
    res_id = fields.Integer(string='Document', required=True, help="Identifier of the rated object", index=True)
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
    rated_partner_id = fields.Many2one('res.partner', string="Rated Operator", help="Owner of the rated resource")
    partner_id = fields.Many2one('res.partner', string='Customer', help="Author of the rating")
    rating = fields.Float(string="Rating Value", group_operator="avg", default=0, help="Rating value: 0=Unhappy, 5=Happy")
    rating_image = fields.Binary('Image', compute='_compute_rating_image')
    rating_text = fields.Selection([
        ('satisfied', 'Satisfied'),
        ('not_satisfied', 'Not satisfied'),
        ('highly_dissatisfied', 'Highly dissatisfied'),
        ('no_rating', 'No Rating yet')], string='Rating', store=True, compute='_compute_rating_text', readonly=True)
    feedback = fields.Text('Comment', help="Reason of the rating")
    message_id = fields.Many2one(
        'mail.message', string="Message",
        index=True, ondelete='cascade',
        help="Associated message when posting a review. Mainly used in website addons.")
    is_internal = fields.Boolean('Visible Internally Only', readonly=False, related='message_id.is_internal', store=True)
    access_token = fields.Char('Security Token', default=_default_access_token, help="Access token to set the rating of the value")
    consumed = fields.Boolean(string="Filled Rating", help="Enabled if the rating has been filled.")

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
        if self.rating >= RATING_LIMIT_SATISFIED:
            rating_int = 5
        elif self.rating >= RATING_LIMIT_OK:
            rating_int = 3
        elif self.rating >= RATING_LIMIT_MIN:
            rating_int = 1
        else:
            rating_int = 0
        return 'rating_%s.png' % rating_int

    def _compute_rating_image(self):
        for rating in self:
            try:
                image_path = get_resource_path('rating', 'static/src/img', rating._get_rating_image_filename())
                rating.rating_image = base64.b64encode(open(image_path, 'rb').read()) if image_path else False
            except (IOError, OSError):
                rating.rating_image = False

    @api.depends('rating')
    def _compute_rating_text(self):
        for rating in self:
            if rating.rating >= RATING_LIMIT_SATISFIED:
                rating.rating_text = 'satisfied'
            elif rating.rating >= RATING_LIMIT_OK:
                rating.rating_text = 'not_satisfied'
            elif rating.rating >= RATING_LIMIT_MIN:
                rating.rating_text = 'highly_dissatisfied'
            else:
                rating.rating_text = 'no_rating'

    @api.model
    def create(self, values):
        if values.get('res_model_id') and values.get('res_id'):
            values.update(self._find_parent_data(values))
        return super(Rating, self).create(values)

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

## File: models\rating_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import timedelta

from odoo import api, fields, models, tools
from odoo.addons.rating.models.rating import RATING_LIMIT_SATISFIED, RATING_LIMIT_OK, RATING_LIMIT_MIN
from odoo.osv import expression


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

    @api.depends('rating_ids.rating', 'rating_ids.consumed')
    def _compute_rating_percentage_satisfaction(self):
        # build domain and fetch data
        domain = [('parent_res_model', '=', self._name), ('parent_res_id', 'in', self.ids), ('rating', '>=', 1), ('consumed', '=', True)]
        if self._rating_satisfaction_days:
            domain += [('write_date', '>=', fields.Datetime.to_string(fields.datetime.now() - timedelta(days=self._rating_satisfaction_days)))]
        data = self.env['rating.rating'].read_group(domain, ['parent_res_id', 'rating'], ['parent_res_id', 'rating'], lazy=False)

        # get repartition of grades per parent id
        default_grades = {'great': 0, 'okay': 0, 'bad': 0}
        grades_per_parent = dict((parent_id, dict(default_grades)) for parent_id in self.ids)  # map: {parent_id: {'great': 0, 'bad': 0, 'ok': 0}}
        for item in data:
            parent_id = item['parent_res_id']
            rating = item['rating']
            if rating > RATING_LIMIT_OK:
                grades_per_parent[parent_id]['great'] += item['__count']
            elif rating > RATING_LIMIT_MIN:
                grades_per_parent[parent_id]['okay'] += item['__count']
            else:
                grades_per_parent[parent_id]['bad'] += item['__count']

        # compute percentage per parent
        for record in self:
            repartition = grades_per_parent.get(record.id, default_grades)
            record.rating_percentage_satisfaction = repartition['great'] * 100 / sum(repartition.values()) if sum(repartition.values()) else -1


class RatingMixin(models.AbstractModel):
    _name = 'rating.mixin'
    _description = "Rating Mixin"

    rating_ids = fields.One2many('rating.rating', 'res_id', string='Rating', groups='base.group_user', domain=lambda self: [('res_model', '=', self._name)], auto_join=True)
    rating_last_value = fields.Float('Rating Last Value', groups='base.group_user', compute='_compute_rating_last_value', compute_sudo=True, store=True)
    rating_last_feedback = fields.Text('Rating Last Feedback', groups='base.group_user', related='rating_ids.feedback')
    rating_last_image = fields.Binary('Rating Last Image', groups='base.group_user', related='rating_ids.rating_image')
    rating_count = fields.Integer('Rating count', compute="_compute_rating_stats", compute_sudo=True)
    rating_avg = fields.Float("Rating Average", compute='_compute_rating_stats', compute_sudo=True)

    @api.depends('rating_ids.rating', 'rating_ids.consumed')
    def _compute_rating_last_value(self):
        for record in self:
            ratings = self.env['rating.rating'].search([('res_model', '=', self._name), ('res_id', '=', record.id), ('consumed', '=', True)], limit=1)
            record.rating_last_value = ratings and ratings.rating or 0

    @api.depends('rating_ids.res_id', 'rating_ids.rating')
    def _compute_rating_stats(self):
        """ Compute avg and count in one query, as thoses fields will be used together most of the time. """
        domain = expression.AND([self._rating_domain(), [('rating', '>=', RATING_LIMIT_MIN)]])
        read_group_res = self.env['rating.rating'].read_group(domain, ['rating:avg'], groupby=['res_id'], lazy=False)  # force average on rating column
        mapping = {item['res_id']: {'rating_count': item['__count'], 'rating_avg': item['rating']} for item in read_group_res}
        for record in self:
            record.rating_count = mapping.get(record.id, {}).get('rating_count', 0)
            record.rating_avg = mapping.get(record.id, {}).get('rating_avg', 0)

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

    def rating_get_partner_id(self):
        if hasattr(self, 'partner_id') and self.partner_id:
            return self.partner_id
        return self.env['res.partner']

    def rating_get_rated_partner_id(self):
        if hasattr(self, 'user_id') and self.user_id.partner_id:
            return self.user_id.partner_id
        return self.env['res.partner']

    def rating_get_access_token(self, partner=None):
        """ Return access token linked to existing ratings, or create a new rating
        that will create the asked token. An explicit call to access rights is
        performed as sudo is used afterwards as this method could be used from
        different sources, notably templates. """
        self.check_access_rights('read')
        self.check_access_rule('read')
        if not partner:
            partner = self.rating_get_partner_id()
        rated_partner = self.rating_get_rated_partner_id()
        ratings = self.rating_ids.sudo().filtered(lambda x: x.partner_id.id == partner.id and not x.consumed)
        if not ratings:
            record_model_id = self.env['ir.model'].sudo().search([('model', '=', self._name)], limit=1).id
            rating = self.env['rating.rating'].sudo().create({
                'partner_id': partner.id,
                'rated_partner_id': rated_partner.id,
                'res_model_id': record_model_id,
                'res_id': self.id,
                'is_internal': False,
            })
        else:
            rating = ratings[0]
        return rating.access_token

    def rating_send_request(self, template, lang=False, subtype_id=False, force_send=True, composition_mode='comment', notif_layout=None):
        """ This method send rating request by email, using a template given
        in parameter.

         :param template: a mail.template record used to compute the message body;
         :param lang: optional lang; it can also be specified directly on the template
           itself in the lang field;
         :param subtype_id: optional subtype to use when creating the message; is
           a note by default to avoid spamming followers;
         :param force_send: whether to send the request directly or use the mail
           queue cron (preferred option);
         :param composition_mode: comment (message_post) or mass_mail (template.send_mail);
         :param notif_layout: layout used to encapsulate the content when sending email;
        """
        if lang:
            template = template.with_context(lang=lang)
        if subtype_id is False:
            subtype_id = self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note')
        if force_send:
            self = self.with_context(mail_notify_force_send=True)  # default value is True, should be set to false if not?
        for record in self:
            record.message_post_with_template(
                template.id,
                composition_mode=composition_mode,
                email_layout_xmlid=notif_layout if notif_layout is not None else 'mail.mail_notification_light',
                subtype_id=subtype_id
            )

    def rating_apply(self, rate, token=None, feedback=None, subtype_xmlid=None):
        """ Apply a rating given a token. If the current model inherits from
        mail.thread mixin, a message is posted on its chatter. User going through
        this method should have at least employee rights because of rating
        manipulation (either employee, either sudo-ed in public controllers after
        security check granting access).

        :param float rate : the rating value to apply
        :param string token : access token
        :param string feedback : additional feedback
        :param string subtype_xmlid : xml id of a valid mail.message.subtype

        :returns rating.rating record
        """
        rating = None
        if token:
            rating = self.env['rating.rating'].search([('access_token', '=', token)], limit=1)
        else:
            rating = self.env['rating.rating'].search([('res_model', '=', self._name), ('res_id', '=', self.ids[0])], limit=1)
        if rating:
            rating.write({'rating': rate, 'feedback': feedback, 'consumed': True})
            if hasattr(self, 'message_post'):
                feedback = tools.plaintext2html(feedback or '')
                self.message_post(
                    body="<img src='/rating/static/src/img/rating_%s.png' alt=':%s/5' style='width:18px;height:18px;float:left;margin-right: 5px;'/>%s"
                    % (rate, rate, feedback),
                    subtype_xmlid=subtype_xmlid or "mail.mt_comment",
                    author_id=rating.partner_id and rating.partner_id.id or None  # None will set the default author in mail_thread.py
                )
            if hasattr(self, 'stage_id') and self.stage_id and hasattr(self.stage_id, 'auto_validation_kanban_state') and self.stage_id.auto_validation_kanban_state:
                if rating.rating > 2:
                    self.write({'kanban_state': 'done'})
                else:
                    self.write({'kanban_state': 'blocked'})
        return rating

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
        data = self.env['rating.rating'].read_group(base_domain, ['rating'], ['rating', 'res_id'])
        # init dict with all posible rate value, except 0 (no value for the rating)
        values = dict.fromkeys(range(1, 6), 0)
        values.update((d['rating'], d['rating_count']) for d in data)
        # add other stats
        if add_stats:
            rating_number = sum(values.values())
            result = {
                'repartition': values,
                'avg': sum(float(key * values[key]) for key in values) / rating_number if rating_number > 0 else 0,
                'total': sum(it['rating_count'] for it in data),
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
            if key >= RATING_LIMIT_SATISFIED:
                res['great'] += data[key]
            elif key >= RATING_LIMIT_OK:
                res['okay'] += data[key]
            else:
                res['bad'] += data[key]
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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import rating
from . import rating_mixin
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

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_frontend" inherit_id="web.assets_frontend" name="Rating: Frontend Assets">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/rating/static/src/scss/rating.scss"/>
        </xpath>
    </template>

</odoo>

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
                <tree string="Rating" create="false" edit="false" sample="1">
                    <field name="res_name"/>
                    <field name="parent_res_name"/>
                    <field name="rated_partner_id"/>
                    <field name="partner_id"/>
                    <field name="rating_text" decoration-danger="rating_text == 'highly_dissatisfied'" decoration-warning="rating_text == 'not_satisfied'" decoration-success="rating_text == 'satisfied'"/>
                    <field name="feedback"/>
                    <field name="create_date"/>
                </tree>
            </field>
        </record>

        <record id="rating_rating_view_form" model="ir.ui.view">
            <field name="name">rating.rating.form</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <form string="Rating" create="false">
                    <sheet>
                        <group>
                            <group>
                                <field name="resource_ref" string="Document"/>
                                <field name="res_name" string="Document" invisible="1"/>
                                <field name="parent_ref" string="Parent Holder"/>
                                <field name="parent_res_name" string="Parent Holder" invisible="1"/>
                                <field name="rated_partner_id"/>
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
                            <div class="oe_kanban_global_click">
                                <div class="o_kanban_image">
                                    <field name="rating_image" widget="image"/>
                                </div>
                                <div class="oe_kanban_details">
                                    <strong>
                                        <field name="rated_partner_id" />
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
                                    </ul>
                                    <i t-if="record.feedback.raw_value" class="fa fa-comment float-right mt4" t-att-title="record.feedback.raw_value"  t-att-aria-label="record.feedback.raw_value" role="img"/>
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
                <pivot string="Rating Average" display_quantity="true" sample="1">
                    <field name="rated_partner_id" type="row"/>
                    <field name="create_date" type="col"/>
                    <field name="rating" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="rating_rating_view_graph" model="ir.ui.view">
           <field name="name">rating.rating.graph</field>
           <field name="model">rating.rating</field>
           <field name="arch" type="xml">
                <graph string="Rating Average" type="bar" sample="1">
                    <field name="create_date" type="row"/>
                    <field name="rating" type="measure"/>
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
                    <filter string="Satisfied" name="rating_happy" domain="[('rating_text', '=', 'satisfied')]"/>
                    <filter string="Not satisfied" name="rating_okay" domain="[('rating_text', '=', 'not_satisfied')]"/>
                    <filter string="Highly dissatisfied" name="rating_unhappy" domain="[('rating_text', '=', 'highly_dissatisfied')]"/>
                    <separator/>
                    <filter string="My Ratings" name="my_ratings" domain="[('rated_partner_id.user_ids', 'in', [uid])]"/>
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

        <record id="rating_rating_view" model="ir.actions.act_window">
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

        <!-- Add menu entry in Technical/Discuss -->
        <menuitem name="Ratings"
            id="rating_rating_menu_technical"
            parent="mail.mail_menu_technical"
            action="rating_rating_view"
            sequence="30"/>

</odoo>

```

## File: views\rating_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
        <!-- External page : thanks message -->
        <template id="rating_external_page_view" name="Rating Page Done">
            <t t-call="web.frontend_layout">
                <div class="container pt-5">
                    <div class="text-center mt-5 pt-5">
                        <h3 class="d-inline">Thank you, we appreciate your feedback!</h3>
                    </div>
                    <div class="text-center">
                        <a role="button" t-att-href="web_base_url" class="btn btn-primary my-5">Go back to the Homepage</a>
                    </div>
                </div>
            </t>
        </template>

        <!-- External page: rate and submit feedback -->
        <template id="rating_external_page_submit" name="Rating Page Submit">
            <t t-call="web.frontend_layout">
                <div class="container card mb-5 mt-4 o_rating_page_submit">
                    <div class="card-body">
                        <div class="row text-center justify-content-center">
                            <h1 class="col-12 mt-5">Thank you for rating our services!</h1>
                            <form t-attf-action="/rate/#{token}/submit_feedback" method="post">
                                <div class="btn-group btn-group-toggle row" data-toggle="buttons">
                                    <t t-foreach="rate_names" t-as="rate_name">
                                        <label t-attf-class="col-xs-12 btn o_rating_label shadow-none {{rate == rate_name and 'active' or ''}}">
                                            <input type="radio" name="rate" t-attf-id="rate_{{rate_name}}" t-att-value="rate_name" t-att-checked="rate == rate_name"/>
                                            <a class="o_rating" href="#">
                                                <img t-attf-src='/rating/static/src/img/rating_#{rate_name}.png' t-att-alt="rate_name_value" t-att-title="rate_name_value"/>
                                            </a>
                                        </label>
                                    </t>
                                </div>
                                <div class="clearfix"></div>
                                <p class="mt-5">
                                    Feel free to write a feedback on your experience:
                                </p>
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                                <textarea class="form-control" name="feedback" rows="8" t-att-value="rating.feedback"></textarea>
                                <button type="submit" class="btn btn-primary" style="margin-top:8px;">Send Feedback</button>
                            </form>
                        </div>
                    </div>
                </div>
            </t>
        </template>
</odoo>

```

