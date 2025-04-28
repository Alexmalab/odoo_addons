# Odoo Module: link_tracker

Category: Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controller
from . import models
from . import tools

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Link Tracker',
    'category': 'Marketing',
    'description': """
Shorten URLs and use them to track clicks and UTMs
""",
    'version': '1.1',
    'depends': ['utm', 'mail'],
    'data': [
        'views/link_tracker_views.xml',
        'views/utm_campaign_views.xml',
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: controller\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import NotFound

from odoo import http
from odoo.http import request


class LinkTracker(http.Controller):

    @http.route('/r/<string:code>', type='http', auth='public', website=True)
    def full_url_redirect(self, code, **post):
        if not request.env['ir.http'].is_a_bot():
            request.env['link.tracker.click'].sudo().add_click(
                code,
                ip=request.httprequest.remote_addr,
                country_code=request.geoip.country_code,
            )
        redirect_url = request.env['link.tracker'].get_url_from_code(code)
        if not redirect_url:
            raise NotFound()
        return request.redirect(redirect_url, code=301, local=False)

```

## File: controller\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\link_tracker.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import random
import string

from werkzeug import urls

from odoo import _, api, fields, models, tools
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools.mail import validate_url

from odoo.addons.mail.tools import link_preview

LINK_TRACKER_UNIQUE_FIELDS = ('url', 'campaign_id', 'medium_id', 'source_id', 'label')

_logger = logging.getLogger(__name__)

LINK_TRACKER_MIN_CODE_LENGTH = 3


class LinkTracker(models.Model):
    """ Link trackers allow users to wrap any URL into a short URL that can be
    tracked by Odoo. Clicks are counter on each link. A tracker is linked to
    UTMs allowing to analyze marketing actions.

    This model is also used in mass_mailing where each link in html body is
    automatically converted into a short link that is tracked and integrates
    UTMs. """
    _name = "link.tracker"
    _rec_name = "short_url"
    _description = "Link Tracker"
    _order = "count DESC"
    _inherit = ["utm.mixin"]

    # URL info
    url = fields.Char(string='Target URL', required=True)
    absolute_url = fields.Char("Absolute URL", compute="_compute_absolute_url")
    short_url = fields.Char(string='Tracked URL', compute='_compute_short_url')
    redirected_url = fields.Char(string='Redirected URL', compute='_compute_redirected_url')
    short_url_host = fields.Char(string='Host of the short URL', compute='_compute_short_url_host')
    title = fields.Char(string='Page Title', store=True)
    label = fields.Char(string='Button label')
    # Tracking
    link_code_ids = fields.One2many('link.tracker.code', 'link_id', string='Codes')
    code = fields.Char(string='Short URL code', compute='_compute_code', inverse="_inverse_code", readonly=False)
    link_click_ids = fields.One2many('link.tracker.click', 'link_id', string='Clicks')
    count = fields.Integer(string='Number of Clicks', compute='_compute_count', store=True)
    # UTMs - enforcing the fact that we want to 'set null' when relation is unlinked
    campaign_id = fields.Many2one(ondelete='set null')
    medium_id = fields.Many2one(ondelete='set null')
    source_id = fields.Many2one(ondelete='set null')

    @api.depends("url")
    def _compute_absolute_url(self):
        for tracker in self:
            url = urls.url_parse(tracker.url)
            if url.scheme:
                tracker.absolute_url = tracker.url
            else:
                tracker.absolute_url = urls.url_join(tracker.get_base_url(), url)

    @api.depends('link_click_ids.link_id')
    def _compute_count(self):
        clicks_data = self.env['link.tracker.click']._read_group(
            [('link_id', 'in', self.ids)],
            ['link_id'],
            ['__count'],
        )
        mapped_data = {link.id: count for link, count in clicks_data}
        for tracker in self:
            tracker.count = mapped_data.get(tracker.id, 0)

    @api.depends('code')
    def _compute_short_url(self):
        for tracker in self:
            tracker.short_url = urls.url_join(tracker.short_url_host or '', tracker.code or '')

    def _compute_short_url_host(self):
        for tracker in self:
            tracker.short_url_host = tracker.get_base_url() + '/r/'

    def _compute_code(self):
        for tracker in self:
            record = self.env['link.tracker.code'].search([('link_id', '=', tracker.id)], limit=1, order='id DESC')
            tracker.code = record.code

    def _inverse_code(self):
        self.ensure_one()
        if not self.code:
            return
        record = self.env['link.tracker.code'].search([('link_id', '=', self.id)], limit=1, order='id DESC')
        if record:
            record.code = self.code

    @api.depends('url')
    def _compute_redirected_url(self):
        """Compute the URL to which we will redirect the user.

        By default, add UTM values as GET parameters. But if the system parameter
        `link_tracker.no_external_tracking` is set, we add the UTM values in the URL
        *only* for URLs that redirect to the local website (base URL).
        """
        no_external_tracking = self.env['ir.config_parameter'].sudo().get_param('link_tracker.no_external_tracking')

        for tracker in self:
            base_domain = urls.url_parse(tracker.get_base_url()).netloc
            parsed = urls.url_parse(tracker.url)
            if no_external_tracking and parsed.netloc and parsed.netloc != base_domain:
                tracker.redirected_url = parsed.to_url()
                continue

            query = parsed.decode_query()
            for key, field_name, cook in self.env['utm.mixin'].tracking_fields():
                field = self._fields[field_name]
                attr = tracker[field_name]
                if field.type == 'many2one':
                    attr = attr.name
                if attr:
                    query[key] = attr
            tracker.redirected_url = parsed.replace(query=urls.url_encode(query)).to_url()

    @api.model
    @api.depends('url')
    def _get_title_from_url(self, url):
        preview = link_preview.get_link_preview_from_url(url)
        if preview and preview.get('og_title'):
            return preview['og_title']
        return url

    @api.constrains(*LINK_TRACKER_UNIQUE_FIELDS)
    def _check_unicity(self):
        """Check that the link trackers are unique."""
        def _format_value(tracker, field_name):
            if field_name == 'label' and not tracker[field_name]:
                return False
            return tracker[field_name]

        # build a query to fetch all needed link trackers at once
        search_query = expression.OR([
            expression.AND([
                [('url', '=', tracker.url)],
                [('campaign_id', '=', tracker.campaign_id.id)],
                [('medium_id', '=', tracker.medium_id.id)],
                [('source_id', '=', tracker.source_id.id)],
                [('label', '=', tracker.label) if tracker.label else ('label', 'in', (False, ''))],
            ])
            for tracker in self
        ])

        # Can not be implemented with a SQL constraint because we care about null values.
        potential_duplicates = self.search(search_query)
        duplicates = self.browse()
        seen = set()
        for tracker in potential_duplicates:
            unique_fields = tuple(_format_value(tracker, field_name) for field_name in LINK_TRACKER_UNIQUE_FIELDS)
            if unique_fields in seen or seen.add(unique_fields):
                duplicates += tracker
        if duplicates:
            error_lines = '\n- '.join(
                str((tracker.url, tracker.campaign_id.name, tracker.medium_id.name, tracker.source_id.name, tracker.label or '""'))
                for tracker in duplicates
            )
            raise UserError(
                _('Combinations of Link Tracker values (URL, campaign, medium, source, and label) must be unique.\n'
                  'The following combinations are already used: \n- %(error_lines)s', error_lines=error_lines))

    @api.model_create_multi
    def create(self, vals_list):
        vals_list = [vals.copy() for vals in vals_list]
        for vals in vals_list:
            if 'url' not in vals:
                raise ValueError(_('Creating a Link Tracker without URL is not possible'))

            if vals['url'].startswith(('?', '#')):
                raise UserError(_("“%s” is not a valid link, links cannot redirect to the current page.", vals['url']))
            vals['url'] = validate_url(vals['url'])

            if not vals.get('title'):
                vals['title'] = self._get_title_from_url(vals['url'])

            # Prevent the UTMs to be set by the values of UTM cookies
            for (__, fname, __) in self.env['utm.mixin'].tracking_fields():
                if fname not in vals:
                    vals[fname] = False

        links = super(LinkTracker, self).create(vals_list)

        link_tracker_codes = self.env['link.tracker.code']._get_random_code_strings(len(vals_list))

        self.env['link.tracker.code'].sudo().create([
            {
                'code': code,
                'link_id': link.id,
            } for link, code in zip(links, link_tracker_codes)
        ])

        return links

    @api.model
    def search_or_create(self, vals_list):
        """Get existing or newly created records matching vals_list items in preserved order supporting duplicates."""
        if not isinstance(vals_list, list):
            _logger.warning("Deprecated usage of LinkTracker.search_or_create which now expects a list of dictionaries as input.")
            vals_list = [vals_list]

        def _format_key(obj):
            """Generate unique 'key' of trackers, allowing to find duplicates."""
            return tuple(
                (field_name, obj[field_name].id if isinstance(obj[field_name], models.BaseModel) else obj[field_name])
                for field_name in LINK_TRACKER_UNIQUE_FIELDS
            )

        def _format_key_domain(field_values):
            """Handle "label" being False / '' and be defensive."""
            return expression.AND([
                [(field_name, '=', value) if value or field_name != 'label' else ('label', 'in', (False, ''))]
                for field_name, value in field_values
            ])

        errors = set()
        for vals in vals_list:
            if 'url' not in vals:
                raise ValueError(_('Creating a Link Tracker without URL is not possible'))
            if vals['url'].startswith(('?', '#')):
                errors.add(_("“%s” is not a valid link, links cannot redirect to the current page.", vals['url']))
            vals['url'] = validate_url(vals['url'])
            # fill vals to use direct accessor in _format_key
            self._add_missing_default_values(vals)
            vals.update({key: False for key in LINK_TRACKER_UNIQUE_FIELDS if not vals.get(key)})
        if errors:
            raise UserError("\n".join(errors))

        # Find unique keys of trackers, then fetch existing trackers
        unique_keys = {_format_key(vals) for vals in vals_list}
        found_trackers = self.search(expression.OR([_format_key_domain(key) for key in unique_keys]))
        key_to_trackers_map = {_format_key(tracker): tracker for tracker in found_trackers}

        if len(unique_keys) != len(found_trackers):
            # Create trackers for values with unique keys not found
            seen_keys = set(key_to_trackers_map.keys())
            new_trackers = self.create([
                vals for vals in vals_list
                if (key := _format_key(vals)) not in seen_keys and not seen_keys.add(key)
            ])
            key_to_trackers_map.update((_format_key(tracker), tracker) for tracker in new_trackers)

        # Build final recordset following input order
        return self.browse([key_to_trackers_map[_format_key(vals)].id for vals in vals_list])

    @api.model
    def convert_links(self, html, vals, blacklist=None):
        raise NotImplementedError('Moved on mail.render.mixin')

    def _convert_links_text(self, body, vals, blacklist=None):
        raise NotImplementedError('Moved on mail.render.mixin')

    def action_view_statistics(self):
        action = self.env['ir.actions.act_window']._for_xml_id('link_tracker.link_tracker_click_action_statistics')
        action['domain'] = [('link_id', '=', self.id)]
        action['context'] = dict(self._context, create=False)
        return action

    def action_visit_page(self):
        return {
            'name': _("Visit Webpage"),
            'type': 'ir.actions.act_url',
            'url': self.url,
            'target': 'new',
        }

    @api.model
    def recent_links(self, filter, limit):
        if filter == 'newest':
            return self.search_read([], order='create_date DESC, id DESC', limit=limit)
        elif filter == 'most-clicked':
            return self.search_read([('count', '!=', 0)], order='count DESC, id DESC', limit=limit)
        elif filter == 'recently-used':
            return self.search_read([('count', '!=', 0)], order='write_date DESC, id DESC', limit=limit)
        else:
            return {'Error': "This filter doesn't exist."}

    @api.model
    def get_url_from_code(self, code):
        code_rec = self.env['link.tracker.code'].sudo().search([('code', '=', code)])

        if not code_rec:
            return None

        return code_rec.link_id.redirected_url


class LinkTrackerCode(models.Model):
    _name = "link.tracker.code"
    _description = "Link Tracker Code"
    _rec_name = 'code'

    code = fields.Char(string='Short URL Code', required=True, store=True)
    link_id = fields.Many2one('link.tracker', 'Link', required=True, ondelete='cascade')

    _sql_constraints = [
        ('code', 'unique( code )', 'Code must be unique.')
    ]

    @api.model
    def _get_random_code_strings(self, n=1):
        size = LINK_TRACKER_MIN_CODE_LENGTH
        while True:
            code_propositions = [
                ''.join(random.choices(string.ascii_letters + string.digits, k=size))
                for __ in range(n)
            ]

            if len(set(code_propositions)) != n or self.search_count([('code', 'in', code_propositions)], limit=1):
                size += 1
            else:
                return code_propositions


class LinkTrackerClick(models.Model):
    _name = "link.tracker.click"
    _rec_name = "link_id"
    _description = "Link Tracker Click"

    campaign_id = fields.Many2one(
        'utm.campaign', 'UTM Campaign', index='btree_not_null',
        related="link_id.campaign_id", store=True, ondelete="set null")
    link_id = fields.Many2one(
        'link.tracker', 'Link',
        index=True, required=True, ondelete='cascade')
    ip = fields.Char(string='Internet Protocol')
    country_id = fields.Many2one('res.country', 'Country')

    def _prepare_click_values_from_route(self, **route_values):
        click_values = dict((fname, route_values[fname]) for fname in self._fields if fname in route_values)
        if not click_values.get('country_id') and route_values.get('country_code'):
            click_values['country_id'] = self.env['res.country'].search([('code', '=', route_values['country_code'])], limit=1).id
        return click_values

    @api.model
    def add_click(self, code, **route_values):
        """ Main API to add a click on a link. """
        tracker_code = self.env['link.tracker.code'].search([('code', '=', code)])
        if not tracker_code:
            return None

        route_values['link_id'] = tracker_code.link_id.id
        click_values = self._prepare_click_values_from_route(**route_values)

        return self.create(click_values)

```

## File: models\mail_render_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from html import unescape

import lxml
import markupsafe
from werkzeug import urls

from odoo import api, models, tools
from odoo.addons.link_tracker.tools.html import find_links_with_urls_and_labels
from odoo.tools.mail import is_html_empty, URL_SKIP_PROTOCOL_REGEX, TEXT_URL_REGEX


class MailRenderMixin(models.AbstractModel):
    _inherit = "mail.render.mixin"

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    @api.model
    def _shorten_links(self, html, link_tracker_vals, blacklist=None, base_url=None):
        """ Shorten links in an html content. It uses the '/r' short URL routing
        introduced in this module. Using the standard Odoo regex local links are
        found and replaced by global URLs (not including mailto, tel, sms).

        TDE FIXME: could be great to have a record to enable website-based URLs

        :param link_tracker_vals: values given to the created link.tracker, containing
          for example: campaign_id, medium_id, source_id, and any other relevant fields
          like mass_mailing_id in mass_mailing;
        :param list blacklist: list of (local) URLs to not shorten (e.g.
          '/unsubscribe_from_list')
        :param str base_url: either given, either based on config parameter

        :return: updated html
        """
        if not html or is_html_empty(html):
            return html
        base_url = base_url or self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        short_schema = base_url + '/r/'

        root_node = lxml.html.fromstring(html)
        link_nodes, urls_and_labels = find_links_with_urls_and_labels(
            root_node, base_url, skip_regex=rf'^{URL_SKIP_PROTOCOL_REGEX}', skip_prefix=short_schema,
            skip_list=blacklist)
        if not link_nodes:
            return html

        links_trackers = self.env['link.tracker'].search_or_create([
            dict(link_tracker_vals, **url_and_label) for url_and_label in urls_and_labels
        ])
        for node, link_tracker in zip(link_nodes, links_trackers):
            node.set("href", link_tracker.short_url)

        new_html = lxml.html.tostring(root_node, encoding="unicode", method="xml")
        if isinstance(html, markupsafe.Markup):
            new_html = markupsafe.Markup(new_html)

        return new_html

    @api.model
    def _shorten_links_text(self, content, link_tracker_vals, blacklist=None, base_url=None):
        """ Shorten links in a string content. Works like ``_shorten_links`` but
        targeting string content, not html.

        :return: updated content
        """
        if not content:
            return content
        base_url = base_url or self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        shortened_schema = base_url + '/r/'
        unsubscribe_schema = base_url + '/sms/'
        for original_url in set(re.findall(TEXT_URL_REGEX, content)):
            # don't shorten already-shortened links or links towards unsubscribe page
            if original_url.startswith(shortened_schema) or original_url.startswith(unsubscribe_schema):
                continue
            # support blacklist items in path, like /u/
            parsed = urls.url_parse(original_url, scheme='http')
            if blacklist and any(item in parsed.path for item in blacklist):
                continue

            create_vals = dict(link_tracker_vals, url=unescape(original_url))
            link = self.env['link.tracker'].search_or_create([create_vals])
            if link.short_url:
                # Ensures we only replace the same link and not a subpart of a longer one, multiple times if applicable
                content = re.sub(re.escape(original_url) + r'(?![\w@:%.+&~#=/-])', link.short_url, content)

        return content

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = ['utm.campaign']
    _description = 'UTM Campaign'

    click_count = fields.Integer(string="Number of clicks generated by the campaign", compute="_compute_clicks_count")

    def _compute_clicks_count(self):
        click_data = self.env['link.tracker.click']._read_group(
            [('campaign_id', 'in', self.ids)],
            ['campaign_id'], ['__count'])

        mapped_data = {campaign.id: count for campaign, count in click_data}

        for campaign in self:
            campaign.click_count = mapped_data.get(campaign.id, 0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import link_tracker
from . import mail_render_mixin
from . import utm

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_link_tracker_user,access.link.tracker.user,model_link_tracker,base.group_user,1,0,0,0
access_link_tracker_public,access.link.tracker.public,model_link_tracker,base.group_public,0,0,0,0
access_link_tracker_system,access.link.tracker.system,model_link_tracker,base.group_system,1,1,1,1
access_link_tracker_code_user,access.link.tracker.code.user,model_link_tracker_code,base.group_user,1,0,0,0
access_link_tracker_code_public,access.link.tracker.code.public,model_link_tracker_code,base.group_public,0,0,0,0
access_link_tracker_code_system,access.link.tracker.code.system,model_link_tracker_code,base.group_system,1,1,1,1
access_link_tracker_click_user,access.link.tracker.click.user,model_link_tracker_click,base.group_user,1,0,0,0
access_link_tracker_click_public,access.link.tracker.click.public,model_link_tracker_click,base.group_public,0,0,0,0
access_link_tracker_click_system,access.link.tracker.click.system,model_link_tracker_click,base.group_system,1,1,1,1

```

## File: tools\html.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from typing import Iterable

import lxml

MAX_LABEL_LENGTH = 40  # arbitrary


def find_links_with_urls_and_labels(root_node, base_url, skip_regex=None, skip_prefix=None, skip_list=None):
    """Return lxml link nodes and respective matching urls (made absolute) and labels found in `root_node`.

    :param lxml.etree._Element root_node: The root node to process
    :param str base_url: base url to prefix relative hrefs
    :param str skip_regex: URL pattern to skip
    :param str skip_prefix: str prefix to skip
    :param Iterable[str] skip_list: URLS to skip

    :rtype: (list[lxml.etree._Element], list[dict])
    """
    link_nodes, urls_and_labels = [], []

    for link_node in root_node.iter(tag="a"):
        original_url = link_node.get("href")
        if not original_url:
            continue
        absolute_url = base_url + original_url if original_url.startswith(('/', '?', '#')) else original_url
        if (
            (skip_regex and re.search(skip_regex, absolute_url))
            or (skip_prefix and absolute_url.startswith(skip_prefix))
            or (skip_list and any(s in absolute_url for s in skip_list))
        ):
            continue

        if link_node.text and (stripped_text := link_node.text.strip()):
            label = stripped_text[:MAX_LABEL_LENGTH]
        else:
            children = link_node.getchildren()
            label = _get_label_from_elements(children)[:MAX_LABEL_LENGTH]

        link_nodes.append(link_node)
        urls_and_labels.append({'url': absolute_url, 'label': label})

    return link_nodes, urls_and_labels


def _get_label_from_elements(elements: Iterable[lxml.etree._Element], image_prefix: str = "[media] ") -> str:
    """Return the first label that can be extracted from a collection of elements"""
    for element in elements:
        if element.tag == "img":
            if img_alt := element.get("alt"):
                return f"{image_prefix}{img_alt}"
            if img_src := element.get("src"):
                img_src_tail = img_src.split("/")[-1]
                return f"{image_prefix}{img_src_tail}"
            return ""
        if isinstance(element, lxml.html.HtmlComment):  # A known "hack"
            continue
        if element.tag == "p" and element.get("class") == "o_outlook_hack":
            children = element.getchildren()
            if label := _get_label_from_elements(children):
                return label
    return ""

```

## File: tools\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import html

```

## File: views\link_tracker_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- LINT.TRACKER -->
        <record id="link_tracker_view_search" model="ir.ui.view">
            <field name="name">link.tracker.view.search</field>
            <field name="model">link.tracker</field>
            <field name="arch" type="xml">
                <search string="Links">
                    <field name="url" string="Title and URL" filter_domain="['|', ('title', 'ilike', self), ('url', 'ilike', self)]"/>
                    <field name="title"/>
                    <field name="label"/>
                    <field name="campaign_id"/>
                    <field name="medium_id"/>
                    <field name="source_id"/>
                    <group expand="0" string="Group By">
                        <filter string="Campaign" name="groupby_campaign_id" context="{'group_by': 'campaign_id'}"/>
                        <filter string="Medium" name="groupby_medium_id" context="{'group_by': 'medium_id'}"/>
                        <filter string="Source" name="groupby_source_id" context="{'group_by': 'source_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="link_tracker_view_form" model="ir.ui.view">
            <field name="name">link.tracker.view.form</field>
            <field name="model">link.tracker</field>
            <field name="arch" type="xml">
                <form string="Website Link" duplicate="0">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button type="object" icon="fa-sign-out" name="action_visit_page"
                                string="Visit Page" class="oe_stat_button">
                                <div class="o_field_widget o_stat_info">
                                    <span class="o_stat_text">Visit Page</span>
                                </div>
                            </button>

                            <button type="object" class="oe_stat_button" name="action_view_statistics" icon="fa-bar-chart-o">
                                <field name="count" string="Clicks" widget="statinfo"/>
                            </button>
                        </div>
                        <group>
                            <group name="url" string="URL">
                                <field name="title"/>
                                <field name="label"/>
                                <field name="url"/>
                                <label for="code" string="Tracked URL"/>
                                <div>
                                    <field name="short_url_host" nolabel="1" readonly="1" class="me-1 oe_inline"/>
                                    <field name="code" nolabel="1" readonly="not code" class="oe_inline o_input"/>
                                </div>
                            </group>
                            <group name="utm" string="UTM">
                                <field name="campaign_id" options="{'create_name_field': 'title'}"/>
                                <field name="medium_id"/>
                                <field name="source_id"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="link_tracker_view_tree" model="ir.ui.view">
            <field name="name">link.tracker.view.list</field>
            <field name="model">link.tracker</field>
            <field name="arch" type="xml">
                <list string="Links" sample="1">
                    <field name="create_date"/>
                    <field name="title"/>
                    <field name="label"/>
                    <field name="url"/>
                    <field name="short_url" optional="hide"/>
                    <field name="count"/>
                    <button name="action_visit_page" type="object" string="Visit Page" icon="fa-external-link"/>
                </list>
            </field>
        </record>

        <record id="link_tracker_view_graph" model="ir.ui.view">
            <field name="name">link.tracker.view.graph</field>
            <field name="model">link.tracker</field>
            <field name="arch" type="xml">
                <graph string="Links" sample="1">
                    <field name="url"/>
                    <field name="count" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="link_tracker_action" model="ir.actions.act_window">
            <field name="name">Link Tracker</field>
            <field name="res_model">link.tracker</field>
            <field name="view_mode">list,form,graph</field>
            <field name="view_id" ref="link_tracker_view_tree"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a link tracker
                </p><p>
                    Trackers are used to collect count stat about click on links and generate short URLs.
                </p>
            </field>
        </record>

        <!-- LINK.TRACKER.CLICK -->
        <record id="link_tracker_click_view_search" model="ir.ui.view">
            <field name="name">link.tracker.click.view.search</field>
            <field name="model">link.tracker.click</field>
            <field name="arch" type="xml">
                <search string="Clicks">
                    <field name="link_id"/>
                    <field name="country_id"/>
                    <group expand="0" string="Group By">
                        <filter string="Link" name="groupby_link_id" domain="[]" context="{'group_by': 'link_id'}"/>
                        <filter string="Country" name="groupby_country_id" context="{'group_by': 'country_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="link_tracker_click_view_form" model="ir.ui.view">
            <field name="name">link.tracker.click.view.form</field>
            <field name="model">link.tracker.click</field>
            <field name="arch" type="xml">
                <form string="Link Click">
                    <sheet>
                        <group>
                            <field name="link_id"/>
                            <field name="ip"/>
                            <field name="country_id" options="{'no_open': True, 'no_create': True}"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="link_tracker_click_view_tree" model="ir.ui.view">
            <field name="name">link.tracker.click.view.list</field>
            <field name="model">link.tracker.click</field>
            <field name="arch" type="xml">
                <list string="Links Clicks">
                    <field name="link_id"/>
                    <field name="ip"/>
                    <field name="country_id"/>
                </list>
            </field>
        </record>

        <record id="link_tracker_click_view_graph" model="ir.ui.view">
            <field name="name">link.tracker.click.view.graph</field>
            <field name="model">link.tracker.click</field>
            <field name="arch" type="xml">
                <graph string="Link Clicks" type="pie" sample="1">
                    <field name="link_id"/>
                    <field name="ip"/>
                    <field name="country_id"/>
                </graph>
            </field>
        </record>

        <record id="link_tracker_click_action_statistics" model="ir.actions.act_window">
            <field name="name">Click Statistics</field>
            <field name="res_model">link.tracker.click</field>
            <field name="view_mode">graph,list,form</field>
            <field name="domain">[]</field>
            <field name="context">{'search_default_groupby_country_id': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
        </record>

        <record id="link_tracker_action_campaign" model="ir.actions.act_window">
            <field name="name">Statistics of Clicks</field>
            <field name="res_model">link.tracker</field>
            <field name="view_mode">list,form,graph</field>
            <field name="view_id" ref="link_tracker.link_tracker_view_tree"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    Create a link tracker
                </p><p>
                    Trackers are used to collect count stat about click on links and generate short URLs.
                </p>
            </field>
            <field name="context">{'search_default_campaign_id': active_id}</field>
        </record>

        <!-- MENUS -->
        <menuitem id="link_tracker_menu_main"
            name="Link Tracker"
            parent="utm.menu_link_tracker_root"
            action="link_tracker_action"
            groups="base.group_no_one"/>
    </data>
</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="utm_campaign_view_form" model="ir.ui.view">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(link_tracker_action_campaign)d"
                    type="action" class="oe_stat_button order-12" icon="fa-mouse-pointer">
                    <field name="click_count" widget="statinfo" string="Clicks"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="utm_campaign_view_kanban" model="ir.ui.view">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//footer/div" position="inside">
                <a t-if="record.click_count" href="#" title="Clicks" role="button"
                    type="action" name="%(link_tracker_action_campaign)d"
                    class="btn-outline-primary rounded-pill me-1 order-4">
                    <span class="badge">
                        <i class="fa fa-fw fa-mouse-pointer" aria-label="Clicks" role="img"/>
                        <field name="click_count"/>
                    </span>
                </a>
            </xpath>
        </field>
    </record>
</odoo>

```

