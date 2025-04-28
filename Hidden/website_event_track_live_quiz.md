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
    'website': 'https://www.odoo.com/page/events',
    'description': "",
    'depends': [
        'website_event_track_live',
        'website_event_track_quiz',
    ],
    'data': [
        'views/assets.xml',
        'views/event_track_templates_page.xml',
    ],
    'demo': [
    ],
    'qweb': [
        'static/src/xml/website_event_track_live_templates.xml',
        'static/src/xml/website_event_track_quiz_templates.xml',
    ],
    'application': False,
    'installable': True,
    'auto_install': True,
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
odoo.define('website_event_track_live_quiz.event_quiz', function (require) {
'use strict';

var Quiz = require('website_event_track_quiz.event.quiz');

var WebsiteEventTrackSuggestionQuiz = Quiz.include({
    xmlDependencies: Quiz.prototype.xmlDependencies.concat([
        '/website_event_track_live_quiz/static/src/xml/website_event_track_quiz_templates.xml',
    ]),

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
        return this._rpc({
            route: '/event_track/get_track_suggestion',
            params: {
                track_id: this.track.id,
            }
        }).then(function (suggestion) {
            self.nextSuggestion = suggestion;
            return Promise.resolve();
        });
    }
});

return WebsiteEventTrackSuggestionQuiz;

});

```

## File: static\src\js\website_event_track_suggestion.js

```javascript
odoo.define('website_event_track_live_quiz.website_event_track_suggestion', function (require) {
'use strict';

var WebsiteEventTrackSuggestion = require('website_event_track_live.website_event_track_suggestion');

var WebsiteEventTrackSuggestionLiveQuiz = WebsiteEventTrackSuggestion.include({
    xmlDependencies: WebsiteEventTrackSuggestion.prototype.xmlDependencies.concat([
        '/website_event_track_live_quiz/static/src/xml/website_event_track_live_templates.xml',
    ]),
    events: _.extend({}, WebsiteEventTrackSuggestion.prototype.events, {
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

return WebsiteEventTrackSuggestionLiveQuiz;

});

```

## File: static\src\xml\website_event_track_live_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

<t t-extend="website_event_track_live.website_event_track_suggestion">
    <t t-jquery="a.owevent_track_suggestion_replay" t-operation="before">
        <a t-if="widget.currentTrack.showQuiz" class="btn btn-primary btn-lg mr-2 owevent_track_suggestion_quiz"
           href="#we_track_quiz_container"
           onclick="$('.o_quiz_js_quiz_container').removeClass('d-none');">
            <span>Take the Quiz</span>
        </a>
    </t>
</t>

</odoo>

```

## File: static\src\xml\website_event_track_quiz_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

<t t-extend="quiz.validation">
    <t t-jquery="button.o_quiz_js_quiz_reset" t-operation="after">
        <a t-if="widget.nextSuggestion"
            t-attf-class="btn border o_quiz_js_quiz_next_track #{widget.track.completed ? 'btn-secondary' : 'btn-light'}"
            t-att-href="widget.nextSuggestion.suggestion.website_url">
            Next Track
        </a>
    </t>
</t>

</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend" name="Event Track Live Quiz Assets Frontend">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_event_track_live_quiz/static/src/js/website_event_track_suggestion.js"></script>
        <script type="text/javascript" src="/website_event_track_live_quiz/static/src/js/event_quiz.js"></script>
    </xpath>
</template>

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

