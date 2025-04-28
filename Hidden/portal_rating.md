# Odoo Module: portal_rating

Category: Hidden

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Portal Rating',
    'category': 'Hidden',
    'version': '1.0',
    'description': """
Bridge module adding rating capabilities on portal. It includes notably
inclusion of rating directly within the customer portal discuss widget.
        """,
    'depends': [
        'portal',
        'rating',
    ],
    'data': [
        'views/rating_rating_views.xml',
        'views/portal_templates.xml',
        'views/rating_templates.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'portal_rating/static/src/scss/portal_rating.scss',
            'portal_rating/static/src/js/portal_chatter.js',
            'portal_rating/static/src/xml/portal_chatter.xml',
            'portal_rating/static/src/js/portal_composer.js',
            'portal_rating/static/src/js/portal_rating_composer.js',
            'portal_rating/static/src/xml/portal_rating_composer.xml',
            'portal_rating/static/src/xml/portal_tools.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal_chatter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.osv import expression

from odoo.addons.portal.controllers import mail


class PortalChatter(mail.PortalChatter):

    def _portal_post_filter_params(self):
        fields = super(PortalChatter, self)._portal_post_filter_params()
        fields += ['rating_value', 'rating_feedback']
        return fields

    @http.route()
    def portal_chatter_post(self, res_model, res_id, message, attachment_ids='', attachment_tokens='', **kwargs):
        if kwargs.get('rating_value'):
            kwargs['rating_feedback'] = kwargs.pop('rating_feedback', message)
        return super(PortalChatter, self).portal_chatter_post(res_model, res_id, message, attachment_ids=attachment_ids, attachment_tokens=attachment_tokens, **kwargs)

    def _setup_portal_message_fetch_extra_domain(self, data):
        domains = [super()._setup_portal_message_fetch_extra_domain(data)]
        if data.get('rating_value', False) is not False:
            domains.append([('rating_value', '=', float(data['rating_value']))])
        return expression.AND(domains)

```

## File: controllers\portal_rating.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request


class PortalRating(http.Controller):

    @http.route(['/website/rating/comment'], type='json', auth="user", methods=['POST'], website=True)
    def publish_rating_comment(self, rating_id, publisher_comment):
        rating = request.env['rating.rating'].search_fetch(
            [('id', '=', int(rating_id))],
            ['publisher_comment', 'publisher_id', 'publisher_datetime'],
        )
        if not rating:
            return {'error': _('Invalid rating')}
        rating.write({'publisher_comment': publisher_comment})
        # return to the front-end the created/updated publisher comment
        return request.env['mail.message']._portal_message_format_rating(
            rating.read(['publisher_comment', 'publisher_id', 'publisher_datetime'])[0]
        )

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal_chatter
from . import portal_rating

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(IrHttp, cls)._get_translation_frontend_modules_name()
        return mods + ['portal_rating']

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import format_datetime


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def _portal_get_default_format_properties_names(self, options=None):
        """ Add request for rating information

        :param dict options: supports 'rating_include' option allowing to
          conditionally include rating information;
        """
        properties_names = super()._portal_get_default_format_properties_names()
        if options and options.get('rating_include'):
            properties_names |= {
                'rating',
                'rating_value',
            }
        return properties_names

    def _portal_message_format(self, properties_names, options=None):
        """ If requested, add rating information to returned formatted values.

        Note: rating information combine both statistics (see 'rating_get_stats'
        if available on model) and rating / publication information. """
        vals_list = super()._portal_message_format(properties_names, options=options)
        if not 'rating' in properties_names:
            return vals_list

        related_rating = self.env['rating.rating'].sudo().search_read(
            [('message_id', 'in', self.ids)],
            ["id", "publisher_comment", "publisher_id", "publisher_datetime", "message_id"]
        )
        message_to_rating = {
            rating['message_id'][0]: self._portal_message_format_rating(rating)
            for rating in related_rating
        }

        rating_stats_by_model = {
            model: {
                record_sudo.id: record_sudo.rating_get_stats()
                for record_sudo in self.env[model].browse({res_id for res_id in model_messages.mapped('res_id') if res_id}).sudo()
            }
            for model, model_messages in self.grouped('model').items()
            if model
        }

        for message, values in zip(self, vals_list):
            values["rating"] = message_to_rating.get(message.id, {})
            if rating_stats := rating_stats_by_model.get(message.model, {}).get(message.res_id):
                 values["rating_stats"] = rating_stats

        return vals_list

    def _portal_message_format_rating(self, rating_values):
        """ From 'rating_values' get an updated version formatted for frontend
        display.

        :param dict rating_values: values coming from reading ratings
          in database;

        :return dict: updated rating_values
        """
        publisher_id, publisher_name = rating_values['publisher_id'] or [False, '']
        rating_values['publisher_avatar'] = f'/web/image/res.partner/{publisher_id}/avatar_128/50x50' if publisher_id else ''
        rating_values['publisher_comment'] = rating_values['publisher_comment'] or ''
        rating_values['publisher_datetime'] = format_datetime(self.env, rating_values['publisher_datetime'])
        rating_values['publisher_id'] = publisher_id
        rating_values['publisher_name'] = publisher_name
        return rating_values

```

## File: models\rating_rating.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, exceptions, _


class Rating(models.Model):
    _inherit = 'rating.rating'

    # Adding information for comment a rating message
    publisher_comment = fields.Text("Publisher comment")
    publisher_id = fields.Many2one('res.partner', 'Commented by',
                                   ondelete='set null', readonly=True,
                                   index='btree_not_null')
    publisher_datetime = fields.Datetime("Commented on", readonly=True)

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            self._synchronize_publisher_values(values)
        ratings = super().create(values_list)
        if any(rating.publisher_comment for rating in ratings):
            ratings._check_synchronize_publisher_values()
        return ratings

    def write(self, values):
        self._synchronize_publisher_values(values)
        return super().write(values)

    def _check_synchronize_publisher_values(self):
        """ Either current user is a member of website restricted editor group
        (done here by fetching the group record then using has_group, as it may
        not be defined and we do not want to make a complete bridge module just
        for that). Either write access on document is granted. """
        editor_group = self.env['ir.model.data']._xmlid_to_res_id('website.group_website_restricted_editor')
        if editor_group and self.env.user.has_group('website.group_website_restricted_editor'):
            return
        for model, model_data in self._classify_by_model().items():
            records = self.env[model].browse(model_data['record_ids'])
            try:
                records.check_access_rights('write')
                records.check_access_rule('write')
            except exceptions.AccessError as e:
                raise exceptions.AccessError(
                    _("Updating rating comment require write access on related record")
                ) from e

    def _synchronize_publisher_values(self, values):
        """ Force publisher partner and date if not given in order to have
        coherent values. Those fields are readonly as they are not meant
        to be modified manually, behaving like a tracking. """
        if values.get('publisher_comment'):
            self._check_synchronize_publisher_values()
            if not values.get('publisher_datetime'):
                values['publisher_datetime'] = fields.Datetime.now()
            if not values.get('publisher_id'):
                values['publisher_id'] = self.env.user.partner_id.id
        return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import mail_message
from . import rating_rating

```

## File: static\src\js\portal_chatter.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import PortalChatter from "@portal/js/portal_chatter";
import { roundPrecision } from "@web/core/utils/numbers";
import { renderToElement } from "@web/core/utils/render";

/**
 * PortalChatter
 *
 * Extends Frontend Chatter to handle rating
 */
PortalChatter.include({
    events: Object.assign({}, PortalChatter.prototype.events, {
        // star based control
        'click .o_website_rating_table_row': '_onClickStarDomain',
        'click .o_website_rating_selection_reset': '_onClickStarDomainReset',
        // publisher comments
        'click .o_wrating_js_publisher_comment_btn': '_onClickPublisherComment',
        'click .o_wrating_js_publisher_comment_edit': '_onClickPublisherComment',
        'click .o_wrating_js_publisher_comment_delete': '_onClickPublisherCommentDelete',
        'click .o_wrating_js_publisher_comment_submit': '_onClickPublisherCommentSubmit',
        'click .o_wrating_js_publisher_comment_cancel': '_onClickPublisherCommentCancel',
    }),
    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        // options
        if (!Object.keys(this.options).includes("display_rating")) {
            this.options = Object.assign({
                'display_rating': false,
                'rating_default_value': 0.0,
            }, this.options);
        }
        // rating card
        this.set('rating_card_values', {});
        this.set('rating_value', false);
        this.on("change:rating_value", this, this._onChangeRatingDomain);
    },
    /**
     * @override
     */
    start: function () {
        this._super.apply(this, arguments);
        this.on("change:rating_card_values", this, this._renderRatingCard);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Update the messages format
     *
     * @param {Array<Object>} messages
     * @returns {Array}
     */
    preprocessMessages: function (messages) {
        var self = this;
        messages = this._super.apply(this, arguments);
        if (this.options['display_rating']) {
            messages.forEach((m, i) => {
                m.rating_value = self.roundToHalf(m['rating_value']);
                m.rating = self._preprocessCommentData(m.rating, i);
            });
        }
        // save messages in the widget to process correctly the publisher comment templates
        this.messages = messages;
        return messages;
    },
    /**
     * Round the given value with a precision of 0.5.
     *
     * Examples:
     * - 1.2 --> 1.0
     * - 1.7 --> 1.5
     * - 1.9 --> 2.0
     *
     * @param {Number} value
     * @returns Number
     **/
    roundToHalf: function (value) {
        var converted = parseFloat(value); // Make sure we have a number
        var decimal = (converted - parseInt(converted, 10));
        decimal = Math.round(decimal * 10);
        if (decimal === 5) {
            return (parseInt(converted, 10) + 0.5);
        }
        if ((decimal < 3) || (decimal > 7)) {
            return Math.round(converted);
        } else {
            return (parseInt(converted, 10) + 0.5);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {Promise}
     */
    _chatterInit: async function () {
        const result = await this._super(...arguments);
        this._updateRatingCardValues(result);
        return result;
    },
    /**
     * @override
     * @param {Array} domain
     * @returns {Promise}
     */
    messageFetch: async function (domain) {
        const result = await this._super(...arguments);
        this._updateRatingCardValues(result);
        return result;
    },
    /**
     * Calculates and Updates rating values i.e. average, percentage
     *
     * @private
     */
    _updateRatingCardValues: function (result) {
        if (!result['messages'][0]?.rating_stats) {
            return;
        }
        const ratingStats = result['messages'][0].rating_stats;
        const self = this;
        const ratingData = {
            'avg': Math.round(ratingStats['avg'] * 100) / 100,
            'percent': [],
        };
        Object.keys(ratingStats['percent'])
            .sort()
            .reverse()
            .forEach((rating) => {
                ratingData["percent"].push({
                    num: self.roundToHalf(rating),
                    percent: roundPrecision(ratingStats['percent'][rating], 0.01),
                });
            });
        this.set('rating_card_values', ratingData);
    },
    /**
     * @override
     */
    _messageFetchPrepareParams: function () {
        var params = this._super.apply(this, arguments);
        if (this.options['display_rating']) {
            params['rating_include'] = true;

            const ratingValue = this.get('rating_value');
            if (ratingValue !== false) {
                params['rating_value'] = ratingValue;
            }
        }
        return params;
    },

    /**
     * render rating card
     *
     * @private
     */
    _renderRatingCard: function () {
        this.$('.o_website_rating_card_container').replaceWith(renderToElement("portal_rating.rating_card", {widget: this}));
    },
    /**
     * Default rating data for publisher comment qweb template
     * @private
     * @param {Integer} messageIndex
     */
    _newPublisherCommentData: function (messageIndex) {
        return {
            mes_index: messageIndex,
            publisher_id: this.options.partner_id,
            publisher_avatar: `/web/image/res.partner/${this.options.partner_id}/avatar_128/50x50`,
            publisher_name: _t("Write your comment"),
            publisher_datetime: '',
            publisher_comment: '',
        };
    },

     /**
     * preprocess the rating data comming from /website/rating/comment or the chatter_init
     * Can be also use to have new rating data for a new publisher comment
     * @param {JSON} rawRating
     * @returns {JSON} the process rating data
     */
    _preprocessCommentData: function (rawRating, messageIndex) {
        var ratingData = {
            id: rawRating.id,
            mes_index: messageIndex,
            publisher_avatar: rawRating.publisher_avatar,
            publisher_comment: rawRating.publisher_comment,
            publisher_datetime: rawRating.publisher_datetime,
            publisher_id: rawRating.publisher_id,
            publisher_name: rawRating.publisher_name,
        };
        var commentData = {...this._newPublisherCommentData(messageIndex), ...ratingData};
        return commentData;
    },

    /** ---------------
     * Selection of elements for the publisher comment feature
     * Only available from a source in a publisher_comment or publisher_comment_form template
     */

    _getCommentContainer: function ($source) {
        return $source.parents(".o_wrating_publisher_container").first().find(".o_wrating_publisher_comment").first();
    },

    _getCommentButton: function ($source) {
        return $source.parents(".o_wrating_publisher_container").first().find(".o_wrating_js_publisher_comment_btn").first();
    },

    _getCommentTextarea: function ($source) {
        return $source.parents(".o_wrating_publisher_container").first().find(".o_portal_rating_comment_input").first();
    },

    _focusTextComment: function ($source) {
        this._getCommentTextarea($source).focus();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Show a spinner and hide messages during loading.
     *
     * @override
     * @returns {Promise}
     */
    _onChangeDomain: function () {
        const spinnerDelayed = setTimeout(()=> {
            this.$('.o_portal_chatter_messages_loading').removeClass('d-none');
            this.$('.o_portal_chatter_messages').addClass('d-none');
        }, 500);
        return this._super.apply(this, arguments).finally(()=>{
            clearTimeout(spinnerDelayed);
            // Hide spinner and show messages
            this.$('.o_portal_chatter_messages_loading').addClass('d-none');
            this.$('.o_portal_chatter_messages').removeClass('d-none');
        });
    },

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickStarDomain: function (ev) {
        var $tr = this.$(ev.currentTarget);
        var num = $tr.data('star');
        this.set('rating_value', num);
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickStarDomainReset: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        this.set('rating_value', false);
    },

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPublisherComment: function (ev) {
        var $source = this.$(ev.currentTarget);
        // If the form is already present => like cancel remove the form
        if (this._getCommentTextarea($source).length === 1) {
            this._getCommentContainer($source).empty();
            return;
        }
        var messageIndex = $source.data("mes_index");
        var data = {is_publisher: this.options['is_user_publisher']};
        data.rating = this._newPublisherCommentData(messageIndex);

        var oldRating = this.messages[messageIndex].rating;
        data.rating.publisher_comment = oldRating.publisher_comment ? oldRating.publisher_comment : '';
        this._getCommentContainer($source).empty().append(renderToElement("portal_rating.chatter_rating_publisher_form", data));
        this._focusTextComment($source);
    },

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPublisherCommentDelete: function (ev) {
        var self = this;
        var $source = this.$(ev.currentTarget);

        var messageIndex = $source.data("mes_index");
        var ratingId = this.messages[messageIndex].rating.id;

        this.rpc('/website/rating/comment', {
            "rating_id": ratingId,
            "publisher_comment": '' // Empty publisher comment means no comment
        }).then(function (res) {
            self.messages[messageIndex].rating = self._preprocessCommentData(res, messageIndex);
            self._getCommentButton($source).removeClass("d-none");
            self._getCommentContainer($source).empty();
        });
    },

     /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPublisherCommentSubmit: function (ev) {
        var self = this;
        var $source = this.$(ev.currentTarget);

        var messageIndex = $source.data("mes_index");
        var comment = this._getCommentTextarea($source).val();
        var ratingId = this.messages[messageIndex].rating.id;

        this.rpc('/website/rating/comment', {
            "rating_id": ratingId,
            "publisher_comment": comment
        }).then(function (res) {

            // Modify the related message
            self.messages[messageIndex].rating = self._preprocessCommentData(res, messageIndex);
            if (self.messages[messageIndex].rating.publisher_comment !== '') {
                // Remove the button comment if exist and render the comment
                self._getCommentButton($source).addClass('d-none');
                self._getCommentContainer($source).empty().append(renderToElement("portal_rating.chatter_rating_publisher_comment", {
                    rating: self.messages[messageIndex].rating,
                    is_publisher: self.options.is_user_publisher
                }));
            } else {
                // Empty string or false considers as no comment
                self._getCommentButton($source).removeClass("d-none");
                self._getCommentContainer($source).empty();
            }
        });
    },

     /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPublisherCommentCancel: function (ev) {
        var $source = this.$(ev.currentTarget);
        var messageIndex = $source.data("mes_index");

        var comment = this.messages[messageIndex].rating.publisher_comment;
        const $commentContainer = this._getCommentContainer($source);
        $commentContainer.empty();
        if (comment) {
            var data = {
                rating: this.messages[messageIndex].rating,
                is_publisher: this.options.is_user_publisher,
            };
            $commentContainer.append(renderToElement("portal_rating.chatter_rating_publisher_comment", data));
        }
    },

    /**
     * @private
     */
    _onChangeRatingDomain: function () {
        var domain = [];
        if (this.get('rating_value')) {
            // TODO dead code: remove in master
            domain = [['rating_value', '=', this.get('rating_value')]];
        }
        this._changeCurrentPage(1, domain);
    },
});

```

## File: static\src\js\portal_composer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import portalComposer from "@portal/js/portal_composer";

var PortalComposer = portalComposer.PortalComposer;

/**
 * PortalComposer
 *
 * Extends Portal Composer to handle rating submission
 */
PortalComposer.include({
    events: Object.assign({}, PortalComposer.prototype.events, {
        'click .stars i': '_onClickStar',
        'mouseleave .stars': '_onMouseleaveStarBlock',
        'mousemove .stars i': '_onMoveStar',
        'mouseleave .stars i': '_onMoveLeaveStar',
    }),

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);

        // apply ratio to default rating value
        if (options.default_rating_value) {
            options.default_rating_value = parseFloat(options.default_rating_value);
        }

        // default options
        this.options = Object.assign({
            'rate_with_void_content': false,
            'default_message': false,
            'default_message_id': false,
            'default_rating_value': 4.0,
            'force_submit_url': false,
        }, this.options);
        // star input widget
        this.labels = {
            '0': "",
            '1': _t("I hate it"),
            '2': _t("I don't like it"),
            '3': _t("It's okay"),
            '4': _t("I like it"),
            '5': _t("I love it"),
        };
        this.user_click = false; // user has click or not
        this.set("star_value", this.options.default_rating_value);
        this.on("change:star_value", this, this._onChangeStarValue);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            // rating stars
            self.$input = self.$('input[name="rating_value"]');
            self.$star_list = self.$('.stars').find('i');
            // if this is the first review, we do not use grey color contrast, even with default rating value.
            if (!self.options.default_message_id) {
                self.$star_list.removeClass('text-black-25');
            }

            // set the default value to trigger the display of star widget and update the hidden input value.
            self.set("star_value", self.options.default_rating_value);
            self.$input.val(self.options.default_rating_value);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _prepareMessageData: function () {
        return Object.assign(this._super(...arguments) || {}, {
            'message_id': this.options.default_message_id,
            'rating_value': this.$input.val()
        });
    },
    /**
     * @private
     */
    _onChangeStarValue: function () {
        var val = this.get("star_value");
        var index = Math.floor(val);
        var decimal = val - index;
        // reset the stars
        this.$star_list.removeClass('fa-star fa-star-half-o').addClass('fa-star-o');

        this.$('.stars').find("i:lt(" + index + ")").removeClass('fa-star-o fa-star-half-o').addClass('fa-star');
        if (decimal) {
            this.$('.stars').find("i:eq(" + index + ")").removeClass('fa-star-o fa-star fa-star-half-o').addClass('fa-star-half-o');
        }
        this.$('.rate_text .badge').text(this.labels[index]);
    },
    /**
     * @private
     */
    _onClickStar: function (ev) {
        var index = this.$('.stars i').index(ev.currentTarget);
        this.set("star_value", index + 1);
        this.user_click = true;
        this.$input.val(this.get("star_value"));
    },
    /**
     * @private
     */
    _onMouseleaveStarBlock: function () {
        this.$('.rate_text').hide();
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onMoveStar: function (ev) {
        var index = this.$('.stars i').index(ev.currentTarget);
        this.$('.rate_text').show();
        this.set("star_value", index + 1);
    },
    /**
     * @private
     */
    _onMoveLeaveStar: function () {
        if (!this.user_click) {
            this.set("star_value", parseInt(this.$input.val()));
        }
        this.user_click = false;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _onSubmitButtonClick: function (ev) {
        return this._super(...arguments).then((result) => {
            const $modal = this.$el.closest('#ratingpopupcomposer');
            $modal.on('hidden.bs.modal', () => {
              this.trigger_up('reload_rating_popup_composer', result);
            });
            $modal.modal('hide');
        }, () => {});
    },

    /**
     * @override
     * @private
     */
    _onSubmitCheckContent: function (ev) {
        if (this.options.rate_with_void_content) {
            if (this.$input.val() === 0) {
                return _t('The rating is required. Please make sure to select one before sending your review.')
            }
            return false;
        }
        return this._super.apply(this, arguments);
    },
});

```

## File: static\src\js\portal_rating_composer.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";
import { session } from "@web/session";
import portalComposer from "@portal/js/portal_composer";
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";

const PortalComposer = portalComposer.PortalComposer;

/**
 * RatingPopupComposer
 *
 * Display the rating average with a static star widget, and open
 * a popup with the portal composer when clicking on it.
 **/
const RatingPopupComposer = publicWidget.Widget.extend({
    selector: '.o_rating_popup_composer',
    custom_events: {
        reload_rating_popup_composer: '_onReloadRatingPopupComposer',
    },

    willStart: function (parent) {
        const def = this._super.apply(this, arguments);

        const options = this.$el.data();
        this.rating_avg = Math.round(options['rating_avg'] * 100) / 100 || 0.0;
        this.rating_count = options['rating_count'] || 0.0;

        this.options = Object.assign({
            'token': false,
            'res_model': false,
            'res_id': false,
            'pid': 0,
            'display_rating': true,
            'csrf_token': odoo.csrf_token,
            'user_id': session.user_id,
        }, options, {});

        return def;
    },

    /**
     * @override
     */
    start: function () {
        return Promise.all([
            this._super.apply(this, arguments),
            this._reloadRatingPopupComposer(),
        ]);
    },

    /**
     * Destroy existing ratingPopup and insert new ratingPopup widget
     *
     * @private
     * @param {Object} data
     */
    _reloadRatingPopupComposer: function () {
        if (this.options.hide_rating_avg) {
            this.$('.o_rating_popup_composer_stars').empty();
        } else {
            const ratingAverage = renderToElement(
                'portal_rating.rating_stars_static', {
                inline_mode: true,
                widget: this,
                val: this.rating_avg,
            });
            this.$('.o_rating_popup_composer_stars').empty().html(ratingAverage);
        }

        // Append the modal
        const modal = renderToElement(
            'portal_rating.PopupComposer', {
            inline_mode: true,
            widget: this,
            val: this.rating_avg,
        }) || '';
        this.$('.o_rating_popup_composer_modal').html(modal);

        if (this._composer) {
            this._composer.destroy();
        }

        // Instantiate the "Portal Composer" widget and insert it into the modal
        this._composer = new PortalComposer(this, this.options);
        return this._composer.appendTo(this.$('.o_rating_popup_composer_modal .o_portal_chatter_composer')).then(() => {
            // Change the text of the button
            this.$('.o_rating_popup_composer_text').text(
                this.options.is_fullscreen ?
                _t('Review') : this.options.default_message_id ?
                _t('Edit Review') : _t('Add Review')
            );
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {OdooEvent} event
     */
    _onReloadRatingPopupComposer: function (event) {
        const data = event.data;

        // Refresh the internal state of the widget
        this.rating_avg = data.rating_avg;
        this.rating_count = data.rating_count;
        this.rating_value = data.rating_value;

        // Clean the dictionary
        delete data.rating_avg;
        delete data.rating_count;
        delete data.rating_value;

        this.options = Object.assign(this.options, data);

        this._reloadRatingPopupComposer();
    }
});

publicWidget.registry.RatingPopupComposer = RatingPopupComposer;

export default RatingPopupComposer;

```

## File: static\src\xml\portal_chatter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <!--
        Inherited templates from portal to custom chatter with rating
    -->
    <t t-inherit="portal.Composer" t-inherit-mode="extension">
        <xpath expr="//textarea" position="inside"><t t-esc="widget.options['default_message'] ? widget.options['default_message'].trim() : ''"/></xpath><!-- need to be one line to avoid \t in textarea -->
        <xpath expr="//button[hasclass('o_portal_chatter_composer_btn')]" position="attributes">
            <attribute name="t-attf-data-action">#{widget.options['force_submit_url'] || '/mail/chatter_post'}</attribute>
        </xpath>
        <xpath expr="//*[hasclass('o_portal_chatter_composer_input')]/div[hasclass('o_portal_chatter_composer_body')]" position="before">
            <t t-call="portal_rating.rating_star_input">
                <t t-set="default_rating" t-value="widget.options['default_rating_value']"/>
            </t>
        </xpath>
    </t>

    <t t-inherit="portal.chatter_messages" t-inherit-mode="extension">
        <xpath expr="//t[@t-out='message.body']" position="before">
            <t t-if="message['rating_value']">
                <t t-call="portal_rating.rating_stars_static">
                    <t t-set="val" t-value="message.rating_value"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//*[hasclass('o_portal_chatter_attachments')]" position="after">
            <!--Only possible if a rating is link to the message, for now we can't comment if no rating
                is link to the message (because publisher comment data
                is on the rating.rating model - one comment max) -->
            <t t-if="message.rating and message.rating.id" t-call="portal_rating.chatter_rating_publisher">
                <t t-set="is_publisher" t-value="widget.options['is_user_publisher']"/>
                <t t-set="rating" t-value="message.rating"/>
            </t>
        </xpath>
    </t>

    <t t-inherit="portal.Chatter" t-inherit-mode="extension">
        <xpath expr="//t[@t-call='portal.chatter_message_count']" position="replace">
            <t t-if="widget.options['display_rating']">
                <t t-call="portal_rating.rating_card"/>
            </t>
            <t t-if="!widget.options['display_rating']">
                <t t-call="portal.chatter_message_count"/>
            </t>
        </xpath>
        <xpath expr="//t[@t-call='portal.chatter_messages']" position="before">
            <div class="o_portal_chatter_messages_loading d-none">
                <div class="d-flex justify-content-center">
                    <div class="spinner-border text-primary" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                </div>
            </div>
        </xpath>
    </t>

    <!--
        New templates specific of rating in Chatter
    -->
    <t t-name="portal_rating.chatter_rating_publisher">
        <div class="o_wrating_publisher_container">
            <button t-if="is_publisher"
                t-attf-class="btn px-2 mb-2 btn-sm border o_wrating_js_publisher_comment_btn {{ rating.publisher_comment !== '' ? 'd-none' : '' }}"
                t-att-data-mes_index="rating.mes_index">
                <i class="fa fa-comment text-muted me-1"/>Comment
            </button>
            <div class="o_wrating_publisher_comment mt-2 mb-2">
                <t t-if="rating.publisher_comment" t-call="portal_rating.chatter_rating_publisher_comment"/>
            </div>
        </div>
    </t>

    <t t-name="portal_rating.chatter_rating_publisher_comment">
        <div class="d-flex o_portal_chatter_message gap-2">
            <img class="o_avatar o_portal_chatter_avatar" t-att-src="rating.publisher_avatar" alt="avatar"/>
            <div class="flex-grow-1">
                <div class="o_portal_chatter_message_title">
                    <div class="d-inline-block">
                        <h5 class="mb-1"><t t-esc="rating.publisher_name"/></h5>
                    </div>
                    <div t-if="is_publisher" class="dropdown d-inline-block">
                        <button class="btn py-0" type="button" id="dropdownMenuButton" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                            <i class="fa fa-ellipsis-v"/>
                        </button>
                        <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                            <button class="dropdown-item o_wrating_js_publisher_comment_edit" t-att-data-mes_index="rating.mes_index">
                                <i class="fa fa-fw fa-pencil me-1"/>Edit
                            </button>
                            <button class="dropdown-item o_wrating_js_publisher_comment_delete" t-att-data-mes_index="rating.mes_index">
                                <i class="fa fa-fw fa-trash-o me-1"/>Delete
                            </button>
                        </div>
                    </div>
                    <p>Published on <t t-esc="rating.publisher_datetime"/></p>
                </div>
                <t t-out="rating.publisher_comment"/>
            </div>
        </div>
    </t>
    <t t-name="portal_rating.chatter_rating_publisher_form">
        <div t-if="is_publisher" class="d-flex o_portal_chatter_message shadow bg-white rounded px-3 py-3 my-1 gap-2">
            <img class="o_avatar o_portal_chatter_avatar" t-att-src="rating.publisher_avatar" alt="avatar"/>
            <div class="flex-grow-1">
                <div class="o_portal_chatter_message_title">
                    <h5 class='mb-1'><t t-esc="rating.publisher_name"/></h5>
                </div>
                <textarea rows="3" class="form-control o_portal_rating_comment_input"><t t-esc="rating.publisher_comment"/></textarea>
                <div>
                    <button class="btn btn-primary mt-2 o_wrating_js_publisher_comment_submit me-1" t-att-data-mes_index="rating.mes_index">
                        <t t-if="rating.publisher_comment === ''">
                            Post comment
                        </t><t t-else="">
                            Update comment
                        </t>
                    </button>
                    <button class="border btn btn-light mt-2 bg-white o_wrating_js_publisher_comment_cancel" t-att-data-mes_index="rating.mes_index">
                        Cancel
                    </button>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\portal_rating_composer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <!--
        Popup Rating Composer Widget
        It can also be used to modify a message
    -->
    <t t-name="portal_rating.PopupComposer">
        <div t-if="widget.options['display_composer']" class="modal fade" id="ratingpopupcomposer" tabindex="-1" role="dialog" aria-labelledby="ratingpopupcomposerlabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content bg-white">
                    <div class="modal-header">
                        <h5 class="modal-title o_rating_popup_composer_label" id="ratingpopupcomposerlabel">
                            <t t-if="widget.options['default_message_id']">
                                Modify your review
                            </t>
                            <t t-else="">
                                Write a review
                            </t>
                        </h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <div class="o_portal_chatter_composer"/>
                    </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\portal_tools.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="portal_rating.rating_stars_static">
        <t t-set="val_integer" t-value="Math.floor(val)"/>
        <t t-set="val_decimal" t-value="val - val_integer"/>
        <t t-set="empty_star" t-value="5 - (val_integer+Math.ceil(val_decimal))"/>
        <div class="o_website_rating_static"
             t-att-style="inline_mode ? 'display:inline' : ''"
             t-attf-aria-label="#{Math.round(val * 100) / 100} stars on 5"
             t-attf-title="#{Math.round(val * 100) / 100} stars on 5">
            <t t-foreach="Array(val_integer)" t-as="num" t-key="num_index">
                <i class="fa fa-star" role="img"></i>
            </t>
            <t t-if="val_decimal">
                <i class="fa fa-star-half-o" role="img"></i>
            </t>
            <t t-foreach="Array(empty_star)" t-as="num" t-key="num_index">
                <i class="fa fa-star text-black-25" role="img"></i>
            </t>
        </div>
    </t>

    <t t-name="portal_rating.rating_card">
        <t t-set="two_columns" t-value="widget.options['two_columns']"/>
        <div class="row o_website_rating_card_container justify-content-center">
            <div t-attf-class="#{two_columns and 'col-lg-12' or 'col-lg-5'}" t-if="Object.keys(widget.get('rating_card_values') || {}).length > 0">
                <p t-if="!two_columns" class="text-center"><strong>Average</strong></p>
                <div t-attf-class="o_website_rating_avg #{two_columns and 'mb-2' or 'text-center'}">
                    <h1><t t-esc="widget.get('rating_card_values')['avg']"/></h1>
                    <t t-call="portal_rating.rating_stars_static">
                        <t t-set="val" t-value="widget.get('rating_card_values')['avg'] || 0"/>
                    </t>
                    <t t-call="portal.chatter_message_count"/>
                </div>
            </div>
            <div t-attf-class="#{two_columns and 'col-lg-12' or 'col-lg-7'}" t-if="Object.keys(widget.get('rating_card_values') || {}).length > 0">
                <hr t-if="two_columns"/>
                <p t-if="!two_columns"><strong>Details</strong></p>
                <t t-set="selected_rating" t-value="widget.get('rating_value')"/>
                <table t-attf-class="o_website_rating_table #{selected_rating and 'o_website_rating_table_has_selection' or ''}">
                    <t t-foreach="widget.get('rating_card_values')['percent']" t-as="percent" t-key="percent_index">
                        <t t-set="row_selected" t-value="percent['num'] == selected_rating"/>
                        <tr t-attf-class="o_website_rating_table_row #{row_selected ? 'o_website_rating_table_row_selected' : (selected_rating ? 'opacity-50' : '')}"
                            t-att-data-star="percent['num']">
                            <td class="o_website_rating_table_star_num text-nowrap" t-att-data-star="percent['num']">
                                <t t-esc="percent['num']"/> stars
                            </td>
                            <td class="o_website_rating_table_progress">
                                <div class="progress">
                                    <div class="progress-bar o_rating_progressbar" role="progressbar" t-att-aria-valuenow="percent['percent']" aria-valuemin="0" aria-valuemax="100" t-att-style="'width:' + percent['percent'] + '%;'" aria-label="Progress bar">
                                    </div>
                                </div>
                            </td>
                            <td class="o_website_rating_table_percent">
                                <strong><t t-esc="Math.round(percent['percent'] * 100) / 100"/>%</strong>
                            </td>
                            <td class="o_website_rating_table_reset">
                                <button t-attf-class="btn btn-link o_website_rating_selection_reset #{row_selected and 'visible' or 'invisible'}"
                                        t-att-data-star="percent['num']">
                                    <i class="fa fa-times d-block" role="img" aria-label="Remove Selection"/>
                                </button>
                            </td>
                        </tr>
                    </t>
                </table>
            </div>
        </div>
    </t>

    <t t-name="portal_rating.rating_star_input">
        <div class="o_rating_star_card" t-if="widget.options['display_rating']">
            <t t-set="val_integer" t-value="Math.floor(default_rating)"/>
            <t t-set="val_decimal" t-value="default_rating - val_integer"/>
            <t t-set="empty_star" t-value="5 - (val_integer+Math.ceil(val_decimal))"/>

            <div class="stars enabled">
                <t t-foreach="Array(val_integer)" t-as="num" t-key="num_index">
                    <i class="fa fa-star" role="img" aria-label="Full star"></i>
                </t>
                <t t-if="val_decimal">
                    <i class="fa fa-star-half-o" role="img" aria-label="Half a star"></i>
                </t>
                <t t-foreach="Array(empty_star)" t-as="num" t-key="num_index">
                    <i class="fa fa-star-o text-black-25" role="img" aria-label="Empty star"></i>
                </t>
            </div>
            <div class="rate_text">
                <span class="badge text-bg-info"></span>
            </div>
            <input type="hidden" readonly="readonly" name="rating_value" t-att-value="default_rating || ''"/>
        </div>
    </t>
</templates>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="message_thread" inherit_id="portal.message_thread">
        <xpath expr="//div[@id='discussion']" position="attributes">
            <attribute name="t-att-data-display_rating">display_rating or False</attribute>
        </xpath>
    </template>
</odoo>

```

## File: views\rating_rating_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rating_rating_view_form" model="ir.ui.view">
        <field name="name">rating.rating.view.form</field>
        <field name="model">rating.rating</field>
        <field name="inherit_id" ref="rating.rating_rating_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='rating_text']" position="after">
                <field name="publisher_comment" string="Comment"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\rating_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--
        Rating widget static: show 5 stars (full or empty) regarding the given rating_avg and rating_count
    -->
    <template id="rating_widget_stars_static" name="Rating: static star widget">
        <t t-set="rating_avg" t-value="round(rating_avg * 100) / 100"/>
        <t t-set="val_decimal" t-value="round(rating_avg % 1, 1)"/>
        <t t-set="val_integer" t-value="int(rating_avg)"/>
        <t t-set="empty_star" t-value="5 - (val_integer+1) if val_decimal else 5 - val_integer"/>
        <div class="o_website_rating_static" t-att-style="inline_mode and 'display:inline'" t-att-title="rating_avg">
            <t t-if="rating_style_compressed">
                <t t-if="rating_avg &lt;= 2">
                    <i class="fa fa-star-o" role="img"></i>
                </t>
                <t t-elif="rating_avg &gt;= 2.1 and rating_avg &lt;= 3.5">
                    <i class="fa fa-star-half-o" role="img"></i>
                </t>
                <t t-else="">
                    <i class="fa fa-star" role="img"></i>
                </t>
                <small class="text-muted ms-1">
                    <t t-esc="rating_avg"/>
                </small>
            </t>
            <t t-else="">
                <t t-foreach="range(0, val_integer)" t-as="num">
                    <i class="fa fa-star" role="img"></i>
                </t>
                <t t-if="val_decimal">
                    <i class="fa fa-star-half-o" role="img"></i>
                </t>
                <t t-foreach="range(0, empty_star)" t-as="num">
                    <i class="fa fa-star-o" role="img"></i>
                </t>
                <small class="text-muted ms-1">
                    (<t t-esc="rating_count"/>)
                </small>
            </t>
        </div>
    </template>

    <!--
        Display static star widget, and open rating composer on click
        This template provide the container of the Popup Rating Composer. The rest is done in js.
        To use this template, you need to call it after setting the following variable in your template or in your controller:
            :float rating_avg : average rating to be displayed with star widget
            :object browserecord : the mail_thread object
            :token string (optional): if you want your chatter to be available for non-logged user,
                     you can use a token to verify the identity of the user;
                     the message will be posted with the identity of the partner_id of the object
    -->
    <template id="rating_stars_static_popup_composer" name="Rating: rating composer in popup">
        <t t-set="display_composer" t-value="not disable_composer and not (request.session.uid and env.user._is_public())"/>

        <div class="d-print-none o_rating_popup_composer o_not_editable p-0"
            contenteditable="false"
            t-att-data-rating_avg="rating_avg or 0.0"
            t-att-data-rating_count="rating_count or 0.0"
            t-att-data-token="token"
            t-att-data-hash="hash"
            t-att-data-pid="pid"
            t-att-data-res_model="object._name"
            t-att-data-res_id="object.id"
            t-att-data-partner_id="request.env.user.partner_id.id"
            t-att-data-default_message="default_message"
            t-att-data-default_message_id="default_message_id"
            t-att-data-default_rating_value="default_rating_value"
            t-att-data-default_attachment_ids="default_attachment_ids"
            t-att-data-force_submit_url="force_submit_url"
            t-att-data-rate_with_void_content="rate_with_void_content"
            t-att-data-disable_composer="disable_composer"
            t-att-data-display_composer="display_composer"
            t-att-data-link_btn_classes="_link_btn_classes"
            t-att-data-icon="icon"
            t-att-data-text_classes="_text_classes"
            t-att-data-hide_rating_avg="hide_rating_avg"
            t-att-data-is_fullscreen="is_fullscreen">
            <div class="d-flex flex-wrap align-items-center">
                <div class="o_rating_popup_composer_stars text-nowrap"/>
                <button t-if="display_composer" type="button"
                    t-att-class="'btn ' + _link_btn_classes or 'btn-primary'"
                    data-bs-toggle="modal" data-bs-target="#ratingpopupcomposer">
                    <i t-if="icon" t-att-class="icon"/>
                    <span t-attf-class="#{_text_classes} o_rating_popup_composer_text">
                        <t t-if="is_fullscreen">Review</t>
                        <t t-elif="default_message_id">Edit Review</t>
                        <t t-else="">Add Review</t>
                    </span>
                </button>
                <div class="o_rating_popup_composer_modal"/>
            </div>
        </div>
    </template>
</odoo>

```

