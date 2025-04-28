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
        'views/assets.xml',
        'views/rating_views.xml',
        'views/portal_templates.xml',
        'views/rating_templates.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal_chatter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request

from odoo.addons.portal.controllers.mail import PortalChatter


class PortalChatter(PortalChatter):

    def _portal_post_filter_params(self):
        fields = super(PortalChatter, self)._portal_post_filter_params()
        fields += ['rating_value', 'rating_feedback']
        return fields

    @http.route()
    def portal_chatter_post(self, res_model, res_id, message, redirect=None, attachment_ids='', attachment_tokens='', **kwargs):
        if kwargs.get('rating_value'):
            kwargs['rating_feedback'] = kwargs.pop('rating_feedback', message)
        return super(PortalChatter, self).portal_chatter_post(res_model, res_id, message, redirect=redirect, attachment_ids=attachment_ids, attachment_tokens=attachment_tokens, **kwargs)

    @http.route()
    def portal_chatter_init(self, res_model, res_id, domain=False, limit=False, **kwargs):
        result = super(PortalChatter, self).portal_chatter_init(res_model, res_id, domain=domain, limit=limit, **kwargs)
        # get the rating statistics about the record
        if kwargs.get('rating_include'):
            record = request.env[res_model].browse(res_id)
            if hasattr(record, 'rating_get_stats'):
                result['rating_stats'] = record.sudo().rating_get_stats()
        return result

    @http.route()
    def portal_message_fetch(self, res_model, res_id, domain=False, limit=False, offset=False, **kw):
        # add 'rating_include' in context, to fetch them in portal_message_format
        if kw.get('rating_include'):
            context = dict(request.context)
            context['rating_include'] = True
            request.context = context
        return super(PortalChatter, self).portal_message_fetch(res_model, res_id, domain=domain, limit=limit, offset=offset, **kw)

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
        rating = request.env['rating.rating'].search([('id', '=', int(rating_id))])
        if not rating:
            return {'error': _('Invalid rating')}
        rating.write({'publisher_comment': publisher_comment})
        # return to the front-end the created/updated publisher comment
        return rating.read(['publisher_comment', 'publisher_id', 'publisher_datetime'])[0]

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


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def _portal_message_format(self, field_list):
        # inlude rating value in data if requested
        if self._context.get('rating_include'):
            field_list += ['rating_value']
        return super(MailMessage, self)._portal_message_format(field_list)

    def _message_format(self, fnames):
        """ Override the method to add information about a publisher comment
        on each rating messages if requested, and compute a plaintext value of it.
        """
        vals_list = super(MailMessage, self)._message_format(fnames)

        if self._context.get('rating_include'):
            infos = ["id", "publisher_comment", "publisher_id", "publisher_datetime", "message_id"]
            related_rating = self.env['rating.rating'].sudo().search([('message_id', 'in', self.ids)]).read(infos)
            mid_rating_tree = dict((rating['message_id'][0], rating) for rating in related_rating)
            for vals in vals_list:
                vals["rating"] = mid_rating_tree.get(vals['id'], {})
        return vals_list

```

## File: models\rating.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models, exceptions, _


class Rating(models.Model):
    _inherit = 'rating.rating'

    # Adding information for comment a rating message
    publisher_comment = fields.Text("Publisher comment")
    publisher_id = fields.Many2one('res.partner', 'Commented by',
                                   ondelete='set null', readonly=True)
    publisher_datetime = fields.Datetime("Commented on", readonly=True)

    def write(self, values):
        if values.get('publisher_comment'):
            if not self.env.user.has_group("website.group_website_publisher"):
                raise exceptions.AccessError(_("Only the publisher of the website can change the rating comment"))
            if not values.get('publisher_datetime'):
                values['publisher_datetime'] = fields.Datetime.now()
            if not values.get('publisher_id'):
                values['publisher_id'] = self.env.user.partner_id.id
        return super(Rating, self).write(values)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import mail_message
from . import rating

```

## File: static\src\js\portal_chatter.js

```javascript
odoo.define('rating.portal.chatter', function (require) {
'use strict';

var core = require('web.core');
var portalChatter = require('portal.chatter');
var utils = require('web.utils');
var time = require('web.time');

var _t = core._t;
var PortalChatter = portalChatter.PortalChatter;
var qweb = core.qweb;

/**
 * PortalChatter
 *
 * Extends Frontend Chatter to handle rating
 */
PortalChatter.include({
    events: _.extend({}, PortalChatter.prototype.events, {
        // star based control
        'click .o_website_rating_select': '_onClickStarDomain',
        'click .o_website_rating_select_text': '_onClickStarDomainReset',
        // publisher comments
        'click .o_wrating_js_publisher_comment_btn': '_onClickPublisherComment',
        'click .o_wrating_js_publisher_comment_edit': '_onClickPublisherComment',
        'click .o_wrating_js_publisher_comment_delete': '_onClickPublisherCommentDelete',
        'click .o_wrating_js_publisher_comment_submit': '_onClickPublisherCommentSubmit',
        'click .o_wrating_js_publisher_comment_cancel': '_onClickPublisherCommentCancel',
    }),
    xmlDependencies: (PortalChatter.prototype.xmlDependencies || [])
        .concat([
            '/portal_rating/static/src/xml/portal_tools.xml',
            '/portal_rating/static/src/xml/portal_chatter.xml'
        ]),

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        // options
        if (!_.contains(this.options, 'display_rating')) {
            this.options = _.defaults(this.options, {
                'display_rating': false,
                'rating_default_value': 0.0,
            });
        }
        // rating card
        this.set('rating_card_values', {});
        this.set('rating_value', false);
        this.on("change:rating_value", this, this._onChangeRatingDomain);
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
            _.each(messages, function (m, i) {
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
     */
    _chatterInit: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function (result) {
            if (!result['rating_stats']) {
                return;
            }
            var ratingData = {
                'avg': Math.round(result['rating_stats']['avg'] * 100) / 100,
                'percent': [],
            };
            _.each(_.keys(result['rating_stats']['percent']).reverse(), function (rating) {
                ratingData['percent'].push({
                    'num': rating,
                    'percent': utils.round_precision(result['rating_stats']['percent'][rating], 0.01),
                });
            });
            self.set('rating_card_values', ratingData);
        });
    },
    /**
     * @override
     */
    _messageFetchPrepareParams: function () {
        var params = this._super.apply(this, arguments);
        if (this.options['display_rating']) {
            params['rating_include'] = true;
        }
        return params;
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
            publisher_avatar: _.str.sprintf('/web/image/%s/%s/image_128/50x50', 'res.partner', this.options.partner_id),
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
            publisher_datetime: rawRating.publisher_datetime ? moment(time.str_to_datetime(rawRating.publisher_datetime)).format('MMMM Do YYYY, h:mm:ss a') : "",
            publisher_comment: rawRating.publisher_comment ? rawRating.publisher_comment : '',
        };

        // split array (id, display_name) of publisher_id into publisher_id and publisher_name
        if (rawRating.publisher_id && rawRating.publisher_id.length >= 2) {
            ratingData.publisher_id = rawRating.publisher_id[0];
            ratingData.publisher_name = rawRating.publisher_id[1];
            ratingData.publisher_avatar = _.str.sprintf('/web/image/%s/%s/image_128/50x50', 'res.partner', ratingData.publisher_id);
        }
        var commentData = _.extend(this._newPublisherCommentData(messageIndex), ratingData);
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
     * @private
     * @param {MouseEvent} ev
     */
    _onClickStarDomain: function (ev) {
        var $tr = this.$(ev.currentTarget);
        var num = $tr.data('star');
        if ($tr.css('opacity') === '1') {
            this.set('rating_value', num);
            this.$('.o_website_rating_select').css({
                'opacity': 0.5,
            });
            this.$('.o_website_rating_select_text[data-star="' + num + '"]').css({
                'visibility': 'visible',
                'opacity': 1,
            });
            this.$('.o_website_rating_select[data-star="' + num + '"]').css({
                'opacity': 1,
            });
        }
    },
    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickStarDomainReset: function (ev) {
        ev.stopPropagation();
        ev.preventDefault();
        this.set('rating_value', false);
        this.$('.o_website_rating_select_text').css('visibility', 'hidden');
        this.$('.o_website_rating_select').css({
            'opacity': 1,
        });
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
        this._getCommentContainer($source).html($(qweb.render("portal_rating.chatter_rating_publisher_form", data)));
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

        this._rpc({
            route: '/website/rating/comment',
            params: {
                "rating_id": ratingId,
                "publisher_comment": '' // Empty publisher comment means no comment
            }
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

        this._rpc({
            route: '/website/rating/comment',
            params: {
                "rating_id": ratingId,
                "publisher_comment": comment
            }
        }).then(function (res) {

            // Modify the related message
            self.messages[messageIndex].rating = self._preprocessCommentData(res, messageIndex);
            if (self.messages[messageIndex].rating.publisher_comment !== '') {
                // Remove the button comment if exist and render the comment
                self._getCommentButton($source).addClass('d-none');
                self._getCommentContainer($source).html($(qweb.render("portal_rating.chatter_rating_publisher_comment", { 
                    rating: self.messages[messageIndex].rating,
                    is_publisher: self.options.is_user_publisher
                })));
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
        if (comment) {
            var data = {
                rating: this.messages[messageIndex].rating,
                is_publisher: this.options.is_user_publisher
            };
            this._getCommentContainer($source).html($(qweb.render("portal_rating.chatter_rating_publisher_comment", data)));
        } else {
            this._getCommentContainer($source).empty();
        }
    },

    /**
     * @private
     */
    _onChangeRatingDomain: function () {
        var domain = [];
        if (this.get('rating_value')) {
            domain = [['rating_value', '=', this.get('rating_value')]];
        }
        this._changeCurrentPage(1, domain);
    },
});
});

```

## File: static\src\js\portal_composer.js

```javascript
odoo.define('rating.portal.composer', function (require) {
'use strict';

var core = require('web.core');
var portalComposer = require('portal.composer');

var _t = core._t;

var PortalComposer = portalComposer.PortalComposer;

/**
 * PortalComposer
 *
 * Extends Portal Composer to handle rating submission
 */
PortalComposer.include({
    events: _.extend({}, PortalComposer.prototype.events, {
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
        this.options = _.defaults(this.options, {
            'default_message': false,
            'default_message_id': false,
            'default_rating_value': 0.0,
            'force_submit_url': false,
        });
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

            // set the default value to trigger the display of star widget and update the hidden input value.
            self.set("star_value", self.options.default_rating_value); 
            self.$input.val(self.options.default_rating_value);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

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
});
});

```

## File: static\src\js\portal_rating_composer.js

```javascript
odoo.define('portal.rating.composer', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var session = require('web.session');
var portalComposer = require('portal.composer');

var PortalComposer = portalComposer.PortalComposer;

/**
 * RatingPopupComposer
 *
 * Display the rating average with a static star widget, and open
 * a popup with the portal composer when clicking on it.
 **/
var RatingPopupComposer = publicWidget.Widget.extend({
    template: 'portal_rating.PopupComposer',
    xmlDependencies: [
        '/portal/static/src/xml/portal_chatter.xml',
        '/portal_rating/static/src/xml/portal_tools.xml',
        '/portal_rating/static/src/xml/portal_rating_composer.xml',
    ],

    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.rating_avg = Math.round(options['ratingAvg'] * 100) / 100 || 0.0;
        this.rating_total = options['ratingTotal'] || 0.0;

        this.options = _.defaults({}, options, {
            'token': false,
            'res_model': false,
            'res_id': false,
            'pid': 0,
            'display_composer': options['disable_composer'] ? false : !session.is_website_user,
            'display_rating': true,
            'csrf_token': odoo.csrf_token,
            'user_id': session.user_id,
        });
    },
    /**
     * @override
     */
    start: function () {
        var defs = [];
        defs.push(this._super.apply(this, arguments));

        // instanciate and insert composer widget
        this._composer = new PortalComposer(this, this.options);
        defs.push(this._composer.replace(this.$('.o_portal_chatter_composer')));

        return Promise.all(defs);
    },
});

publicWidget.registry.RatingPopupComposer = publicWidget.Widget.extend({
    selector: '.o_rating_popup_composer',

    /**
     * @override
     */
    start: function () {
        var ratingPopupData = this.$el.data();
        var ratingPopup = new RatingPopupComposer(this, ratingPopupData);
        return Promise.all([
            this._super.apply(this, arguments),
            ratingPopup.appendTo(this.$el)
        ]);
    },
});

return RatingPopupComposer;

});

```

## File: static\src\xml\portal_chatter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <!--
        Inherited templates from portal to custom chatter with rating
    -->
    <t t-extend="portal.Composer">
        <t t-jquery="textarea" t-operation="inner"><t t-esc="widget.options['default_message'] ? _.str.trim(widget.options['default_message']) : ''"/></t><!-- need to be one line to avoid \t in textarea -->
        <t t-jquery="form" t-operation="attributes">
            <attribute name="t-attf-action">#{widget.options['force_submit_url'] ? widget.options['force_submit_url'] : '/mail/chatter_post'}</attribute>
        </t>
        <t t-jquery=".o_portal_chatter_composer_form input[name='csrf_token']" t-operation="after">
            <t t-call="portal_rating.rating_star_input">
                <t t-set="default_rating" t-value="widget.options['default_rating_value']"/>
            </t>
            <input type="hidden" name="message_id" t-att-value="widget.options['default_message_id']" t-if="widget.options['default_message_id']"/>
        </t>
    </t>

    <t t-extend="portal.chatter_messages">
        <t t-jquery="t[t-raw='message.body']" t-operation="before">
            <t t-if="message['rating_value']">
                <t t-call="portal_rating.rating_stars_static">
                    <t t-set="val" t-value="message.rating_value"/>
                </t>
            </t>
        </t>
        <t t-jquery=".o_portal_chatter_attachments" t-operation="after">
            <!--Only possible if a rating is link to the message, for now we can't comment if no rating
                is link to the message (because publisher comment data
                is on the rating.rating model - one comment max) -->
            <t t-if="message.rating and message.rating.id" t-call="portal_rating.chatter_rating_publisher">
                <t t-set="is_publisher" t-value="widget.options['is_user_publisher']"/>
                <t t-set="rating" t-value="message.rating"/>
            </t>
        </t>
    </t>

    <t t-extend="portal.Chatter">
        <t t-jquery="t[t-call='portal.chatter_message_count']" t-operation="replace">
            <t t-if="widget.options['display_rating']">
                <t t-call="portal_rating.rating_card"/>
            </t>
            <t t-if="!widget.options['display_rating']">
                <t t-call="portal.chatter_message_count"/>
            </t>
        </t>
    </t>

    <!--
        New templates specific of rating in Chatter
    -->
    <t t-name="portal_rating.chatter_rating_publisher">
        <div class="o_wrating_publisher_container">
            <button t-if="is_publisher"
                t-attf-class="btn px-2 mb-2 btn-sm border o_wrating_js_publisher_comment_btn {{ rating.publisher_comment !== '' ? 'd-none' : '' }}"
                t-att-data-mes_index="rating.mes_index">
                <i class="fa fa-comment text-muted mr-1"/>Comment
            </button>
            <div class="o_wrating_publisher_comment mt-2 mb-2">
                <t t-if="rating.publisher_comment" t-call="portal_rating.chatter_rating_publisher_comment"/>
            </div>
        </div>
    </t>

    <t t-name="portal_rating.chatter_rating_publisher_comment">
        <div class="media o_portal_chatter_message">
            <img class="o_portal_chatter_avatar" t-att-src="rating.publisher_avatar" alt="avatar"/>
            <div class="media-body">
                <div class="o_portal_chatter_message_title">
                    <div class="d-inline-block">
                        <h5 class="mb-1"><t t-esc="rating.publisher_name"/></h5>
                    </div>
                    <div t-if="is_publisher" class="dropdown d-inline-block">
                        <button class="btn py-0" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                            <i class="fa fa-ellipsis-v"/>
                        </button>
                        <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                            <button class="dropdown-item o_wrating_js_publisher_comment_edit" t-att-data-mes_index="rating.mes_index">
                                <i class="fa fa-fw fa-pencil mr-1"/>Edit
                            </button>
                            <button class="dropdown-item o_wrating_js_publisher_comment_delete" t-att-data-mes_index="rating.mes_index">
                                <i class="fa fa-fw fa-trash-o mr-1"/>Delete
                            </button>
                        </div>
                    </div>
                    <p>Published on <t t-esc="rating.publisher_datetime"/></p>
                </div>
                <t t-esc="rating.publisher_comment"/>
            </div>
        </div>
    </t>
    <t t-name="portal_rating.chatter_rating_publisher_form">
        <div t-if="is_publisher" class="media o_portal_chatter_message shadow bg-white rounded px-3 py-3 my-1">
            <img class="o_portal_chatter_avatar" t-att-src="rating.publisher_avatar" alt="avatar"/>
            <div class="media-body">
                <div class="o_portal_chatter_message_title">
                    <h5 class='mb-1'><t t-esc="rating.publisher_name"/></h5>
                    <p>Published on <t t-esc="rating.publisher_datetime"/></p>
                </div>
                <textarea rows="3" class="form-control o_portal_rating_comment_input"><t t-esc="rating.publisher_comment"/></textarea>
                <div>
                    <button class="btn btn-primary mt-2 o_wrating_js_publisher_comment_submit" t-att-data-mes_index="rating.mes_index">
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
        It can alse be used to modify a message
    -->
    <t t-name="portal_rating.PopupComposer">
        <div class="d-flex flex-wrap align-items-center">
            <div class="text-nowrap">
                <t t-call="portal_rating.rating_stars_static">
                    <t t-set="val" t-value="widget.rating_avg || 0"/>
                    <t t-set="inline_mode" t-value="true"/>
                </t>
            </div>
            <button t-if="widget.options['display_composer']" type="button"
                t-att-class="'btn btn-sm mx-3 ' + widget.options['link_btn_classes'] or 'btn-primary'"
                data-toggle="modal" data-target="#ratingpopupcomposer">
                <t t-if="widget.options['display_composer']">
                    <t t-if="widget.options['default_message_id']">
                        Modify your review
                    </t>
                    <t t-else="">
                        Add a review
                    </t>
                </t>
            </button>
        </div>

        <div t-if="widget.options['display_composer']" class="modal fade" id="ratingpopupcomposer" tabindex="-1" role="dialog" aria-labelledby="ratingpopupcomposerlabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title o_rating_popup_composer_label" id="ratingpopupcomposerlabel">
                            <t t-if="widget.options['default_message_id']">
                                Modify your review
                            </t>
                            <t t-else="">
                                Add a review
                            </t>
                        </h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span>×</span>
                        </button>
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
        <div class="o_website_rating_static" t-att-style="inline_mode ? 'display:inline' : ''" t-attf-aria-label="#{val} stars on 5" t-attf-title="#{val} stars on 5">
            <t t-foreach="_.range(0, val_integer)" t-as="num">
                <i class="fa fa-star" role="img"></i>
            </t>
            <t t-if="val_decimal">
                <i class="fa fa-star-half-o" role="img"></i>
            </t>
            <t t-foreach="_.range(0, empty_star)" t-as="num">
                <i class="fa fa-star text-black-25" role="img"></i>
            </t>
        </div>
    </t>

    <t t-name="portal_rating.rating_card">
        <div class="row o_website_rating_card_container">
            <div class="col-lg-3 offset-lg-2" t-if="!_.isEmpty(widget.get('rating_card_values'))">
                <p><strong>Average</strong></p>
                <div class="o_website_rating_avg text-center">
                    <h1><t t-esc="widget.get('rating_card_values')['avg']"/></h1>
                    <t t-call="portal_rating.rating_stars_static">
                        <t t-set="val" t-value="widget.get('rating_card_values')['avg'] || 0"/>
                    </t>
                    <t t-call="portal.chatter_message_count"/>
                </div>
            </div>
            <div class="col-lg-6" t-if="!_.isEmpty(widget.get('rating_card_values'))">
                <p><strong>Details</strong></p>
                <div class="o_website_rating_progress_bars">
                    <table class="o_website_rating_progress_table">
                        <t t-foreach="widget.get('rating_card_values')['percent']" t-as="percent">
                            <tr class="o_website_rating_select" t-att-data-star="percent['num']" style="opacity: 1">
                                <td class="o_website_rating_table_star_num" t-att-data-star="percent['num']">
                                    <t t-esc="percent['num']"/> stars
                                </td>
                                <td class="o_website_rating_table_progress">
                                    <div class="progress">
                                        <div class="progress-bar o_rating_progressbar" role="progressbar" t-att-aria-valuenow="percent['percent']" aria-valuemin="0" aria-valuemax="100" t-att-style="'width:' + percent['percent'] + '%;'">
                                        </div>
                                    </div>
                                </td>
                                <td class="o_website_rating_table_percent">
                                    <strong><t t-esc="Math.round(percent['percent'] * 100) / 100"/>%</strong>
                                </td>
                                <td class="o_website_rating_table_reset">
                                    <button class="btn btn-link o_website_rating_select_text" t-att-data-star="percent['num']">
                                        <i class="fa fa-times d-block d-sm-none" role="img" aria-label="Remove Selection"/>
                                        <span class="d-none d-sm-block">Remove Selection</span>
                                    </button>
                                </td>
                            </tr>
                        </t>
                    </table>
                </div>
            </div>
        </div>
    </t>

    <t t-name="portal_rating.rating_star_input">
        <div class="o_rating_star_card" t-if="widget.options['display_rating']">
            <t t-set="val_integer" t-value="Math.floor(default_rating)"/>
            <t t-set="val_decimal" t-value="default_rating - val_integer"/>
            <t t-set="empty_star" t-value="5 - (val_integer+Math.ceil(val_decimal))"/>

            <div class="stars enabled">
                <t t-foreach="_.range(0, val_integer)" t-as="num">
                    <i class="fa fa-star" role="img" aria-label="One star" title="One star"></i>
                </t>
                <t t-if="val_decimal">
                    <i class="fa fa-star-half-o" role="img" aria-label="Half a star" title="Half a star"></i>
                </t>
                <t t-foreach="_.range(0, empty_star)" t-as="num" role="img" t-attf-aria-label="#{empty_star} on 5" t-attf-title="#{empty_star} on 5">
                    <i class="fa fa-star-o text-black-25"></i>
                </t>
            </div>
            <div class="rate_text">
                <span class="badge badge-info"></span>
            </div>
            <input type="hidden" readonly="readonly" name="rating_value" t-att-value="default_rating || ''"/>
        </div>
    </t>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_frontend" inherit_id="web_editor.assets_frontend" name="Portal Rating Assets">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/portal_rating/static/src/scss/portal_rating.scss"/>
        </xpath>
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/portal_rating/static/src/js/portal_chatter.js"></script>
            <script type="text/javascript" src="/portal_rating/static/src/js/portal_composer.js"></script>
            <script type="text/javascript" src="/portal_rating/static/src/js/portal_rating_composer.js"></script>
        </xpath>
    </template>
</odoo>

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
            <t t-foreach="range(0, val_integer)" t-as="num">
                <i class="fa fa-star" role="img"></i>
            </t>
            <t t-if="val_decimal">
                <i class="fa fa-star-half-o" role="img"></i>
            </t>
            <t t-foreach="range(0, empty_star)" t-as="num">
                <i class="fa fa-star text-black-25" role="img"></i>
            </t>
            (<t t-esc="rating_count"/>)
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
        <div class="d-print-none o_rating_popup_composer o_not_editable p-0"
            t-att-data-rating-avg="rating_avg or 0.0"
            t-att-data-rating-total="rating_total or 0.0"
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
            t-att-data-disable_composer="disable_composer"
            t-att-data-link_btn_classes="_link_btn_classes">
        </div>
    </template>
</odoo>

```

## File: views\rating_views.xml

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

