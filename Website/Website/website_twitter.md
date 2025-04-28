# Odoo Module: website_twitter

Category: Website/Website

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
    'name': 'Twitter Snippet',
    'category': 'Website/Website',
    'summary': 'Twitter scroller snippet in website',
    'version': '1.0',
    'description': """
This module adds a Twitter scroller building block to the website builder, so that you can display Twitter feeds on any page of your website.
    """,
    'depends': ['website'],
    'data': [
        'security/ir.model.access.csv',
        'data/website_twitter_data.xml',
        'views/res_config_settings_views.xml',
        'views/website_twitter_snippet_templates.xml'
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_twitter/static/src/scss/website_twitter.scss',
            'website_twitter/static/src/js/website.twitter.animation.js',
        ],
        'website.assets_editor': [
            'website_twitter/static/src/js/website.twitter.editor.js',
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
from odoo import _
from odoo import http
from odoo.http import request


class Twitter(http.Controller):
    @http.route(['/website_twitter/reload'], type='json', auth="user", website=True)
    def twitter_reload(self):
        return request.website.fetch_favorite_tweets()

    @http.route(['/website_twitter/get_favorites'], type='json', auth="public", website=True)
    def get_tweets(self, limit=20):
        key = request.website.sudo().twitter_api_key
        secret = request.website.sudo().twitter_api_secret
        screen_name = request.website.twitter_screen_name
        debug = request.env['res.users'].has_group('website.group_website_publisher')
        if not key or not secret:
            if debug:
                return {"error": _("Please set the Twitter API Key and Secret in the Website Settings.")}
            return []
        if not screen_name:
            if debug:
                return {"error": _("Please set a Twitter screen name to load favorites from, "
                                   "in the Website Settings (it does not have to be yours)")}
            return []
        TwitterTweets = request.env['website.twitter.tweet']
        tweets = TwitterTweets.search(
                [('website_id', '=', request.website.id),
                 ('screen_name', '=', screen_name)],
                limit=int(limit), order="tweet_id desc")
        if len(tweets) < 12:
            if debug:
                return {"error": _("Twitter user @%(username)s has less than 12 favorite tweets. "
                                   "Please add more or choose a different screen name.") % \
                                      {'username': screen_name}}
            else:
                return []
        return tweets.mapped(lambda t: json.loads(t.tweet))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\website_twitter_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_twitter_actions" model="ir.cron">
        <field name="name">Twitter: Fetch new favorites</field>
        <field name="model_id" ref="model_website"/>
        <field name="state">code</field>
        <field name="code">model._refresh_favorite_tweets()</field>
        <field name="interval_number">2</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
    </record>
</odoo>


```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

import requests

from odoo import api, fields, models, _, _lt
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

TWITTER_EXCEPTION = {
    304: _lt('There was no new data to return.'),
    400: _lt('The request was invalid or cannot be otherwise served. Requests without authentication are considered invalid and will yield this response.'),
    401: _lt('Authentication credentials were missing or incorrect. Maybe screen name tweets are protected.'),
    403: _lt('The request is understood, but it has been refused or access is not allowed. Please check your Twitter API Key and Secret.'),
    429: _lt('Request cannot be served due to the applications rate limit having been exhausted for the resource.'),
    500: _lt('Twitter seems broken. Please retry later. You may consider posting an issue on Twitter forums to get help.'),
    502: _lt('Twitter is down or being upgraded.'),
    503: _lt('The Twitter servers are up, but overloaded with requests. Try again later.'),
    504: _lt('The Twitter servers are up, but the request could not be serviced due to some failure within our stack. Try again later.')
}


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    twitter_api_key = fields.Char(
        related='website_id.twitter_api_key', readonly=False,
        string='API Key',
        help='Twitter API key you can get it from https://apps.twitter.com/')
    twitter_api_secret = fields.Char(
        related='website_id.twitter_api_secret', readonly=False,
        string='API secret',
        help='Twitter API secret you can get it from https://apps.twitter.com/')
    twitter_screen_name = fields.Char(
        related='website_id.twitter_screen_name', readonly=False,
        string='Favorites From',
        help='Screen Name of the Twitter Account from which you want to load favorites.'
             'It does not have to match the API Key/Secret.')
    twitter_server_uri = fields.Char(string='Twitter server uri', readonly=True)

    def _get_twitter_exception_message(self, error_code):
        if error_code in TWITTER_EXCEPTION:
            return TWITTER_EXCEPTION[error_code]
        else:
            return _('HTTP Error: Something is misconfigured')

    def _check_twitter_authorization(self):
        try:
            self.website_id.fetch_favorite_tweets()

        except requests.HTTPError as e:
            _logger.info("%s - %s" % (e.response.status_code, e.response.reason), exc_info=True)
            raise UserError("%s - %s" % (e.response.status_code, e.response.reason) + ':' + self._get_twitter_exception_message(e.response.status_code))
        except IOError:
            _logger.info('We failed to reach a twitter server.', exc_info=True)
            raise UserError(_('Internet connection refused: We failed to reach a twitter server.'))
        except Exception:
            _logger.info('Please double-check your Twitter API Key and Secret!', exc_info=True)
            raise UserError(_('Twitter authorization error! Please double-check your Twitter API Key and Secret!'))

    @api.model
    def create(self, vals):
        TwitterConfig = super(ResConfigSettings, self).create(vals)
        if vals.get('twitter_api_key') or vals.get('twitter_api_secret') or vals.get('twitter_screen_name'):
            TwitterConfig._check_twitter_authorization()
        return TwitterConfig

    def write(self, vals):
        TwitterConfig = super(ResConfigSettings, self).write(vals)
        if vals.get('twitter_api_key') or vals.get('twitter_api_secret') or vals.get('twitter_screen_name'):
            self._check_twitter_authorization()
        return TwitterConfig

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        Params = self.env['ir.config_parameter'].sudo()
        res.update({
            'twitter_server_uri': '%s/' % Params.get_param('web.base.url', default='http://yourcompany.odoo.com'),
        })
        return res

```

## File: models\website_twitter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

import requests
from odoo import api, fields, models

API_ENDPOINT = 'https://api.twitter.com'
API_VERSION = '1.1'
REQUEST_TOKEN_URL = '%s/oauth2/token' % API_ENDPOINT
REQUEST_FAVORITE_LIST_URL = '%s/%s/favorites/list.json' % (API_ENDPOINT, API_VERSION)
URLOPEN_TIMEOUT = 10

_logger = logging.getLogger(__name__)


class WebsiteTwitter(models.Model):
    _inherit = 'website'

    twitter_api_key = fields.Char(string='Twitter API key', help='Twitter API Key', groups='base.group_system')
    twitter_api_secret = fields.Char(string='Twitter API secret', help='Twitter API Secret', groups='base.group_system')
    twitter_screen_name = fields.Char(string='Get favorites from this screen name')

    @api.model
    def _request(self, website, url, params=None):
        """Send an authenticated request to the Twitter API."""
        access_token = self._get_access_token(website)
        try:
            request = requests.get(url, params=params, headers={'Authorization': 'Bearer %s' % access_token}, timeout=URLOPEN_TIMEOUT)
            request.raise_for_status()
            return request.json()
        except requests.HTTPError as e:
            _logger.debug("Twitter API request failed with code: %r, msg: %r, content: %r",
                          e.response.status_code, e.response.reason, e.response.content)
            raise

    @api.model
    def _refresh_favorite_tweets(self):
        ''' called by cron job '''
        website = self.env['website'].search([('twitter_api_key', '!=', False),
                                          ('twitter_api_secret', '!=', False),
                                          ('twitter_screen_name', '!=', False)])
        _logger.debug("Refreshing tweets for website IDs: %r", website.ids)
        website.fetch_favorite_tweets()

    def fetch_favorite_tweets(self):
        WebsiteTweets = self.env['website.twitter.tweet']
        tweet_ids = []
        for website in self:
            if not all((website.sudo().twitter_api_key, website.sudo().twitter_api_secret, website.twitter_screen_name)):
                _logger.debug("Skip fetching favorite tweets for unconfigured website %s", website)
                continue
            params = {'screen_name': website.twitter_screen_name}
            last_tweet = WebsiteTweets.search([('website_id', '=', website.id),
                                                     ('screen_name', '=', website.twitter_screen_name)],
                                                     limit=1, order='tweet_id desc')
            if last_tweet:
                params['since_id'] = int(last_tweet.tweet_id)
            _logger.debug("Fetching favorite tweets using params %r", params)
            response = self._request(website, REQUEST_FAVORITE_LIST_URL, params=params)
            for tweet_dict in response:
                tweet_id = tweet_dict['id']  # unsigned 64-bit snowflake ID
                tweet_ids = WebsiteTweets.search([('tweet_id', '=', tweet_id)]).ids
                if not tweet_ids:
                    new_tweet = WebsiteTweets.create(
                            {
                              'website_id': website.id,
                              'tweet': json.dumps(tweet_dict),
                              'tweet_id': tweet_id,  # stored in NUMERIC PG field
                              'screen_name': website.twitter_screen_name,
                            })
                    _logger.debug("Found new favorite: %r, %r", tweet_id, tweet_dict)
                    tweet_ids.append(new_tweet.id)
        return tweet_ids

    def _get_access_token(self, website):
        """Obtain a bearer token."""
        r = requests.post(
            REQUEST_TOKEN_URL,
            data={'grant_type': 'client_credentials',},
            auth=(website.sudo().twitter_api_key, website.sudo().twitter_api_secret),
            timeout=URLOPEN_TIMEOUT,
        )
        r.raise_for_status()
        data = r.json()
        access_token = data['access_token']
        return access_token

```

## File: models\website_twitter_tweet.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class WebsiteTwitterTweet(models.Model):
    _name = 'website.twitter.tweet'
    _description = 'Website Twitter'

    website_id = fields.Many2one('website', string='Website', ondelete='cascade')
    screen_name = fields.Char(string='Screen Name')
    tweet = fields.Text(string='Tweets')

    # Twitter IDs are 64-bit unsigned ints, so we need to store them in
    # unlimited precision NUMERIC columns, which can be done with a
    # float field. Used digits=(0,0) to indicate unlimited.
    # Using VARCHAR would work too but would have sorting problems.
    tweet_id = fields.Float(string='Tweet ID', digits=(0, 0))  # Twitter

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website_twitter
from . import res_config_settings
from . import website_twitter_tweet

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_website_twitter_tweet_public,access of twitter snippet,website_twitter.model_website_twitter_tweet,,1,0,0,0
access_website_twitter_tweet_manage,manage tweets,website_twitter.model_website_twitter_tweet,website.group_website_publisher,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V41.016L12 28l4 4 6-4 5 5 6-6h20l6 16-16.49 26H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M33 29h26v16H29c4.667-7.333 6-12.667 4-16zm11 4l-4 4 4 4 1-1-3-3 3-3-1-1zm4 0l-1 1 3 3-3 3 1 1 4-4-4-4zm-19.056-.074c.013.178.013.356.013.533 0 5.42-4.124 11.663-11.663 11.663-2.322 0-4.48-.673-6.294-1.84.33.038.647.05.99.05 1.916 0 3.68-.647 5.089-1.75a4.106 4.106 0 0 1-3.833-2.843c.254.038.508.063.774.063.368 0 .736-.05 1.079-.14a4.1 4.1 0 0 1-3.287-4.022v-.051c.546.304 1.18.495 1.853.52a4.096 4.096 0 0 1-1.827-3.414c0-.761.203-1.459.558-2.068a11.651 11.651 0 0 0 8.452 4.29 4.627 4.627 0 0 1-.102-.94 4.097 4.097 0 0 1 4.1-4.099 4.09 4.09 0 0 1 2.994 1.295 8.07 8.07 0 0 0 2.602-.99 4.088 4.088 0 0 1-1.802 2.259 8.217 8.217 0 0 0 2.36-.635 8.811 8.811 0 0 1-2.056 2.12z" opacity=".3"/><path fill="#FFF" d="M33 27h26v16H29c4.667-7.333 6-12.667 4-16zm11 4l-4 4 4 4 1-1-3-3 3-3-1-1zm4 0l-1 1 3 3-3 3 1 1 4-4-4-4zm-19.056-.074c.013.178.013.356.013.533 0 5.42-4.124 11.663-11.663 11.663-2.322 0-4.48-.673-6.294-1.84.33.038.647.05.99.05 1.916 0 3.68-.647 5.089-1.75a4.106 4.106 0 0 1-3.833-2.843c.254.038.508.063.774.063.368 0 .736-.05 1.079-.14a4.1 4.1 0 0 1-3.287-4.022v-.051c.546.304 1.18.495 1.853.52a4.096 4.096 0 0 1-1.827-3.414c0-.761.203-1.459.558-2.068a11.651 11.651 0 0 0 8.452 4.29 4.627 4.627 0 0 1-.102-.94 4.097 4.097 0 0 1 4.1-4.099 4.09 4.09 0 0 1 2.994 1.295 8.07 8.07 0 0 0 2.602-.99 4.088 4.088 0 0 1-1.802 2.259 8.217 8.217 0 0 0 2.36-.635 8.811 8.811 0 0 1-2.056 2.12z"/></g></g></svg>
```

## File: static\src\js\website.twitter.animation.js

```javascript
odoo.define('website_twitter.animation', function (require) {
'use strict';

var core = require('web.core');
const {Markup} = require('web.utils');
var publicWidget = require('web.public.widget');

var qweb = core.qweb;

publicWidget.registry.twitter = publicWidget.Widget.extend({
    selector: '.twitter',
    xmlDependencies: ['/website_twitter/static/src/xml/website.twitter.xml'],
    disabledInEditableMode: false,
    events: {
        'mouseenter .wrap-row': '_onEnterRow',
        'mouseleave .wrap-row': '_onLeaveRow',
        'click .twitter_timeline .tweet': '_onTweetClick',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        var $timeline = this.$('.twitter_timeline');

        $timeline.append('<center><div><img src="/website_twitter/static/src/img/loadtweet.gif"></div></center>');
        var def = this._rpc({route: '/website_twitter/get_favorites'}).then(function (data) {
            $timeline.empty();

            if (data.error) {
                $timeline.append(qweb.render('website.Twitter.Error', {data: data}));
                return;
            }

            if (_.isEmpty(data)) {
                return;
            }

            var tweets = _.map(data, function (tweet) {
                // Parse tweet date
                if (_.isEmpty(tweet.created_at)) {
                    tweet.created_at = '';
                } else {
                    var v = tweet.created_at.split(' ');
                    var d = new Date(Date.parse(v[1]+' '+v[2]+', '+v[5]+' '+v[3]+' UTC'));
                    tweet.created_at = d.toDateString();
                }

                // Parse tweet text
                tweet.text = Markup(_.escape(tweet.text)
                    .replace(
                        /[A-Za-z]+:\/\/[A-Za-z0-9-_]+\.[A-Za-z0-9-_:%&~\?\/.=]+/g,
                        function (url) {
                            return _makeLink(url, url);
                        }
                    )
                    .replace(
                        /[@]+[A-Za-z0-9_]+/g,
                        function (screen_name) {
                            return _makeLink('http://twitter.com/' + screen_name.replace('@', ''), screen_name);
                        }
                    )
                    .replace(
                        /[#]+[A-Za-z0-9_]+/g,
                        function (hashtag) {
                            return _makeLink('http://twitter.com/search?q=' + encodeURIComponent(hashtag.replace('#', '')), hashtag);
                        }
                    ));

                return qweb.render('website.Twitter.Tweet', {tweet: tweet});

                function _makeLink(url, text) {
                    return Markup`<a href="${url}" target="_blank" rel="noreferrer noopener">${text}</a>`;
                }
            });

            var f = Math.floor(tweets.length / 3);
            var tweetSlices = [tweets.slice(0, f).join(' '), tweets.slice(f, f * 2).join(' '), tweets.slice(f * 2, tweets.length).join(' ')];

            self.$scroller = $(qweb.render('website.Twitter.Scroller')).appendTo($timeline);
            _.each(self.$scroller.find('div[id^="scroller"]'), function (element, index) {
                var $scrollWrapper = $('<div/>', {class: 'scrollWrapper'});
                var $scrollableArea = $('<div/>', {class: 'scrollableArea'});
                $scrollWrapper.append($scrollableArea)
                              .data('scrollableArea', $scrollableArea);
                $scrollableArea.append(tweetSlices[index]);
                $(element).append($scrollWrapper);
                var totalWidth = 0;
                _.each($scrollableArea.children(), function (area) {
                    totalWidth += $(area).outerWidth(true);
                });
                $scrollableArea.width(totalWidth);
                $scrollWrapper.scrollLeft(index*180);
            });
            self._startScrolling();
        });

        return Promise.all([this._super.apply(this, arguments), def]);
    },
    /**
     * @override
     */
    destroy: function () {
        this._stopScrolling();
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _startScrolling: function () {
        if (!this.$scroller) {
            return;
        }
        _.each(this.$scroller.find('.scrollWrapper'), function (el) {
            var $wrapper = $(el);
            $wrapper.data('getNextElementWidth', true);
            $wrapper.data('autoScrollingInterval', setInterval(function () {
                $wrapper.scrollLeft($wrapper.scrollLeft() + 1);
                if ($wrapper.data('getNextElementWidth')) {
                    $wrapper.data('swapAt', $wrapper.data('scrollableArea').children(':first').outerWidth(true));
                    $wrapper.data('getNextElementWidth', false);
                }
                if ($wrapper.data('swapAt') <= $wrapper.scrollLeft()) {
                    var swap_el = $wrapper.data('scrollableArea').children(':first').detach();
                    $wrapper.data('scrollableArea').append(swap_el);
                    $wrapper.scrollLeft($wrapper.scrollLeft() - swap_el.outerWidth(true));
                    $wrapper.data('getNextElementWidth', true);
                }
            }, 20));
        });
    },
    /**
     * @private
     */
    _stopScrolling: function (wrapper) {
        if (!this.$scroller) {
            return;
        }
        _.each(this.$scroller.find('.scrollWrapper'), function (el) {
            var $wrapper = $(el);
            clearInterval($wrapper.data('autoScrollingInterval'));
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onEnterRow: function () {
        this._stopScrolling();
    },
    /**
     * @private
     */
    _onLeaveRow: function () {
        this._startScrolling();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onTweetClick: function (ev) {
        if (ev.target.tagName === 'A') {
            return;
        }
        var url = $(ev.currentTarget).data('url');
        if (url) {
            window.open(url, '_blank');
        }
    },
});
});

```

## File: static\src\js\website.twitter.editor.js

```javascript
odoo.define('website_twitter.editor', function (require) {
'use strict';

var core = require('web.core');
var dom = require('web.dom');
var sOptions = require('web_editor.snippets.options');

var _t = core._t;

sOptions.registry.twitter = sOptions.Class.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        var $configuration = dom.renderButton({
            attrs: {
                class: 'btn-primary d-none',
                contenteditable: 'false',
            },
            text: _t("Reload"),
        });
        $configuration.appendTo(document.body).on('click', function (ev) {
            ev.preventDefault();
            ev.stopPropagation();
            self._rpc({route: '/website_twitter/reload'});
        });
        this.$target.on('mouseover.website_twitter', function () {
            var $selected = $(this);
            var position = $selected.offset();
            $configuration.removeClass('d-none').offset({
                top: $selected.outerHeight() / 2
                        + position.top
                        - $configuration.outerHeight() / 2,
                left: $selected.outerWidth() / 2
                        + position.left
                        - $configuration.outerWidth() / 2,
            });
        }).on('mouseleave.website_twitter', function (e) {
            var current = document.elementFromPoint(e.clientX, e.clientY);
            if (current === $configuration[0]) {
                return;
            }
            $configuration.addClass('d-none');
        });
        this.$target.on('click.website_twitter', '.lnk_configure', function (e) {
            window.location = e.currentTarget.href;
        });
        this.trigger_up('widgets_stop_request', {
            $target: this.$target,
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    cleanForSave: function () {
        this.$target.find('.twitter_timeline').empty();
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        this.$target.off('.website_twitter');
    },
});
});

```

## File: static\src\xml\website.twitter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="website.Twitter.Tweet">
        <div class="tweet" t-attf-data-url="http://twitter.com/#{tweet.user.screen_name}/status/#{tweet.id_str}" t-attf-data-tweet-id="#{tweet.id_str}">
            <div class="left">
                <img t-att-src="tweet.user.profile_image_url_https" alt="Twitter's user"/>
            </div>
            <div class="right">
                <div class="top">
                    <h4>
                        <t t-esc="tweet.user.name"/>
                        <span>
                            <a t-att-href="'https://twitter.com/' + tweet.user.screen_name" target="_blank"><t t-esc="'@' + tweet.user.screen_name "/></a>
                        </span>
                    </h4>
                    <a class="date" target="_blank" t-attf-href="http://twitter.com/#{tweet.user.screen_name}/status/#{tweet.id_str}"><t t-esc="tweet.created_at"/></a>
                </div>
                <div class="bottom">
                    <p><t t-out="tweet.text"/></p>
                </div>
            </div>
        </div>
    </t>
    <t t-name="website.Twitter.Scroller">
        <div class="wrap-row" contenteditable="false">
            <div class="twitter-row">
                <div class="twitter-scroller">
                    <div id="scroller1"/>
                    <div id="scroller2"/>
                    <div id="scroller3"/>
                </div>
            </div>
        </div>
    </t>
    <t t-name="website.Twitter.Error">
        <div class="container" contenteditable="false">
            <div class="alert alert-warning" role="alert">
                <t t-esc="data.error"/>
                <t t-if='!data.nodata'>
                    <a class="lnk_configure" href="/web#action=website.action_website_configuration"><i class="fa fa-plus-circle"/> Twitter Configuration</a>
                </t>
            </div>
        </div>
    </t>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.twitter</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="google_maps_setting" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="twitter_roller_install_setting">
                    <div class="o_setting_right_pane">
                        <span class="o_form_label">Twitter Roller</span>
                        <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                        <div class="text-muted">
                            Twitter API Credentials
                        </div>
                        <div class="content-group">
                            <div class="row mt16">
                                <label class="col-lg-3 o_light_label" string="API Key" for="twitter_api_key"/>
                                <field name="twitter_api_key" class="oe_inline"/>
                            </div>
                            <div class="row">
                                <label class="col-lg-3 o_light_label" string="API secret" for="twitter_api_secret"/>
                                <field name="twitter_api_secret" password="True" class="oe_inline"/>
                            </div>
                            <a data-toggle="collapse" href="#" data-target="#twitter_tutorial" aria-label="Twitter tutorial">
                                <i class="fa fa-arrow-right"/>
                                Show me how to obtain the Twitter API key and Twitter API secret
                            </a>
                            <div class="row mt16 collapse" id="twitter_tutorial">
                                <blockquote class="small">
                                    <h2>How to configure the Twitter API access</h2>
                                    <ol>
                                        <li>Log in or create an account on <a href="https://developer.twitter.com/" target="new"> https://developer.twitter.com/</a></li>
                                        <li>Once connected, and if not already done, complete the Twitter portal access process on <a href="https://developer.twitter.com/portal/" target="new">https://developer.twitter.com/portal/</a></li>
                                        <li>On the <a href="https://developer.twitter.com/portal/" target="new">Twitter Portal</a>, <strong>create a project</strong> with the following information:
                                            <ul>
                                                <li><strong>Name: </strong> Odoo Twitter Integration</li>
                                                <li><strong>Use Case: </strong> Embedding Tweets in a website</li>
                                                <li><strong>Description: </strong> Odoo Twitter Integration</li>
                                                <li><strong>App Name: </strong> choose a unique name</li>
                                            </ul>
                                        </li>
                                        <li>Copy/Paste the API Key and API Key Secret values into the above fields</li>
                                        <li>Get Elevated access by going to <a href="https://developer.twitter.com/en/portal/products" target="new">https://developer.twitter.com/en/portal/products</a>, click on <strong>Elevated</strong> then on <strong>Apply</strong> and finally complete the form.</li>
                                    </ol>
                                </blockquote>
                            </div>
                            <div class="row">
                                <label class="col-lg-3 o_light_label" string="Favorites From" for="twitter_screen_name"/>
                                <field name="twitter_screen_name" class="oe_inline"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\website_twitter_snippet_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_twitter" name="Twitter Scroller">
        <section class="twitter" data-screen-name="OpenERP" data-limit="15">
            <div class="twitter_timeline o_not_editable"/>
        </section>
    </template>

    <template id="remove_external_snippets" inherit_id="website.external_snippets">
        <xpath expr="//t[@t-install='website_twitter']" position="replace"/>
    </template>
     <template id="website_twitter_snippet" inherit_id="website.snippets">
        <xpath expr="//t[@id='twitter_favorite_tweets_hook']" position="replace">
            <t t-snippet="website_twitter.s_twitter" t-thumbnail="/website/static/src/img/snippets_thumbs/s_twitter_scroll.svg"/>
        </xpath>
    </template>

    <template id="website_twitter_options" inherit_id="website.snippet_options">
        <xpath expr="." position="inside">
            <div data-js="twitter" data-selector=".twitter"/>
        </xpath>
    </template>

    </odoo>

```

