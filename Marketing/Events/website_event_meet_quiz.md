# Odoo Module: website_event_meet_quiz

Category: Marketing/Events

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
    'name': 'Quiz and Meet on community',
    'category': 'Marketing/Events',
    'sequence': 1007,
    'version': '1.0',
    'summary': 'Quiz and Meet on community route',
    'website': 'https://www.odoo.com/page/events',
    'description': "",
    'depends': [
        'website_event_meet',
        'website_event_track_quiz',
    ],
    'data': [
        'views/event_meet_templates.xml',
    ],
    'demo': [
    ],
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\community.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden

from odoo import http
from odoo.addons.website_event.controllers.community import EventCommunityController
from odoo.http import request


class WebsiteEventTrackQuizMeetController(EventCommunityController):

    @http.route(['/event/<model("event.event"):event>/community'], type='http', auth="public", website=True, sitemap=False)
    def community(self, event, page=1, lang=None, **kwargs):
        if not event.can_access_from_current_website():
            raise Forbidden()

        # website_event_track_quiz
        values = self._get_community_leaderboard_render_values(event, kwargs.get('search'), page)

        # website_event_meet
        values.update(self._event_meeting_rooms_get_values(event, lang=lang))
        return request.render('website_event_meet.event_meet', values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import community

```

## File: views\event_meet_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="community_aside" name="Community: Aside (Leaderboard)"
    inherit_id="website_event_meet.community_aside">
    <xpath expr="//div[hasclass('o_wevent_community_aside')]" position="inside">
        <t t-if="top3_visitors" t-call="website_event_meet_quiz.community_leaderboard_small"/>
    </xpath>
</template>

<template id="community_leaderboard_small" name="Small Leaderboard">
    <div class="w-100 d-flex justify-content-between align-items-end o_page_header">
        <h2>Leaderboard</h2>
        <a t-attf-href="#{'/event/%s/community/leaderboard' % (slug(event))}">View all</a>
    </div>
    <t t-call="website_event_meet_quiz.visitor_quiz_points_card"/>
</template>

<template id='visitor_quiz_points_card' name="Small Leaderboard: top3">
    <table class="w-100">
        <tr t-foreach="top3_visitors" t-as="visitor">
            <td><div class="d-flex align-items-center mb-3">
                <b class="mr-2 text-muted" t-esc="visitor['position']"/>
                <img class="rounded-circle float-left"
                    style="width: 32px; height: 32px; object-fit: cover;"
                    t-att-src="image_data_uri(visitor['visitor'].partner_image) if visitor['visitor'].partner_image else '/web/static/src/img/user_placeholder.jpg'"
                    t-att-alt="visitor['visitor'].display_name"/>
                <div class="o_we_leaderboard_name">
                    <span t-if="visitor['visitor'] == current_visitor and not visitor['visitor'].name" class="font-weight-bold">You</span>
                    <span t-else="" class="font-weight-bold" t-esc="visitor['visitor'].display_name"/>
                </div>
            </div></td>
            <td class="text-right">
                <div class="mb-3">
                    <span class="text-nowrap"><t t-esc="visitor['points']"/> xp</span>
                </div>
            </td>
        </tr>
    </table>
</template>

</odoo>

```

