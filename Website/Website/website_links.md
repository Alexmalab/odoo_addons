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
    'qweb': ['static/src/xml/*.xml'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controller\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from odoo import http
from odoo.http import request


class WebsiteUrl(http.Controller):
    @http.route('/website_links/new', type='json', auth='user', methods=['POST'])
    def create_shorten_url(self, **post):
        if 'url' not in post or post['url'] == '':
            return {'error': 'empty_url'}
        return request.env['link.tracker'].create(post).read()

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
            return werkzeug.utils.redirect('/', 301)

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
        for tracker in self:
            base_url = self.env['website'].get_current_website().get_base_url()
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><g><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/></g><path fill="#393939" d="M4 69c-2 0-4-.146-4-4.074l-.374-15.924 20.62-23.304 3.816-3.623.958 1.178 4.235-5.2 5.776-3.22 8.058 7.495-.939 5.116 4.72-.59 8.557 9.214L32.62 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M47.855 45.665l-.068.117a16.426 16.426 0 0 1-6.005 6.005C39.254 53.262 36.493 54 33.5 54c-2.993 0-5.754-.738-8.282-2.213a16.426 16.426 0 0 1-6.005-6.005C17.738 43.254 17 40.493 17 37.5c0-2.993.738-5.754 2.213-8.282a16.426 16.426 0 0 1 6.005-6.005c.16-.093.32-.183.481-.27l.776 2.75a13.543 13.543 0 0 0-4.62 4.524c.1.1.187.157.258.172.058.014.093.078.108.193a.823.823 0 0 0 .053.236c.022.043.104.022.247-.064.13.114.15.25.065.408.014-.014.33.18.945.58.272.244.423.394.451.451.043.158-.028.287-.214.387a1.099 1.099 0 0 0-.194-.193c-.114-.1-.179-.13-.193-.086-.043.071-.04.204.01.397.05.194.126.283.226.269-.1 0-.168.114-.204.344-.036.229-.054.483-.054.762 0 .28-.007.448-.021.505l.043.022c-.043.171-.004.419.118.74.122.323.276.463.462.42-.186.043-.043.35.43.924.086.114.143.179.172.193.042.029.128.082.257.161.13.079.237.15.323.215.086.064.157.14.214.226.058.071.13.232.215.483.086.25.186.419.301.505-.029.086.04.229.204.43.165.2.24.365.226.494a.104.104 0 0 0-.054.021.104.104 0 0 1-.054.022c.043.1.154.2.333.3.18.1.29.194.333.28.015.043.029.114.043.215a.79.79 0 0 0 .065.236c.028.057.086.071.172.043.028-.287-.144-.73-.516-1.332a17.652 17.652 0 0 1-.365-.623 1.235 1.235 0 0 1-.118-.333 1.638 1.638 0 0 0-.097-.312c.029 0 .072.01.129.032.057.022.118.047.182.076.065.028.119.057.162.086.043.028.057.05.043.064-.043.1-.03.226.043.376.071.15.157.283.257.397a29.466 29.466 0 0 0 .623.688c.086.086.187.226.301.419.115.193.115.29 0 .29.13 0 .272.072.43.215.157.143.28.286.365.43.072.114.129.3.172.558.043.258.079.43.107.516.029.1.09.197.183.29.093.093.183.16.269.204l.343.172.28.15c.071.029.204.104.397.226.193.121.347.204.462.247a.99.99 0 0 0 .344.086c.086 0 .19-.018.311-.054.122-.036.219-.06.29-.075.215-.029.423.079.623.322.2.244.351.394.452.451.515.272.909.351 1.181.237-.029.014-.025.068.01.16.037.094.094.205.173.334a21.063 21.063 0 0 0 .311.494c.072.086.2.193.387.322.186.129.315.236.387.322.086-.057.136-.121.15-.193-.043.115.007.258.15.43.144.172.273.243.387.215.2-.043.301-.273.301-.688-.444.215-.795.086-1.053-.387a.419.419 0 0 0-.053-.118 1.369 1.369 0 0 1-.14-.365.305.305 0 0 1 0-.161c.014-.043.05-.065.107-.065.13 0 .2-.025.215-.075.014-.05 0-.14-.043-.268a4.332 4.332 0 0 1-.086-.28c-.014-.114-.093-.258-.236-.43a5.725 5.725 0 0 1-.258-.322c-.071.13-.186.187-.344.172-.157-.014-.272-.079-.343-.193 0 .014-.011.054-.033.118a.511.511 0 0 0-.032.14c-.186 0-.293-.008-.322-.022a2.66 2.66 0 0 0 .054-.376c.021-.207.046-.369.075-.483.014-.057.054-.143.118-.258.064-.115.118-.218.161-.312.043-.093.072-.182.086-.268.014-.086-.018-.154-.097-.204-.078-.05-.204-.068-.376-.054-.272.014-.458.158-.558.43a4.456 4.456 0 0 0-.065.225.728.728 0 0 1-.107.247.535.535 0 0 1-.193.15c-.1.044-.273.058-.516.044-.244-.015-.415-.05-.516-.108-.186-.114-.347-.322-.483-.623-.136-.3-.204-.565-.204-.795 0-.143.018-.333.054-.569.035-.236.057-.415.064-.537.007-.122-.032-.297-.118-.526a1.09 1.09 0 0 0 .193-.205c.086-.107.158-.182.215-.225a.383.383 0 0 1 .097-.032.22.22 0 0 1 .096 0c.029.007.058-.004.086-.033a.272.272 0 0 0 .065-.128.64.64 0 0 0-.086-.065c-.043-.043-.072-.064-.086-.064.1.043.304.032.612-.033.308-.064.505-.053.591.033.215.157.372.143.473-.043a1.71 1.71 0 0 0-.054-.204c-.036-.122-.04-.219-.01-.29.07.386.279.45.622.193.043.043.154.079.333.107.18.029.305.065.376.108.043.028.093.068.15.118.058.05.097.082.119.097.021.014.057.01.107-.011a.648.648 0 0 0 .183-.14c.143.2.229.373.258.516.157.573.293.888.408.945.1.043.179.057.236.043.057-.014.09-.082.097-.204a2.084 2.084 0 0 0 0-.3 5.339 5.339 0 0 0-.032-.27l-.022-.171v-.387l-.021-.172c-.215-.043-.348-.129-.398-.258a.461.461 0 0 1 .032-.397c.072-.136.18-.269.323-.398a.701.701 0 0 1 .172-.075c.1-.036.21-.082.333-.14a.966.966 0 0 0 .268-.171c.3-.272.408-.523.322-.752.1 0 .18-.065.237-.194-.015 0-.05-.021-.108-.064a1.57 1.57 0 0 0-.16-.107.292.292 0 0 0-.097-.043c.08-.045.116-.107.107-.186 4.292 5.112 6.94 8.087 7.945 8.922.644.536 1.599.84 2.865.911zm-11.97 5.37c2.95-.515 5.464-1.869 7.54-4.06-.042-.043-.132-.076-.268-.097a.772.772 0 0 1-.268-.075 3.38 3.38 0 0 0-.516-.172.42.42 0 0 0-.054-.28.573.573 0 0 0-.172-.193 11.49 11.49 0 0 0-.268-.172 11.31 11.31 0 0 1-.236-.15 2.026 2.026 0 0 0-.15-.129 1.525 1.525 0 0 0-.312-.215c-.08-.043-.14-.057-.183-.043a.754.754 0 0 1-.215.022l-.064.021a.594.594 0 0 0-.118.054 1.76 1.76 0 0 1-.119.064.221.221 0 0 0-.086.065c-.014.021-.014.04 0 .054-.3-.244-.558-.402-.773-.473a.607.607 0 0 1-.236-.118 2.04 2.04 0 0 0-.226-.15.314.314 0 0 0-.215-.033.49.49 0 0 0-.247.15.514.514 0 0 0-.129.323 1.629 1.629 0 0 1-.043.279c-.1-.072-.1-.197 0-.376s.115-.311.043-.397c-.043-.086-.118-.119-.225-.097a.719.719 0 0 0-.258.097 4.128 4.128 0 0 0-.247.182c-.1.079-.165.126-.194.14-.028.014-.089.054-.182.118a.723.723 0 0 0-.183.161 1.067 1.067 0 0 0-.129.258 1.3 1.3 0 0 1-.107.236c-.029-.057-.111-.104-.247-.14-.136-.035-.204-.075-.204-.118.028.144.057.394.086.752.028.359.064.63.107.817.1.444.014.788-.258 1.031-.386.358-.594.645-.623.86-.057.315.029.5.258.558 0 .1-.057.247-.172.44-.114.194-.165.348-.15.462 0 .086.014.2.043.344zm16.851-15.059c0-.455-.158-.842-.474-1.16l-3.525-3.553a1.563 1.563 0 0 0-1.153-.478c-.474 0-.88.182-1.22.546.034.034.141.14.322.316.18.177.302.299.365.367.062.069.146.177.254.325.107.148.18.293.22.435.04.143.06.3.06.47 0 .455-.159.842-.475 1.161-.316.319-.7.478-1.153.478-.169 0-.324-.02-.466-.06a1.359 1.359 0 0 1-.432-.221 4.04 4.04 0 0 1-.322-.257c-.067-.062-.189-.185-.364-.367a46.78 46.78 0 0 0-.313-.324c-.373.353-.56.768-.56 1.246 0 .456.158.843.475 1.162l3.49 3.535c.306.307.69.46 1.153.46.452 0 .836-.147 1.152-.443l2.492-2.493c.316-.32.474-.7.474-1.145zm-11.913-12.04c0-.455-.158-.842-.475-1.16l-3.49-3.536a1.563 1.563 0 0 0-1.153-.478c-.44 0-.825.154-1.152.461l-2.491 2.494c-.317.318-.475.7-.475 1.144 0 .455.158.842.475 1.161l3.524 3.552c.305.308.69.461 1.153.461.474 0 .88-.176 1.22-.53l-.322-.315c-.18-.177-.302-.299-.364-.367a4.061 4.061 0 0 1-.255-.325 1.378 1.378 0 0 1-.22-.435 1.75 1.75 0 0 1-.06-.47c0-.455.159-.842.475-1.161.317-.319.7-.478 1.153-.478.169 0 .324.02.466.06.14.04.285.113.432.222.147.108.254.193.322.256.067.062.189.185.364.367l.314.324c.372-.353.559-.768.559-1.246zm15.167 12.04c0 1.367-.48 2.522-1.44 3.467l-2.492 2.493c-.938.945-2.084 1.418-3.44 1.418-1.367 0-2.52-.484-3.457-1.452l-3.49-3.535c-.939-.945-1.407-2.1-1.407-3.467 0-1.4.497-2.59 1.49-3.569l-1.49-1.503c-.972 1.002-2.147 1.503-3.525 1.503-1.356 0-2.508-.478-3.457-1.434l-3.525-3.552c-.95-.957-1.424-2.118-1.424-3.484s.48-2.522 1.44-3.467l2.492-2.493c.938-.945 2.084-1.418 3.44-1.418 1.367 0 2.52.484 3.457 1.452l3.491 3.535c.938.945 1.407 2.1 1.407 3.467 0 1.4-.498 2.59-1.492 3.569l1.492 1.503c.971-1.002 2.146-1.503 3.524-1.503 1.356 0 2.508.478 3.457 1.434l3.525 3.553c.95.956 1.424 2.117 1.424 3.483z" opacity=".3"/><path fill="#FFF" d="M47.855 43.665l-.068.117a16.426 16.426 0 0 1-6.005 6.005C39.254 51.262 36.493 52 33.5 52c-2.993 0-5.754-.738-8.282-2.213a16.426 16.426 0 0 1-6.005-6.005C17.738 41.254 17 38.493 17 35.5c0-2.993.738-5.754 2.213-8.282a16.426 16.426 0 0 1 6.005-6.005c.16-.093.32-.183.481-.27l.776 2.75a13.543 13.543 0 0 0-4.62 4.524c.1.1.187.157.258.172.058.014.093.078.108.193a.823.823 0 0 0 .053.236c.022.043.104.022.247-.064.13.114.15.25.065.408.014-.014.33.18.945.58.272.244.423.394.451.451.043.158-.028.287-.214.387a1.099 1.099 0 0 0-.194-.193c-.114-.1-.179-.13-.193-.086-.043.071-.04.204.01.397.05.194.126.283.226.269-.1 0-.168.114-.204.344-.036.229-.054.483-.054.762 0 .28-.007.448-.021.505l.043.022c-.043.171-.004.419.118.74.122.323.276.463.462.42-.186.043-.043.35.43.924.086.114.143.179.172.193.042.029.128.082.257.161.13.079.237.15.323.215.086.064.157.14.214.226.058.071.13.232.215.483.086.25.186.419.301.505-.029.086.04.229.204.43.165.2.24.365.226.494a.104.104 0 0 0-.054.021.104.104 0 0 1-.054.022c.043.1.154.2.333.3.18.1.29.194.333.28.015.043.029.114.043.215a.79.79 0 0 0 .065.236c.028.057.086.071.172.043.028-.287-.144-.73-.516-1.332a17.652 17.652 0 0 1-.365-.623 1.235 1.235 0 0 1-.118-.333 1.638 1.638 0 0 0-.097-.312c.029 0 .072.01.129.032.057.022.118.047.182.076.065.028.119.057.162.086.043.028.057.05.043.064-.043.1-.03.226.043.376.071.15.157.283.257.397a29.466 29.466 0 0 0 .623.688c.086.086.187.226.301.419.115.193.115.29 0 .29.13 0 .272.072.43.215.157.143.28.286.365.43.072.114.129.3.172.558.043.258.079.43.107.516.029.1.09.197.183.29.093.093.183.16.269.204l.343.172.28.15c.071.029.204.104.397.226.193.121.347.204.462.247a.99.99 0 0 0 .344.086c.086 0 .19-.018.311-.054.122-.036.219-.06.29-.075.215-.029.423.079.623.322.2.244.351.394.452.451.515.272.909.351 1.181.237-.029.014-.025.068.01.16.037.094.094.205.173.334a21.063 21.063 0 0 0 .311.494c.072.086.2.193.387.322.186.129.315.236.387.322.086-.057.136-.121.15-.193-.043.115.007.258.15.43.144.172.273.243.387.215.2-.043.301-.273.301-.688-.444.215-.795.086-1.053-.387a.419.419 0 0 0-.053-.118 1.369 1.369 0 0 1-.14-.365.305.305 0 0 1 0-.161c.014-.043.05-.065.107-.065.13 0 .2-.025.215-.075.014-.05 0-.14-.043-.268a4.332 4.332 0 0 1-.086-.28c-.014-.114-.093-.258-.236-.43a5.725 5.725 0 0 1-.258-.322c-.071.13-.186.187-.344.172-.157-.014-.272-.079-.343-.193 0 .014-.011.054-.033.118a.511.511 0 0 0-.032.14c-.186 0-.293-.008-.322-.022a2.66 2.66 0 0 0 .054-.376c.021-.207.046-.369.075-.483.014-.057.054-.143.118-.258.064-.115.118-.218.161-.312.043-.093.072-.182.086-.268.014-.086-.018-.154-.097-.204-.078-.05-.204-.068-.376-.054-.272.014-.458.158-.558.43a4.456 4.456 0 0 0-.065.225.728.728 0 0 1-.107.247.535.535 0 0 1-.193.15c-.1.044-.273.058-.516.044-.244-.015-.415-.05-.516-.108-.186-.114-.347-.322-.483-.623-.136-.3-.204-.565-.204-.795 0-.143.018-.333.054-.569.035-.236.057-.415.064-.537.007-.122-.032-.297-.118-.526a1.09 1.09 0 0 0 .193-.205c.086-.107.158-.182.215-.225a.383.383 0 0 1 .097-.032.22.22 0 0 1 .096 0c.029.007.058-.004.086-.033a.272.272 0 0 0 .065-.128.64.64 0 0 0-.086-.065c-.043-.043-.072-.064-.086-.064.1.043.304.032.612-.033.308-.064.505-.053.591.033.215.157.372.143.473-.043a1.71 1.71 0 0 0-.054-.204c-.036-.122-.04-.219-.01-.29.07.386.279.45.622.193.043.043.154.079.333.107.18.029.305.065.376.108.043.028.093.068.15.118.058.05.097.082.119.097.021.014.057.01.107-.011a.648.648 0 0 0 .183-.14c.143.2.229.373.258.516.157.573.293.888.408.945.1.043.179.057.236.043.057-.014.09-.082.097-.204a2.084 2.084 0 0 0 0-.3 5.339 5.339 0 0 0-.032-.27l-.022-.171v-.387l-.021-.172c-.215-.043-.348-.129-.398-.258a.461.461 0 0 1 .032-.397c.072-.136.18-.269.323-.398a.701.701 0 0 1 .172-.075c.1-.036.21-.082.333-.14a.966.966 0 0 0 .268-.171c.3-.272.408-.523.322-.752.1 0 .18-.065.237-.194-.015 0-.05-.021-.108-.064a1.57 1.57 0 0 0-.16-.107.292.292 0 0 0-.097-.043c.08-.045.116-.107.107-.186 4.292 5.112 6.94 8.087 7.945 8.922.644.536 1.599.84 2.865.911zm-11.97 5.37c2.95-.515 5.464-1.869 7.54-4.06-.042-.043-.132-.076-.268-.097a.772.772 0 0 1-.268-.075 3.38 3.38 0 0 0-.516-.172.42.42 0 0 0-.054-.28.573.573 0 0 0-.172-.193 11.49 11.49 0 0 0-.268-.172 11.31 11.31 0 0 1-.236-.15 2.026 2.026 0 0 0-.15-.129 1.525 1.525 0 0 0-.312-.215c-.08-.043-.14-.057-.183-.043a.754.754 0 0 1-.215.022l-.064.021a.594.594 0 0 0-.118.054 1.76 1.76 0 0 1-.119.064.221.221 0 0 0-.086.065c-.014.021-.014.04 0 .054-.3-.244-.558-.402-.773-.473a.607.607 0 0 1-.236-.118 2.04 2.04 0 0 0-.226-.15.314.314 0 0 0-.215-.033.49.49 0 0 0-.247.15.514.514 0 0 0-.129.323 1.629 1.629 0 0 1-.043.279c-.1-.072-.1-.197 0-.376s.115-.311.043-.397c-.043-.086-.118-.119-.225-.097a.719.719 0 0 0-.258.097 4.128 4.128 0 0 0-.247.182c-.1.079-.165.126-.194.14-.028.014-.089.054-.182.118a.723.723 0 0 0-.183.161 1.067 1.067 0 0 0-.129.258 1.3 1.3 0 0 1-.107.236c-.029-.057-.111-.104-.247-.14-.136-.035-.204-.075-.204-.118.028.144.057.394.086.752.028.359.064.63.107.817.1.444.014.788-.258 1.031-.386.358-.594.645-.623.86-.057.315.029.5.258.558 0 .1-.057.247-.172.44-.114.194-.165.348-.15.462 0 .086.014.2.043.344zm16.851-15.059c0-.455-.158-.842-.474-1.16l-3.525-3.553a1.563 1.563 0 0 0-1.153-.478c-.474 0-.88.182-1.22.546.034.034.141.14.322.316.18.177.302.299.365.367.062.069.146.177.254.325.107.148.18.293.22.435.04.143.06.3.06.47 0 .455-.159.842-.475 1.161-.316.319-.7.478-1.153.478-.169 0-.324-.02-.466-.06a1.359 1.359 0 0 1-.432-.221 4.04 4.04 0 0 1-.322-.257c-.067-.062-.189-.185-.364-.367a46.78 46.78 0 0 0-.313-.324c-.373.353-.56.768-.56 1.246 0 .456.158.843.475 1.162l3.49 3.535c.306.307.69.46 1.153.46.452 0 .836-.147 1.152-.443l2.492-2.493c.316-.32.474-.7.474-1.145zm-11.913-12.04c0-.455-.158-.842-.475-1.16l-3.49-3.536a1.563 1.563 0 0 0-1.153-.478c-.44 0-.825.154-1.152.461l-2.491 2.494c-.317.318-.475.7-.475 1.144 0 .455.158.842.475 1.161l3.524 3.552c.305.308.69.461 1.153.461.474 0 .88-.176 1.22-.53l-.322-.315c-.18-.177-.302-.299-.364-.367a4.061 4.061 0 0 1-.255-.325 1.378 1.378 0 0 1-.22-.435 1.75 1.75 0 0 1-.06-.47c0-.455.159-.842.475-1.161.317-.319.7-.478 1.153-.478.169 0 .324.02.466.06.14.04.285.113.432.222.147.108.254.193.322.256.067.062.189.185.364.367l.314.324c.372-.353.559-.768.559-1.246zm15.167 12.04c0 1.367-.48 2.522-1.44 3.467l-2.492 2.493c-.938.945-2.084 1.418-3.44 1.418-1.367 0-2.52-.484-3.457-1.452l-3.49-3.535c-.939-.945-1.407-2.1-1.407-3.467 0-1.4.497-2.59 1.49-3.569l-1.49-1.503c-.972 1.002-2.147 1.503-3.525 1.503-1.356 0-2.508-.478-3.457-1.434l-3.525-3.552c-.95-.957-1.424-2.118-1.424-3.484s.48-2.522 1.44-3.467l2.492-2.493c.938-.945 2.084-1.418 3.44-1.418 1.367 0 2.52.484 3.457 1.452l3.491 3.535c.938.945 1.407 2.1 1.407 3.467 0 1.4-.498 2.59-1.492 3.569l1.492 1.503c.971-1.002 2.146-1.503 3.524-1.503 1.356 0 2.508.478 3.457 1.434l3.525 3.553c.95.956 1.424 2.117 1.424 3.483z"/></g></g></svg>
```

## File: static\src\js\website_links.js

```javascript
odoo.define('website_links.website_links', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var _t = core._t;

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
    },
    /**
     * @override
     */
    willStart: function () {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        defs.push(this._rpc({
            model: this.obj,
            method: 'search_read',
            params: {
                fields: ['id', 'name'],
            },
        }).then(function (result) {
            self.objects = _.map(result, function (val) {
                return {id: val.id, text: val.name};
            });
        }));
        return Promise.all(defs);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$el.select2({
            placeholder: self.placeholder,
            allowClear: true,
            createSearchChoice: function (term) {
                if (self._objectExists(term)) {
                    return null;
                }
                return {id: term, text: _.str.sprintf("Create '%s'", term)};
            },
            createSearchChoicePosition: 'bottom',
            multiple: false,
            data: self.objects,
            minimumInputLength: self.objects.length > 100 ? 3 : 0,
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
        return _.find(this.objects, function (val) {
            return val.text.toLowerCase() === query.toLowerCase();
        }) !== undefined;
    },
    /**
     * @private
     * @param {String} name
     */
    _createObject: function (name) {
        var self = this;
        var args = {
            name: name
        };
        if (this.obj === "utm.campaign"){
            args.is_website = true;
        }
        return this._rpc({
            model: this.obj,
            method: 'create',
            args: [args],
        }).then(function (record) {
            self.$el.attr('value', record);
            self.objects.push({'id': record, 'text': name});
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
        if (!ev.added || !_.isString(ev.added.id)) {
            return;
        }
        this._createObject(ev.added.id);
    },
});

var RecentLinkBox = publicWidget.Widget.extend({
    template: 'website_links.RecentLink',
    xmlDependencies: ['/website_links/static/src/xml/recent_link.xml'],
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
    },
    /**
     * @override
     */
    start: function () {
        new ClipboardJS(this.$('.btn_shorten_url_clipboard').get(0));
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _toggleCopyButton: function () {
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
            this._rpc({
                route: '/website_links/add_code',
                params: {
                    init_code: initCode,
                    new_code: newCode,
                },
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

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    getRecentLinks: function (filter) {
        var self = this;
        return this._rpc({
            route: '/website_links/recent_links',
            params: {
                filter: filter,
                limit: 20,
            },
        }).then(function (result) {
            _.each(result.reverse(), function (link) {
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
        _.invoke(this.getChildren(), 'destroy');
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

    /**
     * @override
     */
    start: function () {
        var defs = [this._super.apply(this, arguments)];

        // UTMS selects widgets
        var campaignSelect = new SelectBox(this, 'utm.campaign', _t("e.g. Promotion of June, Winter Newsletter, .."));
        defs.push(campaignSelect.attachTo($('#campaign-select')));

        var mediumSelect = new SelectBox(this, 'utm.medium', _t("e.g. Newsletter, Social Network, .."));
        defs.push(mediumSelect.attachTo($('#channel-select')));

        var sourceSelect = new SelectBox(this, 'utm.source', _t("e.g. Search Engine, Website page, .."));
        defs.push(sourceSelect.attachTo($('#source-select')));

        // Recent Links Widgets
        this.recentLinks = new RecentLinks(this);
        defs.push(this.recentLinks.appendTo($('#o_website_links_recent_links')));
        this.recentLinks.getRecentLinks('newest');

        // Clipboard Library
        new ClipboardJS($('#btn_shorten_url').get(0));

        this.url_copy_animating = false;

        $('[data-toggle="tooltip"]').tooltip();

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
        if (!$('#btn_shorten_url').hasClass('btn-copy') || ev.which === 13) {
            return;
        }

        $('#btn_shorten_url').removeClass('btn-success btn-copy').addClass('btn-primary').html('Get tracked link');
        $('#generated_tracked_link').css('display', 'none');
        $('.o_website_links_utm_forms').show();
    },
    /**
     * @private
     */
    _onShortenUrlButtonClick: function () {
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

        this._rpc({
            route: '/website_links/new',
            params: params,
        }).then(function (result) {
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

return {
    SelectBox: SelectBox,
    RecentLinkBox: RecentLinkBox,
    RecentLinks: RecentLinks,
};
});

```

## File: static\src\js\website_links_charts.js

```javascript
odoo.define('website_links.charts', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var _t = core._t;

var BarChart = publicWidget.Widget.extend({
    jsLibs: [
        '/web/static/lib/Chart/Chart.js',
    ],
    /**
     * @constructor
     * @param {Object} parent
     * @param {Object} beginDate
     * @param {Object} endDate
     * @param {Object} dates
     */
    init: function (parent, beginDate, endDate, dates) {
        this._super.apply(this, arguments);
        this.beginDate = beginDate.locale("en");
        this.endDate = endDate;
        this.number_of_days = this.endDate.diff(this.beginDate, 'days') + 2;
        this.dates = dates;
    },
    /**
     * @override
     */
    start: function () {
        // Fill data for each day (with 0 click for days without data)
        var clicksArray = [];
        var beginDateCopy = this.beginDate;
        for (var i = 0; i < this.number_of_days; i++) {
            var dateKey = beginDateCopy.format('YYYY-MM-DD');
            clicksArray.push([dateKey, (dateKey in this.dates) ? this.dates[dateKey] : 0]);
            beginDateCopy.add(1, 'days');
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
});

var PieChart = publicWidget.Widget.extend({
    jsLibs: [
        '/web/static/lib/Chart/Chart.js',
    ],
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
        };

        var canvas = this.$('canvas')[0];
        var context = canvas.getContext('2d');
        new Chart(context, config);
    },
});

publicWidget.registry.websiteLinksCharts = publicWidget.Widget.extend({
    selector: '.o_website_links_chart',
    events: {
        'click .graph-tabs li a': '_onGraphTabClick',
        'click .copy-to-clipboard': '_onCopyToClipboardClick',
    },

    /**
     * @override
     */
    start: function () {
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

        new ClipboardJS($('.copy-to-clipboard')[0]);

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
                const date = moment(
                    _clicksByDay[i]["__domain"].find((el) => el.length && el.includes(">="))[2]
                        .split(" ")[0], "YYYY MM DD"
                );
                if (i === 0) {
                    beginDate = date;
                }
                formattedClicksByDay[date.locale("en").format("YYYY-MM-DD")] =
                    _clicksByDay[i]["create_date_count"];
            }

            // Process all time line chart data
            var now = moment();
            self.charts.all_time_bar = new BarChart(self, beginDate, now, formattedClicksByDay);
            self.charts.all_time_bar.attachTo($('#all_time_clicks_chart'));

            // Process month line chart data
            beginDate = moment().subtract(30, 'days');
            self.charts.last_month_bar = new BarChart(self, beginDate, now, formattedClicksByDay);
            self.charts.last_month_bar.attachTo($('#last_month_clicks_chart'));

            // Process week line chart data
            beginDate = moment().subtract(7, 'days');
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
        return this._rpc({
            model: 'link.tracker.click',
            method: 'search_count',
            args: [[this.links_domain]],
        });
    },
    /**
     * @private
     */
    _clicksByDay: function () {
        return this._rpc({
            model: 'link.tracker.click',
            method: 'read_group',
            args: [[this.links_domain], ['create_date']],
            kwargs: {groupby: 'create_date:day'},
        });
    },
    /**
     * @private
     */
    _clicksByCountry: function () {
        return this._rpc({
            model: 'link.tracker.click',
            method: 'read_group',
            args: [[this.links_domain], ['country_id']],
            kwargs: {groupby: 'country_id'},
        });
    },
    /**
     * @private
     */
    _lastWeekClicksByCountry: function () {
        // 7 days * 24 hours * 60 minutes * 60 seconds * 1000 milliseconds.
        const aWeekAgoDate = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
        // get the date in the format YYYY-MM-DD.
        const aWeekAgoString = aWeekAgoDate.toISOString().split("T")[0];
        return this._rpc({
            model: 'link.tracker.click',
            method: 'read_group',
            args: [[this.links_domain, ["create_date", ">", aWeekAgoString]], ["country_id"]],
            kwargs: {groupby: 'country_id'},
        });
    },
    /**
     * @private
     */
    _lastMonthClicksByCountry: function () {
        // 30 days * 24 hours * 60 minutes * 60 seconds * 1000 milliseconds.
        const aMonthAgoDate = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
        // get the date in the format YYYY-MM-DD.
        const aMonthAgoString = aMonthAgoDate.toISOString().split("T")[0];
        return this._rpc({
            model: 'link.tracker.click',
            method: 'read_group',
            args: [[this.links_domain, ["create_date", ">", aMonthAgoString]], ["country_id"]],
            kwargs: {groupby: 'country_id'},
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onGraphTabClick: function (ev) {
        ev.preventDefault();
        $('.graph-tabs li a').tab('show');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onCopyToClipboardClick: function (ev) {
        ev.preventDefault();

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

return {
    BarChart: BarChart,
    PieChart: PieChart,
};
});

```

## File: static\src\js\website_links_code_editor.js

```javascript
odoo.define('website_links.code_editor', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');

var _t = core._t;

publicWidget.registry.websiteLinksCodeEditor = publicWidget.Widget.extend({
    selector: '#wrapwrap:has(.o_website_links_edit_code)',
    events: {
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
            return this._rpc({
                route: '/website_links/add_code',
                params: {
                    init_code: initCode,
                    new_code: newCode,
                },
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
});

```

## File: static\src\js\website_links_menu.js

```javascript
/**
 * The purpose of this script is to copy the current URL of the website
 * into the URL form of the URL shortener (module website_links)
 * when the user clicks the link "Share this page" on top of the page.
 */

odoo.define('website_links.website_links_menu', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var websiteNavbarData = require('website.navbar');

var WebsiteLinksMenu = publicWidget.Widget.extend({

    /**
     * @override
     */
    start: function () {
        this.$el.attr('href', '/r?u=' + encodeURIComponent(window.location.href));
        return this._super.apply(this, arguments);
    },
});

websiteNavbarData.websiteNavbarRegistry.add(WebsiteLinksMenu, '#o_website_links_share_page');

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
                    <span class="badge badge-success"><t t-esc="widget.link_obj.campaign_id[1]"/></span>
                    <span class="badge badge-success"><t t-esc="widget.link_obj.medium_id[1]"/></span>
                    <span class="badge badge-success"><t t-esc="widget.link_obj.source_id[1]"/></span>
                </p>
                <p class='o_website_links_code_error' style='color:red;font-weight:bold;'></p>
            </div>

            <div class="col-md-4 col-2">
                <button class="btn btn-info btn_shorten_url_clipboard mt8" t-att-data-clipboard-text="widget.link_obj.short_url">Copy</button>
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
                            <small class="float-right d-none d-md-block mt16" id="filters">
                                <ul class="nav nav-tabs nav-tabs-inline graph-tabs" role="tablist">
                                    <li class="nav-item"><a aria-controls="all_time_charts" href="#all_time_charts" class="nav-link active" role="tab" data-toggle="tab">All Time</a></li>
                                    <li class="nav-item"><a aria-controls="last_month_charts" href="#last_month_charts" class="nav-link" role="tab" data-toggle="tab">Last Month</a></li>
                                    <li class="nav-item"><a aria-controls="last_week_charts" href="#last_week_charts" class="nav-link" role="tab" data-toggle="tab">Last Week</a></li>
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

        <template id="assets_website_links" inherit_id="website.assets_frontend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_links/static/src/js/website_links.js"/>
                <script type="text/javascript" src="/website_links/static/src/js/website_links_code_editor.js"/>
                <script type="text/javascript" src="/website_links/static/src/js/website_links_charts.js"/>
                <link rel="stylesheet" type="text/css" href="/website_links/static/src/css/website_links.css"/>
            </xpath>
        </template>

        <template id="assets_tests" name="Website Links Assets Tests" inherit_id="web.assets_tests">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_links/static/tests/tours/website_links.js"></script>
            </xpath>
        </template>

        <template id="share_page_menu" inherit_id="website.user_navbar">
            <xpath expr="//a[@data-action='promote-current-page']" position="after">
                <a href="/r" id="o_website_links_share_page" class="dropdown-item">
                    <span title="Track this page to count clicks">Link Tracker</span>
                </a>
            </xpath>

        </template>

        <template id="asset_website_links_menu" inherit_id="website.assets_editor">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_links/static/src/js/website_links_menu.js"></script>
            </xpath>
        </template>

        <template id="create_shorten_url">
            <div class="o_website_links_create_tracked_url">
                <div class="container">
                    <h1 class="o_page_header">Link Tracker</h1>
                    <div class="notification"></div>

                    <div class="row">
                        <div class="col-md-7">
                            <form id="o_website_links_link_tracker_form">

                                <div class="form-group row" id="url-form-group">
                                    <label class="col-md-3 col-form-label text-left">URL</label>

                                    <div class="col-md-9">
                                        <input type="text" id="url" class="form-control required-form-control"  required="True" placeholder="e.g. https://www.odoo.com/contactus" t-att-value="u"/>
                                    </div>
                                </div>

                                <div class="o_website_links_utm_forms">
                                    <div class="form-group row">
                                        <label class="col-md-3 col-form-label">Campaign <i class="fa fa-info-circle" data-toggle="tooltip" data-placement="top" role="img" aria-label="Tooltip info" title="Defines the context of your link. It might be an event you want to promote or a special promotion."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="campaign-select"/>
                                        </div>
                                    </div>

                                    <div class="form-group row">
                                        <label class="col-md-3 col-form-label">Medium <i class="fa fa-info-circle" data-toggle="tooltip" data-placement="top" role="img" aria-label="Tooltip info" title="Defines the medium used to share your link. It might be an email, or a Facebook Ads for instance."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="channel-select" />
                                        </div>
                                    </div>

                                    <div class="form-group row">
                                        <label class="col-md-3 col-form-label">Source <i class="fa fa-info-circle" data-toggle="tooltip" data-placement="top" role="img" aria-label="Tooltip info" title="Defines the source from which your traffic will come from, Facebook or Twitter for instance."></i></label>

                                        <div class="col-md-9">
                                            <input type="hidden" class="form-control" id="source-select" />
                                        </div>
                                    </div>
                                </div>

                                <div class="form-group row">
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
                        <small class="float-right d-none d-md-block" id="filters">
                            <ul class="nav nav-tabs nav-tabs-inline graph-tabs" role="tablist">
                                <li class="nav-item"><a aria-controls="filter-newest-links" href="#" class="nav-link active" id="filter-newest-links" role="tab" data-toggle="tab">Newest</a></li>
                                <li class="nav-item"><a aria-controls="filter-most-clicked-links" href="#" class="nav-link" id="filter-most-clicked-links" role="tab" data-toggle="tab">Most Clicked</a></li>
                                <li class="nav-item"><a aria-controls="filter-recently-used-links" href="#" class="nav-link" id="filter-recently-used-links" role="tab" data-toggle="tab">Recently Used</a></li>
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

