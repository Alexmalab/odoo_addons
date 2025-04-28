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
        'event_crm_sale',
        'event_sale',
        'event_sms',
        'payment_demo',
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
    'data': [
        # 'data/event_type_data.xml',  # uncomment to reproduce test tour
        'data/ir_actions_report_data.xml',
        'views/event_registration_templates_reports.xml',
    ],
    'assets': {
        'web.assets_tests': [
            'test_event_full/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\event_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="0">

    <record id="event_booth_category_data_1" model="event.booth.category">
        <field name="description" type="html"><p>Standard</p></field>
        <field name="name">Standard</field>
        <field name="product_id" ref="event_booth_sale.product_product_event_booth"/>
    </record>
    <record id="event_booth_category_data_2" model="event.booth.category">
        <field name="description" type="html"><p>Premium</p></field>
        <field name="name">Premium</field>
        <field name="product_id" ref="event_booth_sale.product_product_event_booth"/>
        <field name="price">90</field>
    </record>

    <record id="event_type_data_full" model="event.type">
        <field name="auto_confirm" eval="True"/>
        <field name="default_timezone">Europe/Paris</field>
        <field name="event_type_booth_ids" eval="[
            (5, 0),
            (0, 0, {'booth_category_id': ref('test_event_full.event_booth_category_data_1'),
                    'name': 'Standard Booth',
                   }
            ),
            (0, 0, {'booth_category_id': ref('test_event_full.event_booth_category_data_1'),
                    'name': 'Standard Booth 2',
                   }
            ),
            (0, 0, {'booth_category_id': ref('test_event_full.event_booth_category_data_2'),
                    'name': 'Premium Booth',
                   }
            ),
            (0, 0, {'booth_category_id': ref('test_event_full.event_booth_category_data_2'),
                    'name': 'Premium Booth 2',
                   }
            )]"/>
        <field name="event_type_mail_ids" eval="[
            (5, 0),
            (0, 0, {'interval_unit': 'now',
                    'interval_type': 'after_sub',
                    'notification_type': 'mail',
                    'template_ref': 'mail.template,%i' % ref('event.event_subscription'),
                   }
            ),
            (0, 0, {'interval_nbr': 1,
                    'interval_unit': 'days',
                    'interval_type': 'before_event',
                    'notification_type': 'mail',
                    'template_ref': 'mail.template,%i' % ref('event.event_reminder'),
                   }
            ),
            (0, 0, {'interval_nbr': 1,
                    'interval_unit': 'days',
                    'interval_type': 'after_event',
                    'notification_type': 'sms',
                    'template_ref': 'sms.template,%i' % ref('event_sms.sms_template_data_event_reminder'),
                   }
            )]"/>
        <field name="event_type_ticket_ids" eval="[
            (5, 0),
            (0, 0, {'description': 'Ticket1 Description',
                    'name': 'Ticket1',
                    'product_id': ref('event_sale.product_product_event'),
                    'seats_max': 10,
                   }
            ),
            (0, 0, {'description': 'Ticket2 Description',
                    'name': 'Ticket2',
                    'product_id': ref('event_sale.product_product_event'),
                    'price': 45,
                   }
            )]"/>
        <field name="has_seats_limitation" eval="True"/>
        <field name="name">Test Type</field>
        <field name="note" type="html"><p>Template note</p></field>
        <field name="question_ids" eval="[(5, 0)]"/>
        <field name="seats_max">30</field>
        <field name="tag_ids" eval="[(5, 0)]"/>
        <field name="ticket_instructions" type="html"><p>Ticket Instructions</p></field>
        <field name="website_menu" eval="True"/>
    </record>

    <record id="event_question_type_full_1" model="event.question">
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_type_id" ref="test_event_full.event_type_data_full"/>
        <field name="title">Question1</field>
    </record>
    <record id="event_question_type_full_1_answer_1" model="event.question.answer">
        <field name="name">Q1-Answer1</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="test_event_full.event_question_type_full_1"/>
    </record>
    <record id="event_question_type_full_1_answer_2" model="event.question.answer">
        <field name="name">Q1-Answer2</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="test_event_full.event_question_type_full_1"/>
    </record>
    <record id="event_question_type_full_2" model="event.question">
        <field name="question_type">simple_choice</field>
        <field name="once_per_order" eval="False"/>
        <field name="event_type_id" ref="test_event_full.event_type_data_full"/>
        <field name="title">Question2</field>
    </record>
    <record id="event_question_type_full_2_answer_1" model="event.question.answer">
        <field name="name">Q2-Answer1</field>
        <field name="sequence">1</field>
        <field name="question_id" ref="test_event_full.event_question_type_full_2"/>
    </record>
    <record id="event_question_type_full_2_answer_2" model="event.question.answer">
        <field name="name">Q2-Answer2</field>
        <field name="sequence">2</field>
        <field name="question_id" ref="test_event_full.event_question_type_full_2"/>
    </record>
    <record id="event_question_type_full_3" model="event.question">
        <field name="question_type">text_box</field>
        <field name="once_per_order" eval="True"/>
        <field name="event_type_id" ref="test_event_full.event_type_data_full"/>
        <field name="title">Question3</field>
    </record>

</data></odoo>

```

## File: data\ir_actions_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="event_registration_report_test" model="ir.actions.report">
        <field name="name">Test Report</field>
        <field name="model">event.registration</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">test_event_full.event_registration_template_report</field>
        <field name="report_file">test_event_full.event_registration_template_report</field>
        <field name="print_report_name">'Badge - %s - %s' % ((object.event_id.name or 'Event').replace('/',''), (object.name or '').replace('/',''))</field>
        <field name="binding_model_id" ref="event.model_event_registration"/>
        <field name="binding_type">report</field>
    </record>
</odoo>

```

## File: static\src\js\tours\wevent_performance_tour.js

```javascript
odoo.define('test_event_full.tour.performance', function (require) {
"use strict";

var tour = require('web_tour.tour');

var registerSteps = [{
    content: "Select 2 units of 'Ticket1' ticket type",
    trigger: '#o_wevent_tickets_collapse .row.o_wevent_ticket_selector[name="Ticket1"] select',
    run: 'text 2',
}, {
    content: "Select 1 unit of 'Ticket2' ticket type",
    trigger: '#o_wevent_tickets_collapse .row.o_wevent_ticket_selector[name="Ticket2"] select',
    run: 'text 1',
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
        $("div[name*='Question1'] select[name*='question_answer-1']").val($("select[name*='question_answer-1'] option:contains('Q1-Answer2')").val());
        $("div[name*='Question2'] select[name*='question_answer-1']").val($("select[name*='question_answer-1'] option:contains('Q2-Answer1')").val());
        $("input[name='2-name']").val("Michel Tractopelle");
        $("input[name='2-phone']").val("0456332211");
        $("input[name='2-email']").val("michel@example.com");
        $("div[name*='Question1'] select[name*='question_answer-2']").val($("select[name*='question_answer-2'] option:contains('Q1-Answer1')").val());
        $("div[name*='Question2'] select[name*='question_answer-2']").val($("select[name*='question_answer-2'] option:contains('Q2-Answer2')").val());
        $("input[name='3-name']").val("Hubert Boitaclous");
        $("input[name='3-phone']").val("0456995511");
        $("input[name='3-email']").val("hubert@example.com");
        $("div[name*='Question1'] select[name*='question_answer-3']").val($("select[name*='question_answer-3'] option:contains('Q1-Answer2')").val());
        $("div[name*='Question2'] select[name*='question_answer-3']").val($("select[name*='question_answer-3'] option:contains('Q2-Answer2')").val());
        $("textarea[name*='question_answer']").text("Random answer from random guy");
    },
}, {
    content: "Validate attendees details",
    extra_trigger: "input[name='1-name'], input[name='2-name'], input[name='3-name']",
    trigger: 'button:contains("Continue")',
    run: 'click',
}, {
    content: "Address filling",
    trigger: 'select[name="country_id"]',
    run: function () {
        $('input[name="name"]').val('Raoulette Poiluchette');
        $('input[name="phone"]').val('0456112233');
        $('input[name="email"]').val('raoulette@example.com');
        $('input[name="street"]').val('Cheesy Crust Street, 42');
        $('input[name="city"]').val('CheeseCity');
        $('input[name="zip"]').val('8888');
        $('#country_id option:eq(1)').attr('selected', true);
    },
}, {
    content: "Next",
    trigger: '.oe_cart .btn:contains("Next")',
}, {
    content: 'Select Test payment provider',
    trigger: '.o_payment_option_card:contains("Demo")'
}, {
    content: 'Add card number',
    trigger: 'input[name="customer_input"]',
    run: 'text 4242424242424242'
}, {
    content: "Pay now",
    extra_trigger: "#cart_products:contains(Ticket1):contains(Ticket2)",
    trigger: 'button:contains(Pay Now)',
    run: 'click',
}, {
    content: 'Payment is successful',
    trigger: '.oe_website_sale_tx_status:contains("Your payment has been successfully processed.")',
    run: function () {}
}];


tour.register('wevent_performance_register', {
    test: true
}, [].concat(
        registerSteps,
    )
);

});

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
    steps = steps.concat([{
        content: `Check we are on the "${talkName}" talk page`,
        trigger: 'div.o_wesession_track_main',
        run: function () {}, // it's a check
    }]);

    if (reminderOn) {
        steps = steps.concat([{
            content: `Check Favorite for ${talkName} was already on`,
            trigger: 'div.o_wetrack_js_reminder i.fa-bell',
            extra_trigger: 'span.o_wetrack_js_reminder_text:contains("Favorite On")',
            run: function () {}, // it's a check
        }]);
    }
    else {
        steps = steps.concat([{
            content: `Check Favorite for ${talkName} was off`,
            trigger: 'span.o_wetrack_js_reminder_text:contains("Set Favorite")',
            run: function () {}, // it's a check
        }]);
        if (toggleReminder) {
            steps = steps.concat([{
                content: "Set Favorite",
                trigger: 'span.o_wetrack_js_reminder_text',
                run: 'click',
            }, {
                content: `Check Favorite for ${talkName} is now on`,
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
}, {
    content: 'Check we are on the talk list page',
    trigger: 'h1:contains("Book your talks")',
    run: function () {} // check
}];

var browseMeetSteps = [{
    content: 'Browse Meet',
    trigger: 'a:contains("Community")',
}, {
    content: 'Check we are on the community page',
    trigger: 'span:contains("Join a room")',
    run: function () {} // check
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

## File: views\event_registration_templates_reports.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="event_registration_template_report">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="registration">
                <t t-call="web.external_layout">
                    <div class="page">
                        <p>This is a sample of an external report.</p>
                    </div>
                </t>
            </t>
        </t>
    </template>
</odoo>

```

