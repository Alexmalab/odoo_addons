# Odoo Module: website_event_track_live_quiz

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Quiz on Live Event Tracks',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Bridge module to support quiz features during "live" tracks. ',
    'website': 'https://www.odoo.com/app/events',
    'depends': [
        'website_event_track_live',
        'website_event_track_quiz',
    ],
    'data': [
        'views/event_track_templates_page.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_event_track_live_quiz/static/src/js/**/*',
            'website_event_track_live_quiz/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\track_live_quiz.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_event_track_live.controllers.track_live import EventTrackLiveController


class EventTrackLiveQuizController(EventTrackLiveController):

    def _prepare_track_suggestion_values(self, track, track_suggestion):
        res = super(EventTrackLiveQuizController, self)._prepare_track_suggestion_values(track, track_suggestion)
        res['current_track']['show_quiz'] = bool(track.quiz_id) and not track.is_quiz_completed
        return res

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import track_live_quiz

```

## File: static\src\js\event_quiz.js

```javascript
/** @odoo-module **/

import Quiz from "@website_event_track_quiz/js/event_quiz";

var WebsiteEventTrackSuggestionQuiz = Quiz.include({
    /**
     * @override
     */
    willStart: function () {
        return Promise.all([
            this._super(...arguments),
            this._getTrackSuggestion()
        ]);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _submitQuiz: function () {
        var self = this;
        return this._super(...arguments).then(function (data) {
            if (data.quiz_completed) {
                self.$('.o_quiz_js_quiz_next_track')
                    .removeClass('btn-light')
                    .addClass('btn-secondary');
            }

            return Promise.resolve(data);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _getTrackSuggestion: function () {
        var self = this;
        return this.rpc('/event_track/get_track_suggestion', {
            track_id: this.track.id,
        }).then(function (suggestion) {
            self.nextSuggestion = suggestion;
            return Promise.resolve();
        });
    }
});

export default WebsiteEventTrackSuggestionQuiz;

```

## File: static\src\js\website_event_track_suggestion.js

```javascript
/** @odoo-module **/

import WebsiteEventTrackSuggestion from "@website_event_track_live/js/website_event_track_suggestion";

var WebsiteEventTrackSuggestionLiveQuiz = WebsiteEventTrackSuggestion.include({
    events: Object.assign({}, WebsiteEventTrackSuggestion.prototype.events, {
        'click .owevent_track_suggestion_quiz': '_onQuizClick'
    }),

    init: function (parent, options) {
        this._super(...arguments);
        this.currentTrack.showQuiz = options.current_track.show_quiz;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * If the user takes the quiz, stop the next suggestion timer
     */
    _onQuizClick: function () {
        clearInterval(this.timerInterval);
        this.$('.owevent_track_suggestion_timer_text_wrapper').remove();
    }
});

export default WebsiteEventTrackSuggestionLiveQuiz;

```

## File: static\src\xml\website_event_track_live_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

<t t-inherit="website_event_track_live.website_event_track_suggestion" t-inherit-mode="extension">
    <xpath expr="//a[hasclass('owevent_track_suggestion_replay')]" position="before">
        <a t-if="widget.currentTrack.showQuiz" class="btn btn-primary btn-lg me-2 owevent_track_suggestion_quiz"
           href="#we_track_quiz_container"
           onclick="$('.o_quiz_js_quiz_container').removeClass('d-none');">
            <span>Take the Quiz</span>
        </a>
    </xpath>
</t>

</odoo>

```

## File: static\src\xml\website_event_track_quiz_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

<t t-inherit="website_event_track_quiz.quiz.validation" t-inherit-mode="extension">
    <xpath expr="//button[hasclass('o_quiz_js_quiz_reset')]" position="after">
        <a t-if="widget.nextSuggestion"
            t-attf-class="btn border o_quiz_js_quiz_next_track #{widget.track.completed ? 'btn-secondary' : 'btn-light'}"
            t-att-href="widget.nextSuggestion.suggestion.website_url">
            Next Track
        </a>
    </xpath>
</t>

</odoo>

```

## File: views\event_track_templates_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="event_track_content"
    name="Main Description: tweak quiz"
    inherit_id="website_event_track_quiz.event_track_content">
    <xpath expr="//div[hasclass('o_we_track_quiz_button')]" position="attributes">
        <attribute name="t-if">track.quiz_id and not track.is_quiz_completed and not track.is_track_upcoming and (not track.is_track_live or track.is_youtube_replay or not track.youtube_video_id)</attribute>
    </xpath>
</template>

</odoo>

```

