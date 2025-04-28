# Odoo Module: test_event_full

Category: Hidden/Tests

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Test Full Event Flow',
    'version': '1.0',
    'category': 'Hidden/Tests',
    'description': """
This module will test the main event flows of Odoo, both frontend and backend.
It installs sale capabilities, front-end flow, eCommerce, questions and
automatic lead generation, full Online support, ...
""",
    'depends': [
        'event',
        'event_booth',
        'event_crm',
        'event_sale',
        'website_event_booth_sale_exhibitor',
        'website_event_crm_questions',
        'website_event_exhibitor',
        'website_event_questions',
        'website_event_meet',
        'website_event_sale',
        'website_event_track',
        'website_event_track_live',
        'website_event_track_quiz',
    ],
    'installable': True,
    'assets': {
        'web.assets_tests': [
            'test_event_full/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: static\src\js\tours\wevent_register_tour.js

```javascript
odoo.define('test_event_full.tour.register', function (require) {
"use strict";

var tour = require('web_tour.tour');

/**
 * TALKS STEPS
 */

var discoverTalkSteps = function (talkName, fromList, reminderOn, toggleReminder) {
    var steps;
    if (fromList) {
        steps = [{
            content: 'Go on "' + talkName + '" talk in List',
            trigger: 'a:contains("' + talkName + '")',
        }];
    }
    else {
        steps = [{
            content: 'Click on Live Track',
            trigger: 'article span:contains("' + talkName + '")',
            run: 'click',
        }];
    }
    if (reminderOn) {
        steps = steps.concat([{
            content: "Check Favorite is on",
            trigger: 'div.o_wetrack_js_reminder i.fa-bell',
            extra_trigger: 'span.o_wetrack_js_reminder_text:contains("Favorite On")',
            run: function () {}, // it's a check
        }]);
    }
    else {
        steps = steps.concat([{
            content: "Check Favorite is Off",
            trigger: 'span.o_wetrack_js_reminder_text:contains("Set Favorite")',
            run: function () {}, // it's a check
        }]);
        if (toggleReminder) {
            steps = steps.concat([{
                content: "Set Favorite",
                trigger: 'span.o_wetrack_js_reminder_text',
                run: 'click',
            }, {
                content: "Check Favorite is On",
                trigger: 'div.o_wetrack_js_reminder i.fa-bell',
                extra_trigger: 'span.o_wetrack_js_reminder_text:contains("Favorite On")',
                run: function () {}, // it's a check
            }]);
        }
    }
    return steps;
};


/**
 * ROOMS STEPS
 */

var discoverRoomSteps = function (roomName) {
    var steps = [{
        content: 'Go on "' + roomName + '" room in List',
        trigger: 'a.o_wevent_meeting_room_card h4:contains("' + roomName + '")',
        run: function() {
            // can't click on it, it will try to launch Jitsi and fail on chrome headless
        },
    }];
    return steps;
};


/**
 * REGISTER STEPS
 */

var registerSteps = [{
    content: 'Go on Register',
    trigger: 'a.btn-primary:contains("Register")',
}, {
    content: "Select 2 units of 'Standard' ticket type",
    trigger: '#o_wevent_tickets_collapse .row:has(.o_wevent_registration_multi_select:contains("Free")) select',
    run: 'text 2',
}, {
    content: "Click on 'Register' button",
    trigger: '#o_wevent_tickets .btn-primary:contains("Register"):not(:disabled)',
    run: 'click',
}, {
    content: "Fill attendees details",
    trigger: 'form[id="attendee_registration"] .btn:contains("Continue")',
    run: function () {
        $("input[name='1-name']").val("Raoulette Poiluchette");
        $("input[name='1-phone']").val("0456112233");
        $("input[name='1-email']").val("raoulette@example.com");
        $("select[name*='question_answer-1']").val($("select[name*='question_answer-1'] option:contains('Consumers')").val());
        $("input[name='2-name']").val("Michel Tractopelle");
        $("input[name='2-phone']").val("0456332211");
        $("input[name='2-email']").val("michel@example.com");
        $("select[name*='question_answer-2']").val($("select[name*='question_answer-1'] option:contains('Research')").val());
        $("textarea[name*='question_answer']").text("An unicorn told me about you. I ate it afterwards.");
    },
}, {
    content: "Validate attendees details",
    extra_trigger: "input[name='1-name'], input[name='2-name'], input[name='3-name']",
    trigger: 'button:contains("Continue")',
    run: 'click',
}, {
    trigger: 'div.o_wereg_confirmed_attendees span:contains("Raoulette Poiluchette")',
    run: function () {} // check
}, {
    trigger: 'div.o_wereg_confirmed_attendees span:contains("Michel Tractopelle")',
    run: function () {} // check
},  {
    content: "Click on 'register favorites talks' button",
    trigger: 'a:contains("register to your favorites talks now")',
    run: 'click',
},  {
    trigger: 'h1:contains("Book your talks")',
    run: function() {},
}];

/**
 * MAIN STEPS
 */

var initTourSteps = function (eventName) {
    return [{
        content: 'Go on "' + eventName + '" page',
        trigger: 'a[href*="/event"]:contains("' + eventName + '"):first',
    }];
};

var browseTalksSteps = [{
    content: 'Browse Talks',
    trigger: 'a:contains("Talks")',
}];

var browseExhibitorsSteps = [{
    content: 'Browse Exhibitors',
    trigger: 'a:contains("Exhibitors")',
}];

var browseMeetSteps = [{
    content: 'Browse Meet',
    trigger: 'a:contains("Community")',
}];


tour.register('wevent_register', {
    url: '/event',
    test: true
}, [].concat(
        initTourSteps('Online Reveal'),
        browseTalksSteps,
        discoverTalkSteps('What This Event Is All About', true, true),
        browseTalksSteps,
        discoverTalkSteps('Live Testimonial', false, false, false),
        browseTalksSteps,
        discoverTalkSteps('Our Last Day Together !', true, false, true),
        browseMeetSteps,
        discoverRoomSteps('Best wood for furniture'),
        registerSteps,
    )
);

});

```

