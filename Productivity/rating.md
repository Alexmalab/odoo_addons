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
        'web.assets_backend': [
            "rating/static/src/core/common/**/*",
            "rating/static/src/core/web/**/*",
        ],
        'web.assets_frontend': [
            'rating/static/src/scss/rating_templates.scss',
        ],
        'web.assets_unit_tests': [
            'rating/static/tests/**/*',
            ('remove', 'rating/static/tests/helpers/**/*'),
        ],
        'web.tests_assets': [
            'rating/static/tests/helpers/**/*',
        ],
        "mail.assets_public": [
            "rating/static/src/core/common/**/*",
        ],
        "portal.assets_chatter": [
            "rating/static/src/core/common/**/*",
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
        rating, record_sudo = self._get_rating_and_record(token)

        if not request.env.user._is_public() and \
                request.env.user.partner_id.commercial_partner_id != rating.partner_id.commercial_partner_id:
            return request.render('rating.rating_external_page_invalid_partner', {
                'model_name': request.env['ir.model']._get(rating.res_model).display_name,
                'name': record_sudo.display_name,
                'web_base_url': rating.get_base_url(),
            })

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

## File: controllers\thread.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.mail.controllers import thread


class ThreadController(thread.ThreadController):

    def _get_allowed_message_post_params(self):
        return super()._get_allowed_message_post_params() | {"rating_value"}

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import thread

```

## File: models\mail_message.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.addons.mail.tools.discuss import Store


class MailMessage(models.Model):
    _inherit = 'mail.message'

    rating_ids = fields.One2many("rating.rating", "message_id", string="Related ratings")
    rating_id = fields.Many2one("rating.rating", compute="_compute_rating_id")
    rating_value = fields.Float(
        'Rating Value', compute='_compute_rating_value', compute_sudo=True,
        store=False, search='_search_rating_value')

    @api.depends("rating_ids.consumed")
    def _compute_rating_id(self):
        for message in self:
            message.rating_id = message.rating_ids.filtered(lambda rating: rating.consumed).sorted(
                "create_date", reverse=True
            )[:1]

    @api.depends('rating_ids', 'rating_ids.rating')
    def _compute_rating_value(self):
        for message in self:
            message.rating_value = message.rating_id.rating if message.rating_id else 0.0

    def _search_rating_value(self, operator, operand):
        ratings = self.env['rating.rating'].sudo().search([
            ('rating', operator, operand),
            ('message_id', '!=', False),
            ("consumed", "=", True),
        ])
        return [('id', 'in', ratings.mapped('message_id').ids)]

    def _to_store(self, store: Store, /, *, fields=None, **kwargs):
        super()._to_store(store, fields=fields, **kwargs)
        if fields is None:
            fields = ["rating_id", "record_rating"]
        if "rating_id" in fields:
            for message in self:
                # sudo: mail.message - guest and portal user can receive rating of accessible message
                store.add(message, {"rating_id": Store.one(message.sudo().rating_id)})
        if "record_rating" in fields:
            for records in self._records_by_model_name().values():
                if issubclass(self.pool[records._name], self.pool["rating.mixin"]):
                    store.add(records, fields=["rating_avg", "rating_count"], as_thread=True)

```

## File: models\mail_thread.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import markupsafe

from odoo import _, api, fields, models, tools


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    rating_ids = fields.One2many('rating.rating', 'res_id', string='Ratings', groups='base.group_user',
                                 domain=lambda self: [('res_model', '=', self._name)], auto_join=True)

    # MAIL OVERRIDES
    # --------------------------------------------------

    def unlink(self):
        """ When removing a record, its rating should be deleted too. """
        record_ids = self.ids
        result = super().unlink()
        self.env['rating.rating'].sudo().search([('res_model', '=', self._name), ('res_id', 'in', record_ids)]).unlink()
        return result

    def _get_message_create_ignore_field_names(self):
        return super()._get_message_create_ignore_field_names() | {"rating_id"}

    # RATING CONFIGURATION
    # --------------------------------------------------

    def _rating_apply_get_default_subtype_id(self):
        return self.env['ir.model.data']._xmlid_to_res_id("mail.mt_comment")

    def _rating_get_operator(self):
        """ Return the operator (partner) that is the person who is rated.

        :return record: res.partner singleton
        """
        if 'user_id' in self and self.user_id.partner_id:
            return self.user_id.partner_id
        return self.env['res.partner']

    def _rating_get_partner(self):
        """ Return the customer (partner) that performs the rating.

        :return record: res.partner singleton
        """
        if 'partner_id' in self and self.partner_id:
            return self.partner_id
        return self.env['res.partner']

    # RATING SUPPORT
    # --------------------------------------------------

    def _rating_get_access_token(self, partner=None):
        """ Return access token linked to existing ratings, or create a new rating
        that will create the asked token. An explicit call to access rights is
        performed as sudo is used afterwards as this method could be used from
        different sources, notably templates. """
        self.check_access('read')
        if not partner:
            partner = self._rating_get_partner()
        rated_partner = self._rating_get_operator()
        rating = next(
            (r for r in self.rating_ids.sudo()
             if r.partner_id.id == partner.id and not r.consumed),
            None)
        if not rating:
            rating = self.env['rating.rating'].sudo().create({
                'partner_id': partner.id,
                'rated_partner_id': rated_partner.id,
                'res_model_id': self.env['ir.model']._get_id(self._name),
                'res_id': self.id,
                'is_internal': False,
            })
        return rating.access_token

    # EXPOSED API
    # --------------------------------------------------

    def rating_send_request(self, template, lang=False, force_send=True):
        """ This method send rating request by email, using a template given in parameter.

         :param record template: a mail.template record used to compute the message body;
         :param str lang: optional lang; it can also be specified directly on the template
           itself in the lang field;
         :param bool force_send: whether to send the request directly or use the mail
           queue cron (preferred option);
        """
        if lang:
            template = template.with_context(lang=lang)
        self.with_context(mail_notify_force_send=force_send).message_post_with_source(
            template,
            email_layout_xmlid='mail.mail_notification_light',
            force_send=force_send,
            subtype_xmlid='mail.mt_note',
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
            raise ValueError(_('Wrong rating value. A rate should be between 0 and 5 (received %d).', rate))
        if token:
            rating = self.env['rating.rating'].search([('access_token', '=', token)], limit=1)
        if not rating:
            raise ValueError(_('Invalid token or rating.'))

        rating.write({'rating': rate, 'feedback': feedback, 'consumed': True})
        if isinstance(self, self.env.registry['mail.thread']):
            if subtype_xmlid is None:
                subtype_id = self._rating_apply_get_default_subtype_id()
            else:
                subtype_id = False
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
                    author_id=rating.partner_id.id or None,  # None will set the default author in mail/mail_thread.py
                    body=rating_body,
                    rating_id=rating.id,
                    scheduled_date=scheduled_datetime,
                    subtype_id=subtype_id,
                    subtype_xmlid=subtype_xmlid,
                )
        return rating

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        rating_id = kwargs.pop('rating_id', False)
        rating_value = kwargs.pop('rating_value', False)
        # create rating.rating record linked to given rating_value. Using sudo as portal users may have
        # rights to create messages and therefore ratings (security should be checked beforehand)
        if rating_value:
            rating_vals = {
                'rating': float(rating_value) if rating_value is not None else False,
                'feedback': tools.html2plaintext(kwargs.get('body', '')),
                'res_model_id': self.env['ir.model']._get_id(self._name),
                'res_id': self.id,
                'consumed': True,
                'partner_id': self.env.user.partner_id.id,
            }
            rating_id = self.env["rating.rating"].sudo().create(rating_vals).id
        if rating_id:
            kwargs["rating_id"] = rating_id
        return super().message_post(**kwargs)

    def _message_post_after_hook(self, message, msg_values):
        """Override to link rating to message as sudo. This is done in
        _message_post_after_hook to be before _notify_thread."""
        # sudo: rating.rating - can link rating to message from same author and thread
        rating = self.env["rating.rating"].browse(msg_values.get("rating_id")).sudo()
        same_author = rating.partner_id and rating.partner_id == message.author_id
        if same_author and rating.res_model == message.model and rating.res_id == message.res_id:
            rating.message_id = message.id
        super()._message_post_after_hook(message, msg_values)

```

## File: models\rating.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import uuid

from odoo import api, fields, models
from odoo.addons.mail.tools.discuss import Store
from odoo.addons.rating.models import rating_data
from odoo.tools.misc import file_open


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
    rating = fields.Float(string="Rating Value", aggregator="avg", default=0)
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
            name = self.env[rating.res_model].sudo().browse(rating.res_id).display_name
            rating.res_name = name or f'{rating.res_model}/{rating.res_id}'

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
                name = self.env[rating.parent_res_model].sudo().browse(rating.parent_res_id).display_name
                name = name or f'{rating.parent_res_model}/{rating.parent_res_id}'
            rating.parent_res_name = name

    def _get_rating_image_filename(self):
        self.ensure_one()
        return 'rating_%s.png' % rating_data._rating_to_threshold(self.rating)

    @api.depends('rating')
    def _compute_rating_image(self):
        self.rating_image_url = False
        self.rating_image = False
        for rating in self:
            image_path = f'rating/static/src/img/{rating._get_rating_image_filename()}'
            rating.rating_image_url = f'/{image_path}'
            try:
                rating.rating_image = base64.b64encode(
                    file_open(image_path, 'rb', filter_ext=('.png',)).read())
            except (IOError, OSError, FileNotFoundError):
                rating.rating_image = False

    @api.depends('rating')
    def _compute_rating_text(self):
        for rating in self:
            rating.rating_text = rating_data._rating_to_text(rating.rating)

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

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

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

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

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _classify_by_model(self):
        """ To ease batch computation of various ratings related methods they
        are classified by model. Ratings not linked to a valid record through
        res_model / res_id are ignored.

        :return dict: for each model having at least one rating in self, have
          a sub-dict containing
            * ratings: ratings related to that model;
            * record IDs: records linked to the ratings of that model, in same
              order;
        """
        data_by_model = {}
        for rating in self.filtered(lambda act: act.res_model and act.res_id):
            if rating.res_model not in data_by_model:
                data_by_model[rating.res_model] = {
                    'ratings': self.env['rating.rating'],
                    'record_ids': [],
                }
            data_by_model[rating.res_model]['ratings'] += rating
            data_by_model[rating.res_model]['record_ids'].append(rating.res_id)
        return data_by_model

    def _to_store(self, store: Store, /, *, fields=None):
        if fields is None:
            fields = ["rating", "rating_image_url", "rating_text"]
        store.add(self._name, self._read_format(fields, load=False))

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

from odoo import api, fields, models
from odoo.addons.rating.models import rating_data
from odoo.osv import expression
from odoo.tools.float_utils import float_compare, float_round


class RatingMixin(models.AbstractModel):
    """This mixin adds rating statistics to mail.thread that already support ratings."""
    _name = 'rating.mixin'
    _description = "Rating Mixin"
    _inherit = 'mail.thread'

    rating_last_value = fields.Float('Rating Last Value', groups='base.group_user', compute='_compute_rating_last_value', compute_sudo=True, store=True, aggregator="avg")
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
        read_group_res = self.env['rating.rating']._read_group(domain, ['res_id'], aggregates=['__count', 'rating:avg'])  # force average on rating column
        mapping = {res_id: {'rating_count': count, 'rating_avg': rating_avg} for res_id, count, rating_avg in read_group_res}
        for record in self:
            record.rating_count = mapping.get(record.id, {}).get('rating_count', 0)
            record.rating_avg = mapping.get(record.id, {}).get('rating_avg', 0)

    def _search_rating_avg(self, operator, value):
        if operator not in rating_data.OPERATOR_MAPPING:
            raise NotImplementedError('This operator %s is not supported in this search method.' % operator)
        rating_read_group = self.env['rating.rating'].sudo()._read_group(
            [('res_model', '=', self._name), ('consumed', '=', True), ('rating', '>=', rating_data.RATING_LIMIT_MIN)],
            ['res_id'], ['rating:avg'])
        res_ids = [
            res_id
            for res_id, rating_avg in rating_read_group
            if rating_data.OPERATOR_MAPPING[operator](float_compare(rating_avg, value, 2), 0)
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
        read_group_res = self.env['rating.rating']._read_group(domain, ['res_id', 'rating'], aggregates=['__count'])
        default_grades = {'great': 0, 'okay': 0, 'bad': 0}
        grades_per_record = {record_id: default_grades.copy() for record_id in self.ids}

        for record_id, rating, count in read_group_res:
            grade = rating_data._rating_to_grade(rating)
            grades_per_record[record_id][grade] += count

        for record in self:
            grade_repartition = grades_per_record.get(record.id, default_grades)
            grade_count = sum(grade_repartition.values())
            record.rating_percentage_satisfaction = grade_repartition['great'] * 100 / grade_count if grade_count else -1

    def write(self, values):
        """ If the rated ressource name is modified, we should update the rating res_name too.
            If the rated ressource parent is changed we should update the parent_res_id too"""
        result = super(RatingMixin, self).write(values)
        for record in self:
            if record._rec_name in values:  # set the res_name of ratings to be recomputed
                res_name_field = self.env['rating.rating']._fields['res_name']
                self.env.add_to_compute(res_name_field, record.rating_ids)
            if record._rating_get_parent_field_name() in values:
                record.rating_ids.sudo().write({'parent_res_id': record[record._rating_get_parent_field_name()].id})

        return result

    def _rating_get_parent_field_name(self):
        """Return the parent relation field name. Should return a Many2One"""
        return None

    def _rating_domain(self):
        """ Returns a normalized domain on rating.rating to select the records to
            include in count, avg, ... computation of current model.
        """
        return ['&', '&', ('res_model', '=', self._name), ('res_id', 'in', self.ids), ('consumed', '=', True)]

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
        rg_data = self.env['rating.rating']._read_group(base_domain, ['rating'], ['__count'])
        # init dict with all possible rate value, except 0 (no value for the rating)
        values = dict.fromkeys(range(1, 6), 0)
        for rating, count in rg_data:
            rating_val_round = float_round(rating, precision_digits=1)
            values[rating_val_round] = values.get(rating_val_round, 0) + count
        # add other stats
        if add_stats:
            rating_number = sum(values.values())
            return {
                'repartition': values,
                'avg': sum(float(key * values[key]) for key in values) / rating_number if rating_number > 0 else 0,
                'total': sum(count for __, count in rg_data),
            }
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
        data = self.env['rating.rating']._read_group(domain, ['parent_res_id', 'rating'], ['__count'])

        # get repartition of grades per parent id
        default_grades = {'great': 0, 'okay': 0, 'bad': 0}
        grades_per_parent = dict((parent_id, dict(default_grades)) for parent_id in self.ids)  # map: {parent_id: {'great': 0, 'bad': 0, 'ok': 0}}
        rating_scores_per_parent = defaultdict(int)  # contains the total of the rating values per record
        for parent_id, rating, count in data:
            grade = rating_data._rating_to_grade(rating)
            grades_per_parent[parent_id][grade] += count
            rating_scores_per_parent[parent_id] += rating * count

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
        rating_read_group = self.env['rating.rating'].sudo()._read_group(domain, ['parent_res_id'], ['rating:avg'])
        parent_res_ids = [
            parent_res_id
            for parent_res_id, rating_avg in rating_read_group
            if rating_data.OPERATOR_MAPPING[operator](float_compare(rating_avg, value, 2), 0)
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

## File: static\src\core\common\message_model_patch.js

```javascript
import { Message } from "@mail/core/common/message_model";
import { Record } from "@mail/core/common/record";

import { patch } from "@web/core/utils/patch";

patch(Message.prototype, {
    setup() {
        super.setup(...arguments);
        this.rating_id = Record.one("rating.rating");
    },
});

```

## File: static\src\core\common\rating_model.js

```javascript
import { Record } from "@mail/core/common/record";

export class Rating extends Record {
    static _name = "rating.rating";
    static id = "id";

    /** @type {number} */
    id;
    /** @type {number} */
    rating;
    /** @type {string} */
    rating_image_url;
    /** @type {string} */
    rating_text;
}
Rating.register();

```

## File: static\src\core\web\messaging_menu_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.MessagingMenu.content" t-inherit-mode="extension">
        <xpath expr="//t[@name='threads']/NotificationItem" position="attributes">
            <attribute name="rating">message?.rating_id</attribute>
            <attribute name="onClick">(isMarkAsRead) => message?.rating_id and !isMarkAsRead ? this.openThread(thread) : this.onClickThread(isMarkAsRead, thread)</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\web\notification_item_patch.js

```javascript
/** @odoo-module */

import { NotificationItem } from "@mail/core/public_web/notification_item";

NotificationItem.props = [...NotificationItem.props, "rating?"];

```

## File: static\src\core\web\notification_item_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.NotificationItem" t-inherit-mode="extension">
        <xpath expr="//*[@name='notificationBody']" position="replace">
            <t t-if="props.rating">
                <span>Rating:</span>
                <img class="o-rating-preview-image ms-2" t-att-src="props.rating.rating_image_url" t-att-alt="props.rating.rating_text"/>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\img\503.svg

```svg
<svg version="1.0" xmlns="http://www.w3.org/2000/svg"
 width="310.000000pt" height="120.000000pt" viewBox="0 0 310.000000 120.000000"
 preserveAspectRatio="xMidYMid meet">

    <g transform="translate(0.000000,120.000000) scale(0.100000,-0.100000)"
    fill="#000000" stroke="none">
        <path fill="#7C6576" d="M1420 1177 c-170 -51 -284 -160 -344 -326 -18 -51 -20 -87 -24 -444
        -3 -372 -3 -389 15 -385 10 2 32 25 49 51 48 73 64 87 100 87 34 0 42 -8 105
        -97 23 -34 35 -43 59 -43 24 0 36 9 59 43 63 90 71 97 106 97 39 0 53 -12 100
        -86 29 -45 40 -54 64 -54 30 0 41 10 102 97 23 34 35 43 58 43 46 0 58 -8 92
        -63 33 -52 56 -77 72 -77 4 0 7 173 5 393 -4 371 -5 395 -25 449 -63 167 -194
        282 -367 322 -89 21 -136 19 -226 -7z m100 -392 c25 -13 60 -60 60 -79 0 -3
        -13 -3 -30 0 -35 8 -60 -9 -60 -40 0 -28 18 -46 47 -46 34 0 29 -27 -9 -51
        -49 -30 -120 -22 -161 20 -28 28 -32 38 -32 87 0 51 3 58 37 90 30 27 46 34
        78 34 23 0 54 -7 70 -15z m340 0 c31 -16 60 -51 60 -72 0 -9 -9 -11 -30 -7
        -35 8 -60 -9 -60 -40 0 -30 18 -46 53 -46 36 0 32 -17 -13 -47 -106 -72 -243
        44 -185 157 34 65 109 89 175 55z"/>
        <path d="M2500 1159 c-95 -15 -270 -115 -270 -155 0 -23 27 -16 79 21 26 19
        75 47 107 62 53 25 70 28 169 28 94 0 117 -3 162 -24 90 -40 153 -127 153
        -212 0 -58 -30 -126 -71 -164 -63 -57 -99 -68 -236 -73 -110 -4 -123 -6 -123
        -22 0 -16 13 -18 133 -22 141 -4 201 -21 255 -71 59 -55 87 -173 62 -262 -14
        -51 -76 -127 -124 -152 -151 -78 -373 -47 -515 71 -48 40 -59 43 -68 21 -13
        -35 134 -132 242 -160 76 -19 204 -19 280 0 85 22 161 77 199 145 29 51 31 62
        31 150 0 82 -4 102 -25 144 -30 60 -79 101 -144 122 l-49 17 51 24 c227 107
        172 437 -83 501 -62 16 -155 20 -215 11z"/>
        <path d="M167 1153 c-11 -10 -8 -511 2 -518 6 -3 41 10 78 29 86 44 136 55
        244 56 156 0 249 -47 304 -154 38 -76 41 -174 6 -251 -42 -94 -166 -183 -307
        -219 -70 -18 -209 -36 -276 -36 -41 0 -48 -3 -48 -20 0 -18 7 -20 73 -20 144
        0 334 45 435 102 72 42 119 87 153 150 32 59 34 69 34 158 -1 117 -24 178 -94
        243 -71 66 -142 90 -271 90 -106 1 -171 -12 -240 -48 -59 -30 -60 -26 -60 184
        0 104 3 196 6 205 5 14 42 16 305 16 292 0 299 0 299 20 0 20 -7 20 -318 20
        -175 0 -322 -3 -325 -7z"/>
    </g>
</svg>

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
            <field name="name">rating.rating.list</field>
            <field name="model">rating.rating</field>
            <field name="arch" type="xml">
                <list string="Ratings" create="false" edit="false" sample="1">
                    <field name="create_date"/>
                    <field name="rated_partner_id" optional="show"/>
                    <field name="partner_id" optional="show"/>
                    <field name="parent_res_name" optional="show"/>
                    <field name="res_name" optional="show"/>
                    <field name="feedback" optional="hide"/>
                    <field name="rating_text" decoration-danger="rating_text == 'ko'" decoration-warning="rating_text == 'ok'" decoration-success="rating_text == 'top'" class="fw-bold" widget="badge"/>
                </list>
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
                            </group>
                        </group>
                        <group class="mw-100" invisible="not feedback">
                            <field name="feedback"/>
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
                    <templates>
                        <t t-name="card" class="row g-0">
                            <aside class="col-4 my-auto align-self-center">
                                <field name="rating_image" widget="image" class="bg-view ms-3" />
                            </aside>
                            <main class="col ps-2">
                                <field name="rated_partner_name" class="fw-bolder"/>
                                <div t-if="record.partner_id.value" class="text-truncate">
                                    by
                                    <span t-att-title="record.partner_id.value">
                                        <field name="partner_id" />
                                    </span>
                                </div>
                                <span class="text-truncate">
                                    for
                                    <a type="object" name="action_open_rated_object" t-att-title="record.res_name.raw_value">
                                        <field name="res_name" />
                                    </a>
                                </span>
                                <div>
                                    on <field name="create_date" />
                                </div>
                                <span class="text-truncate" t-att-title="record.feedback.raw_value">
                                    <field name="feedback"/>
                                </span>
                            </main>
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
                    <templates>
                        <t t-name="card" class="row g-0">
                            <t t-set="val_stars" t-value="Math.round(record.rating.raw_value * 10) / 10"/>
                            <t t-set="val_integer" t-value="Math.floor(val_stars)"/>
                            <t t-set="val_decimal" t-value="val_stars - val_integer"/>
                            <t t-set="empty_star" t-value="5 - (val_integer + Math.ceil(val_decimal))"/>
                            <aside class="col-3 me-2">
                                <div class="display-3 fw-bold text-center text-primary mb-2" t-esc="val_stars"/>
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
                            </aside>
                            <main class="col">
                                <field name="partner_id" class="fw-bold fs-5"/>
                                <div class="mt0">
                                    <i class="fa fa-folder me-2" aria-label="Open folder"></i>
                                    <a type="object" name="action_open_rated_object" t-att-title="record.res_name.raw_value">
                                        <field name="res_name" />
                                    </a>
                                    <div><i class="fa fa-clock-o me-2" aria-label="Create date"/> <field name="create_date" /></div>
                                    <field name="feedback" class="mt-2"/>
                                </div>
                            </main>
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
                    <field name="rating" type="measure" string="Rating (/5)"/>
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
                    <field name="rating" type="measure" string="Rating (/5)"/>
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
                    <filter name="filter_create_date" date="create_date">
                        <filter name="create_date_last_7_days" string="Last 7 Days" domain="[('create_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=7), datetime.time(23, 59, 59)).to_utc())]"/>
                        <filter name="create_date_last_30_days" string="Last 30 Days" domain="[('create_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=30), datetime.time(23, 59, 59)).to_utc())]"/>
                        <filter name="create_date_last_365_days" string="Last 365 Days" domain="[('create_date', '&gt;', datetime.datetime.combine(context_today() - datetime.timedelta(days=365), datetime.time(23, 59, 59)).to_utc())]"/>
                    </filter>
                    <group expand="0" string="Group By">
                        <filter string="Rated Operator" name="responsible" context="{'group_by':'rated_partner_id'}"/>
                        <filter string="Customer" name="customer" context="{'group_by':'partner_id'}"/>
                        <filter string="Rating" name="rating_text" context="{'group_by':'rating_text'}"/>
                        <filter string="Resource" name="resource" context="{'group_by':'res_name'}"/>
                        <filter string="Submitted on" name="month" context="{'group_by': 'create_date:month'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="rating_rating_action" model="ir.actions.act_window">
            <field name="name">Ratings</field>
            <field name="res_model">rating.rating</field>
            <field name="view_mode">kanban,list,graph,pivot,form</field>
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
        <template id="rating_external_page_view" name="Thanks for your Feedback">
            <t t-call="web.frontend_layout">
                <div class="o_rating_page_thank_you d-flex flex-column justify-content-center">
                    <div class="container py-5">
                        <div class="oe_structure" id="oe_structure_rating_header_thanks_you"/>
                        <h3 class="text-center mt-4 pb-2">Thank you for your feedback!</h3>
                        <a t-att-href="web_base_url" class="d-block text-decoration-none text-center mb-4">
                            <i class="fa fa-arrow-left me-1"/> Back to the Homepage
                        </a>
                        <div class="oe_structure" id="oe_structure_rating_footer_thanks_you"/>
                    </div>
                </div>
            </t>
        </template>

        <!-- External page: Invalid partner rating -->
        <template id="rating_external_page_invalid_partner" name="Not allows to rating">
            <t t-call="web.frontend_layout">
                <section class="mt128">
                    <div class="container">
                        <div class="row" >
                            <div class="col-lg-4">
                                <img class="img img-fluid mx-auto" src="/rating/static/src/img/503.svg" alt=""/>
                            </div>
                            <div class="col-lg-8 text-lg-start">
                                <h1>You cannot rate this <t t-out="model_name"/></h1>
                                <p>
                                    Only the customer of "<t t-out="name"/>" or someone from the same company can give it a rating.
                                </p>
                            </div>
                            <hr class="my-5 opacity-100"/>
                            <div class="mb128 col-lg-12">
                                <ul class="list-inline text-center">
                                    <li>
                                        <a t-att-href="web_base_url" class="btn btn-primary">Home</a>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </section>
            </t>
        </template>

        <!-- External page: rate and submit feedback -->
        <template id="rating_external_page_submit" name="Rate our Services">
            <t t-call="web.frontend_layout">
                <div class="container mb-5 mt-4 o_rating_page_submit">
                    <div class="oe_structure" id="oe_structure_rate_submit_header"/>
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
                                Feel free to share feedback on your experience:
                            </p>
                            <input type="hidden" name="csrf_token"
                                   t-att-value="request.csrf_token()"/>
                            <textarea class="form-control" name="feedback" rows="8"
                                      t-att-value="rating.feedback"></textarea>
                            <button type="submit" class="btn btn-primary mt-4"
                                    style="margin-top:8px;">Send Feedback</button>
                        </form>
                        <div class="oe_structure" id="oe_structure_rate_submit_footer"/>
                    </div>
                </div>
            </t>
        </template>
</odoo>

```

