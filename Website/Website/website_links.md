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
        return request.env['link.tracker'].search_or_create([post]).read()

    @http.route('/r', type='http', auth='user', website=True)
    def shorten_url(self, **post):
        return request.render("website_links.page_shorten_url", {
            "can_create_link_tracker": request.env['link.tracker'].has_access('create'),
            "can_create_link_tracker_code": request.env['link.tracker.code'].has_access('create'),
            **post,
        })

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
            return request.render("website_links.graphs", {
                "can_create_link_tracker_code": request.env['link.tracker.code'].has_access('create'),
                **code.link_id.read()[0]
            })
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
import { Component, onWillStart, useState } from "@odoo/owl";
import publicWidget from "@web/legacy/js/public/public_widget";
import { addLoadingEffect } from '@web/core/utils/ui';
import { browser } from "@web/core/browser/browser";
import { rpc } from "@web/core/network/rpc";
import { KeepLast } from "@web/core/utils/concurrency";
import { attachComponent } from "@web_editor/js/core/owl_utils";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import { useService } from "@web/core/utils/hooks";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";

class WebsiteLinksTagsWrapper extends Component {
    static template = "website_links.WebsiteLinksTagsWrapper";
    static components = { SelectMenu, DropdownItem };
    static props = {
        placeholder: { optional: true, type: String },
        model: { optional: true, type: String },
    };

    setup() {
        this.orm = useService("orm");
        this.keepLast = new KeepLast();
        this.state = useState({
            placeholder: this.props.placeholder,
            choices: [],
            value: undefined,
        });
        onWillStart(async () => {
            this.canCreateLinkTracker = await this.orm.call(this.props.model, "has_access", [[], "create"]);
            await this.loadChoice();
        });
    }

    get showCreateOption() {
        return this.select.data.searchValue && !this.state.choices.some(c => c.label === this.select.data.searchValue) && this.canCreateLinkTracker;
    }

    onSelect(value) {
        this.state.value = value;
    }

    async onCreateOption(string, closeFn) {
        const record = await this.orm.call("utm.mixin", "find_or_create_record", [
            this.props.model,
            string,
        ]);
        const choice = {
            label: record.name,
            value: record.id,
        };
        this.state.choices.push(choice);
        this.onSelect(choice.value);
    }

    loadChoice(searchString = "") {
        return new Promise((resolve, reject) => {
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
                ["id", "name"],
                {
                    limit: limit,
                    order: "name, id desc", // Allows to have exact match first
                },
            ];
            const proms = [];
            proms.push(
                this.orm.searchRead(
                    this.props.model,
                    // Exact match + results that start with the search
                    [["name", "=ilike", `${searchString}%`]],
                    ...searchReadParams
                )
            );
            proms.push(
                this.orm.searchRead(
                    this.props.model,
                    // Results that contain the search but do not start
                    // with it
                    [["name", "=ilike", `%_${searchString}%`]],
                    ...searchReadParams
                )
            );
            // Keep last is there in case a RPC takes longer than
            // the debounce delay + next rpc delay for some reason.
            this.keepLast
                .add(Promise.all(proms))
                .then(([startingMatches, endingMatches]) => {
                    const formatChoice = (choice) => {
                        choice.value = choice.id;
                        choice.label = choice.name;
                        return choice;
                    };
                    startingMatches.map(formatChoice);

                    // We loaded max a 2 * limit amount of records but
                    // ensure that we do not display "ending matches" if
                    // we may not have loaded all "starting matches".
                    if (startingMatches.length < limit) {
                        const startingMatchesId = startingMatches.map((value) => value.id);
                        const extraEndingMatches = endingMatches.filter(
                            (value) => !startingMatchesId.includes(value.id)
                        );
                        extraEndingMatches.map(formatChoice);
                        return startingMatches.concat(extraEndingMatches);
                    }
                    // In that case, we made one RPC too much but this
                    // was chosen over not making them go in parallel.
                    // We don't want to display "ending matches" if not
                    // all "starting matches" have been loaded.
                    return startingMatches;
                })
                .then((result) => {
                    this.state.choices = result;
                    resolve();
                })
                .catch(reject);
        });
    }
}

var RecentLinkBox = publicWidget.Widget.extend({
    template: 'website_links.RecentLink',
    events: {
        'click .btn_shorten_url_clipboard': '_onCopyShortenUrl',
    },

    /**
     * @constructor
     * @param {Object} parent
     * @param {Object} obj
     */
    init: function (parent, obj) {
        this._super.apply(this, arguments);
        this.link_obj = obj;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onCopyShortenUrl: async function (ev) {
        ev.preventDefault();
        const copyBtn = ev.currentTarget;
        const tooltip = Tooltip.getOrCreateInstance(copyBtn, {
            title: _t("Link Copied!"),
            trigger: "manual",
            placement: "top",
        });
        setTimeout(
            async () => await browser.navigator.clipboard.writeText(copyBtn.dataset.url)
        );
        tooltip.show();
        setTimeout(() => tooltip.hide(), 1200);
    },
});

var RecentLinks = publicWidget.Widget.extend({
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    getRecentLinks: function (filter) {
        var self = this;
        return rpc('/website_links/recent_links', {
            filter: filter,
            limit: 20,
        }).then(function (result) {
            result.reverse().forEach((link) => {
                self._addLink(link);
            });
            self._updateNotification();
            self._updateFilters(filter);
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
     * Updates the dropdown with the selected filter
     */
    _updateFilters: function(filter) {
        const dropdownBtns = document.querySelectorAll('#recent_links_sort_by a');
        dropdownBtns.forEach((button) => {
            if (button.dataset.filter === filter) {
                document.querySelector('.o_website_links_sort_by').textContent = button.textContent;
                button.classList.add('active');
            } else {
                button.classList.remove('active');
            }
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
        'click #recent_links_sort_by a': '_onRecentLinksFilterChange',
        'click .o_website_links_new_link_tracker': '_onCreateNewLinkTrackerClick',
        'submit #o_website_links_link_tracker_form': '_onFormSubmit',
    },

    /**
     * @override
     */
    start: async function () {
        var defs = [this._super.apply(this, arguments)];

        async function attachSelectComponent(model, placeholderText, el) {
            const props = {
                placeholder: placeholderText,
                model: model,
            };
            await attachComponent(this, el, WebsiteLinksTagsWrapper, props);
        }

        attachSelectComponent.call(
            this,
            "utm.campaign",
            _t("e.g. June Sale, Paris Roadshow, ..."),
            this.el.querySelector("#campaign-select-wrapper"),
        );
        attachSelectComponent.call(
            this,
            "utm.medium",
            _t("e.g. InMails, Ads, Social, ..."),
            this.el.querySelector("#channel-select-wrapper"),
        );
        attachSelectComponent.call(
            this,
            "utm.source",
            _t("e.g. LinkedIn, Facebook, Leads, ..."),
            this.el.querySelector("#source-select-wrapper"),
        );

        // Recent Links Widgets
        this.recentLinks = new RecentLinks(this);
        defs.push(this.recentLinks.appendTo($('#o_website_links_recent_links')));
        this.recentLinks.getRecentLinks('newest');

        $('[data-bs-toggle="tooltip"]').tooltip();

        return Promise.all(defs);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onRecentLinksFilterChange(ev) {
        this.recentLinks.removeLinks();
        this.recentLinks.getRecentLinks(ev.currentTarget.dataset.filter);
    },
    /**
     * @private
     * @param {Event} ev
     * Show the link tracker form back
     */
    _onCreateNewLinkTrackerClick: function (ev) {
        const utmForm = document.querySelector(".o_website_links_utm_forms");
        if (!utmForm.classList.contains("d-none")) {
            return;
        }
        utmForm.classList.remove("d-none");
        document.querySelector("#generated_tracked_link").classList.add("d-none");
        document.querySelector("#btn_shorten_url").classList.remove("d-none");
        document.querySelector("input#url").value = '';
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
        const generateLinkTrackerBtn = document.querySelector("#btn_shorten_url");
        if (generateLinkTrackerBtn.classList.contains("d-none")) {
            return;
        }
        const restoreLoadingBtn = addLoadingEffect(generateLinkTrackerBtn);

        ev.stopPropagation();

        // Get URL and UTMs
        const campaignInputEl = document.querySelector("input[name='campaign-select']");
        const mediumInputEl = document.querySelector("input[name='medium-select']");
        const sourceInputEl = document.querySelector("input[name='source-select']");

        const label = document.querySelector('#label');
        const params = { label: label.value || undefined };
        params.url = $('#url').val();
        if (campaignInputEl.value !== "") {
            params.campaign_id = parseInt(campaignInputEl.value);
        }
        if (mediumInputEl.value !== "") {
            params.medium_id = parseInt(mediumInputEl.value);
        }
        if (sourceInputEl.value !== "") {
            params.source_id = parseInt(sourceInputEl.value);
        }

        rpc('/website_links/new', params).then(function (result) {
            restoreLoadingBtn();
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

                document.querySelector("#generated_tracked_link").classList.remove("d-none");
                document.querySelector("#btn_shorten_url").classList.add("d-none");

                document.querySelector(".copy-to-clipboard").dataset.clipboardText = link.short_url;
                document.querySelector("#short-url-host").textContent = link.short_url_host;
                document.querySelector("#o_website_links_code").textContent = link.code;

                self.recentLinks._addLink(link);

                // Clean notifications, URL and UTM selects
                $('.notification').html('');
                campaignInputEl.value = "";
                mediumInputEl.value = "";
                sourceInputEl.value = "";
                label.value = '';
                document.querySelector(".o_website_links_utm_forms").classList.add("d-none");
            }
        });
    },
});

export default {
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

        this.$('.title').text(_t('%(clicks)s clicks', {clicks: nbClicks}));

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
            options: {
                scales: {
                    y: {
                        ticks: {
                            callback: function(value) {
                                if (Number.isInteger(value)) {
                                    return value;
                                }
                            },
                        }
                    }
                }
            }
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
        this.$('.title').text(_t('%(count)s countries', {count: this.data.length}));

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
import { browser } from "@web/core/browser/browser";
import publicWidget from "@web/legacy/js/public/public_widget";
import { rpc } from "@web/core/network/rpc";

publicWidget.registry.websiteLinksCodeEditor = publicWidget.Widget.extend({
    selector: '#wrapwrap',
    selectorHas: '.o_website_links_edit_code',
    events: {
        'click .copy-to-clipboard': '_onCopyToClipboardClick',
        'click .o_website_links_edit_code': '_onEditCodeClick',
        'click .o_website_links_cancel_edit': '_onCancelEditClick',
        'submit #edit-code-form': '_onEditCodeFormSubmit',
        'click .o_website_links_ok_edit': '_onEditCodeFormSubmit',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onCopyToClipboardClick: async function (ev) {
        ev.preventDefault();
        const copyBtn = ev.currentTarget;
        const tooltip = Tooltip.getOrCreateInstance(copyBtn, {
            title: _t("Link Copied!"),
            trigger: "manual",
            placement: "right",
        });
        setTimeout(
            async () => await browser.navigator.clipboard.writeText(copyBtn.dataset.clipboardText)
        );
        tooltip.show();
        setTimeout(() => tooltip.hide(), 1200);
    },

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
            return rpc('/website_links/add_code', {
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
        <a class="o_website_links_card row mb16 mx-1 py-4 border rounded flex-nowrap text-decoration-none text-black" t-attf-href="{{widget.link_obj.short_url}}+">
            <div class="col-2 col-lg-1 text-center">
                <h4 class="mb-0" t-out="widget.link_obj.count"/>
                <p class="text-muted mb-0">clicks</p>
            </div>
            <div class="col-8 col-lg-9 d-flex flex-column flex-lg-row">
                <div class="w-lg-50 mb-3 mb-lg-0 pe-lg-4">
                    <h4 class="o_website_links_title text-truncate" t-out="widget.link_obj.title"/>
                    <div t-if="widget.link_obj.campaign_id or widget.link_obj.medium_id or widget.link_obj.source_id" class="d-flex flex-wrap gap-1">
                        <span class="badge text-bg-secondary fw-normal" t-out="widget.link_obj.campaign_id[1]"/>
                        <span class="badge text-bg-secondary fw-normal" t-out="widget.link_obj.medium_id[1]"/>
                        <span class="badge text-bg-secondary fw-normal" t-out="widget.link_obj.source_id[1]"/>
                    </div>
                </div>
                <div class="w-lg-50 row align-items-center ps-3">
                    <span class="o_website_links_short_url p-0 d-inline-flex flex-nowrap align-items-stretch border border-primary rounded">
                        <span t-out="widget.link_obj.short_url" class="flex-grow-1 small text-muted text-truncate px-2 py-1 lh-lg"/>
                        <button class="btn btn-primary rounded-0 btn_shorten_url_clipboard py-1" t-att-data-url="widget.link_obj.short_url">
                            <small>Copy</small>
                        </button>
                    </span>
                </div>
            </div>
            <div class="col-2 d-flex align-items-start align-items-lg-center justify-content-center text-break">
                <a role="button" target="_blank" t-att-href="widget.link_obj.url" class="btn btn-secondary">
                    <span class="d-none d-lg-inline-block me-1">Visit Link</span><i class="fa fa-external-link"/>
                </a>
            </div>
        </a>
    </t>
</templates>

```

## File: static\src\xml\website_links_tags_wrapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<t t-name="website_links.WebsiteLinksTagsWrapper">
    <input t-att-name="props.model.split('.')[1]+'-select'" type="hidden" class="form-control" t-att-value="state.value"/>
    <SelectMenu
        t-props="state"
        onInput.bind="loadChoice"
        onSelect.bind="onSelect">
        <t t-if="showCreateOption" t-set-slot="bottomArea" t-slot-scope="select">
            <DropdownItem
                onSelected="() => this.onCreateOption(select.data.searchValue)"
                class="'o_select_menu_item p-2'"
            >
                Create option "<span class="fw-bold text-muted" t-out="select.data.searchValue"/>"
            </DropdownItem>
        </t>
    </SelectMenu>
</t>

</templates>

```

## File: views\link_tracker_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="link_tracker_view_tree" model="ir.ui.view">
        <field name="name">link.tracker.view.list.inherit.website.links</field>
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
                                <li class="breadcrumb-item active truncate_text"><t t-esc="title"/></li>
                            </ol>
                        </div>

                        <input type="hidden" id="code" t-att-value="code" />
                        <input type="hidden" id="link_id" t-att-value="id" />

                        <h1 class="o_page_header mt0 text-truncate"><t t-esc="title"/></h1>

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
                                    <a t-attf-class="#{'' if can_create_link_tracker_code else 'd-none'} o_website_links_edit_code" aria-label="Edit code" title="Edit code"><span class="fa fa-pencil gray"></span></a>
                                    <a class="copy-to-clipboard btn btn-sm btn-primary" t-att-data-clipboard-text="short_url"><i class="fa fa-copy me-2"/>Copy</a>
                                </p>
                                <p class='o_website_links_code_error' style='color:red;font-weight:bold;display:none'></p>
                            </div>
                        </div>

                        <div class="row">
                            <div class="col-md-2">
                                <p><strong>Redirected URL</strong></p>
                            </div>
                            <div class="col-md-10 d-flex flex-nowrap align-items-start gap-1">
                                <p class="truncate_text" t-att-title="redirected_url">
                                    <t t-esc="redirected_url"/>
                                </p>
                                <a class="copy-to-clipboard btn btn-sm btn-primary" t-att-data-clipboard-text="redirected_url"><i class="fa fa-copy me-2"/>Copy</a>
                            </div>
                        </div>
                        <div t-if="campaign_id" class="row">
                            <div class="col-md-2">
                                <p><strong>Campaign</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p><t t-esc="campaign_id[1]"/></p>
                            </div>
                        </div>
                        <div t-if="medium_id" class="row">
                            <div class="col-md-2">
                                <p><strong>Medium</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p><t t-esc="medium_id[1]"/></p>
                            </div>
                        </div>
                        <div t-if="source_id" class="row">
                            <div class="col-md-2">
                                <p><strong>Source</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p><t t-esc="source_id[1]"/></p>
                            </div>
                        </div>
                        <div t-if="label" class="row" >
                            <div class="col-md-2">
                                <p><strong>Name</strong></p>
                            </div>
                            <div class="col-md-10">
                                <p t-out="label"/>
                            </div>
                        </div>

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
                                    <div class="website_links_click_chart mt32" id="all_time_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart mt32" id="all_time_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
                                    </div>
                                </div>

                                <!-- Last Month Charts -->
                                <div role="tabpanel" class="tab-pane" id="last_month_charts">
                                    <div class="website_links_click_chart mt32" id="last_month_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart mt32" id="last_month_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
                                    </div>
                                </div>

                                <!-- Last Week Charts -->
                                <div role="tabpanel" class="tab-pane" id="last_week_charts">
                                    <div class="website_links_click_chart mt32" id="last_week_clicks_chart">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
                                    </div>
                                    <div class="website_links_click_chart mt32" id="last_week_countries_charts">
                                        <h3 class="title"></h3>
                                        <canvas style="height:10em;"></canvas>
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
                    <div class="notification mb-3"/>

                    <div t-attf-class="row mb-4 pt-3 #{'' if can_create_link_tracker else 'd-none'}">
                        <h2 class="o_page_header">Create a Link tracker</h2>
                        <div class="col-md-7">
                            <form id="o_website_links_link_tracker_form">

                                <div class="mb-3 row">
                                    <label class="col-md-3 col-form-label text-start">Target Link</label>
                                    <div class="col-md-9">
                                        <input type="text" id="url" class="form-control required-form-control"  required="True" placeholder="e.g. https://www.odoo.com/contactus" t-att-value="u"/>
                                    </div>
                                </div>

                                <div class="o_website_links_utm_forms">
                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Campaign</label>
                                        <div class="col-md-9" id="campaign-select-wrapper"></div>
                                    </div>
                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Medium</label>
                                        <div class="col-md-9" id="channel-select-wrapper"></div>
                                    </div>
                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Source</label>
                                        <div class="col-md-9" id="source-select-wrapper"></div>
                                    </div>
                                    <div class="mb-3 row">
                                        <label class="col-md-3 col-form-label">Name</label>
                                        <div class="col-md-9">
                                            <input type="text" id="label" class="form-control" placeholder="e.g. &quot;Black Friday Mailing Campaign&quot;" />
                                        </div>
                                    </div>
                                </div>

                                <div class="row">
                                    <div class="offset-md-3 col-md-9">
                                        <button type="submit" class="btn btn-primary mb-3" id="btn_shorten_url">Generate Link Tracker</button>
                                    </div>
                                </div>
                            </form>
                            <div id="generated_tracked_link" class="d-none">
                                <div class="row">
                                    <label class="col-md-3 col-form-label text-start">Tracked Link</label>
                                    <p class="o_website_links_edition col-md-9 d-flex flex-nowrap gap-2 align-items-center mb-0">
                                        <span class="o_website_links_short_url d-flex align-items-center text-nowrap text-truncate" id="short_url"><span id="short-url-host" class="text-truncate"/><span id="o_website_links_code"/></span>
                                        <span class="o_website_links_edit_tools text-nowrap" style="display:none;">
                                            <a role="button" class="o_website_links_ok_edit btn btn-sm btn-primary" href="#">ok</a> or
                                            <a class="o_website_links_cancel_edit" href="#">cancel</a>
                                        </span>
                                        <a t-attf-class="#{'' if can_create_link_tracker_code else 'd-none'} o_website_links_edit_code" aria-label="Edit code" title="Edit code"><i class="fa fa-pencil gray"/></a>
                                        <a class="copy-to-clipboard btn btn-success text-nowrap"><i class="fa fa-copy me-2"/>Copy</a>
                                    </p>
                                </div>
                                <div class="offset-md-3 col-md-9 px-2">
                                    <p class="o_website_links_code_error text-danger fw-bold" style="display:none;"/>
                                    <button class="o_website_links_new_link_tracker btn btn-primary my-3">Create another Tracker</button>
                                </div>
                            </div>
                        </div>

                        <div class="offset-md-1 col-md-4 d-none d-md-block">
                            <p class="text-muted text-justify">Share this page with a <strong>short link</strong> that includes <strong>analytics trackers</strong>.</p>
                            <p class="text-muted text-justify">Those trackers can be used in Google Analytics to track clicks and visitors, or in Odoo reports to track opportunities and related revenues.</p>
                            <a target="_blank" href="https://www.odoo.com/documentation/18.0/applications/websites/website/reporting/link_tracker.html">Read More</a>
                        </div>
                    </div>

                    <div class="o_page_header d-flex justify-content-between align-items-center mb-4 pt-3">
                        <h2>Your tracked links</h2>
                        <small class="d-none d-md-block" id="filters">
                            <span class="text-muted me-1">Sort By:</span>
                            <div class="btn-group">
                                <button data-bs-toggle="dropdown" class="o_website_links_sort_by btn btn-light dropdown-toggle">
                                    Newest
                                </button>
                                <div id="recent_links_sort_by" class="dropdown-menu dropdown-menu-end">
                                    <a data-filter="newest" class="dropdown-item active" href="#">Newest</a>
                                    <a data-filter="most-clicked" class="dropdown-item" href="#">Number of Clicks</a>
                                    <a data-filter="recently-used" class="dropdown-item" href="#">Last Clicks</a>
                                </div>
                            </div>
                        </small>
                    </div>

                    <div id="o_website_links_recent_links" class="pb-4">
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

