# Odoo Module: website_google_map

Category: Website/Website

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
    'name': 'Google Maps',
    'category': 'Website/Website',
    'summary': 'Show your company address on Google Maps',
    'version': '1.0',
    'description': """
Show your company address/partner address on Google Maps. Configure an API key in the Website settings.
    """,
    'depends': ['base_geolocalize', 'website_partner'],
    'data': [
        'views/google_map_templates.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http
from odoo.http import request
from odoo.tools.json import scriptsafe


class GoogleMap(http.Controller):
    '''
    This class generates on-the-fly partner maps that can be reused in every
    website page. To do so, just use an ``<iframe ...>`` whose ``src``
    attribute points to ``/google_map`` (this controller generates a complete
    HTML5 page).

    URL query parameters:
    - ``partner_ids``: a comma-separated list of ids (partners to be shown)
    - ``partner_url``: the base-url to display the partner
        (eg: if ``partner_url`` is ``/partners/``, when the user will click on
        a partner on the map, it will be redirected to <myodoo>.com/partners/<id>)

    In order to resize the map, simply resize the ``iframe`` with CSS
    directives ``width`` and ``height``.
    '''

    @http.route(['/google_map'], type='http', auth="public", website=True, sitemap=False)
    def google_map(self, *arg, **post):
        clean_ids = []
        for partner_id in post.get('partner_ids', "").split(","):
            try:
                clean_ids.append(int(partner_id))
            except ValueError:
                pass
        partners = request.env['res.partner'].sudo().search([("id", "in", clean_ids),
                                                             ('website_published', '=', True), ('is_company', '=', True)])
        partner_data = {
            "counter": len(partners),
            "partners": []
        }
        for partner in partners.with_context(show_address=True):
            partner_data["partners"].append({
                'id': partner.id,
                'name': partner.name,
                'address': '\n'.join(partner.name_get()[0][1].split('\n')[1:]),
                'latitude': str(partner.partner_latitude) if partner.partner_latitude else False,
                'longitude': str(partner.partner_longitude) if partner.partner_longitude else False,
            })
        if 'customers' in post.get('partner_url', ''):
            partner_url = '/customers/'
        else:
            partner_url = '/partners/'

        google_maps_api_key = request.website.google_maps_api_key
        values = {
            'partner_url': partner_url,
            'partner_data': scriptsafe.dumps(partner_data),
            'google_maps_api_key': google_maps_api_key,
        }
        return request.render("website_google_map.google_map", values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M21.133 69H4c-2 0-4-.146-4-4.078v-23.83l22.808-19.397 7.515 1.218L35 19.09l4.05-2.847 10.88 1.975 4.671 13.71-12.054 16.98L21.133 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M42.676 50.887c-.43.304-.876.59-1.34.86C38.753 53.25 35.93 54 32.87 54c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 43.06 16 40.25 16 37.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c2.428-1.411 5.066-2.16 7.915-2.245-.585.74-1.11 1.83-1.576 3.27l-.003.002a.587.587 0 0 0-.12.252.14.14 0 0 0 .017.085c-.092.302-.182.618-.27.948a3.21 3.21 0 0 1-.307-.016l-.22-.022c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.198.124.355.207.473.251.146.059.263.088.35.088.089 0 .195-.019.32-.055a3.67 3.67 0 0 1 .296-.077c.22-.029.432.08.637.329.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a5.832 5.832 0 0 1-.264-.328c-.073.131-.19.19-.351.175-.161-.015-.279-.08-.352-.197a.601.601 0 0 1-.033.12.518.518 0 0 0-.033.142c-.19 0-.3-.007-.33-.021.016-.044.034-.172.056-.383.022-.212.047-.376.077-.492.014-.059.055-.146.12-.263.066-.116.121-.222.165-.317.044-.095.073-.186.088-.273.015-.088-.018-.157-.099-.208-.08-.051-.208-.07-.384-.055-.278.015-.469.16-.571.438a4.52 4.52 0 0 0-.066.23.74.74 0 0 1-.11.25.546.546 0 0 1-.198.154c-.102.044-.278.058-.527.044-.249-.015-.424-.051-.527-.11-.19-.116-.355-.328-.494-.634-.14-.306-.209-.576-.209-.81 0-.145.018-.338.055-.579.037-.24.059-.422.066-.546.005-.083-.035-.262-.12-.536l.623-4.406c1.345 3.236 3.548 6.64 5.82 9.583a.238.238 0 0 0-.03.034 1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35 1.724-.3 3.302-.878 4.734-1.735a39.265 39.265 0 0 0 2.635 2.64zm.189-3.11C33.85 34.856 32.178 33.53 32.178 28.78 32.178 22.274 37.511 17 44.089 17 50.667 17 56 22.274 56 28.78c0 4.749-1.673 6.075-10.687 18.998a1.5 1.5 0 0 1-2.448 0zM43.92 35c3.56 0 6.446-2.892 6.446-6.46s-2.886-6.46-6.446-6.46c-3.56 0-6.447 2.892-6.447 6.46S40.36 35 43.921 35z" opacity=".241"/><path fill="#FFF" d="M42.676 48.887c-.43.304-.876.59-1.34.86C38.753 51.25 35.93 52 32.87 52c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 41.06 16 38.25 16 35.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c2.428-1.411 5.066-2.16 7.915-2.245-.585.74-1.11 1.83-1.576 3.27l-.003.002a.587.587 0 0 0-.12.252.14.14 0 0 0 .017.085c-.092.302-.182.618-.27.948a3.21 3.21 0 0 1-.307-.016l-.22-.022c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.198.124.355.207.473.251.146.059.263.088.35.088.089 0 .195-.019.32-.055a3.67 3.67 0 0 1 .296-.077c.22-.029.432.08.637.329.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a5.832 5.832 0 0 1-.264-.328c-.073.131-.19.19-.351.175-.161-.015-.279-.08-.352-.197a.601.601 0 0 1-.033.12.518.518 0 0 0-.033.142c-.19 0-.3-.007-.33-.021.016-.044.034-.172.056-.383.022-.212.047-.376.077-.492.014-.059.055-.146.12-.263.066-.116.121-.222.165-.317.044-.095.073-.186.088-.273.015-.088-.018-.157-.099-.208-.08-.051-.208-.07-.384-.055-.278.015-.469.16-.571.438a4.52 4.52 0 0 0-.066.23.74.74 0 0 1-.11.25.546.546 0 0 1-.198.154c-.102.044-.278.058-.527.044-.249-.015-.424-.051-.527-.11-.19-.116-.355-.328-.494-.634-.14-.306-.209-.576-.209-.81 0-.145.018-.338.055-.579.037-.24.059-.422.066-.546.005-.083-.035-.262-.12-.536l.623-4.406c1.345 3.236 3.548 6.64 5.82 9.583a.238.238 0 0 0-.03.034 1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35 1.724-.3 3.302-.878 4.734-1.735a39.265 39.265 0 0 0 2.635 2.64zm.189-3.11C33.85 32.856 32.178 31.53 32.178 26.78 32.178 20.274 37.511 15 44.089 15 50.667 15 56 20.274 56 26.78c0 4.749-1.673 6.075-10.687 18.998a1.5 1.5 0 0 1-2.448 0zM43.92 33c3.56 0 6.446-2.892 6.446-6.46s-2.886-6.46-6.446-6.46c-3.56 0-6.447 2.892-6.447 6.46S40.36 33 43.921 33z"/></g></g></svg>
```

## File: static\src\js\website_google_map.js

```javascript
/* global MarkerClusterer, google */
function initialize_map() {
    'use strict';

    // MAP CONFIG AND LOADING
    var map = new google.maps.Map(document.getElementById('odoo-google-map'), {
        zoom: 1,
        center: {lat: 0.0, lng: 0.0},
        mapTypeId: google.maps.MapTypeId.ROADMAP
    });

    // ENABLE ADDRESS GEOCODING
    var Geocoder = new google.maps.Geocoder();

    // INFO BUBBLES
    var infoWindow = new google.maps.InfoWindow();
    var partners = new google.maps.MarkerImage('/website_google_map/static/src/img/partners.png', new google.maps.Size(25, 25));
    var partner_url = document.body.getAttribute('data-partner-url') || '';
    var markers = [];
    var options = {
        imagePath: '/website_google_map/static/src/lib/images/m'
    };

    google.maps.event.addListener(map, 'click', function() {
        infoWindow.close();
    });

    // Display the bubble once clicked
    var onMarkerClick = function() {
        var marker = this;
        var p = marker.partner;
        infoWindow.setContent(
              '<div class="marker">'+
              (partner_url.length ? '<a target="_top" href="'+partner_url+p.id+'"><b>'+p.name +'</b></a>' : '<b>'+p.name+'</b>' )+
              (p.type ? '  <b>' + p.type + '</b>' : '')+
              '  <pre>' + p.address + '</pre>'+
              '</div>'
          );
        infoWindow.open(map, marker);
    };

    // Create a bubble for a partner
    var set_marker = function(partner) {
        // If no lat & long, geocode address
        // TODO: a server cronjob that will store these coordinates in database instead of resolving them on-the-fly
        if (!partner.latitude && !partner.longitude) {
            Geocoder.geocode({'address': partner.address}, function(results, status) {
                if (status === google.maps.GeocoderStatus.OK) {
                    var location = results[0].geometry.location;
                    partner.latitude = location.ob;
                    partner.longitude = location.pb;
                    var marker = new google.maps.Marker({
                        partner: partner,
                        map: map,
                        icon: partners,
                        position: location
                    });
                    google.maps.event.addListener(marker, 'click', onMarkerClick);
                    markers.push(marker);
                } else {
                    console.debug('Geocode was not successful for the following reason: ' + status);
                }
            });
        } else {
            var latLng = new google.maps.LatLng(partner.latitude, partner.longitude);
            var marker = new google.maps.Marker({
                partner: partner,
                icon: partners,
                map: map,
                position: latLng
            });
            google.maps.event.addListener(marker, 'click', onMarkerClick);
            markers.push(marker);
        }
    };

    /* eslint-disable no-undef */
    // Create the markers and cluster them on the map
    if (odoo_partner_data){ /* odoo_partner_data special variable should have been defined in google_map.xml */
        for (var i = 0; i < odoo_partner_data.counter; i++) {
            set_marker(odoo_partner_data.partners[i]);
        }
        new MarkerClusterer(map, markers, options);
    }
    /* eslint-enable no-undef */
}

// Initialize map once the DOM has been loaded
google.maps.event.addDomListener(window, 'load', initialize_map);

```

## File: static\src\lib\markerclusterer.js

```javascript
// ==ClosureCompiler==
// @compilation_level ADVANCED_OPTIMIZATIONS
// @externs_url https://raw.githubusercontent.com/google/closure-compiler/master/contrib/externs/maps/google_maps_api_v3.js
// ==/ClosureCompiler==

/**
 * @name MarkerClusterer for Google Maps v3
 * @version version 1.0
 * @author Luke Mahe
 * @fileoverview
 * The library creates and manages per-zoom-level clusters for large amounts of
 * markers.
 * <br/>
 * This is a v3 implementation of the
 * <a href="http://gmaps-utility-library-dev.googlecode.com/svn/tags/markerclusterer/"
 * >v2 MarkerClusterer</a>.
 */

/**
 * @license
 * Copyright 2010 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */


/**
 * A Marker Clusterer that clusters markers.
 *
 * @param {google.maps.Map} map The Google map to attach to.
 * @param {Array.<google.maps.Marker>=} opt_markers Optional markers to add to
 *   the cluster.
 * @param {Object=} opt_options support the following options:
 *     'gridSize': (number) The grid size of a cluster in pixels.
 *     'maxZoom': (number) The maximum zoom level that a marker can be part of a
 *                cluster.
 *     'zoomOnClick': (boolean) Whether the default behaviour of clicking on a
 *                    cluster is to zoom into it.
 *     'averageCenter': (boolean) Whether the center of each cluster should be
 *                      the average of all markers in the cluster.
 *     'minimumClusterSize': (number) The minimum number of markers to be in a
 *                           cluster before the markers are hidden and a count
 *                           is shown.
 *     'styles': (object) An object that has style properties:
 *       'url': (string) The image url.
 *       'height': (number) The image height.
 *       'width': (number) The image width.
 *       'anchor': (Array) The anchor position of the label text.
 *       'textColor': (string) The text color.
 *       'textSize': (number) The text size.
 *       'backgroundPosition': (string) The position of the backgound x, y.
 *       'iconAnchor': (Array) The anchor position of the icon x, y.
 * @constructor
 * @extends google.maps.OverlayView
 */
function MarkerClusterer(map, opt_markers, opt_options) {
  // MarkerClusterer implements google.maps.OverlayView interface. We use the
  // extend function to extend MarkerClusterer with google.maps.OverlayView
  // because it might not always be available when the code is defined so we
  // look for it at the last possible moment. If it doesn't exist now then
  // there is no point going ahead :)
  this.extend(MarkerClusterer, google.maps.OverlayView);
  this.map_ = map;

  /**
   * @type {Array.<google.maps.Marker>}
   * @private
   */
  this.markers_ = [];

  /**
   *  @type {Array.<Cluster>}
   */
  this.clusters_ = [];

  this.sizes = [53, 56, 66, 78, 90];

  /**
   * @private
   */
  this.styles_ = [];

  /**
   * @type {boolean}
   * @private
   */
  this.ready_ = false;

  var options = opt_options || {};

  /**
   * @type {number}
   * @private
   */
  this.gridSize_ = options['gridSize'] || 60;

  /**
   * @private
   */
  this.minClusterSize_ = options['minimumClusterSize'] || 2;


  /**
   * @type {?number}
   * @private
   */
  this.maxZoom_ = options['maxZoom'] || null;

  this.styles_ = options['styles'] || [];

  /**
   * @type {string}
   * @private
   */
  this.imagePath_ = options['imagePath'] ||
      this.MARKER_CLUSTER_IMAGE_PATH_;

  /**
   * @type {string}
   * @private
   */
  this.imageExtension_ = options['imageExtension'] ||
      this.MARKER_CLUSTER_IMAGE_EXTENSION_;

  /**
   * @type {boolean}
   * @private
   */
  this.zoomOnClick_ = true;

  if (options['zoomOnClick'] != undefined) {
    this.zoomOnClick_ = options['zoomOnClick'];
  }

  /**
   * @type {boolean}
   * @private
   */
  this.averageCenter_ = false;

  if (options['averageCenter'] != undefined) {
    this.averageCenter_ = options['averageCenter'];
  }

  this.setupStyles_();

  this.setMap(map);

  /**
   * @type {number}
   * @private
   */
  this.prevZoom_ = this.map_.getZoom();

  // Add the map event listeners
  var that = this;
  google.maps.event.addListener(this.map_, 'zoom_changed', function() {
    var zoom = that.map_.getZoom();

    if (that.prevZoom_ != zoom) {
      that.prevZoom_ = zoom;
      that.resetViewport();
    }
  });

  google.maps.event.addListener(this.map_, 'idle', function() {
    that.redraw();
  });

  // Finally, add the markers
  if (opt_markers && opt_markers.length) {
    this.addMarkers(opt_markers, false);
  }
}


/**
 * The marker cluster image path.
 *
 * @type {string}
 * @private
 */
MarkerClusterer.prototype.MARKER_CLUSTER_IMAGE_PATH_ = '../images/m';


/**
 * The marker cluster image path.
 *
 * @type {string}
 * @private
 */
MarkerClusterer.prototype.MARKER_CLUSTER_IMAGE_EXTENSION_ = 'png';


/**
 * Extends a objects prototype by anothers.
 *
 * @param {Object} obj1 The object to be extended.
 * @param {Object} obj2 The object to extend with.
 * @return {Object} The new extended object.
 * @ignore
 */
MarkerClusterer.prototype.extend = function(obj1, obj2) {
  return (function(object) {
    for (var property in object.prototype) {
      this.prototype[property] = object.prototype[property];
    }
    return this;
  }).apply(obj1, [obj2]);
};


/**
 * Implementaion of the interface method.
 * @ignore
 */
MarkerClusterer.prototype.onAdd = function() {
  this.setReady_(true);
};

/**
 * Implementaion of the interface method.
 * @ignore
 */
MarkerClusterer.prototype.draw = function() {};

/**
 * Sets up the styles object.
 *
 * @private
 */
MarkerClusterer.prototype.setupStyles_ = function() {
  if (this.styles_.length) {
    return;
  }

  for (var i = 0, size; size = this.sizes[i]; i++) {
    this.styles_.push({
      url: this.imagePath_ + (i + 1) + '.' + this.imageExtension_,
      height: size,
      width: size
    });
  }
};

/**
 *  Fit the map to the bounds of the markers in the clusterer.
 */
MarkerClusterer.prototype.fitMapToMarkers = function() {
  var markers = this.getMarkers();
  var bounds = new google.maps.LatLngBounds();
  for (var i = 0, marker; marker = markers[i]; i++) {
    bounds.extend(marker.getPosition());
  }

  this.map_.fitBounds(bounds);
};


/**
 *  Sets the styles.
 *
 *  @param {Object} styles The style to set.
 */
MarkerClusterer.prototype.setStyles = function(styles) {
  this.styles_ = styles;
};


/**
 *  Gets the styles.
 *
 *  @return {Object} The styles object.
 */
MarkerClusterer.prototype.getStyles = function() {
  return this.styles_;
};


/**
 * Whether zoom on click is set.
 *
 * @return {boolean} True if zoomOnClick_ is set.
 */
MarkerClusterer.prototype.isZoomOnClick = function() {
  return this.zoomOnClick_;
};

/**
 * Whether average center is set.
 *
 * @return {boolean} True if averageCenter_ is set.
 */
MarkerClusterer.prototype.isAverageCenter = function() {
  return this.averageCenter_;
};


/**
 *  Returns the array of markers in the clusterer.
 *
 *  @return {Array.<google.maps.Marker>} The markers.
 */
MarkerClusterer.prototype.getMarkers = function() {
  return this.markers_;
};


/**
 *  Returns the number of markers in the clusterer
 *
 *  @return {Number} The number of markers.
 */
MarkerClusterer.prototype.getTotalMarkers = function() {
  return this.markers_.length;
};


/**
 *  Sets the max zoom for the clusterer.
 *
 *  @param {number} maxZoom The max zoom level.
 */
MarkerClusterer.prototype.setMaxZoom = function(maxZoom) {
  this.maxZoom_ = maxZoom;
};


/**
 *  Gets the max zoom for the clusterer.
 *
 *  @return {number} The max zoom level.
 */
MarkerClusterer.prototype.getMaxZoom = function() {
  return this.maxZoom_;
};


/**
 *  The function for calculating the cluster icon image.
 *
 *  @param {Array.<google.maps.Marker>} markers The markers in the clusterer.
 *  @param {number} numStyles The number of styles available.
 *  @return {Object} A object properties: 'text' (string) and 'index' (number).
 *  @private
 */
MarkerClusterer.prototype.calculator_ = function(markers, numStyles) {
  var index = 0;
  var count = markers.length;
  var dv = count;
  while (dv !== 0) {
    dv = parseInt(dv / 10, 10);
    index++;
  }

  index = Math.min(index, numStyles);
  return {
    text: count,
    index: index
  };
};


/**
 * Set the calculator function.
 *
 * @param {function(Array, number)} calculator The function to set as the
 *     calculator. The function should return a object properties:
 *     'text' (string) and 'index' (number).
 *
 */
MarkerClusterer.prototype.setCalculator = function(calculator) {
  this.calculator_ = calculator;
};


/**
 * Get the calculator function.
 *
 * @return {function(Array, number)} the calculator function.
 */
MarkerClusterer.prototype.getCalculator = function() {
  return this.calculator_;
};


/**
 * Add an array of markers to the clusterer.
 *
 * @param {Array.<google.maps.Marker>} markers The markers to add.
 * @param {boolean=} opt_nodraw Whether to redraw the clusters.
 */
MarkerClusterer.prototype.addMarkers = function(markers, opt_nodraw) {
  for (var i = 0, marker; marker = markers[i]; i++) {
    this.pushMarkerTo_(marker);
  }
  if (!opt_nodraw) {
    this.redraw();
  }
};


/**
 * Pushes a marker to the clusterer.
 *
 * @param {google.maps.Marker} marker The marker to add.
 * @private
 */
MarkerClusterer.prototype.pushMarkerTo_ = function(marker) {
  marker.isAdded = false;
  if (marker['draggable']) {
    // If the marker is draggable add a listener so we update the clusters on
    // the drag end.
    var that = this;
    google.maps.event.addListener(marker, 'dragend', function() {
      marker.isAdded = false;
      that.repaint();
    });
  }
  this.markers_.push(marker);
};


/**
 * Adds a marker to the clusterer and redraws if needed.
 *
 * @param {google.maps.Marker} marker The marker to add.
 * @param {boolean=} opt_nodraw Whether to redraw the clusters.
 */
MarkerClusterer.prototype.addMarker = function(marker, opt_nodraw) {
  this.pushMarkerTo_(marker);
  if (!opt_nodraw) {
    this.redraw();
  }
};


/**
 * Removes a marker and returns true if removed, false if not
 *
 * @param {google.maps.Marker} marker The marker to remove
 * @return {boolean} Whether the marker was removed or not
 * @private
 */
MarkerClusterer.prototype.removeMarker_ = function(marker) {
  var index = -1;
  if (this.markers_.indexOf) {
    index = this.markers_.indexOf(marker);
  } else {
    for (var i = 0, m; m = this.markers_[i]; i++) {
      if (m == marker) {
        index = i;
        break;
      }
    }
  }

  if (index == -1) {
    // Marker is not in our list of markers.
    return false;
  }

  marker.setMap(null);

  this.markers_.splice(index, 1);

  return true;
};


/**
 * Remove a marker from the cluster.
 *
 * @param {google.maps.Marker} marker The marker to remove.
 * @param {boolean=} opt_nodraw Optional boolean to force no redraw.
 * @return {boolean} True if the marker was removed.
 */
MarkerClusterer.prototype.removeMarker = function(marker, opt_nodraw) {
  var removed = this.removeMarker_(marker);

  if (!opt_nodraw && removed) {
    this.resetViewport();
    this.redraw();
    return true;
  } else {
   return false;
  }
};


/**
 * Removes an array of markers from the cluster.
 *
 * @param {Array.<google.maps.Marker>} markers The markers to remove.
 * @param {boolean=} opt_nodraw Optional boolean to force no redraw.
 */
MarkerClusterer.prototype.removeMarkers = function(markers, opt_nodraw) {
  var removed = false;

  for (var i = 0, marker; marker = markers[i]; i++) {
    var r = this.removeMarker_(marker);
    removed = removed || r;
  }

  if (!opt_nodraw && removed) {
    this.resetViewport();
    this.redraw();
    return true;
  }
};


/**
 * Sets the clusterer's ready state.
 *
 * @param {boolean} ready The state.
 * @private
 */
MarkerClusterer.prototype.setReady_ = function(ready) {
  if (!this.ready_) {
    this.ready_ = ready;
    this.createClusters_();
  }
};


/**
 * Returns the number of clusters in the clusterer.
 *
 * @return {number} The number of clusters.
 */
MarkerClusterer.prototype.getTotalClusters = function() {
  return this.clusters_.length;
};


/**
 * Returns the google map that the clusterer is associated with.
 *
 * @return {google.maps.Map} The map.
 */
MarkerClusterer.prototype.getMap = function() {
  return this.map_;
};


/**
 * Sets the google map that the clusterer is associated with.
 *
 * @param {google.maps.Map} map The map.
 */
MarkerClusterer.prototype.setMap = function(map) {
  this.map_ = map;
};


/**
 * Returns the size of the grid.
 *
 * @return {number} The grid size.
 */
MarkerClusterer.prototype.getGridSize = function() {
  return this.gridSize_;
};


/**
 * Sets the size of the grid.
 *
 * @param {number} size The grid size.
 */
MarkerClusterer.prototype.setGridSize = function(size) {
  this.gridSize_ = size;
};


/**
 * Returns the min cluster size.
 *
 * @return {number} The grid size.
 */
MarkerClusterer.prototype.getMinClusterSize = function() {
  return this.minClusterSize_;
};

/**
 * Sets the min cluster size.
 *
 * @param {number} size The grid size.
 */
MarkerClusterer.prototype.setMinClusterSize = function(size) {
  this.minClusterSize_ = size;
};


/**
 * Extends a bounds object by the grid size.
 *
 * @param {google.maps.LatLngBounds} bounds The bounds to extend.
 * @return {google.maps.LatLngBounds} The extended bounds.
 */
MarkerClusterer.prototype.getExtendedBounds = function(bounds) {
  var projection = this.getProjection();

  // Turn the bounds into latlng.
  var tr = new google.maps.LatLng(bounds.getNorthEast().lat(),
      bounds.getNorthEast().lng());
  var bl = new google.maps.LatLng(bounds.getSouthWest().lat(),
      bounds.getSouthWest().lng());

  // Convert the points to pixels and the extend out by the grid size.
  var trPix = projection.fromLatLngToDivPixel(tr);
  trPix.x += this.gridSize_;
  trPix.y -= this.gridSize_;

  var blPix = projection.fromLatLngToDivPixel(bl);
  blPix.x -= this.gridSize_;
  blPix.y += this.gridSize_;

  // Convert the pixel points back to LatLng
  var ne = projection.fromDivPixelToLatLng(trPix);
  var sw = projection.fromDivPixelToLatLng(blPix);

  // Extend the bounds to contain the new bounds.
  bounds.extend(ne);
  bounds.extend(sw);

  return bounds;
};


/**
 * Determins if a marker is contained in a bounds.
 *
 * @param {google.maps.Marker} marker The marker to check.
 * @param {google.maps.LatLngBounds} bounds The bounds to check against.
 * @return {boolean} True if the marker is in the bounds.
 * @private
 */
MarkerClusterer.prototype.isMarkerInBounds_ = function(marker, bounds) {
  return bounds.contains(marker.getPosition());
};


/**
 * Clears all clusters and markers from the clusterer.
 */
MarkerClusterer.prototype.clearMarkers = function() {
  this.resetViewport(true);

  // Set the markers a empty array.
  this.markers_ = [];
};


/**
 * Clears all existing clusters and recreates them.
 * @param {boolean} opt_hide To also hide the marker.
 */
MarkerClusterer.prototype.resetViewport = function(opt_hide) {
  // Remove all the clusters
  for (var i = 0, cluster; cluster = this.clusters_[i]; i++) {
    cluster.remove();
  }

  // Reset the markers to not be added and to be invisible.
  for (var i = 0, marker; marker = this.markers_[i]; i++) {
    marker.isAdded = false;
    if (opt_hide) {
      marker.setMap(null);
    }
  }

  this.clusters_ = [];
};

/**
 *
 */
MarkerClusterer.prototype.repaint = function() {
  var oldClusters = this.clusters_.slice();
  this.clusters_.length = 0;
  this.resetViewport();
  this.redraw();

  // Remove the old clusters.
  // Do it in a timeout so the other clusters have been drawn first.
  window.setTimeout(function() {
    for (var i = 0, cluster; cluster = oldClusters[i]; i++) {
      cluster.remove();
    }
  }, 0);
};


/**
 * Redraws the clusters.
 */
MarkerClusterer.prototype.redraw = function() {
  this.createClusters_();
};


/**
 * Calculates the distance between two latlng locations in km.
 * @see http://www.movable-type.co.uk/scripts/latlong.html
 *
 * @param {google.maps.LatLng} p1 The first lat lng point.
 * @param {google.maps.LatLng} p2 The second lat lng point.
 * @return {number} The distance between the two points in km.
 * @private
*/
MarkerClusterer.prototype.distanceBetweenPoints_ = function(p1, p2) {
  if (!p1 || !p2) {
    return 0;
  }

  var R = 6371; // Radius of the Earth in km
  var dLat = (p2.lat() - p1.lat()) * Math.PI / 180;
  var dLon = (p2.lng() - p1.lng()) * Math.PI / 180;
  var a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(p1.lat() * Math.PI / 180) * Math.cos(p2.lat() * Math.PI / 180) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);
  var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  var d = R * c;
  return d;
};


/**
 * Add a marker to a cluster, or creates a new cluster.
 *
 * @param {google.maps.Marker} marker The marker to add.
 * @private
 */
MarkerClusterer.prototype.addToClosestCluster_ = function(marker) {
  var distance = 40000; // Some large number
  var clusterToAddTo = null;
  var pos = marker.getPosition();
  for (var i = 0, cluster; cluster = this.clusters_[i]; i++) {
    var center = cluster.getCenter();
    if (center) {
      var d = this.distanceBetweenPoints_(center, marker.getPosition());
      if (d < distance) {
        distance = d;
        clusterToAddTo = cluster;
      }
    }
  }

  if (clusterToAddTo && clusterToAddTo.isMarkerInClusterBounds(marker)) {
    clusterToAddTo.addMarker(marker);
  } else {
    var cluster = new Cluster(this);
    cluster.addMarker(marker);
    this.clusters_.push(cluster);
  }
};


/**
 * Creates the clusters.
 *
 * @private
 */
MarkerClusterer.prototype.createClusters_ = function() {
  if (!this.ready_) {
    return;
  }

  // Get our current map view bounds.
  // Create a new bounds object so we don't affect the map.
  var mapBounds = new google.maps.LatLngBounds(this.map_.getBounds().getSouthWest(),
      this.map_.getBounds().getNorthEast());
  var bounds = this.getExtendedBounds(mapBounds);

  for (var i = 0, marker; marker = this.markers_[i]; i++) {
    if (!marker.isAdded && this.isMarkerInBounds_(marker, bounds)) {
      this.addToClosestCluster_(marker);
    }
  }
};


/**
 * A cluster that contains markers.
 *
 * @param {MarkerClusterer} markerClusterer The markerclusterer that this
 *     cluster is associated with.
 * @constructor
 * @ignore
 */
function Cluster(markerClusterer) {
  this.markerClusterer_ = markerClusterer;
  this.map_ = markerClusterer.getMap();
  this.gridSize_ = markerClusterer.getGridSize();
  this.minClusterSize_ = markerClusterer.getMinClusterSize();
  this.averageCenter_ = markerClusterer.isAverageCenter();
  this.center_ = null;
  this.markers_ = [];
  this.bounds_ = null;
  this.clusterIcon_ = new ClusterIcon(this, markerClusterer.getStyles(),
      markerClusterer.getGridSize());
}

/**
 * Determins if a marker is already added to the cluster.
 *
 * @param {google.maps.Marker} marker The marker to check.
 * @return {boolean} True if the marker is already added.
 */
Cluster.prototype.isMarkerAlreadyAdded = function(marker) {
  if (this.markers_.indexOf) {
    return this.markers_.indexOf(marker) != -1;
  } else {
    for (var i = 0, m; m = this.markers_[i]; i++) {
      if (m == marker) {
        return true;
      }
    }
  }
  return false;
};


/**
 * Add a marker the cluster.
 *
 * @param {google.maps.Marker} marker The marker to add.
 * @return {boolean} True if the marker was added.
 */
Cluster.prototype.addMarker = function(marker) {
  if (this.isMarkerAlreadyAdded(marker)) {
    return false;
  }

  if (!this.center_) {
    this.center_ = marker.getPosition();
    this.calculateBounds_();
  } else {
    if (this.averageCenter_) {
      var l = this.markers_.length + 1;
      var lat = (this.center_.lat() * (l-1) + marker.getPosition().lat()) / l;
      var lng = (this.center_.lng() * (l-1) + marker.getPosition().lng()) / l;
      this.center_ = new google.maps.LatLng(lat, lng);
      this.calculateBounds_();
    }
  }

  marker.isAdded = true;
  this.markers_.push(marker);

  var len = this.markers_.length;
  if (len < this.minClusterSize_ && marker.getMap() != this.map_) {
    // Min cluster size not reached so show the marker.
    marker.setMap(this.map_);
  }

  if (len == this.minClusterSize_) {
    // Hide the markers that were showing.
    for (var i = 0; i < len; i++) {
      this.markers_[i].setMap(null);
    }
  }

  if (len >= this.minClusterSize_) {
    marker.setMap(null);
  }

  this.updateIcon();
  return true;
};


/**
 * Returns the marker clusterer that the cluster is associated with.
 *
 * @return {MarkerClusterer} The associated marker clusterer.
 */
Cluster.prototype.getMarkerClusterer = function() {
  return this.markerClusterer_;
};


/**
 * Returns the bounds of the cluster.
 *
 * @return {google.maps.LatLngBounds} the cluster bounds.
 */
Cluster.prototype.getBounds = function() {
  var bounds = new google.maps.LatLngBounds(this.center_, this.center_);
  var markers = this.getMarkers();
  for (var i = 0, marker; marker = markers[i]; i++) {
    bounds.extend(marker.getPosition());
  }
  return bounds;
};


/**
 * Removes the cluster
 */
Cluster.prototype.remove = function() {
  this.clusterIcon_.remove();
  this.markers_.length = 0;
  delete this.markers_;
};


/**
 * Returns the center of the cluster.
 *
 * @return {number} The cluster center.
 */
Cluster.prototype.getSize = function() {
  return this.markers_.length;
};


/**
 * Returns the center of the cluster.
 *
 * @return {Array.<google.maps.Marker>} The cluster center.
 */
Cluster.prototype.getMarkers = function() {
  return this.markers_;
};


/**
 * Returns the center of the cluster.
 *
 * @return {google.maps.LatLng} The cluster center.
 */
Cluster.prototype.getCenter = function() {
  return this.center_;
};


/**
 * Calculated the extended bounds of the cluster with the grid.
 *
 * @private
 */
Cluster.prototype.calculateBounds_ = function() {
  var bounds = new google.maps.LatLngBounds(this.center_, this.center_);
  this.bounds_ = this.markerClusterer_.getExtendedBounds(bounds);
};


/**
 * Determines if a marker lies in the clusters bounds.
 *
 * @param {google.maps.Marker} marker The marker to check.
 * @return {boolean} True if the marker lies in the bounds.
 */
Cluster.prototype.isMarkerInClusterBounds = function(marker) {
  return this.bounds_.contains(marker.getPosition());
};


/**
 * Returns the map that the cluster is associated with.
 *
 * @return {google.maps.Map} The map.
 */
Cluster.prototype.getMap = function() {
  return this.map_;
};


/**
 * Updates the cluster icon
 */
Cluster.prototype.updateIcon = function() {
  var zoom = this.map_.getZoom();
  var mz = this.markerClusterer_.getMaxZoom();

  if (mz && zoom > mz) {
    // The zoom is greater than our max zoom so show all the markers in cluster.
    for (var i = 0, marker; marker = this.markers_[i]; i++) {
      marker.setMap(this.map_);
    }
    return;
  }

  if (this.markers_.length < this.minClusterSize_) {
    // Min cluster size not yet reached.
    this.clusterIcon_.hide();
    return;
  }

  var numStyles = this.markerClusterer_.getStyles().length;
  var sums = this.markerClusterer_.getCalculator()(this.markers_, numStyles);
  this.clusterIcon_.setCenter(this.center_);
  this.clusterIcon_.setSums(sums);
  this.clusterIcon_.show();
};


/**
 * A cluster icon
 *
 * @param {Cluster} cluster The cluster to be associated with.
 * @param {Object} styles An object that has style properties:
 *     'url': (string) The image url.
 *     'height': (number) The image height.
 *     'width': (number) The image width.
 *     'anchor': (Array) The anchor position of the label text.
 *     'textColor': (string) The text color.
 *     'textSize': (number) The text size.
 *     'backgroundPosition: (string) The background postition x, y.
 * @param {number=} opt_padding Optional padding to apply to the cluster icon.
 * @constructor
 * @extends google.maps.OverlayView
 * @ignore
 */
function ClusterIcon(cluster, styles, opt_padding) {
  cluster.getMarkerClusterer().extend(ClusterIcon, google.maps.OverlayView);

  this.styles_ = styles;
  this.padding_ = opt_padding || 0;
  this.cluster_ = cluster;
  this.center_ = null;
  this.map_ = cluster.getMap();
  this.div_ = null;
  this.sums_ = null;
  this.visible_ = false;

  this.setMap(this.map_);
}


/**
 * Triggers the clusterclick event and zoom's if the option is set.
 *
 * @param {google.maps.MouseEvent} event The event to propagate
 */
ClusterIcon.prototype.triggerClusterClick = function(event) {
  var markerClusterer = this.cluster_.getMarkerClusterer();

  // Trigger the clusterclick event.
  google.maps.event.trigger(markerClusterer, 'clusterclick', this.cluster_, event);

  if (markerClusterer.isZoomOnClick()) {
    // Zoom into the cluster.
    this.map_.fitBounds(this.cluster_.getBounds());
  }
};


/**
 * Adding the cluster icon to the dom.
 * @ignore
 */
ClusterIcon.prototype.onAdd = function() {
  this.div_ = document.createElement('DIV');
  if (this.visible_) {
    var pos = this.getPosFromLatLng_(this.center_);
    this.div_.style.cssText = this.createCss(pos);
    this.div_.innerHTML = this.sums_.text;
  }

  var panes = this.getPanes();
  panes.overlayMouseTarget.appendChild(this.div_);

  var that = this;
  var isDragging = false;
  google.maps.event.addDomListener(this.div_, 'click', function(event) {
    // Only perform click when not preceded by a drag
    if (!isDragging) {
      that.triggerClusterClick(event);
    }
  });
  google.maps.event.addDomListener(this.div_, 'mousedown', function() {
    isDragging = false;
  });
  google.maps.event.addDomListener(this.div_, 'mousemove', function() {
    isDragging = true;
  });
};


/**
 * Returns the position to place the div dending on the latlng.
 *
 * @param {google.maps.LatLng} latlng The position in latlng.
 * @return {google.maps.Point} The position in pixels.
 * @private
 */
ClusterIcon.prototype.getPosFromLatLng_ = function(latlng) {
  var pos = this.getProjection().fromLatLngToDivPixel(latlng);

  if (typeof this.iconAnchor_ === 'object' && this.iconAnchor_.length === 2) {
    pos.x -= this.iconAnchor_[0];
    pos.y -= this.iconAnchor_[1];
  } else {
    pos.x -= parseInt(this.width_ / 2, 10);
    pos.y -= parseInt(this.height_ / 2, 10);
  }
  return pos;
};


/**
 * Draw the icon.
 * @ignore
 */
ClusterIcon.prototype.draw = function() {
  if (this.visible_) {
    var pos = this.getPosFromLatLng_(this.center_);
    this.div_.style.top = pos.y + 'px';
    this.div_.style.left = pos.x + 'px';
  }
};


/**
 * Hide the icon.
 */
ClusterIcon.prototype.hide = function() {
  if (this.div_) {
    this.div_.style.display = 'none';
  }
  this.visible_ = false;
};


/**
 * Position and show the icon.
 */
ClusterIcon.prototype.show = function() {
  if (this.div_) {
    var pos = this.getPosFromLatLng_(this.center_);
    this.div_.style.cssText = this.createCss(pos);
    this.div_.style.display = '';
  }
  this.visible_ = true;
};


/**
 * Remove the icon from the map
 */
ClusterIcon.prototype.remove = function() {
  this.setMap(null);
};


/**
 * Implementation of the onRemove interface.
 * @ignore
 */
ClusterIcon.prototype.onRemove = function() {
  if (this.div_ && this.div_.parentNode) {
    this.hide();
    this.div_.parentNode.removeChild(this.div_);
    this.div_ = null;
  }
};


/**
 * Set the sums of the icon.
 *
 * @param {Object} sums The sums containing:
 *   'text': (string) The text to display in the icon.
 *   'index': (number) The style index of the icon.
 */
ClusterIcon.prototype.setSums = function(sums) {
  this.sums_ = sums;
  this.text_ = sums.text;
  this.index_ = sums.index;
  if (this.div_) {
    this.div_.innerHTML = sums.text;
  }

  this.useStyle();
};


/**
 * Sets the icon to the styles.
 */
ClusterIcon.prototype.useStyle = function() {
  var index = Math.max(0, this.sums_.index - 1);
  index = Math.min(this.styles_.length - 1, index);
  var style = this.styles_[index];
  this.url_ = style['url'];
  this.height_ = style['height'];
  this.width_ = style['width'];
  this.textColor_ = style['textColor'];
  this.anchor_ = style['anchor'];
  this.textSize_ = style['textSize'];
  this.backgroundPosition_ = style['backgroundPosition'];
  this.iconAnchor_ = style['iconAnchor'];
};


/**
 * Sets the center of the icon.
 *
 * @param {google.maps.LatLng} center The latlng to set as the center.
 */
ClusterIcon.prototype.setCenter = function(center) {
  this.center_ = center;
};


/**
 * Create the css text based on the position of the icon.
 *
 * @param {google.maps.Point} pos The position.
 * @return {string} The css style text.
 */
ClusterIcon.prototype.createCss = function(pos) {
  var style = [];
  style.push('background-image:url(' + this.url_ + ');');
  var backgroundPosition = this.backgroundPosition_ ? this.backgroundPosition_ : '0 0';
  style.push('background-position:' + backgroundPosition + ';');

  if (typeof this.anchor_ === 'object') {
    if (typeof this.anchor_[0] === 'number' && this.anchor_[0] > 0 &&
        this.anchor_[0] < this.height_) {
      style.push('height:' + (this.height_ - this.anchor_[0]) +
          'px; padding-top:' + this.anchor_[0] + 'px;');
    } else if (typeof this.anchor_[0] === 'number' && this.anchor_[0] < 0 &&
        -this.anchor_[0] < this.height_) {
      style.push('height:' + this.height_ + 'px; line-height:' + (this.height_ + this.anchor_[0]) +
          'px;');
    } else {
      style.push('height:' + this.height_ + 'px; line-height:' + this.height_ +
          'px;');
    }
    if (typeof this.anchor_[1] === 'number' && this.anchor_[1] > 0 &&
        this.anchor_[1] < this.width_) {
      style.push('width:' + (this.width_ - this.anchor_[1]) +
          'px; padding-left:' + this.anchor_[1] + 'px;');
    } else {
      style.push('width:' + this.width_ + 'px; text-align:center;');
    }
  } else {
    style.push('height:' + this.height_ + 'px; line-height:' +
        this.height_ + 'px; width:' + this.width_ + 'px; text-align:center;');
  }

  var txtColor = this.textColor_ ? this.textColor_ : 'black';
  var txtSize = this.textSize_ ? this.textSize_ : 11;

  style.push('cursor:pointer; top:' + pos.y + 'px; left:' +
      pos.x + 'px; color:' + txtColor + '; position:absolute; font-size:' +
      txtSize + 'px; font-family:Arial,sans-serif; font-weight:bold');
  return style.join('');
};


// Export Symbols for Closure
// If you are not going to compile with closure then you can remove the
// code below.
window['MarkerClusterer'] = MarkerClusterer;
MarkerClusterer.prototype['addMarker'] = MarkerClusterer.prototype.addMarker;
MarkerClusterer.prototype['addMarkers'] = MarkerClusterer.prototype.addMarkers;
MarkerClusterer.prototype['clearMarkers'] =
    MarkerClusterer.prototype.clearMarkers;
MarkerClusterer.prototype['fitMapToMarkers'] =
    MarkerClusterer.prototype.fitMapToMarkers;
MarkerClusterer.prototype['getCalculator'] =
    MarkerClusterer.prototype.getCalculator;
MarkerClusterer.prototype['getGridSize'] =
    MarkerClusterer.prototype.getGridSize;
MarkerClusterer.prototype['getExtendedBounds'] =
    MarkerClusterer.prototype.getExtendedBounds;
MarkerClusterer.prototype['getMap'] = MarkerClusterer.prototype.getMap;
MarkerClusterer.prototype['getMarkers'] = MarkerClusterer.prototype.getMarkers;
MarkerClusterer.prototype['getMaxZoom'] = MarkerClusterer.prototype.getMaxZoom;
MarkerClusterer.prototype['getStyles'] = MarkerClusterer.prototype.getStyles;
MarkerClusterer.prototype['getTotalClusters'] =
    MarkerClusterer.prototype.getTotalClusters;
MarkerClusterer.prototype['getTotalMarkers'] =
    MarkerClusterer.prototype.getTotalMarkers;
MarkerClusterer.prototype['redraw'] = MarkerClusterer.prototype.redraw;
MarkerClusterer.prototype['removeMarker'] =
    MarkerClusterer.prototype.removeMarker;
MarkerClusterer.prototype['removeMarkers'] =
    MarkerClusterer.prototype.removeMarkers;
MarkerClusterer.prototype['resetViewport'] =
    MarkerClusterer.prototype.resetViewport;
MarkerClusterer.prototype['repaint'] =
    MarkerClusterer.prototype.repaint;
MarkerClusterer.prototype['setCalculator'] =
    MarkerClusterer.prototype.setCalculator;
MarkerClusterer.prototype['setGridSize'] =
    MarkerClusterer.prototype.setGridSize;
MarkerClusterer.prototype['setMaxZoom'] =
    MarkerClusterer.prototype.setMaxZoom;
MarkerClusterer.prototype['onAdd'] = MarkerClusterer.prototype.onAdd;
MarkerClusterer.prototype['draw'] = MarkerClusterer.prototype.draw;

Cluster.prototype['getCenter'] = Cluster.prototype.getCenter;
Cluster.prototype['getSize'] = Cluster.prototype.getSize;
Cluster.prototype['getMarkers'] = Cluster.prototype.getMarkers;

ClusterIcon.prototype['onAdd'] = ClusterIcon.prototype.onAdd;
ClusterIcon.prototype['draw'] = ClusterIcon.prototype.draw;
ClusterIcon.prototype['onRemove'] = ClusterIcon.prototype.onRemove;

```

## File: views\google_map_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="google_map">
&lt;!DOCTYPE html&gt;
<html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
        <title>World Map</title>
        <link rel="stylesheet" type="text/css" href="/website_google_map/static/src/css/website_google_map.css"/>
    </head>
    <body t-att-data-partner-url="partner_url or None">
        <script>
            var odoo_partner_data = <t t-out="partner_data"/>;
        </script>
        <div id="odoo-google-map"></div>
        <t t-if="google_maps_api_key">
            <script t-attf-src="//maps.google.com/maps/api/js?key=#{google_maps_api_key}"></script>
        </t>
        <t t-else="1">
            <script src="//maps.google.com/maps/api/js"></script>
        </t>
        <script type="text/javascript" src="/website_google_map/static/src/lib/markerclusterer.js"></script>
        <script type="text/javascript" src="/website_google_map/static/src/js/website_google_map.js"></script>
    </body>
</html>
</template>
</odoo>

```

