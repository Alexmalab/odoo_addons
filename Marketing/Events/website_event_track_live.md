# Odoo Module: website_event_track_live

Category: Marketing/Events

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
    'name': 'Live Event Tracks',
    'category': 'Marketing/Events',
    'sequence': 1006,
    'version': '1.0',
    'summary': 'Support live tracks: streaming, participation, youtube',
    'website': 'https://www.odoo.com/app/events',
    'depends': [
        'website_event_track',
    ],
    'data': [
        'views/event_track_templates_list.xml',
        'views/event_track_templates_page.xml',
        'views/event_track_views.xml',
    ],
    'demo': [
        'data/event_track_demo.xml'
    ],
    'installable': True,
    'assets': {
        'web.assets_frontend': [
            'website_event_track_live/static/src/scss/website_event_track_live.scss',
            'website_event_track_live/static/src/js/website_event_track_replay_suggestion.js',
            'website_event_track_live/static/src/js/website_event_track_suggestion.js',
            'website_event_track_live/static/src/js/website_event_track_live.js',
            'website_event_track_live/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo.addons.website_event_track.controllers.event_track import EventTrackController
from odoo.http import request


class WebsiteEventSessionLiveController(EventTrackController):

    def _event_track_page_get_values(self, event, track, **options):
        if 'widescreen' not in options:
            options['widescreen'] = track.youtube_video_url and (track.is_youtube_replay or track.is_track_soon or track.is_track_live or track.is_track_done)
        values = super(WebsiteEventSessionLiveController, self)._event_track_page_get_values(event, track, **options)
        # Youtube disables the chat embed on all mobile devices
        # This regex is a naive attempt at matching their behavior (should work for most cases)
        values['is_mobile_chat_disabled'] = bool(re.match(
            r'^.*(Android|iPad|iPhone).*',
            request.httprequest.headers.get('User-Agent', request.httprequest.headers.get('user-agent', ''))))
        return values

```

## File: controllers\track_live.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http

from odoo.addons.website_event_track.controllers.event_track import EventTrackController
from odoo.osv import expression

class EventTrackLiveController(EventTrackController):

    @http.route('/event_track/get_track_suggestion', type='json', auth='public', website=True)
    def get_next_track_suggestion(self, track_id):
        track = self._fetch_track(track_id)
        track_suggestion = track._get_track_suggestions(
            restrict_domain=expression.AND([
                self._get_event_tracks_domain(track.event_id),
                [('youtube_video_url', '!=', False)]
            ]), limit=1)
        if not track_suggestion:
            return False
        track_suggestion_sudo = track_suggestion.sudo()
        track_sudo = track.sudo()
        return self._prepare_track_suggestion_values(track_sudo, track_suggestion_sudo)

    def _prepare_track_suggestion_values(self, track, track_suggestion):
        return {
            'current_track': {
                'name': track.name,
                'website_image_url': track.website_image_url,
            },
            'suggestion': {
                'id': track_suggestion.id,
                'name': track_suggestion.name,
                'speaker_name': track_suggestion.partner_name,
                'website_url': track_suggestion.website_url
            }
        }

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import track_live
from . import session

```

## File: data\event_track_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Tracks of: "OpenWood: Furniture Collection Online Reveal" -->
    <record id="website_event_track.event_7_track_3" model="event.track">
        <!-- Easy Way To Build a Wooden House -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=qRrA8al77a0</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_5" model="event.track">
        <!-- Top 10 Most Expensive Wood in the World -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=bZ-queeu-lk</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_6" model="event.track">
        <!-- Haul Long Lumber in a Shortbox Truck -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=gV3g5FzmQiM</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_7" model="event.track">
        <!-- Woodworking: How I got started! -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=loP-Nd7M5s8</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_13" model="event.track">
        <!-- Easy Way To Build a Wooden House -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=qRrA8al77a0</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_15" model="event.track">
        <!-- Logs to Lumber -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=66AIsiezq5g</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_17" model="event.track">
        <!-- 10 DIY Furniture Ideas For Absolute Beginners -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=OE4BVOjJp88</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_18" model="event.track">
        <!-- 6 Woodworking tips and tricks for beginners -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=3cME3vK1aaQ</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_19" model="event.track">
        <!-- How I Transformed this Old Block Wall - DIY Timber Cladding Project -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=7X8xuipr4_A</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_22" model="event.track">
        <!-- Basic Set of Tools for the Woodworking Beginner -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=3pc-nnAykTQ</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
    <record id="website_event_track.event_7_track_23" model="event.track">
        <!-- Restoring Old Woodworking Tools -->
        <field name="youtube_video_url">https://www.youtube.com/watch?v=tI5zCo-zLkY</field>
        <field name="is_youtube_replay" eval="True"/>
    </record>
</odoo>

```

## File: models\event_track.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models


class Track(models.Model):
    _inherit = 'event.track'

    youtube_video_url = fields.Char('YouTube Video Link')
    youtube_video_id = fields.Char('YouTube video ID', compute='_compute_youtube_video_id',
        help="Extracted from the video URL and used to infer various links (embed/thumbnail/...)")
    is_youtube_replay = fields.Boolean('Is YouTube Replay',
        help="Check this option if the video is already available on YouTube to avoid showing 'Direct' options (Chat, ...)")
    is_youtube_chat_available = fields.Boolean('Is Chat Available', compute='_compute_is_youtube_chat_available')

    @api.depends('youtube_video_url')
    def _compute_youtube_video_id(self):
        for track in self:
            if track.youtube_video_url:
                regex = r'^.*(youtu.be\/|v\/|u\/\w\/|embed\/|live\/|watch\?v=|&v=)([^#&?]*).*'
                match = re.match(regex, track.youtube_video_url)
                if match and len(match.groups()) == 2 and len(match.group(2)) == 11:
                    track.youtube_video_id = match.group(2)

            if not track.youtube_video_id:
                track.youtube_video_id = False

    @api.depends('youtube_video_id', 'is_youtube_replay', 'date_end', 'is_track_done')
    def _compute_website_image_url(self):
        youtube_thumbnail_tracks = self.filtered(lambda track: not track.website_image and track.youtube_video_id)
        super(Track, self - youtube_thumbnail_tracks)._compute_website_image_url()
        for track in youtube_thumbnail_tracks:
            track.website_image_url = f'https://img.youtube.com/vi/{track.youtube_video_id}/maxresdefault.jpg'

    @api.depends('youtube_video_url', 'is_youtube_replay', 'date', 'date_end', 'is_track_upcoming', 'is_track_live')
    def _compute_is_youtube_chat_available(self):
        for track in self:
            track.is_youtube_chat_available = track.youtube_video_url and not track.is_youtube_replay and (track.is_track_soon or track.is_track_live)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_track

```

## File: static\src\js\website_event_track_live.js

```javascript
/** @odoo-module **/
/* global YT */

import publicWidget from "@web/legacy/js/public/public_widget";
import TrackSuggestionWidget from "@website_event_track_live/js/website_event_track_suggestion";
import ReplaySuggestionWidget from "@website_event_track_live/js/website_event_track_replay_suggestion";
import { rpc } from "@web/core/network/rpc";

publicWidget.registry.websiteEventTrackLive = publicWidget.Widget.extend({
    selector: '.o_wevent_event_track_live',
    custom_events: Object.assign({}, publicWidget.Widget.prototype.custom_events, {
        'video-ended': '_onVideoEnded'
    }),

    start: function () {
        var self = this;
        return this._super(...arguments).then(function () {
            self._setupYoutubePlayer();
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onPlayerReady: function () {
        this.$('.o_wevent_event_track_live_loading').remove();
    },

    _onPlayerStateChange: function (event) {
        switch (event.data) {
            case YT.PlayerState.ENDED:
                this.trigger('video-ended');
                return;
            case YT.PlayerState.PLAYING:
                this.trigger('video-playing');
                return;
            case YT.PlayerState.PAUSED:
                this.trigger('video-paused');
                return;
        };
    },

    _onVideoEnded: function () {
        this.$el.append($('<div/>', {
            class: 'owevent_track_suggestion_loading position-absolute w-100'
        }));
        var self = this;
        rpc('/event_track/get_track_suggestion', {
            track_id: this.$el.data('trackId'),
        }).then(function (suggestion) {
            self.nextSuggestion = suggestion;
            self._showSuggestion();
        });
    },

    _onReplay: function () {
        this.youtubePlayer.seekTo(0);
        this.youtubePlayer.playVideo();
        this.$('.owevent_track_suggestion_loading').remove();
        if (this.outro) {
            delete this.outro;
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _setupYoutubePlayer: function () {
        var self = this;

        var youtubeId = self.$el.data('youtubeVideoId');
        var $youtubeElement = $('<script/>', {src: 'https://www.youtube.com/iframe_api'});
        $(document.head).append($youtubeElement);

        window.onYouTubeIframeAPIReady = function () {
            self.youtubePlayer = new YT.Player('o_wevent_youtube_iframe_container', {
                height: '100%',
                width: '100%',
                videoId: youtubeId,
                playerVars: {
                    autoplay: 1,
                    enablejsapi: 1,
                    rel: 0,
                    origin: window.location.origin,
                    widget_referrer: window.location.origin,
                },
                events: {
                    'onReady': self._onPlayerReady.bind(self),
                    'onStateChange': self._onPlayerStateChange.bind(self)
                }
            });
        };
    },

    /**
     * If a new suggestion has been found, a cover containing a replay button
     * as well as a suggestion will automatically be placed over the Youtube
     * player when the video ends (in non-full screen mode). If no suggestion
     * has been found, the cover will only contain a replay button.
     */
    _showSuggestion: function () {
        if (!this.outro) {
            if (this.nextSuggestion) {
                this.outro = new TrackSuggestionWidget(this, this.nextSuggestion);
            } else {
                var data = this.$el.data();
                this.outro = new ReplaySuggestionWidget(this, {
                    current_track: {
                        name: data.trackName,
                        website_image_url: data.trackWebsiteImageUrl
                    }
                });
            }
            this.outro.appendTo(this.$el);
            this.outro.on('replay', null, this._onReplay.bind(this));
        }
    }
});

```

## File: static\src\js\website_event_track_replay_suggestion.js

```javascript
/** @odoo-module **/

import { PublicWidget } from "@web/legacy/js/public/public_widget";

/**
 * The widget will have the responsibility to manage the interactions between the
 * Youtube player and the cover containing a replay button. This widget will
 * be used when no suggestion can be found in order to hide the Youtube suggestions.
 */
var WebsiteEventReplaySuggestion = PublicWidget.extend({
    template: 'website_event_track_live.website_event_track_replay_suggestion',
    events: {
        'click .owevent_track_suggestion_replay': '_onReplayClick'
    },

    init: function (parent, options) {
        this._super(...arguments);
        this.currentTrack = {
            'name': options.current_track.name,
            'imageSrc': options.current_track.website_image_url,
        };
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * If the user clicks on the replay button, the function will remove the
     * cover and send a new event to the parent to replay the video from the
     * beginning.
     */
    _onReplayClick: function () {
        this.trigger_up('replay');
        this.destroy();
    }
});

export default WebsiteEventReplaySuggestion;

```

## File: static\src\js\website_event_track_suggestion.js

```javascript
/** @odoo-module **/

import { PublicWidget } from "@web/legacy/js/public/public_widget";

var WebsiteEventTrackSuggestion = PublicWidget.extend({
    template: 'website_event_track_live.website_event_track_suggestion',
    events: {
        'click .owevent_track_suggestion_next': '_onNextTrackClick',
        'click .owevent_track_suggestion_close': '_onCloseClick',
        'click .owevent_track_suggestion_replay': '_onReplayClick'
    },

    init: function (parent, options) {
        this._super(...arguments);

        this.currentTrack = {
            'name': options.current_track.name,
            'imageSrc': options.current_track.website_image_url,
        };
        this.suggestion = {
            'name': options.suggestion.name,
            'speakerName': options.suggestion.speaker_name,
            'trackUrl': options.suggestion.website_url,
        };
    },

    start: function () {
        var self = this;
        this._super(...arguments).then(function () {
            self.timerInterval = setInterval(self._updateTimer.bind(self), 1000);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * If the user clicks on replay, remove this suggestion window and send an
     * event to the parent so that it can rewind the video to the beginning.
     */
    _onReplayClick: function () {
        this.trigger_up('replay');
        clearInterval(this.timerInterval);
        this.destroy();
    },

    _onCloseClick: function () {
        clearInterval(this.timerInterval);
        this.$('.owevent_track_suggestion_next').addClass('invisible');
    },

    _onNextTrackClick: function (ev) {
        if ($(ev.target).hasClass('owevent_track_suggestion_close')) {
            return;
        }

        window.location = this.suggestion.trackUrl;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _updateTimer: function () {
        var secondsLeft = parseInt(this.$('.owevent_track_suggestion_timer_text').text());

        if (secondsLeft > 1) {
            secondsLeft -= 1;
            this.$('.owevent_track_suggestion_timer_text').text(secondsLeft);
        } else {
            window.location = this.suggestion.trackUrl;
        }
    }
});

export default WebsiteEventTrackSuggestion;

```

## File: static\src\xml\website_event_track_live_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

<div t-name="website_event_track_live.website_event_track_replay_suggestion"
    class="owevent_track_suggestion position-absolute w-100"
    t-attf-style="background-image: url('#{widget.currentTrack.imageSrc}');">
    <div class="h-100 w-100 p-4 d-flex flex-column align-items-center owevent_track_suggestion_content">
        <div class="flex-grow-1 pt-5 d-flex flex-column align-items-center justify-content-center">
            <div class="text-white">You just watched:</div>
            <div class="h1 text-white" t-out="widget.currentTrack.name"/>
            <div>
                <a href="#" class="owevent_track_suggestion_replay fw-bold">
                    <span>Replay Video</span>
                </a>
            </div>
        </div>
    </div>
</div>

<div t-name="website_event_track_live.website_event_track_suggestion"
    class="owevent_track_suggestion position-absolute w-100"
    t-attf-style="background-image: url('#{widget.currentTrack.imageSrc}');">
    <div class="h-100 w-100 p-4 d-flex flex-column align-items-center owevent_track_suggestion_content">
        <div class="flex-grow-1 pt-5 d-flex flex-column align-items-center justify-content-center">
            <div class="text-white">You just watched:</div>
            <div class="h1 text-white" t-out="widget.currentTrack.name"/>
            <div>
                <a href="#" class="owevent_track_suggestion_replay fw-bold">
                    <span>Replay Video</span>
                </a>
            </div>
        </div>
        <div class="d-flex w-100 flex-column align-items-end">
            <div class="p-2 rounded owevent_track_suggestion_next position-relative">
                <div class="text-white text-end owevent_track_suggestion_close_wrapper">
                    <i class="fa fa-remove owevent_track_suggestion_close position-absolute"/>
                </div>
                <span class="text-white">Up Next:</span>
                <span class="text-primary fw-bold pe-4" t-out="widget.suggestion.name"/>
                <div class="text-white owevent_track_suggestion_timer_text_wrapper">
                    <span>Starts in</span>
                    <span class="owevent_track_suggestion_timer_text">30</span>
                </div>
            </div>
        </div>
    </div>
</div>

</templates>

```

## File: views\event_track_templates_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="tracks_display_list" inherit_id="website_event_track.tracks_display_list">
    <!-- TRACK LIST: ADD REPLAY TAG FOR FINISHED TRACKS -->
    <xpath expr="//div[@t-foreach='tracks']//div[hasclass('col-12')]//span[@t-elif='not track.is_track_done and not track.is_track_soon']" position="after">
        <a t-elif="track.youtube_video_url and (track.is_published or is_event_user)"
            t-att-href="track.website_url" class="badge text-bg-danger">Replay
        </a>
    </xpath>
    <!-- ADD YOUTUBE ICON -->
    <xpath expr="//div[@t-foreach='tracks']//div[hasclass('col-12')]//a/span[@t-field='track.name']" position="before">
        <i t-if="track.date and track.youtube_video_url and (track.is_track_soon or track.is_track_live or track.is_youtube_replay)"
            class="fa fa-youtube-play text-danger me-1"/>
    </xpath>
    <xpath expr="//div[@t-foreach='tracks']//div[hasclass('col-12')]//t/span[@t-field='track.name']" position="before">
        <i t-if="track.date and track.youtube_video_url and (track.is_track_soon or track.is_track_live or track.is_youtube_replay)"
            class="fa fa-youtube-play text-danger me-1"/>
    </xpath>
</template>

</odoo>

```

## File: views\event_track_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- MAIN: REMOVE HEADER/FOOTER IF WIDESCREEN -->
<template id="event_track_main" inherit_id="website_event_track.event_track_main">
    <xpath expr="//t[@t-call='website_event.layout']" position="before">
        <t t-set="no_header" t-value="option_widescreen"/>
        <t t-set="no_footer" t-value="option_widescreen"/>
    </xpath>
</template>

<!-- MAIN: ADD VIDEO -->
<template id="event_track_content" inherit_id="website_event_track.event_track_content">
    <!-- Do not display "starting in" in case of replay mode as video is already available -->
    <xpath expr="//div[@t-elif='track.track_start_remaining']" position="attributes">
        <attribute name="t-elif">track.track_start_remaining and not track.is_youtube_replay</attribute>
    </xpath>
    <xpath expr="//div[@name='o_wesession_track_main']" position="attributes">
        <attribute name="t-attf-class" add="{{'o_youtube_chat' if track.is_youtube_chat_available else ''}}" separator=" "/>
    </xpath>
    <!-- Add video -->
    <xpath expr="//div[@name='o_wesession_track_main']//div" position="before">
        <t t-set="show_youtube_frame" t-value="track.youtube_video_url and (track.is_youtube_replay or track.is_track_soon or track.is_track_live or track.is_track_done)" />
        <div t-if="show_youtube_frame" class="flex-grow-1 o_wevent_event_track_live position-relative"
            t-att-data-track-id="track.id"
            t-att-data-track-name="track.name"
            t-att-data-track-website-image-url="track.website_image_url"
            t-att-data-youtube-video-id="track.youtube_video_id">
            <div class="position-absolute o_wevent_event_track_live_loading w-100 d-flex align-items-center justify-content-center text-white h4">
                <i class="fa fa-spin fa-circle-o-notch position-relative"/>
                <span class="ps-2">Loading Video...</span>
            </div>
            <div id="o_wevent_youtube_iframe_container" class="w-100"/>
        </div>
    </xpath>
</template>

<!-- ASIDE: ADD CHAT TAB FOR VIDEOS -->
<template id="event_track_aside" inherit_id="website_event_track.event_track_aside">
    <xpath expr="//div[@name='o_wevent_online_page_aside']" position="attributes">
        <attribute name="t-attf-class" add="{{'o_youtube_chat' if track.is_youtube_chat_available else ''}}" separator=" "/>
    </xpath>
    <xpath expr="//ul[hasclass('o_wesession_track_aside_nav')]/li" position="before">
        <li t-if="track.is_youtube_chat_available and not is_mobile_chat_disabled" class="nav-item flex-grow-1">
            <a href="#track_chat" aria-controls="track_chat" class="nav-link active" role="tab" data-bs-toggle="tab">
                Chat
            </a>
        </li>
    </xpath>
    <xpath expr="//a[@href='#track_list']" position="attributes">
        <attribute name="t-attf-class">#{'nav-link' if track.is_youtube_chat_available and not is_mobile_chat_disabled else 'nav-link active'}</attribute>
    </xpath>
    <xpath expr="//div[hasclass('o_wesession_track_aside_tabs')]" position="inside">
        <div t-if="track.is_youtube_chat_available and not is_mobile_chat_disabled" id="track_chat"
            class="tab-pane fade show active o_wesession_track_aside_tab_chat" role="tabpanel">
            <iframe t-attf-src="https://www.youtube.com/live_chat?v=#{track.youtube_video_id}&amp;embed_domain=#{hostname}"
                height="100%"
                width="100%"
                frameborder="0" allowfullscreen="allowfullscreen"/>
        </div>
    </xpath>
    <xpath expr="//div[@id='track_list']" position="attributes">
        <attribute name="t-attf-class">#{'tab-pane fade' if track.is_youtube_chat_available and not is_mobile_chat_disabled else 'tab-pane fade show active'}</attribute>
    </xpath>
</template>

</odoo>

```

## File: views\event_track_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="event_track_view_form" model="ir.ui.view">
        <field name="name">event.track.view.form.inherit.live</field>
        <field name="model">event.track</field>
        <field name="inherit_id" ref="website_event_track.view_event_track_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='active']" position="after">
                <field name="youtube_video_url" widget="url" />
                <field name="is_youtube_replay"
                    invisible="not youtube_video_url or youtube_video_url == ''" />
            </xpath>
        </field>
    </record>

</odoo>

```

