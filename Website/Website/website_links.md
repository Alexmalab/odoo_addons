# Odoo Module: website_links

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import controller
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Link Tracker',
    'category': 'Website/Website',
    'summary': 'Generate trackable & short URLs',
    'description': """
Generate short links with analytics trackers (UTM) to share your pages through marketing campaigns.
Those trackers can be used in Google Analytics to track clicks and visitors, or in Odoo reports to analyze the efficiency of those campaigns in terms of lead generation, related revenues (sales orders), recruitment, etc.
    """,
    'version': '1.0',
    'depends': ['website', 'link_tracker'],
    'data': [
        'views/link_tracker_views.xml',
        'views/website_links_template.xml',
        'views/website_links_graphs.xml',
        'security/ir.model.access.csv',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_links/static/src/js/website_links.js',
            'website_links/static/src/js/website_links_code_editor.js',
            'website_links/static/src/js/website_links_charts.js',
            'website_links/static/src/css/website_links.css',
            'website_links/static/src/xml/*.xml',
        ],
        'web.assets_tests': [
            'website_links/static/tests/**/*',
        ],
        'website.assets_editor': [
            'website_links/static/src/services/website_custom_menus.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controller\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class WebsiteUrl(http.Controller):
    @http.route('/website_links/new', type='json', auth='user', methods=['POST'])
    def create_shorten_url(self, **post):
        if 'url' not in post or post['url'] == '':
            return {'error': 'empty_url'}
        return request.env['link.tracker'].search_or_create(post).read()

    @http.route('/r', type='http', auth='user', website=True)
    def shorten_url(self, **post):
        return request.render("website_links.page_shorten_url", post)

    @http.route('/website_links/add_code', type='json', auth='user')
    def add_code(self, **post):
        link_id = request.env['link.tracker.code'].search([('code', '=', post['init_code'])], limit=1).link_id.id
        new_code = request.env['link.tracker.code'].search_count([('code', '=', post['new_code']), ('link_id', '=', link_id)])
        if new_code > 0:
            return new_code.read()
        else:
            return request.env['link.tracker.code'].create({'code': post['new_code'], 'link_id': link_id})[0].read()

    @http.route('/website_links/recent_links', type='json', auth='user')
    def recent_links(self, **post):
        return request.env['link.tracker'].recent_links(post['filter'], post['limit'])

    @http.route('/r/<string:code>+', type='http', auth="user", website=True)
    def statistics_shorten_url(self, code, **post):
        code = request.env['link.tracker.code'].search([('code', '=', code)], limit=1)

        if code:
            return request.render("website_links.graphs", code.link_id.read()[0])
        else:
            return request.redirect('/', code=301)

```

## File: controller\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: models\link_tracker.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _

from werkzeug import urls


class LinkTracker(models.Model):
    _inherit = ['link.tracker']

    def action_visit_page_statistics(self):
        return {
            'name': _("Visit Webpage Statistics"),
            'type': 'ir.actions.act_url',
            'url': '%s+' % (self.short_url),
            'target': 'new',
        }

    def _compute_short_url_host(self):
        current_website = self.env['website'].get_current_website()
        base_url = current_website.get_base_url() if current_website == self.env.company.website_id else self.env.company.get_base_url()
        for tracker in self:
            tracker.short_url_host = urls.url_join(base_url, '/r/')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import link_tracker

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_link_tracker_manager,link.tracker,link_tracker.model_link_tracker,website.group_website_designer,1,1,1,1
access_link_tracker_code_manager,link.tracker.code,link_tracker.model_link_tracker_code,website.group_website_designer,1,1,1,1
access_link_tracker_click_manager,link.tracker.click,link_tracker.model_link_tracker_click,website.group_website_designer,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M.983 23.814a3.356 3.356 0 0 1 0-4.746L11.068 8.983a3.356 3.356 0 0 1 4.746 0l17.203 17.203a3.356 3.356 0 0 1 0 4.746L22.932 41.017a3.356 3.356 0 0 1-4.746 0L.983 23.814Z" fill="#FC868B"/><path d="M16.983 19.068a3.356 3.356 0 0 0 0 4.746l4.53 4.53 7.063-6.598 4.441 4.44a3.356 3.356 0 0 1 0 4.747L28.56 35.39l5.627 5.627a3.356 3.356 0 0 0 4.746 0l10.085-10.085a3.356 3.356 0 0 0 0-4.746L31.814 8.983a3.356 3.356 0 0 0-4.746 0L16.983 19.068Z" fill="#985184"/></svg>

```

## File: static\src\js\website_links.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { browser } from "@web/core/browser/browser";
import { KeepLast } from "@web/core/utils/concurrency";

var SelectBox = publicWidget.Widget.extend({
    events: {
        'change': '_onChange',
    },

    /**
     * @constructor
     * @param {Object} parent
     * @param {Object} obj
     * @param {String} placeholder
     */
    init: function (parent, obj, placeholder) {
        this._super.apply(this, arguments);
        this.obj = obj;
        this.placeholder = placeholder;

        this.orm = this.bindService("orm");
        this.keepLast = new KeepLast();

        // TODO remove in master, contained the whole list of preloaded entries.
        // Now we lazy load results based on the user search. We still save them
        // in this array each time so that the "Create" feature works and so
        // that potential custo may still make sense, or at least do not crash.
        this.objects = [];
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$el.select2({
            placeholder: self.placeholder,
            allowClear: true,
            formatNoMatches: false,
            createSearchChoice: function (term) {
                if (self._objectExists(term)) {
                    return null;
                }
                return { id: term, text: `Create '${term}'` };
            },
            createSearchChoicePosition: 'bottom',
            multiple: false,
            ajax: {
                dataType: 'json',
                data: term => term,
                transport: (params, success, failure) => {
                    // Do not search immediately: wait for the user to stop
                    // typing (basically, this is a debounce).
                    clearTimeout(this._loadDataTimeout);
                    this._loadDataTimeout = setTimeout(() => {
                        // We want to search with a limit and not care about any
                        // pagination implementation. To make this work, we
                        // display the exact match first though, which requires
                        // an extra RPC (could be refactored into a new
                        // controller in master but... see TODO).
                        // TODO at some point this whole app will be moved as a
                        // backend screen, with real m2o fields etc... in which
                        // case the "exact match" feature should be handled by
                        // the ORM somehow ?
                        const limit = 100;
                        const searchReadParams = [
                            ['id', 'name'],
                            {
                                limit: limit,
                                order: 'name, id desc', // Allows to have exact match first
                            },
                        ];
                        const proms = [];
                        proms.push(this.orm.searchRead(
                            this.obj,
                            // Exact match + results that start with the search
                            [['name', '=ilike', `${params.data}%`]],
                            ...searchReadParams
                        ));
                        proms.push(this.orm.searchRead(
                            this.obj,
                            // Results that contain the search but do not start
                            // with it
                            [['name', '=ilike', `%_${params.data}%`]],
                            ...searchReadParams
                        ));
                        // Keep last is there in case a RPC takes longer than
                        // the debounce delay + next rpc delay for some reason.
                        this.keepLast.add(Promise.all(proms)).then(([startingMatches, endingMatches]) => {
                            // We loaded max a 2 * limit amount of records but
                            // ensure that we do not display "ending matches" if
                            // we may not have loaded all "starting matches".
                            if (startingMatches.length < limit) {
                                const startingMatchesId = startingMatches.map((value) => value.id);
                                const extraEndingMatches = endingMatches.filter(
                                    (value) => !startingMatchesId.includes(value.id)
                                );
                                return startingMatches.concat(extraEndingMatches);
                            }
                            // In that case, we made one RPC too much but this
                            // was chosen over not making them go in parallel.
                            // We don't want to display "ending matches" if not
                            // all "starting matches" have been loaded.
                            return startingMatches;
                        })
                        .then(params.success)
                        .catch(params.error);
                    }, 400);
                },
                results: data => {
                    this.objects = data.map(x => ({
                        id: x.id,
                        text: x.name,
                    }));
                    return {
                        results: this.objects,
                    };
                },
            },
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {String} query
     */
    _objectExists: function (query) {
        return this.objects.find(val => val.text.toLowerCase() === query.toLowerCase()) !== undefined;
    },
    /**
     * @private
     * @param {String} name
     */
    _createObject: function (name) {
        var args = {
            name: name
        };
        if (this.obj === "utm.campaign") {
            args.is_auto_campaign = true;
        }
        return this.orm.create(this.obj, [args]).then(record => {
            this.$el.attr('value', record);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Object} ev
     */
    _onChange: function (ev) {
        if (!ev.added || typeof ev.added.id !== "string") {
            return;
        }
        this._createObject(ev.added.id);
    },
});

var RecentLinkBox = publicWidget.Widget.extend({
    template: 'website_links.RecentLink',
    events: {
        'click .btn_shorten_url_clipboard': '_toggleCopyButton',
        'click .o_website_links_edit_code': '_editCode',
        'click .o_website_links_ok_edit': '_onLinksOkClick',
        'click .o_website_links_cancel_edit': '_onLinksCancelClick',
        'submit #o_website_links_edit_code_form': '_onSubmitCode',
    },

    /**
     * @constructor
     * @param {Object} parent
     * @param {Object} obj
     */
    init: function (parent, obj) {
        this._super.apply(this, arguments);
        this.link_obj = obj;
        this.animating_copy = false;
        this.rpc = this.bindService("rpc");
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _toggleCopyButton: async function () {
        await browser.navigator.clipboard.writeText(this.link_obj.short_url);

        if (this.animating_copy) {
            return;
        }

        var self = this;
        this.animating_copy = true;
        var top = this.$('.o_website_links_short_url').position().top;
        this.$('.o_website_links_short_url').clone()
            .css('position', 'absolute')
            .css('left', 15)
            .css('top', top - 2)
            .css('z-index', 2)
            .removeClass('o_website_links_short_url')
            .addClass('animated-link')
            .insertAfter(this.$('.o_website_links_short_url'))
            .animate({
                opacity: 0,
                top: '-=20',
            }, 500, function () {
                self.$('.animated-link').remove();
                self.animating_copy = false;
            });
    },
    /**
     * @private
     * @param {String} message
     */
    _notification: function (message) {
        this.$('.notification').append('<strong>' + message + '</strong>');
    },
    /**
     * @private
     */
    _editCode: function () {
        var initCode = this.$('#o_website_links_code').html();
        this.$('#o_website_links_code').html('<form style="display:inline;" id="o_website_links_edit_code_form"><input type="hidden" id="init_code" value="' + initCode + '"/><input type="text" id="new_code" value="' + initCode + '"/></form>');
        this.$('.o_website_links_edit_code').hide();
        this.$('.copy-to-clipboard').hide();
        this.$('.o_website_links_edit_tools').show();
    },
    /**
     * @private
     */
    _cancelEdit: function () {
        this.$('.o_website_links_edit_code').show();
        this.$('.copy-to-clipboard').show();
        this.$('.o_website_links_edit_tools').hide();
        this.$('.o_website_links_code_error').hide();

        var oldCode = this.$('#o_website_links_edit_code_form #init_code').val();
        this.$('#o_website_links_code').html(oldCode);

        this.$('#code-error').remove();
        this.$('#o_website_links_code form').remove();
    },
    /**
     * @private
     */
    _submitCode: function () {
        var self = this;

        var initCode = this.$('#o_website_links_edit_code_form #init_code').val();
        var newCode = this.$('#o_website_links_edit_code_form #new_code').val();

        if (newCode === '') {
            self.$('.o_website_links_code_error').html(_t("The code cannot be left empty"));
            self.$('.o_website_links_code_error').show();
            return;
        }

        function showNewCode(newCode) {
            self.$('.o_website_links_code_error').html('');
            self.$('.o_website_links_code_error').hide();

            self.$('#o_website_links_code form').remove();

            // Show new code
            var host = self.$('#o_website_links_host').html();
            self.$('#o_website_links_code').html(newCode);

            // Update button copy to clipboard
            self.$('.btn_shorten_url_clipboard').attr('data-clipboard-text', host + newCode);

            // Show action again
            self.$('.o_website_links_edit_code').show();
            self.$('.copy-to-clipboard').show();
            self.$('.o_website_links_edit_tools').hide();
        }

        if (initCode === newCode) {
            showNewCode(newCode);
        } else {
            this.rpc('/website_links/add_code', {
                init_code: initCode,
                new_code: newCode,
            }).then(function (result) {
                showNewCode(result[0].code);
            }, function () {
                self.$('.o_website_links_code_error').show();
                self.$('.o_website_links_code_error').html(_t("This code is already taken"));
            });
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onLinksOkClick: function (ev) {
        ev.preventDefault();
        this._submitCode();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onLinksCancelClick: function (ev) {
        ev.preventDefault();
        this._cancelEdit();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onSubmitCode: function (ev) {
        ev.preventDefault();
        this._submitCode();
    },
});

var RecentLinks = publicWidget.Widget.extend({
    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    getRecentLinks: function (filter) {
        var self = this;
        return this.rpc('/website_links/recent_links', {
            filter: filter,
            limit: 20,
        }).then(function (result) {
            result.reverse().forEach((link) => {
                self._addLink(link);
            });
            self._updateNotification();
        }, function () {
            var message = _t("Unable to get recent links");
            self.$el.append('<div class="alert alert-danger">' + message + '</div>');
        });
    },
    /**
     * @private
     */
    _addLink: function (link) {
        var nbLinks = this.getChildren().length;
        var recentLinkBox = new RecentLinkBox(this, link);
        recentLinkBox.prependTo(this.$el);
        $('.link-tooltip').tooltip();

        if (nbLinks === 0) {
            this._updateNotification();
        }
    },
    /**
     * @private
     */
    removeLinks: function () {
        this.getChildren().forEach((child) => {
            child.destroy();
        });
    },
    /**
     * @private
     */
    _updateNotification: function () {
        if (this.getChildren().length === 0) {
            var message = _t("You don't have any recent links.");
            $('.o_website_links_recent_links_notification').html('<div class="alert alert-info">' + message + '</div>');
        } else {
            $('.o_website_links_recent_links_notification').empty();
        }
    },
});

publicWidget.registry.websiteLinks = publicWidget.Widget.extend({
    selector: '.o_website_links_create_tracked_url',
    events: {
        'click #filter-newest-links': '_onFilterNewestLinksClick',
        'click #filter-most-clicked-links': '_onFilterMostClickedLinksClick',
        'click #filter-recently-used-links': '_onFilterRecentlyUsedLinksClick',
        'click #generated_tracked_link a': '_onGeneratedTrackedLinkClick',
        'keyup #url': '_onUrlKeyUp',
        'click #btn_shorten_url': '_onShortenUrlButtonClick',
        'submit #o_website_links_link_tracker_form': '_onFormSubmit',
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: async function () {
        var defs = [this._super.apply(this, arguments)];

        // UTMS selects widgets
        const campaignSelect = new SelectBox(this, "utm.campaign", _t("e.g. June Sale, Paris Roadshow, ..."));
        defs.push(campaignSelect.attachTo($('#campaign-select')));

        const mediumSelect = new SelectBox(this, "utm.medium", _t("e.g. InMails, Ads, Social, ..."));
        defs.push(mediumSelect.attachTo($('#channel-select')));

        const sourceSelect = new SelectBox(this, "utm.source", _t("e.g. LinkedIn, Facebook, Leads, ..."));
        defs.push(sourceSelect.attachTo($('#source-select')));

        // Recent Links Widgets
        this.recentLinks = new RecentLinks(this);
        defs.push(this.recentLinks.appendTo($('#o_website_links_recent_links')));
        this.recentLinks.getRecentLinks('newest');

        this.url_copy_animating = false;

        $('[data-bs-toggle="tooltip"]').tooltip();

        return Promise.all(defs);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onFilterNewestLinksClick: function () {
        this.recentLinks.removeLinks();
        this.recentLinks.getRecentLinks('newest');
    },
    /**
     * @private
     */
    _onFilterMostClickedLinksClick: function () {
        this.recentLinks.removeLinks();
        this.recentLinks.getRecentLinks('most-clicked');
    },
    /**
     * @private
     */
    _onFilterRecentlyUsedLinksClick: function () {
        this.recentLinks.removeLinks();
        this.recentLinks.getRecentLinks('recently-used');
    },
    /**
     * @private
     */
    _onGeneratedTrackedLinkClick: function () {
        $('#generated_tracked_link a').text(_t("Copied")).removeClass('btn-primary').addClass('btn-success');
        setTimeout(function () {
            $('#generated_tracked_link a').text(_t("Copy")).removeClass('btn-success').addClass('btn-primary');
        }, 5000);
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onUrlKeyUp: function (ev) {
        if (!$('#btn_shorten_url').hasClass('btn-copy') || ev.key === "Enter") {
            return;
        }

        $('#btn_shorten_url').removeClass('btn-success btn-copy').addClass('btn-primary').html('Get tracked link');
        $('#generated_tracked_link').css('display', 'none');
        $('.o_website_links_utm_forms').show();
    },
    /**
     * @private
     */
    _onShortenUrlButtonClick: async function (ev) {
        const textValue = ev.target.dataset.clipboardText;
        await browser.navigator.clipboard.writeText(textValue);

        if (!$('#btn_shorten_url').hasClass('btn-copy') || this.url_copy_animating) {
            return;
        }

        var self = this;
        this.url_copy_animating = true;
        $('#generated_tracked_link').clone()
            .css('position', 'absolute')
            .css('left', '78px')
            .css('bottom', '8px')
            .css('z-index', 2)
            .removeClass('#generated_tracked_link')
            .addClass('url-animated-link')
            .appendTo($('#generated_tracked_link'))
            .animate({
                opacity: 0,
                bottom: '+=20',
            }, 500, function () {
                $('.url-animated-link').remove();
                self.url_copy_animating = false;
            });
    },
    /**
     * Add the RecentLinkBox widget and send the form when the user generate the link
     *
     * @private
     * @param {Event} ev
     */
    _onFormSubmit: function (ev) {
        var self = this;
        ev.preventDefault();

        if ($('#btn_shorten_url').hasClass('btn-copy')) {
            return;
        }

        ev.stopPropagation();

        // Get URL and UTMs
        var campaignID = $('#campaign-select').attr('value');
        var mediumID = $('#channel-select').attr('value');
        var sourceID = $('#source-select').attr('value');

        var params = {};
        params.url = $('#url').val();
        if (campaignID !== '') {
            params.campaign_id = parseInt(campaignID);
        }
        if (mediumID !== '') {
            params.medium_id = parseInt(mediumID);
        }
        if (sourceID !== '') {
            params.source_id = parseInt(sourceID);
        }

        $('#btn_shorten_url').text(_t("Generating link..."));

        this.rpc('/website_links/new', params).then(function (result) {
            if ('error' in result) {
                // Handle errors
                if (result.error === 'empty_url') {
                    $('.notification').html('<div class="alert alert-danger">The URL is empty.</div>');
                } else if (result.error === 'url_not_found') {
                    $('.notification').html('<div class="alert alert-danger">URL not found (404)</div>');
                } else {
                    $('.notification').html('<div class="alert alert-danger">An error occur while trying to generate your link. Try again later.</div>');
                }
            } else {
                // Link generated, clean the form and show the link
                var link = result[0];

                $('#btn_shorten_url').removeClass('btn-primary').addClass('btn-success btn-copy').html('Copy');
                $('#btn_shorten_url').attr('data-clipboard-text', link.short_url);

                $('.notification').html('');
                $('#generated_tracked_link').html(link.short_url);
                $('#generated_tracked_link').css('display', 'inline');

                self.recentLinks._addLink(link);

                // Clean URL and UTM selects
                $('#campaign-select').select2('val', '');
                $('#channel-select').select2('val', '');
                $('#source-select').select2('val', '');

                $('.o_website_links_utm_forms').hide();
            }
        });
    },
});

export default {
    SelectBox: SelectBox,
    RecentLinkBox: RecentLinkBox,
    RecentLinks: RecentLinks,
};

```

## File: static\src\js\website_links_charts.js

```javascript
/** @odoo-module **/

import { loadBundle } from "@web/core/assets";
import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { browser } from "@web/core/browser/browser";
const { DateTime } = luxon;

var BarChart = publicWidget.Widget.extend({
    /**
     * @constructor
     * @param {Object} parent
     * @param {Object} beginDate
     * @param {Object} endDate
     * @param {Object} dates
     */
    init: function (parent, beginDate, endDate, dates) {
        this._super.apply(this, arguments);
        this.beginDate = beginDate.startOf("day");
        this.endDate = endDate.startOf("day");
        if (this.beginDate.toISO() === this.endDate.toISO()) {
            this.endDate = this.endDate.plus({ days: 1 });
        }
        this.number_of_days = this.endDate.diff(this.beginDate).as("days");
        this.dates = dates;
    },
    /**
     * @override
     */
    start: function () {
        // Fill data for each day (with 0 click for days without data)
        var clicksArray = [];
        for (var i = 0; i <= this.number_of_days; i++) {
            var dateKey = this.beginDate.toFormat("yyyy-MM-dd");
            clicksArray.push([dateKey, (dateKey in this.dates) ? this.dates[dateKey] : 0]);
            this.beginDate = this.beginDate.plus({ days: 1 });
        }

        var nbClicks = 0;
        var data = [];
        var labels = [];
        clicksArray.forEach(function (pt) {
            labels.push(pt[0]);
            nbClicks += pt[1];
            data.push(pt[1]);
        });

        this.$('.title').html(nbClicks + _t(' clicks'));

        var config = {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    data: data,
                    fill: 'start',
                    label: _t('# of clicks'),
                    backgroundColor: '#ebf2f7',
                    borderColor: '#6aa1ca',

                }],
            },
        };
        var canvas = this.$('canvas')[0];
        var context = canvas.getContext('2d');
        new Chart(context, config);
    },
    willStart: async function () {
        await loadBundle("web.chartjs_lib");
    },
});

var PieChart = publicWidget.Widget.extend({
    /**
     * @override
     * @param {Object} parent
     * @param {Object} data
     */
    init: function (parent, data) {
        this._super.apply(this, arguments);
        this.data = data;
    },
    /**
     * @override
     */
    start: function () {

        // Process country data to fit into the ChartJS scheme
        var labels = [];
        var data = [];
        for (var i = 0; i < this.data.length; i++) {
            var countryName = this.data[i]['country_id'] ? this.data[i]['country_id'][1] : _t('Undefined');
            labels.push(countryName + ' (' + this.data[i]['country_id_count'] + ')');
            data.push(this.data[i]['country_id_count']);
        }

        // Set title
        this.$('.title').html(this.data.length + _t(' countries'));

        var config = {
            type: 'pie',
            data: {
                labels: labels,
                datasets: [{
                    data: data,
                    label: this.data.length > 0 ? this.data[0].key : _t('No data'),
                }]
            },
            options: {
                aspectRatio: 2,
            },
        };

        var canvas = this.$('canvas')[0];
        var context = canvas.getContext('2d');
        new Chart(context, config);
    },
    willStart: async function () {
        await loadBundle("web.chartjs_lib");
    },
});

publicWidget.registry.websiteLinksCharts = publicWidget.Widget.extend({
    selector: '.o_website_links_chart',
    events: {
        'click .copy-to-clipboard': '_onCopyToClipboardClick',
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    /**
     * @override
     */
    start: async function () {
        var self = this;
        this.charts = {};

        // Get the code of the link
        var linkID = parseInt($('#link_id').val());
        this.links_domain = ['link_id', '=', linkID];

        var defs = [];
        defs.push(this._totalClicks());
        defs.push(this._clicksByDay());
        defs.push(this._clicksByCountry());
        defs.push(this._lastWeekClicksByCountry());
        defs.push(this._lastMonthClicksByCountry());
        defs.push(this._super.apply(this, arguments));

        this.animating_copy = false;

        return Promise.all(defs).then(function (results) {
            var _totalClicks = results[0];
            var _clicksByDay = results[1];
            var _clicksByCountry = results[2];
            var _lastWeekClicksByCountry = results[3];
            var _lastMonthClicksByCountry = results[4];

            if (!_totalClicks) {
                $('#all_time_charts').prepend(_t("There is no data to show"));
                $('#last_month_charts').prepend(_t("There is no data to show"));
                $('#last_week_charts').prepend(_t("There is no data to show"));
                return;
            }

            var formattedClicksByDay = {};
            var beginDate;
            for (var i = 0; i < _clicksByDay.length; i++) {
                // This is a trick to get the date without the local formatting.
                // We can't simply do .locale("en") because some Odoo languages
                // are not supported by moment.js (eg: Arabic Syria).
                // FIXME this now uses luxon, check if this is still needed? Probably can be replaced by deserializeDate
                const date = DateTime.fromFormat(
                    _clicksByDay[i]["__domain"].find((el) => el.length && el.includes(">="))[2]
                        .split(" ")[0], "yyyy-MM-dd"
                );
                if (i === 0) {
                    beginDate = date;
                }
                formattedClicksByDay[date.setLocale("en").toFormat("yyyy-MM-dd")] =
                    _clicksByDay[i]["create_date_count"];
            }

            // Process all time line chart data
            var now = DateTime.now();
            self.charts.all_time_bar = new BarChart(self, beginDate, now, formattedClicksByDay);
            self.charts.all_time_bar.attachTo($('#all_time_clicks_chart'));

            // Process month line chart data
            beginDate = DateTime.now().minus({ days: 30 });
            self.charts.last_month_bar = new BarChart(self, beginDate, now, formattedClicksByDay);
            self.charts.last_month_bar.attachTo($('#last_month_clicks_chart'));

            // Process week line chart data
            beginDate = DateTime.now().minus({ days: 7 });
            self.charts.last_week_bar = new BarChart(self, beginDate, now, formattedClicksByDay);
            self.charts.last_week_bar.attachTo($('#last_week_clicks_chart'));

            // Process pie charts
            self.charts.all_time_pie = new PieChart(self, _clicksByCountry);
            self.charts.all_time_pie.attachTo($('#all_time_countries_charts'));

            self.charts.last_month_pie = new PieChart(self, _lastMonthClicksByCountry);
            self.charts.last_month_pie.attachTo($('#last_month_countries_charts'));

            self.charts.last_week_pie = new PieChart(self, _lastWeekClicksByCountry);
            self.charts.last_week_pie.attachTo($('#last_week_countries_charts'));

            var rowWidth = $('#all_time_countries_charts').parent().width();
            var $chartCanvas = $('#all_time_countries_charts,last_month_countries_charts,last_week_countries_charts').find('canvas');
            $chartCanvas.height(Math.max(_clicksByCountry.length * (rowWidth > 750 ? 1 : 2), 20) + 'em');

        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _totalClicks: function () {
        return this.orm.searchCount("link.tracker.click", [this.links_domain]);
    },
    /**
     * @private
     */
    _clicksByDay: function () {
        return this.orm.readGroup(
            "link.tracker.click",
            [this.links_domain],
            ["create_date"],
            ["create_date:day"]
        );
    },
    /**
     * @private
     */
    _clicksByCountry: function () {
        return this.orm.readGroup(
            "link.tracker.click",
            [this.links_domain],
            ["country_id"],
            ["country_id"]
        );
    },
    /**
     * @private
     */
    _lastWeekClicksByCountry: function () {
        // 7 days * 24 hours * 60 minutes * 60 seconds * 1000 milliseconds.
        const aWeekAgoDate = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
        // get the date in the format YYYY-MM-DD.
        const aWeekAgoString = aWeekAgoDate.toISOString().split("T")[0];
        return this.orm.readGroup(
            "link.tracker.click",
            [this.links_domain, ["create_date", ">", aWeekAgoString]],
            ["country_id"],
            ["country_id"]
        );
    },
    /**
     * @private
     */
    _lastMonthClicksByCountry: function () {
        // 30 days * 24 hours * 60 minutes * 60 seconds * 1000 milliseconds.
        const aMonthAgoDate = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
        // get the date in the format YYYY-MM-DD.
        const aMonthAgoString = aMonthAgoDate.toISOString().split("T")[0];
        return this.orm.readGroup(
            "link.tracker.click",
            [this.links_domain, ["create_date", ">", aMonthAgoString]],
            ["country_id"],
            ["country_id"]
        );
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onCopyToClipboardClick: async function (ev) {
        ev.preventDefault();

        const textValue = ev.target.dataset.clipboardText;
        await browser.navigator.clipboard.writeText(textValue);

        if (this.animating_copy) {
            return;
        }

        this.animating_copy = true;

        $('.o_website_links_short_url').clone()
            .css('position', 'absolute')
            .css('left', '15px')
            .css('bottom', '10px')
            .css('z-index', 2)
            .removeClass('.o_website_links_short_url')
            .addClass('animated-link')
            .appendTo($('.o_website_links_short_url'))
            .animate({
                opacity: 0,
                bottom: '+=20',
            }, 500, function () {
                $('.animated-link').remove();
                this.animating_copy = false;
            });
    },
});

export default {
    BarChart: BarChart,
    PieChart: PieChart,
};

```

## File: static\src\js\website_links_code_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.websiteLinksCodeEditor = publicWidget.Widget.extend({
    selector: '#wrapwrap:has(.o_website_links_edit_code)',
    events: {
        'click .o_website_links_edit_code': '_onEditCodeClick',
        'click .o_website_links_cancel_edit': '_onCancelEditClick',
        'submit #edit-code-form': '_onEditCodeFormSubmit',
        'click .o_website_links_ok_edit': '_onEditCodeFormSubmit',
    },

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {String} newCode
     */
    _showNewCode: function (newCode) {
        $('.o_website_links_code_error').html('');
        $('.o_website_links_code_error').hide();

        $('#o_website_links_code form').remove();

        // Show new code
        var host = $('#short-url-host').html();
        $('#o_website_links_code').html(newCode);

        // Update button copy to clipboard
        $('.copy-to-clipboard').attr('data-clipboard-text', host + newCode);

        // Show action again
        $('.o_website_links_edit_code').show();
        $('.copy-to-clipboard').show();
        $('.o_website_links_edit_tools').hide();
    },
    /**
     * @private
     * @returns {Promise}
     */
    _submitCode: function () {
        var initCode = $('#edit-code-form #init_code').val();
        var newCode = $('#edit-code-form #new_code').val();
        var self = this;

        if (newCode === '') {
            self.$('.o_website_links_code_error').html(_t("The code cannot be left empty"));
            self.$('.o_website_links_code_error').show();
            return;
        }

        this._showNewCode(newCode);

        if (initCode === newCode) {
            this._showNewCode(newCode);
        } else {
            return this.rpc('/website_links/add_code', {
                init_code: initCode,
                new_code: newCode,
            }).then(function (result) {
                self._showNewCode(result[0].code);
            }, function () {
                $('.o_website_links_code_error').show();
                $('.o_website_links_code_error').html(_t("This code is already taken"));
            });
        }

        return Promise.resolve();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onEditCodeClick: function () {
        var initCode = $('#o_website_links_code').html();
        $('#o_website_links_code').html('<form style="display:inline;" id="edit-code-form"><input type="hidden" id="init_code" value="' + initCode + '"/><input type="text" id="new_code" value="' + initCode + '"/></form>');
        $('.o_website_links_edit_code').hide();
        $('.copy-to-clipboard').hide();
        $('.o_website_links_edit_tools').show();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCancelEditClick: function (ev) {
        ev.preventDefault();
        $('.o_website_links_edit_code').show();
        $('.copy-to-clipboard').show();
        $('.o_website_links_edit_tools').hide();
        $('.o_website_links_code_error').hide();

        var oldCode = $('#edit-code-form #init_code').val();
        $('#o_website_links_code').html(oldCode);

        $('#code-error').remove();
        $('#o_website_links_code form').remove();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onEditCodeFormSubmit: function (ev) {
        ev.preventDefault();
        this._submitCode();
    },
});

```

## File: static\src\services\website_custom_menus.js

```javascript
/** @odoo-module  */

import { registry } from '@web/core/registry';

registry.category('website_custom_menus').add('website_links.menu_link_tracker', {
    openWidget: (services) => services.website.goToWebsite({ path: `/r?u=${encodeURIComponent(services.website.contentWindow.location.href)}` }),
    isDisplayed: (env) => env.services.website.currentWebsite && env.services.website.contentWindow,
});

```

## File: static\src\xml\recent_link.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="website_links.RecentLink">
        <div class="row mb16">
            <div class="col-md-1 col-2 text-center">
                <h4><t t-esc="widget.link_obj.count"/></h4>
                <p class="text-muted" style="margin-top: -5px;">clicks</p>
            </div>
            <div class="col-md-7 col-7">
                <h4 class="truncate_text">
                    <img t-attf-src="http://www.google.com/s2/favicons?domain={{ widget.link_obj.url }}" loading="lazy" alt="Icon" onerror="this.src='/website_links/static/img/default_favicon.png'"/>
                    <a class="no-link-style" t-att-href="widget.link_obj.url"><t t-esc="widget.link_obj.title"/></a>
                </h4>
                <p class="text-muted mb0" style="margin-top: -5px;">
                    <span class="o_website_links_short_url text-muted" style="position:relative;">
                        <span id="o_website_links_host"><t t-esc="widget.link_obj.short_url_host"/></span><span id="o_website_links_code"><t t-esc="widget.link_obj.code"/></span>
                    </span>

                    <span class="o_website_links_edit_tools" style="display:none;">
                        <a role="button" class="o_website_links_ok_edit btn btn-sm btn-primary" href="#">ok</a> or
                        <a class="o_website_links_cancel_edit" href="#">cancel</a>
                    </span>

                    <a class="o_website_links_edit_code" aria-label="Edit code" title="Edit code"><span class="fa fa-pencil gray"></span></a>

                    <br/>
                    <span class="badge text-bg-success"><t t-esc="widget.link_obj.campaign_id[1]"/></span>
                    <span class="badge text-bg-success"><t t-esc="widget.link_obj.medium_id[1]"/></span>
                    <span class="badge text-bg-success"><t t-esc="widget.link_obj.source_id[1]"/></span>
                </p>
                <p class='o_website_links_code_error' style='color:red;font-weight:bold;'></p>
            </div>

            <div class="col-md-4 col-2">
                <button class="btn btn-info btn_shorten_url_clipboard mt8">Copy</button>
                <a role="button" t-attf-href="{{widget.link_obj.short_url}}+" class="btn btn-warning mt8">Stats</a>
            </div>
        </div>
    </t>
</templates>

```

## File: views\link_tracker_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="link_tracker_view_tree" model="ir.ui.view">
        <field name="name">link.tracker.view.tree.inherit.website.links</field>
        <field name="model">link.tracker</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_visit_page']" position="after">
                <button name="action_visit_page_statistics" type="object" string="Statistics" icon="fa-bar-chart"/>
            </xpath>
        </field>
    </record>

    <menuitem id="menu_link_tracker"
        name="Link Tracker"
        sequence="25"
        parent="website.menu_current_page"
        action="website.website_preview"/>
</odoo>

```

## File: views\website_links_graphs.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="graphs" name="Link Statistics">
            <t t-call="website.layout">
                <div class="o_website_links_chart">
                    <div class="container">
                        <div class="mt8">
                            <ol class="breadcrumb">
                                <li class="breadcrumb-item"><a href="/r">Link Tracker</a></li>
                                <li class="breadcrumb-item active"><t t-esc="title"/></li>
                            </ol>
                        </div>

                        <input type="hidden" id="code" t-att-value="code" />
                        <input type="hidden" id="link_id" t-att-value="id" />

                        <h1 class="o_page_header mt0"><t t-esc="title"/></h1>

                        <div class="row">
                            <div class="col-md-2">
                                <p><strong>Original URL</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p class="truncate_text mw-100" t-att-title="url"><a t-att-href="url"><t t-esc="url"/></a></p>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-2">
                                <p><strong>Tracked Link</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p>
                                    <span class="o_website_links_short_url" id="short_url"><span id="short-url-host"><t t-esc="short_url_host"/></span><span id="o_website_links_code"><t t-esc="code"/></span></span>
                                    <span class="o_website_links_edit_tools" style="display:none;">
                                        <a role="button" class="o_website_links_ok_edit btn btn-sm btn-primary" href="#">ok</a> or 
                                        <a class="o_website_links_cancel_edit" href="#">cancel</a>
                                    </span>
                                    <a class="o_website_links_edit_code" aria-label="Edit code" title="Edit code"><span class="fa fa-pencil gray"></span></a>
                                    <a class="copy-to-clipboard" t-att-data-clipboard-text="short_url">copy</a>

                                </p>
                                <p class='o_website_links_code_error' style='color:red;font-weight:bold;display:none'></p>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-2">
                                <p><strong>Redirected URL</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p class="truncate_text mw-100" t-att-title="redirected_url">
                                    <t t-esc="redirected_url"/>
                                </p>
                            </div>
                        </div>

                        <t t-if="campaign_id">
                            <div class="row">
                                <div class="col-md-2">
                                    <p><strong>Campaign</strong></p>
                                </div>
                                <div class="col-md-10">
                                    <p><t t-esc="campaign_id[1]"/></p>
                                </div>
                            </div>
                        </t>

                        <t t-if="medium_id">
                            <div class="row">
                                <div class="col-md-2">
                                    <p><strong>Medium</strong></p>
                                </div>
                                <div class="col-md-10">
                                    <p><t t-esc="medium_id[1]"/></p>
                                </div>
                            </div>
                        </t>

                        <t t-if="source_id">
                            <div class="row">
                                <div class="col-md-2">
                                    <p><strong>Source</strong></p>
                                </div>
                                <div class="col-md-10">
                                    <p><t t-esc="source_id[1]"/></p>
                                </div>
                            </div>
                        </t>


                        <h1 class="o_page_header">Statistics
                            <small class="float-end d-none d-md-block mt16" id="filters">
                                <ul class="nav nav-tabs nav-tabs-inline graph-tabs" role="tablist">
                                    <li class="nav-item"><a aria-controls="all_time_charts" href="#all_time_charts" class="nav-link active" role="tab" data-bs-toggle="tab">All Time</a></li>
                                    <li class="nav-item"><a aria-controls="last_month_charts" href="#last_month_charts" class="nav-link" role="tab" data-bs-toggle="tab">Last Month</a></li>
                                    <li class="nav-item"><a aria-controls="last_week_charts" href="#last_week_charts" class="nav-link" role="tab" data-bs-toggle="tab">Last Week</a></li>
                                </ul>
                            </small>
                        </h1>

                        <div class="mb128">
                            <div class="tab-content">
                                <!-- All Time Charts -->
                                <div role="tabpanel" class="tab-pane active" id="all_time_charts">
                                    <div class="website_links_click_chart" id="all_time_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart" id="all_time_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                </div>

                                <!-- Last Month Charts -->
                                <div role="tabpanel" class="tab-pane" id="last_month_charts">
                                    <div class="website_links_click_chart" id="last_month_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart" id="last_month_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                </div>

                                <!-- Last Week Charts -->
                                <div role="tabpanel" class="tab-pane" id="last_week_charts">
                                    <div class="website_links_click_chart" id="last_week_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart" id="last_week_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:20em;"></canvas>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>
</odoo>

```

## File: views\website_links_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="create_shorten_url">
            <div class="o_website_links_create_tracked_url">
                <div class="container">
                    <h1 class="o_page_header">Link Tracker</h1>
                    <div class="notification"></div>

                    <div class="row">
                        <div class="col-md-7">
                            <form id="o_website_links_link_tracker_form">

                                <div class="mb-3 row">
                                    <label class="col-md-3 col-form-label text-start">URL</label>

                                    <div class="col-md-9">
                                        <input type="text" id="url" class="form-control required-form-control"  required="True" placeholder="e.g. https://www.odoo.com/contactus" t-att-value="u"/>
                                    </div>
                                </div>

                                <div class="o_website_links_utm_forms">
                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Campaign <i class="fa fa-info-circle" data-bs-toggle="tooltip" data-bs-placement="top" role="img" aria-label="Tooltip info" title="Defines the context of your link. It might be an event you want to promote or a special promotion."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="campaign-select"/>
                                        </div>
                                    </div>

                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Medium <i class="fa fa-info-circle" data-bs-toggle="tooltip" data-bs-placement="top" role="img" aria-label="Tooltip info" title="Defines the medium used to share your link. It might be an email, or a Facebook Ads for instance."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="channel-select" />
                                        </div>
                                    </div>

                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Source <i class="fa fa-info-circle" data-bs-toggle="tooltip" data-bs-placement="top" role="img" aria-label="Tooltip info" title="Defines the source from which your traffic will come from, Facebook or Twitter for instance."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="source-select" />
                                        </div>
                                    </div>
                                </div>

                                <div class="mb-3 row">
                                    <div class="offset-md-3 col-md-9">
                                        <button type="submit" class="btn btn-primary" id="btn_shorten_url" data-clipboard-text="">Get tracked link</button>

                                        <span id="generated_tracked_link" style="display:none;" class="text-muted"></span>
                                    </div>
                                </div>
                            </form>
                        </div>

                        <div class="offset-md-1 col-md-3 d-none d-md-block">
                            <p class="text-muted text-justify">Share this page with a <strong>short link</strong> that includes <strong>analytics trackers</strong>.</p>
                            <p class="text-muted text-justify">Those trackers can be used in Google Analytics to track clicks and visitors, or in Odoo reports to track opportunities and related revenues.</p>
                        </div>
                    </div>

                    <h2 class="o_page_header">Your tracked links
                        <small class="float-end d-none d-md-block" id="filters">
                            <ul class="nav nav-tabs nav-tabs-inline graph-tabs" role="tablist">
                                <li class="nav-item"><a aria-controls="filter-newest-links" href="#" class="nav-link active" id="filter-newest-links" role="tab" data-bs-toggle="tab">Newest</a></li>
                                <li class="nav-item"><a aria-controls="filter-most-clicked-links" href="#" class="nav-link" id="filter-most-clicked-links" role="tab" data-bs-toggle="tab">Most Clicked</a></li>
                                <li class="nav-item"><a aria-controls="filter-recently-used-links" href="#" class="nav-link" id="filter-recently-used-links" role="tab" data-bs-toggle="tab">Recently Used</a></li>
                            </ul>
                        </small>
                    </h2>

                    <div id="o_website_links_recent_links">
                        <div class="o_website_links_recent_links_notification"></div>
                    </div>
                </div>
            </div>
        </template>

        <template id="page_shorten_url" name="Link Tracker">
            <t t-call="website.layout">
                <t t-call="website_links.create_shorten_url"/>
            </t>
        </template>

</odoo>

```

