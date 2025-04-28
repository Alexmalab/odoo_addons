# Odoo Module: website

Category: Website/Website

This file contains the source code of the Odoo module.

## File: tools.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import contextlib
import re
from unittest.mock import Mock, MagicMock, patch

import werkzeug

import odoo
from odoo.tools.misc import DotDict


def get_video_embed_code(video_url):
    ''' Computes the valid iframe from given URL that can be embedded
        (or False in case of invalid URL).
    '''

    if not video_url:
        return False

    # To detect if we have a valid URL or not
    validURLRegex = r'^(http:\/\/|https:\/\/|\/\/)[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?$'

    # Regex for few of the widely used video hosting services
    ytRegex = r'^(?:(?:https?:)?\/\/)?(?:www\.)?(?:youtu\.be\/|youtube(-nocookie)?\.com\/(?:embed\/|v\/|watch\?v=|watch\?.+&v=))((?:\w|-){11})(?:\S+)?$'
    vimeoRegex = r'\/\/(player.)?vimeo.com\/([a-z]*\/)*([0-9]{6,11})[?]?.*'
    dmRegex = r'.+dailymotion.com\/(video|hub|embed)\/([^_?]+)[^#]*(#video=([^_&]+))?'
    igRegex = r'(.*)instagram.com\/p\/(.[a-zA-Z0-9]*)'
    ykuRegex = r'(.*).youku\.com\/(v_show\/id_|embed\/)(.+)'

    if not re.search(validURLRegex, video_url):
        return False
    else:
        embedUrl = False
        ytMatch = re.search(ytRegex, video_url)
        vimeoMatch = re.search(vimeoRegex, video_url)
        dmMatch = re.search(dmRegex, video_url)
        igMatch = re.search(igRegex, video_url)
        ykuMatch = re.search(ykuRegex, video_url)

        if ytMatch and len(ytMatch.groups()[1]) == 11:
            embedUrl = '//www.youtube%s.com/embed/%s?rel=0' % (ytMatch.groups()[0] or '', ytMatch.groups()[1])
        elif vimeoMatch:
            embedUrl = '//player.vimeo.com/video/%s' % (vimeoMatch.groups()[2])
        elif dmMatch:
            embedUrl = '//www.dailymotion.com/embed/video/%s' % (dmMatch.groups()[1])
        elif igMatch:
            embedUrl = '//www.instagram.com/p/%s/embed/' % (igMatch.groups()[1])
        elif ykuMatch:
            ykuLink = ykuMatch.groups()[2]
            if '.html?' in ykuLink:
                ykuLink = ykuLink.split('.html?')[0]
            embedUrl = '//player.youku.com/embed/%s' % (ykuLink)
        else:
            # We directly use the provided URL as it is
            embedUrl = video_url
        return '<iframe class="embed-responsive-item" src="%s" allowFullScreen="true" frameborder="0"></iframe>' % embedUrl


def werkzeugRaiseNotFound(*args, **kwargs):
    raise werkzeug.exceptions.NotFound()


@contextlib.contextmanager
def MockRequest(
        env, *, routing=True, multilang=True,
        context=None,
        cookies=None, country_code=None, website=None, sale_order_id=None
):
    router = MagicMock()
    match = router.return_value.bind.return_value.match
    if routing:
        match.return_value[0].routing = {
            'type': 'http',
            'website': True,
            'multilang': multilang
        }
    else:
        match.side_effect = werkzeugRaiseNotFound

    if context is None:
        context = {}
    lang_code = context.get('lang', env.context.get('lang', 'en_US'))
    context.setdefault('lang', lang_code)

    request = Mock(
        context=context,
        db=None,
        endpoint=match.return_value[0] if routing else None,
        env=env,
        httprequest=Mock(
            host='localhost',
            path='/hello/',
            app=odoo.http.root,
            environ={'REMOTE_ADDR': '127.0.0.1'},
            cookies=cookies or {},
            referrer='',
        ),
        lang=env['res.lang']._lang_get(lang_code),
        redirect=werkzeug.utils.redirect,
        session=DotDict(
            geoip={'country_code': country_code},
            debug=False,
            sale_order_id=sale_order_id,
        ),
        website=website
    )

    with contextlib.ExitStack() as s:
        odoo.http._request_stack.push(request)
        s.callback(odoo.http._request_stack.pop)
        s.enter_context(patch('odoo.http.root.get_db_router', router))

        yield request

```

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import wizard

import odoo
from odoo import api, SUPERUSER_ID
from functools import partial


def uninstall_hook(cr, registry):
    def rem_website_id_null(dbname):
        db_registry = odoo.modules.registry.Registry.new(dbname)
        with api.Environment.manage(), db_registry.cursor() as cr:
            env = api.Environment(cr, SUPERUSER_ID, {})
            env['ir.model.fields'].search([
                ('name', '=', 'website_id'),
                ('model', '=', 'res.config.settings'),
            ]).unlink()
    cr.after('commit', partial(rem_website_id_null, cr.dbname))

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website',
    'category': 'Website/Website',
    'sequence': 7,
    'summary': 'Enterprise website builder',
    'website': 'https://www.odoo.com/page/website-builder',
    'version': '1.0',
    'description': "",
    'depends': [
        'web',
        'web_editor',
        'http_routing',
        'portal',
        'social_media',
        'auth_signup',
    ],
    'installable': True,
    'data': [
        'data/website_data.xml',
        'data/website_visitor_cron.xml',
        'security/website_security.xml',
        'security/ir.model.access.csv',
        'views/website_templates.xml',
        'views/website_navbar_templates.xml',
        'views/snippets.xml',
        'views/website_views.xml',
        'views/website_visitor_views.xml',
        'views/res_config_settings_views.xml',
        'views/website_rewrite.xml',
        'views/ir_actions_views.xml',
        'views/ir_attachment_views.xml',
        'views/res_partner_views.xml',
        'wizard/base_language_install_views.xml',
    ],
    'demo': [
        'data/website_demo.xml',
    ],
    'qweb': ['static/src/xml/website.backend.xml'],
    'application': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\backend.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.tools.translate import _


class WebsiteBackend(http.Controller):

    @http.route('/website/fetch_dashboard_data', type="json", auth='user')
    def fetch_dashboard_data(self, website_id, date_from, date_to):
        Website = request.env['website']
        has_group_system = request.env.user.has_group('base.group_system')
        has_group_designer = request.env.user.has_group('website.group_website_designer')
        dashboard_data = {
            'groups': {
                'system': has_group_system,
                'website_designer': has_group_designer
            },
            'currency': request.env.company.currency_id.id,
            'dashboards': {
                'visits': {},
            }
        }

        current_website = website_id and Website.browse(website_id) or Website.get_current_website()
        multi_website = request.env.user.has_group('website.group_multi_website')
        websites = multi_website and request.env['website'].search([]) or current_website
        dashboard_data['websites'] = websites.read(['id', 'name'])
        for rec, website in zip(websites, dashboard_data['websites']):
            website['domain'] = rec._get_http_domain()
            if website['id'] == current_website.id:
                website['selected'] = True

        if has_group_designer:
            if current_website.google_management_client_id and current_website.google_analytics_key:
                dashboard_data['dashboards']['visits'] = dict(
                    ga_client_id=current_website.google_management_client_id or '',
                    ga_analytics_key=current_website.google_analytics_key or '',
                )
        return dashboard_data

    @http.route('/website/dashboard/set_ga_data', type='json', auth='user')
    def website_set_ga_data(self, website_id, ga_client_id, ga_analytics_key):
        if not request.env.user.has_group('base.group_system'):
            return {
                'error': {
                    'title': _('Access Error'),
                    'message': _('You do not have sufficient rights to perform that action.'),
                }
            }
        if not ga_analytics_key or not ga_client_id.endswith('.apps.googleusercontent.com'):
            return {
                'error': {
                    'title': _('Incorrect Client ID / Key'),
                    'message': _('The Google Analytics Client ID or Key you entered seems incorrect.'),
                }
            }
        Website = request.env['website']
        current_website = website_id and Website.browse(website_id) or Website.get_current_website()

        request.env['res.config.settings'].create({
            'google_management_client_id': ga_client_id,
            'google_analytics_key': ga_analytics_key,
            'website_id': current_website.id,
        }).execute()
        return True

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import datetime
import json
import os
import logging
import pytz
import requests
import werkzeug.utils
import werkzeug.wrappers

from itertools import islice
from werkzeug import urls
from xml.etree import ElementTree as ET

import odoo

from odoo import http, models, fields, _
from odoo.http import request
from odoo.tools import OrderedSet
from odoo.addons.http_routing.models.ir_http import slug, _guess_mimetype
from odoo.addons.web.controllers.main import Binary
from odoo.addons.portal.controllers.portal import pager as portal_pager
from odoo.addons.portal.controllers.web import Home

logger = logging.getLogger(__name__)

# Completely arbitrary limits
MAX_IMAGE_WIDTH, MAX_IMAGE_HEIGHT = IMAGE_LIMITS = (1024, 768)
LOC_PER_SITEMAP = 45000
SITEMAP_CACHE_TIME = datetime.timedelta(hours=12)


class QueryURL(object):
    def __init__(self, path='', path_args=None, **args):
        self.path = path
        self.args = args
        self.path_args = OrderedSet(path_args or [])

    def __call__(self, path=None, path_args=None, **kw):
        path = path or self.path
        for key, value in self.args.items():
            kw.setdefault(key, value)
        path_args = OrderedSet(path_args or []) | self.path_args
        paths, fragments = {}, []
        for key, value in kw.items():
            if value and key in path_args:
                if isinstance(value, models.BaseModel):
                    paths[key] = slug(value)
                else:
                    paths[key] = u"%s" % value
            elif value:
                if isinstance(value, list) or isinstance(value, set):
                    fragments.append(werkzeug.url_encode([(key, item) for item in value]))
                else:
                    fragments.append(werkzeug.url_encode([(key, value)]))
        for key in path_args:
            value = paths.get(key)
            if value is not None:
                path += '/' + key + '/' + value
        if fragments:
            path += '?' + '&'.join(fragments)
        return path


class Website(Home):

    @http.route('/', type='http', auth="public", website=True)
    def index(self, **kw):
        homepage = request.website.homepage_id
        if homepage and (homepage.sudo().is_visible or request.env.user.has_group('base.group_user')) and homepage.url != '/':
            return request.env['ir.http'].reroute(homepage.url)

        website_page = request.env['ir.http']._serve_page()
        if website_page:
            return website_page
        else:
            top_menu = request.website.menu_id
            first_menu = top_menu and top_menu.child_id and top_menu.child_id.filtered(lambda menu: menu.is_visible)
            if first_menu and first_menu[0].url not in ('/', '', '#') and (not (first_menu[0].url.startswith(('/?', '/#', ' ')))):
                return request.redirect(first_menu[0].url)

        raise request.not_found()

    @http.route('/website/force/<int:website_id>', type='http', auth="user", website=True, sitemap=False, multilang=False)
    def website_force(self, website_id, path='/', isredir=False, **kw):
        """ To switch from a website to another, we need to force the website in
        session, AFTER landing on that website domain (if set) as this will be a
        different session.
        """
        parse = werkzeug.urls.url_parse
        safe_path = parse(path).path

        if not (request.env.user.has_group('website.group_multi_website')
           and request.env.user.has_group('website.group_website_publisher')):
            # The user might not be logged in on the forced website, so he won't
            # have rights. We just redirect to the path as the user is already
            # on the domain (basically a no-op as it won't change domain or
            # force website).
            # Website 1 : 127.0.0.1 (admin)
            # Website 2 : 127.0.0.2 (not logged in)
            # Click on "Website 2" from Website 1
            return request.redirect(safe_path)

        website = request.env['website'].browse(website_id)

        if not isredir and website.domain:
            domain_from = request.httprequest.environ.get('HTTP_HOST', '')
            domain_to = parse(website._get_http_domain()).netloc
            if domain_from != domain_to:
                # redirect to correct domain for a correct routing map
                url_to = urls.url_join(website._get_http_domain(), '/website/force/%s?isredir=1&path=%s' % (website.id, safe_path))
                return request.redirect(url_to)
        website._force()
        return request.redirect(safe_path)

    # ------------------------------------------------------
    # Login - overwrite of the web login so that regular users are redirected to the backend
    # while portal users are redirected to the frontend by default
    # ------------------------------------------------------

    @http.route(website=True, auth="public", sitemap=False)
    def web_login(self, redirect=None, *args, **kw):
        response = super(Website, self).web_login(redirect=redirect, *args, **kw)
        if not redirect and request.params['login_success']:
            if request.env['res.users'].browse(request.uid).has_group('base.group_user'):
                redirect = b'/web?' + request.httprequest.query_string
            else:
                redirect = '/my'
            return http.redirect_with_hash(redirect)
        return response

    # ------------------------------------------------------
    # Business
    # ------------------------------------------------------

    @http.route('/website/get_languages', type='json', auth="user", website=True)
    def website_languages(self, **kwargs):
        return [(lg.code, lg.url_code, lg.name) for lg in request.website.language_ids]

    @http.route('/website/lang/<lang>', type='http', auth="public", website=True, multilang=False)
    def change_lang(self, lang, r='/', **kwargs):
        """ :param lang: supposed to be value of `url_code` field """
        r = request.website._get_relative_url(r)
        if lang == 'default':
            lang = request.website.default_lang_id.url_code
            r = '/%s%s' % (lang, r or '/')
        redirect = werkzeug.utils.redirect(r or ('/%s' % lang), 303)
        lang_code = request.env['res.lang']._lang_get_code(lang)
        redirect.set_cookie('frontend_lang', lang_code)
        return redirect

    @http.route(['/website/country_infos/<model("res.country"):country>'], type='json', auth="public", methods=['POST'], website=True)
    def country_infos(self, country, **kw):
        fields = country.get_address_fields()
        return dict(fields=fields, states=[(st.id, st.name, st.code) for st in country.state_ids], phone_code=country.phone_code)

    @http.route(['/robots.txt'], type='http', auth="public")
    def robots(self, **kwargs):
        return request.render('website.robots', {'url_root': request.httprequest.url_root}, mimetype='text/plain')

    @http.route('/sitemap.xml', type='http', auth="public", website=True, multilang=False, sitemap=False)
    def sitemap_xml_index(self, **kwargs):
        current_website = request.website
        Attachment = request.env['ir.attachment'].sudo()
        View = request.env['ir.ui.view'].sudo()
        mimetype = 'application/xml;charset=utf-8'
        content = None

        def create_sitemap(url, content):
            return Attachment.create({
                'datas': base64.b64encode(content),
                'mimetype': mimetype,
                'type': 'binary',
                'name': url,
                'url': url,
            })
        dom = [('url', '=', '/sitemap-%d.xml' % current_website.id), ('type', '=', 'binary')]
        sitemap = Attachment.search(dom, limit=1)
        if sitemap:
            # Check if stored version is still valid
            create_date = fields.Datetime.from_string(sitemap.create_date)
            delta = datetime.datetime.now() - create_date
            if delta < SITEMAP_CACHE_TIME:
                content = base64.b64decode(sitemap.datas)

        if not content:
            # Remove all sitemaps in ir.attachments as we're going to regenerated them
            dom = [('type', '=', 'binary'), '|', ('url', '=like', '/sitemap-%d-%%.xml' % current_website.id),
                   ('url', '=', '/sitemap-%d.xml' % current_website.id)]
            sitemaps = Attachment.search(dom)
            sitemaps.unlink()

            pages = 0
            locs = request.website.with_user(request.website.user_id).enumerate_pages()
            while True:
                values = {
                    'locs': islice(locs, 0, LOC_PER_SITEMAP),
                    'url_root': request.httprequest.url_root[:-1],
                }
                urls = View.render_template('website.sitemap_locs', values)
                if urls.strip():
                    content = View.render_template('website.sitemap_xml', {'content': urls})
                    pages += 1
                    last_sitemap = create_sitemap('/sitemap-%d-%d.xml' % (current_website.id, pages), content)
                else:
                    break

            if not pages:
                return request.not_found()
            elif pages == 1:
                # rename the -id-page.xml => -id.xml
                last_sitemap.write({
                    'url': "/sitemap-%d.xml" % current_website.id,
                    'name': "/sitemap-%d.xml" % current_website.id,
                })
            else:
                # TODO: in master/saas-15, move current_website_id in template directly
                pages_with_website = ["%d-%d" % (current_website.id, p) for p in range(1, pages + 1)]

                # Sitemaps must be split in several smaller files with a sitemap index
                content = View.render_template('website.sitemap_index_xml', {
                    'pages': pages_with_website,
                    'url_root': request.httprequest.url_root,
                })
                create_sitemap('/sitemap-%d.xml' % current_website.id, content)

        return request.make_response(content, [('Content-Type', mimetype)])

    @http.route('/website/info', type='http', auth="public", website=True)
    def website_info(self, **kwargs):
        try:
            request.website.get_template('website.website_info').name
        except Exception as e:
            return request.env['ir.http']._handle_exception(e)
        Module = request.env['ir.module.module'].sudo()
        apps = Module.search([('state', '=', 'installed'), ('application', '=', True)])
        l10n = Module.search([('state', '=', 'installed'), ('name', '=like', 'l10n_%')])
        values = {
            'apps': apps,
            'l10n': l10n,
            'version': odoo.service.common.exp_version()
        }
        return request.render('website.website_info', values)

    # ------------------------------------------------------
    # Edit
    # ------------------------------------------------------

    @http.route(['/website/pages', '/website/pages/page/<int:page>'], type='http', auth="user", website=True)
    def pages_management(self, page=1, sortby='url', search='', **kw):
        # only website_designer should access the page Management
        if not request.env.user.has_group('website.group_website_designer'):
            raise werkzeug.exceptions.NotFound()

        Page = request.env['website.page']
        searchbar_sortings = {
            'url': {'label': _('Sort by Url'), 'order': 'url'},
            'name': {'label': _('Sort by Name'), 'order': 'name'},
        }
        # default sortby order
        sort_order = searchbar_sortings.get(sortby, 'url')['order'] + ', website_id desc, id'

        domain = request.website.website_domain()
        if search:
            domain += ['|', ('name', 'ilike', search), ('url', 'ilike', search)]

        pages = Page.search(domain, order=sort_order)
        if sortby != 'url' or not request.env.user.has_group('website.group_multi_website'):
            pages = pages.filtered(pages._is_most_specific_page)
        pages_count = len(pages)

        step = 50
        pager = portal_pager(
            url="/website/pages",
            url_args={'sortby': sortby},
            total=pages_count,
            page=page,
            step=step
        )

        pages = pages[(page - 1) * step:page * step]

        values = {
            'pager': pager,
            'pages': pages,
            'search': search,
            'sortby': sortby,
            'searchbar_sortings': searchbar_sortings,
        }
        return request.render("website.list_website_pages", values)

    @http.route(['/website/add/', '/website/add/<path:path>'], type='http', auth="user", website=True)
    def pagenew(self, path="", noredirect=False, add_menu=False, template=False, **kwargs):
        # for supported mimetype, get correct default template
        _, ext = os.path.splitext(path)
        ext_special_case = ext and ext in _guess_mimetype() and ext != '.html'

        if not template and ext_special_case:
            default_templ = 'website.default_%s' % ext.lstrip('.')
            if request.env.ref(default_templ, False):
                template = default_templ

        template = template and dict(template=template) or {}
        page = request.env['website'].new_page(path, add_menu=add_menu, **template)
        url = page['url']
        if noredirect:
            return werkzeug.wrappers.Response(url, mimetype='text/plain')

        if ext_special_case:  # redirect non html pages to backend to edit
            return werkzeug.utils.redirect('/web#id=' + str(page.get('view_id')) + '&view_type=form&model=ir.ui.view')
        return werkzeug.utils.redirect(url + "?enable_editor=1")

    @http.route("/website/get_switchable_related_views", type="json", auth="user", website=True)
    def get_switchable_related_views(self, key):
        views = request.env["ir.ui.view"].get_related_views(key, bundles=False).filtered(lambda v: v.customize_show)
        views = views.sorted(key=lambda v: (v.inherit_id.id, v.name))
        return views.read(['name', 'id', 'key', 'xml_id', 'arch', 'active', 'inherit_id'])

    @http.route('/website/toggle_switchable_view', type='json', auth='user', website=True)
    def toggle_switchable_view(self, view_key):
        request.website.viewref(view_key).toggle()

    @http.route('/website/reset_template', type='http', auth='user', methods=['POST'], website=True, csrf=False)
    def reset_template(self, view_id, mode='soft', redirect='/', **kwargs):
        """ This method will try to reset a broken view.
        Given the mode, the view can either be:
        - Soft reset: restore to previous architeture.
        - Hard reset: it will read the original `arch` from the XML file if the
        view comes from an XML file (arch_fs).
        """
        view = request.env['ir.ui.view'].browse(int(view_id))
        # Deactivate COW to not fix a generic view by creating a specific
        view.with_context(website_id=None).reset_arch(mode)
        return request.redirect(redirect)

    @http.route(['/website/publish'], type='json', auth="user", website=True)
    def publish(self, id, object):
        Model = request.env[object]
        record = Model.browse(int(id))

        values = {}
        if 'website_published' in Model._fields:
            values['website_published'] = not record.website_published
        record.write(values)
        return bool(record.website_published)

    @http.route(['/website/seo_suggest'], type='json', auth="user", website=True)
    def seo_suggest(self, keywords=None, lang=None):
        language = lang.split("_")
        url = "http://google.com/complete/search"
        try:
            req = requests.get(url, params={
                'ie': 'utf8', 'oe': 'utf8', 'output': 'toolbar', 'q': keywords, 'hl': language[0], 'gl': language[1]})
            req.raise_for_status()
            response = req.content
        except IOError:
            return []
        xmlroot = ET.fromstring(response)
        return json.dumps([sugg[0].attrib['data'] for sugg in xmlroot if len(sugg) and sugg[0].attrib['data']])

    # ------------------------------------------------------
    # Themes
    # ------------------------------------------------------

    def _get_customize_views(self, xml_ids):
        View = request.env["ir.ui.view"].with_context(active_test=False)
        if not xml_ids:
            return View
        domain = [("key", "in", xml_ids)] + request.website.website_domain()
        return View.search(domain).filter_duplicate()

    @http.route(['/website/theme_customize_get'], type='json', auth="public", website=True)
    def theme_customize_get(self, xml_ids):
        views = self._get_customize_views(xml_ids)
        return {
            'enabled': views.filtered('active').mapped('key'),
            'names': {view.key: view.name for view in views},
        }

    @http.route(['/website/theme_customize'], type='json', auth="public", website=True)
    def theme_customize(self, enable=None, disable=None, get_bundle=False):
        """ enable or Disable lists of ``xml_id`` of the inherit templates """

        self._get_customize_views(disable).write({'active': False})
        self._get_customize_views(enable).write({'active': True})

        if get_bundle:
            context = dict(request.context)
            return {
                'web.assets_common': request.env['ir.qweb']._get_asset_link_urls('web.assets_common', options=context),
                'web.assets_frontend': request.env['ir.qweb']._get_asset_link_urls('web.assets_frontend', options=context),
                'website.assets_editor': request.env['ir.qweb']._get_asset_link_urls('website.assets_editor', options=context),
            }

        return True

    @http.route(['/website/theme_customize_reload'], type='http', auth="public", website=True)
    def theme_customize_reload(self, href, enable, disable, tab=0, **kwargs):
        self.theme_customize(enable and enable.split(",") or [], disable and disable.split(",") or [])
        return request.redirect(href + ("&theme=true" if "#" in href else "#theme=true") + ("&tab=" + tab))

    @http.route(['/website/make_scss_custo'], type='json', auth='user', website=True)
    def make_scss_custo(self, url, values):
        """
        Params:
            url (str):
                the URL of the scss file to customize (supposed to be a variable
                file which will appear in the assets_common bundle)

            values (dict):
                key,value mapping to integrate in the file's map (containing the
                word hook). If a key is already in the file's map, its value is
                overridden.

        Returns:
            boolean
        """
        request.env['web_editor.assets'].make_scss_customization(url, values)
        return True

    @http.route(['/website/multi_render'], type='json', auth="public", website=True)
    def multi_render(self, ids_or_xml_ids, values=None):
        View = request.env['ir.ui.view']
        res = {}
        for id_or_xml_id in ids_or_xml_ids:
            res[id_or_xml_id] = View.render_template(id_or_xml_id, values)
        return res

    @http.route(['/website/update_visitor_timezone'], type='json', auth="public", website=True)
    def update_visitor_timezone(self, timezone):
        visitor_sudo = request.env['website.visitor']._get_visitor_from_request()
        if visitor_sudo:
            if timezone in pytz.all_timezones:
                visitor_sudo.write({'timezone': timezone})
                return True
        return False

    # ------------------------------------------------------
    # Server actions
    # ------------------------------------------------------

    @http.route([
        '/website/action/<path_or_xml_id_or_id>',
        '/website/action/<path_or_xml_id_or_id>/<path:path>',
    ], type='http', auth="public", website=True)
    def actions_server(self, path_or_xml_id_or_id, **post):
        ServerActions = request.env['ir.actions.server']
        action = action_id = None

        # find the action_id: either an xml_id, the path, or an ID
        if isinstance(path_or_xml_id_or_id, str) and '.' in path_or_xml_id_or_id:
            action = request.env.ref(path_or_xml_id_or_id, raise_if_not_found=False)
        if not action:
            action = ServerActions.search([('website_path', '=', path_or_xml_id_or_id), ('website_published', '=', True)], limit=1)
        if not action:
            try:
                action_id = int(path_or_xml_id_or_id)
            except ValueError:
                pass

        # check it effectively exists
        if action_id:
            action = ServerActions.browse(action_id).exists()
        # run it, return only if we got a Response object
        if action:
            if action.state == 'code' and action.website_published:
                action_res = action.run()
                if isinstance(action_res, werkzeug.wrappers.Response):
                    return action_res

        return request.redirect('/')


# ------------------------------------------------------
# Retrocompatibility routes
# ------------------------------------------------------
class WebsiteBinary(http.Controller):

    @http.route([
        '/website/image',
        '/website/image/<xmlid>',
        '/website/image/<xmlid>/<int:width>x<int:height>',
        '/website/image/<xmlid>/<field>',
        '/website/image/<xmlid>/<field>/<int:width>x<int:height>',
        '/website/image/<model>/<id>/<field>',
        '/website/image/<model>/<id>/<field>/<int:width>x<int:height>'
    ], type='http', auth="public", website=False, multilang=False)
    def content_image(self, id=None, max_width=0, max_height=0, **kw):
        if max_width:
            kw['width'] = max_width
        if max_height:
            kw['height'] = max_height
        if id:
            id, _, unique = id.partition('_')
            kw['id'] = int(id)
            if unique:
                kw['unique'] = unique
        return Binary().content_image(**kw)

    @http.route(['/favicon.ico'], type='http', auth='public', website=True, multilang=False, sitemap=False)
    def favicon(self, **kw):
        # when opening a pdf in chrome, chrome tries to open the default favicon url
        return self.content_image(model='website', id=str(request.website.id), field='favicon', **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import backend
from . import main

```

## File: data\website_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- split between ir_ui_view & website_page record to keep external_id on ir_ui_view -->
        <record id="homepage" model="ir.ui.view">
            <field name="name">Home</field>
            <field name="type">qweb</field>
            <field name="key">website.homepage</field>
            <field name="arch" type="xml">
                <t name="Home" priority="29" t-name="website.homepage">
                    <t t-call="website.layout">
                        <t t-set="pageName" t-value="'homepage'"/>
                        <div id="wrap" class="oe_structure oe_empty"/>
                    </t>
                </t>
            </field>
        </record>

        <record id="contactus" model="ir.ui.view">
            <field name="name">Contact Us</field>
            <field name="type">qweb</field>
            <field name="key">website.contactus</field>
            <field name="arch" type="xml">
                <t name="Contact Us" t-name="website.contactus">
                    <t t-call="website.layout">
                        <div id="wrap">
                            <div class="oe_structure">
                                <section class="pt8 pb8">
                                    <div class="container">
                                        <div class="row s_nb_column_fixed">
                                            <div class="col-lg-12 s_title pt16 pb16">
                                                <h1>Contact us</h1>
                                            </div>
                                        </div>
                                    </div>
                                </section>
                            </div>
                            <div class="container mt-2">
                                <div class="row">
                                    <div class="col-lg-8">
                                        <div class="oe_structure">
                                            <section class="s_text_block">
                                                <div class="container">
                                                    <div class="row">
                                                        <div class="col-lg-12">
                                                            <p>
                                                                Contact us about anything related to our company or services.<br/>
                                                                We'll do our best to get back to you as soon as possible.
                                                            </p>
                                                        </div>
                                                    </div>
                                                </div>
                                            </section>
                                        </div>
                                        <div class="text-center my-3" name="mail_button">
                                            <a role="button" t-attf-href="mailto:{{ res_company.email }}" class="btn btn-primary" id="o_contact_mail">Send us an email</a>
                                        </div>
                                    </div>
                                    <div class="col-lg-4">
                                        <t t-call="website.company_description"/>
                                    </div>
                                </div>
                            </div>
                            <div class="oe_structure mt-2"/>
                        </div>
                    </t>
                </t>
            </field>
        </record>

        <record id="aboutus" model="ir.ui.view">
            <field name="name">About us</field>
            <field name="type">qweb</field>
            <field name="key">website.aboutus</field>
            <field name="arch" type="xml">
                <t name="About us" t-name="website.aboutus">
                    <t t-call="website.layout">
                        <div id="wrap">
                            <div class="oe_structure">
                                <section class="pt8 pb8">
                                    <div class="container">
                                        <div class="row s_nb_column_fixed">
                                            <div class="col-lg-12 s_title pt16 pb16">
                                                <h1 class="text-center">About us</h1>
                                                <h3 class="text-muted text-center">Great products for great people</h3>
                                            </div>
                                        </div>
                                    </div>
                                </section>
                                <section class="s_text_image pt8 pb8">
                                    <div class="container">
                                        <div class="row align-items-center">
                                            <div class="col-lg-6 pt16 pb16">
                                                <p>
                                                    We are a team of passionate people whose goal is to improve everyone's
                                                    life through disruptive products. We build great products to solve your
                                                    business problems.
                                                </p>
                                                <p>
                                                    Our products are designed for small to medium size companies willing to optimize
                                                    their performance.
                                                </p>
                                            </div>
                                            <div class="col-lg-6 pt16 pb16">
                                                <img src="/website/static/src/img/library/business_conference.jpg" class="img img-fluid shadow" alt="Our Team"/>
                                            </div>
                                        </div>
                                    </div>
                                </section>
                            </div>
                        </div>
                    </t>
                </t>
            </field>
        </record>
    </data>

    <data noupdate="1">
        <record id="homepage_page" model="website.page">
            <field name="is_published">True</field>
            <field name="url">/</field>
            <field name="view_id" ref="homepage"/>
            <field name="track">True</field>
        </record>
        <record id="contactus_page" model="website.page">
            <field name="url">/contactus</field>
            <field name="is_published">True</field>
            <field name="view_id" ref="contactus"/>
            <field name="track">True</field>
        </record>
        <record id="aboutus_page" model="website.page">
            <field name="is_published">True</field>
            <field name="url">/aboutus</field>
            <field name="view_id" ref="aboutus"/>
            <field name="track">True</field>
        </record>

        <!-- Default Menu to store module menus for new website -->
        <record id="main_menu" model="website.menu">
          <field name="name">Default Main Menu</field>
          <field name="url">/default-main-menu</field>
        </record>
        <record id="menu_home" model="website.menu">
            <field name="name">Home</field>
            <field name="url">/</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">10</field>
        </record>
        <record id="menu_contactus" model="website.menu">
            <field name="name">Contact us</field>
            <field name="url">/contactus</field>
            <field name="parent_id" ref="website.main_menu"/>
            <field name="sequence" type="int">60</field>
        </record>

        <record id="default_website" model="website">
            <field name="name">My Website</field>
            <field name="domain"></field>
            <field name="company_id" ref="base.main_company"/>
            <field name="user_id" ref="base.public_user"/>
            <!-- Correct homepage will be set during bootstraping --> 
        </record>

        <!-- Open website on install -->
        <record id="action_website" model="ir.actions.act_url">
            <field name="name">Website</field>
            <field name="url">/</field>
            <field name="target">self</field>
        </record>
        <record id="base.open_menu" model="ir.actions.todo">
            <field name="action_id" ref="action_website"/>
            <field name="state">open</field>
        </record>

        <!-- Pre loaded images -->
        <record id="website.business_conference" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">business_conference.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">binary</field>
            <field name="datas" type="base64" file="website/static/src/img/library/business_conference.jpg"/>
        </record>
        <record id="website.library_image_01" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">bridge.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/bridge.jpg</field>
        </record>
        <record id="website.library_image_02" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">deliver.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/deliver.jpg</field>
        </record>
        <record id="website.library_image_03" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">clock.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/clock.jpg</field>
        </record>
        <record id="website.library_image_04" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">manufacturing.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/manufacturing.jpg</field>
        </record>
        <record id="website.library_image_05" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">compass.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/compass.jpg</field>
        </record>
        <record id="website.library_image_06" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">gift.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/gift.jpg</field>
        </record>
        <record id="website.library_image_07" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">mobile_device.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/mobile_device.jpg</field>
        </record>
        <record id="website.library_image_08" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">mobile.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/mobile.jpg</field>
        </record>
        <record id="website.library_image_09" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">office.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/office.jpg</field>
        </record>
        <record id="website.library_image_10" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">ice_coffe.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/ice_coffe.jpg</field>
        </record>
        <record id="website.library_image_11" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">rocket.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/rocket.jpg</field>
        </record>
        <record id="website.library_image_12" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">sell.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/sell.jpg</field>
        </record>
        <record id="website.library_image_13" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">shop.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/shop.jpg</field>
        </record>
        <record id="website.library_image_14" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">sign.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/sign.jpg</field>
        </record>
        <record id="website.library_image_15" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">sweet.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/sweet.jpg</field>
        </record>
        <record id="website.library_image_16" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">wine.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/wine.jpg</field>
        </record>
        <record id="website.library_image_17" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">marketing.jpg</field>
            <field name="res_model">ir.ui.view</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/library/marketing.jpg</field>
        </record>

        <!-- Website Builder Background Images -->
        <record id="website.s_background_image_01" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_01.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/peak.jpg</field>
        </record>
        <record id="website.s_background_image_02" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_02.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/la.jpg</field>
        </record>
        <record id="website.s_background_image_03" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_03.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/panama-sky.jpg</field>
        </record>
        <record id="website.s_background_image_04" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_04.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/cubes.jpg</field>
        </record>
        <record id="website.s_background_image_05" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_05.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/building-profile.jpg</field>
        </record>
        <record id="website.s_background_image_06" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_06.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/type.jpg</field>
        </record>
        <record id="website.s_background_image_07" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_07.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/people.jpg</field>
        </record>
        <record id="website.s_background_image_08" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_08.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/city.jpg</field>
        </record>
        <record id="website.s_background_image_09" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_background_image_09.jpg</field>
            <field name="type">url</field>
            <field name="res_model">ir.ui.view</field>
            <field name="url">/website/static/src/img/backgrounds/sails.jpg</field>
        </record>

        <!-- Snippets' Default Images (to be replaced by themes) -->
        <record id="website.s_cover_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_cover_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_cover.jpg</field>
        </record>
        <record id="website.s_quotes_carousel_demo_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_quotes_carousel_image_01.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_quotes_carousel_1.jpg</field>
        </record>
        <record id="website.s_quotes_carousel_demo_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_quotes_carousel_image_02.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_quotes_carousel_2.jpg</field>
        </record>
        <record id="website.s_quotes_carousel_demo_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_quotes_carousel_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_3.png</field>
        </record>
        <record id="website.s_quotes_carousel_demo_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_quotes_carousel_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="website.s_quotes_carousel_demo_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_quotes_carousel_image_5.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_4.png</field>
        </record>
        <record id="website.s_image_text_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_image_text_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_image_text.jpg</field>
        </record>
        <record id="website.s_text_image_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_text_image_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_text_image.jpg</field>
        </record>
        <record id="website.s_carousel_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_carousel_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_carousel_1.jpg</field>
        </record>
        <record id="website.s_carousel_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_carousel_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_carousel_2.jpg</field>
        </record>
        <record id="website.s_carousel_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_carousel_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_carousel_3.jpg</field>
        </record>
        <record id="website.s_picture_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_picture_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_picture.jpg</field>
        </record>
        <record id="website.s_banner_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_banner_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_banner.jpg</field>
        </record>
        <record id="website.s_parallax_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_parallax_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_parallax.jpg</field>
        </record>
        <record id="website.s_reference_demo_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_1.png</field>
        </record>
        <record id="website.s_reference_demo_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_2.png</field>
        </record>
        <record id="website.s_reference_demo_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_3.png</field>
        </record>
        <record id="website.s_reference_demo_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_4.png</field>
        </record>
        <record id="website.s_reference_demo_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_5.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_5.png</field>
        </record>
        <record id="website.s_company_team_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_1.png</field>
        </record>
        <record id="website.s_company_team_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="website.s_company_team_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_3.png</field>
        </record>
        <record id="website.s_company_team_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_team_member_4.png</field>
        </record>
        <record id="website.s_mega_menu_menu_image_menu_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_mega_menu_menu_image_menu_default_image.png</field>
            <field name="type">url</field>
            <field name="url">/website/static/src/img/snippets_demo/s_references_5.png</field>
        </record>
    </data>
    <data>
        <record id="group_multi_website" model="res.groups">
            <field name="name">Multi-website</field>
            <field name="category_id" ref="base.module_category_hidden"/>
        </record>
    </data>
</odoo>

```

## File: data\website_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="bs_debug_view" model="ir.ui.view">
            <field name="name">BS Debug</field>
            <field name="type">qweb</field>
            <field name="key">website.bs_debug_page_view</field>
            <field name="arch" type="xml">
                <t name="Debug" t-name="website.bs_debug_page_view">
                    <t t-call="website.layout">
                        <t t-set="odoo_theme_colors" t-value="[['alpha', 'Alpha'], ['beta', 'Beta'], ['gamma', 'Gamma'], ['delta', 'Delta'], ['epsilon', 'Epsilon']]"/>
                        <t t-set="bs_theme_colors" t-value="[['primary', 'Primary'], ['secondary', 'Secondary'], ['success', 'Success'], ['info', 'Info'], ['warning', 'Warning'], ['danger', 'Danger'], ['light', 'Light'], ['dark', 'Dark']]"/>
                        <t t-set="bs_gray_colors" t-value="[['white', 'White'], ['100', '100'], ['200', '200'], ['300', '300'], ['400', '400'], ['500', '500'], ['600', '600'], ['700', '700'], ['800', '800'], ['900', '900'], ['black', 'Black']]"/>
                        <t t-set="all_theme_colors" t-value="odoo_theme_colors + bs_theme_colors"/>
                        <div id="wrap" class="oe_structure">
                            <section class="py-2">
                                <div class="container">
                                    <h1>Components</h1>
                                    <div class="row">
                                        <div class="col-md">
                                            <h2>Badge</h2>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <span t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <span t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                            </t>
                                            <h3 class="mt-2 h6">Pill</h3>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <span t-attf-class="badge mb-1 badge-pill badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <span t-attf-class="badge mb-1 badge-pill badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                            </t>
                                            <h3 class="mt-2 h6">Link</h3>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <a href="#" t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></a>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <a href="#" t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></a>
                                            </t>
                                            <h3 class="mt-2 h6">Autosizing</h3>
                                            <div class="h3">
                                                <t t-foreach="odoo_theme_colors" t-as="color">
                                                    <span t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                                </t>
                                                <br/>
                                                <t t-foreach="bs_theme_colors" t-as="color">
                                                    <span t-attf-class="badge mb-1 badge-#{color[0]}"><t t-esc="color[1]"/></span>
                                                </t>
                                            </div>

                                            <h2 class="mt-4">Button</h2>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <h3 class="mt-2 h6">Outline</h3>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-outline-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-outline-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <h3 class="mt-2 h6">Small</h3>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-sm btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-sm btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <h3 class="mt-2 h6">Large</h3>
                                            <t t-foreach="odoo_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-lg btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>
                                            <br/>
                                            <t t-foreach="bs_theme_colors" t-as="color">
                                                <button type="button" t-attf-class="btn mb-1 btn-lg btn-#{color[0]}"><t t-esc="color[1]"/></button>
                                            </t>

                                            <h2 class="mt-4">Dropdown</h2>
                                            <div class="dropdown">
                                                <button type="button" class="btn btn-primary dropdown-toggle" data-toggle="dropdown">Toggle</button>
                                                <div class="dropdown-menu">
                                                    <div class="dropdown-header">Header</div>
                                                    <a class="dropdown-item" href="#">Action</a>
                                                    <a class="dropdown-item" href="#">Something else here</a>
                                                    <div class="dropdown-divider"/>
                                                    <a class="dropdown-item" href="#">Separated link</a>
                                                </div>
                                            </div>

                                            <h2 class="mt-4">Navbar</h2>
                                            <nav class="navbar navbar-expand-lg navbar-light bg-light">
                                                <a class="navbar-brand" href="#">Navbar</a>
                                                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
                                                <span class="navbar-toggler-icon"></span>
                                                </button>

                                                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                                                    <ul class="navbar-nav mr-auto">
                                                        <li class="nav-item active">
                                                            <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
                                                        </li>
                                                        <li class="nav-item">
                                                            <a class="nav-link" href="#">Link</a>
                                                        </li>
                                                        <li class="nav-item">
                                                            <a class="nav-link disabled" href="#">Disabled</a>
                                                        </li>
                                                    </ul>
                                                    <form class="form-inline my-2 my-lg-0">
                                                        <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search"/>
                                                        <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
                                                    </form>
                                                </div>
                                            </nav>

                                            <h2 class="mt-4">Form</h2>
                                            <form>
                                                <div class="form-group">
                                                    <label for="exampleInputEmail1">Email address</label>
                                                    <input type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter email"/>
                                                    <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>
                                                </div>
                                            </form>

                                            <h2 class="mt-4">Pagination</h2>
                                            <nav>
                                                <ul class="pagination">
                                                    <li class="page-item disabled">
                                                        <a class="page-link" href="#" tabindex="-1">Previous</a>
                                                    </li>
                                                    <li class="page-item">
                                                        <a class="page-link" href="#">1</a>
                                                    </li>
                                                    <li class="page-item active">
                                                        <a class="page-link" href="#">2 <span class="sr-only">(current)</span></a>
                                                    </li>
                                                    <li class="page-item">
                                                        <a class="page-link" href="#">3</a>
                                                    </li>
                                                    <li class="page-item">
                                                        <a class="page-link" href="#">Next</a>
                                                    </li>
                                                </ul>
                                            </nav>
                                        </div>
                                        <div class="col-md-auto">
                                            <h2>Alert</h2>
                                            <t t-foreach="all_theme_colors" t-as="color">
                                                <div t-attf-class="alert alert-#{color[0]}">
                                                    This is a "<t t-esc="color[1]"/>" alert with a <a href="#" class="alert-link">link</a>.
                                                </div>
                                            </t>

                                            <h2 class="mt-4">Breadcrumb</h2>
                                            <nav aria-label="breadcrumb">
                                                <ol class="breadcrumb">
                                                    <li class="breadcrumb-item"><a href="#">Home</a></li>
                                                    <li class="breadcrumb-item"><a href="#">Library</a></li>
                                                    <li class="breadcrumb-item active" aria-current="page">Data</li>
                                                </ol>
                                            </nav>

                                            <h2 class="mt-4">Card</h2>
                                            <div class="card">
                                                <div class="card-header">
                                                    Card Header
                                                </div>
                                                <div class="card-body">
                                                    Card Body
                                                </div>
                                                <ul class="list-group list-group-flush">
                                                    <li class="list-group-item">Item 1</li>
                                                    <li class="list-group-item">Item 2</li>
                                                </ul>
                                                <div class="card-footer">
                                                    Card Footer
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </section>
                            <section class="py-2">
                                <div class="container">
                                    <h1>Utilities &amp; Typography</h1>
                                    <div class="row">
                                        <div class="col-md">
                                            <div class="row no-gutters">
                                                <t t-foreach="odoo_theme_colors" t-as="color">
                                                    <div t-attf-class="col-auto bg-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                            <div class="row no-gutters mt-2">
                                                <t t-foreach="bs_theme_colors" t-as="color">
                                                    <div t-attf-class="col-auto bg-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                            <div class="row no-gutters mt-2">
                                                <t t-foreach="bs_gray_colors" t-as="color">
                                                    <div t-attf-class="col-auto bg-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                            <div class="row no-gutters mt-4">
                                                <t t-foreach="odoo_theme_colors" t-as="color">
                                                    <div t-attf-class="col-auto text-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                            <div class="row no-gutters mt-2">
                                                <t t-foreach="bs_theme_colors" t-as="color">
                                                    <div t-attf-class="col-auto text-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                            <div class="row no-gutters mt-2">
                                                <t t-foreach="bs_gray_colors" t-as="color">
                                                    <div t-attf-class="col-auto text-#{color[0]}">
                                                        <div class="py-1 px-3"><t t-esc="color[1]"/></div>
                                                    </div>
                                                </t>
                                            </div>
                                        </div>
                                        <div class="col-md-auto">
                                            <h1>Headings 1</h1>
                                            <h2>Headings 2</h2>
                                            <h3>Headings 3</h3>
                                            <h4>Headings 4</h4>
                                            <h5>Headings 5</h5>
                                            <h6>Headings 6</h6>
                                            <p>Paragraph with <strong>bold</strong>, <span class="text-muted">muted</span> and <em>italic</em> texts</p>
                                            <p><a href="#">Link</a></p>
                                            <p><button type="button" class="btn btn-link">Link button</button></p>
                                        </div>
                                    </div>
                                </div>
                            </section>
                        </div>
                    </t>
                </t>
            </field>
        </record>

        <record id="snippets_debug_view" model="ir.ui.view">
            <field name="name">Snippet Debug</field>
            <field name="type">qweb</field>
            <field name="key">website.snippets_debug_page_view</field>
            <field name="arch" type="xml">
                <t name="Debug" t-name="website.snippets_debug_page_view">
                    <t t-call="website.layout">
                        <style>
                            #snippets_menu, #o_scroll > .o_panel > .o_panel_header {
                                display: none !important;
                            }
                            [data-oe-type="snippet"]:not([data-module-id])::before {
                                content: attr(name);
                                display: block;
                                padding: 16px;
                                background-color: lightgray;
                                color: black;
                                font-size: 24px;
                            }
                            [data-oe-type="snippet"]:not([data-module-id])::after {
                                content: "";
                                display: table;
                                clear: both;
                            }
                        </style>
                        <div id="wrap" class="oe_structure">
                            <t t-call="website.snippets"/>
                        </div>
                    </t>
                </t>
            </field>
        </record>
    </data>

    <data noupdate="1">
        <record id="website2" model="website">
            <field name="name">My Website 2</field>
            <field name="domain"></field>
        </record>

        <!-- BS Debug Page -->
        <!-- Showcase all (most?) BS components and utilities -->
        <record id="bs_debug_page" model="website.page">
            <field name="url">/website/demo/bootstrap</field>
            <field name="is_published">False</field>
            <field name="view_id" ref="bs_debug_view"/>
        </record>

        <!-- Snippet Debug Page -->
        <!-- Showcase all snippets -->
        <record id="snippets_debug_page" model="website.page">
            <field name="url">/website/demo/snippets</field>
            <field name="is_published">False</field>
            <field name="view_id" ref="snippets_debug_view"/>
        </record>
    </data>
</odoo>

```

## File: data\website_visitor_cron.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="website_visitor_cron" model="ir.cron">
        <field name="name">Website Visitor : Archive old visitors</field>
        <field name="model_id" ref="model_website_visitor"/>
        <field name="state">code</field>
        <field name="code">model._cron_archive_visitors()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="active" eval="True"/>
        <field name="doall" eval="False"/>
    </record>
</odoo>

```

## File: doc\website.snippet.rst

```rst
Website Snippet & Blocks
========================

The building blocks appear in the edit bar website. These prebuilt html block
allowing the designer to easily generate content on a page (drag and drop).
Snippets bind javascript object on custom part html code according to their
selector (jQuery) and javascript object. The snippets is also used to create
the drop zone.


Building Blocks
+++++++++++++++

Overwrite ``_getSnippetURL`` to set an other file to load the snippets (use by
website_mail for example)
Overwrite ``_computeSelectorFunctions`` to enable or disable other snippets. By default
the builder check if the node or his parent have the attribute data-oe-model

Trigger:
- ``snippet-dropped`` is triggered on ``#oe_snippets`` whith $target as attribute when a snippet is dropped
- ``snippet-activated`` is triggered on ``#oe_snippets`` (and on snippet) when a snippet is activated


Blocks
++++++

The ``blocks`` are the HTML code that can be drop in the page. The blocks consist
of a body and a thumbnail:
 - thumbnail:
   (have class ``oe_snippet_thumbnail``) contains a picture and a text used to
   display a preview in the edit bar that contains all the block list
 - body:
   (have class ``oe_snippet_body``) is the real part dropped in the page. The class
   ``oe_snippet_body`` is removed before inserting the block in the page.
e.g.:
    <div>
        <div class="oe_snippet_thumbnail">
            <img class="oe_snippet_thumbnail_img" src="...image src..."/>
            <span class="oe_snippet_thumbnail_title">...Block Name...</span>
        </div>
        <div class="oe_snippet_body">
            <!--
                The block with class 'oe_snippet_body' is inserted in the page.
                This class is removed when the block is dropped.
                The block can be made of any html tag and content. -->
        </div>
    </div>


Editor
++++++

The ``editor`` is the frame placed above the block being edited who contains buttons
(move, delete, clone) and customize menu. The ``editor`` load ``options`` based on
selectors defined in snippets


Options
+++++++

The ``option`` is the javascript object used to customize the HTML code.

Object:
 - this.``$target``:
   block html inserted inside the page
 - this.``$el``:
   html li list of this options
 - this.``$overlay``:
   html editor overlay who content resize bar, customize menu...

Methods:
 - ``_setActive``:
   highlight the customize menu item when the user click on customize, and click on
   an item.
 - ``start``:
   called when the editor is created on the DOM
 - ``onFocus``:
   called when the user click inside the block inserted in page and when the
   user drop on block into the page
 - ``onBlur``:
   called when the user click outside the block inserted in page, if the block
   is focused
 - ``onClone``:
   called when the snippet is duplicate
 - ``onRemove``:
   called when the snippet is removed (dom is removing after this tigger)
 - ``onBuilt:
   called just after that a thumbnail is drag and dropped into a drop zone.
   The content is already inserted in the page.
 - ``cleanForSave``:
   is called just before to save the vue. Sometime it's important to remove or add
   some datas (contentEditable, added classes to a running animation...)

Customize Methods:
All javascript option can defiend method call from the template on mouse over, on
click or to reset the default value (<li data-your_js_method="your_value"><a>...</a></li>).
The method receive the variable type (``over``, ``click`` or ``reset``), the method
value and the jQuery object of the HTML li item. (can be use for multi methods)

By default to custom method are defined:

 - ``check_class(type, className, $li)``:
   li must have data-check_class="a_classname_for_test" to call this method. This method
   toggle the className on $target
 - ``selectClass(type, className, $li)``:
   This method remove all other selectClass value (for this option) and add this current ClassName



Snippet
+++++++

The ``snippets`` are the HTML code to defined the drop zone and the linked javascript object.
All HTML li tag defined inside the snippets HTML are insert into the customize menu. All
data attributes is optional:

- ``data-selector``:
  Apply options on all The part of html who match with this jQuery selector.
  E.g.: If the selector is div, all div will be selected and can be highlighted and assigned an editor.
- ``data-js``:
  javascript to call when the ``editor`` is loaded
- ``data-drop-in``:
  The html part can be insert or move beside the selected html block (jQuery selector)
- ``data-drop-near``:
  The html part can be insert or move inside the selected html block (jQuery selector)
- HTML content like <li data-your_js_method="your_value"><a>...</a></li>:
  List of HTML li menu items displayed in customize menu. If the li tag have datas the methods are
  automatically called
- ``no-check``:
  The selectors are automatically compute to have elements inside the branding. If you use this option
  the check is not apply (for e.g.: to have a snippet for the grid view of website_sale)

t-snippet and data-snippet
++++++++++++++++++++++++++

User can call a snippet template with qweb or inside a demo page.

e.g.:

<template id="website.name_of_the_snippet" name="Name of the snippet">
  <hr/>
</template>

Inside #snippet_structure for e.g.: ``<t t-snippet="website.name_of_the_snippet" t-thumbnail="/image_path"/>``
The container of the snippet became not editable (with branding)

Inside a demo page call the snippet with: ``<div data-oe-call="website.name_of_the_template"/>``
The snippets are loaded in one time by js and the page stay editable.

More
++++

- Use the class ``o_not_editable`` to prevent the edition from an area.

```

## File: models\assets.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import models


class Assets(models.AbstractModel):
    _inherit = 'web_editor.assets'

    def make_scss_customization(self, url, values):
        """
        Makes a scss customization of the given file. That file must
        contain a scss map including a line comment containing the word 'hook',
        to indicate the location where to write the new key,value pairs.

        Params:
            url (str):
                the URL of the scss file to customize (supposed to be a variable
                file which will appear in the assets_common bundle)

            values (dict):
                key,value mapping to integrate in the file's map (containing the
                word hook). If a key is already in the file's map, its value is
                overridden.
        """
        custom_url = self.make_custom_asset_file_url(url, 'web.assets_common')
        updatedFileContent = self.get_asset_content(custom_url) or self.get_asset_content(url)
        updatedFileContent = updatedFileContent.decode('utf-8')
        for name, value in values.items():
            pattern = "'%s': %%s,\n" % name
            regex = re.compile(pattern % ".+")
            replacement = pattern % value
            if regex.search(updatedFileContent):
                updatedFileContent = re.sub(regex, replacement, updatedFileContent)
            else:
                updatedFileContent = re.sub(r'( *)(.*hook.*)', r'\1%s\1\2' % replacement, updatedFileContent)

        # Bundle is 'assets_common' as this route is only meant to update
        # variables scss files
        self.save_asset(url, 'web.assets_common', updatedFileContent, 'scss')

    def _get_custom_attachment(self, custom_url, op='='):
        """
        See web_editor.Assets._get_custom_attachment
        Extend to only return the attachments related to the current website.
        """
        if self.env.user.has_group('website.group_website_designer'):
            self = self.sudo()
        website = self.env['website'].get_current_website()
        res = super(Assets, self)._get_custom_attachment(custom_url, op=op)
        return res.with_context(website_id=website.id).filtered(lambda x: not x.website_id or x.website_id == website)

    def _get_custom_view(self, custom_url, op='='):
        """
        See web_editor.Assets._get_custom_view
        Extend to only return the views related to the current website.
        """
        website = self.env['website'].get_current_website()
        res = super(Assets, self)._get_custom_view(custom_url, op=op)
        return res.with_context(website_id=website.id).filter_duplicate()

    def _save_asset_attachment_hook(self):
        """
        See web_editor.Assets._save_asset_attachment_hook
        Extend to add website ID at attachment creation.
        """
        res = super(Assets, self)._save_asset_attachment_hook()

        website = self.env['website'].get_current_website()
        if website:
            res['website_id'] = website.id
        return res

    def _save_asset_view_hook(self):
        """
        See web_editor.Assets._save_asset_view_hook
        Extend to add website ID at view creation.
        """
        res = super(Assets, self)._save_asset_view_hook()

        website = self.env['website'].get_current_website()
        if website:
            res['website_id'] = website.id
        return res

```

## File: models\ir_actions.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from werkzeug import urls

from odoo import api, fields, models
from odoo.http import request


class ServerAction(models.Model):
    """ Add website option in server actions. """

    _name = 'ir.actions.server'
    _inherit = 'ir.actions.server'

    xml_id = fields.Char('External ID', compute='_compute_xml_id', help="ID of the action if defined in a XML file")
    website_path = fields.Char('Website Path')
    website_url = fields.Char('Website Url', compute='_get_website_url', help='The full URL to access the server action through the website.')
    website_published = fields.Boolean('Available on the Website', copy=False,
                                       help='A code server action can be executed from the website, using a dedicated '
                                            'controller. The address is <base>/website/action/<website_path>. '
                                            'Set this field as True to allow users to run this action. If it '
                                            'is set to False the action cannot be run through the website.')

    def _compute_xml_id(self):
        res = self.get_external_id()
        for action in self:
            action.xml_id = res.get(action.id)

    def _compute_website_url(self, website_path, xml_id):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        link = website_path or xml_id or (self.id and '%d' % self.id) or ''
        if base_url and link:
            path = '%s/%s' % ('/website/action', link)
            return urls.url_join(base_url, path)
        return ''

    @api.depends('state', 'website_published', 'website_path', 'xml_id')
    def _get_website_url(self):
        for action in self:
            if action.state == 'code' and action.website_published:
                action.website_url = action._compute_website_url(action.website_path, action.xml_id)
            else:
                action.website_url = False

    @api.model
    def _get_eval_context(self, action):
        """ Override to add the request object in eval_context. """
        eval_context = super(ServerAction, self)._get_eval_context(action)
        if action.state == 'code':
            eval_context['request'] = request
        return eval_context

    @api.model
    def run_action_code_multi(self, action, eval_context=None):
        """ Override to allow returning response the same way action is already
            returned by the basic server action behavior. Note that response has
            priority over action, avoid using both.
        """
        res = super(ServerAction, self).run_action_code_multi(action, eval_context)
        return eval_context.get('response', res)

```

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import fields, models, api, tools
from odoo.exceptions import UserError
from odoo.tools.translate import _
_logger = logging.getLogger(__name__)


class Attachment(models.Model):

    _inherit = "ir.attachment"

    # related for backward compatibility with saas-6
    website_url = fields.Char(string="Website URL", related='local_url', deprecated=True, readonly=False)
    key = fields.Char(help='Technical field used to resolve multiple attachments in a multi-website environment.')
    website_id = fields.Many2one('website')

    @api.model
    def create(self, vals):
        website = self.env['website'].get_current_website(fallback=False)
        if website and 'website_id' not in vals and 'not_force_website_id' not in self.env.context:
            vals['website_id'] = website.id
        return super(Attachment, self).create(vals)

    @api.model
    def get_serving_groups(self):
        return super(Attachment, self).get_serving_groups() + ['website.group_website_designer']

    @api.model
    def get_serve_attachment(self, url, extra_domain=None, extra_fields=None, order=None):
        website = self.env['website'].get_current_website()
        extra_domain = (extra_domain or []) + website.website_domain()
        order = ('website_id, %s' % order) if order else 'website_id'
        return super(Attachment, self).get_serve_attachment(url, extra_domain, extra_fields, order)

    @api.model
    def get_attachment_by_key(self, key, extra_domain=None, order=None):
        website = self.env['website'].get_current_website()
        extra_domain = (extra_domain or []) + website.website_domain()
        order = ('website_id, %s' % order) if order else 'website_id'
        return super(Attachment, self).get_attachment_by_key(key, extra_domain, order)

    def init(self):
        res = super(Attachment, self).init()
        # ir_http._xmlid_to_obj is using this index for multi-website
        tools.create_index(self._cr, 'ir_attachment_key_website_idx', self._table, ['key', 'website_id'])
        return res

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
from lxml import etree
import os
import unittest

import pytz
import werkzeug
import werkzeug.routing
import werkzeug.utils

from functools import partial

import odoo
from odoo import api, models
from odoo import registry, SUPERUSER_ID
from odoo.http import request
from odoo.tools.safe_eval import safe_eval
from odoo.osv.expression import FALSE_DOMAIN

from odoo.addons.http_routing.models.ir_http import ModelConverter, _guess_mimetype
from odoo.addons.portal.controllers.portal import _build_url_w_params

logger = logging.getLogger(__name__)


def sitemap_qs2dom(qs, route, field='name'):
    """ Convert a query_string (can contains a path) to a domain"""
    dom = []
    if qs and qs.lower() not in route:
        needles = qs.strip('/').split('/')
        # needles will be altered and keep only element which one is not in route
        # diff(from=['shop', 'product'], to=['shop', 'product', 'product']) => to=['product']
        unittest.util.unorderable_list_difference(route.strip('/').split('/'), needles)
        if len(needles) == 1:
            dom = [(field, 'ilike', needles[0])]
        else:
            dom = FALSE_DOMAIN
    return dom


def get_request_website():
    """ Return the website set on `request` if called in a frontend context
    (website=True on route).
    This method can typically be used to check if we are in the frontend.

    This method is easy to mock during python tests to simulate frontend
    context, rather than mocking every method accessing request.website.

    Don't import directly the method or it won't be mocked during tests, do:
    ```
    from odoo.addons.website.models import ir_http
    my_var = ir_http.get_request_website()
    ```
    """
    return request and getattr(request, 'website', False) or False


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def routing_map(cls, key=None):
        key = key or (request and request.website_routing)
        return super(Http, cls).routing_map(key=key)

    @classmethod
    def clear_caches(cls):
        super(Http, cls)._clear_routing_map()
        return super(Http, cls).clear_caches()

    @classmethod
    def _slug_matching(cls, adapter, endpoint, **kw):
        for arg in kw:
            if isinstance(kw[arg], models.BaseModel):
                kw[arg] = kw[arg].with_user(request.uid)
        qs = request.httprequest.query_string.decode('utf-8')
        return adapter.build(endpoint, kw) + (qs and '?%s' % qs or '')

    @classmethod
    def _match(cls, path_info, key=None):
        key = key or (request and request.website_routing)
        return super(Http, cls)._match(path_info, key=key)

    @classmethod
    def _generate_routing_rules(cls, modules, converters):
        website_id = request.website_routing
        logger.debug("_generate_routing_rules for website: %s", website_id)
        domain = [('redirect_type', 'in', ('308', '404')), '|', ('website_id', '=', False), ('website_id', '=', website_id)]

        rewrites = dict([(x.url_from, x) for x in request.env['website.rewrite'].sudo().search(domain)])
        cls._rewrite_len[website_id] = len(rewrites)

        for url, endpoint, routing in super(Http, cls)._generate_routing_rules(modules, converters):
            routing = dict(routing)
            if url in rewrites:
                rewrite = rewrites[url]
                url_to = rewrite.url_to
                if rewrite.redirect_type == '308':
                    logger.debug('Add rule %s for %s' % (url_to, website_id))
                    yield url_to, endpoint, routing  # yield new url

                    if url != url_to:
                        logger.debug('Redirect from %s to %s for website %s' % (url, url_to, website_id))
                        _slug_matching = partial(cls._slug_matching, endpoint=endpoint)
                        routing['redirect_to'] = _slug_matching
                        yield url, endpoint, routing  # yield original redirected to new url
                elif rewrite.redirect_type == '404':
                    logger.debug('Return 404 for %s for website %s' % (url, website_id))
                    continue
            else:
                yield url, endpoint, routing

    @classmethod
    def _get_converters(cls):
        """ Get the converters list for custom url pattern werkzeug need to
            match Rule. This override adds the website ones.
        """
        return dict(
            super(Http, cls)._get_converters(),
            model=ModelConverter,
        )

    @classmethod
    def _auth_method_public(cls):
        """ If no user logged, set the public user of current website, or default
            public user as request uid.
            After this method `request.env` can be called, since the `request.uid` is
            set. The `env` lazy property of `request` will be correct.
        """
        if not request.session.uid:
            env = api.Environment(request.cr, SUPERUSER_ID, request.context)
            website = env['website'].get_current_website()
            if website and website.user_id:
                request.uid = website.user_id.id
        if not request.uid:
            super(Http, cls)._auth_method_public()

    @classmethod
    def _register_website_track(cls, response):
        if getattr(response, 'status_code', 0) != 200 or not hasattr(response, 'qcontext'):
            return False
        main_object = response.qcontext.get('main_object')
        website_page = getattr(main_object, '_name', False) == 'website.page' and main_object
        template = response.qcontext.get('response_template')
        view = template and request.env['website'].get_template(template)
        if view and view.track:
            request.env['website.visitor']._handle_webpage_dispatch(response, website_page)

    @classmethod
    def _dispatch(cls):
        """
        In case of rerouting for translate (e.g. when visiting odoo.com/fr_BE/),
        _dispatch calls reroute() that returns _dispatch with altered request properties.
        The second _dispatch will continue until end of process. When second _dispatch is finished, the first _dispatch
        call receive the new altered request and continue.
        At the end, 2 calls of _dispatch (and this override) are made with exact same request properties, instead of one.
        As the response has not been sent back to the client, the visitor cookie does not exist yet when second _dispatch call
        is treated in _handle_webpage_dispatch, leading to create 2 visitors with exact same properties.
        To avoid this, we check if, !!! before calling super !!!, we are in a rerouting request. If not, it means that we are
        handling the original request, in which we should create the visitor. We ignore every other rerouting requests.
        """
        is_rerouting = hasattr(request, 'routing_iteration')

        if request.session.db:
            reg = registry(request.session.db)
            with reg.cursor() as cr:
                env = api.Environment(cr, SUPERUSER_ID, {})
                request.website_routing = env['website'].get_current_website().id

        response = super(Http, cls)._dispatch()

        if not is_rerouting:
            cls._register_website_track(response)
        return response

    @classmethod
    def _add_dispatch_parameters(cls, func):

        # DEPRECATED for /website/force/<website_id> - remove me in master~saas-14.4
        # Force website with query string paramater, typically set from website selector in frontend navbar and inside tests
        force_website_id = request.httprequest.args.get('fw')
        if (force_website_id and request.session.get('force_website_id') != force_website_id
                and request.env.user.has_group('website.group_multi_website')
                and request.env.user.has_group('website.group_website_publisher')):
            request.env['website']._force_website(request.httprequest.args.get('fw'))

        context = {}
        if not request.context.get('tz'):
            context['tz'] = request.session.get('geoip', {}).get('time_zone')
            try:
                pytz.timezone(context['tz'] or '')
            except pytz.UnknownTimeZoneError:
                context.pop('tz')

        request.website = request.env['website'].get_current_website()  # can use `request.env` since auth methods are called
        context['website_id'] = request.website.id
        # This is mainly to avoid access errors in website controllers where there is no
        # context (eg: /shop), and it's not going to propagate to the global context of the tab
        # If the company of the website is not in the allowed companies of the user, set the main
        # company of the user.
        if request.website.company_id in request.env.user.company_ids:
            context['allowed_company_ids'] = request.website.company_id.ids
        else:
            context['allowed_company_ids'] = request.env.user.company_id.ids

        # modify bound context
        request.context = dict(request.context, **context)

        super(Http, cls)._add_dispatch_parameters(func)

        if request.routing_iteration == 1:
            request.website = request.website.with_context(request.context)

    @classmethod
    def _get_frontend_langs(cls):
        if get_request_website():
            return [code for code, _, _ in request.env['res.lang'].get_available()]
        else:
            return super()._get_frontend_langs()

    @classmethod
    def _get_default_lang(cls):
        if getattr(request, 'website', False):
            return request.website.default_lang_id
        return super(Http, cls)._get_default_lang()

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(Http, cls)._get_translation_frontend_modules_name()
        installed = request.registry._init_modules | set(odoo.conf.server_wide_modules)
        return mods + [mod for mod in installed if mod.startswith('website')]

    @classmethod
    def _serve_page(cls):
        req_page = request.httprequest.path
        page_domain = [('url', '=', req_page)] + request.website.website_domain()

        published_domain = page_domain
        # specific page first
        page = request.env['website.page'].sudo().search(published_domain, order='website_id asc', limit=1)
        if page and (request.website.is_publisher() or page.is_visible):
            _, ext = os.path.splitext(req_page)
            return request.render(page.get_view_identifier(), {
                'deletable': True,
                'main_object': page,
            }, mimetype=_guess_mimetype(ext))
        return False

    @classmethod
    def _serve_redirect(cls):
        req_page = request.httprequest.path
        domain = [
            ('redirect_type', 'in', ('301', '302')),
            ('url_from', '=', req_page)
        ]
        domain += request.website.website_domain()
        return request.env['website.rewrite'].sudo().search(domain, limit=1)

    @classmethod
    def _serve_fallback(cls, exception):
        # serve attachment before
        parent = super(Http, cls)._serve_fallback(exception)
        if parent:  # attachment
            return parent
        if not request.is_frontend:
            return False
        website_page = cls._serve_page()
        if website_page:
            return website_page

        redirect = cls._serve_redirect()
        if redirect:
            return request.redirect(_build_url_w_params(redirect.url_to, request.params), code=redirect.redirect_type)

        return False

    @classmethod
    def _get_exception_code_values(cls, exception):
        code, values = super(Http, cls)._get_exception_code_values(exception)
        if request.website.is_publisher() and isinstance(exception, werkzeug.exceptions.NotFound):
            values['path'] = request.httprequest.path[1:]
            values['force_template'] = 'website.page_404'
        return (code, values)

    @classmethod
    def _get_values_500_error(cls, env, values, exception):
        View = env["ir.ui.view"]
        values = super(Http, cls)._get_values_500_error(env, values, exception)
        if 'qweb_exception' in values:
            try:
                # exception.name might be int, string
                exception_template = int(exception.name)
            except:
                exception_template = exception.name
            view = View._view_obj(exception_template)
            if exception.html and exception.html in view.arch:
                values['view'] = view
            else:
                # There might be 2 cases where the exception code can't be found
                # in the view, either the error is in a child view or the code
                # contains branding (<div t-att-data="request.browse('ok')"/>).
                et = etree.fromstring(view.with_context(inherit_branding=False).read_combined(['arch'])['arch'])
                node = et.xpath(exception.path)
                line = node is not None and etree.tostring(node[0], encoding='unicode')
                if line:
                    values['view'] = View._views_get(exception_template).filtered(
                        lambda v: line in v.arch
                    )
                    values['view'] = values['view'] and values['view'][0]
        # Needed to show reset template on translated pages (`_prepare_qcontext` will set it for main lang)
        values['editable'] = request.uid and request.website.is_publisher()
        return values

    @classmethod
    def _get_error_html(cls, env, code, values):
        if values.get('force_template'):
            return env['ir.ui.view'].render_template(values['force_template'], values)
        return super(Http, cls)._get_error_html(env, code, values)

    def binary_content(self, xmlid=None, model='ir.attachment', id=None, field='datas',
                       unique=False, filename=None, filename_field='name', download=False,
                       mimetype=None, default_mimetype='application/octet-stream',
                       access_token=None):
        obj = None
        if xmlid:
            obj = self._xmlid_to_obj(self.env, xmlid)
        elif id and model in self.env:
            obj = self.env[model].browse(int(id))
        if obj and 'website_published' in obj._fields:
            if self.env[obj._name].sudo().search([('id', '=', obj.id), ('website_published', '=', True)]):
                self = self.sudo()
        return super(Http, self).binary_content(
            xmlid=xmlid, model=model, id=id, field=field, unique=unique, filename=filename,
            filename_field=filename_field, download=download, mimetype=mimetype,
            default_mimetype=default_mimetype, access_token=access_token)

    @classmethod
    def _xmlid_to_obj(cls, env, xmlid):
        website_id = env['website'].get_current_website()
        if website_id and website_id.theme_id:
            domain = [('key', '=', xmlid), ('website_id', '=', website_id.id)]
            Attachment = env['ir.attachment']
            if request.env.user.share:
                domain.append(('public', '=', True))
                Attachment = Attachment.sudo()
            obj = Attachment.search(domain)
            if obj:
                return obj[0]

        return super(Http, cls)._xmlid_to_obj(env, xmlid)

    @api.model
    def get_frontend_session_info(self):
        session_info = super(Http, self).get_frontend_session_info()
        session_info.update({
            'is_website_user': request.env.user.id == request.website.user_id.id,
        })
        if request.env.user.has_group('website.group_website_publisher'):
            session_info.update({
                'website_id': request.website.id,
                'website_company_id': request.website.company_id.id,
            })
        return session_info


class ModelConverter(ModelConverter):

    def generate(self, uid, dom=None, args=None):
        Model = request.env[self.model].with_user(uid)
        # Allow to current_website_id directly in route domain
        args.update(current_website_id=request.env['website'].get_current_website().id)
        domain = safe_eval(self.domain, (args or {}).copy())
        if dom:
            domain += dom
        for record in Model.search_read(domain, ['display_name']):
            yield {'loc': (record['id'], record['display_name'])}

```

## File: models\ir_module_module.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class Module(models.Model):
    _inherit = 'ir.module.module'

    def _check(self):
        super()._check()
        View = self.env['ir.ui.view']
        website_views_to_adapt = getattr(self.pool, 'website_views_to_adapt', [])
        if website_views_to_adapt:
            for view_replay in website_views_to_adapt:
                cow_view = View.browse(view_replay[0])
                View._load_records_write_on_cow(cow_view, view_replay[1], view_replay[2])
            self.pool.website_views_to_adapt.clear()

```

## File: models\ir_qweb.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re
from collections import OrderedDict

from odoo import models
from odoo.http import request
from odoo.addons.base.models.assetsbundle import AssetsBundle
from odoo.addons.http_routing.models.ir_http import url_for
from odoo.osv import expression
from odoo.addons.website.models import ir_http
from odoo.tools import html_escape as escape

re_background_image = re.compile(r"(background-image\s*:\s*url\(\s*['\"]?\s*)([^)'\"]+)")


class AssetsBundleMultiWebsite(AssetsBundle):
    def _get_asset_url_values(self, id, unique, extra, name, sep, type):
        website_id = self.env.context.get('website_id')
        website_id_path = website_id and ('%s/' % website_id) or ''
        extra = website_id_path + extra
        res = super(AssetsBundleMultiWebsite, self)._get_asset_url_values(id, unique, extra, name, sep, type)
        return res

    def _get_assets_domain_for_already_processed_css(self, assets):
        res = super(AssetsBundleMultiWebsite, self)._get_assets_domain_for_already_processed_css(assets)
        current_website = self.env['website'].get_current_website(fallback=False)
        res = expression.AND([res, current_website.website_domain()])
        return res

class QWeb(models.AbstractModel):
    """ QWeb object for rendering stuff in the website context """

    _inherit = 'ir.qweb'

    URL_ATTRS = {
        'form':   'action',
        'a':      'href',
        'link':   'href',
        'script': 'src',
        'img':    'src',
    }

    def get_asset_bundle(self, xmlid, files, env=None):
        return AssetsBundleMultiWebsite(xmlid, files, env=env)

    def _post_processing_att(self, tagName, atts, options):
        if atts.get('data-no-post-process'):
            return atts

        atts = super(QWeb, self)._post_processing_att(tagName, atts, options)

        if options.get('inherit_branding') or options.get('rendering_bundle') or \
           options.get('edit_translations') or options.get('debug') or (request and request.session.debug):
            return atts

        website = ir_http.get_request_website()
        if not website and options.get('website_id'):
            website = self.env['website'].browse(options['website_id'])

        if not website:
            return atts

        name = self.URL_ATTRS.get(tagName)
        if request and name and name in atts:
            atts[name] = url_for(atts[name])

        if not website.cdn_activated:
            return atts

        data_name = f'data-{name}'
        if name and (name in atts or data_name in atts):
            atts = OrderedDict(atts)
            if name in atts:
                atts[name] = website.get_cdn_url(atts[name])
            if data_name in atts:
                atts[data_name] = website.get_cdn_url(atts[data_name])
        if isinstance(atts.get('style'), str) and 'background-image' in atts['style']:
            atts = OrderedDict(atts)
            atts['style'] = re_background_image.sub(lambda m: '%s%s' % (m.group(1), website.get_cdn_url(m.group(2))), atts['style'])

        return atts

```

## File: models\ir_qweb_fields.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Contact(models.AbstractModel):
    _inherit = 'ir.qweb.field.contact'

    @api.model
    def get_available_options(self):
        options = super(Contact, self).get_available_options()
        options.update(
            website_description=dict(type='boolean', string=_('Display the website description')),
            UserBio=dict(type='boolean', string=_('Display the biography')),
            badges=dict(type='boolean', string=_('Display the badges'))
        )
        return options

```

## File: models\ir_rule.py

```python
# coding: utf-8
from odoo import api, models
from odoo.addons.website.models import ir_http


class IrRule(models.Model):
    _inherit = 'ir.rule'

    @api.model
    def _eval_context(self):
        res = super(IrRule, self)._eval_context()

        # We need is_frontend to avoid showing website's company items in backend
        # (that could be different than current company). We can't use
        # `get_current_website(falback=False)` as it could also return a website
        # in backend (if domain set & match)..
        is_frontend = ir_http.get_request_website()
        Website = self.env['website']
        res['website'] = is_frontend and Website.get_current_website() or Website
        return res

    def _compute_domain_keys(self):
        """ Return the list of context keys to use for caching ``_compute_domain``. """
        return super(IrRule, self)._compute_domain_keys() + ['website_id']

```

## File: models\ir_translation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class IrTranslation(models.Model):
    _inherit = "ir.translation"

    def _load_module_terms(self, modules, langs):
        """ Add missing website specific translation """
        res = super()._load_module_terms(modules, langs)

        if not langs or not modules:
            return res
        if self.env.context.get('overwrite'):
            conflict_clause = """
                   ON CONFLICT {}
                   DO UPDATE SET (name, lang, res_id, src, type, value, module, state, comments) =
                       (EXCLUDED.name, EXCLUDED.lang, EXCLUDED.res_id, EXCLUDED.src, EXCLUDED.type,
                        EXCLUDED.value, EXCLUDED.module, EXCLUDED.state, EXCLUDED.comments)
                WHERE EXCLUDED.value IS NOT NULL AND EXCLUDED.value != ''
            """;
        else:
            conflict_clause = " ON CONFLICT DO NOTHING"

        # Add specific view translations
        self.env.cr.execute("""
            INSERT INTO ir_translation(name, lang, res_id, src, type, value, module, state, comments)
            SELECT DISTINCT ON (specific.id, t.lang, md5(src)) t.name, t.lang, specific.id, t.src, t.type, t.value, t.module, t.state, t.comments
              FROM ir_translation t
             INNER JOIN ir_ui_view generic
                ON t.type = 'model_terms' AND t.name = 'ir.ui.view,arch_db' AND t.res_id = generic.id
             INNER JOIN ir_ui_view specific
                ON generic.key = specific.key
             WHERE t.lang IN %s and t.module IN %s
               AND generic.website_id IS NULL AND generic.type = 'qweb'
               AND specific.website_id IS NOT NULL""" + conflict_clause.format(
                   "(type, name, lang, res_id, md5(src))"
        ), (tuple(langs), tuple(modules)))

        default_menu = self.env.ref('website.main_menu', raise_if_not_found=False)
        if not default_menu:
            return res

        # Add specific menu translations
        self.env.cr.execute("""
            INSERT INTO ir_translation(name, lang, res_id, src, type, value, module, state, comments)
            SELECT DISTINCT ON (s_menu.id, t.lang) t.name, t.lang, s_menu.id, t.src, t.type, t.value, t.module, t.state, t.comments
              FROM ir_translation t
             INNER JOIN website_menu o_menu
                ON t.type = 'model' AND t.name = 'website.menu,name' AND t.res_id = o_menu.id
             INNER JOIN website_menu s_menu
                ON o_menu.name = s_menu.name AND o_menu.url = s_menu.url
             INNER JOIN website_menu root_menu
                ON s_menu.parent_id = root_menu.id AND root_menu.parent_id IS NULL
             WHERE t.lang IN %s and t.module IN %s
               AND o_menu.website_id IS NULL AND o_menu.parent_id = %s
               AND s_menu.website_id IS NOT NULL""" + conflict_clause.format(
                   "(type, lang, name, res_id) WHERE type = 'model'"
        ), (tuple(langs), tuple(modules), default_menu.id))

        return res

```

## File: models\ir_ui_view.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import uuid
from itertools import groupby

from odoo import api, fields, models, _
from odoo import tools
from odoo.addons.http_routing.models.ir_http import url_for
from odoo.osv import expression
from odoo.http import request

_logger = logging.getLogger(__name__)


class View(models.Model):

    _name = "ir.ui.view"
    _inherit = ["ir.ui.view", "website.seo.metadata"]

    customize_show = fields.Boolean("Show As Optional Inherit", default=False)
    website_id = fields.Many2one('website', ondelete='cascade', string="Website")
    page_ids = fields.One2many('website.page', 'view_id')
    first_page_id = fields.Many2one('website.page', string='Website Page', help='First page linked to this view', compute='_compute_first_page_id')
    track = fields.Boolean(string='Track', default=False, help="Allow to specify for one page of the website to be trackable or not")

    def _compute_first_page_id(self):
        for view in self:
            view.first_page_id = self.env['website.page'].search([('view_id', '=', view.id)], limit=1)

    def name_get(self):
        if not self._context.get('display_website') and not self.env.user.has_group('website.group_multi_website'):
            return super(View, self).name_get()

        res = []
        for view in self:
            view_name = view.name
            if view.website_id:
                view_name += ' [%s]' % view.website_id.name
            res.append((view.id, view_name))
        return res

    def write(self, vals):
        '''COW for ir.ui.view. This way editing websites does not impact other
        websites. Also this way newly created websites will only
        contain the default views.
        '''
        current_website_id = self.env.context.get('website_id')
        if not current_website_id or self.env.context.get('no_cow'):
            return super(View, self).write(vals)

        # We need to consider inactive views when handling multi-website cow
        # feature (to copy inactive children views, to search for specific
        # views, ...)
        # Website-specific views need to be updated first because they might
        # be relocated to new ids by the cow if they are involved in the
        # inheritance tree.
        for view in self.with_context(active_test=False).sorted(key='website_id', reverse=True):
            # Make sure views which are written in a website context receive
            # a value for their 'key' field
            if not view.key and not vals.get('key'):
                view.with_context(no_cow=True).key = 'website.key_%s' % str(uuid.uuid4())[:6]

            # No need of COW if the view is already specific
            if view.website_id:
                super(View, view).write(vals)
                continue

            # Ensure the cache of the pages stay consistent when doing COW.
            # This is necessary when writing view fields from a page record
            # because the generic page will put the given values on its cache
            # but in reality the values were only meant to go on the specific
            # page. Invalidate all fields and not only those in vals because
            # other fields could have been changed implicitly too.
            pages = view.page_ids
            pages.flush(records=pages)
            pages.invalidate_cache(ids=pages.ids)

            # If already a specific view for this generic view, write on it
            website_specific_view = view.search([
                ('key', '=', view.key),
                ('website_id', '=', current_website_id)
            ], limit=1)
            if website_specific_view:
                super(View, website_specific_view).write(vals)
                continue

            # Set key to avoid copy() to generate an unique key as we want the
            # specific view to have the same key
            copy_vals = {'website_id': current_website_id, 'key': view.key}
            # Copy with the 'inherit_id' field value that will be written to
            # ensure the copied view's validation works
            if vals.get('inherit_id'):
                copy_vals['inherit_id'] = vals['inherit_id']
            website_specific_view = view.copy(copy_vals)

            view._create_website_specific_pages_for_view(website_specific_view,
                                                         view.env['website'].browse(current_website_id))

            for inherit_child in view.inherit_children_ids.filter_duplicate().sorted(key=lambda v: (v.priority, v.id)):
                if inherit_child.website_id.id == current_website_id:
                    # In the case the child was already specific to the current
                    # website, we cannot just reattach it to the new specific
                    # parent: we have to copy it there and remove it from the
                    # original tree. Indeed, the order of children 'id' fields
                    # must remain the same so that the inheritance is applied
                    # in the same order in the copied tree.
                    child = inherit_child.copy({'inherit_id': website_specific_view.id, 'key': inherit_child.key})
                    inherit_child.inherit_children_ids.write({'inherit_id': child.id})
                    inherit_child.unlink()
                else:
                    # Trigger COW on inheriting views
                    inherit_child.write({'inherit_id': website_specific_view.id})

            super(View, website_specific_view).write(vals)

        return True

    def _load_records_write_on_cow(self, cow_view, inherit_id, values):
        inherit_id = self.search([
            ('key', '=', self.browse(inherit_id).key),
            ('website_id', 'in', (False, cow_view.website_id.id)),
        ], order='website_id', limit=1).id
        values['inherit_id'] = inherit_id
        cow_view.with_context(no_cow=True).write(values)

    def _create_all_specific_views(self, processed_modules):
        """ When creating a generic child view, we should
            also create that view under specific view trees (COW'd).
            Top level view (no inherit_id) do not need that behavior as they
            will be shared between websites since there is no specific yet.
        """
        # Only for the modules being processed
        regex = '^(%s)[.]' % '|'.join(processed_modules)
        # Retrieve the views through a SQl query to avoid ORM queries inside of for loop
        # Retrieves all the views that are missing their specific counterpart with all the
        # specific view parent id and their website id in one query
        query = """
            SELECT generic.id, ARRAY[array_agg(spec_parent.id), array_agg(spec_parent.website_id)]
              FROM ir_ui_view generic
        INNER JOIN ir_ui_view generic_parent ON generic_parent.id = generic.inherit_id
        INNER JOIN ir_ui_view spec_parent ON spec_parent.key = generic_parent.key
         LEFT JOIN ir_ui_view specific ON specific.key = generic.key AND specific.website_id = spec_parent.website_id
             WHERE generic.type='qweb'
               AND generic.website_id IS NULL
               AND generic.key ~ %s
               AND spec_parent.website_id IS NOT NULL
               AND specific.id IS NULL
          GROUP BY generic.id
        """
        self.env.cr.execute(query, (regex, ))
        result = dict(self.env.cr.fetchall())

        for record in self.browse(result.keys()):
            specific_parent_view_ids, website_ids = result[record.id]
            for specific_parent_view_id, website_id in zip(specific_parent_view_ids, website_ids):
                record.with_context(website_id=website_id).write({
                    'inherit_id': specific_parent_view_id,
                })
        super(View, self)._create_all_specific_views(processed_modules)

    def unlink(self):
        '''This implements COU (copy-on-unlink). When deleting a generic page
        website-specific pages will be created so only the current
        website is affected.
        '''
        current_website_id = self._context.get('website_id')

        if current_website_id and not self._context.get('no_cow'):
            for view in self.filtered(lambda view: not view.website_id):
                for website in self.env['website'].search([('id', '!=', current_website_id)]):
                    # reuse the COW mechanism to create
                    # website-specific copies, it will take
                    # care of creating pages and menus.
                    view.with_context(website_id=website.id).write({'name': view.name})

        specific_views = self.env['ir.ui.view']
        if self and self.pool._init:
            for view in self.filtered(lambda view: not view.website_id):
                specific_views += view._get_specific_views()

        result = super(View, self + specific_views).unlink()
        self.clear_caches()
        return result

    def _create_website_specific_pages_for_view(self, new_view, website):
        for page in self.page_ids:
            # create new pages for this view
            page.copy({
                'view_id': new_view.id,
                'is_published': page.is_published,
            })

    @api.model
    def get_related_views(self, key, bundles=False):
        '''Make this only return most specific views for website.'''
        # get_related_views can be called through website=False routes
        # (e.g. /web_editor/get_assets_editor_resources), so website
        # dispatch_parameters may not be added. Manually set
        # website_id. (It will then always fallback on a website, this
        # method should never be called in a generic context, even for
        # tests)
        self = self.with_context(website_id=self.env['website'].get_current_website().id)
        return super(View, self).get_related_views(key, bundles=bundles)

    def filter_duplicate(self):
        """ Filter current recordset only keeping the most suitable view per distinct key.
            Every non-accessible view will be removed from the set:
              * In non website context, every view with a website will be removed
              * In a website context, every view from another website
        """
        current_website_id = self._context.get('website_id')
        most_specific_views = self.env['ir.ui.view']
        if not current_website_id:
            return self.filtered(lambda view: not view.website_id)

        for view in self:
            # specific view: add it if it's for the current website and ignore
            # it if it's for another website
            if view.website_id and view.website_id.id == current_website_id:
                most_specific_views |= view
            # generic view: add it only if, for the current website, there is no
            # specific view for this view (based on the same `key` attribute)
            elif not view.website_id and not any(view.key == view2.key and view2.website_id and view2.website_id.id == current_website_id for view2 in self):
                most_specific_views |= view

        return most_specific_views

    @api.model
    def _view_get_inherited_children(self, view):
        extensions = super(View, self)._view_get_inherited_children(view)
        return extensions.filter_duplicate()

    @api.model
    def _view_obj(self, view_id):
        ''' Given an xml_id or a view_id, return the corresponding view record.
            In case of website context, return the most specific one.
            :param view_id: either a string xml_id or an integer view_id
            :return: The view record or empty recordset
        '''
        if isinstance(view_id, str) or isinstance(view_id, int):
            return self.env['website'].viewref(view_id)
        else:
            # It can already be a view object when called by '_views_get()' that is calling '_view_obj'
            # for it's inherit_children_ids, passing them directly as object record. (Note that it might
            # be a view_id from another website but it will be filtered in 'get_related_views()')
            return view_id if view_id._name == 'ir.ui.view' else self.env['ir.ui.view']

    @api.model
    def _get_inheriting_views_arch_website(self, view_id):
        return self.env['website'].browse(self._context.get('website_id'))

    @api.model
    def _get_inheriting_views_arch_domain(self, view_id, model):
        domain = super(View, self)._get_inheriting_views_arch_domain(view_id, model)
        current_website = self._get_inheriting_views_arch_website(view_id)
        website_views_domain = current_website.website_domain()
        # when rendering for the website we have to include inactive views
        # we will prefer inactive website-specific views over active generic ones
        if current_website:
            domain = [leaf for leaf in domain if 'active' not in leaf]

        return expression.AND([website_views_domain, domain])

    @api.model
    def get_inheriting_views_arch(self, view_id, model):
        if not self._context.get('website_id'):
            return super(View, self).get_inheriting_views_arch(view_id, model)

        get_inheriting_self = self.with_context(active_test=False)
        if self.pool._init and not self._context.get('load_all_views'):
            view = self.browse(view_id)
            if view.website_id:
                original_view = view._get_original_view()
                original_keys = self.with_context(website_id=False)._get_inheriting_views(original_view.id, model).mapped('key')
                specific_views = self.search([('key', 'in', original_keys), ('website_id', '=', self._context.get('website_id'))])
                check_view_ids = list(self._context.get('check_view_ids') or ()) + specific_views.ids
                get_inheriting_self = self.with_context(check_view_ids=check_view_ids)
        inheriting_views = super(View, get_inheriting_self).get_inheriting_views_arch(view_id, model)

        # prefer inactive website-specific views over active generic ones
        inheriting_views = self.browse([view[1] for view in inheriting_views]).filter_duplicate().filtered('active')

        return [(view.arch, view.id) for view in inheriting_views]

    @api.model
    @tools.ormcache_context('self.env.uid', 'self.env.su', 'xml_id', keys=('website_id',))
    def get_view_id(self, xml_id):
        """If a website_id is in the context and the given xml_id is not an int
        then try to get the id of the specific view for that website, but
        fallback to the id of the generic view if there is no specific.

        If no website_id is in the context, it might randomly return the generic
        or the specific view, so it's probably not recommanded to use this
        method. `viewref` is probably more suitable.

        Archived views are ignored (unless the active_test context is set, but
        then the ormcache_context will not work as expected).
        """
        if 'website_id' in self._context and not isinstance(xml_id, int):
            current_website = self.env['website'].browse(self._context.get('website_id'))
            domain = ['&', ('key', '=', xml_id)] + current_website.website_domain()

            view = self.search(domain, order='website_id', limit=1)
            if not view:
                _logger.warning("Could not find view object with xml_id '%s'", xml_id)
                raise ValueError('View %r in website %r not found' % (xml_id, self._context['website_id']))
            return view.id
        return super(View, self).get_view_id(xml_id)

    def _get_original_view(self):
        """Given a view, retrieve the original view it was COW'd from.
        The given view might already be the original one. In that case it will
        (and should) return itself.
        """
        self.ensure_one()
        domain = [('key', '=', self.key), ('model_data_id', '!=', None)]
        return self.with_context(active_test=False).search(domain, limit=1)  # Useless limit has multiple xmlid should not be possible

    def render(self, values=None, engine='ir.qweb', minimal_qcontext=False):
        """ Render the template. If website is enabled on request, then extend rendering context with website values. """
        new_context = dict(self._context)
        if request and getattr(request, 'is_frontend', False):

            editable = request.website.is_publisher()
            translatable = editable and self._context.get('lang') != request.website.default_lang_id.code
            editable = not translatable and editable

            # in edit mode ir.ui.view will tag nodes
            if not translatable and not self.env.context.get('rendering_bundle'):
                if editable:
                    new_context = dict(self._context, inherit_branding=True)
                elif request.env.user.has_group('website.group_website_publisher'):
                    new_context = dict(self._context, inherit_branding_auto=True)
            if values and 'main_object' in values:
                if request.env.user.has_group('website.group_website_publisher'):
                    func = getattr(values['main_object'], 'get_backend_menu_id', False)
                    values['backend_menu_id'] = func and func() or self.env.ref('website.menu_website_configuration').id

                # Fallback incase main_object dont't inherit 'website.seo.metadata'
                if not hasattr(values['main_object'], 'get_website_meta'):
                    values['main_object'].get_website_meta = lambda: {}

        if self._context != new_context:
            self = self.with_context(new_context)
        return super(View, self).render(values, engine=engine, minimal_qcontext=minimal_qcontext)

    @api.model
    def _prepare_qcontext(self):
        """ Returns the qcontext : rendering context with website specific value (required
            to render website layout template)
        """
        qcontext = super(View, self)._prepare_qcontext()

        if request and getattr(request, 'is_frontend', False):
            Website = self.env['website']
            editable = request.website.is_publisher()
            translatable = editable and self._context.get('lang') != request.env['ir.http']._get_default_lang().code
            editable = not translatable and editable

            cur = Website.get_current_website()
            if self.env.user.has_group('website.group_website_publisher') and self.env.user.has_group('website.group_multi_website'):
                qcontext['multi_website_websites_current'] = {'website_id': cur.id, 'name': cur.name, 'domain': cur._get_http_domain()}
                qcontext['multi_website_websites'] = [
                    {'website_id': website.id, 'name': website.name, 'domain': website._get_http_domain()}
                    for website in Website.search([]) if website != cur
                ]

                cur_company = self.env.company
                qcontext['multi_website_companies_current'] = {'company_id': cur_company.id, 'name': cur_company.name}
                qcontext['multi_website_companies'] = [
                    {'company_id': comp.id, 'name': comp.name}
                    for comp in self.env.user.company_ids if comp != cur_company
                ]

            qcontext.update(dict(
                self._context.copy(),
                main_object=self,
                website=request.website,
                url_for=url_for,
                res_company=request.website.company_id.sudo(),
                default_lang_code=request.env['ir.http']._get_default_lang().code,
                languages=request.env['res.lang'].get_available(),
                translatable=translatable,
                editable=editable,
                # retrocompatibility, remove me in master
                menu_data={'children': []} if request.website.is_user() else None,
            ))

        return qcontext

    @api.model
    def get_default_lang_code(self):
        website_id = self.env.context.get('website_id')
        if website_id:
            lang_code = self.env['website'].browse(website_id).default_lang_id.code
            return lang_code
        else:
            return super(View, self).get_default_lang_code()

    def redirect_to_page_manager(self):
        return {
            'type': 'ir.actions.act_url',
            'url': '/website/pages',
            'target': 'self',
        }

    def _read_template_keys(self):
        return super(View, self)._read_template_keys() + ['website_id']

    @api.model
    def _save_oe_structure_hook(self):
        res = super(View, self)._save_oe_structure_hook()
        res['website_id'] = self.env['website'].get_current_website().id
        return res

    @api.model
    def _set_noupdate(self):
        '''If website is installed, any call to `save` from the frontend will
        actually write on the specific view (or create it if not exist yet).
        In that case, we don't want to flag the generic view as noupdate.
        '''
        if not self._context.get('website_id'):
            super(View, self)._set_noupdate()

    def save(self, value, xpath=None):
        self.ensure_one()
        current_website = self.env['website'].get_current_website()
        # xpath condition is important to be sure we are editing a view and not
        # a field as in that case `self` might not exist (check commit message)
        if xpath and self.key and current_website:
            # The first time a generic view is edited, if multiple editable parts
            # were edited at the same time, multiple call to this method will be
            # done but the first one may create a website specific view. So if there
            # already is a website specific view, we need to divert the super to it.
            website_specific_view = self.env['ir.ui.view'].search([
                ('key', '=', self.key),
                ('website_id', '=', current_website.id)
            ], limit=1)
            if website_specific_view:
                self = website_specific_view
        super(View, self).save(value, xpath=xpath)

```

## File: models\mixins.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from werkzeug.urls import url_join

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import url_for
from odoo.http import request
from odoo.osv import expression
from odoo.exceptions import AccessError

logger = logging.getLogger(__name__)


class SeoMetadata(models.AbstractModel):

    _name = 'website.seo.metadata'
    _description = 'SEO metadata'

    is_seo_optimized = fields.Boolean("SEO optimized", compute='_compute_is_seo_optimized')
    website_meta_title = fields.Char("Website meta title", translate=True)
    website_meta_description = fields.Text("Website meta description", translate=True)
    website_meta_keywords = fields.Char("Website meta keywords", translate=True)
    website_meta_og_img = fields.Char("Website opengraph image")

    def _compute_is_seo_optimized(self):
        for record in self:
            record.is_seo_optimized = record.website_meta_title and record.website_meta_description and record.website_meta_keywords

    def _default_website_meta(self):
        """ This method will return default meta information. It return the dict
            contains meta property as a key and meta content as a value.
            e.g. 'og:type': 'website'.

            Override this method in case you want to change default value
            from any model. e.g. change value of og:image to product specific
            images instead of default images
        """
        self.ensure_one()
        company = request.website.company_id.sudo()
        title = (request.website or company).name
        if 'name' in self:
            title = '%s | %s' % (self.name, title)

        if request.website.social_default_image:
            img = request.website.image_url(request.website, 'social_default_image')
            img300 = request.website.image_url(request.website, 'social_default_image', size='300x300')
        else:
            img = request.website.image_url(company, 'logo')
            img300 = request.website.image_url(company, 'logo', size='300x300')

        # Default meta for OpenGraph
        default_opengraph = {
            'og:type': 'website',
            'og:title': title,
            'og:site_name': company.name,
            'og:url': url_join(request.httprequest.url_root, url_for(request.httprequest.path)),
            'og:image': img,
        }
        # Default meta for Twitter
        default_twitter = {
            'twitter:card': 'summary_large_image',
            'twitter:title': title,
            'twitter:image': img300,
        }
        if company.social_twitter:
            default_twitter['twitter:site'] = "@%s" % company.social_twitter.split('/')[-1]

        return {
            'default_opengraph': default_opengraph,
            'default_twitter': default_twitter
        }

    def get_website_meta(self):
        """ This method will return final meta information. It will replace
            default values with user's custom value (if user modified it from
            the seo popup of frontend)

            This method is not meant for overridden. To customize meta values
            override `_default_website_meta` method instead of this method. This
            method only replaces user custom values in defaults.
        """
        root_url = request.httprequest.url_root.strip('/')
        default_meta = self._default_website_meta()
        opengraph_meta, twitter_meta = default_meta['default_opengraph'], default_meta['default_twitter']
        if self.website_meta_title:
            opengraph_meta['og:title'] = self.website_meta_title
            twitter_meta['twitter:title'] = self.website_meta_title
        if self.website_meta_description:
            opengraph_meta['og:description'] = self.website_meta_description
            twitter_meta['twitter:description'] = self.website_meta_description
        opengraph_meta['og:image'] = url_join(root_url, url_for(self.website_meta_og_img or opengraph_meta['og:image']))
        twitter_meta['twitter:image'] = url_join(root_url, url_for(self.website_meta_og_img or twitter_meta['twitter:image']))
        return {
            'opengraph_meta': opengraph_meta,
            'twitter_meta': twitter_meta,
            'meta_description': default_meta.get('default_meta_description')
        }


class WebsiteMultiMixin(models.AbstractModel):

    _name = 'website.multi.mixin'
    _description = 'Multi Website Mixin'

    website_id = fields.Many2one(
        "website",
        string="Website",
        ondelete="restrict",
        help="Restrict publishing to this website.",
    )

    def can_access_from_current_website(self, website_id=False):
        can_access = True
        for record in self:
            if (website_id or record.website_id.id) not in (False, request.website.id):
                can_access = False
                continue
        return can_access


class WebsitePublishedMixin(models.AbstractModel):

    _name = "website.published.mixin"
    _description = 'Website Published Mixin'

    website_published = fields.Boolean('Visible on current website', related='is_published', readonly=False)
    is_published = fields.Boolean('Is Published', copy=False, default=lambda self: self._default_is_published())
    can_publish = fields.Boolean('Can Publish', compute='_compute_can_publish')
    website_url = fields.Char('Website URL', compute='_compute_website_url', help='The full URL to access the document through the website.')

    @api.depends_context('lang')
    def _compute_website_url(self):
        for record in self:
            record.website_url = '#'

    def _default_is_published(self):
        return False

    def website_publish_button(self):
        self.ensure_one()
        return self.write({'website_published': not self.website_published})

    def open_website_url(self):
        return {
            'type': 'ir.actions.act_url',
            'url': self.website_url,
            'target': 'self',
        }

    @api.model_create_multi
    def create(self, vals_list):
        records = super(WebsitePublishedMixin, self).create(vals_list)
        is_publish_modified = any(
            [set(v.keys()) & {'is_published', 'website_published'} for v in vals_list]
        )
        if is_publish_modified and not all(record.can_publish for record in records):
            raise AccessError(self._get_can_publish_error_message())

        return records

    def write(self, values):
        if 'is_published' in values and not all(record.can_publish for record in self):
            raise AccessError(self._get_can_publish_error_message())

        return super(WebsitePublishedMixin, self).write(values)

    def create_and_get_website_url(self, **kwargs):
        return self.create(kwargs).website_url

    def _compute_can_publish(self):
        """ This method can be overridden if you need more complex rights management than just 'website_publisher'
        The publish widget will be hidden and the user won't be able to change the 'website_published' value
        if this method sets can_publish False """
        for record in self:
            record.can_publish = True

    @api.model
    def _get_can_publish_error_message(self):
        """ Override this method to customize the error message shown when the user doesn't
        have the rights to publish/unpublish. """
        return _("You do not have the rights to publish/unpublish")


class WebsitePublishedMultiMixin(WebsitePublishedMixin):

    _name = 'website.published.multi.mixin'
    _inherit = ['website.published.mixin', 'website.multi.mixin']
    _description = 'Multi Website Published Mixin'

    website_published = fields.Boolean(compute='_compute_website_published',
                                       inverse='_inverse_website_published',
                                       search='_search_website_published',
                                       related=False, readonly=False)

    @api.depends('is_published', 'website_id')
    @api.depends_context('website_id')
    def _compute_website_published(self):
        current_website_id = self._context.get('website_id')
        for record in self:
            if current_website_id:
                record.website_published = record.is_published and (not record.website_id or record.website_id.id == current_website_id)
            else:
                record.website_published = record.is_published

    def _inverse_website_published(self):
        for record in self:
            record.is_published = record.website_published

    def _search_website_published(self, operator, value):
        if not isinstance(value, bool) or operator not in ('=', '!='):
            logger.warning('unsupported search on website_published: %s, %s', operator, value)
            return [()]

        if operator in expression.NEGATIVE_TERM_OPERATORS:
            value = not value

        current_website_id = self._context.get('website_id')
        is_published = [('is_published', '=', value)]
        if current_website_id:
            on_current_website = self.env['website'].website_domain(current_website_id)
            return (['!'] if value is False else []) + expression.AND([is_published, on_current_website])
        else:  # should be in the backend, return things that are published anywhere
            return is_published

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Company(models.Model):
    _inherit = "res.company"

    def google_map_img(self, zoom=8, width=298, height=298):
        partner = self.sudo().partner_id
        return partner and partner.google_map_img(zoom, width, height) or None

    def google_map_link(self, zoom=8):
        partner = self.sudo().partner_id
        return partner and partner.google_map_link(zoom) or None

    def _get_public_user(self):
        self.ensure_one()
        # We need sudo to be able to see public users from others companies too
        public_users = self.env.ref('base.group_public').sudo().with_context(active_test=False).users
        public_users_for_website = public_users.filtered(lambda user: user.company_id == self)

        if public_users_for_website:
            return public_users_for_website[0]
        else:
            return self.env.ref('base.public_user').sudo().copy({
                'name': 'Public user for %s' % self.name,
                'login': 'public-user@company-%s.com' % self.id,
                'company_id': self.id,
                'company_ids': [(6, 0, [self.id])],
            })

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def _default_website(self):
        return self.env['website'].search([('company_id', '=', self.env.company.id)], limit=1)

    website_id = fields.Many2one('website', string="website",
                                 default=_default_website, ondelete='cascade')
    website_name = fields.Char('Website Name', related='website_id.name', readonly=False)
    website_domain = fields.Char('Website Domain', related='website_id.domain', readonly=False)
    website_country_group_ids = fields.Many2many(related='website_id.country_group_ids', readonly=False)
    website_company_id = fields.Many2one(related='website_id.company_id', string='Website Company', readonly=False)
    website_logo = fields.Binary(related='website_id.logo', readonly=False)
    language_ids = fields.Many2many(related='website_id.language_ids', relation='res.lang', readonly=False)
    website_language_count = fields.Integer(string='Number of languages', compute='_compute_website_language_count', readonly=True)
    website_default_lang_id = fields.Many2one(string='Default language', related='website_id.default_lang_id',
                                              readonly=False, relation='res.lang')
    website_default_lang_code = fields.Char('Default language code', related='website_id.default_lang_id.code', readonly=False)
    specific_user_account = fields.Boolean(related='website_id.specific_user_account', readonly=False,
                                           help='Are newly created user accounts website specific')

    google_analytics_key = fields.Char('Google Analytics Key', related='website_id.google_analytics_key', readonly=False)
    google_management_client_id = fields.Char('Google Client ID', related='website_id.google_management_client_id', readonly=False)
    google_management_client_secret = fields.Char('Google Client Secret', related='website_id.google_management_client_secret', readonly=False)

    cdn_activated = fields.Boolean(related='website_id.cdn_activated', readonly=False)
    cdn_url = fields.Char(related='website_id.cdn_url', readonly=False)
    cdn_filters = fields.Text(related='website_id.cdn_filters', readonly=False)
    module_website_version = fields.Boolean("A/B Testing")
    module_website_links = fields.Boolean("Link Trackers")
    auth_signup_uninvited = fields.Selection(compute="_compute_auth_signup",
        inverse="_set_auth_signup")

    social_twitter = fields.Char(related='website_id.social_twitter', readonly=False)
    social_facebook = fields.Char(related='website_id.social_facebook', readonly=False)
    social_github = fields.Char(related='website_id.social_github', readonly=False)
    social_linkedin = fields.Char(related='website_id.social_linkedin', readonly=False)
    social_youtube = fields.Char(related='website_id.social_youtube', readonly=False)
    social_instagram = fields.Char(related='website_id.social_instagram', readonly=False)

    @api.depends('website_id', 'social_twitter', 'social_facebook', 'social_github', 'social_linkedin', 'social_youtube', 'social_instagram')
    def has_social_network(self):
        self.has_social_network = self.social_twitter or self.social_facebook or self.social_github \
            or self.social_linkedin or self.social_youtube or self.social_instagram

    def inverse_has_social_network(self):
        if not self.has_social_network:
            self.social_twitter = ''
            self.social_facebook = ''
            self.social_github = ''
            self.social_linkedin = ''
            self.social_youtube = ''
            self.social_instagram = ''

    has_social_network = fields.Boolean("Configure Social Network", compute=has_social_network, inverse=inverse_has_social_network)

    favicon = fields.Binary('Favicon', related='website_id.favicon', readonly=False)
    social_default_image = fields.Binary('Default Social Share Image', related='website_id.social_default_image', readonly=False)

    google_maps_api_key = fields.Char(related='website_id.google_maps_api_key', readonly=False)
    group_multi_website = fields.Boolean("Multi-website", implied_group="website.group_multi_website")

    @api.depends('website_id.auth_signup_uninvited')
    def _compute_auth_signup(self):
        for config in self:
            config.auth_signup_uninvited = config.website_id.auth_signup_uninvited

    def _set_auth_signup(self):
        for config in self:
            config.website_id.auth_signup_uninvited = config.auth_signup_uninvited

    @api.depends('website_id')
    def has_google_analytics(self):
        self.has_google_analytics = bool(self.google_analytics_key)

    @api.depends('website_id')
    def has_google_analytics_dashboard(self):
        self.has_google_analytics_dashboard = bool(self.google_management_client_id)

    @api.depends('website_id')
    def has_google_maps(self):
        self.has_google_maps = bool(self.google_maps_api_key)

    def inverse_has_google_analytics(self):
        if not self.has_google_analytics:
            self.has_google_analytics_dashboard = False
            self.google_analytics_key = False

    def inverse_has_google_maps(self):
        if not self.has_google_maps:
            self.google_maps_api_key = False

    def inverse_has_google_analytics_dashboard(self):
        if not self.has_google_analytics_dashboard:
            self.google_management_client_id = False
            self.google_management_client_secret = False

    has_google_analytics = fields.Boolean("Google Analytics", compute=has_google_analytics, inverse=inverse_has_google_analytics)
    has_google_analytics_dashboard = fields.Boolean("Google Analytics Dashboard", compute=has_google_analytics_dashboard, inverse=inverse_has_google_analytics_dashboard)
    has_google_maps = fields.Boolean("Google Maps", compute=has_google_maps, inverse=inverse_has_google_maps)

    @api.onchange('language_ids')
    def _onchange_language_ids(self):
        # If current default language is removed from language_ids
        # update the website_default_lang_id
        language_ids = self.language_ids._origin
        if not language_ids:
            self.website_default_lang_id = False
        elif self.website_default_lang_id not in language_ids:
            self.website_default_lang_id = language_ids[0]

    @api.depends('language_ids')
    def _compute_website_language_count(self):
        for config in self:
            config.website_language_count = len(self.language_ids)

    def set_values(self):
        super(ResConfigSettings, self).set_values()

    def open_template_user(self):
        action = self.env.ref('base.action_res_users').read()[0]
        action['res_id'] = literal_eval(self.env['ir.config_parameter'].sudo().get_param('base.template_portal_user_id', 'False'))
        action['views'] = [[self.env.ref('base.view_users_form').id, 'form']]
        return action

    def website_go_to(self):
        self.website_id._force()
        return {
            'type': 'ir.actions.act_url',
            'url': '/',
            'target': 'self',
        }

    def action_website_create_new(self):
        return {
            'view_mode': 'form',
            'view_id': self.env.ref('website.view_website_form').id,
            'res_model': 'website',
            'type': 'ir.actions.act_window',
            'target': 'new',
            'res_id': False,
        }

```

## File: models\res_lang.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, tools, _
from odoo.addons.website.models import ir_http
from odoo.exceptions import UserError
from odoo.http import request


class Lang(models.Model):
    _inherit = "res.lang"

    def write(self, vals):
        if 'active' in vals and not vals['active']:
            if self.env['website'].search([('language_ids', 'in', self._ids)]):
                raise UserError(_("Cannot deactivate a language that is currently used on a website."))
        return super(Lang, self).write(vals)

    @api.model
    @tools.ormcache_context(keys=("website_id",))
    def get_available(self):
        """ Return the available languages as a list of (code, name) sorted by name. """
        website = ir_http.get_request_website()
        if website:
            return sorted([(lang.code, lang.url_code, lang.name) for lang in request.website.language_ids])
        return super(Lang, self).get_available()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug

from odoo import models, fields


def urlplus(url, params):
    return werkzeug.Href(url)(params or None)


class Partner(models.Model):
    _name = 'res.partner'
    _inherit = ['res.partner', 'website.published.multi.mixin']

    visitor_ids = fields.Many2many('website.visitor', 'website_visitor_partner_rel', 'partner_id', 'visitor_id', string='Visitors')

    def google_map_img(self, zoom=8, width=298, height=298):
        google_maps_api_key = self.env['website'].get_current_website().google_maps_api_key
        if not google_maps_api_key:
            return False
        params = {
            'center': '%s, %s %s, %s' % (self.street or '', self.city or '', self.zip or '', self.country_id and self.country_id.display_name or ''),
            'size': "%sx%s" % (width, height),
            'zoom': zoom,
            'sensor': 'false',
            'key': google_maps_api_key,
        }
        return urlplus('//maps.googleapis.com/maps/api/staticmap', params)

    def google_map_link(self, zoom=10):
        params = {
            'q': '%s, %s %s, %s' % (self.street or '', self.city or '', self.zip or '', self.country_id and self.country_id.display_name or ''),
            'z': zoom,
        }
        return urlplus('https://maps.google.com/maps', params)

    def _get_name(self):
        name = super(Partner, self)._get_name()
        if self._context.get('display_website') and self.env.user.has_group('website.group_multi_website'):
            if self.website_id:
                name += ' [%s]' % self.website_id.name
        return name

    def _compute_display_name(self):
        self2 = self.with_context(display_website=False)
        super(Partner, self2)._compute_display_name()


```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.http import request

_logger = logging.getLogger(__name__)


class ResUsers(models.Model):
    _inherit = 'res.users'

    website_id = fields.Many2one('website', related='partner_id.website_id', store=True, related_sudo=False, readonly=False)

    _sql_constraints = [
        # Partial constraint, complemented by a python constraint (see below).
        ('login_key', 'unique (login, website_id)', 'You can not have two users with the same login!'),
    ]

    @api.constrains('login', 'website_id')
    def _check_login(self):
        """ Do not allow two users with the same login without website """
        self.flush(['login', 'website_id'])
        self.env.cr.execute(
            """SELECT login
                 FROM res_users
                WHERE login IN (SELECT login FROM res_users WHERE id IN %s AND website_id IS NULL)
                  AND website_id IS NULL
             GROUP BY login
               HAVING COUNT(*) > 1
            """,
            (tuple(self.ids),)
        )
        if self.env.cr.rowcount:
            raise ValidationError(_('You can not have two users with the same login!'))

    @api.model
    def _get_login_domain(self, login):
        website = self.env['website'].get_current_website()
        return super(ResUsers, self)._get_login_domain(login) + website.website_domain()

    @api.model
    def _get_login_order(self):
        return 'website_id, ' + super(ResUsers, self)._get_login_order()

    @api.model
    def _signup_create_user(self, values):
        current_website = self.env['website'].get_current_website()
        if request and current_website.specific_user_account:
            values['company_id'] = current_website.company_id.id
            values['company_ids'] = [(4, current_website.company_id.id)]
            values['website_id'] = current_website.id
        new_user = super(ResUsers, self)._signup_create_user(values)
        return new_user

    @api.model
    def _get_signup_invitation_scope(self):
        current_website = self.env['website'].get_current_website()
        return current_website.auth_signup_uninvited or super(ResUsers, self)._get_signup_invitation_scope()

    @classmethod
    def authenticate(cls, db, login, password, user_agent_env):
        """ Override to link the logged in user's res.partner to website.visitor """
        uid = super(ResUsers, cls).authenticate(db, login, password, user_agent_env)
        if uid:
            with cls.pool.cursor() as cr:
                env = api.Environment(cr, uid, {})
                visitor_sudo = env['website.visitor']._get_visitor_from_request()
                if visitor_sudo:
                    partner = env.user.partner_id
                    partner_visitor = env['website.visitor'].with_context(active_test=False).sudo().search([('partner_id', '=', partner.id)])
                    if partner_visitor and partner_visitor.id != visitor_sudo.id:
                        # Link history to older Visitor and delete the newest
                        visitor_sudo.website_track_ids.write({'visitor_id': partner_visitor.id})
                        visitor_sudo.unlink()
                        # If archived (most likely by the cron for inactivity reasons), reactivate the partner's visitor
                        if not partner_visitor.active:
                            partner_visitor.write({'active': True})
                    else:
                        vals = {
                            'partner_id': partner.id,
                            'name': partner.name
                        }
                        visitor_sudo.write(vals)
        return uid

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import inspect
import logging
import hashlib
import re


from werkzeug import urls
from werkzeug.datastructures import OrderedMultiDict
from werkzeug.exceptions import NotFound

from odoo import api, fields, models, tools, http
from odoo.addons.base.models.ir_model import MODULE_UNINSTALL_FLAG
from odoo.addons.http_routing.models.ir_http import slugify, _guess_mimetype
from odoo.addons.website.models.ir_http import sitemap_qs2dom
from odoo.addons.portal.controllers.portal import pager
from odoo.exceptions import UserError
from odoo.http import request
from odoo.modules.module import get_resource_path
from odoo.osv.expression import FALSE_DOMAIN
from odoo.tools.translate import _

logger = logging.getLogger(__name__)


DEFAULT_CDN_FILTERS = [
    "^/[^/]+/static/",
    "^/web/(css|js)/",
    "^/web/image",
    "^/web/content",
    # retrocompatibility
    "^/website/image/",
]


class Website(models.Model):

    _name = "website"
    _description = "Website"

    @api.model
    def website_domain(self, website_id=False):
        return [('website_id', 'in', (False, website_id or self.id))]

    def _active_languages(self):
        return self.env['res.lang'].search([]).ids

    def _default_language(self):
        lang_code = self.env['ir.default'].get('res.partner', 'lang')
        def_lang_id = self.env['res.lang']._lang_get_id(lang_code)
        return def_lang_id or self._active_languages()[0]

    name = fields.Char('Website Name', required=True)
    domain = fields.Char('Website Domain',
        help='Will be prefixed by http in canonical URLs if no scheme is specified')
    country_group_ids = fields.Many2many('res.country.group', 'website_country_group_rel', 'website_id', 'country_group_id',
                                         string='Country Groups', help='Used when multiple websites have the same domain.')
    company_id = fields.Many2one('res.company', string="Company", default=lambda self: self.env.company, required=True)
    language_ids = fields.Many2many('res.lang', 'website_lang_rel', 'website_id', 'lang_id', 'Languages', default=_active_languages)
    default_lang_id = fields.Many2one('res.lang', string="Default Language", default=_default_language, required=True)
    auto_redirect_lang = fields.Boolean('Autoredirect Language', default=True, help="Should users be redirected to their browser's language")

    def _default_social_facebook(self):
        return self.env.ref('base.main_company').social_facebook

    def _default_social_github(self):
        return self.env.ref('base.main_company').social_github

    def _default_social_linkedin(self):
        return self.env.ref('base.main_company').social_linkedin

    def _default_social_youtube(self):
        return self.env.ref('base.main_company').social_youtube

    def _default_social_instagram(self):
        return self.env.ref('base.main_company').social_instagram

    def _default_social_twitter(self):
        return self.env.ref('base.main_company').social_twitter

    def _default_logo(self):
        image_path = get_resource_path('website', 'static/src/img', 'website_logo.png')
        with tools.file_open(image_path, 'rb') as f:
            return base64.b64encode(f.read())

    logo = fields.Binary('Website Logo', default=_default_logo, help="Display this logo on the website.")
    social_twitter = fields.Char('Twitter Account', default=_default_social_twitter)
    social_facebook = fields.Char('Facebook Account', default=_default_social_facebook)
    social_github = fields.Char('GitHub Account', default=_default_social_github)
    social_linkedin = fields.Char('LinkedIn Account', default=_default_social_linkedin)
    social_youtube = fields.Char('Youtube Account', default=_default_social_youtube)
    social_instagram = fields.Char('Instagram Account', default=_default_social_instagram)
    social_default_image = fields.Binary(string="Default Social Share Image", help="If set, replaces the company logo as the default social share image.")

    google_analytics_key = fields.Char('Google Analytics Key')
    google_management_client_id = fields.Char('Google Client ID')
    google_management_client_secret = fields.Char('Google Client Secret')

    google_maps_api_key = fields.Char('Google Maps API Key')

    user_id = fields.Many2one('res.users', string='Public User', required=True)
    cdn_activated = fields.Boolean('Content Delivery Network (CDN)')
    cdn_url = fields.Char('CDN Base URL', default='')
    cdn_filters = fields.Text('CDN Filters', default=lambda s: '\n'.join(DEFAULT_CDN_FILTERS), help="URL matching those filters will be rewritten using the CDN Base URL")
    partner_id = fields.Many2one(related='user_id.partner_id', relation='res.partner', string='Public Partner', readonly=False)
    menu_id = fields.Many2one('website.menu', compute='_compute_menu', string='Main Menu')
    homepage_id = fields.Many2one('website.page', string='Homepage')

    def _default_favicon(self):
        img_path = get_resource_path('web', 'static/src/img/favicon.ico')
        with tools.file_open(img_path, 'rb') as f:
            return base64.b64encode(f.read())

    favicon = fields.Binary(string="Website Favicon", help="This field holds the image used to display a favicon on the website.", default=_default_favicon)
    theme_id = fields.Many2one('ir.module.module', help='Installed theme')

    specific_user_account = fields.Boolean('Specific User Account', help='If True, new accounts will be associated to the current website')
    auth_signup_uninvited = fields.Selection([
        ('b2b', 'On invitation'),
        ('b2c', 'Free sign up'),
    ], string='Customer Account', default='b2b')

    @api.onchange('language_ids')
    def _onchange_language_ids(self):
        language_ids = self.language_ids._origin
        if language_ids and self.default_lang_id not in language_ids:
            self.default_lang_id = language_ids[0]

    def _compute_menu(self):
        for website in self:
            menus = self.env['website.menu'].browse(website._get_menu_ids())

            # use field parent_id (1 query) to determine field child_id (2 queries by level)"
            for menu in menus:
                menu._cache['child_id'] = ()
            for menu in menus:
                # don't add child menu if parent is forbidden
                if menu.parent_id and menu.parent_id in menus:
                    menu.parent_id._cache['child_id'] += (menu.id,)

            top_menus = menus.filtered(lambda m: not m.parent_id)
            website.menu_id = top_menus and top_menus[0].id or False

    # self.env.uid for ir.rule groups on menu
    @tools.ormcache('self.env.uid', 'self.id')
    def _get_menu_ids(self):
        return self.env['website.menu'].search([('website_id', '=', self.id)]).ids

    @api.model
    def create(self, vals):
        self._handle_favicon(vals)

        if 'user_id' not in vals:
            company = self.env['res.company'].browse(vals.get('company_id'))
            vals['user_id'] = company._get_public_user().id if company else self.env.ref('base.public_user').id

        res = super(Website, self).create(vals)
        res._bootstrap_homepage()

        if not self.env.user.has_group('website.group_multi_website') and self.search_count([]) > 1:
            all_user_groups = 'base.group_portal,base.group_user,base.group_public'
            groups = self.env['res.groups'].concat(*(self.env.ref(it) for it in all_user_groups.split(',')))
            groups.write({'implied_ids': [(4, self.env.ref('website.group_multi_website').id)]})

        return res

    def write(self, values):
        public_user_to_change_websites = self.env['website']
        self._handle_favicon(values)

        self.clear_caches()

        if 'company_id' in values and 'user_id' not in values:
            public_user_to_change_websites = self.filtered(lambda w: w.sudo().user_id.company_id.id != values['company_id'])
            if public_user_to_change_websites:
                company = self.env['res.company'].browse(values['company_id'])
                super(Website, public_user_to_change_websites).write(dict(values, user_id=company and company._get_public_user().id))

        result = super(Website, self - public_user_to_change_websites).write(values)
        if 'cdn_activated' in values or 'cdn_url' in values or 'cdn_filters' in values:
            # invalidate the caches from static node at compile time
            self.env['ir.qweb'].clear_caches()
        return result

    @api.model
    def _handle_favicon(self, vals):
        if 'favicon' in vals:
            vals['favicon'] = tools.image_process(vals['favicon'], size=(256, 256), crop='center', output_format='ICO')

    def unlink(self):
        if not self.env.context.get(MODULE_UNINSTALL_FLAG, False):
            website = self.search([('id', 'not in', self.ids)], limit=1)
            if not website:
                raise UserError(_('You must keep at least one website.'))

        # Do not delete invoices, delete what's strictly necessary
        attachments_to_unlink = self.env['ir.attachment'].search([
            ('website_id', 'in', self.ids),
            '|', '|',
            ('key', '!=', False),  # theme attachment
            ('url', 'ilike', '.custom.'),  # customized theme attachment
            ('url', 'ilike', '.assets\\_'),
        ])
        attachments_to_unlink.unlink()
        return super(Website, self).unlink()

    # ----------------------------------------------------------
    # Page Management
    # ----------------------------------------------------------
    def _bootstrap_homepage(self):
        Page = self.env['website.page']
        standard_homepage = self.env.ref('website.homepage', raise_if_not_found=False)
        if not standard_homepage:
            return

        new_homepage_view = '''<t name="Homepage" t-name="website.homepage%s">
        <t t-call="website.layout">
            <t t-set="pageName" t-value="'homepage'"/>
            <div id="wrap" class="oe_structure oe_empty"/>
            </t>
        </t>''' % (self.id)
        standard_homepage.with_context(website_id=self.id).arch_db = new_homepage_view

        homepage_page = Page.search([
            ('website_id', '=', self.id),
            ('key', '=', standard_homepage.key),
        ], limit=1)
        if not homepage_page:
            homepage_page = Page.create({
                'website_published': True,
                'url': '/',
                'view_id': self.with_context(website_id=self.id).viewref('website.homepage').id,
            })
        # prevent /-1 as homepage URL
        homepage_page.url = '/'
        self.homepage_id = homepage_page

        # Bootstrap default menu hierarchy, create a new minimalist one if no default
        default_menu = self.env.ref('website.main_menu')
        self.copy_menu_hierarchy(default_menu)

    def copy_menu_hierarchy(self, top_menu):
        def copy_menu(menu, t_menu):
            new_menu = menu.copy({
                'parent_id': t_menu.id,
                'website_id': self.id,
            })
            for submenu in menu.child_id:
                copy_menu(submenu, new_menu)
        for website in self:
            new_top_menu = top_menu.copy({
                'name': _('Top Menu for Website %s') % website.id,
                'website_id': website.id,
            })
            for submenu in top_menu.child_id:
                copy_menu(submenu, new_top_menu)

    @api.model
    def new_page(self, name=False, add_menu=False, template='website.default_page', ispage=True, namespace=None):
        """ Create a new website page, and assign it a xmlid based on the given one
            :param name : the name of the page
            :param template : potential xml_id of the page to create
            :param namespace : module part of the xml_id if none, the template module name is used
        """
        if namespace:
            template_module = namespace
        else:
            template_module, _ = template.split('.')
        page_url = '/' + slugify(name, max_length=1024, path=True)
        page_url = self.get_unique_path(page_url)
        page_key = slugify(name)
        result = dict({'url': page_url, 'view_id': False})

        if not name:
            name = 'Home'
            page_key = 'home'

        template_record = self.env.ref(template)
        website_id = self._context.get('website_id')
        key = self.get_unique_key(page_key, template_module)
        view = template_record.copy({'website_id': website_id, 'key': key})

        view.with_context(lang=None).write({
            'arch': template_record.arch.replace(template, key),
            'name': name,
        })

        if view.arch_fs:
            view.arch_fs = False

        website = self.get_current_website()
        if ispage:
            page = self.env['website.page'].create({
                'url': page_url,
                'website_id': website.id,  # remove it if only one website or not?
                'view_id': view.id,
            })
            result['view_id'] = view.id
        if add_menu:
            self.env['website.menu'].create({
                'name': name,
                'url': page_url,
                'parent_id': website.menu_id.id,
                'page_id': page.id,
                'website_id': website.id,
            })
        return result

    @api.model
    def guess_mimetype(self):
        return _guess_mimetype()

    def get_unique_path(self, page_url):
        """ Given an url, return that url suffixed by counter if it already exists
            :param page_url : the url to be checked for uniqueness
        """
        inc = 0
        # we only want a unique_path for website specific.
        # we need to be able to have /url for website=False, and /url for website=1
        # in case of duplicate, page manager will allow you to manage this case
        domain_static = [('website_id', '=', self.get_current_website().id)]  # .website_domain()
        page_temp = page_url
        while self.env['website.page'].with_context(active_test=False).sudo().search([('url', '=', page_temp)] + domain_static):
            inc += 1
            page_temp = page_url + (inc and "-%s" % inc or "")
        return page_temp

    def get_unique_key(self, string, template_module=False):
        """ Given a string, return an unique key including module prefix.
            It will be suffixed by a counter if it already exists to garantee uniqueness.
            :param string : the key to be checked for uniqueness, you can pass it with 'website.' or not
            :param template_module : the module to be prefixed on the key, if not set, we will use website
        """
        if template_module:
            string = template_module + '.' + string
        else:
            if not string.startswith('website.'):
                string = 'website.' + string

        # Look for unique key
        key_copy = string
        inc = 0
        domain_static = self.get_current_website().website_domain()
        while self.env['website.page'].with_context(active_test=False).sudo().search([('key', '=', key_copy)] + domain_static):
            inc += 1
            key_copy = string + (inc and "-%s" % inc or "")
        return key_copy

    @api.model
    def page_search_dependencies(self, page_id=False):
        """ Search dependencies just for information. It will not catch 100%
            of dependencies and False positive is more than possible
            Each module could add dependences in this dict
            :returns a dictionnary where key is the 'categorie' of object related to the given
                view, and the value is the list of text and link to the resource using given page
        """
        dependencies = {}
        if not page_id:
            return dependencies

        page = self.env['website.page'].browse(int(page_id))
        website = self.env['website'].browse(self._context.get('website_id'))
        url = page.url

        # search for website_page with link
        website_page_search_dom = [('view_id.arch_db', 'ilike', url)] + website.website_domain()
        pages = self.env['website.page'].search(website_page_search_dom)
        page_key = _('Page')
        if len(pages) > 1:
            page_key = _('Pages')
        page_view_ids = []
        for page in pages:
            dependencies.setdefault(page_key, [])
            dependencies[page_key].append({
                'text': _('Page <b>%s</b> contains a link to this page') % page.url,
                'item': page.name,
                'link': page.url,
            })
            page_view_ids.append(page.view_id.id)

        # search for ir_ui_view (not from a website_page) with link
        page_search_dom = [('arch_db', 'ilike', url), ('id', 'not in', page_view_ids)] + website.website_domain()
        views = self.env['ir.ui.view'].search(page_search_dom)
        view_key = _('Template')
        if len(views) > 1:
            view_key = _('Templates')
        for view in views:
            dependencies.setdefault(view_key, [])
            dependencies[view_key].append({
                'text': _('Template <b>%s (id:%s)</b> contains a link to this page') % (view.key or view.name, view.id),
                'link': '/web#id=%s&view_type=form&model=ir.ui.view' % view.id,
                'item': _('%s (id:%s)') % (view.key or view.name, view.id),
            })
        # search for menu with link
        menu_search_dom = [('url', 'ilike', '%s' % url)] + website.website_domain()

        menus = self.env['website.menu'].search(menu_search_dom)
        menu_key = _('Menu')
        if len(menus) > 1:
            menu_key = _('Menus')
        for menu in menus:
            dependencies.setdefault(menu_key, []).append({
                'text': _('This page is in the menu <b>%s</b>') % menu.name,
                'link': '/web#id=%s&view_type=form&model=website.menu' % menu.id,
                'item': menu.name,
            })

        return dependencies

    @api.model
    def page_search_key_dependencies(self, page_id=False):
        """ Search dependencies just for information. It will not catch 100%
            of dependencies and False positive is more than possible
            Each module could add dependences in this dict
            :returns a dictionnary where key is the 'categorie' of object related to the given
                view, and the value is the list of text and link to the resource using given page
        """
        dependencies = {}
        if not page_id:
            return dependencies

        page = self.env['website.page'].browse(int(page_id))
        website = self.env['website'].browse(self._context.get('website_id'))
        key = page.key

        # search for website_page with link
        website_page_search_dom = [
            ('view_id.arch_db', 'ilike', key),
            ('id', '!=', page.id)
        ] + website.website_domain()
        pages = self.env['website.page'].search(website_page_search_dom)
        page_key = _('Page')
        if len(pages) > 1:
            page_key = _('Pages')
        page_view_ids = []
        for p in pages:
            dependencies.setdefault(page_key, [])
            dependencies[page_key].append({
                'text': _('Page <b>%s</b> is calling this file') % p.url,
                'item': p.name,
                'link': p.url,
            })
            page_view_ids.append(p.view_id.id)

        # search for ir_ui_view (not from a website_page) with link
        page_search_dom = [
            ('arch_db', 'ilike', key), ('id', 'not in', page_view_ids),
            ('id', '!=', page.view_id.id),
        ] + website.website_domain()
        views = self.env['ir.ui.view'].search(page_search_dom)
        view_key = _('Template')
        if len(views) > 1:
            view_key = _('Templates')
        for view in views:
            dependencies.setdefault(view_key, [])
            dependencies[view_key].append({
                'text': _('Template <b>%s (id:%s)</b> is calling this file') % (view.key or view.name, view.id),
                'item': _('%s (id:%s)') % (view.key or view.name, view.id),
                'link': '/web#id=%s&view_type=form&model=ir.ui.view' % view.id,
            })

        return dependencies

    # ----------------------------------------------------------
    # Languages
    # ----------------------------------------------------------

    def _get_alternate_languages(self, canonical_params):
        self.ensure_one()

        if not self._is_canonical_url(canonical_params=canonical_params):
            # no hreflang on non-canonical pages
            return []

        languages = self.language_ids
        if len(languages) <= 1:
            # no hreflang if no alternate language
            return []

        langs = []
        shorts = []

        for lg in languages:
            lg_codes = lg.code.split('_')
            short = lg_codes[0]
            shorts.append(short)
            langs.append({
                'hreflang': ('-'.join(lg_codes)).lower(),
                'short': short,
                'href': self._get_canonical_url_localized(lang=lg, canonical_params=canonical_params),
            })

        # if there is only one region for a language, use only the language code
        for lang in langs:
            if shorts.count(lang['short']) == 1:
                lang['hreflang'] = lang['short']

        # add the default
        langs.append({
            'hreflang': 'x-default',
            'href': self._get_canonical_url_localized(lang=self.default_lang_id, canonical_params=canonical_params),
        })

        return langs

    # ----------------------------------------------------------
    # Utilities
    # ----------------------------------------------------------

    @api.model
    def get_current_website(self, fallback=True):
        if request and request.session.get('force_website_id'):
            website_id = self.browse(request.session['force_website_id']).exists()
            if not website_id:
                # Don't crash is session website got deleted
                request.session.pop('force_website_id')
            else:
                return website_id

        website_id = self.env.context.get('website_id')
        if website_id:
            return self.browse(website_id)

        # The format of `httprequest.host` is `domain:port`
        domain_name = request and request.httprequest.host or ''

        country = request.session.geoip.get('country_code') if request and request.session.geoip else False
        country_id = False
        if country:
            country_id = self.env['res.country'].search([('code', '=', country)], limit=1).id

        website_id = self._get_current_website_id(domain_name, country_id, fallback=fallback)
        return self.browse(website_id)

    @tools.cache('domain_name', 'country_id', 'fallback')
    @api.model
    def _get_current_website_id(self, domain_name, country_id, fallback=True):
        """Get the current website id.

        First find all the websites for which the configured `domain` (after
        ignoring a potential scheme) is equal to the given
        `domain_name`. If there is only one result, return it immediately.

        If there are no website found for the given `domain_name`, either
        fallback to the first found website (no matter its `domain`) or return
        False depending on the `fallback` parameter.

        If there are multiple websites for the same `domain_name`, we need to
        filter them out by country. We return the first found website matching
        the given `country_id`. If no found website matching `domain_name`
        corresponds to the given `country_id`, the first found website for
        `domain_name` will be returned (no matter its country).

        :param domain_name: the domain for which we want the website.
            In regard to the `url_parse` method, only the `netloc` part should
            be given here, no `scheme`.
        :type domain_name: string

        :param country_id: id of the country for which we want the website
        :type country_id: int

        :param fallback: if True and no website is found for the specificed
            `domain_name`, return the first website (without filtering them)
        :type fallback: bool

        :return: id of the found website, or False if no website is found and
            `fallback` is False
        :rtype: int or False

        :raises: if `fallback` is True but no website at all is found
        """
        def _remove_port(domain_name):
            return (domain_name or '').split(':')[0]

        def _filter_domain(website, domain_name, ignore_port=False):
            """Ignore `scheme` from the `domain`, just match the `netloc` which
            is host:port in the version of `url_parse` we use."""
            # Here we add http:// to the domain if it's not set because
            # `url_parse` expects it to be set to correctly return the `netloc`.
            website_domain = urls.url_parse(website._get_http_domain()).netloc
            if ignore_port:
                website_domain = _remove_port(website_domain)
                domain_name = _remove_port(domain_name)
            return website_domain.lower() == (domain_name or '').lower()

        # Sort on country_group_ids so that we fall back on a generic website:
        # websites with empty country_group_ids will be first.
        found_websites = self.search([('domain', 'ilike', _remove_port(domain_name))]).sorted('country_group_ids')
        # Filter for the exact domain (to filter out potential subdomains) due
        # to the use of ilike.
        websites = found_websites.filtered(lambda w: _filter_domain(w, domain_name))
        # If there is no domain matching for the given port, ignore the port.
        websites = websites or found_websites.filtered(lambda w: _filter_domain(w, domain_name, ignore_port=True))

        if not websites:
            if not fallback:
                return False
            return self.search([], limit=1).id
        elif len(websites) == 1:
            return websites.id
        else:  # > 1 website with the same domain
            country_specific_websites = websites.filtered(lambda website: country_id in website.country_group_ids.mapped('country_ids').ids)
            return country_specific_websites[0].id if country_specific_websites else websites[0].id

    def _force(self):
        self._force_website(self.id)

    def _force_website(self, website_id):
        if request:
            request.session['force_website_id'] = website_id and str(website_id).isdigit() and int(website_id)

    @api.model
    def is_publisher(self):
        return self.env['ir.model.access'].check('ir.ui.view', 'write', False)

    @api.model
    def is_user(self):
        return self.env['ir.model.access'].check('ir.ui.menu', 'read', False)

    @api.model
    def is_public_user(self):
        return request.env.user.id == request.website.user_id.id

    @api.model
    def viewref(self, view_id, raise_if_not_found=True):
        ''' Given an xml_id or a view_id, return the corresponding view record.
            In case of website context, return the most specific one.

            If no website_id is in the context, it will return the generic view,
            instead of a random one like `get_view_id`.

            Look also for archived views, no matter the context.

            :param view_id: either a string xml_id or an integer view_id
            :param raise_if_not_found: should the method raise an error if no view found
            :return: The view record or empty recordset
        '''
        View = self.env['ir.ui.view']
        view = View
        if isinstance(view_id, str):
            if 'website_id' in self._context:
                domain = [('key', '=', view_id)] + self.env['website'].website_domain(self._context.get('website_id'))
                order = 'website_id'
            else:
                domain = [('key', '=', view_id)]
                order = View._order
            views = View.with_context(active_test=False).search(domain, order=order)
            if views:
                view = views.filter_duplicate()
            else:
                # we handle the raise below
                view = self.env.ref(view_id, raise_if_not_found=False)
                # self.env.ref might return something else than an ir.ui.view (eg: a theme.ir.ui.view)
                if not view or view._name != 'ir.ui.view':
                    # make sure we always return a recordset
                    view = View
        elif isinstance(view_id, int):
            view = View.browse(view_id)
        else:
            raise ValueError('Expecting a string or an integer, not a %s.' % (type(view_id)))

        if not view and raise_if_not_found:
            raise ValueError('No record found for unique ID %s. It may have been deleted.' % (view_id))
        return view

    @api.model
    def get_template(self, template):
        View = self.env['ir.ui.view']
        if isinstance(template, int):
            view_id = template
        else:
            if '.' not in template:
                template = 'website.%s' % template
            view_id = View.get_view_id(template)
        if not view_id:
            raise NotFound
        return View.browse(view_id)

    @api.model
    def pager(self, url, total, page=1, step=30, scope=5, url_args=None):
        return pager(url, total, page=page, step=step, scope=scope, url_args=url_args)

    def rule_is_enumerable(self, rule):
        """ Checks that it is possible to generate sensible GET queries for
            a given rule (if the endpoint matches its own requirements)
            :type rule: werkzeug.routing.Rule
            :rtype: bool
        """
        endpoint = rule.endpoint
        methods = endpoint.routing.get('methods') or ['GET']

        converters = list(rule._converters.values())
        if not ('GET' in methods and
                endpoint.routing['type'] == 'http' and
                endpoint.routing['auth'] in ('none', 'public') and
                endpoint.routing.get('website', False) and
                all(hasattr(converter, 'generate') for converter in converters)):
                return False

        # dont't list routes without argument having no default value or converter
        spec = inspect.getargspec(endpoint.method.original_func)

        # remove self and arguments having a default value
        defaults_count = len(spec.defaults or [])
        args = spec.args[1:(-defaults_count or None)]

        # check that all args have a converter
        return all((arg in rule._converters) for arg in args)

    def enumerate_pages(self, query_string=None, force=False):
        """ Available pages in the website/CMS. This is mostly used for links
            generation and can be overridden by modules setting up new HTML
            controllers for dynamic pages (e.g. blog).
            By default, returns template views marked as pages.
            :param str query_string: a (user-provided) string, fetches pages
                                     matching the string
            :returns: a list of mappings with two keys: ``name`` is the displayable
                      name of the resource (page), ``url`` is the absolute URL
                      of the same.
            :rtype: list({name: str, url: str})
        """

        router = http.root.get_db_router(request.db)
        url_set = set()

        sitemap_endpoint_done = set()

        for rule in router.iter_rules():
            if 'sitemap' in rule.endpoint.routing:
                if rule.endpoint in sitemap_endpoint_done:
                    continue
                sitemap_endpoint_done.add(rule.endpoint)

                func = rule.endpoint.routing['sitemap']
                if func is False:
                    continue
                for loc in func(self.env, rule, query_string):
                    yield loc
                continue

            if not self.rule_is_enumerable(rule):
                continue

            converters = rule._converters or {}
            if query_string and not converters and (query_string not in rule.build({}, append_unknown=False)[1]):
                continue

            values = [{}]
            # converters with a domain are processed after the other ones
            convitems = sorted(
                converters.items(),
                key=lambda x: (hasattr(x[1], 'domain') and (x[1].domain != '[]'), rule._trace.index((True, x[0]))))

            for (i, (name, converter)) in enumerate(convitems):
                newval = []
                for val in values:
                    query = i == len(convitems) - 1 and query_string
                    if query:
                        r = "".join([x[1] for x in rule._trace[1:] if not x[0]])  # remove model converter from route
                        query = sitemap_qs2dom(query, r, self.env[converter.model]._rec_name)
                        if query == FALSE_DOMAIN:
                            continue
                    for value_dict in converter.generate(uid=self.env.uid, dom=query, args=val):
                        newval.append(val.copy())
                        value_dict[name] = value_dict['loc']
                        del value_dict['loc']
                        newval[-1].update(value_dict)
                values = newval

            for value in values:
                domain_part, url = rule.build(value, append_unknown=False)
                if not query_string or query_string.lower() in url.lower():
                    page = {'loc': url}
                    if url in url_set:
                        continue
                    url_set.add(url)

                    yield page

        # '/' already has a http.route & is in the routing_map so it will already have an entry in the xml
        domain = [('url', '!=', '/')]
        if not force:
            domain += [('website_indexed', '=', True)]
            # is_visible
            domain += [('website_published', '=', True), '|', ('date_publish', '=', False), ('date_publish', '<=', fields.Datetime.now())]

        if query_string:
            domain += [('url', 'like', query_string)]

        pages = self.get_website_pages(domain)

        for page in pages:
            record = {'loc': page['url'], 'id': page['id'], 'name': page['name']}
            if page.view_id and page.view_id.priority != 16:
                record['priority'] = min(round(page.view_id.priority / 32.0, 1), 1)
            if page['write_date']:
                record['lastmod'] = page['write_date'].date()
            yield record

    def get_website_pages(self, domain=[], order='name', limit=None):
        domain += self.get_current_website().website_domain()
        pages = self.env['website.page'].search(domain, order='name', limit=limit)
        return pages

    def search_pages(self, needle=None, limit=None):

        name = slugify(needle, max_length=50, path=True)
        res = []
        for page in self.enumerate_pages(query_string=name, force=True):
            res.append(page)
            if len(res) == limit:
                break
        return res

    @api.model
    def image_url(self, record, field, size=None):
        """ Returns a local url that points to the image field of a given browse record. """
        sudo_record = record.sudo()
        sha = hashlib.sha1(str(getattr(sudo_record, '__last_update')).encode('utf-8')).hexdigest()[0:7]
        size = '' if size is None else '/%s' % size
        return '/web/image/%s/%s/%s%s?unique=%s' % (record._name, record.id, field, size, sha)

    def get_cdn_url(self, uri):
        self.ensure_one()
        if not uri:
            return ''
        cdn_url = self.cdn_url
        cdn_filters = (self.cdn_filters or '').splitlines()
        for flt in cdn_filters:
            if flt and re.match(flt, uri):
                return urls.url_join(cdn_url, uri)
        return uri

    @api.model
    def action_dashboard_redirect(self):
        if self.env.user.has_group('base.group_system') or self.env.user.has_group('website.group_website_designer'):
            return self.env.ref('website.backend_dashboard').read()[0]
        return self.env.ref('website.action_website').read()[0]

    def button_go_website(self):
        self._force()
        return {
            'type': 'ir.actions.act_url',
            'url': '/',
            'target': 'self',
        }

    def _get_http_domain(self):
        """Get the domain of the current website, prefixed by http if no
        scheme is specified.

        Empty string if no domain is specified on the website.
        """
        self.ensure_one()
        if not self.domain:
            return ''
        res = urls.url_parse(self.domain)
        return 'http://' + self.domain if not res.scheme else self.domain

    def get_base_url(self):
        self.ensure_one()
        return self._get_http_domain() or super(BaseModel, self).get_base_url()

    def _get_canonical_url_localized(self, lang, canonical_params):
        """Returns the canonical URL for the current request with translatable
        elements appropriately translated in `lang`.

        If `request.endpoint` is not true, returns the current `path` instead.

        `url_quote_plus` is applied on the returned path.
        """
        self.ensure_one()
        if request.endpoint:
            router = http.root.get_db_router(request.db).bind('')
            arguments = dict(request.endpoint_arguments)
            for key, val in list(arguments.items()):
                if isinstance(val, models.BaseModel):
                    if val.env.context.get('lang') != lang.code:
                        arguments[key] = val.with_context(lang=lang.code)
            path = router.build(request.endpoint, arguments)
        else:
            # The build method returns a quoted URL so convert in this case for consistency.
            path = urls.url_quote_plus(request.httprequest.path, safe='/')
        lang_path = ('/' + lang.url_code) if lang != self.default_lang_id else ''
        canonical_query_string = '?%s' % urls.url_encode(canonical_params) if canonical_params else ''
        return self.get_base_url() + lang_path + path + canonical_query_string

    def _get_canonical_url(self, canonical_params):
        """Returns the canonical URL for the current request."""
        self.ensure_one()
        return self._get_canonical_url_localized(lang=request.lang, canonical_params=canonical_params)

    def _is_canonical_url(self, canonical_params):
        """Returns whether the current request URL is canonical."""
        self.ensure_one()
        # Compare OrderedMultiDict because the order is important, there must be
        # only one canonical and not params permutations.
        params = request.httprequest.args
        canonical_params = canonical_params or OrderedMultiDict()
        if params != canonical_params:
            return False
        # Compare URL at the first rerouting iteration (if available) because
        # it's the one with the language in the path.
        # It is important to also test the domain of the current URL.
        current_url = request.httprequest.url_root[:-1] + (hasattr(request, 'rerouting') and request.rerouting[0] or request.httprequest.path)
        canonical_url = self._get_canonical_url_localized(lang=request.lang, canonical_params=None)
        # A request path with quotable characters (such as ",") is never
        # canonical because request.httprequest.base_url is always unquoted,
        # and canonical url is always quoted, so it is never possible to tell
        # if the current URL is indeed canonical or not.
        return current_url == canonical_url

    def _get_relative_url(self, url):
        return urls.url_parse(url).replace(scheme='', netloc='').to_url()


class BaseModel(models.AbstractModel):
    _inherit = 'base'

    def get_base_url(self):
        """
        Returns baseurl about one given record.
        If a website_id field exists in the current record we use the url
        from this website as base url.

        :return: the base url for this record
        :rtype: string

        """
        self.ensure_one()
        if 'website_id' in self and self.website_id.domain:
            return self.website_id._get_http_domain()
        else:
            return super(BaseModel, self).get_base_url()

```

## File: models\website_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import html_translate


class Menu(models.Model):

    _name = "website.menu"
    _description = "Website Menu"

    _parent_store = True
    _order = "sequence, id"

    def _default_sequence(self):
        menu = self.search([], limit=1, order="sequence DESC")
        return menu.sequence or 0

    def _compute_field_is_mega_menu(self):
        for menu in self:
            menu.is_mega_menu = bool(menu.mega_menu_content)

    def _set_field_is_mega_menu(self):
        for menu in self:
            if menu.is_mega_menu:
                if not menu.mega_menu_content:
                    default_content = self.env['ir.ui.view'].render_template('website.s_mega_menu_multi_menus')
                    menu.mega_menu_content = default_content.decode()
            else:
                menu.mega_menu_content = False
                menu.mega_menu_classes = False

    name = fields.Char('Menu', required=True, translate=True)
    url = fields.Char('Url', default='')
    page_id = fields.Many2one('website.page', 'Related Page', ondelete='cascade')
    new_window = fields.Boolean('New Window')
    sequence = fields.Integer(default=_default_sequence)
    website_id = fields.Many2one('website', 'Website', ondelete='cascade')
    parent_id = fields.Many2one('website.menu', 'Parent Menu', index=True, ondelete="cascade")
    child_id = fields.One2many('website.menu', 'parent_id', string='Child Menus')
    parent_path = fields.Char(index=True)
    is_visible = fields.Boolean(compute='_compute_visible', string='Is Visible')
    group_ids = fields.Many2many('res.groups', string='Visible Groups',
                                 help="User need to be at least in one of these groups to see the menu")
    is_mega_menu = fields.Boolean(compute=_compute_field_is_mega_menu, inverse=_set_field_is_mega_menu)
    mega_menu_content = fields.Html(translate=html_translate, sanitize=False, prefetch=True)
    mega_menu_classes = fields.Char()

    def name_get(self):
        if not self._context.get('display_website') and not self.env.user.has_group('website.group_multi_website'):
            return super(Menu, self).name_get()

        res = []
        for menu in self:
            menu_name = menu.name
            if menu.website_id:
                menu_name += ' [%s]' % menu.website_id.name
            res.append((menu.id, menu_name))
        return res

    @api.model
    def create(self, vals):
        ''' In case a menu without a website_id is trying to be created, we duplicate
            it for every website.
            Note: Particulary useful when installing a module that adds a menu like
                  /shop. So every website has the shop menu.
                  Be careful to return correct record for ir.model.data xml_id in case
                  of default main menus creation.
        '''
        self.clear_caches()
        # Only used when creating website_data.xml default menu
        if vals.get('url') == '/default-main-menu':
            return super(Menu, self).create(vals)

        if 'website_id' in vals:
            return super(Menu, self).create(vals)
        elif self._context.get('website_id'):
            vals['website_id'] = self._context.get('website_id')
            return super(Menu, self).create(vals)
        else:
            # create for every site
            for website in self.env['website'].search([]):
                w_vals = dict(vals, **{
                    'website_id': website.id,
                    'parent_id': website.menu_id.id,
                })
                res = super(Menu, self).create(w_vals)
            # if creating a default menu, we should also save it as such
            default_menu = self.env.ref('website.main_menu', raise_if_not_found=False)
            if default_menu and vals.get('parent_id') == default_menu.id:
                res = super(Menu, self).create(vals)
        return res  # Only one record is returned but multiple could have been created

    def write(self, values):
        res = super().write(values)
        if 'website_id' in values or 'group_ids' in values or 'sequence' in values:
            self.clear_caches()
        return res

    def unlink(self):
        self.clear_caches()
        default_menu = self.env.ref('website.main_menu', raise_if_not_found=False)
        menus_to_remove = self
        for menu in self.filtered(lambda m: default_menu and m.parent_id.id == default_menu.id):
            menus_to_remove |= self.env['website.menu'].search([('url', '=', menu.url),
                                                                ('website_id', '!=', False),
                                                                ('id', '!=', menu.id)])
        return super(Menu, menus_to_remove).unlink()

    def _compute_visible(self):
        for menu in self:
            visible = True
            if menu.page_id and not menu.page_id.sudo().is_visible and not menu.user_has_groups('base.group_user'):
                visible = False
            menu.is_visible = visible

    @api.model
    def clean_url(self):
        # clean the url with heuristic
        if self.page_id:
            url = self.page_id.sudo().url
        else:
            url = self.url
            if url and not self.url.startswith('/'):
                if '@' in self.url:
                    if not self.url.startswith('mailto'):
                        url = 'mailto:%s' % self.url
                elif not self.url.startswith('http'):
                    url = '/%s' % self.url
        return url

    # would be better to take a menu_id as argument
    @api.model
    def get_tree(self, website_id, menu_id=None):
        def make_tree(node):
            is_homepage = bool(node.page_id and self.env['website'].browse(website_id).homepage_id.id == node.page_id.id)
            menu_node = {
                'fields': {
                    'id': node.id,
                    'name': node.name,
                    'url': node.page_id.url if node.page_id else node.url,
                    'new_window': node.new_window,
                    'is_mega_menu': node.is_mega_menu,
                    'sequence': node.sequence,
                    'parent_id': node.parent_id.id,
                },
                'children': [],
                'is_homepage': is_homepage,
            }
            for child in node.child_id:
                menu_node['children'].append(make_tree(child))
            return menu_node

        menu = menu_id and self.browse(menu_id) or self.env['website'].browse(website_id).menu_id
        return make_tree(menu)

    @api.model
    def save(self, website_id, data):
        def replace_id(old_id, new_id):
            for menu in data['data']:
                if menu['id'] == old_id:
                    menu['id'] = new_id
                if menu['parent_id'] == old_id:
                    menu['parent_id'] = new_id
        to_delete = data['to_delete']
        if to_delete:
            self.browse(to_delete).unlink()
        for menu in data['data']:
            mid = menu['id']
            # new menu are prefixed by new-
            if isinstance(mid, str):
                new_menu = self.create({'name': menu['name'], 'website_id': website_id})
                replace_id(mid, new_menu.id)
        for menu in data['data']:
            menu_id = self.browse(menu['id'])
            # if the url match a website.page, set the m2o relation
            # except if the menu url is '#', meaning it will be used as a menu container, most likely for a dropdown
            if menu['url'] == '#':
                if menu_id.page_id:
                    menu_id.page_id = None
            else:
                domain = self.env["website"].website_domain(website_id) + [
                    "|",
                    ("url", "=", menu["url"]),
                    ("url", "=", "/" + menu["url"]),
                ]
                page = self.env["website.page"].search(domain, limit=1)
                if page:
                    menu['page_id'] = page.id
                    menu['url'] = page.url
                elif menu_id.page_id:
                    menu_id.page_id.write({'url': menu['url']})
            menu_id.write(menu)

        return True

```

## File: models\website_page.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.http_routing.models.ir_http import slugify
from odoo import api, fields, models


class Page(models.Model):
    _name = 'website.page'
    _inherits = {'ir.ui.view': 'view_id'}
    _inherit = 'website.published.multi.mixin'
    _description = 'Page'
    _order = 'website_id'

    url = fields.Char('Page URL')
    view_id = fields.Many2one('ir.ui.view', string='View', required=True, ondelete="cascade")
    website_indexed = fields.Boolean('Page Indexed', default=True)
    date_publish = fields.Datetime('Publishing Date')
    # This is needed to be able to display if page is a menu in /website/pages
    menu_ids = fields.One2many('website.menu', 'page_id', 'Related Menus')
    is_homepage = fields.Boolean(compute='_compute_homepage', inverse='_set_homepage', string='Homepage')
    is_visible = fields.Boolean(compute='_compute_visible', string='Is Visible')

    # Page options
    header_overlay = fields.Boolean()
    header_color = fields.Char()

    # don't use mixin website_id but use website_id on ir.ui.view instead
    website_id = fields.Many2one(related='view_id.website_id', store=True, readonly=False, ondelete='cascade')
    arch = fields.Text(related='view_id.arch', readonly=False, depends_context=('website_id',))

    def _compute_homepage(self):
        for page in self:
            page.is_homepage = page == self.env['website'].get_current_website().homepage_id

    def _set_homepage(self):
        for page in self:
            website = self.env['website'].get_current_website()
            if page.is_homepage:
                if website.homepage_id != page:
                    website.write({'homepage_id': page.id})
            else:
                if website.homepage_id == page:
                    website.write({'homepage_id': None})

    def _compute_visible(self):
        for page in self:
            page.is_visible = page.website_published and (
                not page.date_publish or page.date_publish < fields.Datetime.now()
            )

    def _is_most_specific_page(self, page_to_test):
        '''This will test if page_to_test is the most specific page in self.'''
        pages_for_url = self.sorted(key=lambda p: not p.website_id).filtered(lambda page: page.url == page_to_test.url)

        # this works because pages are _order'ed by website_id
        most_specific_page = pages_for_url[0]

        return most_specific_page == page_to_test

    @api.model
    def get_page_info(self, id):
        return self.browse(id).read(
            ['id', 'name', 'url', 'website_published', 'website_indexed', 'date_publish', 'menu_ids', 'is_homepage', 'website_id'],
        )

    def get_view_identifier(self):
        """ Get identifier of this page view that may be used to render it """
        return self.view_id.id

    @api.model
    def save_page_info(self, website_id, data):
        website = self.env['website'].browse(website_id)
        page = self.browse(int(data['id']))

        # If URL has been edited, slug it
        original_url = page.url
        url = data['url']
        if not url.startswith('/'):
            url = '/' + url
        if page.url != url:
            url = '/' + slugify(url, max_length=1024, path=True)
            url = self.env['website'].get_unique_path(url)

        # If name has changed, check for key uniqueness
        if page.name != data['name']:
            page_key = self.env['website'].get_unique_key(slugify(data['name']))
        else:
            page_key = page.key

        menu = self.env['website.menu'].search([('page_id', '=', int(data['id']))])
        if not data['is_menu']:
            # If the page is no longer in menu, we should remove its website_menu
            if menu:
                menu.unlink()
        else:
            # The page is now a menu, check if has already one
            if menu:
                menu.write({'url': url})
            else:
                self.env['website.menu'].create({
                    'name': data['name'],
                    'url': url,
                    'page_id': data['id'],
                    'parent_id': website.menu_id.id,
                    'website_id': website.id,
                })

        # Edits via the page manager shouldn't trigger the COW
        # mechanism and generate new pages. The user manages page
        # visibility manually with is_published here.
        w_vals = {
            'key': page_key,
            'name': data['name'],
            'url': url,
            'is_published': data['website_published'],
            'website_indexed': data['website_indexed'],
            'date_publish': data['date_publish'] or None,
            'is_homepage': data['is_homepage'],
        }
        page.with_context(no_cow=True).write(w_vals)

        # Create redirect if needed
        if data['create_redirect']:
            self.env['website.rewrite'].create({
                'name': data['name'],
                'redirect_type': data['redirect_type'],
                'url_from': original_url,
                'url_to': url,
                'website_id': website.id,
            })

        return url

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        if default:
            if not default.get('view_id'):
                view = self.env['ir.ui.view'].browse(self.view_id.id)
                new_view = view.copy({'website_id': default.get('website_id')})
                default['view_id'] = new_view.id

            default['url'] = default.get('url', self.env['website'].get_unique_path(self.url))
        return super(Page, self).copy(default=default)

    @api.model
    def clone_page(self, page_id, clone_menu=True):
        """ Clone a page, given its identifier
            :param page_id : website.page identifier
        """
        page = self.browse(int(page_id))
        new_page = page.copy(dict(name=page.name, website_id=self.env['website'].get_current_website().id))
        # Should not clone menu if the page was cloned from one website to another
        # Eg: Cloning a generic page (no website) will create a page with a website, we can't clone menu (not same container)
        if clone_menu and new_page.website_id == page.website_id:
            menu = self.env['website.menu'].search([('page_id', '=', page_id)], limit=1)
            if menu:
                # If the page being cloned has a menu, clone it too
                menu.copy({'url': new_page.url, 'name': menu.name, 'page_id': new_page.id})

        return new_page.url + '?enable_editor=1'

    def unlink(self):
        # When a website_page is deleted, the ORM does not delete its
        # ir_ui_view. So we got to delete it ourself, but only if the
        # ir_ui_view is not used by another website_page.
        for page in self:
            # Other pages linked to the ir_ui_view of the page being deleted (will it even be possible?)
            pages_linked_to_iruiview = self.search(
                [('view_id', '=', page.view_id.id), ('id', '!=', page.id)]
            )
            if not pages_linked_to_iruiview and not page.view_id.inherit_children_ids:
                # If there is no other pages linked to that ir_ui_view, we can delete the ir_ui_view
                page.view_id.unlink()
        return super(Page, self).unlink()

    def write(self, vals):
        if 'url' in vals and not vals['url'].startswith('/'):
            vals['url'] = '/' + vals['url']
        return super(Page, self).write(vals)

    def get_website_meta(self):
        self.ensure_one()
        return self.view_id.get_website_meta()

```

## File: models\website_rewrite.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
import werkzeug

from odoo import models, fields, api, _
from odoo.exceptions import AccessDenied, ValidationError

import logging
_logger = logging.getLogger(__name__)


class WebsiteRoute(models.Model):
    _rec_name = 'path'
    _name = 'website.route'
    _description = "All Website Route"
    _order = 'path'

    path = fields.Char('Route')

    @api.model
    def _name_search(self, name='', args=None, operator='ilike', limit=100, name_get_uid=None):
        res = super(WebsiteRoute, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)
        if not len(res):
            self._refresh()
            return super(WebsiteRoute, self)._name_search(name=name, args=args, operator=operator, limit=limit, name_get_uid=name_get_uid)
        return res

    def _refresh(self):
        _logger.debug("Refreshing website.route")
        ir_http = self.env['ir.http']
        tocreate = []
        paths = {rec.path: rec for rec in self.search([])}
        for url, _, routing in ir_http._generate_routing_rules(self.pool._init_modules, converters=ir_http._get_converters()):
            if 'GET' in (routing.get('methods') or ['GET']):
                if paths.get(url):
                    paths.pop(url)
                else:
                    tocreate.append({'path': url})

        if tocreate:
            _logger.info("Add %d website.route" % len(tocreate))
            self.create(tocreate)

        if paths:
            find = self.search([('path', 'in', list(paths.keys()))])
            _logger.info("Delete %d website.route" % len(find))
            find.unlink()


class WebsiteRewrite(models.Model):
    _name = 'website.rewrite'
    _description = "Website rewrite"

    name = fields.Char('Name', required=True)
    website_id = fields.Many2one('website', string="Website", ondelete='cascade', index=True)
    active = fields.Boolean(default=True)
    url_from = fields.Char('URL from', index=True)
    route_id = fields.Many2one('website.route')
    url_to = fields.Char("URL to")
    redirect_type = fields.Selection([
        ('404', '404 Not Found'),
        ('301', '301 Moved permanently'),
        ('302', '302 Moved temporarily'),
        ('308', '308 Redirect / Rewrite'),
    ], string='Action', default="302",
        help='''Type of redirect/Rewrite:\n
        301 Moved permanently: The browser will keep in cache the new url.
        302 Moved temporarily: The browser will not keep in cache the new url and ask again the next time the new url.
        404 Not Found: If you want remove a specific page/controller (e.g. Ecommerce is installed, but you don't want /shop on a specific website)
        308 Redirect / Rewrite: If you want rename a controller with a new url. (Eg: /shop -> /garden - Both url will be accessible but /shop will automatically be redirected to /garden)
    ''')

    sequence = fields.Integer()

    @api.onchange('route_id')
    def _onchange_route_id(self):
        self.url_from = self.route_id.path
        self.url_to = self.route_id.path

    @api.constrains('url_to', 'url_from', 'redirect_type')
    def _check_url_to(self):
        for rewrite in self:
            if rewrite.redirect_type == '308':
                if not rewrite.url_to:
                    raise ValidationError(_('"URL to" can not be empty.'))
                elif not rewrite.url_to.startswith('/'):
                    raise ValidationError(_('"URL to" must start with a leading slash.'))
                for param in re.findall('/<.*?>', rewrite.url_from):
                    if param not in rewrite.url_to:
                        raise ValidationError(_('"URL to" must contain parameter %s used in "URL from".') % param)
                for param in re.findall('/<.*?>', rewrite.url_to):
                    if param not in rewrite.url_from:
                        raise ValidationError(_('"URL to" cannot contain parameter %s which is not used in "URL from".') % param)
                try:
                    converters = self.env['ir.http']._get_converters()
                    routing_map = werkzeug.routing.Map(strict_slashes=False, converters=converters)
                    rule = werkzeug.routing.Rule(rewrite.url_to)
                    routing_map.add(rule)
                except ValueError as e:
                    raise ValidationError(_('"URL to" is invalid: %s') % e)

    def name_get(self):
        result = []
        for rewrite in self:
            name = "%s - %s" % (rewrite.redirect_type, rewrite.name)
            result.append((rewrite.id, name))
        return result

    @api.model
    def create(self, vals):
        res = super(WebsiteRewrite, self).create(vals)
        self._invalidate_routing()
        return res

    def write(self, vals):
        res = super(WebsiteRewrite, self).write(vals)
        self._invalidate_routing()
        return res

    def unlink(self):
        res = super(WebsiteRewrite, self).unlink()
        self._invalidate_routing()
        return res

    def _invalidate_routing(self):
        # call clear_caches on this worker to reload routing table
        self.env['ir.http'].clear_caches()

    def refresh_routes(self):
        self.env['website.route']._refresh()

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
import uuid

from odoo import fields, models, api, registry, _
from odoo.addons.base.models.res_partner import _tz_get
from odoo.exceptions import UserError
from odoo.tools.misc import _format_time_ago
from odoo.http import request
from odoo.osv import expression


class WebsiteTrack(models.Model):
    _name = 'website.track'
    _description = 'Visited Pages'
    _order = 'visit_datetime DESC'
    _log_access = False

    visitor_id = fields.Many2one('website.visitor', ondelete="cascade", index=True, required=True, readonly=True)
    page_id = fields.Many2one('website.page', index=True, ondelete='cascade', readonly=True)
    url = fields.Text('Url', index=True)
    visit_datetime = fields.Datetime('Visit Date', default=fields.Datetime.now, required=True, readonly=True)


class WebsiteVisitor(models.Model):
    _name = 'website.visitor'
    _description = 'Website Visitor'
    _order = 'last_connection_datetime DESC'

    name = fields.Char('Name')
    access_token = fields.Char(required=True, default=lambda x: uuid.uuid4().hex, index=False, copy=False, groups='base.group_website_publisher')
    active = fields.Boolean('Active', default=True)
    website_id = fields.Many2one('website', "Website", readonly=True)
    partner_id = fields.Many2one('res.partner', string="Linked Partner", help="Partner of the last logged in user.")
    partner_image = fields.Binary(related='partner_id.image_1920')

    # localisation and info
    country_id = fields.Many2one('res.country', 'Country', readonly=True)
    country_flag = fields.Binary(related="country_id.image", string="Country Flag")
    lang_id = fields.Many2one('res.lang', string='Language', help="Language from the website when visitor has been created")
    timezone = fields.Selection(_tz_get, string='Timezone')
    email = fields.Char(string='Email', compute='_compute_email_phone')
    mobile = fields.Char(string='Mobile Phone', compute='_compute_email_phone')

    # Visit fields
    visit_count = fields.Integer('Number of visits', default=1, readonly=True, help="A new visit is considered if last connection was more than 8 hours ago.")
    website_track_ids = fields.One2many('website.track', 'visitor_id', string='Visited Pages History', readonly=True)
    visitor_page_count = fields.Integer('Page Views', compute="_compute_page_statistics", help="Total number of visits on tracked pages")
    page_ids = fields.Many2many('website.page', string="Visited Pages", compute="_compute_page_statistics", search="_search_page_ids")
    page_count = fields.Integer('# Visited Pages', compute="_compute_page_statistics", help="Total number of tracked page visited")
    last_visited_page_id = fields.Many2one('website.page', string="Last Visited Page", compute="_compute_last_visited_page_id")

    # Time fields
    create_date = fields.Datetime('First connection date', readonly=True)
    last_connection_datetime = fields.Datetime('Last Connection', default=fields.Datetime.now, help="Last page view date", readonly=True)
    time_since_last_action = fields.Char('Last action', compute="_compute_time_statistics", help='Time since last page view. E.g.: 2 minutes ago')
    is_connected = fields.Boolean('Is connected ?', compute='_compute_time_statistics', help='A visitor is considered as connected if his last page view was within the last 5 minutes.')

    _sql_constraints = [
        ('access_token_unique', 'unique(access_token)', 'Access token should be unique.'),
        ('partner_uniq', 'unique(partner_id)', 'A partner is linked to only one visitor.'),
    ]

    @api.depends('name')
    def name_get(self):
        res = []
        for record in self:
            res.append((
                record.id,
                record.name or _('Website Visitor #%s') % record.id
            ))
        return res

    @api.depends('partner_id.email_normalized', 'partner_id.mobile', 'partner_id.phone')
    def _compute_email_phone(self):
        results = self.env['res.partner'].search_read(
            [('id', 'in', self.partner_id.ids)],
            ['id', 'email_normalized', 'mobile', 'phone'],
        )
        mapped_data = {
            result['id']: {
                'email_normalized': result['email_normalized'],
                'mobile': result['mobile'] if result['mobile'] else result['phone']
            } for result in results
        }

        for visitor in self:
            visitor.email = mapped_data.get(visitor.partner_id.id, {}).get('email_normalized')
            visitor.mobile = mapped_data.get(visitor.partner_id.id, {}).get('mobile')

    @api.depends('website_track_ids')
    def _compute_page_statistics(self):
        results = self.env['website.track'].read_group(
            [('visitor_id', 'in', self.ids), ('url', '!=', False)], ['visitor_id', 'page_id', 'url'], ['visitor_id', 'page_id', 'url'], lazy=False)
        mapped_data = {}
        for result in results:
            visitor_info = mapped_data.get(result['visitor_id'][0], {'page_count': 0, 'visitor_page_count': 0, 'page_ids': set()})
            visitor_info['visitor_page_count'] += result['__count']
            visitor_info['page_count'] += 1
            if result['page_id']:
                visitor_info['page_ids'].add(result['page_id'][0])
            mapped_data[result['visitor_id'][0]] = visitor_info

        for visitor in self:
            visitor_info = mapped_data.get(visitor.id, {'page_count': 0, 'visitor_page_count': 0, 'page_ids': set()})
            visitor.page_ids = [(6, 0, visitor_info['page_ids'])]
            visitor.visitor_page_count = visitor_info['visitor_page_count']
            visitor.page_count = visitor_info['page_count']

    def _search_page_ids(self, operator, value):
        if operator not in ('like', 'ilike', 'not like', 'not ilike', '=like', '=ilike', '=', '!='):
            raise ValueError(_('This operator is not supported'))
        return [('website_track_ids.page_id.name', operator, value)]

    @api.depends('website_track_ids.page_id')
    def _compute_last_visited_page_id(self):
        results = self.env['website.track'].read_group([('visitor_id', 'in', self.ids)],
                                                       ['visitor_id', 'page_id', 'visit_datetime:max'],
                                                       ['visitor_id', 'page_id'], lazy=False)
        mapped_data = {result['visitor_id'][0]: result['page_id'][0] for result in results if result['page_id']}
        for visitor in self:
            visitor.last_visited_page_id = mapped_data.get(visitor.id, False)

    @api.depends('last_connection_datetime')
    def _compute_time_statistics(self):
        for visitor in self:
            visitor.time_since_last_action = _format_time_ago(self.env, (datetime.now() - visitor.last_connection_datetime))
            visitor.is_connected = (datetime.now() - visitor.last_connection_datetime) < timedelta(minutes=5)

    def _prepare_visitor_send_mail_values(self):
        if self.partner_id.email:
            return {
                'res_model': 'res.partner',
                'res_id': self.partner_id.id,
                'partner_ids': [self.partner_id.id],
            }
        return {}

    def action_send_mail(self):
        self.ensure_one()
        visitor_mail_values = self._prepare_visitor_send_mail_values()
        if not visitor_mail_values:
            raise UserError(_("There is no email linked this visitor."))
        compose_form = self.env.ref('mail.email_compose_message_wizard_form', False)
        ctx = dict(
            default_model=visitor_mail_values.get('res_model'),
            default_res_id=visitor_mail_values.get('res_id'),
            default_use_template=False,
            default_partner_ids=[(6, 0, visitor_mail_values.get('partner_ids'))],
            default_composition_mode='comment',
            default_reply_to=self.env.user.partner_id.email,
        )
        return {
            'name': _('Compose Email'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form.id, 'form')],
            'view_id': compose_form.id,
            'target': 'new',
            'context': ctx,
        }

    def _get_visitor_from_request(self, force_create=False):
        """ Return the visitor as sudo from the request if there is a visitor_uuid cookie.
            It is possible that the partner has changed or has disconnected.
            In that case the cookie is still referencing the old visitor and need to be replaced
            with the one of the visitor returned !!!. """

        # This function can be called in json with mobile app.
        # In case of mobile app, no uid is set on the jsonRequest env.
        # In case of multi db, _env is None on request, and request.env unbound.
        if not request:
            return None
        Visitor = self.env['website.visitor'].sudo()
        visitor = Visitor
        access_token = request.httprequest.cookies.get('visitor_uuid')
        if access_token:
            visitor = Visitor.with_context(active_test=False).search([('access_token', '=', access_token)])
            # Prefetch access_token and other fields. Since access_token has a restricted group and we access
            # a non restricted field (partner_id) first it is not fetched and will require an additional query to be retrieved.
            visitor.access_token

        if not self.env.user._is_public():
            partner_id = self.env.user.partner_id
            if not visitor or visitor.partner_id and visitor.partner_id != partner_id:
                # Partner and no cookie or wrong cookie
                visitor = Visitor.with_context(active_test=False).search([('partner_id', '=', partner_id.id)])
        elif visitor and visitor.partner_id:
            # Cookie associated to a Partner
            visitor = Visitor

        if force_create and not visitor:
            visitor = self._create_visitor()

        return visitor

    def _handle_webpage_dispatch(self, response, website_page):
        # get visitor. Done here to avoid having to do it multiple times in case of override.
        visitor_sudo = self._get_visitor_from_request(force_create=True)
        if request.httprequest.cookies.get('visitor_uuid', '') != visitor_sudo.access_token:
            expiration_date = datetime.now() + timedelta(days=365)
            response.set_cookie('visitor_uuid', visitor_sudo.access_token, expires=expiration_date)
        self._handle_website_page_visit(response, website_page, visitor_sudo)

    def _handle_website_page_visit(self, response, website_page, visitor_sudo):
        """ Called on dispatch. This will create a website.visitor if the http request object
        is a tracked website page or a tracked view. Only on tracked elements to avoid having
        too much operations done on every page or other http requests.
        Note: The side effect is that the last_connection_datetime is updated ONLY on tracked elements."""
        url = request.httprequest.url
        website_track_values = {
            'url': url,
            'visit_datetime': datetime.now(),
        }
        if website_page:
            website_track_values['page_id'] = website_page.id
            domain = [('page_id', '=', website_page.id)]
        else:
            domain = [('url', '=', url)]
        visitor_sudo._add_tracking(domain, website_track_values)
        if visitor_sudo.lang_id.id != request.lang.id:
            visitor_sudo.write({'lang_id': request.lang.id})

    def _add_tracking(self, domain, website_track_values):
        """ Add the track and update the visitor"""
        domain = expression.AND([domain, [('visitor_id', '=', self.id)]])
        last_view = self.env['website.track'].sudo().search(domain, limit=1)
        if not last_view or last_view.visit_datetime < datetime.now() - timedelta(minutes=30):
            website_track_values['visitor_id'] = self.id
            self.env['website.track'].create(website_track_values)
        self._update_visitor_last_visit()

    def _create_visitor(self, website_track_values=None):
        """ Create a visitor and add a track to it if website_track_values is set."""
        country_code = request.session.get('geoip', {}).get('country_code', False)
        country_id = request.env['res.country'].sudo().search([('code', '=', country_code)], limit=1).id if country_code else False
        vals = {
            'lang_id': request.lang.id,
            'country_id': country_id,
            'website_id': request.website.id,
        }
        if not self.env.user._is_public():
            vals['partner_id'] = self.env.user.partner_id.id
            vals['name'] = self.env.user.partner_id.name
        if website_track_values:
            vals['website_track_ids'] = [(0, 0, website_track_values)]
        return self.sudo().create(vals)

    def _cron_archive_visitors(self):
        one_week_ago = datetime.now() - timedelta(days=7)
        visitors_to_archive = self.env['website.visitor'].sudo().search([('last_connection_datetime', '<', one_week_ago)])
        visitors_to_archive.write({'active': False})

    def _update_visitor_last_visit(self):
        """ We need to do this part here to avoid concurrent updates error. """
        try:
            with self.env.cr.savepoint():
                query_lock = "SELECT * FROM website_visitor where id = %s FOR NO KEY UPDATE NOWAIT"
                self.env.cr.execute(query_lock, (self.id,), log_exceptions=False)

                date_now = datetime.now()
                query = "UPDATE website_visitor SET "
                if self.last_connection_datetime < (date_now - timedelta(hours=8)):
                    query += "visit_count = visit_count + 1,"
                query += """
                    active = True,
                    last_connection_datetime = %s
                    WHERE id = %s
                """
                self.env.cr.execute(query, (date_now, self.id), log_exceptions=False)
        except Exception:
            pass

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import assets
from . import ir_actions
from . import ir_attachment
from . import ir_http
from . import ir_module_module
from . import ir_qweb
from . import ir_qweb_fields
from . import mixins
from . import website
from . import website_menu
from . import website_page
from . import website_rewrite
from . import ir_rule
from . import ir_translation
from . import ir_ui_view
from . import res_company
from . import res_partner
from . import res_users
from . import res_config_settings
from . import res_lang
from . import website_visitor

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_website_public,website,website.model_website,,1,0,0,0
access_website_designer,website,website.model_website,group_website_designer,1,1,1,1
access_website_menu,access_website_menu,model_website_menu,,1,0,0,0
access_website_menu_designer,Web Menu Manager,model_website_menu,group_website_designer,1,1,1,1
access_website_rewrite,access_website_rewrite,model_website_rewrite,,0,0,0,0
access_website_rewrite_designer,Web Rewrite Manager,model_website_rewrite,group_website_designer,1,1,1,1
access_website_page,access_website_page,model_website_page,,1,0,0,0
access_website_page_designer,Web Page Manager,model_website_page,group_website_designer,1,1,1,1
access_website,web menu manager,website.model_website,group_website_designer,1,1,1,1
access_website_ir_ui_view,access_website_ir_ui_view,model_ir_ui_view,group_website_designer,1,1,1,1
access_seo_public,access_seo_public,model_website_seo_metadata,,1,0,0,0
access_seo_manager,access_seo_manager,model_website_seo_metadata,group_website_designer,1,1,1,1
access_seo_designer,access_seo_designer,model_website_seo_metadata,group_website_designer,1,1,1,1
access_website_visitor_designer,access_website_visitor_designer,model_website_visitor,website.group_website_designer,1,1,0,1
access_website_track_designer,access_website_track_designer,model_website_track,website.group_website_designer,1,1,1,1
access_website_track_system,access_website_track_system,model_website_track,base.group_system,1,1,1,1
access_website_route_designer,access_website_designer_route,model_website_route,group_website_designer,1,1,1,1

```

## File: security\website_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_website_website">
        <field name="sequence">23</field>
    </record>

    <record id="group_website_publisher" model="res.groups">
        <field name="name">Restricted Editor</field>
        <field name="category_id" ref="base.module_category_website_website"/>
    </record>
    <record id="group_website_designer" model="res.groups">
        <field name="name">Editor and Designer</field>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        <field name="implied_ids" eval="[(4, ref('group_website_publisher'))]"/>
        <field name="category_id" ref="base.module_category_website_website"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4, ref('group_website_designer'))]"/>
    </record>
    <!-- FIXME: groups on existing users should probably be updated when implied_ids is, or existing users don't get the relevant implied groups on module installation... -->
    <record id="base.user_admin" model="res.users">
        <field name="groups_id" eval="[(4, ref('website.group_website_designer'))]"/>
    </record>

    <record id="base.group_system" model="res.groups">
        <field name="implied_ids" eval="[(4, ref('website.group_website_designer'))]"/>
    </record>

    <record id="website_menu" model="ir.rule">
        <field name="name">Website menu: group_ids</field>
        <field name="model_id" ref="model_website_menu"/>
        <field name="domain_force">['|', ('group_ids', '=', False), ('group_ids', 'in', [g.id for g in user.groups_id])]</field>
    </record>

    <record id="website_designer_edit_qweb" model="ir.rule">
        <field name="name">website_designer: Manage Website and qWeb view</field>
        <field name="model_id" ref="base.model_ir_ui_view"/>
        <field name="domain_force">[('type', '=', 'qweb')]</field>
        <field name="groups" eval="[(4, ref('group_website_designer'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="website_designer_view" model="ir.rule">
        <field name="name">website_designer: global view</field>
        <field name="model_id" ref="base.model_ir_ui_view"/>
        <field name="domain_force">[('type', '!=', 'qweb')]</field>
        <field name="groups" eval="[(4, ref('group_website_designer'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>
    <record id="website_group_system_edit_all_views" model="ir.rule">
        <field name="name">Administration Settings: Manage all views</field>
        <field name="model_id" ref="base.model_ir_ui_view"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('base.group_system'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
    <record id="website_page_rule_public" model="ir.rule">
        <field name="name">website.page: portal/public: read published pages</field>
        <field name="model_id" ref="website.model_website_page"/>
        <field name="domain_force">[('website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#DA956B"/><stop offset="100%" stop-color="#CC7039"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-.146-4-4.074V42.409l18.596-21.11L36 14l18 11.204-1.95 22.765L36.782 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M35 15.571c3.888 0 7.473.958 10.756 2.874a21.332 21.332 0 0 1 7.799 7.799c1.916 3.283 2.874 6.868 2.874 10.756s-.958 7.473-2.874 10.756a21.332 21.332 0 0 1-7.799 7.799C42.473 57.47 38.888 58.429 35 58.429s-7.473-.958-10.756-2.874a21.332 21.332 0 0 1-7.799-7.799C14.53 44.473 13.571 40.888 13.571 37s.958-7.473 2.874-10.756a21.332 21.332 0 0 1 7.799-7.799c3.283-1.916 6.868-2.874 10.756-2.874zm7.645 14.537c-.037.019-.125.107-.265.265-.14.158-.265.247-.377.265.038 0 .08-.046.126-.14.046-.092.093-.194.14-.306a.856.856 0 0 1 .097-.195c.112-.13.316-.27.614-.419.26-.111.744-.223 1.45-.335.633-.149 1.108-.046 1.424.307-.037-.037.05-.158.265-.363.214-.204.349-.316.404-.334.056-.038.196-.08.419-.126.223-.046.363-.116.418-.21l.056-.613c-.223.019-.386-.046-.488-.195-.102-.15-.163-.344-.181-.586 0 .037-.056.111-.168.223 0-.13-.042-.205-.125-.223a.671.671 0 0 0-.321.028c-.13.037-.214.046-.251.028a1.095 1.095 0 0 1-.419-.21c-.093-.083-.167-.237-.223-.46a7.936 7.936 0 0 0-.112-.419c-.037-.093-.125-.19-.265-.293-.14-.102-.228-.2-.265-.293a3.026 3.026 0 0 1-.07-.153 3.41 3.41 0 0 0-.083-.181.589.589 0 0 0-.112-.154.213.213 0 0 0-.153-.07c-.056 0-.121.047-.196.14a5.451 5.451 0 0 0-.209.279c-.065.093-.107.14-.125.14a.228.228 0 0 0-.168-.042.89.89 0 0 0-.125.028.47.47 0 0 0-.126.083.78.78 0 0 1-.14.098.637.637 0 0 1-.237.084 1.824 1.824 0 0 0-.237.055c.28-.093.27-.195-.028-.307-.186-.074-.335-.102-.446-.083.167-.075.237-.186.21-.335a.671.671 0 0 0-.238-.39h.14c-.02-.075-.098-.154-.238-.238a3.42 3.42 0 0 0-.488-.237 2.402 2.402 0 0 1-.363-.167c-.149-.093-.465-.182-.948-.265-.484-.084-.79-.089-.921-.014-.093.111-.135.209-.126.293.01.083.047.213.112.39.065.177.098.293.098.349.018.112-.033.232-.154.363-.12.13-.181.241-.181.334 0 .13.13.275.39.433.26.158.354.358.28.6-.056.149-.205.297-.447.446-.242.15-.39.26-.446.335-.093.149-.107.321-.042.516.065.196.162.349.293.46.037.038.05.075.042.112-.01.037-.042.08-.098.126a1.249 1.249 0 0 1-.154.111c-.046.028-.107.06-.181.098l-.084.056c-.204.093-.395.037-.572-.168a1.804 1.804 0 0 1-.376-.725c-.13-.465-.28-.744-.447-.837-.428-.149-.697-.14-.809.028-.093-.242-.474-.484-1.144-.726-.465-.167-1.004-.204-1.618-.111.111-.019.111-.158 0-.419-.13-.279-.307-.39-.53-.335a1.4 1.4 0 0 0 .111-.488c.019-.214.028-.34.028-.376.056-.242.168-.456.335-.642a5.35 5.35 0 0 0 .46-.614c.065-.112.07-.167.014-.167.651.074 1.116-.028 1.395-.307.093-.093.2-.252.321-.475.121-.223.219-.381.293-.474.168-.112.298-.163.39-.154.094.01.229.06.405.154.177.093.312.14.405.14.26.018.405-.084.433-.307a.606.606 0 0 0-.21-.559c.223.02.251-.139.084-.474a1.065 1.065 0 0 0-.223-.251c-.223-.074-.475-.028-.754.14-.148.074-.13.148.056.223-.018-.019-.107.079-.265.293a2.36 2.36 0 0 1-.46.488c-.15.112-.298.065-.447-.14-.018-.018-.07-.144-.153-.376-.084-.233-.172-.358-.265-.377-.149 0-.298.14-.447.419.056-.15-.046-.289-.307-.419-.26-.13-.483-.204-.67-.223.354-.223.28-.474-.222-.753-.13-.075-.321-.121-.572-.14-.252-.019-.433.019-.544.112-.093.13-.145.237-.154.32-.01.084.037.159.14.224.102.065.2.116.293.153.093.037.2.075.32.112.121.037.2.065.238.084.26.186.334.316.223.39a3.374 3.374 0 0 1-.237.098c-.121.046-.228.088-.321.125-.093.038-.15.075-.168.112-.055.074-.055.205 0 .39.056.187.038.317-.055.391-.093-.093-.177-.256-.252-.488-.074-.233-.14-.386-.195-.46.13.167-.102.223-.697.167l-.28-.028c-.074 0-.223.019-.446.056a2.073 2.073 0 0 1-.572.028.519.519 0 0 1-.377-.223c-.074-.15-.074-.335 0-.558.02-.075.056-.093.112-.056a4.18 4.18 0 0 1-.307-.265c-.13-.121-.223-.2-.279-.237-.856.279-1.73.66-2.623 1.143.112.02.224.01.335-.027.093-.038.214-.098.363-.182a9.41 9.41 0 0 1 .279-.153c.632-.26 1.023-.326 1.172-.196l.14-.14c.26.299.446.531.557.698-.13-.074-.409-.083-.837-.027-.372.111-.576.223-.614.334.13.224.177.391.14.503a4.358 4.358 0 0 1-.32-.28 2.213 2.213 0 0 0-.405-.306c-.13-.075-.27-.121-.419-.14-.298 0-.502.01-.614.028-2.716 1.488-4.901 3.553-6.557 6.194.13.13.242.205.335.224.074.018.121.102.14.25.018.15.041.252.07.308.027.055.134.027.32-.084.168.149.195.325.084.53.018-.019.428.232 1.228.753.353.317.548.512.586.586.055.205-.038.372-.28.502-.018-.037-.102-.12-.25-.25-.15-.13-.233-.168-.252-.112-.056.093-.05.265.014.516.065.251.163.367.293.349-.13 0-.218.149-.265.446-.046.298-.07.628-.07.99 0 .363-.009.582-.028.656l.056.028c-.056.223-.004.544.154.963.158.418.358.6.6.544-.242.056-.056.456.558 1.2.111.149.186.232.223.25.056.038.167.108.335.21.167.102.307.195.418.28.112.083.205.18.28.292.074.093.167.302.278.628.112.325.242.544.39.656-.036.111.052.297.266.558.214.26.312.474.293.641a.136.136 0 0 0-.07.028.136.136 0 0 1-.07.028c.056.13.2.26.433.39.232.131.377.252.432.364.019.055.038.148.056.279.019.13.047.232.084.306.037.075.111.093.223.056.037-.372-.186-.948-.67-1.73a22.925 22.925 0 0 1-.474-.809 1.604 1.604 0 0 1-.153-.432 2.128 2.128 0 0 0-.126-.405c.037 0 .093.014.167.042.075.028.154.06.238.098.083.037.153.074.209.111.056.037.074.065.056.084-.056.13-.037.293.056.488.093.196.204.368.334.516a38.267 38.267 0 0 0 .81.893c.111.112.241.293.39.544.149.251.149.377 0 .377.168 0 .354.093.558.28.205.185.363.371.474.557.093.149.168.39.224.726.056.334.102.558.14.67.036.13.115.255.236.376.121.12.238.21.35.265l.446.223.362.195c.093.038.265.135.516.293a3.6 3.6 0 0 0 .6.321c.186.075.335.112.447.112.111 0 .246-.023.404-.07.158-.046.284-.079.377-.098.28-.037.549.103.81.419.26.316.455.512.585.586.67.353 1.181.456 1.535.307-.037.018-.033.088.014.21.046.12.12.264.223.432a27.356 27.356 0 0 0 .404.641c.093.112.26.251.503.419.242.167.409.307.502.418.112-.074.177-.158.195-.25-.056.148.01.334.196.557.186.224.353.317.502.28.26-.056.39-.354.39-.894-.576.28-1.032.112-1.367-.502a.544.544 0 0 0-.07-.153 1.766 1.766 0 0 1-.181-.474.396.396 0 0 1 0-.21c.019-.056.065-.084.14-.084.167 0 .26-.032.279-.097.018-.065 0-.182-.056-.349a5.625 5.625 0 0 1-.112-.363c-.018-.148-.12-.334-.307-.558a7.435 7.435 0 0 1-.334-.418c-.094.167-.242.242-.447.223-.205-.019-.353-.102-.446-.251a.77.77 0 0 1-.042.153.664.664 0 0 0-.042.182c-.242 0-.381-.01-.419-.028.019-.056.042-.219.07-.488.028-.27.06-.48.098-.628a1.55 1.55 0 0 1 .153-.335 5.76 5.76 0 0 0 .21-.405c.055-.12.093-.237.111-.348.019-.112-.023-.2-.125-.266-.103-.065-.265-.088-.489-.07-.353.02-.595.205-.725.559a5.787 5.787 0 0 0-.084.293.945.945 0 0 1-.14.32.694.694 0 0 1-.25.196c-.13.056-.354.074-.67.056-.316-.019-.54-.065-.67-.14-.242-.149-.45-.418-.628-.809-.176-.39-.265-.735-.265-1.032 0-.186.024-.433.07-.74.047-.307.075-.54.084-.697.01-.158-.042-.386-.154-.684.056-.037.14-.125.252-.265.111-.14.204-.237.279-.293a.498.498 0 0 1 .125-.042.285.285 0 0 1 .126 0c.037.01.074-.004.111-.042a.354.354 0 0 0 .084-.167.831.831 0 0 0-.112-.084c-.055-.056-.093-.083-.111-.083.13.055.395.041.795-.042.4-.084.656-.07.767.042.28.204.484.186.614-.056a2.22 2.22 0 0 0-.07-.265c-.046-.158-.05-.284-.014-.377.093.502.363.586.81.251.055.056.2.102.432.14.233.037.395.083.488.14.056.036.121.088.196.153.074.065.125.107.153.125.028.019.074.014.14-.014a.84.84 0 0 0 .237-.181c.186.26.297.484.335.67.204.744.38 1.153.53 1.227.13.056.232.075.307.056.074-.019.116-.107.125-.265.01-.158.01-.288 0-.39a6.933 6.933 0 0 0-.042-.35l-.028-.223v-.502l-.027-.223c-.28-.056-.452-.168-.517-.335a.599.599 0 0 1 .042-.516c.093-.177.233-.349.419-.516a.91.91 0 0 1 .223-.098c.13-.046.274-.107.432-.181.159-.075.275-.15.35-.224.39-.353.53-.678.418-.976.13 0 .232-.084.307-.251-.019 0-.065-.028-.14-.084a2.04 2.04 0 0 0-.21-.14.379.379 0 0 0-.125-.055c.168-.093.186-.242.056-.447.093-.055.163-.158.21-.307.046-.148.116-.241.209-.279.167.224.362.242.586.056.13-.149.14-.297.027-.446.093-.13.284-.228.572-.293.289-.065.46-.154.517-.265.13.037.204.018.223-.056.018-.074.028-.186.028-.335 0-.149.028-.26.083-.335.075-.093.214-.176.419-.25.205-.075.325-.122.363-.14l.474-.307c.056-.075.056-.112 0-.112a1.01 1.01 0 0 0 .865-.307c.186-.204.13-.39-.167-.558.055-.111.027-.2-.084-.265a1.46 1.46 0 0 0-.419-.153c.056-.019.163-.024.321-.014.158.009.256-.005.293-.042.28-.186.214-.335-.195-.447-.316-.093-.716.019-1.2.335zm-4.548 24.47c3.832-.67 7.096-2.427 9.794-5.273-.056-.056-.172-.098-.35-.126-.176-.028-.292-.06-.348-.098-.335-.13-.558-.204-.67-.223a.547.547 0 0 0-.07-.362.745.745 0 0 0-.223-.252c-.083-.055-.2-.13-.348-.223a14.687 14.687 0 0 1-.307-.195 2.631 2.631 0 0 0-.196-.168 9.374 9.374 0 0 0-.195-.153 1.98 1.98 0 0 0-.21-.126c-.101-.055-.18-.074-.236-.055a.98.98 0 0 1-.28.028l-.083.027a.771.771 0 0 0-.153.07 2.285 2.285 0 0 1-.154.084.288.288 0 0 0-.112.084c-.018.028-.018.05 0 .07-.39-.317-.725-.521-1.004-.614a.788.788 0 0 1-.307-.154 2.65 2.65 0 0 0-.293-.195.408.408 0 0 0-.279-.042c-.102.019-.21.084-.32.195-.094.093-.15.233-.168.419a2.115 2.115 0 0 1-.056.362c-.13-.093-.13-.255 0-.488.13-.232.149-.404.056-.516-.056-.112-.154-.153-.293-.126a.934.934 0 0 0-.335.126 5.36 5.36 0 0 0-.32.237c-.131.102-.215.163-.252.182a2.67 2.67 0 0 0-.237.153.939.939 0 0 0-.237.21 1.385 1.385 0 0 0-.168.334 1.688 1.688 0 0 1-.14.307c-.036-.074-.143-.135-.32-.181-.177-.047-.265-.098-.265-.154.037.186.074.512.111.977.038.465.084.818.14 1.06.13.577.019 1.023-.335 1.34-.502.464-.772.836-.809 1.115-.074.41.037.651.335.726 0 .13-.075.32-.223.572-.15.25-.214.45-.196.6 0 .111.019.26.056.446z" opacity=".3"/><path fill="#FFF" d="M35 13.571c3.888 0 7.473.958 10.756 2.874a21.332 21.332 0 0 1 7.799 7.799c1.916 3.283 2.874 6.868 2.874 10.756s-.958 7.473-2.874 10.756a21.332 21.332 0 0 1-7.799 7.799C42.473 55.47 38.888 56.429 35 56.429s-7.473-.958-10.756-2.874a21.332 21.332 0 0 1-7.799-7.799C14.53 42.473 13.571 38.888 13.571 35s.958-7.473 2.874-10.756a21.332 21.332 0 0 1 7.799-7.799c3.283-1.916 6.868-2.874 10.756-2.874zm7.645 14.537c-.037.019-.125.107-.265.265-.14.158-.265.247-.377.265.038 0 .08-.046.126-.14.046-.092.093-.194.14-.306a.856.856 0 0 1 .097-.195c.112-.13.316-.27.614-.419.26-.111.744-.223 1.45-.335.633-.149 1.108-.046 1.424.307-.037-.037.05-.158.265-.363.214-.204.349-.316.404-.334.056-.038.196-.08.419-.126.223-.046.363-.116.418-.21l.056-.613c-.223.019-.386-.046-.488-.195-.102-.15-.163-.344-.181-.586 0 .037-.056.111-.168.223 0-.13-.042-.205-.125-.223a.671.671 0 0 0-.321.028c-.13.037-.214.046-.251.028a1.095 1.095 0 0 1-.419-.21c-.093-.083-.167-.237-.223-.46a7.936 7.936 0 0 0-.112-.419c-.037-.093-.125-.19-.265-.293-.14-.102-.228-.2-.265-.293a3.026 3.026 0 0 1-.07-.153 3.41 3.41 0 0 0-.083-.181.589.589 0 0 0-.112-.154.213.213 0 0 0-.153-.07c-.056 0-.121.047-.196.14a5.451 5.451 0 0 0-.209.279c-.065.093-.107.14-.125.14a.228.228 0 0 0-.168-.042.89.89 0 0 0-.125.028.47.47 0 0 0-.126.083.78.78 0 0 1-.14.098.637.637 0 0 1-.237.084 1.824 1.824 0 0 0-.237.055c.28-.093.27-.195-.028-.307-.186-.074-.335-.102-.446-.083.167-.075.237-.186.21-.335a.671.671 0 0 0-.238-.39h.14c-.02-.075-.098-.154-.238-.238a3.42 3.42 0 0 0-.488-.237 2.402 2.402 0 0 1-.363-.167c-.149-.093-.465-.182-.948-.265-.484-.084-.79-.089-.921-.014-.093.111-.135.209-.126.293.01.083.047.213.112.39.065.177.098.293.098.349.018.112-.033.232-.154.363-.12.13-.181.241-.181.334 0 .13.13.275.39.433.26.158.354.358.28.6-.056.149-.205.297-.447.446-.242.15-.39.26-.446.335-.093.149-.107.321-.042.516.065.196.162.349.293.46.037.038.05.075.042.112-.01.037-.042.08-.098.126a1.249 1.249 0 0 1-.154.111c-.046.028-.107.06-.181.098l-.084.056c-.204.093-.395.037-.572-.168a1.804 1.804 0 0 1-.376-.725c-.13-.465-.28-.744-.447-.837-.428-.149-.697-.14-.809.028-.093-.242-.474-.484-1.144-.726-.465-.167-1.004-.204-1.618-.111.111-.019.111-.158 0-.419-.13-.279-.307-.39-.53-.335a1.4 1.4 0 0 0 .111-.488c.019-.214.028-.34.028-.376.056-.242.168-.456.335-.642a5.35 5.35 0 0 0 .46-.614c.065-.112.07-.167.014-.167.651.074 1.116-.028 1.395-.307.093-.093.2-.252.321-.475.121-.223.219-.381.293-.474.168-.112.298-.163.39-.154.094.01.229.06.405.154.177.093.312.14.405.14.26.018.405-.084.433-.307a.606.606 0 0 0-.21-.559c.223.02.251-.139.084-.474a1.065 1.065 0 0 0-.223-.251c-.223-.074-.475-.028-.754.14-.148.074-.13.148.056.223-.018-.019-.107.079-.265.293a2.36 2.36 0 0 1-.46.488c-.15.112-.298.065-.447-.14-.018-.018-.07-.144-.153-.376-.084-.233-.172-.358-.265-.377-.149 0-.298.14-.447.419.056-.15-.046-.289-.307-.419-.26-.13-.483-.204-.67-.223.354-.223.28-.474-.222-.753-.13-.075-.321-.121-.572-.14-.252-.019-.433.019-.544.112-.093.13-.145.237-.154.32-.01.084.037.159.14.224.102.065.2.116.293.153.093.037.2.075.32.112.121.037.2.065.238.084.26.186.334.316.223.39a3.374 3.374 0 0 1-.237.098c-.121.046-.228.088-.321.125-.093.038-.15.075-.168.112-.055.074-.055.205 0 .39.056.187.038.317-.055.391-.093-.093-.177-.256-.252-.488-.074-.233-.14-.386-.195-.46.13.167-.102.223-.697.167l-.28-.028c-.074 0-.223.019-.446.056a2.073 2.073 0 0 1-.572.028.519.519 0 0 1-.377-.223c-.074-.15-.074-.335 0-.558.02-.075.056-.093.112-.056a4.18 4.18 0 0 1-.307-.265c-.13-.121-.223-.2-.279-.237-.856.279-1.73.66-2.623 1.143.112.02.224.01.335-.027.093-.038.214-.098.363-.182a9.41 9.41 0 0 1 .279-.153c.632-.26 1.023-.326 1.172-.196l.14-.14c.26.299.446.531.557.698-.13-.074-.409-.083-.837-.027-.372.111-.576.223-.614.334.13.224.177.391.14.503a4.358 4.358 0 0 1-.32-.28 2.213 2.213 0 0 0-.405-.306c-.13-.075-.27-.121-.419-.14-.298 0-.502.01-.614.028-2.716 1.488-4.901 3.553-6.557 6.194.13.13.242.205.335.224.074.018.121.102.14.25.018.15.041.252.07.308.027.055.134.027.32-.084.168.149.195.325.084.53.018-.019.428.232 1.228.753.353.317.548.512.586.586.055.205-.038.372-.28.502-.018-.037-.102-.12-.25-.25-.15-.13-.233-.168-.252-.112-.056.093-.05.265.014.516.065.251.163.367.293.349-.13 0-.218.149-.265.446-.046.298-.07.628-.07.99 0 .363-.009.582-.028.656l.056.028c-.056.223-.004.544.154.963.158.418.358.6.6.544-.242.056-.056.456.558 1.2.111.149.186.232.223.25.056.038.167.108.335.21.167.102.307.195.418.28.112.083.205.18.28.292.074.093.167.302.278.628.112.325.242.544.39.656-.036.111.052.297.266.558.214.26.312.474.293.641a.136.136 0 0 0-.07.028.136.136 0 0 1-.07.028c.056.13.2.26.433.39.232.131.377.252.432.364.019.055.038.148.056.279.019.13.047.232.084.306.037.075.111.093.223.056.037-.372-.186-.948-.67-1.73a22.925 22.925 0 0 1-.474-.809 1.604 1.604 0 0 1-.153-.432 2.128 2.128 0 0 0-.126-.405c.037 0 .093.014.167.042.075.028.154.06.238.098.083.037.153.074.209.111.056.037.074.065.056.084-.056.13-.037.293.056.488.093.196.204.368.334.516a38.267 38.267 0 0 0 .81.893c.111.112.241.293.39.544.149.251.149.377 0 .377.168 0 .354.093.558.28.205.185.363.371.474.557.093.149.168.39.224.726.056.334.102.558.14.67.036.13.115.255.236.376.121.12.238.21.35.265l.446.223.362.195c.093.038.265.135.516.293a3.6 3.6 0 0 0 .6.321c.186.075.335.112.447.112.111 0 .246-.023.404-.07.158-.046.284-.079.377-.098.28-.037.549.103.81.419.26.316.455.512.585.586.67.353 1.181.456 1.535.307-.037.018-.033.088.014.21.046.12.12.264.223.432a27.356 27.356 0 0 0 .404.641c.093.112.26.251.503.419.242.167.409.307.502.418.112-.074.177-.158.195-.25-.056.148.01.334.196.557.186.224.353.317.502.28.26-.056.39-.354.39-.894-.576.28-1.032.112-1.367-.502a.544.544 0 0 0-.07-.153 1.766 1.766 0 0 1-.181-.474.396.396 0 0 1 0-.21c.019-.056.065-.084.14-.084.167 0 .26-.032.279-.097.018-.065 0-.182-.056-.349a5.625 5.625 0 0 1-.112-.363c-.018-.148-.12-.334-.307-.558a7.435 7.435 0 0 1-.334-.418c-.094.167-.242.242-.447.223-.205-.019-.353-.102-.446-.251a.77.77 0 0 1-.042.153.664.664 0 0 0-.042.182c-.242 0-.381-.01-.419-.028.019-.056.042-.219.07-.488.028-.27.06-.48.098-.628a1.55 1.55 0 0 1 .153-.335 5.76 5.76 0 0 0 .21-.405c.055-.12.093-.237.111-.348.019-.112-.023-.2-.125-.266-.103-.065-.265-.088-.489-.07-.353.02-.595.205-.725.559a5.787 5.787 0 0 0-.084.293.945.945 0 0 1-.14.32.694.694 0 0 1-.25.196c-.13.056-.354.074-.67.056-.316-.019-.54-.065-.67-.14-.242-.149-.45-.418-.628-.809-.176-.39-.265-.735-.265-1.032 0-.186.024-.433.07-.74.047-.307.075-.54.084-.697.01-.158-.042-.386-.154-.684.056-.037.14-.125.252-.265.111-.14.204-.237.279-.293a.498.498 0 0 1 .125-.042.285.285 0 0 1 .126 0c.037.01.074-.004.111-.042a.354.354 0 0 0 .084-.167.831.831 0 0 0-.112-.084c-.055-.056-.093-.083-.111-.083.13.055.395.041.795-.042.4-.084.656-.07.767.042.28.204.484.186.614-.056a2.22 2.22 0 0 0-.07-.265c-.046-.158-.05-.284-.014-.377.093.502.363.586.81.251.055.056.2.102.432.14.233.037.395.083.488.14.056.036.121.088.196.153.074.065.125.107.153.125.028.019.074.014.14-.014a.84.84 0 0 0 .237-.181c.186.26.297.484.335.67.204.744.38 1.153.53 1.227.13.056.232.075.307.056.074-.019.116-.107.125-.265.01-.158.01-.288 0-.39a6.933 6.933 0 0 0-.042-.35l-.028-.223v-.502l-.027-.223c-.28-.056-.452-.168-.517-.335a.599.599 0 0 1 .042-.516c.093-.177.233-.349.419-.516a.91.91 0 0 1 .223-.098c.13-.046.274-.107.432-.181.159-.075.275-.15.35-.224.39-.353.53-.678.418-.976.13 0 .232-.084.307-.251-.019 0-.065-.028-.14-.084a2.04 2.04 0 0 0-.21-.14.379.379 0 0 0-.125-.055c.168-.093.186-.242.056-.447.093-.055.163-.158.21-.307.046-.148.116-.241.209-.279.167.224.362.242.586.056.13-.149.14-.297.027-.446.093-.13.284-.228.572-.293.289-.065.46-.154.517-.265.13.037.204.018.223-.056.018-.074.028-.186.028-.335 0-.149.028-.26.083-.335.075-.093.214-.176.419-.25.205-.075.325-.122.363-.14l.474-.307c.056-.075.056-.112 0-.112a1.01 1.01 0 0 0 .865-.307c.186-.204.13-.39-.167-.558.055-.111.027-.2-.084-.265a1.46 1.46 0 0 0-.419-.153c.056-.019.163-.024.321-.014.158.009.256-.005.293-.042.28-.186.214-.335-.195-.447-.316-.093-.716.019-1.2.335zm-4.548 24.47c3.832-.67 7.096-2.427 9.794-5.273-.056-.056-.172-.098-.35-.126-.176-.028-.292-.06-.348-.098-.335-.13-.558-.204-.67-.223a.547.547 0 0 0-.07-.362.745.745 0 0 0-.223-.252c-.083-.055-.2-.13-.348-.223a14.687 14.687 0 0 1-.307-.195 2.631 2.631 0 0 0-.196-.168 9.374 9.374 0 0 0-.195-.153 1.98 1.98 0 0 0-.21-.126c-.101-.055-.18-.074-.236-.055a.98.98 0 0 1-.28.028l-.083.027a.771.771 0 0 0-.153.07 2.285 2.285 0 0 1-.154.084.288.288 0 0 0-.112.084c-.018.028-.018.05 0 .07-.39-.317-.725-.521-1.004-.614a.788.788 0 0 1-.307-.154 2.65 2.65 0 0 0-.293-.195.408.408 0 0 0-.279-.042c-.102.019-.21.084-.32.195-.094.093-.15.233-.168.419a2.115 2.115 0 0 1-.056.362c-.13-.093-.13-.255 0-.488.13-.232.149-.404.056-.516-.056-.112-.154-.153-.293-.126a.934.934 0 0 0-.335.126 5.36 5.36 0 0 0-.32.237c-.131.102-.215.163-.252.182a2.67 2.67 0 0 0-.237.153.939.939 0 0 0-.237.21 1.385 1.385 0 0 0-.168.334 1.688 1.688 0 0 1-.14.307c-.036-.074-.143-.135-.32-.181-.177-.047-.265-.098-.265-.154.037.186.074.512.111.977.038.465.084.818.14 1.06.13.577.019 1.023-.335 1.34-.502.464-.772.836-.809 1.115-.074.41.037.651.335.726 0 .13-.075.32-.223.572-.15.25-.214.45-.196.6 0 .111.019.26.056.446z"/></g></g></svg>
```

## File: static\lib\jstz.min.js

```javascript
/* jstz.min.js Version: 1.0.6 Build date: 2015-11-04 */
!function(e){var a=function(){"use strict";var e="s",s={DAY:864e5,HOUR:36e5,MINUTE:6e4,SECOND:1e3,BASELINE_YEAR:2014,MAX_SCORE:864e6,AMBIGUITIES:{"America/Denver":["America/Mazatlan"],"Europe/London":["Africa/Casablanca"],"America/Chicago":["America/Mexico_City"],"America/Asuncion":["America/Campo_Grande","America/Santiago"],"America/Montevideo":["America/Sao_Paulo","America/Santiago"],"Asia/Beirut":["Asia/Amman","Asia/Jerusalem","Europe/Helsinki","Asia/Damascus","Africa/Cairo","Asia/Gaza","Europe/Minsk"],"Pacific/Auckland":["Pacific/Fiji"],"America/Los_Angeles":["America/Santa_Isabel"],"America/New_York":["America/Havana"],"America/Halifax":["America/Goose_Bay"],"America/Godthab":["America/Miquelon"],"Asia/Dubai":["Asia/Yerevan"],"Asia/Jakarta":["Asia/Krasnoyarsk"],"Asia/Shanghai":["Asia/Irkutsk","Australia/Perth"],"Australia/Sydney":["Australia/Lord_Howe"],"Asia/Tokyo":["Asia/Yakutsk"],"Asia/Dhaka":["Asia/Omsk"],"Asia/Baku":["Asia/Yerevan"],"Australia/Brisbane":["Asia/Vladivostok"],"Pacific/Noumea":["Asia/Vladivostok"],"Pacific/Majuro":["Asia/Kamchatka","Pacific/Fiji"],"Pacific/Tongatapu":["Pacific/Apia"],"Asia/Baghdad":["Europe/Minsk","Europe/Moscow"],"Asia/Karachi":["Asia/Yekaterinburg"],"Africa/Johannesburg":["Asia/Gaza","Africa/Cairo"]}},i=function(e){var a=-e.getTimezoneOffset();return null!==a?a:0},r=function(){var a=i(new Date(s.BASELINE_YEAR,0,2)),r=i(new Date(s.BASELINE_YEAR,5,2)),n=a-r;return 0>n?a+",1":n>0?r+",1,"+e:a+",0"},n=function(){var e,a;if("undefined"!=typeof Intl&&"undefined"!=typeof Intl.DateTimeFormat&&(e=Intl.DateTimeFormat(),"undefined"!=typeof e&&"undefined"!=typeof e.resolvedOptions))return a=e.resolvedOptions().timeZone,a&&(a.indexOf("/")>-1||"UTC"===a)?a:void 0},o=function(e){for(var a=new Date(e,0,1,0,0,1,0).getTime(),s=new Date(e,12,31,23,59,59).getTime(),i=a,r=new Date(i).getTimezoneOffset(),n=null,o=null;s-864e5>i;){var t=new Date(i),A=t.getTimezoneOffset();A!==r&&(r>A&&(n=t),A>r&&(o=t),r=A),i+=864e5}return n&&o?{s:u(n).getTime(),e:u(o).getTime()}:!1},u=function l(e,a,i){"undefined"==typeof a&&(a=s.DAY,i=s.HOUR);for(var r=new Date(e.getTime()-a).getTime(),n=e.getTime()+a,o=new Date(r).getTimezoneOffset(),u=r,t=null;n-i>u;){var A=new Date(u),c=A.getTimezoneOffset();if(c!==o){t=A;break}u+=i}return a===s.DAY?l(t,s.HOUR,s.MINUTE):a===s.HOUR?l(t,s.MINUTE,s.SECOND):t},t=function(e,a,s,i){if("N/A"!==s)return s;if("Asia/Beirut"===a){if("Africa/Cairo"===i.name&&13983768e5===e[6].s&&14116788e5===e[6].e)return 0;if("Asia/Jerusalem"===i.name&&13959648e5===e[6].s&&14118588e5===e[6].e)return 0}else if("America/Santiago"===a){if("America/Asuncion"===i.name&&14124816e5===e[6].s&&1397358e6===e[6].e)return 0;if("America/Campo_Grande"===i.name&&14136912e5===e[6].s&&13925196e5===e[6].e)return 0}else if("America/Montevideo"===a){if("America/Sao_Paulo"===i.name&&14136876e5===e[6].s&&1392516e6===e[6].e)return 0}else if("Pacific/Auckland"===a&&"Pacific/Fiji"===i.name&&14142456e5===e[6].s&&13961016e5===e[6].e)return 0;return s},A=function(e,i){for(var r=function(a){for(var r=0,n=0;n<e.length;n++)if(a.rules[n]&&e[n]){if(!(e[n].s>=a.rules[n].s&&e[n].e<=a.rules[n].e)){r="N/A";break}if(r=0,r+=Math.abs(e[n].s-a.rules[n].s),r+=Math.abs(a.rules[n].e-e[n].e),r>s.MAX_SCORE){r="N/A";break}}return r=t(e,i,r,a)},n={},o=a.olson.dst_rules.zones,u=o.length,A=s.AMBIGUITIES[i],c=0;u>c;c++){var m=o[c],l=r(o[c]);"N/A"!==l&&(n[m.name]=l)}for(var f in n)if(n.hasOwnProperty(f))for(var d=0;d<A.length;d++)if(A[d]===f)return f;return i},c=function(e){var s=function(){for(var e=[],s=0;s<a.olson.dst_rules.years.length;s++){var i=o(a.olson.dst_rules.years[s]);e.push(i)}return e},i=function(e){for(var a=0;a<e.length;a++)if(e[a]!==!1)return!0;return!1},r=s(),n=i(r);return n?A(r,e):e},m=function(){var e=n();return e||(e=a.olson.timezones[r()],"undefined"!=typeof s.AMBIGUITIES[e]&&(e=c(e))),{name:function(){return e}}};return{determine:m}}();a.olson=a.olson||{},a.olson.timezones={"-720,0":"Etc/GMT+12","-660,0":"Pacific/Pago_Pago","-660,1,s":"Pacific/Apia","-600,1":"America/Adak","-600,0":"Pacific/Honolulu","-570,0":"Pacific/Marquesas","-540,0":"Pacific/Gambier","-540,1":"America/Anchorage","-480,1":"America/Los_Angeles","-480,0":"Pacific/Pitcairn","-420,0":"America/Phoenix","-420,1":"America/Denver","-360,0":"America/Guatemala","-360,1":"America/Chicago","-360,1,s":"Pacific/Easter","-300,0":"America/Bogota","-300,1":"America/New_York","-270,0":"America/Caracas","-240,1":"America/Halifax","-240,0":"America/Santo_Domingo","-240,1,s":"America/Asuncion","-210,1":"America/St_Johns","-180,1":"America/Godthab","-180,0":"America/Argentina/Buenos_Aires","-180,1,s":"America/Montevideo","-120,0":"America/Noronha","-120,1":"America/Noronha","-60,1":"Atlantic/Azores","-60,0":"Atlantic/Cape_Verde","0,0":"UTC","0,1":"Europe/London","60,1":"Europe/Berlin","60,0":"Africa/Lagos","60,1,s":"Africa/Windhoek","120,1":"Asia/Beirut","120,0":"Africa/Johannesburg","180,0":"Asia/Baghdad","180,1":"Europe/Moscow","210,1":"Asia/Tehran","240,0":"Asia/Dubai","240,1":"Asia/Baku","270,0":"Asia/Kabul","300,1":"Asia/Yekaterinburg","300,0":"Asia/Karachi","330,0":"Asia/Kolkata","345,0":"Asia/Kathmandu","360,0":"Asia/Dhaka","360,1":"Asia/Omsk","390,0":"Asia/Rangoon","420,1":"Asia/Krasnoyarsk","420,0":"Asia/Jakarta","480,0":"Asia/Shanghai","480,1":"Asia/Irkutsk","525,0":"Australia/Eucla","525,1,s":"Australia/Eucla","540,1":"Asia/Yakutsk","540,0":"Asia/Tokyo","570,0":"Australia/Darwin","570,1,s":"Australia/Adelaide","600,0":"Australia/Brisbane","600,1":"Asia/Vladivostok","600,1,s":"Australia/Sydney","630,1,s":"Australia/Lord_Howe","660,1":"Asia/Kamchatka","660,0":"Pacific/Noumea","690,0":"Pacific/Norfolk","720,1,s":"Pacific/Auckland","720,0":"Pacific/Majuro","765,1,s":"Pacific/Chatham","780,0":"Pacific/Tongatapu","780,1,s":"Pacific/Apia","840,0":"Pacific/Kiritimati"},a.olson.dst_rules={years:[2008,2009,2010,2011,2012,2013,2014],zones:[{name:"Africa/Cairo",rules:[{e:12199572e5,s:12090744e5},{e:1250802e6,s:1240524e6},{e:12858804e5,s:12840696e5},!1,!1,!1,{e:14116788e5,s:1406844e6}]},{name:"Africa/Casablanca",rules:[{e:12202236e5,s:12122784e5},{e:12508092e5,s:12438144e5},{e:1281222e6,s:12727584e5},{e:13120668e5,s:13017888e5},{e:13489704e5,s:1345428e6},{e:13828392e5,s:13761e8},{e:14142888e5,s:14069448e5}]},{name:"America/Asuncion",rules:[{e:12050316e5,s:12243888e5},{e:12364812e5,s:12558384e5},{e:12709548e5,s:12860784e5},{e:13024044e5,s:1317528e6},{e:1333854e6,s:13495824e5},{e:1364094e6,s:1381032e6},{e:13955436e5,s:14124816e5}]},{name:"America/Campo_Grande",rules:[{e:12032172e5,s:12243888e5},{e:12346668e5,s:12558384e5},{e:12667212e5,s:1287288e6},{e:12981708e5,s:13187376e5},{e:13302252e5,s:1350792e6},{e:136107e7,s:13822416e5},{e:13925196e5,s:14136912e5}]},{name:"America/Goose_Bay",rules:[{e:122559486e4,s:120503526e4},{e:125704446e4,s:123648486e4},{e:128909886e4,s:126853926e4},{e:13205556e5,s:129998886e4},{e:13520052e5,s:13314456e5},{e:13834548e5,s:13628952e5},{e:14149044e5,s:13943448e5}]},{name:"America/Havana",rules:[{e:12249972e5,s:12056436e5},{e:12564468e5,s:12364884e5},{e:12885012e5,s:12685428e5},{e:13211604e5,s:13005972e5},{e:13520052e5,s:13332564e5},{e:13834548e5,s:13628916e5},{e:14149044e5,s:13943412e5}]},{name:"America/Mazatlan",rules:[{e:1225008e6,s:12074724e5},{e:12564576e5,s:1238922e6},{e:1288512e6,s:12703716e5},{e:13199616e5,s:13018212e5},{e:13514112e5,s:13332708e5},{e:13828608e5,s:13653252e5},{e:14143104e5,s:13967748e5}]},{name:"America/Mexico_City",rules:[{e:12250044e5,s:12074688e5},{e:1256454e6,s:12389184e5},{e:12885084e5,s:1270368e6},{e:1319958e6,s:13018176e5},{e:13514076e5,s:13332672e5},{e:13828572e5,s:13653216e5},{e:14143068e5,s:13967712e5}]},{name:"America/Miquelon",rules:[{e:12255984e5,s:12050388e5},{e:1257048e6,s:12364884e5},{e:12891024e5,s:12685428e5},{e:1320552e6,s:12999924e5},{e:13520016e5,s:1331442e6},{e:13834512e5,s:13628916e5},{e:14149008e5,s:13943412e5}]},{name:"America/Santa_Isabel",rules:[{e:12250116e5,s:1207476e6},{e:12564612e5,s:12389256e5},{e:12885156e5,s:12703752e5},{e:13199652e5,s:13018248e5},{e:13514148e5,s:13332744e5},{e:13828644e5,s:13653288e5},{e:1414314e6,s:13967784e5}]},{name:"America/Santiago",rules:[{e:1206846e6,s:1223784e6},{e:1237086e6,s:12552336e5},{e:127035e7,s:12866832e5},{e:13048236e5,s:13138992e5},{e:13356684e5,s:13465584e5},{e:1367118e6,s:13786128e5},{e:13985676e5,s:14100624e5}]},{name:"America/Sao_Paulo",rules:[{e:12032136e5,s:12243852e5},{e:12346632e5,s:12558348e5},{e:12667176e5,s:12872844e5},{e:12981672e5,s:1318734e6},{e:13302216e5,s:13507884e5},{e:13610664e5,s:1382238e6},{e:1392516e6,s:14136876e5}]},{name:"Asia/Amman",rules:[{e:1225404e6,s:12066552e5},{e:12568536e5,s:12381048e5},{e:12883032e5,s:12695544e5},{e:13197528e5,s:13016088e5},!1,!1,{e:14147064e5,s:13959576e5}]},{name:"Asia/Damascus",rules:[{e:12254868e5,s:120726e7},{e:125685e7,s:12381048e5},{e:12882996e5,s:12701592e5},{e:13197492e5,s:13016088e5},{e:13511988e5,s:13330584e5},{e:13826484e5,s:1364508e6},{e:14147028e5,s:13959576e5}]},{name:"Asia/Dubai",rules:[!1,!1,!1,!1,!1,!1,!1]},{name:"Asia/Gaza",rules:[{e:12199572e5,s:12066552e5},{e:12520152e5,s:12381048e5},{e:1281474e6,s:126964086e4},{e:1312146e6,s:130160886e4},{e:13481784e5,s:13330584e5},{e:13802292e5,s:1364508e6},{e:1414098e6,s:13959576e5}]},{name:"Asia/Irkutsk",rules:[{e:12249576e5,s:12068136e5},{e:12564072e5,s:12382632e5},{e:12884616e5,s:12697128e5},!1,!1,!1,!1]},{name:"Asia/Jerusalem",rules:[{e:12231612e5,s:12066624e5},{e:1254006e6,s:1238112e6},{e:1284246e6,s:12695616e5},{e:131751e7,s:1301616e6},{e:13483548e5,s:13330656e5},{e:13828284e5,s:13645152e5},{e:1414278e6,s:13959648e5}]},{name:"Asia/Kamchatka",rules:[{e:12249432e5,s:12067992e5},{e:12563928e5,s:12382488e5},{e:12884508e5,s:12696984e5},!1,!1,!1,!1]},{name:"Asia/Krasnoyarsk",rules:[{e:12249612e5,s:12068172e5},{e:12564108e5,s:12382668e5},{e:12884652e5,s:12697164e5},!1,!1,!1,!1]},{name:"Asia/Omsk",rules:[{e:12249648e5,s:12068208e5},{e:12564144e5,s:12382704e5},{e:12884688e5,s:126972e7},!1,!1,!1,!1]},{name:"Asia/Vladivostok",rules:[{e:12249504e5,s:12068064e5},{e:12564e8,s:1238256e6},{e:12884544e5,s:12697056e5},!1,!1,!1,!1]},{name:"Asia/Yakutsk",rules:[{e:1224954e6,s:120681e7},{e:12564036e5,s:12382596e5},{e:1288458e6,s:12697092e5},!1,!1,!1,!1]},{name:"Asia/Yekaterinburg",rules:[{e:12249684e5,s:12068244e5},{e:1256418e6,s:1238274e6},{e:12884724e5,s:12697236e5},!1,!1,!1,!1]},{name:"Asia/Yerevan",rules:[{e:1224972e6,s:1206828e6},{e:12564216e5,s:12382776e5},{e:1288476e6,s:12697272e5},{e:13199256e5,s:13011768e5},!1,!1,!1]},{name:"Australia/Lord_Howe",rules:[{e:12074076e5,s:12231342e5},{e:12388572e5,s:12545838e5},{e:12703068e5,s:12860334e5},{e:13017564e5,s:1317483e6},{e:1333206e6,s:13495374e5},{e:13652604e5,s:1380987e6},{e:139671e7,s:14124366e5}]},{name:"Australia/Perth",rules:[{e:12068136e5,s:12249576e5},!1,!1,!1,!1,!1,!1]},{name:"Europe/Helsinki",rules:[{e:12249828e5,s:12068388e5},{e:12564324e5,s:12382884e5},{e:12884868e5,s:1269738e6},{e:13199364e5,s:13011876e5},{e:1351386e6,s:13326372e5},{e:13828356e5,s:13646916e5},{e:14142852e5,s:13961412e5}]},{name:"Europe/Minsk",rules:[{e:12249792e5,s:12068352e5},{e:12564288e5,s:12382848e5},{e:12884832e5,s:12697344e5},!1,!1,!1,!1]},{name:"Europe/Moscow",rules:[{e:12249756e5,s:12068316e5},{e:12564252e5,s:12382812e5},{e:12884796e5,s:12697308e5},!1,!1,!1,!1]},{name:"Pacific/Apia",rules:[!1,!1,!1,{e:13017528e5,s:13168728e5},{e:13332024e5,s:13489272e5},{e:13652568e5,s:13803768e5},{e:13967064e5,s:14118264e5}]},{name:"Pacific/Fiji",rules:[!1,!1,{e:12696984e5,s:12878424e5},{e:13271544e5,s:1319292e6},{e:1358604e6,s:13507416e5},{e:139005e7,s:1382796e6},{e:14215032e5,s:14148504e5}]},{name:"Europe/London",rules:[{e:12249828e5,s:12068388e5},{e:12564324e5,s:12382884e5},{e:12884868e5,s:1269738e6},{e:13199364e5,s:13011876e5},{e:1351386e6,s:13326372e5},{e:13828356e5,s:13646916e5},{e:14142852e5,s:13961412e5}]}]},"undefined"!=typeof module&&"undefined"!=typeof module.exports?module.exports=a:"undefined"!=typeof define&&null!==define&&null!=define.amd?define([],function(){return a}):"undefined"==typeof e?window.jstz=a:e.jstz=a}();
```

## File: static\src\js\set_view_track.js

```javascript
odoo.define('website.set_view_track', function (require) {
"use strict";

var CustomizeMenu = require('website.customizeMenu');
var Widget = require('web.Widget');

var TrackPage = Widget.extend({
    template: 'website.track_page',
    xmlDependencies: ['/website/static/src/xml/track_page.xml'],
    events: {
        'change #switch-track-page': '_onTrackChange',
    },

    /**
     * @override
     */
    start: function () {
        this.$input = this.$('#switch-track-page');
        this._isTracked().then((data) => {
            if (data[0]['track']) {
                this.track = true;
                this.$input.attr('checked', 'checked');
            } else {
                this.track = false;
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _isTracked: function (val) {
        var viewid = $('html').data('viewid');
        if (!viewid) {
            return Promise.reject();
        } else {
            return this._rpc({
                model: 'ir.ui.view',
                method: 'read',
                args: [[viewid], ['track']],
            });
        }
    },
    /**
     * @private
     */
    _onTrackChange: function (ev) {
        var checkboxValue = this.$input.is(':checked');
        if (checkboxValue !== this.track) {
            this.track = checkboxValue;
            this._trackPage(checkboxValue);
        }
    },
    /**
     * @private
     */
    _trackPage: function (val) {
        var viewid = $('html').data('viewid');
        if (!viewid) {
            return Promise.reject();
        } else {
            return this._rpc({
                model: 'ir.ui.view',
                method: 'write',
                args: [[viewid], {track: val}],
            });
        }
    },
});

CustomizeMenu.include({
    _loadCustomizeOptions: function () {
        var self = this;
        var def = this._super.apply(this, arguments);
        return def.then(function () {
            if (!self.__trackpageLoaded) {
                self.__trackpageLoaded = true;
                self.trackPage = new TrackPage(self);
                self.trackPage.appendTo(self.$el.children('.dropdown-menu'));
            }
        });
    },
});

});

```

## File: static\src\js\user_custom_javascript.js

```javascript
//
// This file is meant to regroup your javascript code. You can either copy/past
// any code that should be executed on each page loading or write your own
// taking advantage of the Odoo framework to create new behaviors or modify
// existing ones. For example, doing this will greet any visitor with a 'Hello,
// world !' message in a popup:
//
/*
odoo.define('website.user_custom_code', function (require) {
'use strict';

var Dialog = require('web.Dialog');
var publicWidget = require('web.public.widget');

publicWidget.registry.HelloWorldPopup = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    start: function () {
        Dialog.alert(this, "Hello, world!");
        return this._super.apply(this, arguments);
    },
})
});
*/

```

## File: static\src\js\utils.js

```javascript
odoo.define('website.utils', function (require) {
'use strict';

var ajax = require('web.ajax');
var core = require('web.core');

var qweb = core.qweb;

/**
 * Allows to load anchors from a page.
 *
 * @param {string} url
 * @returns {Deferred<string[]>}
 */
function loadAnchors(url) {
    return new Promise(function (resolve, reject) {
        if (url !== window.location.pathname && url[0] !== '#') {
            $.get(window.location.origin + url).then(resolve, reject);
        } else {
            resolve(document.body.outerHTML);
        }
    }).then(function (response) {
        return _.map($(response).find('[id][data-anchor=true]'), function (el) {
            return '#' + el.id;
        });
    });
}

/**
 * Allows the given input to propose existing website URLs.
 *
 * @param {ServicesMixin|Widget} self - an element capable to trigger an RPC
 * @param {jQuery} $input
 */
function autocompleteWithPages(self, $input) {
    $input.autocomplete({
        source: function (request, response) {
            if (request.term[0] === '#') {
                loadAnchors(request.term).then(function (anchors) {
                    response(anchors);
                });
            } else {
                return self._rpc({
                    model: 'website',
                    method: 'search_pages',
                    args: [null, request.term],
                    kwargs: {
                        limit: 15,
                    },
                }).then(function (exists) {
                    var rs = _.map(exists, function (r) {
                        return r.loc;
                    });
                    response(rs.sort());
                });
            }
        },
        close: function () {
            self.trigger_up('website_url_chosen');
        },
    });
}

/**
 * @param {jQuery} $element
 */
function onceAllImagesLoaded($element) {
    var defs = _.map($element.find('img').addBack('img'), function (img) {
        if (img.complete) {
            return; // Already loaded
        }
        var def = new Promise(function (resolve, reject) {
            $(img).one('load', function () {
                resolve();
            });
        });
        return def;
    });
    return Promise.all(defs);
}

/**
 * @deprecated
 * @todo create Dialog.prompt instead of this
 */
function prompt(options, _qweb) {
    /**
     * A bootstrapped version of prompt() albeit asynchronous
     * This was built to quickly prompt the user with a single field.
     * For anything more complex, please use editor.Dialog class
     *
     * Usage Ex:
     *
     * website.prompt("What... is your quest ?").then(function (answer) {
     *     arthur.reply(answer || "To seek the Holy Grail.");
     * });
     *
     * website.prompt({
     *     select: "Please choose your destiny",
     *     init: function () {
     *         return [ [0, "Sub-Zero"], [1, "Robo-Ky"] ];
     *     }
     * }).then(function (answer) {
     *     mame_station.loadCharacter(answer);
     * });
     *
     * @param {Object|String} options A set of options used to configure the prompt or the text field name if string
     * @param {String} [options.window_title=''] title of the prompt modal
     * @param {String} [options.input] tell the modal to use an input text field, the given value will be the field title
     * @param {String} [options.textarea] tell the modal to use a textarea field, the given value will be the field title
     * @param {String} [options.select] tell the modal to use a select box, the given value will be the field title
     * @param {Object} [options.default=''] default value of the field
     * @param {Function} [options.init] optional function that takes the `field` (enhanced with a fillWith() method) and the `dialog` as parameters [can return a promise]
     */
    if (typeof options === 'string') {
        options = {
            text: options
        };
    }
    var xmlDef;
    if (_.isUndefined(_qweb)) {
        _qweb = 'website.prompt';
        xmlDef = ajax.loadXML('/website/static/src/xml/website.xml', core.qweb);
    }
    options = _.extend({
        window_title: '',
        field_name: '',
        'default': '', // dict notation for IE<9
        init: function () {},
    }, options || {});

    var type = _.intersection(Object.keys(options), ['input', 'textarea', 'select']);
    type = type.length ? type[0] : 'input';
    options.field_type = type;
    options.field_name = options.field_name || options[type];

    var def = new Promise(function (resolve, reject) {
        Promise.resolve(xmlDef).then(function () {
            var dialog = $(qweb.render(_qweb, options)).appendTo('body');
            options.$dialog = dialog;
            var field = dialog.find(options.field_type).first();
            field.val(options['default']); // dict notation for IE<9
            field.fillWith = function (data) {
                if (field.is('select')) {
                    var select = field[0];
                    data.forEach(function (item) {
                        select.options[select.options.length] = new window.Option(item[1], item[0]);
                    });
                } else {
                    field.val(data);
                }
            };
            var init = options.init(field, dialog);
            Promise.resolve(init).then(function (fill) {
                if (fill) {
                    field.fillWith(fill);
                }
                dialog.modal('show');
                field.focus();
                dialog.on('click', '.btn-primary', function () {
                    var backdrop = $('.modal-backdrop');
                    resolve({ val: field.val(), field: field, dialog: dialog });
                    dialog.modal('hide').remove();
                        backdrop.remove();
                });
            });
            dialog.on('hidden.bs.modal', function () {
                    var backdrop = $('.modal-backdrop');
                reject();
                dialog.remove();
                    backdrop.remove();
            });
            if (field.is('input[type="text"], select')) {
                field.keypress(function (e) {
                    if (e.which === 13) {
                        e.preventDefault();
                        dialog.find('.btn-primary').trigger('click');
                    }
                });
            }
        });
    });

    return def;
}

function websiteDomain(self) {
    var websiteID;
    self.trigger_up('context_get', {
        callback: function (ctx) {
            websiteID = ctx['website_id'];
        },
    });
    return ['|', ['website_id', '=', false], ['website_id', '=', websiteID]];
}

/**
 * Converts a base64 SVG into a base64 PNG.
 *
 * @param {string|HTMLImageElement} src - an URL to a SVG or a *loaded* image
 *      with such an URL. This allows the call to this method to be potentially
 *      not return a Promise.
 * @param {boolean} [noAsync=false] In case, the given first parameter is a
 *      loaded image, this parameter allows to ignore problematic images and
 *      return a (problematic) PNG result synchronously.
 * @returns {Promise<string>|string} a base64 PNG (as result of a Promise or not)
 */
function svgToPNG(src, noAsync = false) {
    function checkImg(imgEl) {
        // Firefox does not support drawing SVG to canvas unless it has width
        // and height attributes set on the root <svg>.
        return (imgEl.naturalHeight !== 0);
    }
    function toPNGViaCanvas(imgEl) {
        const canvas = document.createElement('canvas');
        canvas.width = imgEl.width;
        canvas.height = imgEl.height;
        canvas.getContext('2d').drawImage(imgEl, 0, 0);
        return canvas.toDataURL('image/png');
    }

    // In case we receive a loaded image with the given src and that this image
    // is not problematic, we can convert it to PNG synchronously.
    if (src instanceof HTMLImageElement) {
        const loadedImgEl = src;
        if (noAsync || checkImg(loadedImgEl)) {
            return toPNGViaCanvas(loadedImgEl);
        }
        src = loadedImgEl.src;
    }

    // At this point, we either did not receive a loaded image or the received
    // loaded image is problematic => we have to do some asynchronous code and
    // the function will thus return a Promise.
    return new Promise(resolve => {
        const imgEl = new Image();
        imgEl.onload = () => {
            if (checkImg(imgEl)) {
                resolve(imgEl);
                return;
            }

            // Set arbitrary height on image and attach it to the DOM to force
            // width computation.
            imgEl.height = 1000;
            imgEl.style.opacity = 0;
            document.body.appendChild(imgEl);

            const request = new XMLHttpRequest();
            request.open('GET', imgEl.src, true);
            request.onload = () => {
                // Convert the data URI to a SVG element
                const parser = new DOMParser();
                const result = parser.parseFromString(request.responseText, 'text/xml');
                const svgEl = result.getElementsByTagName("svg")[0];

                // Add the attributes Firefox needs and remove the image from
                // the DOM.
                svgEl.setAttribute('width', imgEl.width);
                svgEl.setAttribute('height', imgEl.height);
                imgEl.remove();

                // Convert the SVG element to a data URI
                const svg64 = btoa(new XMLSerializer().serializeToString(svgEl));
                const finalImg = new Image();
                finalImg.onload = () => {
                    resolve(finalImg);
                };
                finalImg.src = `data:image/svg+xml;base64,${svg64}`;
            };
            request.send();
        };
        imgEl.src = src;
    }).then(loadedImgEl => toPNGViaCanvas(loadedImgEl));
}

return {
    loadAnchors: loadAnchors,
    autocompleteWithPages: autocompleteWithPages,
    onceAllImagesLoaded: onceAllImagesLoaded,
    prompt: prompt,
    websiteDomain: websiteDomain,
    svgToPNG: svgToPNG,
};
});

```

## File: static\src\js\visitor_timezone.js

```javascript
//
// This file is meant to determine the timezone of a website visitor
// If the visitor already exists, no need to keep the timezone cookie
// as the timezone is set on the visitor.
//
odoo.define('website.visitor_timezone', function (require) {
'use strict';

var ajax = require('web.ajax');
var utils = require('web.utils');
var publicWidget = require('web.public.widget');

publicWidget.registry.visitorTimezone = publicWidget.Widget.extend({
    selector: '#wrapwrap',

    start: function () {
        if (!localStorage.getItem('website.found_visitor_timezone')) {
            var timezone = jstz.determine().name();
            this._rpc({
                route: '/website/update_visitor_timezone',
                params: {
                    'timezone': timezone,
                },
            }).then(function (result) {
                if (result) {
                    localStorage.setItem('website.found_visitor_timezone', true);
                }
            });
        }
        return this._super.apply(this, arguments);
    },
});

return publicWidget.registry.visitorTimezone;

});

```

## File: static\src\js\backend\button.js

```javascript
odoo.define('website.backend.button', function (require) {
'use strict';

var AbstractField = require('web.AbstractField');
var core = require('web.core');
var field_registry = require('web.field_registry');

var _t = core._t;

var WebsitePublishButton = AbstractField.extend({
    className: 'o_stat_info',
    supportedFieldTypes: ['boolean'],

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * A boolean field is always set since false is a valid value.
     *
     * @override
     */
    isSet: function () {
        return true;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * This widget is supposed to be used inside a stat button and, as such, is
     * rendered the same way in edit and readonly mode.
     *
     * @override
     * @private
     */
    _render: function () {
        this.$el.empty();
        var text = this.value ? _t("Published") : _t("Unpublished");
        var hover = this.value ? _t("Unpublish") : _t("Publish");
        var valColor = this.value ? 'text-success' : 'text-danger';
        var hoverColor = this.value ? 'text-danger' : 'text-success';
        var $val = $('<span>').addClass('o_stat_text o_not_hover ' + valColor).text(text);
        var $hover = $('<span>').addClass('o_stat_text o_hover ' + hoverColor).text(hover);
        this.$el.append($val).append($hover);
    },
});

var WidgetWebsiteButtonIcon = AbstractField.extend({
    template: 'WidgetWebsiteButtonIcon',
    events: {
        'click': '_onClick',
    },

    /**
    * @override
    */
    start: function () {
        this.$icon = this.$('.o_button_icon');
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    isSet: function () {
        return true;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _render: function () {
        this._super.apply(this, arguments);

        var published = this.value;
        var info = published ? _t("Published") : _t("Unpublished");
        this.$el.attr('aria-label', info)
                .prop('title', info);
        this.$icon.toggleClass('text-danger', !published)
                .toggleClass('text-success', published);
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * Redirects to the website page of the record.
     *
     * @private
     */
    _onClick: function () {
        this.trigger_up('button_clicked', {
            attrs: {
                type: 'object',
                name: 'open_website_url',
            },
            record: this.record,
        });
    },
});

field_registry
    .add('website_redirect_button', WidgetWebsiteButtonIcon)
    .add('website_publish_button', WebsitePublishButton);
});

```

## File: static\src\js\backend\dashboard.js

```javascript
odoo.define('website.backend.dashboard', function (require) {
'use strict';

var AbstractAction = require('web.AbstractAction');
var ajax = require('web.ajax');
var core = require('web.core');
var Dialog = require('web.Dialog');
var field_utils = require('web.field_utils');
var pyUtils = require('web.py_utils');
var session = require('web.session');
var time = require('web.time');
var web_client = require('web.web_client');

var _t = core._t;
var QWeb = core.qweb;

var COLORS = ["#1f77b4", "#aec7e8"];
var FORMAT_OPTIONS = {
    // allow to decide if utils.human_number should be used
    humanReadable: function (value) {
        return Math.abs(value) >= 1000;
    },
    // with the choices below, 1236 is represented by 1.24k
    minDigits: 1,
    decimals: 2,
    // avoid comma separators for thousands in numbers when human_number is used
    formatterCallback: function (str) {
        return str;
    },
};

var Dashboard = AbstractAction.extend({
    hasControlPanel: true,
    contentTemplate: 'website.WebsiteDashboardMain',
    jsLibs: [
        '/web/static/lib/Chart/Chart.js',
    ],
    events: {
        'click .js_link_analytics_settings': 'on_link_analytics_settings',
        'click .o_dashboard_action': 'on_dashboard_action',
        'click .o_dashboard_action_form': 'on_dashboard_action_form',
    },

    init: function(parent, context) {
        this._super(parent, context);

        this.DATE_FORMAT = time.getLangDateFormat();
        this.date_range = 'week';  // possible values : 'week', 'month', year'
        this.date_from = moment.utc().subtract(1, 'week');
        this.date_to = moment.utc();

        this.dashboards_templates = ['website.dashboard_header', 'website.dashboard_content'];
        this.graphs = [];
        this.chartIds = {};
    },

    willStart: function() {
        var self = this;
        return Promise.all([ajax.loadLibs(this), this._super()]).then(function() {
            return self.fetch_data();
        }).then(function(){
            var website = _.findWhere(self.websites, {selected: true});
            self.website_id = website ? website.id : false;
        });
    },

    start: function() {
        var self = this;
        return this._super().then(function() {
            self.update_cp();
            self.render_graphs();
        });
    },

    on_attach_callback: function () {
        this._isInDom = true;
        this.render_graphs();
        this._super.apply(this, arguments);
    },
    on_detach_callback: function () {
        this._isInDom = false;
        this._super.apply(this, arguments);
    },
    /**
     * Fetches dashboard data
     */
    fetch_data: function() {
        var self = this;
        var prom = this._rpc({
            route: '/website/fetch_dashboard_data',
            params: {
                website_id: this.website_id || false,
                date_from: this.date_from.year()+'-'+(this.date_from.month()+1)+'-'+this.date_from.date(),
                date_to: this.date_to.year()+'-'+(this.date_to.month()+1)+'-'+this.date_to.date(),
            },
        });
        prom.then(function (result) {
            self.data = result;
            self.dashboards_data = result.dashboards;
            self.currency_id = result.currency_id;
            self.groups = result.groups;
            self.websites = result.websites;
        });
        return prom;
    },

    on_link_analytics_settings: function(ev) {
        ev.preventDefault();

        var self = this;
        var dialog = new Dialog(this, {
            size: 'medium',
            title: _t('Connect Google Analytics'),
            $content: QWeb.render('website.ga_dialog_content', {
                ga_key: this.dashboards_data.visits.ga_client_id,
                ga_analytics_key: this.dashboards_data.visits.ga_analytics_key,
            }),
            buttons: [
                {
                    text: _t("Save"),
                    classes: 'btn-primary',
                    close: true,
                    click: function() {
                        var ga_client_id = dialog.$el.find('input[name="ga_client_id"]').val();
                        var ga_analytics_key = dialog.$el.find('input[name="ga_analytics_key"]').val();
                        self.on_save_ga_client_id(ga_client_id, ga_analytics_key);
                    },
                },
                {
                    text: _t("Cancel"),
                    close: true,
                },
            ],
        }).open();
    },

    on_go_to_website: function (ev) {
        ev.preventDefault();
        var website = _.findWhere(this.websites, {selected: true});
        window.location.href = `/website/force/${website.id}`;
    },

    on_save_ga_client_id: function(ga_client_id, ga_analytics_key) {
        var self = this;
        return this._rpc({
            route: '/website/dashboard/set_ga_data',
            params: {
                'website_id': self.website_id,
                'ga_client_id': ga_client_id,
                'ga_analytics_key': ga_analytics_key,
            },
        }).then(function (result) {
            if (result.error) {
                self.do_warn(result.error.title, result.error.message);
                return;
            }
            self.on_date_range_button('week');
        });
    },

    render_dashboards: function() {
        var self = this;
        _.each(this.dashboards_templates, function(template) {
            self.$('.o_website_dashboard').append(QWeb.render(template, {widget: self}));
        });
    },

    render_graph: function(div_to_display, chart_values, chart_id) {
        var self = this;

        this.$(div_to_display).empty();
        var $canvasContainer = $('<div/>', {class: 'o_graph_canvas_container'});
        this.$canvas = $('<canvas/>').attr('id', chart_id);
        $canvasContainer.append(this.$canvas);
        this.$(div_to_display).append($canvasContainer);

        var labels = chart_values[0].values.map(function (date) {
            return moment(date[0], "YYYY-MM-DD", 'en');
        });

        var datasets = chart_values.map(function (group, index) {
            return {
                label: group.key,
                data: group.values.map(function (value) {
                    return value[1];
                }),
                dates: group.values.map(function (value) {
                    return value[0];
                }),
                fill: false,
                borderColor: COLORS[index],
            };
        });

        var ctx = this.$canvas[0];
        this.chart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: datasets,
            },
            options: {
                legend: {
                    display: false,
                },
                maintainAspectRatio: false,
                scales: {
                    yAxes: [{
                        type: 'linear',
                        ticks: {
                            beginAtZero: true,
                            callback: this.formatValue.bind(this),
                        },
                    }],
                    xAxes: [{
                        ticks: {
                            callback: function (moment) {
                                return moment.format(self.DATE_FORMAT);
                            },
                        }
                    }],
                },
                tooltips: {
                    mode: 'index',
                    intersect: false,
                    bodyFontColor: 'rgba(0,0,0,1)',
                    titleFontSize: 13,
                    titleFontColor: 'rgba(0,0,0,1)',
                    backgroundColor: 'rgba(255,255,255,0.6)',
                    borderColor: 'rgba(0,0,0,0.2)',
                    borderWidth: 2,
                    callbacks: {
                        title: function (tooltipItems, data) {
                            return data.datasets[0].label;
                        },
                        label: function (tooltipItem, data) {
                            var moment = data.labels[tooltipItem.index];
                            var date = tooltipItem.datasetIndex === 0 ?
                                        moment :
                                        moment.subtract(1, self.date_range);
                            return date.format(self.DATE_FORMAT) + ': ' + self.formatValue(tooltipItem.yLabel);
                        },
                        labelColor: function (tooltipItem, chart) {
                            var dataset = chart.data.datasets[tooltipItem.datasetIndex];
                            return {
                                borderColor: dataset.borderColor,
                                backgroundColor: dataset.borderColor,
                            };
                        },
                    }
                }
            }
        });
    },

    render_graphs: function() {
        var self = this;
        if (this._isInDom) {
            _.each(this.graphs, function(e) {
                var renderGraph = self.groups[e.group] &&
                                    self.dashboards_data[e.name].summary.order_count;
                if (!self.chartIds[e.name]) {
                    self.chartIds[e.name] = _.uniqueId('chart_' + e.name);
                }
                var chart_id = self.chartIds[e.name];
                if (renderGraph) {
                    self.render_graph('.o_graph_' + e.name, self.dashboards_data[e.name].graph, chart_id);
                }
            });
            this.render_graph_analytics(this.dashboards_data.visits.ga_client_id);
        }
    },

    render_graph_analytics: function(client_id) {
        if (!this.dashboards_data.visits || !this.dashboards_data.visits.ga_client_id) {
          return;
        }

        this.load_analytics_api();

        var $analytics_components = this.$('.js_analytics_components');
        this.addLoader($analytics_components);

        var self = this;
        gapi.analytics.ready(function() {

            $analytics_components.empty();
            // 1. Authorize component
            var $analytics_auth = $('<div>').addClass('col-lg-12');
            window.onOriginError = function () {
                $analytics_components.find('.js_unauthorized_message').remove();
                self.display_unauthorized_message($analytics_components, 'not_initialized');
            };
            gapi.analytics.auth.authorize({
                container: $analytics_auth[0],
                clientid: client_id
            });

            $analytics_auth.appendTo($analytics_components);

            self.handle_analytics_auth($analytics_components);
            gapi.analytics.auth.on('signIn', function() {
                delete window.onOriginError;
                self.handle_analytics_auth($analytics_components);
            });

        });
    },

    on_date_range_button: function(date_range) {
        if (date_range === 'week') {
            this.date_range = 'week';
            this.date_from = moment.utc().subtract(1, 'weeks');
        } else if (date_range === 'month') {
            this.date_range = 'month';
            this.date_from = moment.utc().subtract(1, 'months');
        } else if (date_range === 'year') {
            this.date_range = 'year';
            this.date_from = moment.utc().subtract(1, 'years');
        } else {
            console.log('Unknown date range. Choose between [week, month, year]');
            return;
        }

        var self = this;
        Promise.resolve(this.fetch_data()).then(function () {
            self.$('.o_website_dashboard').empty();
            self.render_dashboards();
            self.render_graphs();
        });

    },

    on_website_button: function(website_id) {
        var self = this;
        this.website_id = website_id;
        Promise.resolve(this.fetch_data()).then(function () {
            self.$('.o_website_dashboard').empty();
            self.render_dashboards();
            self.render_graphs();
        });
    },

    on_reverse_breadcrumb: function() {
        var self = this;
        web_client.do_push_state({});
        this.update_cp();
        this.fetch_data().then(function() {
            self.$('.o_website_dashboard').empty();
            self.render_dashboards();
            self.render_graphs();
        });
    },

    on_dashboard_action: function (ev) {
        ev.preventDefault();
        var self = this
        var $action = $(ev.currentTarget);
        var additional_context = {};
        if (this.date_range === 'week') {
            additional_context = {search_default_week: true};
        } else if (this.date_range === 'month') {
            additional_context = {search_default_month: true};
        } else if (this.date_range === 'year') {
            additional_context = {search_default_year: true};
        }
        this._rpc({
            route: '/web/action/load',
            params: {
                'action_id': $action.attr('name'),
            },
        })
        .then(function (action) {
            action.domain = pyUtils.assembleDomains([action.domain, `[('website_id', '=', ${self.website_id})]`]);
            return self.do_action(action, {
                'additional_context': additional_context,
                'on_reverse_breadcrumb': self.on_reverse_breadcrumb
            });
        });
    },

    on_dashboard_action_form: function (ev) {
        ev.preventDefault();
        var $action = $(ev.currentTarget);
        this.do_action({
            name: $action.attr('name'),
            res_model: $action.data('res_model'),
            res_id: $action.data('res_id'),
            views: [[false, 'form']],
            type: 'ir.actions.act_window',
        }, {
            on_reverse_breadcrumb: this.on_reverse_breadcrumb
        });
    },

    update_cp: function() {
        var self = this;
        if (!this.$searchview) {
            this.$searchview = $(QWeb.render("website.DateRangeButtons", {
                widget: this,
            }));
            this.$searchview.find('button.js_date_range').click(function(ev) {
                self.$searchview.find('button.js_date_range.active').removeClass('active');
                $(ev.target).addClass('active');
                self.on_date_range_button($(ev.target).data('date'));
            });
            this.$searchview.find('button.js_website').click(function(ev) {
                self.$searchview.find('button.js_website.active').removeClass('active');
                $(ev.target).addClass('active');
                self.on_website_button($(ev.target).data('website-id'));
            });
        }

        var $buttons = $(QWeb.render("website.GoToButtons"));
        $buttons.on('click', this.on_go_to_website.bind(this));

        this.updateControlPanel({
            cp_content: {
                $searchview: this.$searchview,
                $buttons: $buttons,
            },
        });
    },

    // Loads Analytics API
    load_analytics_api: function() {
        var self = this;
        if (!("gapi" in window)) {
            (function(w,d,s,g,js,fjs){
                g=w.gapi||(w.gapi={});g.analytics={q:[],ready:function(cb){this.q.push(cb);}};
                js=d.createElement(s);fjs=d.getElementsByTagName(s)[0];
                js.src='https://apis.google.com/js/platform.js';
                fjs.parentNode.insertBefore(js,fjs);js.onload=function(){g.load('analytics');};
            }(window,document,'script'));
            gapi.analytics.ready(function() {
                self.analytics_create_components();
            });
        }
    },

    handle_analytics_auth: function($analytics_components) {
        $analytics_components.find('.js_unauthorized_message').remove();

        // Check if the user is authenticated and has the right to make API calls
        if (!gapi.analytics.auth.getAuthResponse()) {
            this.display_unauthorized_message($analytics_components, 'not_connected');
        } else if (gapi.analytics.auth.getAuthResponse() && gapi.analytics.auth.getAuthResponse().scope.indexOf('https://www.googleapis.com/auth/analytics') === -1) {
            this.display_unauthorized_message($analytics_components, 'no_right');
        } else {
            this.make_analytics_calls($analytics_components);
        }
    },

    display_unauthorized_message: function($analytics_components, reason) {
        $analytics_components.prepend($(QWeb.render('website.unauthorized_analytics', {reason: reason})));
    },

    make_analytics_calls: function($analytics_components) {
        // 2. ActiveUsers component
        var $analytics_users = $('<div>');
        var activeUsers = new gapi.analytics.ext.ActiveUsers({
            container: $analytics_users[0],
            pollingInterval: 10,
        });
        $analytics_users.appendTo($analytics_components);

        // 3. View Selector
        var $analytics_view_selector = $('<div>').addClass('col-lg-12 o_properties_selection');
        var viewSelector = new gapi.analytics.ViewSelector({
            container: $analytics_view_selector[0],
        });
        viewSelector.execute();
        $analytics_view_selector.appendTo($analytics_components);

        // 4. Chart graph
        var start_date = '7daysAgo';
        if (this.date_range === 'month') {
            start_date = '30daysAgo';
        } else if (this.date_range === 'year') {
            start_date = '365daysAgo';
        }
        var $analytics_chart_2 = $('<div>').addClass('col-lg-6 col-12');
        var breakdownChart = new gapi.analytics.googleCharts.DataChart({
            query: {
                'dimensions': 'ga:date',
                'metrics': 'ga:sessions',
                'start-date': start_date,
                'end-date': 'yesterday'
            },
            chart: {
                type: 'LINE',
                container: $analytics_chart_2[0],
                options: {
                    title: 'All',
                    width: '100%',
                    tooltip: {isHtml: true},
                }
            }
        });
        $analytics_chart_2.appendTo($analytics_components);

        // 5. Chart table
        var $analytics_chart_1 = $('<div>').addClass('col-lg-6 col-12');
        var mainChart = new gapi.analytics.googleCharts.DataChart({
            query: {
                'dimensions': 'ga:medium',
                'metrics': 'ga:sessions',
                'sort': '-ga:sessions',
                'max-results': '6'
            },
            chart: {
                type: 'TABLE',
                container: $analytics_chart_1[0],
                options: {
                    width: '100%'
                }
            }
        });
        $analytics_chart_1.appendTo($analytics_components);

        // Events handling & animations

        var table_row_listener;

        viewSelector.on('change', function(ids) {
            var options = {query: {ids: ids}};
            activeUsers.set({ids: ids}).execute();
            mainChart.set(options).execute();
            breakdownChart.set(options).execute();

            if (table_row_listener) { google.visualization.events.removeListener(table_row_listener); }
        });

        mainChart.on('success', function(response) {
            var chart = response.chart;
            var dataTable = response.dataTable;

            table_row_listener = google.visualization.events.addListener(chart, 'select', function() {
                var options;
                if (chart.getSelection().length) {
                    var row =  chart.getSelection()[0].row;
                    var medium =  dataTable.getValue(row, 0);
                    options = {
                        query: {
                            filters: 'ga:medium==' + medium,
                        },
                        chart: {
                            options: {
                                title: medium,
                            }
                        }
                    };
                } else {
                    options = {
                        chart: {
                            options: {
                                title: 'All',
                            }
                        }
                    };
                    delete breakdownChart.get().query.filters;
                }
                breakdownChart.set(options).execute();
            });
        });

        // Add CSS animation to visually show the when users come and go.
        activeUsers.once('success', function() {
            var element = this.container.firstChild;
            var timeout;

            this.on('change', function(data) {
                element = this.container.firstChild;
                var animationClass = data.delta > 0 ? 'is-increasing' : 'is-decreasing';
                element.className += (' ' + animationClass);

                clearTimeout(timeout);
                timeout = setTimeout(function() {
                    element.className = element.className.replace(/ is-(increasing|decreasing)/g, '');
                }, 3000);
            });
        });
    },

    /*
     * Credits to https://github.com/googleanalytics/ga-dev-tools
     * This is the Active Users component that polls
     * the number of active users on Analytics each 5 secs
     */
    analytics_create_components: function() {

        gapi.analytics.createComponent('ActiveUsers', {

            initialize: function() {
                this.activeUsers = 0;
                gapi.analytics.auth.once('signOut', this.handleSignOut_.bind(this));
            },

            execute: function() {
                // Stop any polling currently going on.
                if (this.polling_) {
                    this.stop();
                }

                this.render_();

                // Wait until the user is authorized.
                if (gapi.analytics.auth.isAuthorized()) {
                    this.pollActiveUsers_();
                } else {
                    gapi.analytics.auth.once('signIn', this.pollActiveUsers_.bind(this));
                }
            },

            stop: function() {
                clearTimeout(this.timeout_);
                this.polling_ = false;
                this.emit('stop', {activeUsers: this.activeUsers});
            },

            render_: function() {
                var opts = this.get();

                // Render the component inside the container.
                this.container = typeof opts.container === 'string' ?
                    document.getElementById(opts.container) : opts.container;

                this.container.innerHTML = opts.template || this.template;
                this.container.querySelector('b').innerHTML = this.activeUsers;
            },

            pollActiveUsers_: function() {
                var options = this.get();
                var pollingInterval = (options.pollingInterval || 5) * 1000;

                if (isNaN(pollingInterval) || pollingInterval < 5000) {
                    throw new Error('Frequency must be 5 seconds or more.');
                }

                this.polling_ = true;
                gapi.client.analytics.data.realtime
                    .get({ids:options.ids, metrics:'rt:activeUsers'})
                    .then(function(response) {
                        var result = response.result;
                        var newValue = result.totalResults ? +result.rows[0][0] : 0;
                        var oldValue = this.activeUsers;

                        this.emit('success', {activeUsers: this.activeUsers});

                        if (newValue !== oldValue) {
                            this.activeUsers = newValue;
                            this.onChange_(newValue - oldValue);
                        }

                        if (this.polling_) {
                            this.timeout_ = setTimeout(this.pollActiveUsers_.bind(this), pollingInterval);
                        }
                    }.bind(this));
            },

            onChange_: function(delta) {
                var valueContainer = this.container.querySelector('b');
                if (valueContainer) { valueContainer.innerHTML = this.activeUsers; }

                this.emit('change', {activeUsers: this.activeUsers, delta: delta});
                if (delta > 0) {
                    this.emit('increase', {activeUsers: this.activeUsers, delta: delta});
                } else {
                    this.emit('decrease', {activeUsers: this.activeUsers, delta: delta});
                }
            },

            handleSignOut_: function() {
                this.stop();
                gapi.analytics.auth.once('signIn', this.handleSignIn_.bind(this));
            },

            handleSignIn_: function() {
                this.pollActiveUsers_();
                gapi.analytics.auth.once('signOut', this.handleSignOut_.bind(this));
            },

            template:
                '<div class="ActiveUsers">' +
                    'Active Users: <b class="ActiveUsers-value"></b>' +
                '</div>'

        });
    },

    // Utility functions
    addLoader: function(selector) {
        var loader = '<span class="fa fa-3x fa-spin fa-spinner fa-pulse"/>';
        selector.html("<div class='o_loader'>" + loader + "</div>");
    },
    getValue: function(d) { return d[1]; },
    format_number: function(value, type, digits, symbol) {
        if (type === 'currency') {
            return this.render_monetary_field(value, this.currency_id);
        } else {
            return field_utils.format[type](value || 0, {digits: digits}) + ' ' + symbol;
        }
    },
    formatValue: function (value) {
        var formatter = field_utils.format.float;
        var formatedValue = formatter(value, undefined, FORMAT_OPTIONS);
        return formatedValue;
    },
    render_monetary_field: function(value, currency_id) {
        var currency = session.get_currency(currency_id);
        var formatted_value = field_utils.format.float(value || 0, {digits: currency && currency.digits});
        if (currency) {
            if (currency.position === "after") {
                formatted_value += currency.symbol;
            } else {
                formatted_value = currency.symbol + formatted_value;
            }
        }
        return formatted_value;
    },

});

core.action_registry.add('backend_dashboard', Dashboard);

return Dashboard;
});

```

## File: static\src\js\backend\res_config_settings.js

```javascript
odoo.define('website.settings', function (require) {

var BaseSettingController = require('base.settings').Controller;
var FormController = require('web.FormController');

BaseSettingController.include({

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Bypasses the discard confirmation dialog when going to a website because
     * the target website will be the one selected.
     *
     * Without this override, it is impossible to go to a website other than the
     * first because discarding will revert it back to the default value.
     *
     * @override
     */
    _onButtonClicked: function (ev) {
        if (ev.data.attrs.name === 'website_go_to') {
            FormController.prototype._onButtonClicked.apply(this, arguments);
        } else {
            this._super.apply(this, arguments);
        }
    },
});
});

```

## File: static\src\js\content\compatibility.js

```javascript
odoo.define('website.content.compatibility', function (require) {
'use strict';

/**
 * Tweaks the website rendering so that the old browsers correctly render the
 * content too.
 */

require('web.dom_ready');

// Check the browser and its version and add the info as an attribute of the
// HTML element so that css selectors can match it
var browser = _.findKey($.browser, function (v) { return v === true; });
if ($.browser.mozilla && +$.browser.version.replace(/^([0-9]+\.[0-9]+).*/, '\$1') < 20) {
    browser = 'msie';
}
browser += (',' + $.browser.version);
var mobileRegex = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i;
if (mobileRegex.test(window.navigator.userAgent.toLowerCase())) {
    browser += ',mobile';
}
document.documentElement.setAttribute('data-browser', browser);

// Check if flex is supported and add the info as an attribute of the HTML
// element so that css selectors can match it (only if not supported)
var htmlStyle = document.documentElement.style;
var isFlexSupported = (('flexWrap' in htmlStyle)
                    || ('WebkitFlexWrap' in htmlStyle)
                    || ('msFlexWrap' in htmlStyle));
if (!isFlexSupported) {
    document.documentElement.setAttribute('data-no-flex', '');
}

return {
    browser: browser,
    isFlexSupported: isFlexSupported,
};
});

```

## File: static\src\js\content\lazy_template_call.js

```javascript
odoo.define('website.content.lazy_template_call', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.LazyTemplateRenderer = publicWidget.Widget.extend({
    selector: '#wrapwrap:has([data-oe-call])',

    /**
     * Lazy replaces the `[data-oe-call]` elements by their corresponding
     * template content.
     *
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        var $oeCalls = this.$('[data-oe-call]');
        var oeCalls = _.uniq($oeCalls.map(function () {
            return $(this).data('oe-call');
        }).get());
        if (!oeCalls.length) {
            return def;
        }

        var renderDef = this._rpc({
            route: '/website/multi_render',
            params: {
                'ids_or_xml_ids': oeCalls,
            },
        }).then(function (data) {
            _.each(data, function (d, k) {
                var $data = $(d).addClass('o_block_' + k);
                $oeCalls.filter('[data-oe-call="' + k + '"]').each(function () {
                    $(this).replaceWith($data.clone());
                });
            });
        });

        return Promise.all([def, renderDef]);
    },
});
});

```

## File: static\src\js\content\menu.js

```javascript
odoo.define('website.content.menu', function (require) {
'use strict';

var dom = require('web.dom');
var publicWidget = require('web.public.widget');
var wUtils = require('website.utils');

publicWidget.registry.affixMenu = publicWidget.Widget.extend({
    selector: 'header.o_affix_enabled',

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        var self = this;
        this.$headerClone = this.$target.clone().addClass('o_header_affix affix').removeClass('o_affix_enabled').removeAttr('id');
        this.$headerClone.insertAfter(this.$target);
        this.$headers = this.$target.add(this.$headerClone);
        this.$dropdowns = this.$headers.find('.dropdown');
        this.$dropdownMenus = this.$headers.find('.dropdown-menu');
        this.$navbarCollapses = this.$headers.find('.navbar-collapse');

        this._adaptDefaultOffset();
        wUtils.onceAllImagesLoaded(this.$headerClone).then(function () {
            self._adaptDefaultOffset();
        });

        // Handle events for the collapse menus
        _.each(this.$headerClone.find('[data-toggle="collapse"]'), function (el) {
            var $source = $(el);
            var targetIDSelector = $source.attr('data-target');
            var $target = self.$headerClone.find(targetIDSelector);
            $source.attr('data-target', targetIDSelector + '_clone');
            $target.attr('id', targetIDSelector.substr(1) + '_clone');
        });
        // While scrolling through navbar menus, body should not be scrolled with it
        this.$headerClone.find('div.navbar-collapse').on('show.bs.collapse', function () {
            $(document.body).addClass('overflow-hidden');
        }).on('hide.bs.collapse', function () {
            $(document.body).removeClass('overflow-hidden');
        });

        // Window Handlers
        $(window).on('resize.affixMenu scroll.affixMenu', _.throttle(this._onWindowUpdate.bind(this), 200));
        setTimeout(this._onWindowUpdate.bind(this), 0); // setTimeout to allow override with advanced stuff... see themes

        return def.then(function () {
            self.trigger_up('widgets_start_request', {
                $target: self.$headerClone,
            });
        });
    },
    /**
     * @override
     */
    destroy: function () {
        if (this.$headerClone) {
            this.$headerClone.remove();
            $(window).off('.affixMenu');
        }
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptDefaultOffset: function () {
        var bottom = this.$target.offset().top + this._getHeaderHeight();
        this.$headerClone.css('margin-top', Math.min(-200, -bottom) + 'px');
    },
    /**
     * @private
     */
    _getHeaderHeight: function () {
        return this.$headerClone.outerHeight();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the window is resized or scrolled -> updates affix status and
     * automatically closes submenus.
     *
     * @private
     */
    _onWindowUpdate: function () {
        if (this.$navbarCollapses.hasClass('show')) {
            return;
        }

        var wOffset = $(window).scrollTop();
        var hOffset = this.$target.scrollTop();
        this.$headerClone.toggleClass('affixed', wOffset > (hOffset + 300));

        // Reset opened menus
        this.$dropdowns.add(this.$dropdownMenus).removeClass('show');
        this.$navbarCollapses.removeClass('show').attr('aria-expanded', false);
    },
});

/**
 * Auto adapt the header layout so that elements are not wrapped on a new line.
 *
 * Note: this works well with the affixMenu... by chance (autohideMenu is called
 * after alphabetically).
 */
publicWidget.registry.autohideMenu = publicWidget.Widget.extend({
    selector: 'header #top_menu',

    /**
     * @override
     */
    start: function () {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        this.noAutohide = this.$el.closest('.o_no_autohide_menu').length;
        if (!this.noAutohide) {
            var $navbar = this.$el.closest('.navbar');
            defs.push(wUtils.onceAllImagesLoaded($navbar));

            // The previous code will make sure we wait for images to be fully
            // loaded before initializing the auto more menu. But in some cases,
            // it is not enough, we also have to wait for fonts or even extra
            // scripts. Those will have no impact on the feature in most cases
            // though, so we will only update the auto more menu at that time,
            // no wait for it to initialize the feature.
            var $window = $(window);
            $window.on('load.autohideMenu', function () {
                $window.trigger('resize');
            });
        }
        return Promise.all(defs).then(function () {
            if (!self.noAutohide) {
                dom.initAutoMoreMenu(self.$el, {unfoldable: '.divider, .divider ~ li'});
            }
            self.$el.removeClass('o_menu_loading');
        });
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        if (!this.noAutohide) {
            $(window).off('.autohideMenu');
            dom.destroyAutoMoreMenu(this.$el);
        }
    },
});

/**
 * Note: this works well with the affixMenu... by chance (menuDirection is
 * called after alphabetically).
 *
 * @todo check bootstrap v4: maybe handled automatically now ?
 */
publicWidget.registry.menuDirection = publicWidget.Widget.extend({
    selector: 'header .navbar .nav',
    events: {
        'show.bs.dropdown': '_onDropdownShow',
    },

    /**
     * @override
     */
    start: function () {
        this.defaultAlignment = this.$el.is('.ml-auto, .ml-auto ~ *') ? 'right' : 'left';
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {string} alignment - either 'left' or 'right'
     * @param {integer} liOffset
     * @param {integer} liWidth
     * @param {integer} menuWidth
     * @returns {boolean}
     */
    _checkOpening: function (alignment, liOffset, liWidth, menuWidth, windowWidth) {
        if (alignment === 'left') {
            // Check if ok to open the dropdown to the right (no window overflow)
            return (liOffset + menuWidth <= windowWidth);
        } else {
            // Check if ok to open the dropdown to the left (no window overflow)
            return (liOffset + liWidth - menuWidth >= 0);
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onDropdownShow: function (ev) {
        var $li = $(ev.target);
        var $menu = $li.children('.dropdown-menu');
        var liOffset = $li.offset().left;
        var liWidth = $li.outerWidth();
        var menuWidth = $menu.outerWidth();
        var windowWidth = $(window).outerWidth();

        $menu.removeClass('dropdown-menu-left dropdown-menu-right');

        var alignment = this.defaultAlignment;
        if ($li.nextAll(':visible').length === 0) {
            // The dropdown is the last menu item, open to the left
            alignment = 'right';
        }

        // If can't open in the current direction because it would overflow the
        // window, change the direction. But if the other direction would do the
        // same, change back the direction.
        for (var i = 0 ; i < 2 ; i++) {
            if (!this._checkOpening(alignment, liOffset, liWidth, menuWidth, windowWidth)) {
                alignment = (alignment === 'left' ? 'right' : 'left');
            }
        }

        $menu.addClass('dropdown-menu-' + alignment);
    },
});
});

```

## File: static\src\js\content\snippets.animation.js

```javascript
odoo.define('website.content.snippets.animation', function (require) {
'use strict';

/**
 * Provides a way to start JS code for snippets' initialization and animations.
 */

const ajax = require('web.ajax');
var Class = require('web.Class');
var config = require('web.config');
var core = require('web.core');
var mixins = require('web.mixins');
var publicWidget = require('web.public.widget');
var utils = require('web.utils');

var qweb = core.qweb;

// Initialize fallbacks for the use of requestAnimationFrame,
// cancelAnimationFrame and performance.now()
window.requestAnimationFrame = window.requestAnimationFrame
    || window.webkitRequestAnimationFrame
    || window.mozRequestAnimationFrame
    || window.msRequestAnimationFrame
    || window.oRequestAnimationFrame;
window.cancelAnimationFrame = window.cancelAnimationFrame
    || window.webkitCancelAnimationFrame
    || window.mozCancelAnimationFrame
    || window.msCancelAnimationFrame
    || window.oCancelAnimationFrame;
if (!window.performance || !window.performance.now) {
    window.performance = {
        now: function () {
            return Date.now();
        }
    };
}

/**
 * Add the notion of edit mode to public widgets.
 */
publicWidget.Widget.include({
    /**
     * Indicates if the widget should not be instantiated in edit. The default
     * is true, indeed most (all?) defined widgets only want to initialize
     * events and states which should not be active in edit mode (this is
     * especially true for non-website widgets).
     *
     * @type {boolean}
     */
    disabledInEditableMode: true,
    /**
     * Acts as @see Widget.events except that the events are only binded if the
     * Widget instance is instanciated in edit mode. The property is not
     * considered if @see disabledInEditableMode is false.
     */
    edit_events: null,
    /**
     * Acts as @see Widget.events except that the events are only binded if the
     * Widget instance is instanciated in readonly mode. The property only
     * makes sense if @see disabledInEditableMode is false, you should simply
     * use @see Widget.events otherwise.
     */
    read_events: null,

    /**
     * Initializes the events that will need to be binded according to the
     * given mode.
     *
     * @constructor
     * @param {Object} parent
     * @param {Object} [options]
     * @param {boolean} [options.editableMode=false]
     *        true if the page is in edition mode
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);

        this.editableMode = this.options.editableMode || false;
        var extraEvents = this.editableMode ? this.edit_events : this.read_events;
        if (extraEvents) {
            this.events = _.extend({}, this.events || {}, extraEvents);
        }
    },
});

/**
 * In charge of handling one animation loop using the requestAnimationFrame
 * feature. This is used by the `Animation` class below and should not be called
 * directly by an end developer.
 *
 * This uses a simple API: it can be started, stopped, played and paused.
 */
var AnimationEffect = Class.extend(mixins.ParentedMixin, {
    /**
     * @constructor
     * @param {Object} parent
     * @param {function} updateCallback - the animation update callback
     * @param {string} [startEvents=scroll]
     *        space separated list of events which starts the animation loop
     * @param {jQuery|DOMElement} [$startTarget=window]
     *        the element(s) on which the startEvents are listened
     * @param {Object} [options]
     * @param {function} [options.getStateCallback]
     *        a function which returns a value which represents the state of the
     *        animation, i.e. for two same value, no refreshing of the animation
     *        is needed. Can be used for optimization. If the $startTarget is
     *        the window element, this defaults to returning the current
     *        scoll offset of the window or the size of the window for the
     *        scroll and resize events respectively.
     * @param {string} [options.endEvents]
     *        space separated list of events which pause the animation loop. If
     *        not given, the animation is stopped after a while (if no
     *        startEvents is received again)
     * @param {jQuery|DOMElement} [options.$endTarget=$startTarget]
     *        the element(s) on which the endEvents are listened
     */
    init: function (parent, updateCallback, startEvents, $startTarget, options) {
        mixins.ParentedMixin.init.call(this);
        this.setParent(parent);

        options = options || {};
        this._minFrameTime = 1000 / (options.maxFPS || 100);

        // Initialize the animation startEvents, startTarget, endEvents, endTarget and callbacks
        this._updateCallback = updateCallback;
        this.startEvents = startEvents || 'scroll';
        this.$startTarget = $($startTarget || window);
        if (options.getStateCallback) {
            this._getStateCallback = options.getStateCallback;
        } else if (this.startEvents === 'scroll' && this.$startTarget[0] === window) {
            this._getStateCallback = function () {
                return window.pageYOffset;
            };
        } else if (this.startEvents === 'resize' && this.$startTarget[0] === window) {
            this._getStateCallback = function () {
                return {
                    width: window.innerWidth,
                    height: window.innerHeight,
                };
            };
        } else {
            this._getStateCallback = function () {
                return undefined;
            };
        }
        this.endEvents = options.endEvents || false;
        this.$endTarget = options.$endTarget ? $(options.$endTarget) : this.$startTarget;

        this._updateCallback = this._updateCallback.bind(parent);
        this._getStateCallback = this._getStateCallback.bind(parent);

        // Add a namespace to events using the generated uid
        this._uid = '_animationEffect' + _.uniqueId();
        this.startEvents = _processEvents(this.startEvents, this._uid);
        if (this.endEvents) {
            this.endEvents = _processEvents(this.endEvents, this._uid);
        }

        function _processEvents(events, namespace) {
            events = events.split(' ');
            return _.each(events, function (e, index) {
                events[index] += ('.' + namespace);
            }).join(' ');
        }
    },
    /**
     * @override
     */
    destroy: function () {
        mixins.ParentedMixin.destroy.call(this);
        this.stop();
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Initializes when the animation must be played and paused and initializes
     * the animation first frame.
     */
    start: function () {
        // Initialize the animation first frame
        this._paused = false;
        this._rafID = window.requestAnimationFrame((function (t) {
            this._update(t);
            this._paused = true;
        }).bind(this));

        // Initialize the animation play/pause events
        if (this.endEvents) {
            /**
             * If there are endEvents, the animation should begin playing when
             * the startEvents are triggered on the $startTarget and pause when
             * the endEvents are triggered on the $endTarget.
             */
            this.$startTarget.on(this.startEvents, (function (e) {
                if (this._paused) {
                    _.defer(this.play.bind(this, e));
                }
            }).bind(this));
            this.$endTarget.on(this.endEvents, (function () {
                if (!this._paused) {
                    _.defer(this.pause.bind(this));
                }
            }).bind(this));
        } else {
            /**
             * Else, if there is no endEvents, the animation should begin playing
             * when the startEvents are *continuously* triggered on the
             * $startTarget or fully played once. To achieve this, the animation
             * begins playing and is scheduled to pause after 2 seconds. If the
             * startEvents are triggered during that time, this is not paused
             * for another 2 seconds. This allows to describe an "effect"
             * animation (which lasts less than 2 seconds) or an animation which
             * must be playing *during* an event (scroll, mousemove, resize,
             * repeated clicks, ...).
             */
            var pauseTimer = null;
            this.$startTarget.on(this.startEvents, _.throttle((function (e) {
                this.play(e);

                clearTimeout(pauseTimer);
                pauseTimer = _.delay((function () {
                    this.pause();
                    pauseTimer = null;
                }).bind(this), 2000);
            }).bind(this), 250, {trailing: false}));
        }
    },
    /**
     * Pauses the animation and destroys the attached events which trigger the
     * animation to be played or paused.
     */
    stop: function () {
        this.$startTarget.off(this.startEvents);
        if (this.endEvents) {
            this.$endTarget.off(this.endEvents);
        }
        this.pause();
    },
    /**
     * Forces the requestAnimationFrame loop to start.
     *
     * @param {Event} e - the event which triggered the animation to play
     */
    play: function (e) {
        this._newEvent = e;
        if (!this._paused) {
            return;
        }
        this._paused = false;
        this._rafID = window.requestAnimationFrame(this._update.bind(this));
        this._lastUpdateTimestamp = undefined;
    },
    /**
     * Forces the requestAnimationFrame loop to stop.
     */
    pause: function () {
        if (this._paused) {
            return;
        }
        this._paused = true;
        window.cancelAnimationFrame(this._rafID);
        this._lastUpdateTimestamp = undefined;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Callback which is repeatedly called by the requestAnimationFrame loop.
     * It controls the max fps at which the animation is running and initializes
     * the values that the update callback needs to describe the animation
     * (state, elapsedTime, triggered event).
     *
     * @private
     * @param {DOMHighResTimeStamp} timestamp
     */
    _update: function (timestamp) {
        if (this._paused) {
            return;
        }
        this._rafID = window.requestAnimationFrame(this._update.bind(this));

        // Check the elapsed time since the last update callback call.
        // Consider it 0 if there is no info of last timestamp and leave this
        // _update call if it was called too soon (would overflow the set max FPS).
        var elapsedTime = 0;
        if (this._lastUpdateTimestamp) {
            elapsedTime = timestamp - this._lastUpdateTimestamp;
            if (elapsedTime < this._minFrameTime) {
                return;
            }
        }

        // Check the new animation state thanks to the get state callback and
        // store its new value. If the state is the same as the previous one,
        // leave this _update call, except if there is an event which triggered
        // the "play" method again.
        var animationState = this._getStateCallback(elapsedTime, this._newEvent);
        if (!this._newEvent
         && animationState !== undefined
         && _.isEqual(animationState, this._animationLastState)) {
            return;
        }
        this._animationLastState = animationState;

        // Call the update callback with frame parameters
        this._updateCallback(this._animationLastState, elapsedTime, this._newEvent);
        this._lastUpdateTimestamp = timestamp; // Save the timestamp at which the update callback was really called
        this._newEvent = undefined; // Forget the event which triggered the last "play" call
    },
});

/**
 * Also register AnimationEffect automatically (@see effects, _prepareEffects).
 */
var Animation = publicWidget.Widget.extend({
    /**
     * The max FPS at which all the automatic animation effects will be
     * running by default.
     */
    maxFPS: 100,
    /**
     * @see this._prepareEffects
     *
     * @type {Object[]}
     * @type {string} startEvents
     *       The names of the events which trigger the effect to begin playing.
     * @type {string} [startTarget]
     *       A selector to find the target where to listen for the start events
     *       (if no selector, the window target will be used). If the whole
     *       $target of the animation should be used, use the 'selector' string.
     * @type {string} [endEvents]
     *       The name of the events which trigger the end of the effect (if none
     *       is defined, the animation will stop after a while
     *       @see AnimationEffect.start).
     * @type {string} [endTarget]
     *       A selector to find the target where to listen for the end events
     *       (if no selector, the startTarget will be used). If the whole
     *       $target of the animation should be used, use the 'selector' string.
     * @type {string} update
     *       A string which refers to a method which will be used as the update
     *       callback for the effect. It receives 3 arguments: the animation
     *       state, the elapsedTime since last update and the event which
     *       triggered the animation (undefined if just a new update call
     *       without trigger).
     * @type {string} [getState]
     *       The animation state is undefined by default, the scroll offset for
     *       the particular {startEvents: 'scroll'} effect and an object with
     *       width and height for the particular {startEvents: 'resize'} effect.
     *       There is the possibility to define the getState callback of the
     *       animation effect with this key. This allows to improve performance
     *       even further in some cases.
     */
    effects: [],

    /**
     * Initializes the animation. The method should not be called directly as
     * called automatically on animation instantiation and on restart.
     *
     * Also, prepares animation's effects and start them if any.
     *
     * @override
     */
    start: function () {
        this._prepareEffects();
        _.each(this._animationEffects, function (effect) {
            effect.start();
        });
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Registers `AnimationEffect` instances.
     *
     * This can be done by extending this method and calling the @see _addEffect
     * method in it or, better, by filling the @see effects property.
     *
     * @private
     */
    _prepareEffects: function () {
        this._animationEffects = [];

        var self = this;
        _.each(this.effects, function (desc) {
            self._addEffect(self[desc.update], desc.startEvents, _findTarget(desc.startTarget), {
                getStateCallback: desc.getState && self[desc.getState],
                endEvents: desc.endEvents || undefined,
                $endTarget: _findTarget(desc.endTarget),
                maxFPS: self.maxFPS,
            });

            // Return the DOM element matching the selector in the form
            // described above.
            function _findTarget(selector) {
                if (selector) {
                    if (selector === 'selector') {
                        return self.$target;
                    }
                    return self.$(selector);
                }
                return undefined;
            }
        });
    },
    /**
     * Registers a new `AnimationEffect` according to given parameters.
     *
     * @private
     * @see AnimationEffect.init
     */
    _addEffect: function (updateCallback, startEvents, $startTarget, options) {
        this._animationEffects.push(
            new AnimationEffect(this, updateCallback, startEvents, $startTarget, options)
        );
    },
});

//::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

var registry = publicWidget.registry;

registry.slider = publicWidget.Widget.extend({
    selector: '.carousel',
    disabledInEditableMode: false,
    edit_events: {
        'slid.bs.carousel': '_onEditionSlide',
    },

    /**
     * @override
     */
    start: function () {
        if (!this.editableMode) {
            this.$('img').on('load.slider', this._onImageLoaded.bind(this));
            this._computeHeights();
        }
        this.$target.carousel();
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        this.$('img').off('.slider');
        this.$target.carousel('pause');
        this.$target.removeData('bs.carousel');
        _.each(this.$('.carousel-item'), function (el) {
            $(el).css('min-height', '');
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _computeHeights: function () {
        var maxHeight = 0;
        var $items = this.$('.carousel-item');
        _.each($items, function (el) {
            var $item = $(el);
            var isActive = $item.hasClass('active');
            $item.addClass('active');
            var height = $item.outerHeight();
            if (height > maxHeight) {
                maxHeight = height;
            }
            $item.toggleClass('active', isActive);
        });
        _.each($items, function (el) {
            $(el).css('min-height', maxHeight);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onEditionSlide: function () {
        this._computeHeights();
    },
    /**
     * @private
     */
    _onImageLoaded: function () {
        this._computeHeights();
    },
});

registry.parallax = Animation.extend({
    selector: '.parallax',
    disabledInEditableMode: false,
    effects: [{
        startEvents: 'scroll',
        update: '_onWindowScroll',
    }],

    /**
     * @override
     */
    start: function () {
        this._rebuild();
        $(window).on('resize.animation_parallax', _.debounce(this._rebuild.bind(this), 500));
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        $(window).off('.animation_parallax');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Prepares the background element which will scroll at a different speed
     * according to the viewport dimensions and other snippet parameters.
     *
     * @private
     */
    _rebuild: function () {
        // Add/find bg DOM element to hold the parallax bg (support old v10.0 parallax)
        if (!this.$bg || !this.$bg.length) {
            this.$bg = this.$('> .s_parallax_bg');
            if (!this.$bg.length) {
                this.$bg = $('<span/>', {
                    class: 's_parallax_bg' + (this.$target.hasClass('oe_custom_bg') ? ' oe_custom_bg' : ''),
                }).prependTo(this.$target);
            }
        }
        var urlTarget = this.$target.css('background-image');
        if (urlTarget !== 'none') {
            this.$bg.css('background-image', urlTarget);
        }
        this.$target.css('background-image', 'none');

        // Get parallax speed
        this.speed = parseFloat(this.$target.attr('data-scroll-background-ratio') || 0);

        // Reset offset if parallax effect will not be performed and leave
        this.$target.toggleClass('s_parallax_is_fixed', this.speed === 1);
        var noParallaxSpeed = (this.speed === 0 || this.speed === 1);
        this.$target.toggleClass('s_parallax_no_overflow_hidden', noParallaxSpeed);
        if (noParallaxSpeed) {
            this.$bg.css({
                transform: '',
                top: '',
                bottom: '',
            });
            return;
        }

        // Initialize parallax data according to snippet and viewport dimensions
        this.viewport = document.body.clientHeight - $('#wrapwrap').position().top;
        this.visibleArea = [this.$target.offset().top];
        this.visibleArea.push(this.visibleArea[0] + this.$target.innerHeight() + this.viewport);
        this.ratio = this.speed * (this.viewport / 10);

        // Provide a "safe-area" to limit parallax
        this.$bg.css({
            top: -this.ratio,
            bottom: -this.ratio,
        });
    },

    //--------------------------------------------------------------------------
    // Effects
    //--------------------------------------------------------------------------

    /**
     * Describes how to update the snippet when the window scrolls.
     *
     * @private
     * @param {integer} scrollOffset
     */
    _onWindowScroll: function (scrollOffset) {
        // Speed == 0 is no effect and speed == 1 is handled by CSS only
        if (this.speed === 0 || this.speed === 1) {
            return;
        }

        // Perform translation if the element is visible only
        var vpEndOffset = scrollOffset + this.viewport;
        if (vpEndOffset >= this.visibleArea[0]
         && vpEndOffset <= this.visibleArea[1]) {
            this.$bg.css('transform', 'translateY(' + _getNormalizedPosition.call(this, vpEndOffset) + 'px)');
        }

        function _getNormalizedPosition(pos) {
            // Normalize scroll in a 1 to 0 range
            var r = (pos - this.visibleArea[1]) / (this.visibleArea[0] - this.visibleArea[1]);
            // Normalize accordingly to current options
            return Math.round(this.ratio * (2 * r - 1));
        }
    },
});

registry.share = publicWidget.Widget.extend({
    selector: '.s_share, .oe_share', // oe_share for compatibility

    /**
     * @override
     */
    start: function () {
        var urlRegex = /(\?(?:|.*&)(?:u|url|body)=)(.*?)(&|#|$)/;
        var titleRegex = /(\?(?:|.*&)(?:title|text|subject)=)(.*?)(&|#|$)/;
        var url = encodeURIComponent(window.location.href);
        var title = encodeURIComponent($('title').text());
        this.$('a').each(function () {
            var $a = $(this);
            $a.attr('href', function (i, href) {
                return href.replace(urlRegex, function (match, a, b, c) {
                    return a + url + c;
                }).replace(titleRegex, function (match, a, b, c) {
                    return a + title + c;
                });
            });
            if ($a.attr('target') && $a.attr('target').match(/_blank/i) && !$a.closest('.o_editable').length) {
                $a.on('click', function () {
                    window.open(this.href, '', 'menubar=no,toolbar=no,resizable=yes,scrollbars=yes,height=550,width=600');
                    return false;
                });
            }
        });

        return this._super.apply(this, arguments);
    },
});

const MobileYoutubeAutoplayMixin = {
    /**
     * Takes care of any necessary setup for autoplaying video. In practice,
     * this method will load the youtube iframe API for mobile environments
     * because mobile environments don't support the youtube autoplay param
     * passed in the url.
     *
     * @private
     * @param {string} src - The source url of the video
     */
    _setupAutoplay: function (src) {
        let promise = Promise.resolve();

        this.isYoutubeVideo = src.indexOf('youtube') >= 0;
        this.isMobileEnv = config.device.size_class <= config.device.SIZES.LG && config.device.touch;

        if (this.isYoutubeVideo && this.isMobileEnv && !window.YT) {
            const oldOnYoutubeIframeAPIReady = window.onYouTubeIframeAPIReady;
            promise = new Promise(resolve => {
                window.onYouTubeIframeAPIReady = () => {
                    if (oldOnYoutubeIframeAPIReady) {
                        oldOnYoutubeIframeAPIReady();
                    }
                    return resolve();
                };
            });
            ajax.loadJS('https://www.youtube.com/iframe_api');
        }

        return promise;
    },
    /**
     * @private
     * @param {DOMElement} iframeEl - the iframe containing the video player
     */
    _triggerAutoplay: function (iframeEl) {
        // YouTube does not allow to auto-play video in mobile devices, so we
        // have to play the video manually.
        if (this.isMobileEnv && this.isYoutubeVideo) {
            new window.YT.Player(iframeEl, {
                events: {
                    onReady: ev => ev.target.playVideo(),
                }
            });
        }
    },
};

registry.mediaVideo = publicWidget.Widget.extend(MobileYoutubeAutoplayMixin, {
    selector: '.media_iframe_video',

    /**
     * @override
     */
    start: function () {
        // TODO: this code should be refactored to make more sense and be better
        // integrated with Odoo (this refactoring should be done in master).

        const proms = [this._super.apply(this, arguments)];
        let iframeEl = this.$target[0].querySelector(':scope > iframe');

        // The following code is only there to ensure compatibility with
        // videos added before bug fixes or new Odoo versions where the
        // <iframe/> element is properly saved.
        if (!iframeEl) {
            iframeEl = this._generateIframe();
        }

        if (!iframeEl) {
            // Something went wrong: no iframe is present in the DOM and the
            // widget was unable to create one on the fly.
            return Promise.all(proms);
        }

        proms.push(this._setupAutoplay(iframeEl.getAttribute('src')));
        return Promise.all(proms).then(() => {
            this._triggerAutoplay(iframeEl);
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _generateIframe: function () {
        // Bug fix / compatibility: empty the <div/> element as all information
        // to rebuild the iframe should have been saved on the <div/> element
        this.$target.empty();

        // Add extra content for size / edition
        this.$target.append(
            '<div class="css_editable_mode_display">&nbsp;</div>' +
            '<div class="media_iframe_video_size">&nbsp;</div>'
        );

        // Rebuild the iframe. Depending on version / compatibility / instance,
        // the src is saved in the 'data-src' attribute or the
        // 'data-oe-expression' one (the latter is used as a workaround in 10.0
        // system but should obviously be reviewed in master).
        var src = _.escape(this.$target.data('oe-expression') || this.$target.data('src'));
        // Validate the src to only accept supported domains we can trust
        var m = src.match(/^(?:https?:)?\/\/([^/?#]+)/);
        if (!m) {
            // Unsupported protocol or wrong URL format, don't inject iframe
            return;
        }
        var domain = m[1].replace(/^www\./, '');
        var supportedDomains = ['youtu.be', 'youtube.com', 'youtube-nocookie.com', 'instagram.com', 'vine.co', 'player.vimeo.com', 'vimeo.com', 'dailymotion.com', 'player.youku.com', 'youku.com'];
        if (!_.contains(supportedDomains, domain)) {
            // Unsupported domain, don't inject iframe
            return;
        }
        const iframeEl = $('<iframe/>', {
            src: src,
            frameborder: '0',
            allowfullscreen: 'allowfullscreen',
        })[0];
        this.$target.append(iframeEl);
        return iframeEl;
    },
});

registry.backgroundVideo = publicWidget.Widget.extend(MobileYoutubeAutoplayMixin, {
    selector: '.o_background_video',
    xmlDependencies: ['/website/static/src/xml/website.background.video.xml'],
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var proms = [this._super(...arguments)];

        this.videoSrc = this.el.dataset.bgVideoSrc;
        this.iframeID = _.uniqueId('o_bg_video_iframe_');
        proms.push(this._setupAutoplay(this.videoSrc));
        if (this.isYoutubeVideo && this.isMobileEnv && !this.videoSrc.includes('enablejsapi=1')) {
            // Compatibility: when choosing an autoplay youtube video via the
            // media manager, the API was not automatically enabled before but
            // only enabled here in the case of background videos.
            // TODO migrate those old cases so this code can be removed?
            this.videoSrc += '&enablejsapi=1';
        }

        var throttledUpdate = _.throttle(() => this._adjustIframe(), 50);

        var $dropdownMenu = this.$el.closest('.dropdown-menu');
        if ($dropdownMenu.length) {
            this.$dropdownParent = $dropdownMenu.parent();
            this.$dropdownParent.on('shown.bs.dropdown.backgroundVideo', throttledUpdate);
        }

        $(window).on('resize.' + this.iframeID, throttledUpdate);

        return Promise.all(proms).then(() => this._appendBgVideo());
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);

        if (this.$dropdownParent) {
            this.$dropdownParent.off('.backgroundVideo');
        }

        $(window).off('resize.' + this.iframeID);

        if (this.$bgVideoContainer) {
            this.$bgVideoContainer.remove();
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adjusts iframe sizes and position so that it fills the container and so
     * that it is centered in it.
     *
     * @private
     */
    _adjustIframe: function () {
        if (!this.$iframe) {
            return;
        }

        this.$iframe.removeClass('show');

        // Adjust the iframe
        var wrapperWidth = this.$target.innerWidth();
        var wrapperHeight = this.$target.innerHeight();
        var relativeRatio = (wrapperWidth / wrapperHeight) / (16 / 9);
        var style = {};
        if (relativeRatio >= 1.0) {
            style['width'] = '100%';
            style['height'] = (relativeRatio * 100) + '%';
            style['left'] = '0';
            style['top'] = (-(relativeRatio - 1.0) / 2 * 100) + '%';
        } else {
            style['width'] = ((1 / relativeRatio) * 100) + '%';
            style['height'] = '100%';
            style['left'] = (-((1 / relativeRatio) - 1.0) / 2 * 100) + '%';
            style['top'] = '0';
        }
        this.$iframe.css(style);

        void this.$iframe[0].offsetWidth; // Force style addition
        this.$iframe.addClass('show');
    },
    /**
     * Append background video related elements to the target.
     *
     * @private
     */
    _appendBgVideo: function () {
        var $oldContainer = this.$bgVideoContainer || this.$('> .o_bg_video_container');
        this.$bgVideoContainer = $(qweb.render('website.background.video', {
            videoSrc: this.videoSrc,
            iframeID: this.iframeID,
        }));
        this.$iframe = this.$bgVideoContainer.find('.o_bg_video_iframe');
        this.$iframe.one('load', () => {
            this.$bgVideoContainer.find('.o_bg_video_loading').remove();
        });
        this.$bgVideoContainer.prependTo(this.$target);
        $oldContainer.remove();

        this._adjustIframe();
        this._triggerAutoplay(this.$iframe[0]);
    },
});

registry.ul = publicWidget.Widget.extend({
    selector: 'ul.o_ul_folded, ol.o_ul_folded',
    events: {
        'click .o_ul_toggle_next': '_onToggleNextClick',
        'click .o_ul_toggle_self': '_onToggleSelfClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when a "toggle next" ul is clicked.
     *
     * @private
     */
    _onToggleNextClick: function (ev) {
        ev.preventDefault();
        var $target = $(ev.currentTarget);
        $target.toggleClass('o_open');
        $target.closest('li').next().toggleClass('o_close');
    },
    /**
     * Called when a "toggle self" ul is clicked.
     *
     * @private
     */
    _onToggleSelfClick: function (ev) {
        ev.preventDefault();
        var $target = $(ev.currentTarget);
        $target.toggleClass('o_open');
        $target.closest('li').find('ul,ol').toggleClass('o_close');
    },
});

registry.gallery = publicWidget.Widget.extend({
    selector: '.o_gallery:not(.o_slideshow)',
    xmlDependencies: ['/website/static/src/xml/website.gallery.xml'],
    events: {
        'click img': '_onClickImg',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when an image is clicked. Opens a dialog to browse all the images
     * with a bigger size.
     *
     * @private
     * @param {Event} ev
     */
    _onClickImg: function (ev) {
        var self = this;
        var $cur = $(ev.currentTarget);

        var urls = [];
        var idx = undefined;
        var milliseconds = undefined;
        var params = undefined;
        var $images = $cur.closest('.o_gallery').find('img');
        var size = 0.8;
        var dimensions = {
            min_width: Math.round(window.innerWidth * size * 0.9),
            min_height: Math.round(window.innerHeight * size),
            max_width: Math.round(window.innerWidth * size * 0.9),
            max_height: Math.round(window.innerHeight * size),
            width: Math.round(window.innerWidth * size * 0.9),
            height: Math.round(window.innerHeight * size)
        };

        $images.each(function () {
            urls.push($(this).attr('src'));
        });
        var $img = ($cur.is('img') === true) ? $cur : $cur.closest('img');
        idx = urls.indexOf($img.attr('src'));

        milliseconds = $cur.closest('.o_gallery').data('interval') || false;
        var $modal = $(qweb.render('website.gallery.slideshow.lightbox', {
            srcs: urls,
            index: idx,
            dim: dimensions,
            interval: milliseconds,
            id: _.uniqueId('slideshow_'),
        }));
        $modal.modal({
            keyboard: true,
            backdrop: true,
        });
        $modal.on('hidden.bs.modal', function () {
            $(this).hide();
            $(this).siblings().filter('.modal-backdrop').remove(); // bootstrap leaves a modal-backdrop
            $(this).remove();
        });
        $modal.find('.modal-content, .modal-body.o_slideshow').css('height', '100%');
        $modal.appendTo(document.body);

        $modal.one('shown.bs.modal', function () {
            self.trigger_up('widgets_start_request', {
                editableMode: false,
                $target: $modal.find('.modal-body.o_slideshow'),
            });
        });
    },
});

registry.gallerySlider = publicWidget.Widget.extend({
    selector: '.o_slideshow',
    xmlDependencies: ['/website/static/src/xml/website.gallery.xml'],
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$carousel = this.$target.is('.carousel') ? this.$target : this.$target.find('.carousel');
        this.$indicator = this.$carousel.find('.carousel-indicators');
        this.$prev = this.$indicator.find('li.o_indicators_left').css('visibility', ''); // force visibility as some databases have it hidden
        this.$next = this.$indicator.find('li.o_indicators_right').css('visibility', '');
        var $lis = this.$indicator.find('li[data-slide-to]');
        var nbPerPage = Math.floor(this.$indicator.width() / $lis.first().outerWidth(true)) - 3; // - navigator - 1 to leave some space
        var realNbPerPage = nbPerPage || 1;
        var nbPages = Math.ceil($lis.length / realNbPerPage);

        var index;
        var page;
        update();

        function hide() {
            $lis.each(function (i) {
                $(this).toggleClass('d-none', i < page * nbPerPage || i >= (page + 1) * nbPerPage);
            });
            if (self.editableMode) { // do not remove DOM in edit mode
                return;
            }
            if (page <= 0) {
                self.$prev.detach();
            } else {
                self.$prev.prependTo(self.$indicator);
            }
            if (page >= nbPages - 1) {
                self.$next.detach();
            } else {
                self.$next.appendTo(self.$indicator);
            }
        }

        function update() {
            const active = $lis.filter('.active');
            index = active.length ? $lis.index(active) : 0;
            page = Math.floor(index / realNbPerPage);
            hide();
        }

        this.$carousel.on('slide.bs.carousel.gallery_slider', function () {
            setTimeout(function () {
                var $item = self.$carousel.find('.carousel-inner .carousel-item-prev, .carousel-inner .carousel-item-next');
                var index = $item.index();
                $lis.removeClass('active')
                    .filter('[data-slide-to="' + index + '"]')
                    .addClass('active');
            }, 0);
        });
        this.$indicator.on('click.gallery_slider', '> li:not([data-slide-to])', function () {
            page += ($(this).hasClass('o_indicators_left') ? -1 : 1);
            page = Math.max(0, Math.min(nbPages - 1, page)); // should not be necessary
            self.$carousel.carousel(page * realNbPerPage);
            hide();
        });
        this.$carousel.on('slid.bs.carousel.gallery_slider', update);

        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);

        if (!this.$indicator) {
            return;
        }

        this.$prev.prependTo(this.$indicator);
        this.$next.appendTo(this.$indicator);
        this.$carousel.off('.gallery_slider');
        this.$indicator.off('.gallery_slider');
    },
});

registry.socialShare = publicWidget.Widget.extend({
    selector: '.oe_social_share',
    xmlDependencies: ['/website/static/src/xml/website.share.xml'],
    events: {
        'mouseenter': '_onMouseEnter',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _bindSocialEvent: function () {
        this.$('.oe_social_facebook').click($.proxy(this._renderSocial, this, 'facebook'));
        this.$('.oe_social_twitter').click($.proxy(this._renderSocial, this, 'twitter'));
        this.$('.oe_social_linkedin').click($.proxy(this._renderSocial, this, 'linkedin'));
    },
    /**
     * @private
     */
    _render: function () {
        this.$el.popover({
            content: qweb.render('website.social_hover', {medias: this.socialList}),
            placement: 'bottom',
            container: this.$el,
            html: true,
            trigger: 'manual',
            animation: false,
        }).popover("show");

        this.$el.off('mouseleave.socialShare').on('mouseleave.socialShare', function () {
            var self = this;
            setTimeout(function () {
                if (!$(".popover:hover").length) {
                    $(self).popover('dispose');
                }
            }, 200);
        });
    },
    /**
     * @private
     */
    _renderSocial: function (social) {
        var url = this.$el.data('urlshare') || document.URL.split(/[?#]/)[0];
        url = encodeURIComponent(url);
        var title = document.title.split(" | ")[0];  // get the page title without the company name
        var hashtags = ' #' + document.title.split(" | ")[1].replace(' ', '') + ' ' + this.hashtags;  // company name without spaces (for hashtag)
        var socialNetworks = {
            'facebook': 'https://www.facebook.com/sharer/sharer.php?u=' + url,
            'twitter': 'https://twitter.com/intent/tweet?original_referer=' + url + '&text=' + encodeURIComponent(title + hashtags + ' - ') + url,
            'linkedin': 'https://www.linkedin.com/shareArticle?mini=true&url=' + url + '&title=' + encodeURIComponent(title),
        };
        if (!_.contains(_.keys(socialNetworks), social)) {
            return;
        }
        var wHeight = 500;
        var wWidth = 500;
        window.open(socialNetworks[social], '', 'menubar=no, toolbar=no, resizable=yes, scrollbar=yes, height=' + wHeight + ',width=' + wWidth);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the user hovers the animation element -> open the social
     * links popover.
     *
     * @private
     */
    _onMouseEnter: function () {
        var social = this.$el.data('social');
        this.socialList = social ? social.split(',') : ['facebook', 'twitter', 'linkedin'];
        this.hashtags = this.$el.data('hashtags') || '';

        this._render();
        this._bindSocialEvent();
    },
});

registry.facebookPage = publicWidget.Widget.extend({
    selector: '.o_facebook_page',
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        var params = _.pick(this.$el.data(), 'href', 'height', 'tabs', 'small_header', 'hide_cover', 'show_facepile');
        if (!params.href) {
            return def;
        }
        params.width = utils.confine(Math.floor(this.$el.width()), 180, 500);

        var src = $.param.querystring('https://www.facebook.com/plugins/page.php', params);
        this.$iframe = $('<iframe/>', {
            src: src,
            class: 'o_temp_auto_element',
            width: params.width,
            height: params.height,
            css: {
                border: 'none',
                overflow: 'hidden',
            },
            scrolling: 'no',
            frameborder: '0',
            allowTransparency: 'true',
        });
        this.$el.append(this.$iframe);

        return def;
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);

        if (this.$iframe) {
            this.$iframe.remove();
        }
    },
});

registry.anchorSlide = publicWidget.Widget.extend({
    selector: 'a[href^="/"][href*="#"], a[href^="#"]',
    events: {
        'click': '_onAnimateClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onAnimateClick: function (ev) {
        if (this.$target[0].pathname !== window.location.pathname) {
            return;
        }
        var hash = this.$target[0].hash;
        if (!utils.isValidAnchor(hash)) {
            return;
        }
        var $anchor = $(hash);
        if (!$anchor.length || !$anchor.attr('data-anchor')) {
            return;
        }
        ev.preventDefault();
        $('html, body').animate({
            scrollTop: $anchor.offset().top,
        }, 500);
    },
});

return {
    Widget: publicWidget.Widget,
    Animation: Animation,
    registry: registry,

    Class: Animation, // Deprecated
};
});

```

## File: static\src\js\content\website_root.js

```javascript
odoo.define('website.root', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var publicRootData = require('web.public.root');
require("web.zoomodoo");

var _t = core._t;

var websiteRootRegistry = publicRootData.publicRootRegistry;

var WebsiteRoot = publicRootData.PublicRoot.extend({
    events: _.extend({}, publicRootData.PublicRoot.prototype.events || {}, {
        'click .js_change_lang': '_onLangChangeClick',
        'click .js_publish_management .js_publish_btn': '_onPublishBtnClick',
        'click .js_multi_website_switch': '_onWebsiteSwitch',
        'shown.bs.modal': '_onModalShown',
    }),
    custom_events: _.extend({}, publicRootData.PublicRoot.prototype.custom_events || {}, {
        'ready_to_clean_for_save': '_onWidgetsStopRequest',
        'will_remove_snippet': '_onWidgetsStopRequest',
        seo_object_request: '_onSeoObjectRequest',
    }),

    /**
     * @override
     */
    start: function () {
        // Compatibility lang change ?
        if (!this.$('.js_change_lang').length) {
            var $links = this.$('ul.js_language_selector li a:not([data-oe-id])');
            var m = $(_.min($links, function (l) {
                return $(l).attr('href').length;
            })).attr('href');
            $links.each(function () {
                var $link = $(this);
                var t = $link.attr('href');
                var l = (t === m) ? "default" : t.split('/')[1];
                $link.data('lang', l).addClass('js_change_lang');
            });
        }

        // Enable magnify on zommable img
        this.$('.zoomable img[data-zoom]').zoomOdoo();

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getContext: function (context) {
        var html = document.documentElement;
        return _.extend({
            'website_id': html.getAttribute('data-website-id') | 0,
        }, this._super.apply(this, arguments));
    },
    /**
     * @override
     */
    _getExtraContext: function (context) {
        var html = document.documentElement;
        return _.extend({
            'editable': !!(html.dataset.editable || $('[data-oe-model]').length), // temporary hack, this should be done in python
            'translatable': !!html.dataset.translatable,
            'edit_translations': !!html.dataset.edit_translations,
        }, this._super.apply(this, arguments));
    },
    /**
     * @override
     */
    _getPublicWidgetsRegistry: function (options) {
        var registry = this._super.apply(this, arguments);
        if (options.editableMode) {
            return _.pick(registry, function (PublicWidget) {
                return !PublicWidget.prototype.disabledInEditableMode;
            });
        }
        return registry;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onWidgetsStartRequest: function (ev) {
        ev.data.options = _.clone(ev.data.options || {});
        ev.data.options.editableMode = ev.data.editableMode;
        this._super.apply(this, arguments);
    },
    /**
     * @todo review
     * @private
     */
    _onLangChangeClick: function (ev) {
        ev.preventDefault();

        var $target = $(ev.currentTarget);
        // retrieve the hash before the redirect
        var redirect = {
            lang: $target.data('url_code'),
            url: encodeURIComponent($target.attr('href').replace(/[&?]edit_translations[^&?]+/, '')),
            hash: encodeURIComponent(window.location.hash)
        };
        window.location.href = _.str.sprintf("/website/lang/%(lang)s?r=%(url)s%(hash)s", redirect);
    },
    /**
    /**
     * Checks information about the page SEO object.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onSeoObjectRequest: function (ev) {
        var res = this._unslugHtmlDataObject('seo-object');
        ev.data.callback(res);
    },
    /**
     * Returns a model/id object constructed from html data attribute.
     *
     * @private
     * @param {string} dataAttr
     * @returns {Object} an object with 2 keys: model and id, or null
     * if not found
     */
    _unslugHtmlDataObject: function (dataAttr) {
        var repr = $('html').data(dataAttr);
        var match = repr && repr.match(/(.+)\((\d+),(.*)\)/);
        if (!match) {
            return null;
        }
        return {
            model: match[1],
            id: match[2] | 0,
        };
    },
    /**
     * @todo review
     * @private
     */
    _onPublishBtnClick: function (ev) {
        ev.preventDefault();

        var $data = $(ev.currentTarget).parents(".js_publish_management:first");
        this._rpc({
            route: $data.data('controller') || '/website/publish',
            params: {
                id: +$data.data('id'),
                object: $data.data('object'),
            },
        })
        .then(function (result) {
            $data.toggleClass("css_published", result).toggleClass("css_unpublished", !result);
            $data.find('input').prop("checked", result);
            $data.parents("[data-publish]").attr("data-publish", +result ? 'on' : 'off');
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onWebsiteSwitch: function (ev) {
        var websiteId = ev.currentTarget.getAttribute('website-id');
        var websiteDomain = ev.currentTarget.getAttribute('domain');
        let url = `/website/force/${websiteId}`;
        if (websiteDomain && window.location.hostname !== websiteDomain) {
            url = websiteDomain + url;
        }
        const path = window.location.pathname + window.location.search + window.location.hash;
        window.location.href = $.param.querystring(url, {'path': path});
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onModalShown: function (ev) {
        $(ev.target).addClass('modal_shown');
    },
});

return {
    WebsiteRoot: WebsiteRoot,
    websiteRootRegistry: websiteRootRegistry,
};
});

```

## File: static\src\js\content\website_root_instance.js

```javascript
odoo.define('root.widget', function (require) {
'use strict';

var lazyloader = require('web.public.lazyloader');
var websiteRootData = require('website.root');

var websiteRoot = new websiteRootData.WebsiteRoot(null);
return lazyloader.allScriptsLoaded.then(function () {
    return websiteRoot.attachTo(document.body).then(function () {
        return websiteRoot;
    });
});
});

```

## File: static\src\js\editor\editor.js

```javascript
odoo.define('website.editor', function (require) {
'use strict';

var weWidgets = require('web_editor.widget');
var wUtils = require('website.utils');

weWidgets.LinkDialog.include({
    /**
     * Allows the URL input to propose existing website pages.
     *
     * @override
     */
    start: function () {
        wUtils.autocompleteWithPages(this, this.$('input[name="url"]'));
        return this._super.apply(this, arguments);
    },
});
});

```

## File: static\src\js\editor\editor_menu.js

```javascript
odoo.define('website.editor.menu', function (require) {
'use strict';

var Dialog = require('web.Dialog');
var Widget = require('web.Widget');
var core = require('web.core');
var Wysiwyg = require('web_editor.wysiwyg.root');

var _t = core._t;

var WysiwygMultizone = Wysiwyg.extend({
    assetLibs: Wysiwyg.prototype.assetLibs.concat(['website.compiled_assets_wysiwyg']),
    _getWysiwygContructor: function () {
        return odoo.__DEBUG__.services['web_editor.wysiwyg.multizone'];
    }
});

var EditorMenu = Widget.extend({
    template: 'website.editorbar',
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],
    events: {
        'click button[data-action=save]': '_onSaveClick',
        'click button[data-action=cancel]': '_onCancelClick',
    },
    custom_events: {
        request_save: '_onSnippetRequestSave',
        get_clean_html: '_onGetCleanHTML',
    },

    /**
     * @override
     */
    willStart: function () {
        var self = this;
        this.$el = null; // temporary null to avoid hidden error (@see start)
        return this._super()
            .then(function () {
                var $wrapwrap = $('#wrapwrap');
                $wrapwrap.removeClass('o_editable'); // clean the dom before edition
                self.editable($wrapwrap).addClass('o_editable');
                self.wysiwyg = self._wysiwygInstance();
            });
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$el.css({width: '100%'});
        return this.wysiwyg.attachTo($('#wrapwrap')).then(function () {
            self.trigger_up('edit_mode');
            self.$el.css({width: ''});
        });
    },
    /**
     * @override
     */
    destroy: function () {
        this.trigger_up('readonly_mode');
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Asks the user if they really wants to discard their changes (if any),
     * then simply reloads the page if they want to.
     *
     * @param {boolean} [reload=true]
     *        true if the page has to be reloaded when the user answers yes
     *        (do nothing otherwise but add this to allow class extension)
     * @returns {Deferred}
     */
    cancel: function (reload) {
        var self = this;
        var def = new Promise(function (resolve, reject) {
            if (!self.wysiwyg.isDirty()) {
                resolve();
            } else {
                var confirm = Dialog.confirm(self, _t("If you discard the current edition, all unsaved changes will be lost. You can cancel to return to the edition mode."), {
                    confirm_callback: resolve,
                });
                confirm.on('closed', self, reject);
            }
        });

        return def.then(function () {
            self.trigger_up('edition_will_stopped');
            var $wrapwrap = $('#wrapwrap');
            self.editable($wrapwrap).removeClass('o_editable');
            if (reload !== false) {
                window.onbeforeunload = null;
                self.wysiwyg.destroy();
                return self._reload();
            } else {
                self.wysiwyg.destroy();
                self.trigger_up('readonly_mode');
                self.trigger_up('edition_was_stopped');
                self.destroy();
            }
        });
    },
    /**
     * Asks the snippets to clean themself, then saves the page, then reloads it
     * if asked to.
     *
     * @param {boolean} [reload=true]
     *        true if the page has to be reloaded after the save
     * @returns {Deferred}
     */
    save: function (reload) {
        var self = this;
        this.trigger_up('edition_will_stopped');
        return this.wysiwyg.save(false).then(function (result) {
            var $wrapwrap = $('#wrapwrap');
            self.editable($wrapwrap).removeClass('o_editable');
            if (result.isDirty && reload !== false) {
                // remove top padding because the connected bar is not visible
                $('body').removeClass('o_connected_user');
                return self._reload();
            } else {
                self.wysiwyg.destroy();
                self.trigger_up('edition_was_stopped');
                self.destroy();
            }
        });
    },
    /**
     * Returns the editable areas on the page.
     *
     * @param {DOM} $wrapwrap
     * @returns {jQuery}
     */
    editable: function ($wrapwrap) {
        return $wrapwrap.find('[data-oe-model]')
            .not('.o_not_editable')
            .filter(function () {
                var $parent = $(this).closest('.o_editable, .o_not_editable');
                return !$parent.length || $parent.hasClass('o_editable');
            })
            .not('link, script')
            .not('[data-oe-readonly]')
            .not('img[data-oe-field="arch"], br[data-oe-field="arch"], input[data-oe-field="arch"]')
            .not('.oe_snippet_editor')
            .not('hr, br, input, textarea')
            .add('.o_editable');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _wysiwygInstance: function () {
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        return new WysiwygMultizone(this, {
            snippets: 'website.snippets',
            recordInfo: {
                context: context,
                data_res_model: 'website',
                data_res_id: context.website_id,
            }
        });
    },
    /**
     * Reloads the page in non-editable mode, with the right scrolling.
     *
     * @private
     * @returns {Deferred} (never resolved, the page is reloading anyway)
     */
    _reload: function () {
        $('body').addClass('o_wait_reload');
        this.wysiwyg.destroy();
        this.$el.hide();
        window.location.hash = 'scrollTop=' + window.document.body.scrollTop;
        window.location.reload(true);
        return new Promise(function () {});
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the "Discard" button is clicked -> discards the changes.
     *
     * @private
     */
    _onCancelClick: function () {
        this.cancel(true);
    },
    /**
     * Get the cleaned value of the editable element.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onGetCleanHTML: function (ev) {
        ev.data.callback(this.wysiwyg.getValue({$layout: ev.data.$layout}));
    },
    /**
     * Snippet (menu_data) can request to save the document to leave the page
     *
     * @private
     * @param {OdooEvent} ev
     * @param {object} ev.data
     * @param {function} ev.data.onSuccess
     * @param {function} ev.data.onFailure
     */
    _onSnippetRequestSave: function (ev) {
        this.save(false).then(ev.data.onSuccess, ev.data.onFailure);
    },
    /**
     * Called when the "Save" button is clicked -> saves the changes.
     *
     * @private
     */
    _onSaveClick: function () {
        this.save();
    },
});

return EditorMenu;
});

```

## File: static\src\js\editor\editor_menu_translate.js

```javascript
odoo.define('website.editor.menu.translate', function (require) {
'use strict';

require('web.dom_ready');
var core = require('web.core');
var Dialog = require('web.Dialog');
var localStorage = require('web.local_storage');
var Wysiwyg = require('web_editor.wysiwyg.root');
var EditorMenu = require('website.editor.menu');

var _t = core._t;

var localStorageNoDialogKey = 'website_translator_nodialog';

var TranslatorInfoDialog = Dialog.extend({
    template: 'website.TranslatorInfoDialog',
    xmlDependencies: Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/translator.xml']
    ),

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super(parent, _.extend({
            title: _t("Translation Info"),
            buttons: [
                {text: _t("Ok, never show me this again"), classes: 'btn-primary', close: true, click: this._onStrongOk.bind(this)},
                {text: _t("Ok"), close: true}
            ],
        }, options || {}));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the "strong" ok is clicked -> adapt localstorage to make sure
     * the dialog is never displayed again.
     *
     * @private
     */
    _onStrongOk: function () {
        localStorage.setItem(localStorageNoDialogKey, true);
    },
});

var WysiwygTranslate = Wysiwyg.extend({
    assetLibs: Wysiwyg.prototype.assetLibs.concat(['website.compiled_assets_wysiwyg']),
    _getWysiwygContructor: function () {
        return odoo.__DEBUG__.services['web_editor.wysiwyg.multizone.translate'];
    }
});

var TranslatorMenu = EditorMenu.extend({

    /**
     * @override
     */
    start: function () {
        if (!localStorage.getItem(localStorageNoDialogKey)) {
            new TranslatorInfoDialog(this).open();
        }

        return this._super();
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Returns the editable areas on the page.
     *
     * @param {DOM} $wrapwrap
     * @returns {jQuery}
     */
    editable: function ($wrapwrap) {
    	var selector = '[data-oe-translation-id], '+
        	'[data-oe-model][data-oe-id][data-oe-field], ' +
        	'[placeholder*="data-oe-translation-id="], ' +
        	'[title*="data-oe-translation-id="], ' +
        	'[alt*="data-oe-translation-id="]';
        var $edit = $wrapwrap.find(selector);
        $edit.filter(':has(' + selector + ')').attr('data-oe-readonly', true);
        return $edit.not('[data-oe-readonly]');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _wysiwygInstance: function () {
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        return new WysiwygTranslate(this, {lang: context.lang});
    },
});

return TranslatorMenu;
});

```

## File: static\src\js\editor\rte.summernote.js

```javascript
odoo.define('website.rte.summernote', function (require) {
'use strict';

var core = require('web.core');
require('web_editor.rte.summernote');

var eventHandler = $.summernote.eventHandler;
var renderer = $.summernote.renderer;
var tplIconButton = renderer.getTemplate().iconButton;
var _t = core._t;

var fn_tplPopovers = renderer.tplPopovers;
renderer.tplPopovers = function (lang, options) {
    var $popover = $(fn_tplPopovers.call(this, lang, options));
    $popover.find('.note-image-popover .btn-group:has([data-value="img-thumbnail"])').append(
        tplIconButton('fa fa-object-ungroup', {
            title: _t('Transform the picture (click twice to reset transformation)'),
            event: 'transform',
        }));
    return $popover;
};

$.summernote.pluginEvents.transform = function (event, editor, layoutInfo, sorted) {
    var $selection = layoutInfo.handle().find('.note-control-selection');
    var $image = $($selection.data('target'));

    if ($image.data('transfo-destroy')) {
        $image.removeData('transfo-destroy');
        return;
    }

    $image.transfo();

    var mouseup = function (event) {
        $('.note-popover button[data-event="transform"]').toggleClass('active', $image.is('[style*="transform"]'));
    };
    $(document).on('mouseup', mouseup);

    var mousedown = function (event) {
        if (!$(event.target).closest('.transfo-container').length) {
            $image.transfo('destroy');
            $(document).off('mousedown', mousedown).off('mouseup', mouseup);
        }
        if ($(event.target).closest('.note-popover').length) {
            $image.data('transfo-destroy', true).attr('style', ($image.attr('style') || '').replace(/[^;]*transform[\w:]*;?/g, ''));
        }
        $image.trigger('content_changed');
    };
    $(document).on('mousedown', mousedown);
};

var fn_boutton_update = eventHandler.modules.popover.button.update;
eventHandler.modules.popover.button.update = function ($container, oStyle) {
    fn_boutton_update.call(this, $container, oStyle);
    $container.find('button[data-event="transform"]')
        .toggleClass('active', $(oStyle.image).is('[style*="transform"]'))
        .toggleClass('d-none', !$(oStyle.image).is('img'));
};
});

```

## File: static\src\js\editor\snippets.options.js

```javascript
odoo.define('website.editor.snippets.options', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var weWidgets = require('wysiwyg.widgets');
var options = require('web_editor.snippets.options');

var _t = core._t;
var qweb = core.qweb;

options.Class.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Refreshes all public widgets related to the given element.
     *
     * @private
     * @param {jQuery} [$el=this.$target]
     */
    _refreshPublicWidgets: function ($el) {
        this.trigger_up('widgets_start_request', {
            editableMode: true,
            $target: $el || this.$target,
        });
    },
});

options.registry.background.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getEditableMedia: function () {
        if (!this._hasBgvideo()) {
            return this._super(...arguments);
        }
        return this.$('.o_bg_video_iframe')[0];
    },
    /**
     * @override
     */
    _getMediaDialogOptions: function () {
        return _.extend(this._super(...arguments), {
            // For now, disable the possibility to have a parallax video bg
            noVideos: this.$target.is('.parallax, .s_parallax_bg'),
            isForBgVideo: true,
        });
    },
    /**
     * @override
     */
    _setActive: function () {
        this._super(...arguments);
        if (this._hasBgvideo()) {
            this.$el.find('[data-choose-image]').addClass('active');
        }
    },
    /**
     * Updates the background video used by the snippet.
     *
     * @private
     * @see this.selectClass for parameters
     */
    _setBgVideo: function (previewMode, value) {
        this.$('> .o_bg_video_container').toggleClass('d-none', previewMode === true);

        if (previewMode !== false) {
            return;
        }

        var target = this.$target[0];
        target.classList.toggle('o_background_video', !!(value && value.length));
        if (value && value.length) {
            target.dataset.bgVideoSrc = value;
        } else {
            delete target.dataset.bgVideoSrc;
        }
        this._refreshPublicWidgets();
        this._setActive();
    },
    /**
     * Returns whether the current target has a background video or not.
     *
     * @private
     * @returns {boolean}
     */
    _hasBgvideo: function () {
        return this.$target[0].classList.contains('o_background_video');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
     _onBackgroundColorUpdate: function (ev, previewMode) {
        var ret = this._super(...arguments);
        if (ret) {
            this._setBgVideo(previewMode);
        }
        return ret;
    },
    /**
     * @override
     */
    _onSaveMediaDialog: function (data) {
        if (!data.bgVideoSrc) {
            this._setBgVideo(false);
            this._super(...arguments);
            return;
        }
        // if the user chose a video, only add the video without removing the
        // background
        this._setBgVideo(false, data.bgVideoSrc);
    },
});

options.registry.menu_data = options.Class.extend({
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],

    /**
     * When the users selects a menu, a dialog is opened to ask him if he wants
     * to follow the link (and leave editor), edit the menu or do nothing.
     *
     * @override
     */
    onFocus: function () {
        var self = this;
        (new Dialog(this, {
            title: _t("Confirmation"),
            $content: $(core.qweb.render('website.leaving_current_page_edition')),
            buttons: [
                {text: _t("Go to Link"), classes: 'btn-primary', click: function () {
                    self.trigger_up('request_save', {
                        reload: false,
                        onSuccess: function () {
                            window.location.href = self.$target.attr('href');
                        },
                    });
                }},
                {text: _t("Edit the menu"), classes: 'btn-primary', click: function () {
                    this.trigger_up('action_demand', {
                        actionName: 'edit_menu',
                        params: [
                            function () {
                                var prom = new Promise(function (resolve, reject) {
                                    self.trigger_up('request_save', {
                                        onSuccess: resolve,
                                        onFailure: reject,
                                    });
                                });
                                return prom;
                            },
                        ],
                    });
                }},
                {text: _t("Stay on this page"), close: true}
            ]
        })).open();
    },
});

options.registry.company_data = options.Class.extend({
    /**
     * Fetches data to determine the URL where the user can edit its company
     * data. Saves the info in the prototype to do this only once.
     *
     * @override
     */
    start: function () {
        var proto = options.registry.company_data.prototype;
        var prom;
        var self = this;
        if (proto.__link === undefined) {
            prom = this._rpc({route: '/web/session/get_session_info'}).then(function (session) {
                return self._rpc({
                    model: 'res.users',
                    method: 'read',
                    args: [session.uid, ['company_id']],
                });
            }).then(function (res) {
                proto.__link = '/web#action=base.action_res_company_form&view_type=form&id=' + (res && res[0] && res[0].company_id[0] || 1);
            });
        }
        return Promise.all([this._super.apply(this, arguments), prom]);
    },
    /**
     * When the users selects company data, opens a dialog to ask him if he
     * wants to be redirected to the company form view to edit it.
     *
     * @override
     */
    onFocus: function () {
        var self = this;
        var proto = options.registry.company_data.prototype;

        Dialog.confirm(this, _t("Do you want to edit the company data ?"), {
            confirm_callback: function () {
                self.trigger_up('request_save', {
                    reload: false,
                    onSuccess: function () {
                        window.location.href = proto.__link;
                    },
                });
            },
        });
    },
});

/**
 * @todo should be refactored / reviewed
 */
options.registry.carousel = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;

        this.$target.carousel({interval: false});
        this.id = this.$target.attr('id');
        this.$inner = this.$target.find('.carousel-inner');
        this.$indicators = this.$target.find('.carousel-indicators');
        this.$target.carousel('pause');
        this._rebindEvents();

        var def = this._super.apply(this, arguments);

        // set background and prepare to clean for save
        this.$target.on('slid.bs.carousel', function () {
            self.$target.carousel('pause');
            self.trigger_up('option_update', {
                optionNames: ['background', 'background_position', 'colorpicker', 'sizing_y'],
                name: 'target',
                data: self.$target.find('.carousel-item.active'),
            });
        });

        return def;
    },
    /**
     * Associates unique ID on slider elements.
     *
     * @override
     */
    onBuilt: function () {
        this.id = 'myCarousel' + new Date().getTime();
        this.$target.attr('id', this.id);
        this.$target.find('[data-target]').attr('data-target', '#' + this.id);
        this._rebindEvents();
    },
    /**
     * @override
     */
    onFocus: function () {
        // Needs to be done on focus, not on start, as all other options are
        // maybe not all initialized in start
        this.$target.trigger('slid.bs.carousel');
    },
    /**
     * Associates unique ID on cloned slider elements.
     *
     * @override
     */
    onClone: function () {
        var id = 'myCarousel' + new Date().getTime();
        this.$target.attr('id', id);
        _.each(this.$target.find('[data-slide], [data-slide-to]'), function (el) {
            var $el = $(el);
            if ($el.attr('data-target')) {
                $el.attr('data-target', '#' + id);
            } else if ($el.attr('href')) {
                $el.attr('href', '#' + id);
            }
        });
    },
    /**
     * @override
     */
    cleanForSave: function () {
        this._super.apply(this, arguments);
        this.$target.find('.carousel-item').removeClass('next prev left right active')
            .first().addClass('active');
        this.$target.find('.carousel-indicators').find('li').removeClass('active').html('')
            .first().addClass('active');
        this.$target.removeClass('oe_img_bg ' + this._class).css('background-image', '');
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Adds a slide.
     *
     * @see this.selectClass for parameters
     */
    addSlide: function (previewMode) {
        var self = this;
        var cycle = this.$inner.find('.carousel-item').length;
        var $active = this.$inner.find('.carousel-item.active, .carousel-item.prev, .carousel-item.next').first();
        var index = $active.index();
        this.$('.carousel-control-prev, .carousel-control-next, .carousel-indicators').removeClass('d-none');
        // we added a space after the <li> in the line below to keep the same space between the indicators
        this.$indicators.append('<li data-target="#' + this.id + '" data-slide-to="' + cycle + '"></li> ');
        // Need to remove editor data from the clone so it gets its own.
        $active.clone(false)
            .removeClass('active')
            .insertAfter($active);
        _.defer(function () {
            self.$target.carousel().carousel(++index);
            self._rebindEvents();
        });
    },
    /**
     * Removes the current slide.
     *
     * @see this.selectClass for parameters.
     */
    removeSlide: function (previewMode) {
        if (this.remove_process) {
            return;
        }

        var self = this;

        var $items = this.$inner.find('.carousel-item');
        var cycle = $items.length - 1;
        var $active = $items.filter('.active');
        var index = $active.index();

        if (cycle > 0) {
            this.remove_process = true;
            this.$target.on('slid.bs.carousel.slide_removal', function (event) {
                $active.remove();
                self.$indicators.find('li:last').remove();
                self.$target.off('slid.bs.carousel.slide_removal');
                self._rebindEvents();
                self.remove_process = false;
                if (cycle === 1) {
                    self.$target.find('.carousel-control-prev, .carousel-control-next, .carousel-indicators').addClass('d-none');
                }
            });
            _.defer(function () {
                self._refreshPublicWidgets();
                self.$target.carousel(index > 0 ? --index : cycle);
            });
        }
    },
    /**
     * Changes the interval for autoplay.
     *
     * @see this.selectClass for parameters
     */
    interval: function (previewMode, value) {
        this.$target.attr('data-interval', value);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);
        this.$el.find('[data-interval]').removeClass('active')
            .filter('[data-interval=' + this.$target.attr('data-interval') + ']').addClass('active');
    },
    /**
     * Rebinds carousel events on indicators.
     *
     * @private
     */
    _rebindEvents: function () {
        var self = this;
        this.$target.find('.carousel-control-prev, .carousel-control-next').off('click').on('click', function () {
            self.$target.carousel($(this).data('slide'));
        });
        this.$target.find('.carousel-indicators [data-slide-to]').off('click').on('click', function () {
            self.$target.carousel(+$(this).data('slide-to'));
        });

        /* Fix: backward compatibility saas-3 */
        this.$target.find('.item.text_image, .item.image_text, .item.text_only').find('.container > .carousel-caption > div, .container > img.carousel-image').attr('contentEditable', 'true');
    },
});

options.registry.navTabs = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        this._findLinksAndPanes();
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onBuilt: function () {
        this._generateUniqueIDs();
    },
    /**
     * @override
     */
    onClone: function () {
        this._generateUniqueIDs();
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Creates a new tab and tab-pane.
     *
     * @see this.selectClass for parameters
     */
    addTab: function (previewMode, value, $opt) {
        var $activeItem = this.$navLinks.filter('.active').parent();
        var $activePane = this.$tabPanes.filter('.active');

        var $navItem = $activeItem.clone();
        var $navLink = $navItem.find('.nav-link').removeClass('active show');
        var $tabPane = $activePane.clone().removeClass('active show');
        $navItem.insertAfter($activeItem);
        $tabPane.insertAfter($activePane);
        this._findLinksAndPanes();
        this._generateUniqueIDs();

        $navLink.tab('show');
    },
    /**
     * Removes the current active tab and its content.
     *
     * @see this.selectClass for parameters
     */
    removeTab: function (previewMode, value, $opt) {
        var self = this;

        var $activeLink = this.$navLinks.filter('.active');
        var $activePane = this.$tabPanes.filter('.active');

        var $next = this.$navLinks.eq((this.$navLinks.index($activeLink) + 1) % this.$navLinks.length);
        $next.one('shown.bs.tab', function () {
            $activeLink.parent().remove();
            $activePane.remove();
            self._findLinksAndPanes();
            self._setActive(); // TODO forced to do this because we do not return deferred for options
        });
        $next.tab('show');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _findLinksAndPanes: function () {
        this.$navLinks = this.$target.find('.nav-link');
        var $el = this.$target;
        do {
            $el = $el.parent();
            this.$tabPanes = $el.find('.tab-pane');
        } while (this.$tabPanes.length === 0 && !$el.is('body'));
    },
    /**
     * @private
     */
    _generateUniqueIDs: function () {
        for (var i = 0 ; i < this.$navLinks.length ; i++) {
            var id = _.now() + '_' + _.uniqueId();
            var idLink = 'nav_tabs_link_' + id;
            var idContent = 'nav_tabs_content_' + id;
            this.$navLinks.eq(i).attr({
                'id': idLink,
                'href': '#' + idContent,
                'aria-controls': idContent,
            });
            this.$tabPanes.eq(i).attr({
                'id': idContent,
                'aria-labelledby': idLink,
            });
        }
    },
    /**
     * @private
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);
        this.$el.filter('[data-remove-tab]').toggleClass('d-none', this.$tabPanes.length <= 2);
    },
});

options.registry.sizing_x = options.registry.sizing.extend({
    /**
     * @override
     */
    onClone: function (options) {
        this._super.apply(this, arguments);
        // Below condition is added to remove offset of target element only
        // and not its children to avoid design alteration of a container/block.
        if (options.isCurrent) {
            var _class = this.$target.attr('class').replace(/\s*(offset-xl-|offset-lg-)([0-9-]+)/g, '');
            this.$target.attr('class', _class);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getSize: function () {
        var width = this.$target.closest('.row').width();
        var gridE = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12];
        var gridW = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
        this.grid = {
            e: [_.map(gridE, function (v) { return 'col-lg-' + v; }), _.map(gridE, function (v) { return width/12*v; }), 'width'],
            w: [_.map(gridW, function (v) { return 'offset-lg-' + v; }), _.map(gridW, function (v) { return width/12*v; }), 'margin-left'],
        };
        return this.grid;
    },
    /**
     * @override
     */
    _onResize: function (compass, beginClass, current) {
        if (compass === 'w') {
            // don't change the right border position when we change the offset (replace col size)
            var beginCol = Number(beginClass.match(/col-lg-([0-9]+)|$/)[1] || 0);
            var beginOffset = Number(beginClass.match(/offset-lg-([0-9-]+)|$/)[1] || beginClass.match(/offset-xl-([0-9-]+)|$/)[1] || 0);
            var offset = Number(this.grid.w[0][current].match(/offset-lg-([0-9-]+)|$/)[1] || 0);
            if (offset < 0) {
                offset = 0;
            }
            var colSize = beginCol - (offset - beginOffset);
            if (colSize <= 0) {
                colSize = 1;
                offset = beginOffset + beginCol - 1;
            }
            this.$target.attr('class',this.$target.attr('class').replace(/\s*(offset-xl-|offset-lg-|col-lg-)([0-9-]+)/g, ''));

            this.$target.addClass('col-lg-' + (colSize > 12 ? 12 : colSize));
            if (offset > 0) {
                this.$target.addClass('offset-lg-' + offset);
            }
        }
        this._super.apply(this, arguments);
    },
});

options.registry.layout_column = options.Class.extend({

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Changes the number of columns.
     *
     * @see this.selectClass for parameters
     */
    selectCount: function (previewMode, value, $opt) {
        this._updateColumnCount(value - this.$target.children().length);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adds new columns which are clones of the last column or removes the
     * last x columns.
     *
     * @private
     * @param {integer} count - positif to add, negative to remove
     */
    _updateColumnCount: function (count) {
        if (!count) {
            return;
        }

        this.trigger_up('request_history_undo_record', {$target: this.$target});

        var colsLength = this.$target.children().length + count;
        if (count > 0) {
            var $lastColumn = this.$target.children().last();
            for (var i = 0; i < count; i++) {
                this.trigger_up('clone_snippet', {$snippet: $lastColumn});
            }
        } else {
            var self = this;
            _.each(this.$target.children().slice(count), function (el) {
                self.trigger_up('remove_snippet', {$snippet: $(el)});
            });
        }

        this._resizeColumns(colsLength);
        this.trigger_up('cover_update');
    },
    /**
     * Resizes the columns so that they are kept on one row.
     *
     * @private
     * @param {number} [colsLength] (default to the actual number of columns)
     */
    _resizeColumns: function (colsLength) {
        var $columns = this.$target.children();
        colsLength = colsLength || $columns.length;
        var colSize = Math.floor(12 / colsLength) || 1;
        var colOffset = Math.floor((12 - colSize * colsLength) / 2);
        var colClass = 'col-lg-' + colSize;
        _.each($columns, function (column) {
            var $column = $(column);
            $column.attr('class', $column.attr('class').replace(/\b(col|offset)-lg(-\d+)?\b/g, ''));
            $column.addClass(colClass);
        });
        if (colOffset) {
            $columns.first().addClass('offset-lg-' + colOffset);
        }
        // TODO: remove in master. This is used to keep the UI in sync, but
        // won't be needed once option methods are properly asynchronous.
        this.colsLength = colsLength;
    },
    /**
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);
        this.$el.find('[data-select-count]').removeClass('active')
            .filter('[data-select-count=' + (this.colsLength || this.$target.children().length) + ']').addClass('active');
    },
});

options.registry.parallax = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$target.on('snippet-option-change snippet-option-preview', function () {
            self._refreshPublicWidgets();
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onFocus: function () {
        this.trigger_up('option_update', {
            optionNames: ['background', 'background_position'],
            name: 'target',
            data: this.$target.find('> .s_parallax_bg'),
        });
        // Refresh the parallax animation on focus; at least useful because
        // there may have been changes in the page that influenced the parallax
        // rendering (new snippets, ...).
        // TODO make this automatic.
        this._refreshPublicWidgets();
    },
    /**
     * @override
     */
    onMove: function () {
        this._refreshPublicWidgets();
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Changes the scrolling speed of the parallax effect.
     *
     * @see this.selectClass for parameters
     */
    scroll: function (previewMode, value) {
        this.$target.attr('data-scroll-background-ratio', value);
        this._refreshPublicWidgets();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);
        this.$el.find('[data-scroll]').removeClass('active')
            .filter('[data-scroll="' + (this.$target.attr('data-scroll-background-ratio') || 0) + '"]').addClass('active');
    },
});

var FacebookPageDialog = weWidgets.Dialog.extend({
    xmlDependencies: weWidgets.Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.facebook_page.xml']
    ),
    template: 'website.facebook_page_dialog',
    events: _.extend({}, weWidgets.Dialog.prototype.events || {}, {
        'change': '_onOptionChange',
    }),

    /**
     * @constructor
     */
    init: function (parent, fbData, options) {
        this._super(parent, _.extend({
            title: _t("Facebook Page"),
        }, options || {}));

        this.fbData = $.extend(true, {}, fbData);
        this.final_data = this.fbData;
    },
    /**
     * @override
     */
    start: function () {
        this.$previewPage = this.$('.o_facebook_page');
        this.opened().then(this._renderPreview.bind(this));
        return this._super.apply(this, arguments);
    },

    //------------------------------------------------------------------
    // Private
    //------------------------------------------------------------------

    /**
     * Manages Facebook page preview. Also verifies if the page exists on
     * Facebook or not.
     *
     * @private
     */
    _renderPreview: function () {
        var self = this;
        var match = this.fbData.href.match(/^(?:https?:\/\/)?(?:www\.)?(?:fb|facebook)\.com\/(?:([\w.]+)|[^/?#]+-([0-9]{15,16}))(?:$|[\/?# ])/);
        if (match) {
            // Check if the page exists on Facebook or not
            $.ajax({
                url: 'https://graph.facebook.com/' + (match[2] || match[1]) + '/picture',
                statusCode: {
                    200: function () {
                        self._toggleWarning(true);

                        // Managing height based on options
                        if (self.fbData.tabs) {
                            self.fbData.height = self.fbData.tabs === 'events' ? 300 : 500;
                        } else if (self.fbData.small_header) {
                            self.fbData.height = self.fbData.show_facepile ? 165 : 70;
                        } else if (!self.fbData.small_header) {
                            self.fbData.height = self.fbData.show_facepile ? 225 : 150;
                        }
                        options.registry.facebookPage.prototype.markFbElement(self.getParent(), self.$previewPage, self.fbData);
                    },
                    404: function () {
                        self._toggleWarning(false);
                    },
                },
            });
        } else {
            this._toggleWarning(false);
        }
    },
    /**
     * Toggles the warning message and save button and destroy iframe preview.
     *
     * @private
     * @param {boolean} toggle
     */
    _toggleWarning: function (toggle) {
        this.trigger_up('widgets_stop_request', {
            $target: this.$previewPage,
        });
        this.$('.facebook_page_warning').toggleClass('d-none', toggle);
        this.$footer.find('.btn-primary').prop('disabled', !toggle);
    },

    //------------------------------------------------------------------
    // Handlers
    //------------------------------------------------------------------

    /**
     * Called when a facebook option is changed -> adapt the preview and saved
     * data.
     *
     * @private
     */
    _onOptionChange: function () {
        var self = this;
        // Update values in fbData
        this.fbData.tabs = _.map(this.$('.o_facebook_tabs input:checked'), function (tab) { return tab.name; }).join(',');
        this.fbData.href = this.$('.o_facebook_page_url').val();
        _.each(this.$('.o_facebook_options input'), function (el) {
            self.fbData[el.name] = $(el).prop('checked');
        });
        this._renderPreview();
    },
});
options.registry.facebookPage = options.Class.extend({
    /**
     * Initializes the required facebook page data to create the iframe.
     *
     * @override
     */
    willStart: function () {
        var defs = [this._super.apply(this, arguments)];

        var defaults = {
            href: false,
            height: 215,
            width: 350,
            tabs: '',
            small_header: false,
            hide_cover: false,
            show_facepile: false,
        };
        this.fbData = _.defaults(_.pick(this.$target.data(), _.keys(defaults)), defaults);

        if (!this.fbData.href) {
            // Fetches the default url for facebook page from website config
            var self = this;
            defs.push(this._rpc({
                model: 'website',
                method: 'search_read',
                args: [[], ['social_facebook']],
                limit: 1,
            }).then(function (res) {
                if (res) {
                    self.fbData.href = res[0].social_facebook || 'https://www.facebook.com/Odoo';
                }
            }));
        }

        return Promise.all(defs);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$target.on('click.facebook_page_option', '.o_add_facebook_page', function (ev) {
            ev.preventDefault();
            self.fbPageOptions();
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this._super.apply(this, arguments);
        this.$target.off('.facebook_page_option');
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Opens a dialog to configure the facebook page options.
     *
     * @see this.selectClass for parameters
     */
    fbPageOptions: function () {
        var dialog = new FacebookPageDialog(this, this.fbData).open();
        dialog.on('save', this, function (fbData) {
            this.$target.empty();
            this.fbData = fbData;
            options.registry.facebookPage.prototype.markFbElement(this, this.$target, this.fbData);
        });
    },

    //--------------------------------------------------------------------------
    // Static
    //--------------------------------------------------------------------------

    /**
     * @static
     */
    markFbElement: function (self, $el, fbData) {
        _.each(fbData, function (value, key) {
            $el.attr('data-' + key, value);
            $el.data(key, value);
        });
        self._refreshPublicWidgets($el);
    },
});

options.registry.ul = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$target.on('mouseup', '.o_ul_toggle_self, .o_ul_toggle_next', function () {
            self.trigger_up('cover_update');
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    cleanForSave: function () {
        this._super();
        if (!this.$target.hasClass('o_ul_folded')) {
            this.$target.find('.o_close').removeClass('o_close');
            this.$target.find('li').css('list-style', '');
        }
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    toggleClass: function () {
        this._super.apply(this, arguments);

        this.trigger_up('widgets_stop_request', {
            $target: this.$target,
        });

        this.$target.find('.o_ul_toggle_self, .o_ul_toggle_next').remove();
        this.$target.find('li:has(>ul,>ol)').map(function () {
            // get if the li contain a text label
            var texts = _.filter(_.toArray(this.childNodes), function (a) { return a.nodeType === 3;});
            if (!texts.length || !texts.reduce(function (a,b) { return a.textContent + b.textContent;}).match(/\S/)) {
                return;
            }
            $(this).children('ul,ol').addClass('o_close');
            return $(this).children(':not(ul,ol)')[0] || this;
        })
        .prepend('<a href="#" class="o_ul_toggle_self fa" />');
        var $li = this.$target.find('li:has(+li:not(>.o_ul_toggle_self)>ul, +li:not(>.o_ul_toggle_self)>ol)');
        $li.css('list-style', this.$target.hasClass('o_ul_folded') ? 'none' : '');
        $li.map(function () { return $(this).children()[0] || this; })
            .prepend('<a href="#" class="o_ul_toggle_next fa" />');
        $li.removeClass('o_open').next().addClass('o_close');
        this.$target.find('li').removeClass('o_open');
        this._refreshPublicWidgets();
    },
});

options.registry.collapse = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$target.on('shown.bs.collapse hidden.bs.collapse', '[role="tabpanel"]', function () {
            self.trigger_up('cover_update');
            self.$target.trigger('content_changed');
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onBuilt: function () {
        this._createIDs();
    },
    /**
     * @override
     */
    onClone: function () {
        this._createIDs();
    },
    /**
     * @override
     */
    onMove: function () {
        this._createIDs();
        var $panel = this.$target.find('.collapse').removeData('bs.collapse');
        if ($panel.attr('aria-expanded') === 'true') {
            $panel.closest('.accordion').find('.collapse[aria-expanded="true"]')
                .filter(function () {return this !== $panel[0];})
                .collapse('hide')
                .one('hidden.bs.collapse', function () {
                    $panel.trigger('shown.bs.collapse');
                });
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Associates unique ids on collapse elements.
     *
     * @private
     */
    _createIDs: function () {
        let time = new Date().getTime();
        const $tablist = this.$target.closest('[role="tablist"]');
        const $tab = this.$target.find('[role="tab"]');
        const $panel = this.$target.find('[role="tabpanel"]');

        const setUniqueId = ($elem, label) => {
            let elemId = $elem.attr('id');
            if (!elemId || $('[id="' + elemId + '"]').length > 1) {
                do {
                    time++;
                    elemId = label + time;
                } while ($('#' + elemId).length);
                $elem.attr('id', elemId);
            }
            return elemId;
        };

        const tablistId = setUniqueId($tablist, 'myCollapse');
        $panel.attr('data-parent', '#' + tablistId);
        $panel.data('parent', '#' + tablistId);

        const panelId = setUniqueId($panel, 'myCollapseTab');
        $tab.attr('data-target', '#' + panelId);
        $tab.data('target', '#' + panelId);
    },
});

options.registry.gallery = options.Class.extend({
    xmlDependencies: ['/website/static/src/xml/website.gallery.xml'],

    /**
     * @override
     */
    start: function () {
        var self = this;

        // The snippet should not be editable
        this.$target.addClass('o_fake_not_editable').attr('contentEditable', false);

        // Make sure image previews are updated if images are changed
        this.$target.on('save.gallery', 'img', function (ev) {
            var $img = $(ev.currentTarget);
            var index = self.$target.find('.carousel-item.active').index();
            self.$('.carousel:first li[data-target]:eq(' + index + ')')
                .css('background-image', 'url(' + $img.attr('src') + ')');
        });

        // When the snippet is empty, an edition button is the default content
        // TODO find a nicer way to do that to have editor style
        this.$target.on('click.gallery', '.o_add_images', function (e) {
            e.stopImmediatePropagation();
            self.addImages(false);
        });

        this.$target.on('dropped.gallery', 'img', function (ev) {
            self.mode(null, self.getMode());
            if (!ev.target.height) {
                $(ev.target).one('load', function () {
                    setTimeout(function () {
                        self.trigger_up('cover_update');
                    });
                });
            }
        });

        if (this.$('.container:first > *:not(div)').length) {
            self.mode(null, self.getMode());
        }

        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onBuilt: function () {
        this._adaptNavigationIDs();
    },
    /**
     * @override
     */
    onClone: function () {
        this._adaptNavigationIDs();
    },
    /**
     * @override
     */
    cleanForSave: function () {
        if (this.$target.hasClass('slideshow')) {
            this.$target.removeAttr('style');
        }
    },
    /**
     * @override
     */
    destroy() {
        this._super(...arguments);
        this.$target.off('.gallery');
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Allows to select images to add as part of the snippet.
     *
     * @see this.selectClass for parameters
     */
    addImages: function (previewMode) {
        var self = this;
        var $container = this.$('.container:first');
        var dialog = new weWidgets.MediaDialog(this, {multiImages: true, onlyImages: true, mediaWidth: 1920});
        var lastImage = _.last(this._getImages());
        var index = lastImage ? this._getIndex(lastImage) : -1;
        dialog.on('save', this, function (attachments) {
            for (var i = 0 ; i < attachments.length; i++) {
                $('<img/>', {
                    class: 'img img-fluid',
                    src: attachments[i].image_src,
                    'data-index': ++index,
                }).appendTo($container);
            }
            self._reset();
            self.trigger_up('cover_update');
            this._setActive();
        });
        dialog.open();
    },
    /**
     * Allows to change the number of columns when displaying images with a
     * grid-like layout.
     *
     * @see this.selectClass for parameters
     */
    columns: function (previewMode, value) {
        this.$target.attr('data-columns', value);

        var $activeMode = this.$el.find('.active[data-mode]');
        this.mode(null, $activeMode.data('mode'), $activeMode);
    },
    /**
     * Get the image target's layout mode (slideshow, masonry, grid or nomode).
     *
     * @returns {String('slideshow'|'masonry'|'grid'|'nomode')}
     */
    getMode: function () {
        var mode = 'slideshow';
        if (this.$target.hasClass('o_masonry')) {
            mode = 'masonry';
        }
        if (this.$target.hasClass('o_grid')) {
            mode = 'grid';
        }
        if (this.$target.hasClass('o_nomode')) {
            mode = 'nomode';
        }
        return mode;
    },
    /**
     * Displays the images with the "grid" layout.
     */
    grid: function () {
        var imgs = this._getImages();
        var $row = $('<div/>', {class: 'row'});
        var columns = this._getColumns();
        var colClass = 'col-lg-' + (12 / columns);
        var $container = this._replaceContent($row);

        _.each(imgs, function (img, index) {
            var $img = $(img);
            var $col = $('<div/>', {class: colClass});
            $col.append($img).appendTo($row);
            if ((index + 1) % columns === 0) {
                $row = $('<div/>', {class: 'row'});
                $row.appendTo($container);
            }
        });
        this.$target.css('height', '');
    },
    /**
     * Allows to changes the interval of automatic slideshow (not active in
     * edit mode).
     */
    interval: function (previewMode, value) {
        this.$target.find('.carousel:first').attr('data-interval', value);
    },
    /**
     * Displays the images with the "masonry" layout.
     */
    masonry: function () {
        var self = this;
        var imgs = this._getImages();
        var columns = this._getColumns();
        var colClass = 'col-lg-' + (12 / columns);
        var cols = [];

        var $row = $('<div/>', {class: 'row'});
        this._replaceContent($row);

        // Create columns
        for (var c = 0; c < columns; c++) {
            var $col = $('<div/>', {class: 'col o_snippet_not_selectable ' + colClass});
            $row.append($col);
            cols.push($col[0]);
        }

        // Dispatch images in columns by always putting the next one in the
        // smallest-height column
        while (imgs.length) {
            var min = Infinity;
            var $lowest;
            _.each(cols, function (col) {
                var $col = $(col);
                var height = $col.is(':empty') ? 0 : $col.find('img').last().offset().top + $col.find('img').last().height() - self.$target.offset().top;
                if (height < min) {
                    min = height;
                    $lowest = $col;
                }
            });
            $lowest.append(imgs.pop());
        }
    },
    /**
     * Allows to change the images layout. @see grid, masonry, nomode, slideshow
     *
     * @see this.selectClass for parameters
     */
    mode: function (previewMode, value, $opt) {
        this.$target.css('height', '');
        this.$target
            .removeClass('o_nomode o_masonry o_grid o_slideshow')
            .addClass('o_' + value);
        this[value]();
        this.trigger_up('cover_update');
    },
    /**
     * Displays the images with the standard layout: floating images.
     */
    nomode: function () {
        var $row = $('<div/>', {class: 'row'});
        var imgs = this._getImages();

        this._replaceContent($row);

        _.each(imgs, function (img) {
            var wrapClass = 'col-lg-3';
            if (img.width >= img.height * 2 || img.width > 600) {
                wrapClass = 'col-lg-6';
            }
            var $wrap = $('<div/>', {class: wrapClass}).append(img);
            $row.append($wrap);
        });
    },
    /**
     * Allows to remove all images. Restores the snippet to the way it was when
     * it was added in the page.
     *
     * @see this.selectClass for parameters
     */
    removeAllImages: function (previewMode) {
        var $addImg = $('<div>', {
            class: 'alert alert-info css_non_editable_mode_hidden text-center',
        });
        var $text = $('<span>', {
            class: 'o_add_images',
            style: 'cursor: pointer;',
            text: _t(" Add Images"),
        });
        var $icon = $('<i>', {
            class: ' fa fa-plus-circle',
        });
        this._replaceContent($addImg.append($icon).append($text));
    },
    /**
     * Displays the images with a "slideshow" layout.
     */
    slideshow: function () {
        var imgStyle = this.$el.find('.active[data-styling]').data('styling') || '';
        var urls = _.map(this._getImages(), function (img) {
            return $(img).attr('src');
        });
        var currentInterval = this.$target.find('.carousel:first').attr('data-interval');
        var params = {
            srcs : urls,
            index: 0,
            title: "",
            interval : currentInterval || this.$target.data('interval') || 0,
            id: 'slideshow_' + new Date().getTime(),
            userStyle: imgStyle,
        },
        $slideshow = $(qweb.render('website.gallery.slideshow', params));
        this._replaceContent($slideshow);
        _.each(this.$('img'), function (img, index) {
            $(img).attr({contenteditable: true, 'data-index': index});
        });
        this.$target.css('height', Math.round(window.innerHeight * 0.7));

        // Apply layout animation
        this.$target.off('slide.bs.carousel').off('slid.bs.carousel');
        this.$('li.fa').off('click');
        this._refreshPublicWidgets();
    },
    /**
     * Allows to change the style of the individual images.
     *
     * @see this.selectClass for parameters
     */
    styling: function (previewMode, value) {
        var classes = _.map(this.$el.find('[data-styling]'), function (el) {
            return $(el).data('styling');
        }).join(' ');
        this.$('img').removeClass(classes).addClass(value);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Handles image removals and image index updates.
     *
     * @override
     */
    notify: function (name, data) {
        this._super.apply(this, arguments);
        if (name === 'image_removed') {
            data.$image.remove(); // Force the removal of the image before reset
            this._reset();
        } else if (name === 'image_index_request') {
            var imgs = this._getImages();
            var position = _.indexOf(imgs, data.$image[0]);
            imgs.splice(position, 1);
            switch (data.position) {
                case 'first':
                    imgs.unshift(data.$image[0]);
                    break;
                case 'prev':
                    imgs.splice(position - 1, 0, data.$image[0]);
                    break;
                case 'next':
                    imgs.splice(position + 1, 0, data.$image[0]);
                    break;
                case 'last':
                    imgs.push(data.$image[0]);
                    break;
            }
            _.each(imgs, function (img, index) {
                // Note: there might be more efficient ways to do that but it is
                // more simple this way and allows compatibility with 10.0 where
                // indexes were not the same as positions.
                $(img).attr('data-index', index);
            });
            this._reset();
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptNavigationIDs: function () {
        var uuid = new Date().getTime();
        this.$target.find('.carousel').attr('id', 'slideshow_' + uuid);
        _.each(this.$target.find('[data-slide], [data-slide-to]'), function (el) {
            var $el = $(el);
            if ($el.attr('data-target')) {
                $el.attr('data-target', '#slideshow_' + uuid);
            } else if ($el.attr('href')) {
                $el.attr('href', '#slideshow_' + uuid);
            }
        });
    },
    /**
     * Returns the images, sorted by index.
     *
     * @private
     * @returns {DOMElement[]}
     */
    _getImages: function () {
        var imgs = this.$('img').get();
        var self = this;
        imgs.sort(function (a, b) {
            return self._getIndex(a) - self._getIndex(b);
        });
        return imgs;
    },
    /**
     * Returns the index associated to a given image.
     *
     * @private
     * @param {DOMElement} img
     * @returns {integer}
     */
    _getIndex: function (img) {
        return img.dataset.index || 0;
    },
    /**
     * Returns the currently selected column option.
     *
     * @private
     * @returns {integer}
     */
    _getColumns: function () {
        return parseInt(this.$target.attr('data-columns')) || 3;
    },
    /**
     * Empties the container, adds the given content and returns the container.
     *
     * @private
     * @param {jQuery} $content
     * @returns {jQuery} the main container of the snippet
     */
    _replaceContent: function ($content) {
        var $container = this.$('.container:first');
        $container.empty().append($content);
        return $container;
    },
    /**
     * @override
     */
    _setActive: function () {
        this._super(...arguments);

        var activeModeSelectors = [];
        for (const className of this.$target[0].classList) {
            if (className.startsWith('o_')) {
                activeModeSelectors.push('[data-mode="' + className.substring(2) + '"]');
            }
        }
        var activeMode = this.$el.find('[data-mode]')
            .removeClass('active')
            .filter(activeModeSelectors.join(', '))
            .addClass('active')
            .data('mode');

        var carousel = this.$target[0].querySelector('.carousel');
        var activeInterval = carousel ? (carousel.dataset.interval || 0) : undefined;
        var $intervalOptions = this.$el.find('[data-interval]');
        $intervalOptions.removeClass('active')
            .filter('[data-interval="' + activeInterval + '"]')
            .addClass('active');
        $intervalOptions.closest('we-collapse-area')[0]
            .classList.toggle('d-none', activeMode !== 'slideshow');

        var columns = this._getColumns();
        var $columnOptions = this.$el.find('[data-columns]');
        $columnOptions.removeClass('active')
            .filter('[data-columns="' + columns + '"]')
            .addClass('active');
        $columnOptions.closest('we-collapse-area')[0]
            .classList.toggle('d-none', !(activeMode === 'grid' || activeMode === 'masonry'));

        this.el.querySelector('.o_w_image_spacing_option')
            .classList.toggle('d-none', activeMode === 'slideshow');

        var $stylingOptions = this.$el.find('[data-styling]');
        $stylingOptions.removeClass('active');
        var img = this.$target[0].querySelector('img');
        var activeStyleSelectors = [];
        if (img) {
            for (const className of img.classList) {
                activeStyleSelectors.push('[data-styling="' + className + '"]');
            }
        }
        var $toEnable = activeStyleSelectors.length
            ? $stylingOptions.filter(activeStyleSelectors.join(', '))
            : null;
        if (!$toEnable || !$toEnable.length) {
            $toEnable = $stylingOptions.first();
        }
        $toEnable.addClass('active');
    },
});

options.registry.gallery_img = options.Class.extend({
    /**
     * Rebuilds the whole gallery when one image is removed.
     *
     * @override
     */
    onRemove: function () {
        this.trigger_up('option_update', {
            optionName: 'gallery',
            name: 'image_removed',
            data: {
                $image: this.$target,
            },
        });
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Allows to change the position of an image (its order in the image set).
     *
     * @see this.selectClass for parameters
     */
    position: function (previewMode, value) {
        this.trigger_up('deactivate_snippet');
        this.trigger_up('option_update', {
            optionName: 'gallery',
            name: 'image_index_request',
            data: {
                $image: this.$target,
                position: value,
            },
        });
    },
});

options.registry.topMenuTransparency = options.Class.extend({

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Handles the toggling between normal and overlay positions of the header.
     *
     * @see this.selectClass for params
     */
    transparent: function (previewMode, value, $opt) {
        var self = this;
        this.trigger_up('action_demand', {
            actionName: 'toggle_page_option',
            params: [{name: 'header_overlay'}],
            onSuccess: function () {
                self.trigger_up('action_demand', {
                    actionName: 'toggle_page_option',
                    params: [{name: 'header_color', value: ''}],
                });
            },
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);

        this.trigger_up('action_demand', {
            actionName: 'get_page_option',
            params: ['header_overlay'],
            onSuccess: value => {
                this.$el.find('[data-transparent]').toggleClass('active', !!value);
            },
        });
    },
});

options.registry.topMenuColor = options.registry.colorpicker.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        var def = this._super.apply(this, arguments);
        this.$target.on('snippet-option-change', function () {
            self.onFocus();
        });
        return def;
    },
    /**
     * @override
     */
    onFocus: function () {
        this.trigger_up('action_demand', {
            actionName: 'get_page_option',
            params: ['header_overlay'],
            onSuccess: value => {
                this.$el.toggleClass('d-none', !value);
                if (!value) {
                    this.$el.find('button.selected').removeClass('selected');
                }
            },
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onColorButtonClick: function () {
        this._super.apply(this, arguments);
        var bgs = this.$target.attr('class').match(/bg-(\w|-)+/g);
        var allowedBgs = this.classes.split(' ');
        var color = _.intersection(bgs, allowedBgs).join(' ');
        this.trigger_up('action_demand', {
            actionName: 'toggle_page_option',
            params: [{name: 'header_color', value: color}],
        });
    },
    /**
     * @override
     */
    _onColorResetButtonClick: function () {
        this._super.apply(this, arguments);
        this.trigger_up('action_demand', {
            actionName: 'toggle_page_option',
            params: [{name: 'header_color', value: ''}],
        });
    },
});

/**
 * Handles the edition of snippet's anchor name.
 */
options.registry.anchorName = options.Class.extend({
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    onClone: function () {
        this.$target.removeAttr('data-anchor');
        this.$target.filter(':not(.carousel)').removeAttr('id');
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    openAnchorDialog: function (previewMode, value, $opt) {
        var self = this;
        var buttons = [{
            text: _t("Save"),
            classes: 'btn-primary',
            click: function () {
                var $input = this.$('.o_input_anchor_name');
                var anchorName = $input.val().trim().replace(/\s/g, '_');
                if (self.$target[0].id === anchorName) {
                    // If the chosen anchor name is already the one used by the
                    // element, close the dialog and do nothing else
                    this.close();
                    return;
                }

                var isValid = /^[\w-]+$/.test(anchorName);
                var alreadyExists = isValid && $('#' + anchorName).length > 0;
                var anchorOK = isValid && !alreadyExists;
                this.$('.o_anchor_not_valid').toggleClass('d-none', isValid);
                this.$('.o_anchor_already_exists').toggleClass('d-none', !alreadyExists);
                $input.toggleClass('is-invalid', !anchorOK);
                if (anchorOK) {
                    self._setAnchorName(anchorName);
                    this.close();
                }
            },
        }, {
            text: _t("Discard"),
            close: true,
        }];
        if (this.$target.attr('id')) {
            buttons.push({
                text: _t("Remove"),
                classes: 'btn-link ml-auto',
                icon: 'fa-trash',
                close: true,
                click: function () {
                    self._setAnchorName();
                },
            });
        }
        new Dialog(this, {
            title: _t("Link Anchor"),
            $content: $(qweb.render('website.dialog.anchorName', {
                currentAnchor: this.$target.attr('id'),
            })),
            buttons: buttons,
        }).open();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {String} value
     */
    _setAnchorName: function (value) {
        if (value) {
            this.$target.attr({
                'id': value,
                'data-anchor': true,
            });
        } else {
            this.$target.removeAttr('id data-anchor');
        }
        this.$target.trigger('content_changed');
    },
});

/**
 * Allows edition of 'cover_properties' in website models which have such
 * fields (blogs, posts, events, ...).
 */
options.registry.CoverProperties = options.Class.extend({
    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);

        this.$image = this.$target.find('.o_record_cover_image');
        this.$filter = this.$target.find('.o_record_cover_filter');
    },
    /**
     * @override
     */
    start: function () {
        this.$filterValueOpts = this.$el.find('[data-filter-value]');
        this.$filterColorOpts = this.$el.find('[data-filter-color]');
        this.filterColorClasses = this.$filterColorOpts.map(function () {
            return $(this).data('filterColor');
        }).get().join(' ');

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    clear: function (previewMode, value, $opt) {
        this.selectClass(previewMode, '', $());
        this.$image.css('background-image', '');
    },
    /**
     * @see this.selectClass for parameters
     */
    change: function (previewMode, value, $opt) {
        var $image = $('<img/>');
        var background = this.$image.css('background-image');
        if (background && background !== 'none') {
            $image.attr('src', background.match(/^url\(["']?(.+?)["']?\)$/)[1]);
        }

        var editor = new weWidgets.MediaDialog(this, {
            mediaWidth: 1920,
            onlyImages: true,
        }, $image[0]).open();
        editor.on('save', this, function (image) {
            var src = image.src;
            this.$image.css('background-image', src ? ('url(' + src + ')') : '');
            if (!this.$target.hasClass('o_record_has_cover')) {
                var $opt = this.$el.find('.o_record_cover_opt_size_default[data-select-class]');
                this.selectClass(previewMode, $opt.data('selectClass'), $opt);
            }
            this._setActive();
        });
    },
    /**
     * @see this.selectClass for parameters
     */
    filterValue: function (previewMode, value, $opt) {
        this.$filter.css('opacity', value);
    },
    /**
     * @see this.selectClass for parameters
     */
    filterColor: function (previewMode, value, $opt) {
        this.$filter.removeClass(this.filterColorClasses);
        if (value) {
            this.$filter.addClass(value);
        }

        var $firstVisibleFilterOpt = this.$filterValueOpts.eq(1);
        if (parseFloat(this.$filter.css('opacity')) < parseFloat($firstVisibleFilterOpt.data('filterValue'))) {
            this.filterValue(previewMode, $firstVisibleFilterOpt.data('filterValue'), $firstVisibleFilterOpt);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _setActive: function () {
        this._super.apply(this, arguments);

        _.each(this.$el.children(), el => {
            var $el = $(el);

            if (!$el.is('[data-change]')) {
                $el.removeClass('d-none');

                ['size', 'filters', 'text_size', 'text_align'].forEach(optName => {
                    var $opts = $el.find('[data-cover-opt="' + optName + '"]');
                    var notAllowed = (this.$target.data('use_' + optName) !== 'True');

                    if ($opts.length && (!this.$target.hasClass('o_record_has_cover') || notAllowed)) {
                        $el.addClass('d-none');
                    }
                });
            }
        });

        this.$el.find('[data-clear]').toggleClass('d-none', !this.$target.hasClass('o_record_has_cover'));

        this.$filterValueOpts.removeClass('active');
        this.$filterColorOpts.removeClass('active');

        var activeFilterValue = this.$filterValueOpts
            .filter((i, el) => {
                return (parseFloat($(el).data('filterValue')).toFixed(1) === parseFloat(this.$filter.css('opacity')).toFixed(1));
            }).addClass('active').data('filterValue');

        var activeFilterColor = this.$filterColorOpts
            .filter((i, el) => {
                return this.$filter.hasClass($(el).data('filterColor'));
            }).addClass('active').data('filterColor');

        this.$target[0].dataset.coverClass = this.$el.find('.active[data-cover-opt="size"]').data('selectClass') || '';
        this.$target[0].dataset.textSizeClass = this.$el.find('.active[data-cover-opt="text_size"]').data('selectClass') || '';
        this.$target[0].dataset.textAlignClass = this.$el.find('.active[data-cover-opt="text_align"]').data('selectClass') || '';
        this.$target[0].dataset.filterValue = activeFilterValue || 0.0;
        this.$target[0].dataset.filterColor = activeFilterColor || '';
    },
});
});

```

## File: static\src\js\editor\widget_link.js

```javascript
odoo.define('website.editor.link', function (require) {
'use strict';

var weWidgets = require('wysiwyg.widgets');
var wUtils = require('website.utils');

weWidgets.LinkDialog.include({
    xmlDependencies: (weWidgets.LinkDialog.prototype.xmlDependencies || []).concat(
        ['/website/static/src/xml/website.editor.xml']
    ),
    events: _.extend({}, weWidgets.LinkDialog.prototype.events || {}, {
        'change select[name="link_anchor"]': '_onAnchorChange',
        'input input[name="url"]': '_onURLInput',
    }),
    custom_events: _.extend({}, weWidgets.LinkDialog.prototype.custom_events || {}, {
        website_url_chosen: '_onAutocompleteClose',
    }),
    LINK_DEBOUNCE: 1000,

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this._adaptPageAnchor = _.debounce(this._adaptPageAnchor, this.LINK_DEBOUNCE);
    },
    /**
     * Allows the URL input to propose existing website pages.
     *
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        wUtils.autocompleteWithPages(this, this.$('input[name="url"]'));
        this.opened(this._adaptPageAnchor.bind(this));
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptPageAnchor: function () {
        var urlInputValue = this.$('input[name="url"]').val();
        var $pageAnchor = this.$('.o_link_dialog_page_anchor');
        var isFromWebsite = urlInputValue[0] === '/';
        var $selectMenu = this.$('select[name="link_anchor"]');
        var $anchorsLoading = this.$('.o_anchors_loading');

        $anchorsLoading.removeClass('d-none');
        $pageAnchor.toggleClass('d-none', !isFromWebsite);
        $selectMenu.empty();
        wUtils.loadAnchors(urlInputValue).then(function (anchors) {
            _.each(anchors, function (anchor) {
                $selectMenu.append($('<option>', {text: anchor}));
            });
            always();
        }).guardedCatch(always);

        function always() {
            $anchorsLoading.addClass('d-none');
            $selectMenu.prop("selectedIndex", -1);
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onAutocompleteClose: function () {
        this._onURLInput();
    },
    /**
     * @private
     */
    _onAnchorChange: function () {
        var anchorValue = this.$('[name="link_anchor"]').val();
        var $urlInput = this.$('[name="url"]');
        var urlInputValue = $urlInput.val();
        if (urlInputValue.indexOf('#') > -1) {
            urlInputValue = urlInputValue.substr(0, urlInputValue.indexOf('#'));
        }
        $urlInput.val(urlInputValue + anchorValue);
    },
    /**
     * @override
     */
    _onURLInput: function () {
        this._super.apply(this, arguments);
        this._adaptPageAnchor();
    },
});
});

```

## File: static\src\js\editor\wysiwyg_multizone.js

```javascript
odoo.define('web_editor.wysiwyg.multizone', function (require) {
'use strict';

var Wysiwyg = require('web_editor.wysiwyg');
var snippetsEditor = require('web_editor.snippet.editor');

/**
 * Show/hide the dropdowns associated to the given toggles and allows to wait
 * for when it is fully shown/hidden.
 *
 * Note: this also takes care of the fact the 'toggle' method of bootstrap does
 * not properly work in all cases.
 *
 * @param {jQuery} $toggles
 * @param {boolean} [show]
 * @returns {Promise<jQuery>}
 */
function toggleDropdown($toggles, show) {
    return Promise.all(_.map($toggles, toggle => {
        var $toggle = $(toggle);
        var $dropdown = $toggle.parent();
        var shown = $dropdown.hasClass('show');
        if (shown === show) {
            return;
        }
        var toShow = !shown;
        return new Promise(resolve => {
            $dropdown.one(
                toShow ? 'shown.bs.dropdown' : 'hidden.bs.dropdown',
                () => resolve()
            );
            $toggle.dropdown(toShow ? 'show' : 'hide');
        });
    })).then(() => $toggles);
}

/**
 * HtmlEditor
 * Intended to edit HTML content. This widget uses the Wysiwyg editor
 * improved by odoo.
 *
 * class editable: o_editable
 * class non editable: o_not_editable
 *
 */
var WysiwygMultizone = Wysiwyg.extend({
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.options.toolbarHandler = $('#web_editor-top-edit');
        this.options.saveElement = function ($el, context, withLang) {
            var outerHTML = this._getEscapedElement($el).prop('outerHTML');
            return self._saveElement(outerHTML, self.options.recordInfo, $el[0]);
        };

        // Mega menu initialization: handle dropdown openings by hand
        var $megaMenuToggles = this.$('.o_mega_menu_toggle');
        $megaMenuToggles.removeAttr('data-toggle').dropdown('dispose');
        $megaMenuToggles.on('click.wysiwyg_multizone', ev => {
            var $toggle = $(ev.currentTarget);

            // Each time we toggle a dropdown, we will destroy the dropdown
            // behavior afterwards to keep manual control of it
            var dispose = ($els => $els.dropdown('dispose'));

            // First hide all other mega menus
            toggleDropdown($megaMenuToggles.not($toggle), false).then(dispose);

            // Then toggle the clicked one
            toggleDropdown($toggle)
                .then(dispose)
                .then($el => {
                    var isShown = $el.parent().hasClass('show');
                    this.editor.snippetsMenu.toggleMegaMenuSnippets(isShown);
                });
        });

        // TODO remove me in master, this should just be solved in master XML
        // if required. Keep this in stable for now though.
        _.each(this.$('.oe_structure[data-editor-message]'), el => {
            if (!el.dataset.editorMessage || el.dataset.editorMessage === "False") {
                return;
            }
            var isBlank = !el.innerHTML.trim();
            if (isBlank) {
                el.innerHTML = '';
            }
            el.classList.toggle('oe_empty', isBlank);
        });

        return this._super.apply(this, arguments).then(() => {
            // Showing Mega Menu snippets if one dropdown is already opened
            if (this.$('.o_mega_menu').hasClass('show')) {
                this.editor.snippetsMenu.toggleMegaMenuSnippets(true);
            }
        });
    },
    /**
     * @override
     * @returns {Promise}
     */
    save: function () {
        if (this.isDirty()) {
            return this._restoreMegaMenus()
                .then(() => this.editor.save(false))
                .then(() => ({isDirty: true}));
        } else {
            return {isDirty: false};
        }
    },
    /**
     * @override
     */
    destroy: function () {
        this._restoreMegaMenus();
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _getEditableArea: function () {
        return $(':o_editable');
    },
    /**
     * @private
     * @param {HTMLElement} editable
     */
    _saveCoverProperties: function (editable) {
        var el = editable.closest('.o_record_cover_container');
        if (!el) {
            return;
        }

        var resModel = el.dataset.resModel;
        var resID = parseInt(el.dataset.resId);
        if (!resModel || !resID) {
            throw new Error('There should be a model and id associated to the cover');
        }

        this.__savedCovers = this.__savedCovers || {};
        this.__savedCovers[resModel] = this.__savedCovers[resModel] || [];

        if (this.__savedCovers[resModel].includes(resID)) {
            return;
        }
        this.__savedCovers[resModel].push(resID);

        var cssBgImage = $(el.querySelector('.o_record_cover_image')).css('background-image');
        var coverProps = {
            'background-image': cssBgImage.replace(/"/g, '').replace(window.location.protocol + "//" + window.location.host, ''),
            'background-color': el.dataset.filterColor,
            'opacity': el.dataset.filterValue,
            'resize_class': el.dataset.coverClass,
            'text_size_class': el.dataset.textSizeClass,
            'text_align_class': el.dataset.textAlignClass,
        };

        return this._rpc({
            model: resModel,
            method: 'write',
            args: [
                resID,
                {'cover_properties': JSON.stringify(coverProps)}
            ],
        });
    },
    /**
     * Saves one (dirty) element of the page.
     *
     * @private
     * @param {jQuery} $el - the element to save
     * @param {Object} context - the context to use for the saving rpc
     * @param {boolean} [withLang=false]
     *        false if the lang must be omitted in the context (saving "master"
     *        page element)
     */
    _saveElement: function (outerHTML, recordInfo, editable) {
        var promises = [];

        var $el = $(editable);

        // Saving a view content
        var viewID = $el.data('oe-id');
        if (viewID) {
            promises.push(this._rpc({
                model: 'ir.ui.view',
                method: 'save',
                args: [
                    viewID,
                    outerHTML,
                    $el.data('oe-xpath') || null,
                ],
                context: recordInfo.context,
            }));
        }

        // Saving mega menu options
        if ($el.data('oe-field') === 'mega_menu_content') {
            // On top of saving the mega menu content like any other field
            // content, we must save the custom classes that were set on the
            // menu itself.
            // FIXME normally removing the 'show' class should not be necessary here
            // TODO check that editor classes are removed here as well
            var classes = _.without($el.attr('class').split(' '), 'dropdown-menu', 'o_mega_menu', 'show');
            promises.push(this._rpc({
                model: 'website.menu',
                method: 'write',
                args: [
                    [parseInt($el.data('oe-id'))],
                    {
                        'mega_menu_classes': classes.join(' '),
                    },
                ],
            }));
        }

        // Saving cover properties on related model if any
        var prom = this._saveCoverProperties(editable);
        if (prom) {
            promises.push(prom);
        }

        return Promise.all(promises);
    },
    /**
     * Restores mega menu behaviors and closes them (important to do before
     * saving otherwise they would be saved opened).
     *
     * @private
     * @returns {Promise}
     */
    _restoreMegaMenus: function () {
        var $megaMenuToggles = this.$('.o_mega_menu_toggle');
        $megaMenuToggles.off('.wysiwyg_multizone')
            .attr('data-toggle', 'dropdown')
            .dropdown({});
        return toggleDropdown($megaMenuToggles, false);
    },
});

snippetsEditor.Class.include({
    /**
     * @private
     * @param {boolean} show
     */
    toggleMegaMenuSnippets: function (show) {
        setTimeout(() => this._activateSnippet(false));
        this.$('#snippet_mega_menu').toggleClass('d-none', !show);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _insertDropzone: function ($hook) {
        var $hookParent = $hook.parent();
        var $dropzone = this._super(...arguments);
        $dropzone.attr('data-editor-message', $hookParent.attr('data-editor-message'));
        $dropzone.attr('data-editor-sub-message', $hookParent.attr('data-editor-sub-message'));
        return $dropzone;
    },
});

return WysiwygMultizone;
});

```

## File: static\src\js\editor\wysiwyg_multizone_translate.js

```javascript
odoo.define('web_editor.wysiwyg.multizone.translate', function (require) {
'use strict';

var core = require('web.core');
var webDialog = require('web.Dialog');
var WysiwygMultizone = require('web_editor.wysiwyg.multizone');
var rte = require('web_editor.rte');
var Dialog = require('wysiwyg.widgets.Dialog');
var websiteNavbarData = require('website.navbar');

var _t = core._t;


var RTETranslatorWidget = rte.Class.extend({
    /**
     * If the element holds a translation, saves it. Otherwise, fallback to the
     * standard saving but with the lang kept.
     *
     * @override
     */
    _saveElement: function ($el, context, withLang) {
        var self = this;
        if ($el.data('oe-translation-id')) {
            return this._rpc({
                model: 'ir.translation',
                method: 'save_html',
                args: [
                    [+$el.data('oe-translation-id')],
                    this._getEscapedElement($el).html()
                ],
                context: context,
            });
        }
        return this._super($el, context, withLang === undefined ? true : withLang);
    },
});

var AttributeTranslateDialog = Dialog.extend({
    /**
     * @constructor
     */
    init: function (parent, options, node) {
        this._super(parent, _.extend({
            title: _t("Translate Attribute"),
            buttons: [
                {text:  _t("Close"), classes: 'btn-primary', click: this.save}
            ],
        }, options || {}));
        this.translation = $(node).data('translation');
    },
    /**
     * @override
     */
    start: function () {
        var $group = $('<div/>', {class: 'form-group'}).appendTo(this.$el);
        _.each(this.translation, function (node, attr) {
            var $node = $(node);
            var $label = $('<label class="col-form-label"></label>').text(attr);
            var $input = $('<input class="form-control"/>').val($node.html());
            $input.on('change keyup', function () {
                var value = $input.val();
                $node.html(value).trigger('change', node);
                $node.data('$node').attr($node.data('attribute'), value).trigger('translate');
                $node.trigger('change');
            });
            $group.append($label).append($input);
        });
        return this._super.apply(this, arguments);
    }
});

var WysiwygTranslate = WysiwygMultizone.extend({
    custom_events: _.extend({}, WysiwygMultizone.prototype.custom_events || {}, {
        ready_to_save: '_onSave',
        rte_change: '_onChange',
    }),

    /**
     * @override
     * @param {string} options.lang
     */
    init: function (parent, options) {
        this.lang = options.lang;
        options.recordInfo = _.defaults({
                context: {lang: this.lang}
            }, options.recordInfo, options);
        this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.editor = new (this.Editor)(this, Object.assign({Editor: RTETranslatorWidget}, this.options));
        this.$editor = this.editor.rte.editable();
        var promise = this.editor.prependTo(this.$editor[0].ownerDocument.body);

        return promise.then(function () {
            self._relocateEditorBar();
            var attrs = ['placeholder', 'title', 'alt'];
            _.each(attrs, function (attr) {
                self._getEditableArea().filter('[' + attr + '*="data-oe-translation-id="]').filter(':empty, input, select, textarea, img').each(function () {
                    var $node = $(this);
                    var translation = $node.data('translation') || {};
                    var trans = $node.attr(attr);
                    var match = trans.match(/<span [^>]*data-oe-translation-id="([0-9]+)"[^>]*>(.*)<\/span>/);
                    var $trans = $(trans).addClass('d-none o_editable o_editable_translatable_attribute').appendTo('body');
                    $trans.data('$node', $node).data('attribute', attr);

                    translation[attr] = $trans[0];
                    $node.attr(attr, match[2]);

                    var select2 = $node.data('select2');
                    if (select2) {
                        select2.blur();
                        $node.on('translate', function () {
                            select2.blur();
                        });
                        $node = select2.container.find('input');
                    }
                    $node.addClass('o_translatable_attribute').data('translation', translation);
                });
            });

            self.translations = [];
            self.$editables_attr = self._getEditableArea().filter('.o_translatable_attribute');
            self.$editables_attribute = $('.o_editable_translatable_attribute');

            self.$editables_attribute.on('change', function () {
                self.trigger_up('rte_change', {target: this});
            });

            self._markTranslatableNodes();
        });
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {Boolean}
     */
    isDirty: function () {
        return this._super() || this.$editables_attribute.hasClass('o_dirty');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Return the editable area.
     *
     * @override
     * @returns {JQuery}
     */
    _getEditableArea: function () {
        var $editables = this._super();
        return $editables.add(this.$editables_attribute);
    },
    /**
     * Return an object describing the linked record.
     *
     * @override
     * @param {Object} options
     * @returns {Object} {res_id, res_model, xpath}
     */
    _getRecordInfo: function (options) {
        options = options || {};
        var recordInfo = this._super(options);
        var $editable = $(options.target).closest(this._getEditableArea());
        if (!$editable.length) {
            $editable = $(this._getFocusedEditable());
        }
        recordInfo.context.lang = this.lang;
        recordInfo.translation_id = $editable.data('oe-translation-id')|0;
        return recordInfo;
    },
    /**
     * @override
     * @returns {Object} the summernote configuration
     */
    _editorOptions: function () {
        var options = this._super();
        options.toolbar = [
            // todo: hide this feature for field (data-oe-model)
            ['font', ['bold', 'italic', 'underline', 'clear']],
            ['fontsize', ['fontsize']],
            ['color', ['color']],
            // keep every time
            ['history', ['undo', 'redo']],
        ];
        return options;
    },
    /**
     * Called when text is edited -> make sure text is not messed up and mark
     * the element as dirty.
     *
     * @override
     * @param {Jquery Event} [ev]
     */
    _onChange: function (ev) {
        var $node = $(ev.data.target);
        if (!$node.length) {
            return;
        }
        $node.find('div,p').each(function () { // remove P,DIV elements which might have been inserted because of copy-paste
            var $p = $(this);
            $p.after($p.html()).remove();
        });
        var trans = this._getTranlationObject($node[0]);
        $node.toggleClass('o_dirty', trans.value !== $node.html().replace(/[ \t\n\r]+/, ' '));
    },
    /**
     * Returns a translation object.
     *
     * @private
     * @param {Node} node
     * @returns {Object}
     */
    _getTranlationObject: function (node) {
        var $node = $(node);
        var id = +$node.data('oe-translation-id');
        if (!id) {
            id = $node.data('oe-model')+','+$node.data('oe-id')+','+$node.data('oe-field');
        }
        var trans = _.find(this.translations, function (trans) {
            return trans.id === id;
        });
        if (!trans) {
            this.translations.push(trans = {'id': id});
        }
        return trans;
    },
    /**
     * @private
     */
    _markTranslatableNodes: function () {
        var self = this;
        this._getEditableArea().each(function () {
            var $node = $(this);
            var trans = self._getTranlationObject(this);
            trans.value = (trans.value ? trans.value : $node.html() ).replace(/[ \t\n\r]+/, ' ');
        });
        this._getEditableArea().prependEvent('click.translator', function (ev) {
            if (ev.ctrlKey || !$(ev.target).is(':o_editable')) {
                return;
            }
            ev.preventDefault();
            ev.stopPropagation();
        });

        // attributes

        this.$editables_attr.each(function () {
            var $node = $(this);
            var translation = $node.data('translation');
            _.each(translation, function (node, attr) {
                var trans = self._getTranlationObject(node);
                trans.value = (trans.value ? trans.value : $node.html() ).replace(/[ \t\n\r]+/, ' ');
                $node.attr('data-oe-translation-state', (trans.state || 'to_translate'));
            });
        });

        this.$editables_attr.prependEvent('mousedown.translator click.translator mouseup.translator', function (ev) {
            if (ev.ctrlKey) {
                return;
            }
            ev.preventDefault();
            ev.stopPropagation();
            if (ev.type !== 'mousedown') {
                return;
            }

            new AttributeTranslateDialog(self, {}, ev.target).open();
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onSave: function (ev) {
        ev.stopPropagation();
    },
});

return WysiwygTranslate;
});

```

## File: static\src\js\menu\content.js

```javascript
odoo.define('website.contentMenu', function (require) {
'use strict';

var Class = require('web.Class');
var core = require('web.core');
var Dialog = require('web.Dialog');
var time = require('web.time');
var weWidgets = require('wysiwyg.widgets');
var websiteNavbarData = require('website.navbar');
var websiteRootData = require('website.root');
var Widget = require('web.Widget');

var _t = core._t;
var qweb = core.qweb;

var PagePropertiesDialog = weWidgets.Dialog.extend({
    template: 'website.pagesMenu.page_info',
    xmlDependencies: weWidgets.Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.pageProperties.xml']
    ),
    events: _.extend({}, weWidgets.Dialog.prototype.events, {
        'keyup input#page_name': '_onNameChanged',
        'keyup input#page_url': '_onUrlChanged',
        'change input#create_redirect': '_onCreateRedirectChanged',
    }),

    /**
     * @constructor
     * @override
     */
    init: function (parent, page_id, options) {
        var self = this;
        var serverUrl = window.location.origin + '/';
        var length_url = serverUrl.length;
        var serverUrlTrunc = serverUrl;
        if (length_url > 30) {
            serverUrlTrunc = serverUrl.slice(0,14) + '..' + serverUrl.slice(-14);
        }
        this.serverUrl = serverUrl;
        this.serverUrlTrunc = serverUrlTrunc;
        this.current_page_url = window.location.pathname;
        this.page_id = page_id;

        var buttons = [
            {text: _t("Save"), classes: 'btn-primary', click: this.save},
            {text: _t("Discard"), close: true},
        ];
        if (options.fromPageManagement) {
            buttons.push({
                text: _t("Go To Page"),
                icon: 'fa-globe',
                classes: 'btn-link float-right',
                click: function (e) {
                    window.location.href = '/' + self.page.url;
                },
            });
        }
        buttons.push({
            text: _t("Delete Page"),
            icon: 'fa-trash',
            classes: 'btn-link float-right',
            click: function (e) {
                _deletePage.call(this, self.page_id, options.fromPageManagement);
            },
        });
        this._super(parent, _.extend({}, {
            title: _t("Page Properties"),
            size: 'medium',
            buttons: buttons,
        }, options || {}));
    },
    /**
     * @override
     */
    willStart: function () {
        var defs = [this._super.apply(this, arguments)];
        var self = this;

        defs.push(this._rpc({
            model: 'website.page',
            method: 'get_page_info',
            args: [this.page_id],
        }).then(function (page) {
            page[0].url = _.str.startsWith(page[0].url, '/') ? page[0].url.substring(1) : page[0].url;
            self.page = page[0];
        }));

        defs.push(this._rpc({
            model: 'website.rewrite',
            method: 'fields_get',
        }).then(function (fields) {
            self.fields = fields;
        }));

        return Promise.all(defs);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;

        var defs = [this._super.apply(this, arguments)];

        this.$('.ask_for_redirect').addClass('d-none');
        this.$('.redirect_type').addClass('d-none');
        this.$('.warn_about_call').addClass('d-none');

        defs.push(this._getPageDependencies(this.page_id)
        .then(function (dependencies) {
            var dep_text = [];
            _.each(dependencies, function (value, index) {
                if (value.length > 0) {
                    dep_text.push(value.length + ' ' + index.toLowerCase());
                }
            });
            dep_text = dep_text.join(', ');
            self.$('#dependencies_redirect').html(qweb.render('website.show_page_dependencies', { dependencies: dependencies, dep_text: dep_text }));
            self.$('a.o_dependencies_redirect_link').on('click', () => {
                self.$('.o_dependencies_redirect_list_popover').popover({
                    html: true,
                    title: _t('Dependencies'),
                    boundary: 'viewport',
                    placement: 'right',
                    trigger: 'focus',
                    content: () => {
                        return qweb.render('website.get_tooltip_dependencies', {
                            dependencies: dependencies,
                        });
                    },
                    template: qweb.render('website.page_dependencies_popover'),
                }).popover('toggle');
            });
        }));

        defs.push(this._getSupportedMimetype()
        .then(function (mimetypes) {
            self.supportedMimetype = mimetypes;
        }));

        defs.push(this._getPageKeyDependencies(this.page_id)
        .then(function (dependencies) {
            var dep_text = [];
            _.each(dependencies, function (value, index) {
                if (value.length > 0) {
                    dep_text.push(value.length + ' ' + index.toLowerCase());
                }
            });
            dep_text = dep_text.join(', ');
            self.$('.warn_about_call').html(qweb.render('website.show_page_key_dependencies', {dependencies: dependencies, dep_text: dep_text}));
            self.$('.warn_about_call [data-toggle="popover"]').popover({
               container: 'body',
            });
        }));

        defs.push(this._rpc({model: 'res.users',
                             method: 'has_group',
                             args: ['website.group_multi_website']})
                  .then(function (has_group) {
                      if (!has_group) {
                          self.$('#website_restriction').addClass('hidden');
                      }
                  }));

        var datepickersOptions = {
            minDate: moment({y: 1900}),
            maxDate: moment().add(200, 'y'),
            calendarWeeks: true,
            icons : {
                time: 'fa fa-clock-o',
                date: 'fa fa-calendar',
                next: 'fa fa-chevron-right',
                previous: 'fa fa-chevron-left',
                up: 'fa fa-chevron-up',
                down: 'fa fa-chevron-down',
            },
            locale : moment.locale(),
            format : time.getLangDatetimeFormat(),
            widgetPositioning : {
                horizontal: 'auto',
                vertical: 'top',
            },
             widgetParent: 'body',
        };
        if (this.page.date_publish) {
            datepickersOptions.defaultDate = time.str_to_datetime(this.page.date_publish);
        }
        this.$('#date_publish_container').datetimepicker(datepickersOptions);

        return Promise.all(defs);
    },
    /**
     * @override
     */
    destroy: function () {
        $('.popover').popover('hide');
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    save: function (data) {
        var self = this;
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        var url = this.$('#page_url').val();

        var $date_publish = this.$("#date_publish");
        $date_publish.closest(".form-group").removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
        var date_publish = $date_publish.val();
        if (date_publish !== "") {
            date_publish = this._parse_date(date_publish);
            if (!date_publish) {
                $date_publish.closest(".form-group").addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                return;
            }
        }
        var params = {
            id: this.page.id,
            name: this.$('#page_name').val(),
            // Replace duplicate following '/' by only one '/'
            url: url.replace(/\/{2,}/g, '/'),
            is_menu: this.$('#is_menu').prop('checked'),
            is_homepage: this.$('#is_homepage').prop('checked'),
            website_published: this.$('#is_published').prop('checked'),
            create_redirect: this.$('#create_redirect').prop('checked'),
            redirect_type: this.$('#redirect_type').val(),
            website_indexed: this.$('#is_indexed').prop('checked'),
            date_publish: date_publish,
        };
        this._rpc({
            model: 'website.page',
            method: 'save_page_info',
            args: [[context.website_id], params],
        }).then(function (url) {
            // If from page manager: reload url, if from page itself: go to
            // (possibly) new url
            var mo;
            self.trigger_up('main_object_request', {
                callback: function (value) {
                    mo = value;
                },
            });
            if (mo.model === 'website.page') {
                window.location.href = url.toLowerCase();
            } else {
                window.location.reload(true);
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Retrieves the page URL dependencies for the given object id.
     *
     * @private
     * @param {integer} moID
     * @returns {Promise<Array>}
     */
    _getPageDependencies: function (moID) {
        return this._rpc({
            model: 'website',
            method: 'page_search_dependencies',
            args: [moID],
        });
    },
    /**
     * Retrieves the page's key dependencies for the given object id.
     *
     * @private
     * @param {integer} moID
     * @returns {Promise<Array>}
     */
    _getPageKeyDependencies: function (moID) {
        return this._rpc({
            model: 'website',
            method: 'page_search_key_dependencies',
            args: [moID],
        });
    },
    /**
     * Retrieves supported mimtype
     *
     * @private
     * @returns {Promise<Array>}
     */
    _getSupportedMimetype: function () {
        return this._rpc({
            model: 'website',
            method: 'guess_mimetype',
        });
    },
    /**
     * Returns information about the page main object.
     *
     * @private
     * @returns {Object} model and id
     */
    _getMainObject: function () {
        var repr = $('html').data('main-object');
        var m = repr.match(/(.+)\((\d+),(.*)\)/);
        return {
            model: m[1],
            id: m[2] | 0,
        };
    },
    /**
     * Converts a string representing the browser datetime
     * (exemple: Albanian: '2018-Qer-22 15.12.35.')
     * to a string representing UTC in Odoo's datetime string format
     * (exemple: '2018-04-22 13:12:35').
     *
     * The time zone of the datetime string is assumed to be the one of the
     * browser and it will be converted to UTC (standard for Odoo).
     *
     * @private
     * @param {String} value A string representing a datetime.
     * @returns {String|false} A string representing an UTC datetime if the given value is valid, false otherwise.
     */
    _parse_date: function (value) {
        var datetime = moment(value, time.getLangDatetimeFormat(), true);
        if (datetime.isValid()) {
            return time.datetime_to_str(datetime.toDate());
        }
        else {
            return false;
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onUrlChanged: function () {
        var url = this.$('input#page_url').val();
        this.$('.ask_for_redirect').toggleClass('d-none', url === this.page.url);
    },
    /**
     * @private
     */
    _onNameChanged: function () {
        var name = this.$('input#page_name').val();
        // If the file type is a supported mimetype, check if it is t-called.
        // If so, warn user. Note: different from page_search_dependencies which
        // check only for url and not key
        var ext = '.' + this.page.name.split('.').pop();
        if (ext in this.supportedMimetype && ext !== '.html') {
            this.$('.warn_about_call').toggleClass('d-none', name === this.page.name);
        }
    },
    /**
     * @private
     */
    _onCreateRedirectChanged: function () {
        var createRedirect = this.$('input#create_redirect').prop('checked');
        this.$('.redirect_type').toggleClass('d-none', !createRedirect);
    },
});

var MenuEntryDialog = weWidgets.LinkDialog.extend({
    xmlDependencies: weWidgets.LinkDialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.contentMenu.xml']
    ),

    /**
     * @constructor
     */
    init: function (parent, options, editable, data) {
        this._super(parent, _.extend({
            title: _t("Add a menu item"),
        }, options || {}), editable, _.extend({
            needLabel: true,
            text: data.name || '',
            isNewWindow: data.new_window,
        }, data || {}));

        this.menuType = data.menuType;
    },
    /**
     * @override
     */
    start: function () {
        // Remove style related elements
        this.$('.o_link_dialog_preview').remove();
        this.$('input[name="is_new_window"], .link-style').closest('.form-group').remove();
        this.$modal.find('.modal-lg').removeClass('modal-lg');
        this.$('form.col-lg-8').removeClass('col-lg-8').addClass('col-12');

        // Adapt URL label
        this.$('label[for="o_link_dialog_label_input"]').text(_t("Menu Label"));

        // Auto add '#' URL and hide the input if for mega menu
        if (this.menuType === 'mega') {
            var $url = this.$('input[name="url"]');
            $url.val('#').trigger('change');
            $url.closest('.form-group').addClass('d-none');
        }

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    save: function () {
        var $e = this.$('#o_link_dialog_label_input');
        if (!$e.val() || !$e[0].checkValidity()) {
            $e.closest('.form-group').addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            $e.focus();
            return;
        }
        return this._super.apply(this, arguments);
    },
});

var SelectEditMenuDialog = weWidgets.Dialog.extend({
    template: 'website.contentMenu.dialog.select',
    xmlDependencies: weWidgets.Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.contentMenu.xml']
    ),

    /**
     * @constructor
     * @override
     */
    init: function (parent, options) {
        var self = this;
        self.roots = [{id: null, name: _t("Top Menu")}];
        $('[data-content_menu_id]').each(function () {
            // Remove name fallback in master
            self.roots.push({id: $(this).data('content_menu_id'), name: $(this).attr('name') || $(this).data('menu_name')});
        });
        this._super(parent, _.extend({}, {
            title: _t("Select a Menu"),
            save_text: _t("Continue")
        }, options || {}));
    },
    /**
     * @override
     */
    save: function () {
        this.final_data = parseInt(this.$el.find('select').val() || null);
        this._super.apply(this, arguments);
    },
});

var EditMenuDialog = weWidgets.Dialog.extend({
    template: 'website.contentMenu.dialog.edit',
    xmlDependencies: weWidgets.Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.contentMenu.xml']
    ),
    events: _.extend({}, weWidgets.Dialog.prototype.events, {
        'click a.js_add_menu': '_onAddMenuButtonClick',
        'click button.js_delete_menu': '_onDeleteMenuButtonClick',
        'click button.js_edit_menu': '_onEditMenuButtonClick',
    }),

    /**
     * @constructor
     * @override
     */
    init: function (parent, options, rootID) {
        this._super(parent, _.extend({}, {
            title: _t("Edit Menu"),
            size: 'medium',
        }, options || {}));
        this.rootID = rootID;
    },
    /**
     * @override
     */
    willStart: function () {
        var defs = [this._super.apply(this, arguments)];
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        defs.push(this._rpc({
            model: 'website.menu',
            method: 'get_tree',
            args: [context.website_id, this.rootID],
        }).then(menu => {
            this.menu = menu;
            this.rootMenuID = menu.fields['id'];
            this.flat = this._flatenize(menu);
            this.toDelete = [];
        }));
        return Promise.all(defs);
    },
    /**
     * @override
     */
    start: function () {
        var r = this._super.apply(this, arguments);
        this.$('.oe_menu_editor').nestedSortable({
            listType: 'ul',
            handle: 'div',
            items: 'li',
            maxLevels: 2,
            toleranceElement: '> div',
            forcePlaceholderSize: true,
            opacity: 0.6,
            placeholder: 'oe_menu_placeholder',
            tolerance: 'pointer',
            attribute: 'data-menu-id',
            expression: '()(.+)', // nestedSortable takes the second match of an expression (*sigh*)
            isAllowed: (placeholder, placeholderParent, currentItem) => {
                return !placeholderParent
                    || !currentItem[0].dataset.megaMenu && !placeholderParent[0].dataset.megaMenu;
            },
        });
        return r;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    save: function () {
        var _super = this._super.bind(this);
        var newMenus = this.$('.oe_menu_editor').nestedSortable('toArray', {startDepthCount: 0});
        var levels = [];
        var data = [];
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        // Resequence, re-tree and remove useless data
        newMenus.forEach(menu => {
            if (menu.id) {
                levels[menu.depth] = (levels[menu.depth] || 0) + 1;
                var menuFields = this.flat[menu.id].fields;
                menuFields['sequence'] = levels[menu.depth];
                menuFields['parent_id'] = menu['parent_id'] || this.rootMenuID;
                data.push(menuFields);
            }
        });
        return this._rpc({
            model: 'website.menu',
            method: 'save',
            args: [
                context.website_id,
                {
                    'data': data,
                    'to_delete': this.toDelete,
                }
            ],
        }).then(function () {
            return _super();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns a mapping id -> menu item containing all the menu items in the
     * given menu hierarchy.
     *
     * @private
     * @param {Object} node
     * @param {Object} [_dict] internal use: the mapping being built
     * @returns {Object}
     */
    _flatenize: function (node, _dict) {
        _dict = _dict || {};
        _dict[node.fields['id']] = node;
        node.children.forEach(child => {
            this._flatenize(child, _dict);
        });
        return _dict;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the "add menu" button is clicked -> Opens the appropriate
     * dialog to edit this new menu.
     *
     * @private
     * @param {Event} ev
     */
    _onAddMenuButtonClick: function (ev) {
        var menuType = ev.currentTarget.dataset.type;
        var dialog = new MenuEntryDialog(this, {}, null, {
            menuType: menuType,
        });
        dialog.on('save', this, link => {
            var newMenu = {
                'fields': {
                    'id': _.uniqueId('new-'),
                    'name': link.text,
                    'url': link.url,
                    'new_window': link.isNewWindow,
                    'is_mega_menu': menuType === 'mega',
                    'sequence': 0,
                    'parent_id': false,
                },
                'children': [],
                'is_homepage': false,
            };
            this.flat[newMenu.fields['id']] = newMenu;
            this.$('.oe_menu_editor').append(
                qweb.render('website.contentMenu.dialog.submenu', {submenu: newMenu})
            );
        });
        dialog.open();
    },
    /**
     * Called when the "delete menu" button is clicked -> Deletes this menu.
     *
     * @private
     */
    _onDeleteMenuButtonClick: function (ev) {
        var $menu = $(ev.currentTarget).closest('[data-menu-id]');
        var menuID = parseInt($menu.data('menu-id'));
        if (menuID) {
            this.toDelete.push(menuID);
        }
        $menu.remove();
    },
    /**
     * Called when the "edit menu" button is clicked -> Opens the appropriate
     * dialog to edit this menu.
     *
     * @private
     */
    _onEditMenuButtonClick: function (ev) {
        var $menu = $(ev.currentTarget).closest('[data-menu-id]');
        var menuID = $menu.data('menu-id');
        var menu = this.flat[menuID];
        if (menu) {
            var dialog = new MenuEntryDialog(this, {}, null, _.extend({
                menuType: menu.fields['is_mega_menu'] ? 'mega' : undefined,
            }, menu.fields));
            dialog.on('save', this, link => {
                _.extend(menu.fields, {
                    'name': link.text,
                    'url': link.url,
                    'new_window': link.isNewWindow,
                });
                $menu.find('.js_menu_label').first().text(menu.fields['name']);
            });
            dialog.open();
        } else {
            Dialog.alert(null, "Could not find menu entry");
        }
    },
});

var PageOption = Class.extend({
    /**
     * @constructor
     * @param {string} name
     *        the option's name = the field's name in website.page model
     * @param {*} value
     * @param {function} setValueCallback
     *        a function which simulates an option's value change without
     *        asking the server to change it
     */
    init: function (name, value, setValueCallback) {
        this.name = name;
        this.value = value;
        this.isDirty = false;
        this.setValueCallback = setValueCallback;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Sets the new option's value thanks to the related callback.
     *
     * @param {*} [value]
     *        by default: consider the current value is a boolean and toggle it
     */
    setValue: function (value) {
        if (value === undefined) {
            value = !this.value;
        }
        this.setValueCallback.call(this, value);
        this.value = value;
        this.isDirty = true;
    },
});

var ContentMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    xmlDependencies: ['/website/static/src/xml/website.xml'],
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        edit_menu: '_editMenu',
        get_page_option: '_getPageOption',
        on_save: '_onSave',
        page_properties: '_pageProperties',
        toggle_page_option: '_togglePageOption',
    }),
    pageOptionsSetValueCallbacks: {
        header_overlay: function (value) {
            $('#wrapwrap').toggleClass('o_header_overlay', value);
        },
        header_color: function (value) {
            $('#wrapwrap > header').removeClass(this.value)
                                   .addClass(value);
        },
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        this.pageOptions = {};
        _.each($('.o_page_option_data'), function (el) {
            var value = el.value;
            if (value === "True") {
                value = true;
            } else if (value === "False") {
                value = false;
            }
            self.pageOptions[el.name] = new PageOption(
                el.name,
                value,
                self.pageOptionsSetValueCallbacks[el.name]
            );
        });
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user which menu to edit if multiple menus exist on the page.
     * Then opens the menu edition dialog.
     * Then executes the given callback once the edition is saved, to finally
     * reload the page.
     *
     * @private
     * @param {function} [beforeReloadCallback]
     * @returns {Promise}
     *          Unresolved if the menu is edited and saved as the page will be
     *          reloaded.
     *          Resolved otherwise.
     */
    _editMenu: function (beforeReloadCallback) {
        var self = this;
        return new Promise(function (resolve) {
            function resolveWhenEditMenuDialogIsCancelled(rootID) {
                return self._openEditMenuDialog(rootID, beforeReloadCallback).then(resolve);
            }
            if ($('[data-content_menu_id]').length) {
                var select = new SelectEditMenuDialog(self);
                select.on('save', self, resolveWhenEditMenuDialogIsCancelled);
                select.on('cancel', self, resolve);
                select.open();
            } else {
                resolveWhenEditMenuDialogIsCancelled(null);
            }
        });
    },
    /**
     *
     * @param {*} rootID
     * @param {function|undefied} beforeReloadCallback function that returns a promise
     * @returns {Promise}
     */
    _openEditMenuDialog: function (rootID, beforeReloadCallback) {
        var self = this;
        return new Promise(function (resolve) {
            var dialog = new EditMenuDialog(self, {}, rootID);
            dialog.on('save', self, function () {
                // Before reloading the page after menu modification, does the
                // given action to do.
                if (beforeReloadCallback) {
                    // Reload the page so that the menu modification are shown
                    beforeReloadCallback().then(function () {
                        window.location.reload(true);
                    });
                } else {
                    window.location.reload(true);
                }
            });
            dialog.on('cancel', self, resolve);
            dialog.open();
        });
    },

    /**
     * Retrieves the value of a page option.
     *
     * @private
     * @param {string} name
     * @returns {Promise<*>}
     */
    _getPageOption: function (name) {
        var option = this.pageOptions[name];
        if (!option) {
            return Promise.reject();
        }
        return Promise.resolve(option.value);
    },
    /**
     * On save, simulated page options have to be server-saved.
     *
     * @private
     * @returns {Promise}
     */
    _onSave: function () {
        var self = this;
        var defs = _.map(this.pageOptions, function (option, optionName) {
            if (option.isDirty) {
                return self._togglePageOption({
                    name: optionName,
                    value: option.value,
                }, true, true);
            }
        });
        return Promise.all(defs);
    },
    /**
     * Opens the page properties dialog.
     *
     * @private
     * @returns {Promise}
     */
    _pageProperties: function () {
        var mo;
        this.trigger_up('main_object_request', {
            callback: function (value) {
                mo = value;
            },
        });
        var dialog = new PagePropertiesDialog(this, mo.id, {}).open();
        return dialog.opened();
    },
    /**
     * Toggles a page option.
     *
     * @private
     * @param {Object} params
     * @param {string} params.name
     * @param {*} [params.value] (change value by default true -> false -> true)
     * @param {boolean} [forceSave=false]
     * @param {boolean} [noReload=false]
     * @returns {Promise}
     */
    _togglePageOption: function (params, forceSave, noReload) {
        // First check it is a website page
        var mo;
        this.trigger_up('main_object_request', {
            callback: function (value) {
                mo = value;
            },
        });
        if (mo.model !== 'website.page') {
            return Promise.reject();
        }

        // Check if this is a valid option
        var option = this.pageOptions[params.name];
        if (!option) {
            return Promise.reject();
        }

        // Toggle the value
        option.setValue(params.value);

        // If simulate is true, it means we want the option to be toggled but
        // not saved on the server yet
        if (!forceSave) {
            return Promise.resolve();
        }

        // If not, write on the server page and reload the current location
        var vals = {};
        vals[params.name] = option.value;
        var prom = this._rpc({
            model: 'website.page',
            method: 'write',
            args: [[mo.id], vals],
        });
        if (noReload) {
            return prom;
        }
        return prom.then(function () {
            window.location.reload();
            return new Promise(function () {});
        });
    },
});

var PageManagement = Widget.extend({
    xmlDependencies: ['/website/static/src/xml/website.xml'],
    events: {
        'click a.js_page_properties': '_onPagePropertiesButtonClick',
        'click a.js_clone_page': '_onClonePageButtonClick',
        'click a.js_delete_page': '_onDeletePageButtonClick',
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Retrieves the page dependencies for the given object id.
     *
     * @private
     * @param {integer} moID
     * @returns {Promise<Array>}
     */
    _getPageDependencies: function (moID) {
        return this._rpc({
            model: 'website',
            method: 'page_search_dependencies',
            args: [moID],
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onPagePropertiesButtonClick: function (ev) {
        var moID = $(ev.currentTarget).data('id');
        var dialog = new PagePropertiesDialog(this,moID, {'fromPageManagement': true}).open();
        return dialog;
    },
    _onClonePageButtonClick: function (ev) {
        var pageId = $(ev.currentTarget).data('id');
        this._rpc({
            model: 'website.page',
            method: 'clone_page',
            args: [pageId],
        }).then(function (path) {
            window.location.href = path;
        });
    },
    _onDeletePageButtonClick: function (ev) {
        var pageId = $(ev.currentTarget).data('id');
        _deletePage.call(this, pageId, true);
    },
});

/**
 * Deletes the page after showing a dependencies warning for the given page id.
 *
 * @private
 * @param {integer} pageId - The ID of the page to be deleted
 * @param {Boolean} fromPageManagement
 *                  Is the function called by the page manager?
 *                  It will affect redirect after page deletion: reload or '/'
 */
// TODO: This function should be integrated in a widget in the future
function _deletePage(pageId, fromPageManagement) {
    var self = this;
    new Promise(function (resolve, reject) {
        // Search the page dependencies
        self._getPageDependencies(pageId)
        .then(function (dependencies) {
            // Inform the user about those dependencies and ask him confirmation
            return new Promise(function (confirmResolve, confirmReject) {
                Dialog.safeConfirm(self, "", {
                    title: _t("Delete Page"),
                    $content: $(qweb.render('website.delete_page', {dependencies: dependencies})),
                    confirm_callback: confirmResolve,
                    cancel_callback: resolve,
                });
            });
        }).then(function () {
            // Delete the page if the user confirmed
            return self._rpc({
                model: 'website.page',
                method: 'unlink',
                args: [pageId],
            });
        }).then(function () {
            if (fromPageManagement) {
                window.location.reload(true);
            } else {
                window.location.href = '/';
            }
        }, reject);
    });
}

websiteNavbarData.websiteNavbarRegistry.add(ContentMenu, '#content-menu');
websiteRootData.websiteRootRegistry.add(PageManagement, '#list_website_pages');

return {
    PagePropertiesDialog: PagePropertiesDialog,
    ContentMenu: ContentMenu,
    EditMenuDialog: EditMenuDialog,
    MenuEntryDialog: MenuEntryDialog,
    SelectEditMenuDialog: SelectEditMenuDialog,
};
});

```

## File: static\src\js\menu\customize.js

```javascript
odoo.define('website.customizeMenu', function (require) {
'use strict';

var core = require('web.core');
var Widget = require('web.Widget');
var websiteNavbarData = require('website.navbar');
var WebsiteAceEditor = require('website.ace');

var qweb = core.qweb;

var CustomizeMenu = Widget.extend({
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],
    events: {
        'show.bs.dropdown': '_onDropdownShow',
        'click .dropdown-item[data-view-key]': '_onCustomizeOptionClick',
    },

    /**
     * @override
     */
    willStart: function () {
        this.viewName = $(document.documentElement).data('view-xmlid');
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    start: function () {
        if (!this.viewName) {
            _.defer(this.destroy.bind(this));
        }

        if (this.$el.is('.show')) {
            this._loadCustomizeOptions();
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Enables/Disables a view customization whose id is given.
     *
     * @private
     * @param {string} viewKey
     * @returns {Promise}
     *          Unresolved if the customization succeeded as the page will be
     *          reloaded.
     *          Rejected otherwise.
     */
    _doCustomize: function (viewKey) {
        return this._rpc({
            route: '/website/toggle_switchable_view',
            params: {
                'view_key': viewKey,
            },
        }).then(function () {
            window.location.reload();
            return new Promise(function () {});
        });
    },
    /**
     * Loads the information about the views which can be enabled/disabled on
     * the current page and shows them as switchable elements in the menu.
     *
     * @private
     * @return {Promise}
     */
    _loadCustomizeOptions: function () {
        if (this.__customizeOptionsLoaded) {
            return Promise.resolve();
        }
        this.__customizeOptionsLoaded = true;

        var $menu = this.$el.children('.dropdown-menu');
        return this._rpc({
            route: '/website/get_switchable_related_views',
            params: {
                key: this.viewName,
            },
        }).then(function (result) {
            var currentGroup = '';
            if (result.length) {
                $menu.append($('<div/>', {
                    class: 'dropdown-divider',
                    role: 'separator',
                }));
            }
            _.each(result, function (item) {
                if (currentGroup !== item.inherit_id[1]) {
                    currentGroup = item.inherit_id[1];
                    $menu.append('<li class="dropdown-header">' + currentGroup + '</li>');
                }
                var $a = $('<a/>', {href: '#', class: 'dropdown-item', 'data-view-key': item.key, role: 'menuitem'})
                            .append(qweb.render('website.components.switch', {id: 'switch-' + item.id, label: item.name}));
                $a.find('input').prop('checked', !!item.active);
                $menu.append($a);
            });
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when a view's related switchable element is clicked -> enable /
     * disable the related view.
     *
     * @private
     * @param {Event} ev
     */
    _onCustomizeOptionClick: function (ev) {
        ev.preventDefault();
        var viewKey = $(ev.currentTarget).data('viewKey');
        this._doCustomize(viewKey);
    },
    /**
     * @private
     */
    _onDropdownShow: function () {
        this._loadCustomizeOptions();
    },
});

var AceEditorMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        close_all_widgets: '_hideEditor',
        edit: '_enterEditMode',
        ace: '_launchAce',
    }),

    /**
     * Launches the ace editor automatically when the corresponding hash is in
     * the page URL.
     *
     * @override
     */
    start: function () {
        if (window.location.hash.substr(0, WebsiteAceEditor.prototype.hash.length) === WebsiteAceEditor.prototype.hash) {
            this._launchAce();
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * When handling the "edit" website action, the ace editor has to be closed.
     *
     * @private
     */
    _enterEditMode: function () {
        this._hideEditor();
    },
    /**
     * @private
     */
    _hideEditor: function () {
        if (this.globalEditor) {
            this.globalEditor.do_hide();
        }
    },
    /**
     * Launches the ace editor to be able to edit the templates and scss files
     * which are used by the current page.
     *
     * @private
     * @returns {Promise}
     */
    _launchAce: function () {
        var self = this;
        var prom = new Promise(function (resolve, reject) {
            self.trigger_up('action_demand', {
                actionName: 'close_all_widgets',
                onSuccess: resolve,
            });
        });
        prom.then(function () {
            if (self.globalEditor) {
                self.globalEditor.do_show();
                return Promise.resolve();
            } else {
                var currentHash = window.location.hash;
                var indexOfView = currentHash.indexOf("?res=");
                var initialResID = undefined;
                if (indexOfView >= 0) {
                    initialResID = currentHash.substr(indexOfView + ("?res=".length));
                    var parsedResID = parseInt(initialResID, 10);
                    if (parsedResID) {
                        initialResID = parsedResID;
                    }
                }

                self.globalEditor = new WebsiteAceEditor(self, $(document.documentElement).data('view-xmlid'), {
                    initialResID: initialResID,
                    defaultBundlesRestriction: [
                        'web.assets_frontend',
                        'web.assets_frontend_minimal',
                        'web.assets_frontend_lazy',
                    ],
                });
                return self.globalEditor.appendTo(document.body);
            }
        });

        return prom;
    },
});

websiteNavbarData.websiteNavbarRegistry.add(CustomizeMenu, '#customize-menu');
websiteNavbarData.websiteNavbarRegistry.add(AceEditorMenu, '#html_editor');

return CustomizeMenu;
});

```

## File: static\src\js\menu\debug_manager.js

```javascript
odoo.define('website.debugManager', function (require) {
'use strict';

var config = require('web.config');
var DebugManager = require('web.DebugManager');
var websiteNavbarData = require('website.navbar');

var DebugManagerMenu = websiteNavbarData.WebsiteNavbar.include({
    /**
     * @override
     */
    start: function () {
        if (config.isDebug()) {
            new DebugManager(this).prependTo(this.$('.o_menu_systray'));
        }
        return this._super.apply(this, arguments);
    },
});

return DebugManagerMenu;
});

```

## File: static\src\js\menu\edit.js

```javascript
odoo.define('website.editMenu', function (require) {
'use strict';

var core = require('web.core');
var EditorMenu = require('website.editor.menu');
var websiteNavbarData = require('website.navbar');

var _t = core._t;

/**
 * Adds the behavior when clicking on the 'edit' button (+ editor interaction)
 */
var EditPageMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    assetLibs: ['web_editor.compiled_assets_wysiwyg', 'website.compiled_assets_wysiwyg'],

    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions, {
        edit: '_startEditMode',
        on_save: '_onSave',
    }),
    custom_events: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.custom_events || {}, {
        content_will_be_destroyed: '_onContentWillBeDestroyed',
        content_was_recreated: '_onContentWasRecreated',
        snippet_will_be_cloned: '_onSnippetWillBeCloned',
        snippet_cloned: '_onSnippetCloned',
        snippet_dropped: '_onSnippetDropped',
        edition_will_stopped: '_onEditionWillStop',
        edition_was_stopped: '_onEditionWasStopped',
    }),

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        var context;
        this.trigger_up('context_get', {
            extra: true,
            callback: function (ctx) {
                context = ctx;
            },
        });
        this._editorAutoStart = (context.editable && window.location.search.indexOf('enable_editor') >= 0);
        var url = window.location.href.replace(/([?&])&*enable_editor[^&#]*&?/, '\$1');
        window.history.replaceState({}, null, url);
    },
    /**
     * Auto-starts the editor if necessary or add the welcome message otherwise.
     *
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        // If we auto start the editor, do not show a welcome message
        if (this._editorAutoStart) {
            return Promise.all([def, this._startEditMode()]);
        }

        // Check that the page is empty
        var $wrap = this._targetForEdition().filter('#wrapwrap.homepage').find('#wrap');

        if ($wrap.length && $wrap.html().trim() === '') {
            // If readonly empty page, show the welcome message
            this.$welcomeMessage = $(core.qweb.render('website.homepage_editor_welcome_message'));
            this.$welcomeMessage.addClass('o_homepage_editor_welcome_message');
            this.$welcomeMessage.css('min-height', $wrap.parent('main').height() - ($wrap.outerHeight(true) - $wrap.height()));
            $wrap.empty().append(this.$welcomeMessage);
        }

        setTimeout(function () {
            if ($('.o_tooltip.o_animated').length) {
                $('.o_tooltip_container').addClass('show');
            }
        }, 1000); // ugly hack to wait that tooltip is loaded

        return def;
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Creates an editor instance and appends it to the DOM. Also remove the
     * welcome message if necessary.
     *
     * @private
     * @returns {Promise}
     */
    _startEditMode: async function () {
        var self = this;
        if (this.editModeEnable) {
            return;
        }
        this.trigger_up('widgets_stop_request', {
            $target: this._targetForEdition(),
        });
        if (this.$welcomeMessage) {
            this.$welcomeMessage.detach(); // detach from the readonly rendering before the clone by summernote
        }
        this.editModeEnable = true;
        await new EditorMenu(this).prependTo(document.body);
        var $target = this._targetForEdition();
        this.$editorMessageElements = $target
            .find('.oe_structure.oe_empty, [data-oe-type="html"]')
            .not('[data-editor-message]')
            .attr('data-editor-message', _t('DRAG BUILDING BLOCKS HERE'));
        var res = await new Promise(function (resolve, reject) {
            self.trigger_up('widgets_start_request', {
                editableMode: true,
                onSuccess: resolve,
                onFailure: reject,
            });
        });
        // Trigger a mousedown on the main edition area to focus it,
        // which is required for Summernote to activate.
        this.$editorMessageElements.mousedown();
        return res;
    },
    /**
     * On save, the editor will ask to parent widgets if something needs to be
     * done first. The website navbar will receive that demand and asks to its
     * action-capable components to do something. For example, the content menu
     * handles page-related options saving. However, some users with limited
     * access rights do not have the content menu... but the website navbar
     * expects that the save action is performed. So, this empty action is
     * defined here so that all users have an 'on_save' related action.
     *
     * @private
     * @todo improve the system to somehow declare required/optional actions
     */
    _onSave: function () {},

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns the target for edition.
     *
     * @private
     * @returns {JQuery}
     */
    _targetForEdition: function () {
        return $('#wrapwrap'); // TODO should know about this element another way
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when content will be destroyed in the page. Notifies the
     * WebsiteRoot that is should stop the public widgets.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onContentWillBeDestroyed: function (ev) {
        this.trigger_up('widgets_stop_request', {
            $target: ev.data.$target,
        });
    },
    /**
     * Called when content was recreated in the page. Notifies the
     * WebsiteRoot that is should start the public widgets.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onContentWasRecreated: function (ev) {
        this.trigger_up('widgets_start_request', {
            editableMode: true,
            $target: ev.data.$target,
        });
    },
    /**
     * Called when edition will stop. Notifies the
     * WebsiteRoot that is should stop the public widgets.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onEditionWillStop: function (ev) {
        this.$editorMessageElements && this.$editorMessageElements.removeAttr('data-editor-message');
        this.trigger_up('widgets_stop_request', {
            $target: this._targetForEdition(),
        });
    },
    /**
     * Called when edition was stopped. Notifies the
     * WebsiteRoot that is should start the public widgets.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onEditionWasStopped: function (ev) {
        this.trigger_up('widgets_start_request', {
            $target: this._targetForEdition(),
        });
        this.editModeEnable = false;
    },
    /**
     * Called when a snippet is about to be cloned in the page. Notifies the
     * WebsiteRoot that is should destroy the animations for this snippet.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onSnippetWillBeCloned: function (ev) {
        this.trigger_up('widgets_stop_request', {
            $target: ev.data.$target,
        });
    },
    /**
     * Called when a snippet is cloned in the page. Notifies the WebsiteRoot
     * that is should start the public widgets for this snippet and the snippet it
     * was cloned from.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onSnippetCloned: function (ev) {
        this.trigger_up('widgets_start_request', {
            editableMode: true,
            $target: ev.data.$target,
        });
        // TODO: remove in saas-12.5, undefined $origin will restart #wrapwrap
        if (ev.data.$origin) {
            this.trigger_up('widgets_start_request', {
                editableMode: true,
                $target: ev.data.$origin,
            });
        }
    },
    /**
     * Called when a snippet is dropped in the page. Notifies the WebsiteRoot
     * that is should start the public widgets for this snippet.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onSnippetDropped: function (ev) {
        this.trigger_up('widgets_start_request', {
            editableMode: true,
            $target: ev.data.$target,
        });
    },
});

websiteNavbarData.websiteNavbarRegistry.add(EditPageMenu, '#edit-page-menu');
});

```

## File: static\src\js\menu\mobile_view.js

```javascript
odoo.define('website.mobile', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var websiteNavbarData = require('website.navbar');

var _t = core._t;

var MobilePreviewDialog = Dialog.extend({
    /**
     * Tweaks the modal so that it appears as a phone and modifies the iframe
     * rendering to show more accurate mobile view.
     *
     * @override
     */
    start: function () {
        var self = this;
        this.$modal.addClass('oe_mobile_preview');
        this.$modal.on('click', '.modal-header', function () {
            self.$el.toggleClass('o_invert_orientation');
        });
        this.$iframe = $('<iframe/>', {
            id: 'mobile-viewport',
            src: $.param.querystring(window.location.href, 'mobilepreview'),
        });
        this.$iframe.on('load', function (e) {
            self.$iframe.contents().find('body').removeClass('o_connected_user');
            self.$iframe.contents().find('#oe_main_menu_navbar').remove();
        });
        this.$iframe.appendTo(this.$el);

        return this._super.apply(this, arguments);
    },
});

var MobileMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        'show-mobile-preview': '_onMobilePreviewClick',
    }),

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the mobile action is triggered -> instantiate the mobile
     * preview dialog.
     *
     * @private
     */
    _onMobilePreviewClick: function () {
        new MobilePreviewDialog(this, {
            title: _t('Mobile preview') + ' <span class="fa fa-refresh"/>',
        }).open();
    },
});

websiteNavbarData.websiteNavbarRegistry.add(MobileMenu, '#mobile-menu');

return {
    MobileMenu: MobileMenu,
    MobilePreviewDialog: MobilePreviewDialog,
};
});

```

## File: static\src\js\menu\navbar.js

```javascript
odoo.define('website.navbar', function (require) {
'use strict';

var core = require('web.core');
var dom = require('web.dom');
var publicWidget = require('web.public.widget');
var concurrency = require('web.concurrency');
var Widget = require('web.Widget');
var websiteRootData = require('website.root');

var qweb = core.qweb;

var websiteNavbarRegistry = new publicWidget.RootWidgetRegistry();

var WebsiteNavbar = publicWidget.RootWidget.extend({
    xmlDependencies: ['/website/static/src/xml/website.xml'],
    events: _.extend({}, publicWidget.RootWidget.prototype.events || {}, {
        'click [data-action]': '_onActionMenuClick',
        'mouseover > ul > li.dropdown:not(.show)': '_onMenuHovered',
        'click .o_mobile_menu_toggle': '_onMobileMenuToggleClick',
        'mouseover #oe_applications:not(:has(.dropdown-item))': '_onOeApplicationsHovered',
    }),
    custom_events: _.extend({}, publicWidget.RootWidget.prototype.custom_events || {}, {
        'action_demand': '_onActionDemand',
        'edit_mode': '_onEditMode',
        'readonly_mode': '_onReadonlyMode',
        'ready_to_save': '_onSave',
    }),

    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        var self = this;
        var initPromise = new Promise(function (resolve) {
            self.resolveInit = resolve;
        });
        this._widgetDefs = [initPromise];
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        dom.initAutoMoreMenu(this.$('ul.o_menu_sections'), {
            maxWidth: function () {
                // The navbar contains different elements in community and
                // enterprise, so we check for both of them here only
                return self.$el.width()
                    - (self.$('.o_menu_systray').outerWidth(true) || 0)
                    - (self.$('ul#oe_applications').outerWidth(true) || 0)
                    - (self.$('.o_menu_toggle').outerWidth(true) || 0)
                    - (self.$('.o_menu_brand').outerWidth(true) || 0);
            },
        });
        return this._super.apply(this, arguments).then(function () {
            self.resolveInit();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _attachComponent: function () {
        var def = this._super.apply(this, arguments);
        this._widgetDefs.push(def);
        return def;
    },
    /**
     * As the WebsiteNavbar instance is designed to be unique, the associated
     * registry has been instantiated outside of the class and is simply
     * returned here.
     *
     * @override
     */
    _getRegistry: function () {
        return websiteNavbarRegistry;
    },
    /**
     * Searches for the automatic widget {@see RootWidget} which can handle that
     * action.
     *
     * @private
     * @param {string} actionName
     * @param {Array} params
     * @returns {Promise}
     */
    _handleAction: function (actionName, params, _i) {
        var self = this;
        return this._whenReadyForActions().then(function () {
            var defs = [];
            _.each(self._widgets, function (w) {
                if (!w.handleAction) {
                    return;
                }

                var def = w.handleAction(actionName, params);
                if (def !== null) {
                    defs.push(def);
                }
            });
            if (!defs.length) {
                // Handle the case where all action-capable components are not
                // instantiated yet (rare) -> retry some times to eventually abort
                if (_i > 50) {
                    console.warn(_.str.sprintf("Action '%s' was not able to be handled.", actionName));
                    return Promise.reject();
                }
                return concurrency.delay(100).then(function () {
                    return self._handleAction(actionName, params, (_i || 0) + 1);
                });
            }
            return Promise.all(defs).then(function (values) {
                if (values.length === 1) {
                    return values[0];
                }
                return values;
            });
        });
    },
    /**
     * @private
     */
    _whenReadyForActions: function () {
        return Promise.all(this._widgetDefs);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the backend applications menu is hovered -> fetch the
     * available menus and insert it in DOM.
     *
     * @private
     * @param {Event} ev
     */
    _onOeApplicationsHovered: function (ev) {
        var self = this;
        this._rpc({
            model: 'ir.ui.menu',
            method: 'load_menus_root',
            args: [],
        }).then(function (result) {
            self.$('#oe_applications .dropdown-menu').html(
                $(qweb.render('website.oe_applications_menu', {menu_data: result}))
            );
        });
    },
    /**
     * Called when an action menu is clicked -> searches for the automatic
     * widget {@see RootWidget} which can handle that action.
     *
     * @private
     * @param {Event} ev
     */
    _onActionMenuClick: function (ev) {
        var $button = $(ev.currentTarget);
        $button.prop('disabled', true);
        var always = function () {
            $button.prop('disabled', false);
        };
        this._handleAction($button.data('action')).then(always).guardedCatch(always);
    },
    /**
     * Called when an action is asked to be executed from a child widget ->
     * searches for the automatic widget {@see RootWidget} which can handle
     * that action.
     */
    _onActionDemand: function (ev) {
        var def = this._handleAction(ev.data.actionName, ev.data.params);
        if (ev.data.onSuccess) {
            def.then(ev.data.onSuccess);
        }
        if (ev.data.onFailure) {
            def.guardedCatch(ev.data.onFailure);
        }
    },
    /**
     * Called in response to edit mode activation -> hides the navbar.
     *
     * @private
     */
    _onEditMode: function () {
        this.$el.addClass('editing_mode');
        this.do_hide();
    },
    /**
     * Called when a submenu is hovered -> automatically opens it if another
     * menu was already opened.
     *
     * @private
     * @param {Event} ev
     */
    _onMenuHovered: function (ev) {
        var $opened = this.$('> ul > li.dropdown.show');
        if ($opened.length) {
            $opened.find('.dropdown-toggle').dropdown('toggle');
            $(ev.currentTarget).find('.dropdown-toggle').dropdown('toggle');
        }
    },
    /**
     * Called when the mobile menu toggle button is click -> modifies the DOM
     * to open the mobile menu.
     *
     * @private
     */
    _onMobileMenuToggleClick: function () {
        this.$el.parent().toggleClass('o_mobile_menu_opened');
    },
    /**
     * Called in response to edit mode activation -> hides the navbar.
     *
     * @private
     */
    _onReadonlyMode: function () {
        this.$el.removeClass('editing_mode');
        this.do_show();
    },
    /**
     * Called in response to edit mode saving -> checks if action-capable
     * children have something to save.
     *
     * @private
     * @param {OdooEvent} ev
     */
    _onSave: function (ev) {
        ev.data.defs.push(this._handleAction('on_save'));
    },
});

var WebsiteNavbarActionWidget = Widget.extend({
    /**
     * 'Action name' -> 'Handler name' object
     *
     * Any [data-action="x"] element inside the website navbar will
     * automatically trigger an action "x". This action can then be handled by
     * any `WebsiteNavbarActionWidget` instance if the action name "x" is
     * registered in this `actions` object.
     */
    actions: {},

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Checks if the widget can execute an action whose name is given, with the
     * given parameters. If it is the case, execute that action.
     *
     * @param {string} actionName
     * @param {Array} params
     * @returns {Promise|null} action's promise or null if no action was found
     */
    handleAction: function (actionName, params) {
        var action = this[this.actions[actionName]];
        if (action) {
            return Promise.resolve(action.apply(this, params || []));
        }
        return null;
    },
});

websiteRootData.websiteRootRegistry.add(WebsiteNavbar, '#oe_main_menu_navbar');

return {
    WebsiteNavbar: WebsiteNavbar,
    websiteNavbarRegistry: websiteNavbarRegistry,
    WebsiteNavbarActionWidget: WebsiteNavbarActionWidget,
};
});

```

## File: static\src\js\menu\new_content.js

```javascript
odoo.define('website.newMenu', function (require) {
'use strict';

var core = require('web.core');
var Dialog = require('web.Dialog');
var websiteNavbarData = require('website.navbar');
var wUtils = require('website.utils');

var qweb = core.qweb;
var _t = core._t;

var enableFlag = 'enable_new_content';

var NewContentMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        close_all_widgets: '_handleCloseDemand',
        new_page: '_createNewPage',
    }),
    events: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.events || {}, {
        'click': '_onBackgroundClick',
        'click [data-module-id]': '_onModuleIdClick',
        'keydown': '_onBackgroundKeydown',
    }),
    // allow text to be customized with inheritance
    newContentText: {
        failed: _t('Failed to install "%s"'),
        installInProgress: _t("The installation of an App is already in progress."),
        installNeeded: _t('Do you want to install the "%s" App?'),
        installPleaseWait: _t('Installing "%s"'),
    },

    /**
     * Prepare the navigation and find the modules to install.
     * Move not installed module buttons after installed modules buttons,
     * but keep the original index to be able to move back the pending install
     * button at its final position, so the user can click at the same place.
     *
     * @override
     */
    start: function () {
        this.pendingInstall = false;
        this.$newContentMenuChoices = this.$('#o_new_content_menu_choices');

        var $modules = this.$newContentMenuChoices.find('.o_new_content_element');
        _.each($modules, function (el, index) {
            var $el = $(el);
            $el.data('original-index', index);
            if ($el.data('module-id')) {
                $el.appendTo($el.parent());
                $el.find('a i, a p').addClass('text-muted');
            }
        });

        this.$firstLink = this.$newContentMenuChoices.find('a:eq(0)');
        this.$lastLink = this.$newContentMenuChoices.find('a:last');

        if ($.deparam.querystring()[enableFlag] !== undefined) {
            this._showMenu();
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Asks the user information about a new page to create, then creates it and
     * redirects the user to this new page.
     *
     * @private
     * @returns {Promise} Unresolved if there is a redirection
     */
    _createNewPage: function () {
        return wUtils.prompt({
            id: 'editor_new_page',
            window_title: _t("New Page"),
            input: _t("Page Title"),
            init: function () {
                var $group = this.$dialog.find('div.form-group');
                $group.removeClass('mb0');

                var $add = $('<div/>', {'class': 'form-group mb0 row'})
                            .append($('<span/>', {'class': 'offset-md-3 col-md-9 text-left'})
                                    .append(qweb.render('website.components.switch', {id: 'switch_addTo_menu', label: _t("Add to menu")})));
                $add.find('input').prop('checked', true);
                $group.after($add);
            }
        }).then(function (result) {
            var val = result.val;
            var $dialog = result.dialog;
            if (!val) {
                return;
            }
            var url = '/website/add/' + encodeURIComponent(val);
            if ($dialog.find('input[type="checkbox"]').is(':checked')) url +='?add_menu=1';
            document.location = url;
            return new Promise(function () {});
        });
    },
    /**
     * @private
     */
    _handleCloseDemand: function () {
        this._hideMenu();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Set the focus on the first link
     *
     * @private
     */
    _focusFirstLink: function () {
        this.$firstLink.focus();
    },
    /**
     * Set the focus on the last link
     *
     * @private
     */
    _focusLastLink: function () {
        this.$lastLink.focus();
    },
    /**
     * Hide the menu
     *
     * @private
     */
    _hideMenu: function () {
        this.$newContentMenuChoices.addClass('o_hidden');
        $('body').removeClass('o_new_content_open');
    },
    /**
     * Install a module
     *
     * @private
     * @param {number} moduleId: the module to install
     * @return {Promise}
     */
    _install: function (moduleId) {
        this.pendingInstall = true;
        $('body').css('pointer-events', 'none');
        return this._rpc({
            model: 'ir.module.module',
            method: 'button_immediate_install',
            args: [[moduleId]],
        }).guardedCatch(function () {
            $('body').css('pointer-events', '');
        });
    },
    /**
     * Show the menu
     *
     * @private
     * @returns {Promise}
     */
    _showMenu: function () {
        var self = this;
        return new Promise(function (resolve, reject) {
            self.trigger_up('action_demand', {
                actionName: 'close_all_widgets',
                onSuccess: resolve,
            });
        }).then(function () {
            self.firstTab = true;
            self.$newContentMenuChoices.removeClass('o_hidden');
            $('body').addClass('o_new_content_open');
            self.$('> a').focus();
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when the menu's toggle button is clicked:
     *  -> Opens the menu and reset the tab navigation (if closed)
     *  -> Close the menu (if open)
     * Called when a click outside the menu's options occurs -> Close the menu
     *
     * @private
     * @param {Event} ev
     */
    _onBackgroundClick: function (ev) {
        if (this.$newContentMenuChoices.hasClass('o_hidden')) {
            this._showMenu();
        } else {
            this._hideMenu();
        }
    },
    /**
     * Called when a keydown occurs:
     *  ESC -> Closes the modal
     *  TAB -> Navigation (captured in the modal)
     *
     * @private
     * @param {Event} ev
     */
    _onBackgroundKeydown: function (ev) {
        switch (ev.which) {
            case $.ui.keyCode.ESCAPE:
                this._hideMenu();
                break;
            case $.ui.keyCode.TAB:
                if (ev.shiftKey) {
                    if (this.firstTab || document.activeElement === this.$firstLink[0]) {
                        this._focusLastLink();
                        ev.preventDefault();
                    }
                } else {
                    if (this.firstTab || document.activeElement === this.$lastLink[0]) {
                        this._focusFirstLink();
                        ev.preventDefault();
                    }
                }
                this.firstTab = false;
                break;
        }
    },
    /**
     * Open the install dialog related to an element:
     *  - open the dialog depending on access right and another pending install
     *  - if ok to install, prepare the install action:
     *      - call the proper action on click
     *      - change the button text and style
     *      - handle the result (reload on the same page or error)
     *
     * @private
     * @param {Event} ev
     */
    _onModuleIdClick: function (ev) {
        var self = this;
        var $el = $(ev.currentTarget);
        var $i = $el.find('a i');
        var $p = $el.find('a p');

        var title = $p.text();
        var content = '';
        var buttons;

        var moduleId = $el.data('module-id');
        var name = $el.data('module-shortdesc');

        ev.stopPropagation();
        ev.preventDefault();

        if (this.pendingInstall) {
            content = this.newContentText.installInProgress;
        } else {
            content = _.str.sprintf(this.newContentText.installNeeded, name);
            buttons = [{
                text: _t("Install"),
                classes: 'btn-primary',
                close: true,
                click: function () {
                    // move the element where it will be after installation
                    var $finalPosition = self.$newContentMenuChoices
                        .find('.o_new_content_element:not([data-module-id])')
                        .filter(function () {
                            return $(this).data('original-index') < $el.data('original-index');
                        }).last();
                    if ($finalPosition) {
                        $el.fadeTo(400, 0, function () {
                            $el.insertAfter($finalPosition);
                            // change style to use spinner
                            $i.removeClass()
                                .addClass('fa fa-spin fa-spinner fa-pulse');
                            $p.removeClass('text-muted')
                                .text(_.str.sprintf(self.newContentText.installPleaseWait, name));
                            $el.fadeTo(1000, 1);
                        });
                    }

                    self._install(moduleId).then(function () {
                        window.location.href = window.location.origin + window.location.pathname + '?' + enableFlag;
                    }, function () {
                        $i.removeClass()
                            .addClass('fa fa-exclamation-triangle');
                        $p.text(_.str.sprintf(self.newContentText.failed, name));
                    });
                }
            },{
                text: _t("Cancel"),
                close: true,
            }];
        }

        new Dialog(this, {
            title: title,
            size: 'medium',
            $content: $('<p/>', {text: content}),
            buttons: buttons
        }).open();
    },
});

websiteNavbarData.websiteNavbarRegistry.add(NewContentMenu, '.o_new_content_menu');

return NewContentMenu;
});

```

## File: static\src\js\menu\seo.js

```javascript
odoo.define('website.seo', function (require) {
'use strict';

var core = require('web.core');
var Class = require('web.Class');
var Dialog = require('web.Dialog');
var mixins = require('web.mixins');
var rpc = require('web.rpc');
var Widget = require('web.Widget');
var weWidgets = require('wysiwyg.widgets');
var websiteNavbarData = require('website.navbar');

var _t = core._t;

// This replaces \b, because accents(e.g. à, é) are not seen as word boundaries.
// Javascript \b is not unicode aware, and words beginning or ending by accents won't match \b
var WORD_SEPARATORS_REGEX = '([\\u2000-\\u206F\\u2E00-\\u2E7F\'!"#\\$%&\\(\\)\\*\\+,\\-\\.\\/:;<=>\\?¿¡@\\[\\]\\^_`\\{\\|\\}~\\s]+|^|$)';

var Suggestion = Widget.extend({
    template: 'website.seo_suggestion',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    events: {
        'click .o_seo_suggestion': 'select',
    },

    init: function (parent, options) {
        this.keyword = options.keyword;
        this._super(parent);
    },
    select: function () {
        this.trigger('selected', this.keyword);
    },
});

var SuggestionList = Widget.extend({
    template: 'website.seo_suggestion_list',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],

    init: function (parent, options) {
        this.root = options.root;
        this.language = options.language;
        this.htmlPage = options.htmlPage;
        this._super(parent);
    },
    start: function () {
        this.refresh();
    },
    refresh: function () {
        var self = this;
        self.$el.append(_t("Loading..."));
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        var language = self.language || context.lang.toLowerCase();
        this._rpc({
            route: '/website/seo_suggest',
            params: {
                keywords: self.root,
                lang: language,
            },
        }).then(function (keyword_list) {
            self.addSuggestions(JSON.parse(keyword_list));
        });
    },
    addSuggestions: function (keywords) {
        var self = this;
        self.$el.empty();
        // TODO Improve algorithm + Ajust based on custom user keywords
        var regex = new RegExp(WORD_SEPARATORS_REGEX + self.root + WORD_SEPARATORS_REGEX, 'gi');
        keywords = _.map(_.uniq(keywords), function (word) {
            return word.replace(regex, '').trim();
        });
        // TODO Order properly ?
        _.each(keywords, function (keyword) {
            if (keyword) {
                var suggestion = new Suggestion(self, {
                    keyword: keyword,
                });
                suggestion.on('selected', self, function (word, language) {
                    self.trigger('selected', word, language);
                });
                suggestion.appendTo(self.$el);
            }
        });
     },
});

var Keyword = Widget.extend({
    template: 'website.seo_keyword',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    events: {
        'click a[data-action=remove-keyword]': 'destroy',
    },

    init: function (parent, options) {
        this.keyword = options.word;
        this.language = options.language;
        this.htmlPage = options.htmlPage;
        this.used_h1 = this.htmlPage.isInHeading1(this.keyword);
        this.used_h2 = this.htmlPage.isInHeading2(this.keyword);
        this.used_content = this.htmlPage.isInBody(this.keyword);
        this._super(parent);
    },
    start: function () {
        var self = this;
        this.$('.o_seo_keyword_suggestion').empty();
        this.suggestionList = new SuggestionList(this, {
            root: this.keyword,
            language: this.language,
            htmlPage: this.htmlPage,
        });
        this.suggestionList.on('selected', this, function (word, language) {
            this.trigger('selected', word, language);
        });
        return this.suggestionList.appendTo(this.$('.o_seo_keyword_suggestion')).then(function() {
            self.htmlPage.on('title-changed', self, self._updateTitle);
            self.htmlPage.on('description-changed', self, self._updateDescription);
            self._updateTitle();
            self._updateDescription();
        });
    },
    destroy: function () {
        this.trigger('removed');
        this._super();
    },
    _updateTitle: function () {
        var $title = this.$('.js_seo_keyword_title');
        if (this.htmlPage.isInTitle(this.keyword)) {
            $title.css('visibility', 'visible');
        } else {
            $title.css('visibility', 'hidden');
        }
    },
    _updateDescription: function () {
        var $description = this.$('.js_seo_keyword_description');
        if (this.htmlPage.isInDescription(this.keyword)) {
            $description.css('visibility', 'visible');
        } else {
            $description.css('visibility', 'hidden');
        }
    },
});

var KeywordList = Widget.extend({
    template: 'website.seo_list',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    maxKeywords: 10,

    init: function (parent, options) {
        this.htmlPage = options.htmlPage;
        this._super(parent);
    },
    start: function () {
        var self = this;
        var existingKeywords = self.htmlPage.keywords();
        if (existingKeywords.length > 0) {
            _.each(existingKeywords, function (word) {
                self.add.call(self, word);
            });
        }
    },
    keywords: function () {
        var result = [];
        this.$('.js_seo_keyword').each(function () {
            result.push($(this).data('keyword'));
        });
        return result;
    },
    isFull: function () {
        return this.keywords().length >= this.maxKeywords;
    },
    exists: function (word) {
        return _.contains(this.keywords(), word);
    },
    add: async function (candidate, language) {
        var self = this;
        // TODO Refine
        var word = candidate ? candidate.replace(/[,;.:<>]+/g, ' ').replace(/ +/g, ' ').trim().toLowerCase() : '';
        if (word && !self.isFull() && !self.exists(word)) {
            var keyword = new Keyword(self, {
                word: word,
                language: language,
                htmlPage: this.htmlPage,
            });
            keyword.on('removed', self, function () {
               self.trigger('list-not-full');
               self.trigger('content-updated', true);
            });
            keyword.on('selected', self, function (word, language) {
                self.trigger('selected', word, language);
            });
            await keyword.appendTo(self.$el);
        }
        if (self.isFull()) {
            self.trigger('list-full');
        }
        self.trigger('content-updated');
    },
});

var Preview = Widget.extend({
    template: 'website.seo_preview',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],

    init: function (parent, options) {
        this.title = options.title;
        this.url = options.url;
        this.description = options.description;
        if (this.description.length > 160) {
            this.description = this.description.substring(0, 159) + '…';
        }
        this._super(parent);
    },
});

var HtmlPage = Class.extend(mixins.PropertiesMixin, {
    init: function () {
        mixins.PropertiesMixin.init.call(this);
        this.initTitle = this.title();
        this.defaultTitle = $('meta[name="default_title"]').attr('content');
        this.initDescription = this.description();
    },
    url: function () {
        return window.location.origin + window.location.pathname;
    },
    title: function () {
        return $('title').text().trim();
    },
    changeTitle: function (title) {
        // TODO create tag if missing
        $('title').text(title.trim() || this.defaultTitle);
        this.trigger('title-changed', title);
    },
    description: function () {
        return ($('meta[name=description]').attr('content') || '').trim();
    },
    changeDescription: function (description) {
        // TODO create tag if missing
        $('meta[name=description]').attr('content', description);
        this.trigger('description-changed', description);
    },
    keywords: function () {
        var $keywords = $('meta[name=keywords]');
        var parsed = ($keywords.length > 0) && $keywords.attr('content') && $keywords.attr('content').split(',');
        return (parsed && parsed[0]) ? parsed: [];
    },
    changeKeywords: function (keywords) {
        // TODO create tag if missing
        $('meta[name=keywords]').attr('content', keywords.join(','));
    },
    headers: function (tag) {
        return $('#wrap '+tag).map(function () {
            return $(this).text();
        });
    },
    getOgMeta: function () {
        var ogImageUrl = $('meta[property="og:image"]').attr('content');
        var title = $('meta[property="og:title"]').attr('content');
        var description = $('meta[property="og:description"]').attr('content');
        return {
            ogImageUrl: ogImageUrl && ogImageUrl.replace(window.location.origin, ''),
            metaTitle: title,
            metaDescription: description,
        };
    },
    images: function () {
        return $('#wrap img').map(function () {
            var $img = $(this);
            return  {
                src: $img.attr('src'),
                alt: $img.attr('alt'),
            };
        });
    },
    company: function () {
        return $('html').attr('data-oe-company-name');
    },
    bodyText: function () {
        return $('body').children().not('.oe_seo_configuration').text();
    },
    heading1: function () {
        return $('body').children().not('.oe_seo_configuration').find('h1').text();
    },
    heading2: function () {
        return $('body').children().not('.oe_seo_configuration').find('h2').text();
    },
    isInBody: function (text) {
        return new RegExp(WORD_SEPARATORS_REGEX + text + WORD_SEPARATORS_REGEX, 'gi').test(this.bodyText());
    },
    isInTitle: function (text) {
        return new RegExp(WORD_SEPARATORS_REGEX + text + WORD_SEPARATORS_REGEX, 'gi').test(this.title());
    },
    isInDescription: function (text) {
        return new RegExp(WORD_SEPARATORS_REGEX + text + WORD_SEPARATORS_REGEX, 'gi').test(this.description());
    },
    isInHeading1: function (text) {
        return new RegExp(WORD_SEPARATORS_REGEX + text + WORD_SEPARATORS_REGEX, 'gi').test(this.heading1());
    },
    isInHeading2: function (text) {
        return new RegExp(WORD_SEPARATORS_REGEX + text + WORD_SEPARATORS_REGEX, 'gi').test(this.heading2());
    },
});

var MetaTitleDescription = Widget.extend({
    // Form and preview for SEO meta title and meta description
    //
    // We only want to show an alert for "description too small" on those cases
    // - at init and the description is not empty
    // - we reached past the minimum and went back to it
    // - focus out of the field
    // Basically we don't want the too small alert when the field is empty and
    // we start typing on it.
    template: 'website.seo_meta_title_description',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    events: {
        'input input[name=website_meta_title]': '_titleChanged',
        'input textarea[name=website_meta_description]': '_descriptionOnInput',
        'change textarea[name=website_meta_description]': '_descriptionOnChange',
    },
    maxRecommendedDescriptionSize: 300,
    minRecommendedDescriptionSize: 50,
    showDescriptionTooSmall: false,

    /**
     * @override
     */
    init: function (parent, options) {
        this.htmlPage = options.htmlPage;
        this.canEditTitle = !!options.canEditTitle;
        this.canEditDescription = !!options.canEditDescription;
        this.isIndexed = !!options.isIndexed;
        this.previewDescription = options.previewDescription;
        this._super(parent, options);
    },
    /**
     * @override
     */
    start: function () {
        this.$title = this.$('input[name=website_meta_title]');
        this.$description = this.$('textarea[name=website_meta_description]');
        this.$warning = this.$('div#website_meta_description_warning');
        this.$preview = this.$('.js_seo_preview');

        if (!this.canEditTitle) {
            this.$title.attr('disabled', true);
        }
        if (!this.canEditDescription) {
            this.$description.attr('disabled', true);
        }
        if (this.htmlPage.title().trim() !== this.htmlPage.defaultTitle.trim()) {
            this.$title.val(this.htmlPage.title());
        }
        if (this.htmlPage.description().trim() !== this.previewDescription) {
            this.$description.val(this.htmlPage.description());
        }

        this._descriptionOnChange();
    },
    /**
     * Get the current title
     */
    getTitle: function () {
        return this.$title.val().trim() || this.htmlPage.defaultTitle;
    },
    /**
     * Get the current description
     */
    getDescription: function () {
        return this.getRealDescription() || this.previewDescription;
    },
    /**
     * Get the current description chosen by the user
     */
    getRealDescription: function () {
        return this.$description.val() || '';
    },
    /**
     * @private
     */
    _titleChanged: function () {
        var self = this;
        self._renderPreview();
        self.trigger('title-changed');
    },
    /**
     * @private
     */
    _descriptionOnChange: function () {
        this.showDescriptionTooSmall = true;
        this._descriptionOnInput();
    },
    /**
     * @private
     */
    _descriptionOnInput: function () {
        var length = this.getDescription().length;

        if (length >= this.minRecommendedDescriptionSize) {
            this.showDescriptionTooSmall = true;
        } else if (length === 0) {
            this.showDescriptionTooSmall = false;
        }

        if (length > this.maxRecommendedDescriptionSize) {
            this.$warning.text(_t('Your description looks too long.')).show();
        } else if (this.showDescriptionTooSmall && length < this.minRecommendedDescriptionSize) {
            this.$warning.text(_t('Your description looks too short.')).show();
        } else {
            this.$warning.hide();
        }

        this._renderPreview();
        this.trigger('description-changed');
    },
    /**
     * @private
     */
    _renderPreview: function () {
        var indexed = this.isIndexed;
        var preview = "";
        if (indexed) {
            preview = new Preview(this, {
                title: this.getTitle(),
                description: this.getDescription(),
                url: this.htmlPage.url(),
            });
        } else {
            preview = new Preview(this, {
                description: _t("You have hidden this page from search results. It won't be indexed by search engines."),
            });
        }
        this.$preview.empty();
        preview.appendTo(this.$preview);
    },
});

var MetaKeywords = Widget.extend({
    // Form and table for SEO meta keywords
    template: 'website.seo_meta_keywords',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    events: {
        'keyup input[name=website_meta_keywords]': '_confirmKeyword',
        'click button[data-action=add]': '_addKeyword',
    },

    init: function (parent, options) {
        this.htmlPage = options.htmlPage;
        this._super(parent, options);
    },
    start: function () {
        var self = this;
        this.$input = this.$('input[name=website_meta_keywords]');
        this.keywordList = new KeywordList(this, {htmlPage: this.htmlPage});
        this.keywordList.on('list-full', this, function () {
            self.$input.attr({
                readonly: 'readonly',
                placeholder: "Remove a keyword first"
            });
            self.$('button[data-action=add]').prop('disabled', true).addClass('disabled');
        });
        this.keywordList.on('list-not-full', this, function () {
            self.$input.removeAttr('readonly').attr('placeholder', "");
            self.$('button[data-action=add]').prop('disabled', false).removeClass('disabled');
        });
        this.keywordList.on('selected', this, function (word, language) {
            self.keywordList.add(word, language);
        });
        this.keywordList.on('content-updated', this, function (removed) {
            self._updateTable(removed);
        });
        return this.keywordList.insertAfter(this.$('.table thead')).then(function() {
            self._getLanguages();
            self._updateTable();
        });
    },
    _addKeyword: function () {
        var $language = this.$('select[name=seo_page_language]');
        var keyword = this.$input.val();
        var language = $language.val().toLowerCase();
        this.keywordList.add(keyword, language);
        this.$input.val('').focus();
    },
    _confirmKeyword: function (e) {
        if (e.keyCode === 13) {
            this._addKeyword();
        }
    },
    _getLanguages: function () {
        var self = this;
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        this._rpc({
            route: '/website/get_languages',
        }).then(function (data) {
            self.$('#language-box').html(core.qweb.render('Configurator.language_promote', {
                'language': data,
                'def_lang': context.lang
            }));
        });
    },
    /*
     * Show the table if there is at least one keyword. Hide it otherwise.
     *
     * @private
     * @param {boolean} removed: a keyword is about to be removed,
     *   we need to exclude it from the count
     */
    _updateTable: function (removed) {
        var min = removed ? 1 : 0;
        if (this.keywordList.keywords().length > min) {
            this.$('table').show();
        } else {
            this.$('table').hide();
        }
    },
});

var MetaImageSelector = Widget.extend({
    template: 'website.seo_meta_image_selector',
    xmlDependencies: ['/website/static/src/xml/website.seo.xml'],
    events: {
        'click .o_meta_img_upload': '_onClickUploadImg',
        'click .o_meta_img': '_onClickSelectImg',
    },
    /**
     * @override
     * @param {widget} parent
     * @param {Object} data
     */
    init: function (parent, data) {
        this.metaTitle = data.title || '';
        this.activeMetaImg = data.metaImg;
        this.serverUrl = data.htmlpage.url();
        data.pageImages.unshift(_.str.sprintf('/web/image/website/%s/logo', odoo.session_info.website_id));
        data.pageImages.unshift(_.str.sprintf('/web/image/website/%s/social_default_image', odoo.session_info.website_id));
        this.images = _.uniq(data.pageImages);
        this.customImgUrl = _.contains(data.pageImages, data.metaImg) ? false : data.metaImg;
        this.previewDescription = data.previewDescription;
        this._setDescription(this.previewDescription);
        this._super(parent);
    },
    setTitle: function (title) {
        this.metaTitle = title;
        this._updateTemplateBody();
    },
    setDescription: function (description) {
        this._setDescription(description);
        this._updateTemplateBody();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Set the description, applying ellipsis if too long.
     *
     * @private
    */
    _setDescription: function (description) {
        this.metaDescription = description || this.previewDescription;
        if (this.metaDescription.length > 160) {
            this.metaDescription = this.metaDescription.substring(0, 159) + '…';
        }
    },

    /**
     * Update template.
     *
     * @private
    */
    _updateTemplateBody: function () {
        this.$el.empty();
        this.images = _.uniq(this.images);
        this.$el.append(core.qweb.render('website.og_image_body', {widget: this}));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called when a select image from list -> change the preview accordingly.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onClickSelectImg: function (ev) {
        var $img = $(ev.currentTarget);
        this.activeMetaImg = $img.find('img').attr('src');
        this._updateTemplateBody();
    },
    /**
     * Open a mediaDialog to select/upload image.
     *
     * @private
     * @param {MouseEvent} ev
     */
    _onClickUploadImg: function (ev) {
        var self = this;
        var $image = $('<img/>');
        var mediaDialog = new weWidgets.MediaDialog(this, {
            onlyImages: true,
            res_model: 'ir.ui.view',
        }, $image[0]);
        mediaDialog.open();
        mediaDialog.on('save', this, function (image) {
            self.activeMetaImg = image.src;
            self.customImgUrl = image.src;
            self._updateTemplateBody();
        });
    },
});

var SeoConfigurator = Dialog.extend({
    template: 'website.seo_configuration',
    xmlDependencies: Dialog.prototype.xmlDependencies.concat(
        ['/website/static/src/xml/website.seo.xml']
    ),
    canEditTitle: false,
    canEditDescription: false,
    canEditKeywords: false,
    canEditLanguage: false,

    init: function (parent, options) {
        options = options || {};
        _.defaults(options, {
            title: _t('Optimize SEO'),
            buttons: [
                {text: _t('Save'), classes: 'btn-primary', click: this.update},
                {text: _t('Discard'), close: true},
            ],
        });

        this._super(parent, options);
    },
    start: function () {
        var self = this;

        this.$modal.addClass('oe_seo_configuration');

        this.htmlPage = new HtmlPage();

        this.disableUnsavableFields().then(function () {
            // Image selector
            self.metaImageSelector = new MetaImageSelector(self, {
                htmlpage: self.htmlPage,
                title: self.htmlPage.getOgMeta().metaTitle,
                metaImg : self.metaImg || self.htmlPage.getOgMeta().ogImageUrl,
                pageImages : _.pluck(self.htmlPage.images().get(), 'src'),
                previewDescription: _t('The description will be generated by social media based on page content unless you specify one.'),
            });
            self.metaImageSelector.appendTo(self.$('.js_seo_image'));

            // title and description
            self.metaTitleDescription = new MetaTitleDescription(self, {
                htmlPage: self.htmlPage,
                canEditTitle: self.canEditTitle,
                canEditDescription: self.canEditDescription,
                isIndexed: self.isIndexed,
                previewDescription: _t('The description will be generated by search engines based on page content unless you specify one.'),
            });
            self.metaTitleDescription.on('title-changed', self, self.titleChanged);
            self.metaTitleDescription.on('description-changed', self, self.descriptionChanged);
            self.metaTitleDescription.appendTo(self.$('.js_seo_meta_title_description'));

            // keywords
            self.metaKeywords = new MetaKeywords(self, {htmlPage: self.htmlPage});
            self.metaKeywords.appendTo(self.$('.js_seo_meta_keywords'));
        });
    },
    /*
     * Reset meta tags to their initial value if not saved.
     *
     * @private
     */
    destroy: function () {
        if (!this.savedData) {
            this.htmlPage.changeTitle(this.htmlPage.initTitle);
            this.htmlPage.changeDescription(this.htmlPage.initDescription);
        }
        this._super.apply(this, arguments);
    },
    disableUnsavableFields: function () {
        var self = this;
        return this.loadMetaData().then(function (data) {
            // We only need a reload for COW when the copy is happening, therefore:
            // - no reload if we are not editing a view (condition: website_id === undefined)
            // - reload if generic page (condition: website_id === false)
            self.reloadOnSave = data.website_id === undefined ? false : !data.website_id;
            //If website.page, hide the google preview & tell user his page is currently unindexed
            self.isIndexed = (data && ('website_indexed' in data)) ? data.website_indexed : true;
            self.canEditTitle = data && ('website_meta_title' in data);
            self.canEditDescription = data && ('website_meta_description' in data);
            self.canEditKeywords = data && ('website_meta_keywords' in data);
            self.metaImg = data.website_meta_og_img;
            if (!self.canEditTitle && !self.canEditDescription && !self.canEditKeywords) {
                // disable the button to prevent an error if the current page doesn't use the mixin
                // we make the check here instead of on the view because we don't need to check
                // at every page load, just when the rare case someone clicks on this link
                // TODO don't show the modal but just an alert in this case
                self.$footer.find('button[data-action=update]').attr('disabled', true);
            }
        });
    },
    update: function () {
        var self = this;
        var data = {};
        if (this.canEditTitle) {
            data.website_meta_title = this.metaTitleDescription.$title.val();
        }
        if (this.canEditDescription) {
            data.website_meta_description = this.metaTitleDescription.$description.val();
        }
        if (this.canEditKeywords) {
            data.website_meta_keywords = this.metaKeywords.keywordList.keywords().join(', ');
        }
        data.website_meta_og_img = this.metaImageSelector.activeMetaImg;
        this.saveMetaData(data).then(function () {
            // We want to reload if we are editing a generic page
            // because it will become a specific page after this change (COW)
            // and we want the user to be on the page he just created.
            if (self.reloadOnSave) {
                window.location.href = self.htmlPage.url();
            } else {
                self.htmlPage.changeKeywords(self.metaKeywords.keywordList.keywords());
                self.savedData = true;
                self.close();
            }
        });
    },
    getMainObject: function () {
        var mainObject;
        this.trigger_up('main_object_request', {
            callback: function (value) {
                mainObject = value;
            },
        });
        return mainObject;
    },
    getSeoObject: function () {
        var seoObject;
        this.trigger_up('seo_object_request', {
            callback: function (value) {
                seoObject = value;
            },
        });
        return seoObject;
    },
    loadMetaData: function () {
        var obj = this.getSeoObject() || this.getMainObject();
        return new Promise(function (resolve, reject) {
            if (!obj) {
                // return Promise.reject(new Error("No main_object was found."));
                resolve(null);
            } else {
                var fields = ['website_meta_title', 'website_meta_description', 'website_meta_keywords'
                                ,'website_meta_og_img'];
                if (obj.model === 'website.page') {
                    fields.push('website_indexed');
                    fields.push('website_id');
                }
                rpc.query({
                    model: obj.model,
                    method: 'read',
                    args: [[obj.id], fields],
                }).then(function (data) {
                    if (data.length) {
                        var meta = data[0];
                        meta.model = obj.model;
                        resolve(meta);
                    } else {
                        resolve(null);
                    }
                }).guardedCatch(reject);
            }
        });
    },
    saveMetaData: function (data) {
        var obj = this.getSeoObject() || this.getMainObject();
        if (!obj) {
            return Promise.reject();
        } else {
            return this._rpc({
                model: obj.model,
                method: 'write',
                args: [[obj.id], data],
            });
        }
    },
    titleChanged: function () {
        var self = this;
        _.defer(function () {
            var title = self.metaTitleDescription.getTitle();
            self.htmlPage.changeTitle(title);
            self.metaImageSelector.setTitle(title);
        });
    },
    descriptionChanged: function () {
        var self = this;
        _.defer(function () {
            var description = self.metaTitleDescription.getRealDescription();
            self.htmlPage.changeDescription(description);
            self.metaImageSelector.setDescription(description);
        });
    },
});

var SeoMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        'promote-current-page': '_promoteCurrentPage',
    }),

    init: function (parent, options) {
        this._super(parent, options);

        if ($.deparam.querystring().enable_seo !== undefined) {
            this._promoteCurrentPage();
        }
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Opens the SEO configurator dialog.
     *
     * @private
     */
    _promoteCurrentPage: function () {
        new SeoConfigurator(this).open();
    },
});

websiteNavbarData.websiteNavbarRegistry.add(SeoMenu, '#promote-menu');

return {
    SeoConfigurator: SeoConfigurator,
    SeoMenu: SeoMenu,
};
});

```

## File: static\src\js\menu\translate.js

```javascript
odoo.define('website.translateMenu', function (require) {
'use strict';

var utils = require('web.utils');
var TranslatorMenu = require('website.editor.menu.translate');
var websiteNavbarData = require('website.navbar');

var TranslatePageMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    assetLibs: ['web_editor.compiled_assets_wysiwyg', 'website.compiled_assets_wysiwyg'],

    actions: _.extend({}, websiteNavbarData.WebsiteNavbar.prototype.actions || {}, {
        edit_master: '_goToMasterPage',
        translate: '_startTranslateMode',
    }),

    /**
     * @override
     */
    start: function () {
        var context;
        this.trigger_up('context_get', {
            extra: true,
            callback: function (ctx) {
                context = ctx;
            },
        });
        this._mustEditTranslations = context.edit_translations;
        if (this._mustEditTranslations) {
            var url = window.location.href.replace(/([?&])&*edit_translations[^&#]*&?/, '\$1');
            window.history.replaceState({}, null, url);

            this._startTranslateMode();
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Redirects the user to the same page but in the original language and in
     * edit mode.
     *
     * @private
     * @returns {Promise}
     */
    _goToMasterPage: function () {
        var current = document.createElement('a');
        current.href = window.location.toString();
        current.search += (current.search ? '&' : '?') + 'enable_editor=1';
        // we are in translate mode, the pathname starts with '/<url_code/'
        current.pathname = current.pathname.substr(Math.max(0, current.pathname.indexOf('/', 1)));

        var link = document.createElement('a');
        link.href = '/website/lang/default';
        link.search += (link.search ? '&' : '?') + 'r=' + encodeURIComponent(current.pathname + current.search + current.hash);

        window.location = link.href;
        return new Promise(function () {});
    },
    /**
     * Redirects the user to the same page in translation mode (or start the
     * translator is translation mode is already enabled).
     *
     * @private
     * @returns {Promise}
     */
    _startTranslateMode: function () {
        if (!this._mustEditTranslations) {
            window.location.search += '&edit_translations';
            return new Promise(function () {});
        }

        var translator = new TranslatorMenu(this);

        // We don't want the BS dropdown to close
        // when clicking in a element to translate
        $('.dropdown-menu').on('click', '.o_editable', function (ev) {
            ev.stopPropagation();
        });

        return translator.prependTo(document.body);
    },
});

websiteNavbarData.websiteNavbarRegistry.add(TranslatePageMenu, '.o_menu_systray:has([data-action="translate"])');
});

```

## File: static\src\js\tours\banner.js

```javascript
odoo.define("website.tour.banner", function (require) {
"use strict";

var core = require("web.core");
var tour = require("web_tour.tour");

var _t = core._t;

tour.register("banner", {
    url: "/",
}, [{
    trigger: "a[data-action=edit]",
    content: _t("<b>Click Edit</b> to start designing your homepage."),
    extra_trigger: ".homepage",
    position: "bottom",
}, {
    trigger: "#snippet_structure .oe_snippet:eq(1) .oe_snippet_thumbnail",
    content: _t("Drag the <i>Cover</i> block and drop it in your page."),
    position: "bottom",
    run: "drag_and_drop #wrap",
}, {
    trigger: "#wrapwrap .s_cover h1",
    content: _t("<b>Click on a text</b> to start editing it. <i>It's that easy to edit your content!</i>"),
    position: "bottom",
    run: "text",
}, {
    trigger: ".o_we_customize_panel",
    extra_trigger: "#wrapwrap .s_cover h1:not(:containsExact(\"Catchy Headline\"))",
    content: _t("Customize any block through this menu. Try to change the background color of this block."),
    position: "right",
}, {
    trigger: '.o_we_add_snippet_btn',
    content: _t("Go back to the blocks menu."),
    position: 'bottom',
}, {
    trigger: "#snippet_structure .oe_snippet:eq(3) .oe_snippet_thumbnail",
    content: _t("Drag another block in your page, below the cover."),
    position: "bottom",
    run: "drag_and_drop #wrap",
}, {
    trigger: "button[data-action=save]",
    content: _t("Click the <b>Save</b> button."),
    position: "bottom",
}, {
    trigger: "a[data-action=show-mobile-preview]",
    content: _t("Good Job! You have designed your homepage. Let's check how this page looks like on <b>mobile devices</b>."),
    position: "bottom",
}, {
    trigger: ".modal-dialog:has(#mobile-viewport) button[data-dismiss=modal]",
    content: _t("After having checked how it looks on mobile, <b>close the preview</b>."),
    position: "right",
}, {
    trigger: "#new-content-menu > a",
    content: _t("<p><b>Your homepage is live.</b></p><p>Let's add a new page for your site.</p>"),
    position: "bottom",
},  {
    trigger: "a[data-action=new_page]",
    content: _t("<p><b>Click here</b> to create a new page.</p>"),
    position: "bottom",
}, {
    trigger: ".modal-dialog #editor_new_page input[type=text]",
    content: _t("<p>Enter a title for the page.</p>"),
    position: "bottom",
}, {
    trigger: ".modal-footer button.btn-primary.btn-continue",
    content: _t("Click on <b>Continue</b> to create the page."),
    position: "bottom",
}, {
    trigger: "#snippet_structure .oe_snippet:eq(3) .oe_snippet_thumbnail",
    content: _t("Drag the block and drop it in your new page."),
    position: "bottom",
    run: "drag_and_drop #wrap",
}, {
    trigger: "button[data-action=save]",
    content: _t("Click the <b>Save</b> button."),
    position: "bottom",
}, {
    trigger: ".js_publish_management .js_publish_btn",
    content: _t("<b>That's it!</b><p>Your page is all set to go live. Click the <b>Publish</b> button to publish it on the website.</p>"),
    position: "bottom",
}]);
});

//==============================================================================

odoo.define("website.tour.contact", function (require) {
"use strict";

var core = require("web.core");
var tour = require("web_tour.tour");
var _t = core._t;

tour.register("contact", {
    url: "/page/contactus",
}, [{
    trigger: "li#customize-menu",
    content: _t("<b>Install a contact form</b> to improve this page."),
    extra_trigger: "#o_contact_mail",
    position: "bottom",
}, {
    trigger: "li#install_apps",
    content: _t("<b>Install new apps</b> to get more features. Let's install the <i>'Contact form'</i> app."),
    position: "bottom",
}]);
});

```

## File: static\src\js\tours\customize.js

```javascript
odoo.define('website.tour.customize', function (require) {
'use strict';

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;

tour.register('theme_customize', {
    url: '/',
}, [{
    trigger: 'button.o_theme_customize_color_primary, button.o_theme_customize_color_alpha',
    content: _t("Click here to choose your main branding color.<br/>It will recompute the palette with suggested matching colors."),
    position: 'bottom',
}]);
});

```

## File: static\src\js\widgets\ace.js

```javascript
odoo.define("website.ace", function (require) {
"use strict";

var AceEditor = require('web_editor.ace');

/**
 * Extends the default view editor so that the URL hash is updated with view ID
 */
var WebsiteAceEditor = AceEditor.extend({
    hash: '#advanced-view-editor',

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    do_hide: function () {
        this._super.apply(this, arguments);
        window.location.hash = "";
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _displayResource: function () {
        this._super.apply(this, arguments);
        this._updateHash();
    },
    /**
     * @override
     */
    _saveResources: function () {
        return this._super.apply(this, arguments).then((function () {
            var defs = [];
            if (this.currentType === 'xml') {
                // When saving a view, the view ID might change. Thus, the
                // active ID in the URL will be incorrect. After the save
                // reload, that URL ID won't be found and JS will crash.
                // We need to find the new ID (either because the view became
                // specific or because its parent was edited too and the view
                // got copy/unlink).
                var selectedView = _.findWhere(this.views, {id: this._getSelectedResource()});
                var context;
                this.trigger_up('context_get', {
                    callback: function (ctx) {
                        context = ctx;
                    },
                });
                defs.push(this._rpc({
                    model: 'ir.ui.view',
                    method: 'search_read',
                    fields: ['id'],
                    domain: [['key', '=', selectedView.key], ['website_id', '=', context.website_id]],
                }).then((function (view) {
                    if (view[0]) {
                        this._updateHash(view[0].id);
                    }
                }).bind(this)));
            }
            return Promise.all(defs).then((function () {
                window.location.reload();
                return new Promise(function () {});
            }));
        }).bind(this));
    },
    /**
     * @override
     */
    _switchType(type) {
        const ret = this._super(...arguments);

        if (type === 'scss') {
            // By default show the user_custom_rules.scss one as some people
            // would write rules in user_custom_bootstrap_overridden.scss
            // otherwise, not reading the comment inside explaining how that
            // file should be used.
            this._displayResource('/website/static/src/scss/user_custom_rules.scss');
        }

        return ret;
    },
    /**
     * @override
     */
    _resetResource: function () {
        return this._super.apply(this, arguments).then((function () {
            window.location.reload();
            return new Promise(function () {});
        }).bind(this));
    },
    /**
     * Adds the current resource ID in the URL.
     *
     * @private
     */
    _updateHash: function (resID) {
        window.location.hash = this.hash + "?res=" + (resID || this._getSelectedResource());
    },
});

return WebsiteAceEditor;
});

```

## File: static\src\js\widgets\theme.js

```javascript
odoo.define('website.theme', function (require) {
'use strict';

var config = require('web.config');
var core = require('web.core');
var Dialog = require('web.Dialog');
var Widget = require('web.Widget');
var weWidgets = require('wysiwyg.widgets');
var ColorpickerDialog = require('web.ColorpickerDialog');
var websiteNavbarData = require('website.navbar');

var _t = core._t;

var templateDef = null;

var QuickEdit = Widget.extend({
    xmlDependencies: ['/website/static/src/xml/website.editor.xml'],
    template: 'website.theme_customize_active_input',
    events: {
        'keydown input': '_onInputKeydown',
        'click .btn-secondary': '_onResetClick',
        'focusout': '_onFocusOut',
    },

    /**
     * @constructor
     */
    init: function (parent, value, unit) {
        this._super.apply(this, arguments);
        this.value = value;
        this.unit = unit;
        this._onFocusOut = _.debounce(this._onFocusOut, 100);
    },
    /**
     * @override
     */
    start: function () {
        this.$input = this.$('input');
        this.$input.select();
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {string} [value]
     */
    _save: function (value) {
        if (value === undefined) {
            value = parseFloat(this.$input.val());
            value = isNaN(value) ? 'null' : (value + this.unit);
        }
        this.trigger_up('QuickEdit:save', {
            value: value,
        });
        this.destroy();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onInputKeydown: function (ev) {
        var inputValue = this.$input.val();
        var value = 0;
        if (inputValue !== '') {
            value = parseFloat(this.$input.val());
            if (isNaN(value)) {
                return;
            }
        }
        switch (ev.which) {
            case $.ui.keyCode.UP:
                this.$input.val(value + 1);
                break;
            case $.ui.keyCode.DOWN:
                this.$input.val(value - 1);
                break;
            case $.ui.keyCode.ENTER:
                // Do not listen to change events, we want the user to be able
                // to confirm in all cases.
                this._save();
                break;
        }
    },
    /**
     * @private
     */
    _onFocusOut: function () {
        if (!this.$el.has(document.activeElement).length) {
            this._save();
        }
    },
    /**
     * @private
     */
    _onResetClick: function () {
        this._save('null');
    },
});

var ThemeCustomizeDialog = Dialog.extend({
    xmlDependencies: (Dialog.prototype.xmlDependencies || [])
        .concat(['/website/static/src/xml/website.editor.xml']),

    template: 'website.theme_customize',
    events: {
        'change .o_theme_customize_option_input': '_onChange',
        'click .checked .o_theme_customize_option_input[type="radio"]': '_onChange',
        'click .o_theme_customize_add_google_font': '_onAddGoogleFontClick',
        'click .o_theme_customize_delete_google_font': '_onDeleteGoogleFontClick',
    },

    CUSTOM_BODY_IMAGE_XML_ID: 'option_custom_body_image',

    /**
     * @constructor
     */
    init: function (parent, options) {
        options = options || {};
        this._super(parent, _.extend({
            title: _t("Customize Theme"),
            buttons: [],
        }, options));

        this.defaultTab = options.tab || 0;
        this.fontVariables = [];
    },
    /**
     * @override
     */
    willStart: function () {
        if (templateDef === null) {
            templateDef = this._rpc({
                model: 'ir.ui.view',
                method: 'read_template',
                args: ['website.theme_customize'],
            }).then(function (data) {
                if (!/^<templates>/.test(data)) {
                    data = _.str.sprintf('<templates>%s</templates>', data);
                }
                return core.qweb.add_template(data);
            });
        }
        return Promise.all([this._super.apply(this, arguments), templateDef]);
    },
    /**
     * @override
     */
    start: function () {
        var self = this;

        this.PX_BY_REM = parseFloat($(document.documentElement).css('font-size'));

        this.$modal.addClass('o_theme_customize_modal');

        this.style = window.getComputedStyle(document.documentElement);
        this.nbFonts = parseInt(this.style.getPropertyValue('--number-of-fonts'));
        var googleFontsProperty = this.style.getPropertyValue('--google-fonts').trim();
        this.googleFonts = googleFontsProperty ? googleFontsProperty.split(/\s*,\s*/g) : [];

        var $tabs;
        var loadDef = this._loadViews().then(function (data) {
            self._generateDialogHTML(data);
            $tabs = self.$('[data-toggle="tab"]');

            // Hide the tab navigation if only one tab
            if ($tabs.length <= 1) {
                $tabs.closest('.nav').addClass('d-none');
            }
        });

        // Enable the first option tab or the given default tab
        this.opened().then(function () {
            // Hack to hide primary/secondary if they are equal to alpha/beta
            // (this is the case with default values but not in some themes).
            var $primary = self.$('.o_theme_customize_color[data-color="primary"]');
            var $alpha = self.$('.o_theme_customize_color[data-color="alpha"]');
            var $secondary = self.$('.o_theme_customize_color[data-color="secondary"]');
            var $beta = self.$('.o_theme_customize_color[data-color="beta"]');

            var sameAlphaPrimary = self.style.getPropertyValue('--is-alpha-primary').trim() == 'true';
            var sameBetaSecondary = self.style.getPropertyValue('--is-beta-secondary').trim() == 'true';

            if (!sameAlphaPrimary) {
                $alpha.prev().text(_t("Extra Color"));
            }
            if (!sameBetaSecondary) {
                $beta.prev().text(_t("Extra Color"));
            }

            $primary = $primary.closest('.o_theme_customize_option');
            $alpha = $alpha.closest('.o_theme_customize_option');
            $secondary = $secondary.closest('.o_theme_customize_option');
            $beta = $beta.closest('.o_theme_customize_option');

            $primary.toggleClass('d-none', sameAlphaPrimary);
            $secondary.toggleClass('d-none', sameBetaSecondary);

            if (!sameAlphaPrimary && sameBetaSecondary) {
                $beta.insertBefore($alpha);
            } else if (sameAlphaPrimary && !sameBetaSecondary) {
                $secondary.insertAfter($alpha);
            }

            $alpha.tooltip({title: _t('Changing this color will regenerate the default theme color scheme'), delay: { "show": 100, "hide": 100 }, container: 'body'});
            $tabs.eq(self.defaultTab).tab('show');
        });

        return Promise.all([this._super.apply(this, arguments), loadDef]);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _chooseBodyCustomImage: function () {
        var self = this;
        var def = new Promise(function (resolve, reject) {
            var $image = $('<img/>');
            var editor = new weWidgets.MediaDialog(self, {
                mediaWidth: 1920,
                onlyImages: true,
            }, $image[0]);

            editor.on('save', self, function (media) { // TODO use scss customization instead (like for user colors)
                self._rpc({
                    model: 'ir.model.data',
                    method: 'get_object_reference',
                    args: ['website', self.CUSTOM_BODY_IMAGE_XML_ID],
                }).then(function (data) {
                    return self._rpc({
                        model: 'ir.ui.view',
                        method: 'save',
                        args: [
                            data[1],
                            '#wrapwrap { background-image: url("' + media.src + '"); }',
                            '//style',
                        ],
                    });
                }).then(resolve).guardedCatch(resolve);
            });
            editor.on('cancel', self, function () {
                resolve();
            });

            editor.open();
        });

        return def;
    },
    /**
     * @private
     * @param {Object} data - @see this._loadViews
     */
    _generateDialogHTML: function (data) {
        var self = this;
        var $contents = this.$el.children('content');
        if ($contents.length === 0) {
            return;
        }

        $contents.remove();
        this.$el.append(core.qweb.render('website.theme_customize_modal_layout'));
        var $navLinksContainer = this.$('.nav');
        var $navContents = this.$('.tab-content');

        _.each($contents, function (content) {
            var $content = $(content);

            var contentID = _.uniqueId('content-');

            // Build the nav tab for the content
            $navLinksContainer.append($('<li/>', {
                class: 'nav-item mb-1',
            }).append($('<a/>', {
                href: '#' + contentID,
                class: 'nav-link',
                'data-toggle': 'tab',
                text: $content.attr('string'),
            })));

            // Build the tab pane for the content
            var $navContent = $(core.qweb.render('website.theme_customize_modal_content', {
                id: contentID,
                title: $content.attr('title'),
            }));
            $navContents.append($navContent);
            var $optionsContainer = $navContent.find('.o_options_container');

            // Process content items
            _processItems($content.children(), $optionsContainer, false);
        });

        this.$('[title]').tooltip();

        this.$inputs = self.$('.o_theme_customize_option_input');
        // Enable data-xmlid="" inputs if none of their neighbors were enabled
        _.each(this.$inputs.filter('[data-xmlid=""]'), function (input) {
            var $input = $(input);
            var $neighbors = self.$inputs.filter('[name="' + $input.attr('name') + '"]').not($input);
            if ($neighbors.length && !$neighbors.filter(':checked').length) {
                $input.prop('checked', true);
            }
        });
        this._setActive();
        this._updateValues();

        function _processItems($items, $container, isSelectionContainer) {
            var optionsName = _.uniqueId('option-');
            var alone = ($items.length === 1);

            _.each($items, function (item) {
                var $item = $(item);
                var $element;

                switch (item.tagName) {
                    case 'OPT':
                        var widgetName = $item.data('widget');

                        var xmlid = $item.data('xmlid');

                        var renderingOptions = _.extend({
                            string: $item.attr('string') || xmlid && data.names[xmlid.split(',')[0].trim()],
                            icon: $item.data('icon'),
                            font: $item.data('font'),
                        }, $item.data());

                        var checked;
                        if (widgetName === 'auto') {
                            var propValue = self.style.getPropertyValue('--' + $item.data('variable')).trim();
                            checked = (propValue === $item.attr('data-value'));
                        } else {
                            checked = (xmlid === undefined || xmlid && !_.difference(self._getXMLIDs($item), data.enabled).length);
                        }

                        // Build the options template
                        $element = $(core.qweb.render('website.theme_customize_modal_option', _.extend({
                            alone: alone,
                            name: xmlid === undefined && widgetName !== 'auto' ? _.uniqueId('option-') : optionsName,
                            id: $item.attr('id') || _.uniqueId('o_theme_customize_input_id_'),
                            checked: checked,
                            widget: widgetName,
                        }, renderingOptions)));
                        $element.find('input')
                            .addClass('o_theme_customize_option_input')
                            .attr({
                                'data-xmlid': xmlid,
                                'data-enable': $item.data('enable'),
                                'data-disable': $item.data('disable'),
                                'data-reload': $item.data('reload'),
                            });

                        if (widgetName) {
                            var $widget = $(core.qweb.render('website.theme_customize_widget_' + widgetName, renderingOptions));
                            $element.find('label').append($widget);
                        }

                        if (isSelectionContainer) {
                            $element.removeClass("my-1 flex-grow-0").addClass("dropdown-item p-0");
                            $element.find('label')
                                .addClass('justify-content-start')
                                .attr('data-font-id', $item.data('font'));
                        }
                        break;

                    case 'LIST':
                        $element = $('<div/>', {class: 'py-1 px-2 o_theme_customize_option_list'});
                        _processItems($item.children(), $element, false);
                        break;

                    case 'SELECTION':
                        $element = $(core.qweb.render('website.theme_customize_dropdown_option'));
                        _processItems($item.children(), $element.find('.o_theme_customize_selection'), true);
                        break;

                    case 'FONTSELECTION':
                        var $options = $();
                        var variable = $item.data('variable');
                        self.fontVariables.push(variable);
                        _.times(self.nbFonts, function (font) {
                            $options = $options.add($('<opt/>', {
                                'data-widget': 'auto',
                                'data-variable': variable,
                                'data-value': font + 1,
                                'data-font': font + 1,
                            }));
                        });
                        $element = $(core.qweb.render('website.theme_customize_dropdown_option'));
                        var $selection = $element.find('.o_theme_customize_selection');
                        _processItems($options, $selection, true);

                        if (self.googleFonts.length) {
                            var $googleFontItems = $selection.children().slice(-self.googleFonts.length);
                            _.each($googleFontItems, function (el, index) {
                                $(el).append(core.qweb.render('website.theme_customize_delete_font', {
                                    'index': index,
                                }));
                            });
                        }
                        $selection.append($(core.qweb.render('website.theme_customize_add_google_font_option', {
                            'variable': variable,
                        })));
                        break;

                    default:
                        _processItems($item.children(), $container, false);
                        return;
                }

                if ($container.hasClass('form-row')) {
                    var $col = $('<div/>', {
                        class: _.str.sprintf('col-%s', $item.data('col') || 6),
                    });

                    if (item.tagName === 'LIST') {
                        $col.addClass('mt-2');
                        $col.append($('<h6/>', {text: $item.attr('string')}));
                    }

                    $col.append($element);
                    $element = $col;
                }

                $element.attr('data-depends', $item.data('depends'));
                $container.append($element);
            });
        }
    },
    /**
     * @private
     */
    _loadViews: function () {
        return this._rpc({
            route: '/website/theme_customize_get',
            params: {
                'xml_ids': this._getXMLIDs(this.$inputs || this.$('[data-xmlid]')),
            },
        });
    },
    /**
     * @private
     */
    _getInputs: function (string) {
        if (!string) {
            return $();
        }
        return this.$inputs.filter('#' + string.replace(/\s*,\s*/g, ', #'));
    },
    /**
     * @private
     */
    _getXMLIDs: function ($inputs) {
        var xmlIDs = [];
        _.each($inputs, function (input) {
            var $input = $(input);
            var xmlID = $input.data('xmlid');
            if (xmlID) {
                xmlIDs = xmlIDs.concat(xmlID.split(/\s*,\s*/));
            }
        });
        return xmlIDs;
    },
    /**
     * @private
     * @param {object} [values]
     *        When a new set of google fonts are saved, other variables
     *        potentially have to be adapted.
     */
    _makeGoogleFontsCusto: function (values) {
        values = values ? _.clone(values) : {};
        if (this.googleFonts.length) {
            values['google-fonts'] = "('" + this.googleFonts.join("', '") + "')";
        } else {
            values['google-fonts'] = 'null';
        }
        return this._makeSCSSCusto('/website/static/src/scss/options/user_values.scss', values).then(function () {
            window.location.hash = 'theme=true';
            window.location.reload();
        });
    },
    /**
     * @private
     */
    _makeSCSSCusto: function (url, values) {
        return this._rpc({
            route: '/website/make_scss_custo',
            params: {
                'url': url,
                'values': values,
            },
        });
    },
    /**
     * @private
     */
    _pickColor: function (colorElement) {
        var self = this;
        var $color = $(colorElement);
        var colorName = $color.data('color');
        var colorType = $color.data('colorType');

        return new Promise(function (resolve, reject) {
            var colorpicker = new ColorpickerDialog(self, {
                defaultColor: $color.css('background-color'),
            });
            var chosenColor = undefined;
            colorpicker.on('colorpicker:saved', self, function (ev) {
                ev.stopPropagation();
                chosenColor = ev.data.cssColor;
            });
            colorpicker.on('closed', self, function (ev) {
                if (chosenColor === undefined) {
                    resolve();
                    return;
                }

                var baseURL = '/website/static/src/scss/options/colors/';
                var url = _.str.sprintf('%suser_%scolor_palette.scss', baseURL, (colorType ? (colorType + '_') : ''));

                var colors = {};
                colors[colorName] = chosenColor;
                if (colorName === 'alpha') {
                    colors['beta'] = 'null';
                    colors['gamma'] = 'null';
                    colors['delta'] = 'null';
                    colors['epsilon'] = 'null';
                }

                self._makeSCSSCusto(url, colors).then(resolve).guardedCatch(resolve);
            });
            colorpicker.open();
        });
    },
    /**
     * @private
     */
    _processChange: function ($inputs) {
        var self = this;
        var defs = [];

        var $options = $inputs.closest('.o_theme_customize_option');

        // Handle body image changes
        var $bodyImageInputs = $inputs.filter('[data-xmlid*="website.' + this.CUSTOM_BODY_IMAGE_XML_ID + '"]:checked');
        defs = defs.concat(_.map($bodyImageInputs, function () {
            return self._chooseBodyCustomImage();
        }));

        // Handle color changes
        var $colors = $options.find('.o_theme_customize_color');
        defs = defs.concat(_.map($colors, function (colorElement) {
            return self._pickColor($(colorElement));
        }));

        // Handle input changes
        var $inputsData = $options.find('.o_theme_customize_input');
        defs = defs.concat(_.map($inputsData, function (inputData, i) {
            return self._quickEdit($(inputData));
        }));

        // Handle auto changes
        var $autoWidgetOptions = $options.has('.o_theme_customize_auto');
        if ($autoWidgetOptions.length > 1) {
            $autoWidgetOptions = $autoWidgetOptions.has('input:checked');
        }
        var $autosData = $autoWidgetOptions.find('.o_theme_customize_auto');
        defs = defs.concat(_.map($autosData, function (autoData) {
            return self._setAuto($(autoData));
        }));

        return Promise.all(defs);
    },
    /**
     * @private
     */
    _quickEdit: function ($inputData) {
        var self = this;
        var text = $inputData.text().trim();
        var value = parseFloat(text) || '';
        var unit = (text.match(/[^\s\d]+$/) || ['px'])[0];

        return new Promise(function (resolve, reject) {
            var qEdit = new QuickEdit(self, value, unit);
            qEdit.on('QuickEdit:save', self, function (ev) {
                ev.stopPropagation();

                var value = ev.data.value;
                // Convert back to rem if needed
                if ($inputData.data('unit') === 'rem' && unit === 'px' && value !== 'null') {
                    value = parseFloat(value) / self.PX_BY_REM + 'rem';
                }

                var values = {};
                values[$inputData.data('variable')] = value;
                self._makeSCSSCusto('/website/static/src/scss/options/user_values.scss', values)
                    .then(resolve)
                    .guardedCatch(resolve);
            });
            qEdit.appendTo($inputData.closest('.o_theme_customize_option'));
        });
    },
    /**
     * @private
     */
    _setAuto: function ($autoData) {
        var self = this;
        var values = {};
        var isChecked = $autoData.siblings('.o_theme_customize_option_input').prop('checked');
        values[$autoData.data('variable')] = isChecked ? $autoData.data('value') : 'null';

        return new Promise(function (resolve, reject) {
            self._makeSCSSCusto('/website/static/src/scss/options/user_values.scss', values)
                .then(resolve)
                .guardedCatch(resolve);
        });
    },
    /**
     * @private
     */
    _setActive: function () {
        var self = this;

        // First enforce that all input groups have only one element checked as
        // it is supposed to be (it might not be the case on initialization, for
        // exemple if we had data-xmlid="A" and data-xmlid="A,B" and if A and B
        // are active, the 2 related inputs would be checked).
        var $radioXMLInputs = this.$inputs.filter('[type="radio"][data-xmlid]');
        var optionNames = _.uniq(_.map($radioXMLInputs, function (option) {
            return option.name;
        }));
        _.each(optionNames, function (optionName) {
            var $inputs = $radioXMLInputs.filter('[name="' + optionName + '"]:checked');
            if ($inputs.length > 1) {
                $inputs.prop('checked', false);

                var maxNbXMLIDs = -1;
                var $maxInput = null;
                _.each($inputs, function (input) {
                    var $input = $(input);
                    var xmlID = $input.data('xmlid');
                    var nbXMLIDs = xmlID ? xmlID.split(',').length : 0;
                    if (nbXMLIDs >= maxNbXMLIDs) {
                        maxNbXMLIDs = nbXMLIDs;
                        $maxInput = $input;
                    }
                });
                $maxInput.prop('checked', true);
            }
        });

        // Look at all options to see if they are enabled or disabled
        var $enable = this.$inputs.filter(':checked');

        // Mark the labels as checked accordingly
        this.$('label').removeClass('checked');
        $enable.closest('label:not(.o_switch)').addClass('checked');

        // Mark the option sets as checked if all their option are checked/unchecked
        var $sets = this.$inputs.filter('[data-enable], [data-disable]').not('[data-xmlid]');
        _.each($sets, function (set) {
            var $set = $(set);
            var checked = true;
            if (self._getInputs($set.data('enable')).not(':checked').length) {
                checked = false;
            }
            if (self._getInputs($set.data('disable')).filter(':checked').length) {
                checked = false;
            }
            $set.prop('checked', checked).closest('label:not(.o_switch)').toggleClass('checked', checked);
        });

        // Make the hidden sections visible if their dependencies are met
        _.each(this.$('[data-depends]'), function (hidden) {
            var $hidden = $(hidden);
            var depends = $hidden.data('depends');
            var dependencies = depends ? depends.split(/\s*,\s*/g) : [];
            var enabled = _.all(dependencies, function (dep) {
                var toBeChecked = (dep[0] !== '!');
                if (!toBeChecked) {
                    dep = dep.substr(1);
                }
                return self._getInputs(dep).is(':checked') === toBeChecked;
            });
            $hidden.toggleClass('d-none', !enabled);
        });
    },
    /**
     * @private
     */
    _updateStyle: function (enable, disable, reload) {
        var self = this;

        var $loading = $('<i/>', {class: 'fa fa-refresh fa-spin'});
        this.$modal.find('.modal-title').append($loading);

        if (reload || config.isDebug('assets')) {
            window.location.href = $.param.querystring('/website/theme_customize_reload', {
                href: window.location.href,
                enable: (enable || []).join(','),
                disable: (disable || []).join(','),
                tab: this.$('.nav-link.active').parent().index(),
            });
            return Promise.resolve();
        }

        return this._rpc({
            route: '/website/theme_customize',
            params: {
                'enable': enable,
                'disable': disable,
                'get_bundle': true,
            },
        }).then(function (bundles) {
            var $allLinks = $();
            var defs = _.map(bundles, function (bundleURLs, bundleName) {
                var $links = $('link[href*="' + bundleName + '"]');
                $allLinks = $allLinks.add($links);
                var $newLinks = $();
                _.each(bundleURLs, function (url) {
                    $newLinks = $newLinks.add($('<link/>', {
                        type: 'text/css',
                        rel: 'stylesheet',
                        href: url,
                    }));
                });

                var linksLoaded = new Promise(function (resolve, reject) {
                    var nbLoaded = 0;
                    $newLinks.on('load', function () {
                        if (++nbLoaded >= $newLinks.length) {
                            resolve();
                        }
                    });
                    $newLinks.on('error', function () {
                        reject();
                        window.location.hash = 'theme=true';
                        window.location.reload();
                    });
                });
                $links.last().after($newLinks);
                return linksLoaded;
            });
            return Promise.all(defs).then(function () {
                $loading.remove();
                $allLinks.remove();
            }).guardedCatch(function () {
                $loading.remove();
                $allLinks.remove();
            });
        }).then(function () {
            // Some public widgets may depend on the variables that were
            // customized, so we have to restart them.
            self.trigger_up('widgets_start_request');
        });
    },
    /**
     * @private
     */
    _updateValues: function () {
        var self = this;
        // Put user values
        _.each(this.$('.o_theme_customize_color'), function (el) {
            var $el = $(el);
            var value = self.style.getPropertyValue('--' + $el.data('color')).trim();
            $el.css('background-color', value);
        });
        _.each(this.$('.o_theme_customize_input'), function (el) {
            var $el = $(el);
            var value = self.style.getPropertyValue('--' + $el.data('variable')).trim();

            // Convert rem values to px values
            if (_.str.endsWith(value, 'rem')) {
                value = parseFloat(value) * self.PX_BY_REM + 'px';
            }

            var $span = $el.find('span');
            $span.removeClass().text('');
            switch (value) {
                case '':
                case 'false':
                case 'true':
                    // When null or a boolean value, shows an icon which tells
                    // the user that there is no numeric/text value
                    $span.addClass('fa fa-ban text-danger');
                    break;
                default:
                    $span.text(value);
            }
        });
        _.each(this.$('.o_theme_customize_dropdown'), function (dropdown) {
            var $dropdown = $(dropdown);
            $dropdown.find('.dropdown-item.active').removeClass('active');
            var $checked = $dropdown.find('label.checked');
            $checked.closest('.dropdown-item').addClass('active');

            var classes = 'btn btn-light dropdown-toggle w-100 o_text_overflow o_theme_customize_dropdown_btn';
            if ($checked.data('font-id')) {
                classes += _.str.sprintf(' o_theme_customize_option_font_%s', $checked.data('font-id'));
            }
            var $btn = $('<button/>', {
                type: 'button',
                class: classes,
                'data-toggle': 'dropdown',
                html: $dropdown.find('label.checked > span').text() || '&#8203;',
            });
            $dropdown.find('.o_theme_customize_dropdown_btn').remove();
            $dropdown.prepend($btn);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onChange: function (ev) {
        var self = this;

        // Checkout the option that changed
        var $option = $(ev.currentTarget);
        if ($option.is(':disabled')) {
            return;
        }
        this.$inputs.prop('disabled', true);

        var $options = $option;
        var checked = $option.is(':checked');

        // If it was enabled, enable/disable the related input (see data-enable,
        // data-disable) and retain the ones that actually changed
        if (checked) {
            var $inputs;
            // Input to enable
            $inputs = this._getInputs($option.data('enable'));
            $options = $options.add($inputs.filter(':not(:checked)'));
            $inputs.prop('checked', true);
            // Input to disable
            $inputs = this._getInputs($option.data('disable'));
            $options = $options.add($inputs.filter(':checked'));
            $inputs.prop('checked', false);
        }
        var optionNames = _.uniq(_.map($options, function (option) {
            return option.name;
        }));
        $options = this.$inputs.filter(function (i, input) {
            return _.contains(optionNames, input.name);
        });

        // Look at all options to see if they are enabled or disabled
        var $enable = $options.filter('[data-xmlid]:checked');
        var $disable = $options.filter('[data-xmlid]:not(:checked)');

        this._setActive();

        // Update the style according to the whole set of options
        self._processChange($options).then(function () {
            return self._updateStyle(
                self._getXMLIDs($enable),
                self._getXMLIDs($disable),
                $option.data('reload') && window.location.href.match(new RegExp($option.data('reload')))
            );
        }).then(function () {
            self._updateValues();
            self.$inputs.prop('disabled', false);
        });
    },
    /**
     * @private
     */
    _onAddGoogleFontClick: function (ev) {
        var self = this;
        var variable = $(ev.currentTarget).data('variable');
        new Dialog(this, {
            title: _t("Add a Google Font"),
            $content: $(core.qweb.render('website.dialog.addGoogleFont')),
            buttons: [
                {
                    text: _t("Save"),
                    classes: 'btn-primary',
                    click: function () {
                        var $input = this.$('.o_input_google_font');
                        var m = $input.val().match(/\bfamily=([\w+]+)/);
                        if (!m) {
                            $input.addClass('is-invalid');
                            return;
                        }
                        var font = m[1].replace(/\+/g, ' ');
                        self.googleFonts.push(font);
                        var values = {};
                        values[variable] = self.nbFonts + 1;
                        return self._makeGoogleFontsCusto(values);
                    },
                },
                {
                    text: _t("Discard"),
                    close: true,
                },
            ],
        }).open();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onDeleteGoogleFontClick: function (ev) {
        var self = this;
        ev.preventDefault();

        var nbBaseFonts = this.nbFonts - this.googleFonts.length;

        // Remove Google font
        var googleFontIndex = $(ev.currentTarget).data('fontIndex');
        this.googleFonts.splice(googleFontIndex, 1);

        // Adapt font variable indexes to the removal
        var values = {};
        _.each(this.fontVariables, function (variable) {
            var value = parseInt(self.style.getPropertyValue('--' + variable));
            var googleFontValue = nbBaseFonts + 1 + googleFontIndex;
            if (value === googleFontValue) {
                // If an element is using the google font being removed, reset
                // it to the first base font.
                values[variable] = 1;
            } else if (value > googleFontValue) {
                // If an element is using a google font whose index is higher
                // than the one of the font being removed, that index must be
                // lowered by 1 so that the font is unchanged.
                values[variable] = value - 1;
            }
        });

        return this._makeGoogleFontsCusto(values);
    },
});

var ThemeCustomizeMenu = websiteNavbarData.WebsiteNavbarActionWidget.extend({
    actions: _.extend({}, websiteNavbarData.WebsiteNavbarActionWidget.prototype.actions || {}, {
        'customize_theme': '_openThemeCustomizeDialog',
    }),

    /**
     * Automatically opens the theme customization dialog if the corresponding
     * hash is in the page URL.
     *
     * @override
     */
    start: function () {
        if ((window.location.hash || '').indexOf('theme=true') > 0) {
            var tab = window.location.hash.match(/tab=(\d+)/);
            this._openThemeCustomizeDialog(tab ? tab[1] : false);
            window.location.hash = '';
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Actions
    //--------------------------------------------------------------------------

    /**
     * Instantiates and opens the theme customization dialog.
     *
     * @private
     * @param {string} tab
     * @returns {Promise}
     */
    _openThemeCustomizeDialog: function (tab) {
        return new ThemeCustomizeDialog(this, {tab: tab}).open();
    },
});

websiteNavbarData.websiteNavbarRegistry.add(ThemeCustomizeMenu, '#theme_customize');

return ThemeCustomizeDialog;
});

```

## File: static\src\xml\track_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

	<t t-name="website.track_page">
		<div role="separator" class="dropdown-divider"/>
		<a href="#" name="switch-track-page" class="dropdown-item" role="menuitem">
			<label class="o_switch" for="switch-track-page">
				<input id="switch-track-page" type="checkbox"/>
				<span/>
				Track visitor
			</label>
		</a>
	</t>

</templates>

```

## File: static\src\xml\translator.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
<div t-name="website.TranslatorInfoDialog">
    <p>You are about to enter the translation mode.</p>
    <p>Here are the visuals used to help you translate efficiently:</p>
    <ul class="oe_translate_examples">
        <li data-oe-translation-state="to_translate">Content to translate</li>
        <li data-oe-translation-state="translated">Translated content</li>
    </ul>
    <p>
        In this mode, you can only translate texts. To change the structure of the page, you must edit the master page.
        Each modification on the master page is automatically applied to all translated versions.
    </p>
</div>
</templates>

```

## File: static\src\xml\website.backend.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="WidgetWebsiteButton" class="o_stat_info published">
        <span class="o_stat_text o_value"/>
        <span class="o_stat_text">On Website</span>
    </div>

    <t t-name="WidgetWebsiteButtonIcon">
        <button type="button" class="btn oe_stat_button">
            <i class="fa fa-fw o_button_icon fa-globe"/>
            <div class="o_stat_info">
                <span class="o_stat_text">Go to<br/>Website</span>
            </div>
        </button>
    </t>

    <t t-name="website.WebsiteDashboardMain">
        <div class="o_dashboards">
            <div class="container-fluid o_website_dashboard">
                <t t-call="website.dashboard_header"/>
                <t t-call="website.dashboard_content"/>
            </div>
        </div>
    </t>

    <t t-name="website.dashboard_header">
        <div class="row o_dashboard_common"/>
    </t>

    <t t-name="website.dashboard_content">
        <div class="o_website_dashboard_content">
            <t t-call="website.google_analytics_content"/>
        </div>
    </t>
    <t t-name="website.google_analytics_content">
        <div class="row o_dashboard_visits" t-if="widget.groups.website_designer">
            <div class="col-12 o_box">
                <h2>Visits</h2>
                <div t-if="widget.dashboards_data.visits &amp;&amp; widget.dashboards_data.visits.ga_client_id">
                    <div class="row js_analytics_components"/>
                    <a href="#" class="js_link_analytics_settings">Edit my Analytics Client ID</a>
                </div>
                <div t-if="!(widget.dashboards_data.visits &amp;&amp; widget.dashboards_data.visits.ga_client_id)" class="col-lg-12">
                    <div class="o_demo_background">
                        <div class="o_layer">
                        </div>
                        <div class="o_buttons text-center">
                            <h3>There is no data currently available.</h3>
                            <button class="btn btn-primary js_link_analytics_settings d-block mx-auto mb8">Connect Google Analytics</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </t>

    <div t-name="website.unauthorized_analytics" class="col-12 js_unauthorized_message mb16">
        <span t-if="reason === 'not_connected'">You need to log in to your Google Account before: </span>
        <span t-if="reason === 'no_right'">You do not seem to have access to this Analytics Account.</span>
        <p t-if="reason === 'not_initialized'">
            Google deprecated both its "Universal Analytics" and "Google Sign-In" API. It means that only accounts and keys created before 2020 will be able to integrate their Analytics dashboard in Odoo (or any other website). This will be possible only up to mid 2023. After that, those services won't work anymore, at all.<br />
            New Google Analytics accounts and keys are now using Google Analytics 4 which, for now, can't be integrated/embed in external websites.<br />
            Those accounts should now check their Analytics dashboard in the Google platform directly.
        </p>
        <span t-if="reason === 'not_initialized'">Google Analytics initialization failed. Maybe this domain is not whitelisted in your Google Analytics project for this client ID.</span>
    </div>

    <div t-name="website.ga_dialog_content">
        Your Tracking ID: <input type="text" name="ga_analytics_key" placeholder="UA-XXXXXXXX-Y" t-att-value="ga_analytics_key" style="width: 100%"></input>
        <a href="https://www.odoo.com/documentation/13.0/applications/websites/website/optimize/google_analytics.html" target="_blank">
            <i class="fa fa-arrow-right"/>
            How to get my Tracking ID
        </a>
        <br/><br/>
        Your Client ID: <input type="text" name="ga_client_id" t-att-value="ga_key" style="width: 100%"></input>
        <a href="https://www.odoo.com/documentation/13.0/applications/websites/website/optimize/google_analytics_dashboard.html" target="_blank">
            <i class="fa fa-arrow-right"/>
            How to get my Client ID
        </a>
    </div>

    <t t-name="website.DateRangeButtons">
        <!-- TODO: Hide in mobile as it is going to push in control panel and it breaks UI, maybe we will improve it in future -->
        <div class="btn-group o_date_range_buttons d-none d-md-inline-flex float-right">
            <button class="btn btn-secondary js_date_range active" data-date="week">Last Week</button>
            <button class="btn btn-secondary js_date_range" data-date="month">Last Month</button>
            <button class="btn btn-secondary js_date_range" data-date="year">Last Year</button>
        </div>
        <div class="btn-group d-none d-md-inline-block float-right" style="margin-right: 20px;">
            <t t-foreach="widget.websites" t-as="website">
                <button t-attf-class="btn btn-secondary js_website #{website.selected ? 'active' : ''}"
                        t-att-data-website-id="website.id">
                    <t t-esc="website.name"/>
                </button>
            </t>
        </div>
    </t>

    <t t-name="website.GoToButtons">
        <a role="button" href="/" class="btn btn-primary" title="Go to Website">
            Go to Website
        </a>
    </t>

</templates>

```

## File: static\src\xml\website.background.video.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="website.background.video">
        <div class="o_bg_video_container">
            <div class="o_bg_video_loading d-flex justify-content-center align-items-center text-primary">
                <div class="spinner-border" style="width: 4em; height: 4em;" role="status">
                    <span class="sr-only">Loading...</span>
                </div>
            </div>
            <iframe t-att-id="iframeID"
                    class="o_bg_video_iframe fade"
                    frameBorder="0"
                    t-att-src="videoSrc"/>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website.contentMenu.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
<t t-name="website.contentMenu.dialog.submenu">
    <li t-att-data-menu-id="submenu.fields['id']" t-att-data-mega-menu="submenu.fields['is_mega_menu'] ? true : undefined">
        <div class="input-group">
            <div class="input-group-prepend">
                <span class="input-group-text fa fa-bars" role="img" aria-label="Dropdown menu" title="Dropdown menu"/>
            </div>
            <span class="form-control d-flex align-items-center">
                <span class="js_menu_label o_text_overflow flex-grow-1">
                    <t t-esc="submenu.fields['name']"/>
                </span>
                <span t-if="submenu.fields['is_mega_menu']" class="badge badge-primary">Mega Menu</span>
                <i t-if="submenu.is_homepage" class="fa fa-home ml-3" role="img" aria-label="Home" title="Home"/>
            </span>
            <span class="input-group-append">
                <button type="button" class="btn btn-primary js_edit_menu fa fa-pencil-square-o" aria-label="Edit Menu Item" title="Edit Menu Item"/>
                <button type="button" class="btn btn-danger js_delete_menu fa fa-trash-o" aria-label="Delete Menu Item" title="Delete Menu Item"/>
            </span>
        </div>
        <t t-set="children" t-value="submenu.children"/>
        <ul t-if="children">
            <t t-foreach="children" t-as="submenu">
                <t t-call="website.contentMenu.dialog.submenu"/>
            </t>
        </ul>
    </li>
</t>
<div t-name="website.contentMenu.dialog.select">
    <select class="form-control mb16" t-if="widget.roots">
        <t t-foreach="widget.roots" t-as="root">
            <option t-att-value="root.id"><t t-esc="root.name"/></option>
        </t>
    </select>
</div>
<div t-name="website.contentMenu.dialog.edit">
    <select class="form-control mb16" t-if="widget.roots">
        <t t-foreach="widget.roots" t-as="root">
            <option t-att-value="root.id"><t t-esc="root.name"/></option>
        </t>
    </select>
    <ul class="oe_menu_editor list-unstyled">
        <t t-foreach="widget.menu.children" t-as="submenu">
            <t t-call="website.contentMenu.dialog.submenu"/>
        </t>
    </ul>
    <div class="mt32">
        <small class="float-right text-muted">
            Drag to the right to get a submenu
        </small>
        <a href="#" class="js_add_menu">
            <i class="fa fa-plus-circle"/> Add Menu Item
        </a><br/>
        <a href="#" class="js_add_menu" data-type="mega">
            <i class="fa fa-plus-circle"/> Add Mega Menu Item
        </a>
    </div>
</div>
</templates>

```

## File: static\src\xml\website.editor.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="website.homepage_editor_welcome_message" class="container text-center o_homepage_editor_welcome_message">
        <h2 class="mt0">Welcome to your <b>Homepage</b>!</h2>
        <p class="lead d-none d-md-block">Let's start designing.</p>
        <div class="o_tooltip_container d-none d-md-inline-flex fade">Follow all the <div class="o_tooltip bottom"/> signs to get your website ready in no time.</div>
    </div>
    <div t-name="website.leaving_current_page_edition">
        <p>What do you want to do?</p>
        <p class="text-muted">Your current changes will be saved automatically.</p>
    </div>

    <!-- Editor top bar which contains the summernote tools and save/discard buttons -->
    <t t-name="website.editorbar">
        <div id="web_editor-top-edit">
            <form class="navbar-form text-muted">
                <button type="button" class="btn btn-secondary" data-action="cancel"><i class="fa fa-times"/> Discard</button>
                <button type="button" class="btn btn-primary" data-action="save"><i class="fa fa-floppy-o"/> Save</button>
            </form>
        </div>
    </t>
    <!-- Custom checkbox (material-design-like toggle) -->
    <t t-name="website.components.switch">
        <label class="o_switch" t-att-for="id">
            <input type="checkbox" t-att-id="id" t-att-checked="checked ? 'checked' : undefined"/>
            <span/>
            <div t-if="label"><t t-esc="label"/></div>
        </label>
    </t>

    <div t-name="website.theme_customize_modal_layout" class="d-flex align-items-start">
        <ul class="nav flex-column flex-shrink-0 w-25"/>
        <div class="tab-content pl-3 pb-2 flex-grow-1"/>
    </div>
    <div t-name="website.theme_customize_modal_content" t-att-id="id" class="tab-pane">
        <h5 class="mt-0"><t t-esc="title"/></h5>
        <div class="form-row justify-content-between o_options_container"/>
    </div>
    <t t-name="website.theme_customize_modal_option">
        <div t-attf-class="o_theme_customize_option my-1 flex-grow-0 #{font ? 'o_theme_customize_option_font_' + font : ''} #{widget and widget != 'auto' ? 'o_theme_customize_with_widget' : ''}">
            <img t-if="icon" t-att-src="icon"/>

            <t t-set="label" t-value="font ? '' : string"/>
            <t t-if="alone and (!widget or widget == 'auto')" t-call="website.components.switch"/>
            <label t-else="">
                <input t-att-id="id" t-att-name="name" type="radio" t-att-checked="checked ? 'checked' : undefined"/>
                <span><t t-esc="label"/></span>
            </label>
        </div>
    </t>
    <t t-name="website.theme_customize_delete_font">
        <t t-set="delete_font_title">Delete this font</t>
        <button type="button"
            class="btn btn-link d-flex align-items-center text-danger fa fa-trash-o o_theme_customize_delete_google_font"
            t-att-aria-label="delete_font_title"
            t-att-title="delete_font_title"
            t-att-data-font-index="index"/>
    </t>
    <t t-name="website.theme_customize_add_google_font_option">
        <a class="dropdown-item p-2 o_theme_customize_add_google_font" t-att-data-variable="variable" href="#">
            <i class="fa fa-plus"/> Add a Google Font
        </a>
    </t>
    <t t-name="website.theme_customize_dropdown_option">
        <div t-attf-class="dropdown o_theme_customize_dropdown">
            <div class="dropdown-menu rounded-0 o_theme_customize_selection" role="menu"/>
        </div>
    </t>
    <t t-name="website.theme_customize_widget_color">
        <div t-attf-class="o_theme_customize_color o_theme_customize_color_#{color}"
            t-att-data-color="color"
            t-att-data-color-type="colorType"/>
    </t>
    <t t-name="website.theme_customize_widget_input">
        <div class="o_theme_customize_input"
            t-att-data-variable="variable"
            t-att-data-unit="unit">
            <i class="fa fa-edit"/>
            <span/>
        </div>
    </t>
    <t t-name="website.theme_customize_widget_auto">
        <div class="o_theme_customize_auto"
            t-att-data-variable="variable"
            t-att-data-value="value"/>
    </t>
    <t t-name="website.theme_customize_active_input">
        <div class="input-group input-group-sm align-items-center o_theme_customize_active_input">
            <input type="text" class="form-control" t-att-value="widget.value"/>
            <div class="input-group-append">
                <div t-if="widget.unit" class="input-group-text"><t t-esc="widget.unit"/></div>
                <button type="button" class="btn btn-secondary fa fa-undo" title="Reset"/>
            </div>
        </div>
    </t>

    <t t-extend="wysiwyg.widgets.link">
        <t t-jquery="#o_link_dialog_url_input" t-operation="after">
            <small class="form-text text-muted">Hint: Type '/' to search an existing page and '#' to link to an anchor.</small>
        </t>
        <t t-jquery="div.o_url_input" t-operation="after">
            <div class="form-group row o_link_dialog_page_anchor d-none">
                <label class="col-form-label col-md-3" for="o_link_dialog_anchor_input">Page Anchor</label>
                <div class="col-md-9">
                    <select name="link_anchor" class="form-control link-style"></select>
                    <small class="form-text font-weight-bold o_anchors_loading">Loading...</small>
                </div>
            </div>
        </t>
    </t>
    <!-- Anchor Name option dialog -->
    <div t-name="website.dialog.anchorName">
        <div class="form-group row">
            <label class="col-form-label col-md-3" for="anchorName">Choose an anchor name</label>
            <div class="col-md-9">
                <input type="text" class="form-control o_input_anchor_name" id="anchorName" t-attf-value="#{currentAnchor}" placeholder="Anchor name"/>
                <div class="invalid-feedback">
                    <p class="d-none o_anchor_not_valid">The chosen name is not valid (use only a-Z A-Z 0-9 - _)</p>
                    <p class="d-none o_anchor_already_exists">The chosen name already exists</p>
                </div>
            </div>
        </div>
    </div>
    <!-- Add a Google Font option dialog -->
    <div t-name="website.dialog.addGoogleFont">
        <div class="form-group row">
            <label class="col-form-label col-md-3" for="google_font_html">Google Font HTML</label>
            <div class="col-md-9">
                <textarea id="google_font_html" class="form-control o_input_google_font"
                    placeholder="&lt;link href='https://fonts.googleapis.com/css?family=Bonbon&amp;display=swap' rel='stylesheet'&gt;" style="height: 100px;"/>
                <span class="float-right text-muted">
                    Select one font on <a target="_blank" href="https://fonts.google.com">fonts.google.com</a> and copy paste the embed code here.
                </span>
            </div>
        </div>
    </div>
</templates>

```

## File: static\src\xml\website.facebook_page.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
<div t-name="website.facebook_page_dialog">
    <div class="row">
        <form class="col-lg-6">
            <div class="form-group form-row">
                <label class="col-form-label col-md-3">Page URL</label>
                <div class="col-md-9">
                    <input class="form-control o_facebook_page_url" required="required" t-att-value="widget.fbData.href"/>
                </div>
            </div>
            <div class="form-group form-row o_facebook_tabs">
                <t t-set="fb_tabs" t-value="widget.fbData.tabs and widget.fbData.tabs.split(',') or []"/>
                <label class="col-form-label col-md-3">Tabs</label>
                <div class="col-md-9">
                    <div class="custom-control custom-checkbox">
                        <input type="checkbox" id="o_facebook_page_tab_timeline_checkbox" class="custom-control-input" name="timeline" t-att-checked="_.contains(fb_tabs, 'timeline') or None"/>
                        <label class="custom-control-label" for="o_facebook_page_tab_timeline_checkbox">Timeline</label>
                    </div>
                    <div class="custom-control custom-checkbox">
                        <input type="checkbox" id="o_facebook_page_tab_events_checkbox" class="custom-control-input" name="events" t-att-checked="_.contains(fb_tabs, 'events') or None"/>
                        <label class="custom-control-label" for="o_facebook_page_tab_events_checkbox">Events</label>
                    </div>
                    <div class="custom-control custom-checkbox">
                        <input type="checkbox" id="o_facebook_page_tab_messages_checkbox" class="custom-control-input" name="messages" t-att-checked="_.contains(fb_tabs, 'messages') or None"/>
                        <label class="custom-control-label" for="o_facebook_page_tab_messages_checkbox">Messages</label>
                    </div>
                </div>
             </div>
            <div class="form-group form-row o_facebook_options">
                <label class="col-form-label col-md-3">Options</label>
                <div class="col-md-9 mt8">
                <label class="o_switch">
                    <input name="small_header" type="checkbox" t-att-checked="widget.fbData.small_header or None"/>
                    <span/>
                    Use Small Header
                </label>
                </div>
                <div class="offset-md-3 mt16 col-md-9">
                <label class="o_switch">
                    <input name="hide_cover" type="checkbox" t-att-checked="widget.fbData.hide_cover or None"/>
                    <span/>
                    Hide Cover Photo
                </label>
                </div>
                <!-- TODO: Remove this option in master (in the meantime we hide it). -->
                <div class="offset-md-3 mt16 col-md-9 d-none">
                <label class="o_switch">
                    <input name="show_facepile" type="checkbox" t-att-checked="widget.fbData.show_facepile or None"/>
                    <span/>
                    Show Friend's Faces
                </label>
                </div>
            </div>
        </form>
        <div class="col-lg-6" style="border-left: 1px solid #eeeeee;">
            <div class="form-group text-center">
                <label>Preview</label>
                <div class="alert alert-warning d-none facebook_page_warning text-center" role="alert">
                    <div>
                        <i class="fa fa-exclamation-triangle fa-3x" role="img" aria-label="Warning" title="Warning"></i>
                    </div>
                    <h4 class="mb0">Invalid Facebook Page Url</h4>
                    <div>Please enter valid facebook page URL for preview</div>
                </div>
                <div class="o_facebook_page o_facebook_preview"/>
            </div>
        </div>
    </div>
</div>
</templates>

```

## File: static\src\xml\website.gallery.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <!--
        ========================================================================
        Gallery Slideshow

        This template is used to display a slideshow of images inside a
        bootstrap carousel.

        ========================================================================
    -->
    <t t-name="website.gallery.slideshow">
        <div t-attf-id="#{id}" class="carousel slide" data-ride="carousel" t-attf-data-interval="#{interval}" style="margin: 0 12px;">
            <div class="carousel-inner" style="padding: 0;">
                 <t t-foreach="srcs" t-as="src">
                    <div t-attf-class="carousel-item #{src_index == index and 'active' or ''}">
                        <img t-attf-class="img img-fluid d-block #{userStyle}" t-att-src="src" alt="Slide image"/>
                    </div>
                 </t>
            </div>

            <ul class="carousel-indicators">
                <li class="o_indicators_left text-center" aria-label="Previous" title="Previous">
                    <i class="fa fa-chevron-left"/>
                </li>
                <t t-foreach="srcs" t-as="src">
                    <li t-attf-data-target="##{id}" t-att-data-slide-to="src_index" t-att-class="src_index == index and 'active'" t-attf-style="background-image: url(#{src})"></li>
                </t>
                <li class="o_indicators_right text-center" aria-label="Next" title="Next">
                    <i class="fa fa-chevron-right"/>
                </li>
            </ul>

            <a class="carousel-control-prev text-black" t-attf-href="##{id}" data-slide="prev" aria-label="Previous" title="Previous"><span class="fa fa-chevron-left"/></a>
            <a class="carousel-control-next text-black" t-attf-href="##{id}" data-slide="next" aria-label="Next" title="Next"><span class="fa fa-chevron-right"/></a>
        </div>
    </t>

    <!--
        ========================================================================
        Gallery Slideshow LightBox

        This template is used to display a lightbox with a slideshow.

        This template wraps website.gallery.slideshow in a bootstrap modal
        dialog.
        ========================================================================
    -->
    <t t-name="website.gallery.slideshow.lightbox">
        <div role="dialog" class="modal o_technical_modal fade" aria-labbelledby="Image Gallery Dialog">
            <div class="modal-dialog modal-lg" role="Picture Gallery"
                t-attf-style="min-width: #{dim.min_width}px ; min-height: #{dim.min_height}px ; max-width: #{dim.max_width}px ; max-height: #{dim.max_height}px ; height: #{dim.height}px ;">
                <div class="modal-content">
                    <main class="modal-body o_slideshow">
                        <button type="button" class="close" data-dismiss="modal" style="position: absolute; right: 12px; top: 10px;"><span role="img" aria-label="Close">&amp;times;</span><span class="sr-only">Close</span></button>
                        <t t-call="website.gallery.slideshow"></t>
                    </main>

                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website.pageProperties.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

<!-- Tooltip Dependencies -->

<t t-name="website.get_tooltip_dependencies">
    <t t-foreach="dependencies" t-as="dep">
        <b><t t-esc="dep"/></b>
        <ul>
            <li t-foreach="dep_value" t-as="item">
                <a t-att-href="item_value['link']"
                   t-att-title="item_value['item']"
                   class="o_text_overflow">
                    <t t-esc="item_value['item']"/>
                </a>
            </li>
        </ul>
    </t>
</t>
<t t-name="website.show_page_key_dependencies">
    <div class="col-md-9 offset-md-3">
        <span class="text-muted" id="warn_about_call_message">
            <t t-set="depTooltip">
                <t t-call="website.get_tooltip_dependencies"/>
            </t>
            It looks like your file is being called by
            <a href="#" data-toggle="popover" t-att-data-content="depTooltip" data-html="true" title="Dependencies"><t t-esc="dep_text" /></a>.
            Changing its name will break these calls.
        </span>
    </div>
</t>
<t t-name="website.show_page_dependencies">
    <t t-set="depTooltip">
        <t t-call="website.get_tooltip_dependencies"/>
    </t>
    (could be used in <a href="#" class="o_dependencies_redirect_link"><t t-esc="dep_text" /></a>)
</t>
<t t-name="website.page_dependencies_popover">
    <div class="popover o_redirect_old_url" role="tooltip">
        <div class="arrow"/>
        <h3 class="popover-header"/>
        <div class="popover-body"/>
    </div>
</t>

<!-- Page Properties -->

<div t-name="website.pagesMenu.page_info" class="o_page_management_info">
    <form>
        <ul class="nav nav-tabs" role="tablist">
            <li class="nav-item"><a aria-controls="basic_page_info" role="tab" data-toggle="tab" class="nav-link active" href="#basic_page_info">Name</a></li>
            <li class="nav-item"><a aria-controls="advances_page_info" role="tab" data-toggle="tab" class="nav-link" href="#advances_page_info">Publish</a></li>
        </ul>
        <div class="tab-content mt16">
            <div role="tabpanel" id="basic_page_info" class="tab-pane fade show active">
                <div class="form-group row">
                    <label class="col-form-label col-md-3" for="page_name">Page Name</label>
                    <div class="col-md-9">
                        <input type="text" class="form-control" id="page_name" t-att-value="widget.page.name" />
                    </div>
                </div>
                <div class="form-group warn_about_call"></div>
                <div class="form-group row">
                    <label class="col-form-label col-md-3" for="page_url">Page URL</label>
                    <div class="col-md-9">
                        <div class="input-group">
                            <div class="input-group-prepend">
                                <span class="input-group-text" t-att-title="widget.serverUrl"><small><t t-esc="widget.serverUrlTrunc"/></small></span>
                            </div>
                            <input type="text" class="form-control" id="page_url" t-att-value="widget.page.url" />
                        </div>
                    </div>
                </div>
                <div class="form-group row ask_for_redirect">
                    <label class="col-form-label col-md-3" for="create_redirect">Redirect Old URL</label>
                    <div class="col-md-2">
                        <a>
                            <label class="o_switch" for="create_redirect" >
                                <input type="checkbox" id="create_redirect"/>
                                <span/>
                            </label>
                        </a>
                    </div>
                    <div class="col-md-7 mt4 o_dependencies_redirect_list_popover">
                        <span class="text-muted" id="dependencies_redirect"></span>
                    </div>
                </div>
                <div class="form-group row ask_for_redirect">
                    <label class="col-form-label col-md-3 redirect_type" for="redirect_type">Type</label>
                    <div class="col-md-6 redirect_type">
                        <select class="form-control" id="redirect_type">
                            <option t-foreach="widget.fields.redirect_type.selection" t-as="field" t-att-value="field[0]"><t t-esc="field[0] + ': ' + field[1]" /></option>
                        </select>
                    </div>
                </div>
            </div>
            <div role="tabpanel" id="advances_page_info" class="tab-pane fade">
                <div class="form-group row">
                    <label class="control-label col-md-5" for="is_menu">Show in Top Menu</label>
                    <div class="col-sm-2">
                        <label class="o_switch" for="is_menu" >
                            <input type="checkbox" t-att-checked="widget.page.menu_ids.length > 0 ? true : undefined" id="is_menu"/>
                            <span/>
                        </label>
                    </div>
                </div>
                <div class="form-group row">
                    <label class="control-label col-md-5" for="is_homepage">Use as Homepage</label>
                    <div class="col-sm-2">
                        <label class="o_switch" for="is_homepage" >
                            <input type="checkbox" t-att-checked="widget.page.is_homepage ? true : undefined" id="is_homepage"/>
                            <span/>
                        </label>
                    </div>
                </div>
                <div class="form-group row">
                    <label class="control-label col-md-5" for="is_indexed">
                        Indexed
                        <i class="fa fa-question-circle-o" title="Hide this page from search results" role="img" aria-label="Info"></i>
                    </label>
                    <div class="col-md-2">
                        <label class="o_switch" for="is_indexed" >
                            <input type="checkbox" t-att-checked="widget.page.website_indexed ? true : undefined" id="is_indexed"/>
                            <span/>
                        </label>
                    </div>
                </div>
                <div class="form-group row">
                    <label class="control-label col-md-5" for="is_published">Publish</label>
                    <div class="col-sm-2">
                        <label class="o_switch js_publish_btn" for="is_published">
                            <input type="checkbox" t-att-checked="widget.page.website_published ? true : undefined" id="is_published"/>
                            <span/>
                        </label>
                    </div>
                </div>
                <div class="form-group row">
                    <label class="control-label col-md-5" for="date_publish">Publishing Date</label>
                    <div class="col-md-7">
                        <div class="input-group date" id="date_publish_container" data-target-input="nearest">
                            <input type="text" class="form-control datetimepicker-input" data-target="#date_publish_container" id="date_publish"/>
                            <div class="input-group-append" data-target="#date_publish_container" data-toggle="datetimepicker">
                                <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </form>
</div>
</templates>

```

## File: static\src\xml\website.seo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="Configurator.language_promote">
        <t t-foreach="language" t-as="lang">
            <option t-att-value="lang[0]" t-att-selected="lang[0] == def_lang ? 'selected' : null"><t t-esc="lang[2]" /></option>
        </t>
    </t>

    <div t-name="website.seo_configuration" role="form">
        <section class="js_seo_meta_title_description"/>
        <section class="js_seo_meta_keywords"/>
        <section class="js_seo_image"/>
    </div>

    <t t-name="website.seo_suggestion_list">
        <ul class="list-inline mb0">
            <!-- filled in JS -->
        </ul>
    </t>

    <t t-name="website.seo_list">
        <tbody>
            <!-- filled in JS -->
        </tbody>
    </t>

    <t t-name="website.seo_keyword">
        <tr class="js_seo_keyword" t-att-data-keyword="widget.keyword">
            <td t-esc="widget.keyword"/>
            <td class="text-center"><i t-if="widget.used_h1" class="fa fa-check" t-attf-title="{{ widget.keyword }} is used in page first level heading"/></td>
            <td class="text-center"><i t-if="widget.used_h2" class="fa fa-check" t-attf-title="{{ widget.keyword }} is used in page second level heading"/></td>
            <td class="text-center"><i class="js_seo_keyword_title fa fa-check" style="visibility: hidden;" t-attf-title="{{ widget.keyword }} is used in page title"/></td>
            <td class="text-center"><i class="js_seo_keyword_description fa fa-check" style="visibility: hidden;" t-attf-title="{{ widget.keyword }} is used in page description"/></td>
            <td class="text-center"><i t-if="widget.used_content" class="fa fa-check" t-attf-title="{{ widget.keyword }} is used in page content"/></td>
            <td class="o_seo_keyword_suggestion"/>
            <td class="text-center"><a href="#" class="oe_remove" data-action="remove-keyword" t-attf-title="Remove {{ widget.keyword }}"><i class="fa fa-trash"/></a></td>
        </tr>
    </t>

    <t t-name="website.seo_suggestion">
        <li class="list-inline-item">
            <span class="o_seo_suggestion badge badge-info" t-att-data-keyword="widget.keyword" t-attf-title="Add {{ widget.keyword }}" t-esc="widget.keyword"/>
        </li>
    </t>

    <t t-name="website.seo_preview">
        <div class="oe_seo_preview_g">
            <div class="rc">
                <div class="r"><t t-esc="widget.title"/></div>
                <div class="s">
                    <div class="kv"><t t-esc="widget.url"/></div>
                    <div class="st"><t t-esc="widget.description"/></div>
                </div>
            </div>
        </div>
    </t>

    <div t-name="website.seo_meta_title_description">
        <div class="row">
            <div class="col-lg-6">
                <div class="form-group">
                    <label for="website_meta_title">
                        Title <i class="fa fa-question-circle-o" title="The title will take a default value unless you specify one."/>
                    </label>
                    <input type="text" name="website_meta_title" id="website_meta_title" class="form-control" placeholder="Keep empty to use default value" maxlength="70" size="70"/>
                </div>
                <div class="form-group">
                    <label for="website_meta_description">
                        Description <i class="fa fa-question-circle-o" t-att-title="widget.previewDescription"/>
                    </label>
                    <textarea name="website_meta_description" id="website_meta_description" placeholder="Keep empty to use default value" class="form-control"/>
                    <div class="alert alert-warning mt16 mb0 small" id="website_meta_description_warning" style="display: none;"/>
                </div>
            </div>
            <div class="col-lg-6">
                <div class="card-header">Preview</div>
                <div class="card mb0 p-0">
                    <div class="card-body">
                        <div class="js_seo_preview"/>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <t t-name="website.seo_meta_keywords">
        <label for="website_meta_keywords">
            Keywords
        </label>
        <div class="form-inline" role="form">
            <div class="input-group">
                <input type="text" name="website_meta_keywords" id="website_meta_keywords" class="form-control" placeholder="Keyword" maxlength="30"/>
                <span title="The language of the keyword and related keywords." class="input-group-append">
                    <select name="seo_page_language" id="language-box" class="btn form-control"/>
                </span>
                <span class="input-group-append">
                    <button data-action="add" class="btn btn-primary btn-sm" type="button">Add</button>
                </span>
            </div>
        </div>
        <div class="table-responsive mt16">
            <table class="table table-sm">
                <thead>
                    <tr>
                        <th>Keyword</th>
                        <th class="text-center" title="Used in page first level heading">H1</th>
                        <th class="text-center" title="Used in page second level heading">H2</th>
                        <th class="text-center" title="Used in page title">T</th>
                        <th class="text-center" title="Used in page description">D</th>
                        <th class="text-center" title="Used in page content">C</th>
                        <th title="Most searched topics related to your keyword, ordered by importance">Related keywords</th>
                        <th class="text-center"></th>
                    </tr>
                </thead>
                <!-- body inserted in JS -->
            </table>
        </div>
    </t>

    <div t-name="website.seo_meta_image_selector" class="o_seo_og_image">
        <t t-call="website.og_image_body"/>
    </div>

    <t t-name="website.og_image_body">
        <h4><small>Select an image for social share</small></h4>
        <div class="row">
            <div class="col-lg-6">
                <t t-foreach="widget.images" t-as="image">
                    <div t-attf-class="o_meta_img mt4 #{image === widget.activeMetaImg and ' o_active_image' or ''}">
                        <img t-att-src="image"/>
                    </div>
                </t>
                <div t-if="widget.customImgUrl" t-attf-class="o_meta_img mt4 #{widget.customImgUrl === widget.activeMetaImg and ' o_active_image' or ''}">
                    <span class="o-custom-label w-100 text-white text-center">Custom</span>
                    <img t-att-src="widget.customImgUrl"/>
                </div>
                <div class="o_meta_img_upload mt4" title="Click to choose more images">
                    <i class="fa fa-upload"/>
                </div>
            </div>
            <div class="col-lg-6">
                <div class="card p-0 mb16">
                    <div class="card-header">Social Preview</div>
                    <img class="card-img-top o_meta_active_img" t-att-src="widget.activeMetaImg"/>
                    <div class="card-body px-3 py-2">
                        <h6 class="text-alpha card-title mb0"><t t-esc="widget.metaTitle"/></h6>
                        <small class="card-subtitle text-muted"><t t-esc="widget.serverUrl"/></small>
                        <p t-esc="widget.metaDescription"/>
                  </div>
                </div>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website.share.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="website.social_hover">
        <div class="text-nowrap css_editable_mode_hidden">
            <t t-foreach="medias" t-as="media">
                <a href="#"
                    t-attf-class="fa fa-3x fa-#{media}-square text-#{media} oe_social_#{media}"
                    t-att-title="media"
                    t-att-aria-label="media"/>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\xml\website.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="website.prompt">
        <div role="dialog" class="modal o_technical_modal" tabindex="-1">
                <div class="modal-dialog">
                <div class="modal-content">
                    <header class="modal-header" t-if="window_title">
                        <h3 class="modal-title"><t t-esc="window_title"/></h3>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                    </header>
                    <main class="modal-body">
                        <form role="form" t-att-id="id">
                            <div class="form-group row mb0">
                                <label for="page-name" class="col-md-4 col-form-label">
                                    <t t-esc="field_name"/>:
                                </label>
                                <div class="col-md-8">
                                    <input t-if="field_type == 'input'" type="text" class="form-control" required="required"/>
                                    <textarea t-if="field_type == 'textarea'" class="form-control" required="required" rows="5"></textarea>
                                    <select t-if="field_type == 'select'" class="form-control"></select>
                                </div>
                            </div>
                        </form>
                    </main>
                    <footer class="modal-footer">
                        <button type="button" class="btn btn-primary btn-continue">Continue</button>
                        <button type="button" class="btn btn-secondary" data-dismiss="modal" aria-label="Cancel">Cancel</button>
                    </footer>
                </div>
            </div>
        </div>
    </t>

    <t t-name="website.dependencies">
        <p class="text-warning">Don't forget to update all links referring to this page.</p>
        <t t-if="dependencies and _.keys(dependencies).length">
            <p class="text-warning">We found these ones:</p>
            <div t-foreach="dependencies" t-as="type" class="mb16">
                <a class="collapsed fa fa-caret-right" data-toggle="collapse" t-attf-href="#collapseDependencies#{type_index}" aria-expanded="false" t-attf-aria-controls="collapseDependencies#{type_index}">
                    <t t-esc="type"/>&amp;nbsp;
                    <span class="text-muted"><t t-esc="type_value.length"/> found(s)</span>
                </a>
                <div t-attf-id="collapseDependencies#{type_index}" class="collapse" aria-expanded="false">
                    <ul>
                        <li t-foreach="type_value" t-as="error">
                            <a t-if="!_.contains(['', '#', false], error.link)" t-att-href="error.link">
                                <t t-raw="error.text"/>
                            </a>
                            <t t-else="">
                                <t t-raw="error.text"/>
                            </t>
                        </li>
                    </ul>
                </div>
            </div>
        </t>
    </t>

    <div t-name="website.delete_page">
        <p>Are you sure you want to delete this page ?</p>
        <t t-call="website.dependencies"/>
    </div>

    <div t-name="website.rename_page">
        <div class="card">
            <div class="card-body">
                <form>
                    <div class="form-group row mb0">
                        <label for="new_name" class="col-form-label col-md-4">Rename Page To:</label>
                        <div class="col-md-8">
                            <input type="text" class="form-control" id="new_name" placeholder="e.g. About Us"/>
                        </div>
                    </div>
                </form>
            </div>
        </div>
        <t t-call="website.dependencies"/>
    </div>

    <t t-name="website.oe_applications_menu">
        <t t-as="menu" t-foreach="menu_data.children">
            <a role="menuitem" class="dropdown-item"
               t-att-data-action-id="menu.action ? menu.action.split(',')[1] : undefined"
               t-att-data-action-model="menu.action ? menu.action.split(',')[0] : undefined"
               t-att-data-menu="menu.id"
               t-att-data-menu-xmlid="menu.xmlid"
               t-att-href="_.str.sprintf('/web#menu_id=%s&amp;action=%s', menu.id, menu.action ? menu.action.split(',')[1] : '')">
                <span class="oe_menu_text" t-esc="menu.name"/>
            </a>
        </t>
    </t>
</templates>

```

## File: views\ir_actions_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record model="ir.ui.view" id="view_server_action_form_website">
            <field name="name">ir.actions.server.form.website</field>
            <field name="model">ir.actions.server</field>
            <field name="inherit_id" ref="base.view_server_action_form"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//field[@name='state']" position="after">
                        <field name="website_published"
                            attrs="{'invisible': [('state', '!=', 'code')]}"/>
                        <field name="xml_id" invisible="1"/>
                        <field name="website_path"
                            attrs="{'invisible': ['|', ('website_published', '!=', True), ('state', '!=', 'code')]}"/>
                        <field name="website_url" readonly="1" widget="url"
                            attrs="{'invisible': ['|', ('website_published', '!=', True), ('state', '!=', 'code')]}"/>
                    </xpath>
                </data>
            </field>
        </record>

        <record model="ir.ui.view" id="view_server_action_tree_website">
            <field name="name">ir.actions.server.tree.website</field>
            <field name="model">ir.actions.server</field>
            <field name="inherit_id" ref="base.view_server_action_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='model_id']" position="after">
                    <field name="website_url"/>
                </xpath>
            </field>
        </record>

        <record model="ir.ui.view" id="view_server_action_search_website">
            <field name="name">ir.actions.server.search.website</field>
            <field name="model">ir.actions.server</field>
            <field name="inherit_id" ref="base.view_server_action_search"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='state']" position="after">
                    <filter string="Website" name="website"
                        domain="[('website_published', '=', True), ('state', '=', 'code')]"/>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\ir_attachment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <record id="view_attachment_form_inherit_website" model="ir.ui.view">
        <field name="name">ir.attachment.form.inherit.website</field>
        <field name="model">ir.attachment</field>
        <field name="inherit_id" ref="base.view_attachment_form"/>
        <field name="arch" type="xml">
            <field name="mimetype" position="after">
                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
            </field>
        </field>
    </record>
    <record id="view_attachment_tree_inherit_website" model="ir.ui.view">
       <field name="name">ir.attachment.tree.inherit.website</field>
       <field name="model">ir.attachment</field>
       <field name="inherit_id" ref="base.view_attachment_tree"/>
       <field name="arch" type="xml">
           <field name="name" position="after">
               <field name="website_id" groups="website.group_multi_website"/>
           </field>
       </field>
   </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.website</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="20"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Website" string="Website" data-key="website" groups="website.group_website_designer">
                        <h2 groups="website.group_multi_website">Select the Website to Configure</h2>
                        <div class="row mt16 o_settings_container" id="website_selection_settings" groups="website.group_multi_website">
                            <div class="col-xs-12 col-md-6 o_setting_box" id="website">
                                <div class="o_setting_right_pane">
                                    <label string="Website" for="website_id"/>
                                    <div class="text-muted">
                                        Settings on this page will apply to this website
                                    </div>
                                    <div class="mt16">
                                        <field name="website_id" widget="selection"/>
                                    </div>
                                    <div>
                                        <button name="action_website_create_new" type="object" string="Create a New Website" class="btn-secondary" icon="fa-arrow-right"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Website <button name="action_website_create_new" type="object" string="New" class="ml-2 btn btn-link" icon="fa-plus" groups="!website.group_multi_website"/></h2>
                        <div class="row mt16 o_settings_container" id="website_settings_placeholder" attrs="{'invisible': [('website_id', '!=', False)]}">
                            <div class="col-12 o_setting_box">
                                <div class="text-muted" groups="website.group_multi_website">
                                    Select a website to load its settings.
                                </div>
                                <div class="text-muted" groups="!website.group_multi_website">
                                    There is no website available for this company. You could create a new one.
                                </div>
                            </div>
                        </div>
                        <!-- !! Every fields inside this container should be website specific (related to website record) !! -->
                        <div class="row mt16 o_settings_container" id="website_settings" attrs="{'invisible': [('website_id', '=', False)]}">
                            <div class="col-12 o_setting_box" id="website_action_setting" style="margin-left: 30px; margin-bottom: 16px;">
                                <button name="website_go_to" type="object" string="Go to Website" class="btn btn-primary" icon="fa-globe"/>
                                <button name="%(website.action_website_add_features)d" type="action" string="Add features" class="ml-2 btn btn-secondary" icon="fa-plus"/>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="domain_setting">
                                <div class="o_setting_right_pane">
                                    <label for="website_name" string="Website Title"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Name and favicon of your website
                                    </div>
                                    <div class="content-group">
                                        <div class="row mt16">
                                            <label class="col-lg-3 o_light_label" string="Name" for="website_name"/>
                                            <field name="website_name" attrs="{'required': [('website_id', '!=', False)]}"/>
                                        </div>
                                        <div class="row">
                                            <label class="col-lg-3 o_light_label" for="favicon" />
                                            <field name="favicon" widget="image" class="float-left oe_avatar"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="company_settings" groups="base.group_multi_company">
                                <div class="o_setting_right_pane">
                                    <label string="Company" for="website_company_id"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        The company this website belongs to
                                    </div>
                                    <field name="website_company_id" attrs="{'required': [('website_id', '!=', False)]}" />
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="languages_setting">
                                <div class="o_setting_right_pane">
                                    <label for="language_ids"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Languages available on your website
                                    </div>
                                    <div class="content-group">
                                        <div class="mt16">
                                            <field name="language_ids" widget="many2many_tags" options="{'no_create': True, 'no_open': True}"
                                                attrs="{'required': [('website_id', '!=', False)]}"/>
                                        </div>
                                        <field name="website_language_count" invisible="1"/>
                                        <div class="mt8" attrs="{'invisible':[('website_language_count', '&lt;', 2)]}">
                                            <label class="o_light_label mr8" string="Default" for="website_default_lang_id"/>
                                            <field name="website_default_lang_id" widget="selection" attrs="{'required': [('website_id', '!=', False)]}"/>
                                        </div>
                                    </div>
                                    <div class="mt8">
                                        <button type="action" name="%(base.action_view_base_language_install)d" string="Install new language" class="btn-link" icon="fa-arrow-right"/>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="domain_settings" groups="website.group_multi_website">
                                <div class="o_setting_right_pane">
                                    <label string="Domain" for="website_domain"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Display this website when users visit this domain
                                    </div>
                                    <div class="mt8">
                                        <field name="website_domain" placeholder="https://www.odoo.com"/>
                                    </div>
                                    <div class="mt8 text-muted" title="You can have 2 websites with same domain AND a condition on country group to select wich website use.">
                                        Once the selection of available websites by domain is done, you can filter by country group.
                                    </div>
                                    <div class="mt8">
                                        <field name="website_country_group_ids" widget="many2many_tags"/>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="auth_signup_uninvited_setting"
                                title=" To send invitations in B2B mode, open a contact or select several ones in list view and click on 'Portal Access Management' option in the dropdown menu *Action*.">
                                <div class="o_setting_left_pane">
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="auth_signup_uninvited"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Let your customers log in to see their documents
                                    </div>
                                    <div class="mt8">
                                        <field name="auth_signup_uninvited" class="o_light_label" widget="radio" required="True"/>
                                    </div>
                                    <div class="mt8 content-group">
                                        <button type="object" name="open_template_user" string="Default Access Rights" icon="fa-arrow-right" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="website_logo_setting">
                                <div class="o_setting_right_pane">
                                    <label for="website_logo"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Display this logo on the website.
                                    </div>
                                    <field name="website_logo" widget="image" class="w-25 mt-2"/>
                                </div>
                            </div>
                            <div class="col-12 col-lg-offset-6 col-lg-6 o_setting_box" id="google_analytics_setting">
                                <div class="o_setting_left_pane">
                                    <field name="has_google_analytics"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="has_google_analytics"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Track visits in Google Analytics
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('has_google_analytics', '=', False)]}">
                                        <div class="row mt16">
                                            <label class="col-lg-3 o_light_label" string="Tracking ID" for="google_analytics_key"/>
                                            <field name="google_analytics_key" placeholder="UA-XXXXXXXX-Y"
                                                attrs="{'required': [('has_google_analytics', '=', True)]}"/>
                                        </div>
                                    </div>
                                    <div attrs="{'invisible': [('has_google_analytics', '=', False)]}">
                                        <a href="https://www.odoo.com/documentation/13.0/applications/websites/website/optimize/google_analytics.html"
                                                class="oe_link" target="_blank">
                                            <i class="fa fa-arrow-right"/>
                                            How to get my Tracking ID
                                        </a>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="google_analytics_dashboard_setting" attrs="{'invisible': [('has_google_analytics', '=', False)]}">
                                <div class="o_setting_left_pane">
                                    <field name="has_google_analytics_dashboard"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="has_google_analytics_dashboard"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Follow your website traffic in Odoo.
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('has_google_analytics_dashboard', '=', False)]}">
                                        <div class="row mt16">
                                            <label class="col-lg-3 o_light_label" string="Client ID" for="google_management_client_id"/>
                                            <field name="google_management_client_id" attrs="{'required': [('has_google_analytics_dashboard', '=', True)]}"/>
                                        </div>
                                        <div class="row">
                                            <label class="col-lg-3 o_light_label" string="Client Secret" for="google_management_client_secret"/>
                                            <field name="google_management_client_secret" attrs="{'required': [('has_google_analytics_dashboard', '=', True)]}"/>
                                        </div>
                                    </div>
                                    <div attrs="{'invisible': [('has_google_analytics_dashboard', '=', False)]}">
                                        <a href="https://www.odoo.com/documentation/13.0/applications/websites/website/optimize/google_analytics_dashboard.html"
                                            class="oe_link" target="_blank">
                                            <i class="fa fa-arrow-right"/>
                                            How to get my Client ID
                                        </a>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" groups="website.group_multi_website">
                                <div class="o_setting_left_pane">
                                    <field name="specific_user_account"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="specific_user_account"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Force your user to create an account per website
                                    </div>
                                </div>
                            </div>

                            <div class="col-12 col-lg-6 o_setting_box" id="cdn_setting" title="A CDN helps you serve your website’s content with high availability and high performance to any visitor wherever they are located." groups="base.group_no_one">
                                <div class="o_setting_left_pane">
                                    <field name="cdn_activated"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="cdn_activated"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Use a CDN to optimize the availability of your website's content
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('cdn_activated', '=', False)]}">
                                        <div class="row mt16">
                                            <label class="col-lg-3 o_light_label" for="cdn_url"/>
                                            <field name="cdn_url"
                                                   attrs="{'required': [('cdn_activated', '=', True)]}"
                                                   placeholder="//mycompany.mycdn.com/"
                                                   t-translation="off"/>
                                        </div>
                                        <div class="row" >
                                            <label class="col-lg-3 o_light_label" for="cdn_filters"/>
                                            <field name="cdn_filters" class="oe_inline"
                                                   attrs="{'required': [('cdn_activated', '=', True)]}"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="has_social_network"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Social Media" for="has_social_network"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Add links to social media on your website
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('has_social_network', '=', False)]}">
                                        <div class="row">
                                            <label for="social_twitter" string="Twitter" class="col-md-3 o_light_label"/>
                                            <field name="social_twitter"/>
                                        </div>
                                        <div class="row">
                                            <label for="social_facebook" string="Facebook" class="col-md-3 o_light_label"/>
                                            <field name="social_facebook"/>
                                        </div>
                                        <div class="row">
                                            <label for="social_github" string="GitHub" class="col-md-3 o_light_label"/>
                                            <field name="social_github"/>
                                        </div>
                                        <div class="row">
                                            <label for="social_linkedin" string="LinkedIn" class="col-md-3 o_light_label"/>
                                            <field name="social_linkedin"/>
                                        </div>
                                        <div class="row">
                                            <label for="social_youtube" string="YouTube" class="col-md-3 o_light_label"/>
                                            <field name="social_youtube"/>
                                        </div>
                                        <div class="row">
                                            <label for="social_instagram" string="Instagram" class="col-md-3 o_light_label"/>
                                            <field name="social_instagram"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="social_default_image_setting">
                                <div class="o_setting_right_pane">
                                    <label string="Default Social Share Image" for="social_default_image"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        If set, replaces the company logo as the default social share image.
                                    </div>
                                    <field name="social_default_image" widget="image" class="w-25 mt-2"/>
                                </div>
                            </div>

                        </div>
                        <h2>Features</h2>
                        <div class="row mt16 o_settings_container" id="webmaster_settings">
                            <div id="multi_website" class="col-12 col-md-6 o_setting_box">
                                <div class="o_setting_left_pane">
                                    <field name="group_multi_website"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Multi-Website" for="group_multi_website"/>
                                    <div class="text-muted">
                                        Manage multiple websites
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="google_maps_setting">
                                <div class="o_setting_left_pane">
                                    <field name="has_google_maps"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="has_google_maps"/>
                                    <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                                    <div class="text-muted">
                                        Use Google Map on your website (<a href="/contactus">Contact Us</a> page, snippets, ...)
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('has_google_maps', '=', False)]}">
                                        <div class="row mt16">
                                            <label class="col-lg-3 o_light_label" string="API Key" for="google_maps_api_key"/>
                                            <field name="google_maps_api_key" attrs="{'required': [('has_google_maps', '=', True)]}" />
                                        </div>
                                    </div>
                                    <div class="mt8" attrs="{'invisible': [('has_google_maps', '=', False)]}">
                                        <a role="button" class="btn-link" target="_blank"
                                           href="https://console.developers.google.com/flows/enableapi?apiid=maps_backend,static_maps_backend&amp;keyType=CLIENT_SIDE&amp;reusekey=true">
                                            <i class="fa fa-arrow-right"/>
                                            Create a Google Project and Get a Key
                                        </a>
                                        <a role="button" class="btn-link" target="_blank"
                                           href="https://cloud.google.com/maps-platform/pricing">
                                            <i class="fa fa-arrow-right"/>
                                            Enable billing on your Google Project
                                        </a>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="utm_tracker_settings" title="Analyze the efficiency of your marketing campaigns by using trackable UTM trackers (campaigns, medium, sources). Create trackers and follow clicks from the Promote menu of your website. Those trackers can be used in Google Analytics or in Odoo reports where you can see the opportunities and sales revenue generated thanks to your links.">
                                <div class="o_setting_left_pane">
                                    <field name="module_website_links"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_website_links"/>
                                    <div class="text-muted">
                                        Track clicks on UTM links
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_website_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'website', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_website_global_configuration" parent="menu_website_configuration"
            sequence="100" name="Configuration" groups="base.group_system"/>

        <menuitem name="Settings"
            id="menu_website_website_settings"
            action="action_website_configuration"
            parent="menu_website_global_configuration"
            groups="base.group_system"
            sequence="10"/>

        <menuitem id="menu_website_add_features" parent="website.menu_website_global_configuration"
            sequence="20" groups="base.group_system" action="action_website_add_features"/>

        <menuitem name="Websites"
            id="menu_website_websites_list"
            action="action_website_list"
            parent="menu_website_global_configuration"
            groups="base.group_no_one"
            sequence="10"
            />

        <menuitem name="Pages"
            id="menu_website_pages_list"
            action="action_website_pages_list"
            parent="menu_website_global_configuration"
            sequence="30"
            />

        <menuitem name="Menus"
            id="menu_website_menu_list"
            action="action_website_menu"
            parent="menu_website_global_configuration"
            sequence="45"
            groups="base.group_no_one"/>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_partner_form_inherit_website" model="ir.ui.view">
        <field name="name">res.partner.form.website.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='misc']/field[@name='company_id']" position="after">
                <field name="website_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_title" name="Title">
    <section class="pt32 pb32">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_title pt16 pb16" style="text-align: center;">
                    <h1 class="s_title_default"><font style="font-size: 62px;">Your Site Title</font></h1>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_cover" name="Cover">
    <section class="s_cover parallax s_parallax_is_fixed bg-black-50 pt96 pb96" data-scroll-background-ratio="1">
        <span class="s_parallax_bg oe_img_bg oe_custom_bg" style="background-image: url('/web/image/website.s_cover_default_image'); background-position: 50% 0;"/>
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_title" data-name="Title">
                    <h1 class="s_title_thin" style="font-size: 62px; text-align: center;">Catchy Headline</h1>
                </div>
                <div class="col-lg-12 s_text pt16 pb16" data-name="Text">
                    <p class="lead" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                </div>
                <div class="col-lg-12 s_btn text-center pt16 pb16" data-name="Buttons">
                    <a href="/aboutus" class="btn btn-delta rounded-circle">About us</a>
                    <a href="/contactus" class="btn btn-primary rounded-circle">Contact us</a>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_text_image" name="Text - Image">
    <section class="s_text_image pt32 pb32">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 pt16 pb16">
                    <h2>A Section Subtitle</h2>
                    <p>Write one or two paragraphs describing your product or services. <br/>To be successful your content needs to be useful to your readers.</p>
                    <p>Start with the customer – find out what they want and give it to them.</p>
                    <div class="s_btn text-left pt16 pb16" data-name="Buttons">
                        <a href="#" class="btn btn-primary">Learn more</a>
                    </div>
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <img src="/web/image/website.s_text_image_default_image" class="img img-fluid mx-auto" alt="Odoo • Text and Image"/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_image_text" name="Image - Text">
    <section class="s_text_image pt32 pb32">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 pt16 pb16">
                    <img src="/web/image/website.s_image_text_default_image" class="img img-fluid mx-auto" alt="Odoo • Image and Text"/>
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <h2>Section Subtitle</h2>
                    <p>Write one or two paragraphs describing your product or services. <br/>To be successful your content needs to be useful to your readers.</p>
                    <p>Start with the customer – find out what they want and give it to them.</p>
                    <div class="s_btn text-left pt16 pb16">
                        <a href="#" class="btn btn-outline-primary">Discover more</a>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_banner" name="Banner">
    <section class="s_banner parallax s_parallax_is_fixed pt96 pb96" data-scroll-background-ratio="1">
        <span class="s_parallax_bg oe_img_bg oe_custom_bg" style="background-image: url('/web/image/website.s_banner_default_image'); background-position: 50% 0;"/>
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-7 bg-white jumbotron rounded pt32 pb32" data-name="Box">
                    <div class="row">
                        <div class="col-lg-12 s_title s_col_no_bgcolor" data-name="Title">
                            <h1 class="s_title_thin"><font style="font-size: 62px;"><b>Sell Online.</b> Easily.</font></h1>
                        </div>
                        <div class="col-lg-12 pt8 pb32 s_col_no_bgcolor" data-name="Text">
                            <p class="lead">This is a simple hero unit, a simple jumbotron-style component <br/>for calling extra attention to featured content or information.</p>
                        </div>
                        <div class="col-lg-12 s_btn text-left pt16 pb16" data-name="Button">
                            <a href="#" class="btn btn-epsilon rounded-circle">Do something</a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_text_block" name="Text block">
    <section class="s_text_block pt32 pb32">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 pt32 pb32">
                    <p class="lead">A great way to catch your reader's attention is to tell a story. <br/>Everything you consider writing can be told as a story.</p>
                    <p><b>Great stories have personality.</b> Consider telling a great story that provides personality. Writing a story with personality for potential clients will assists with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.</p>
                    <p><b>Great stories are for everyone even when only written for just one person.</b> If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.</p>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_features" name="Features">
    <section class="s_features pt32 pb32">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 pt32 pb32 text-center">
                    <i class="fa fa-3x fa-gear rounded-circle bg-gamma m-3"/>
                    <h3>First Feature</h3>
                    <p>Tell what's the value for the <br/>customer for this feature.</p>
                </div>
                <div class="col-lg-4 pt32 pb32 text-center">
                    <i class="fa fa-3x fa-photo rounded bg-epsilon m-3"/>
                    <h3>Second Feature</h3>
                    <p>Write what the customer would like to know, <br/>not what you want to show.</p>
                </div>
                <div class="col-lg-4 pt32 pb32 text-center">
                    <i class="fa fa-3x fa-leaf rounded-leaf bg-primary m-3"/>
                    <h3>Third Feature</h3>
                    <p>A small explanation of this great <br/>feature, in clear words.</p>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_three_columns" name="Columns">
    <section class="s_three_columns bg-200 pt32 pb32">
        <div class="container">
            <div class="row d-flex align-items-stretch">
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white">
                        <img class="card-img-top" src="/web/image/website.library_image_11" alt="Odoo - Sample 1 for three columns"/>
                        <div class="card-body">
                            <h3 class="card-title">Feature One</h3>
                            <p class="card-text">Adapt these three columns to fit your design need. To duplicate, delete or move columns, select the column and use the top icons to perform your action.</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white">
                        <img class="card-img-top" src="/web/image/website.library_image_13" alt="Odoo - Sample 2 for three columns"/>
                        <div class="card-body">
                            <h3 class="card-title">Feature Two</h3>
                            <p class="card-text">To add a fourth column, reduce the size of these three columns using the right icon of each block. Then, duplicate one of the columns to create a new one as a copy.</p>
                        </div>
                        <div class="card-footer">
                            <i class="fa fa-info-circle mr-1"/> <small>Additional information</small>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white">
                        <img class="card-img-top" src="/web/image/website.library_image_07" alt="Odoo - Sample 3 for three columns"/>
                        <div class="card-body">
                            <h3 class="card-title">Feature Three</h3>
                            <p class="card-text">Delete the above image or replace it with a picture that illustrates your message. Click on the picture to change its <em>rounded corner</em> style.</p>
                            <div class="s_btn text-left pb0 pt16" data-name="Button">
                                <a href="#" class="btn btn-primary btn-sm">Learn more</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_picture" name="Picture">
    <section class="s_picture bg-200 pt48 pb24">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_title pt16 pb16">
                    <h2 class="s_title_thin" style="text-align: center;"><font style="font-size: 62px;">A punchy Headline</font></h2>
                </div>
                <div class="col-lg-12 pt16 pb16">
                    <p style="text-align: center;">Choose a vibrant image and write an inspiring paragraph about it.<br/> It does not have to be long, but it should reinforce your image.</p>
                </div>
                <div class="col-lg-8 offset-lg-2 pb24">
                    <figure class="figure">
                        <img src="/web/image/website.s_picture_default_image" class="figure-img img-fluid rounded img-thumbnail padding-large" alt="Odoo • A picture with a caption"/>
                        <figcaption class="figure-caption py-3 text-center"><i class="fa fa-1x fa-picture-o mr-2"/>Add a caption to enhance the meaning of this image.</figcaption>
                    </figure>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_carousel" name="Carousel">
    <div id="myCarousel" class="s_carousel s_carousel_default carousel slide" data-interval="10000">
        <!-- Indicators -->
        <ol class="carousel-indicators">
            <li data-target="#myCarousel" data-slide-to="0" class="active"/>
            <li data-target="#myCarousel" data-slide-to="1"/>
            <li data-target="#myCarousel" data-slide-to="2"/>
        </ol>
        <!-- Content -->
        <div class="carousel-inner">
            <!-- #01 -->
            <div class="carousel-item active oe_custom_bg oe_img_bg pt152 pb152" style="background-image: url('/web/image/website.s_carousel_default_image_1');" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <div class="carousel-content col-lg-7">
                            <div class="s_title pb8" data-name="Title">
                                <h2 class="s_title_default"><font style="font-size: 62px;">Slide Title</font></h2>
                            </div>
                            <p class="lead">Use this snippet to presents your content in a slideshow-like format.<br/> Don't write about products or services here, write about solutions.</p>
                            <div class="s_btn text-left pt16 pb16" data-name="Buttons">
                                <a href="/aboutus" class="btn btn-secondary flat">About us</a>
                                <a href="/contactus" class="btn btn-primary flat">Contact us</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- #02 -->
            <div class="carousel-item oe_custom_bg oe_img_bg pt96 pb96" style="background-image: url('/web/image/website.s_carousel_default_image_2');" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <div class="carousel-content col-lg-8 offset-lg-2 bg-black-50 text-center pt48 pb40">
                            <h2 style="font-size: 62px;">Clever Slogan</h2>
                            <div class="s_hr pt8 pb32">
                                <hr class="s_hr_5px s_hr_dotted border-600 w-25 border-epsilon mx-auto text-center"/>
                            </div>
                            <p class="lead">Storytelling is powerful.<br/> It draws readers in and engages them.</p>
                            <div class="s_btn text-center pt16 pb16" data-name="Buttons">
                                <a href="/" class="btn btn-epsilon rounded-circle">Start your journey</a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <!-- #03 -->
            <div class="carousel-item oe_custom_bg oe_img_bg pt128 pb128" style="background-image: url('/web/image/website.s_carousel_default_image_3');" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <div class="carousel-content col-lg-6 offset-lg-6">
                            <h2><font style="font-size: 62px; background-color: rgb(255, 255, 255);">Edit this title</font></h2>
                            <h4><font style="background-color: rgb(255, 255, 255);">Good writing is simple, but not simplistic.</font></h4>
                            <p class="mt24">Good copy starts with understanding how your product or service helps your customers. Simple words communicate better than big words and pompous language.</p>
                            <t t-call="website.s_share">
                                <t t-set="_classes" t-value="'text-left pt16 pb16'"/>
                                <t t-set="_no_title" t-value="True"/>
                            </t>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- Controls -->
        <div class="carousel-control-prev" data-target="#myCarousel" data-slide="prev" role="img" aria-label="Previous" title="Previous">
            <span class="carousel-control-prev-icon"/>
            <span class="sr-only">Previous</span>
        </div>
        <div class="carousel-control-next" data-target="#myCarousel" data-slide="next" role="img" aria-label="Next" title="Next">
            <span class="carousel-control-next-icon"/>
            <span class="sr-only">Next</span>
        </div>
    </div>
</template>

<template id="s_alert" name="Alert">
    <div class="s_alert s_alert_md alert-delta w-100 clearfix">
        <i class="fa fa-2x fa-info-circle s_alert_icon"/>
        <div class="s_alert_content">
            <p>Explain the benefits you offer. <br/>Don't write about products or services here, write about solutions.</p>
        </div>
    </div>
</template>

<template id="s_card" name="Card">
    <div class="s_card card bg-white w-100">
        <h4 class="card-header">Feature Title</h4>
        <div class="card-body">
            <p class="card-text">A card is a flexible and extensible content container. It includes options for headers and footers, a wide variety of content, contextual background colors, and powerful display options.</p>
        </div>
        <div class="card-footer">
            <i class="fa fa-1x fa-clock-o mr8"/><small>2 days ago</small>
        </div>
    </div>
</template>

<template id="s_share" name="Share">
    <div t-attf-class="s_share #{_classes}">
        <h4 t-if="not _no_title" class="s_share_title">Share</h4>
        <a href="https://www.facebook.com/sharer/sharer.php?u={url}" t-attf-class="s_share_facebook #{_link_classes}" target="_blank">
            <i t-attf-class="fa fa-1x fa-facebook #{not _link_classes and 'rounded shadow-sm'}"/>
        </a>
        <a href="https://twitter.com/intent/tweet?text={title}&amp;url={url}" t-attf-class="s_share_twitter #{_link_classes}" target="_blank">
            <i t-attf-class="fa fa-1x fa-twitter #{not _link_classes and 'rounded shadow-sm'}"/>
        </a>
        <a href="http://www.linkedin.com/shareArticle?mini=true&amp;url={url}&amp;title={title}&amp;" t-attf-class="s_share_linkedin #{_link_classes}" target="_blank">
            <i t-attf-class="fa fa-1x fa-linkedin #{not _link_classes and 'rounded shadow-sm'}"/>
        </a>
        <a href="mailto:?body={url}&amp;subject={title}" t-attf-class="s_share_email #{_link_classes}">
            <i t-attf-class="fa fa-1x fa-envelope #{not _link_classes and 'rounded shadow-sm'}"/>
        </a>
    </div>
</template>

<template id="s_rating" name="Rating">
    <div class="s_rating row">
        <div class="col-lg-12 s_rating_stars s_rating_3 s_rating_1x pt16 pb16">
            <h4>Quality</h4>
            <i class="fa fa-1x"/>
            <i class="fa fa-1x"/>
            <i class="fa fa-1x"/>
            <i class="fa fa-1x"/>
            <i class="fa fa-1x"/>
            <div class="s_rating_bar"/>
        </div>
    </div>
</template>

<template id="s_btn" name="Button">
    <div class="s_btn text-center pt16 pb16" data-name="Buttons">
        <a href="#" class="btn btn-primary">Read more</a>
    </div>
</template>

<template id="s_hr" name="Separator">
    <div class="s_hr text-left pt32 pb32">
        <hr class="s_hr_1px s_hr_solid border-600 w-100 mx-auto"/>
    </div>
</template>

<template id="s_facebook_page" name="Facebook Page">
    <div class="o_facebook_page">
        <div class="o_facebook_alert alert alert-info" role="status">
            <span class="o_add_facebook_page">
                <i class="fa fa-plus-circle"/> Add Facebook Page
            </span>
        </div>
    </div>
</template>

<template id="s_image_gallery" name="Image Gallery">
    <section class="o_gallery o_spc-medium o_slideshow s_image_gallery" data-columns="3" style="height: 500px; overflow: hidden;">
        <div class="container">
            <div class="alert alert-info css_non_editable_mode_hidden text-center" role="status"><span class="o_add_images" style="cursor: pointer;"><i class="fa fa-plus-circle"/> Add Images</span></div>
        </div>
    </section>
</template>

<template id="s_comparisons" name="Comparisons">
    <section class="s_comparisons pt48 pb24">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_title pt16 pb16" style="text-align: center;">
                    <h1 class="s_title_thin"><font style="font-size: 62px;">Our offers</font></h1>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-4 s_col_no_bgcolor text-center pt16 pb16" data-name="Box">
                    <div class="card bg-200">
                        <h4 class="card-header">Beginner</h4>
                        <div class="card-body text-center">
                            <h2 class="card-title">
                                <span class="s_comparisons_currency">$</span>
                                <span class="s_comparisons_price"><b>35</b></span>
                                <span class="s_comparisons_decimal">.00</span>
                            </h2>
                            <small>/ month</small>
                        </div>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item">Basic sales &amp; marketing for up to 2 users</li>
                            <li class="list-group-item">Account &amp; Sales management</li>
                            <li class="list-group-item">No customization</li>
                            <li class="list-group-item">No support</li>
                        </ul>
                        <div class="card-footer">
                            <p><i>Instant setup, satisfied or reimbursed.</i></p>
                            <a href="/contactus" class="btn btn-secondary">Order now</a>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor text-center pt16 pb16" data-name="Box">
                    <div class="card bg-primary">
                        <h4 class="card-header">Professional</h4>
                        <div class="card-body">
                            <h2 class="card-title">
                                <span class="s_comparisons_currency">$</span>
                                <span class="s_comparisons_price"><b>65</b></span>
                                <span class="s_comparisons_decimal">.00</span>
                            </h2>
                            <small>/ month</small>
                        </div>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item">Complete CRM for any size team</li>
                            <li class="list-group-item">Get access to all modules</li>
                            <li class="list-group-item">Limited customization</li>
                            <li class="list-group-item">Email support</li>
                        </ul>
                        <div class="card-footer">
                            <p><i>Instant setup, satisfied or reimbursed.</i></p>
                            <a href="/contactus" class="btn btn-light">Start now</a>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor text-center pt16 pb16" data-name="Box">
                    <div class="card bg-secondary">
                        <h4 class="card-header">Expert</h4>
                        <div class="card-body text-center">
                            <h2 class="card-title">
                                <span class="s_comparisons_currency">$</span>
                                <span class="s_comparisons_price"><b>125</b></span>
                                <span class="s_comparisons_decimal">.00</span>
                            </h2>
                            <small>/ month</small>
                        </div>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item">Unlimited CRM power and support</li>
                            <li class="list-group-item">Get access to all modules and features</li>
                            <li class="list-group-item">Unlimited customization</li>
                            <li class="list-group-item">24x7 toll-free support</li>
                        </ul>
                        <div class="card-footer">
                            <p><i>Instant setup, satisfied or reimbursed.</i></p>
                            <a href="/contactus" class="btn btn-light">Contact us</a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_company_team" name="Company Team">
    <section class="s_company_team">
        <div class="container">
            <h1 style="text-align: center;">Meet the Executive Team</h1>
            <div class="row s_nb_column_fixed">
                <div class="col-lg-6 pt16 pb16">
                    <div class="row s_col_no_resize s_col_no_bgcolor">
                        <div class="col-lg-4 ">
                            <img alt="Company team" src="/web/image/website.s_company_team_image_1" class="img-fluid rounded-circle d-block mx-auto shadow"/>
                        </div>
                        <div class="col-lg-8">
                            <h4>Tony Fred, CEO</h4>
                            <p class="text-muted">
                                Founder and chief visionary, Tony is the driving force behind Company. He loves
                                to keep his hands full by participating in the development of the software,
                                marketing and the Customer Experience strategies.
                            </p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <div class="row s_col_no_resize s_col_no_bgcolor">
                        <div class="col-lg-4">
                            <img alt="Company team" src="/web/image/website.s_company_team_image_2" class="img-fluid rounded-circle d-block mx-auto shadow"/>
                        </div>
                        <div class="col-lg-8">
                            <h4>Mich Stark, COO</h4>
                            <p class="text-muted">
                                Mich loves taking on challenges. With his multi-year experience as Commercial
                                Director in the software industry, Mich has helped Company to get where it
                                is today. Mich is among the best minds.
                            </p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <div class="row s_col_no_resize s_col_no_bgcolor">
                        <div class="col-lg-4">
                            <img alt="Company team" src="/web/image/website.s_company_team_image_3" class="img-fluid rounded-circle d-block mx-auto shadow"/>
                        </div>
                        <div class="col-lg-8">
                            <h4>Aline Turner, CTO</h4>
                            <p class="text-muted">
                                Aline is one of the iconic person in life who can say she loves what she does.
                                She mentors 100+ in-house developers and looks after the community of over
                                thousands developers.
                            </p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <div class="row s_col_no_resize s_col_no_bgcolor">
                        <div class="col-lg-4">
                            <img alt="Company team" src="/web/image/website.s_company_team_image_4" class="img-fluid rounded-circle d-block mx-auto shadow"/>
                        </div>
                        <div class="col-lg-8">
                            <h4>Iris Joe, CFO</h4>
                            <p class="text-muted">
                                Iris, with her international experience, helps us easily understand the numbers and
                                improves them. She is determined to drive success and delivers her professional
                                acumen to bring Company at the next level.
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_call_to_action" name="Call to Action">
    <section class="s_call_to_action bg-200 pt48 pb24">
        <div class="container">
            <div class="row">
                <div class="col-lg-9 pb24">
                    <h3><b>50,000+ companies</b> run Odoo to grow their businesses.</h3>
                    <p class="lead">Join us and make your company a better place.</p>
                </div>
                <div class="col-lg-3 s_btn text-right pt8 pb24">
                    <a href="/contactus" class="btn btn-primary btn-lg">
                        <span>Contact us</span>
                        <i class="fa fa-1x fa-fw fa-arrow-circle-right ml-1"/>
                    </a>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_references" name="References">
    <section class="s_references bg-200 pt16 pb16">
        <div class="container">
            <div class="row">
                <div class="col-lg-2 pt32">
                    <h4>References</h4>
                </div>
                <div class="col-lg-2">
                    <img src="/web/image/website.s_reference_demo_image_1" class="img img-fluid " alt="Demo Logo"/>
                </div>
                <div class="col-lg-2">
                    <img src="/web/image/website.s_reference_demo_image_2" class="img img-fluid " alt="Demo Logo"/>
                </div>
                <div class="col-lg-2">
                    <img src="/web/image/website.s_reference_demo_image_3" class="img img-fluid " alt="Demo Logo"/>
                </div>
                <div class="col-lg-2">
                    <img src="/web/image/website.s_reference_demo_image_4" class="img img-fluid " alt="Demo Logo"/>
                </div>
                <div class="col-lg-2">
                    <img src="/web/image/website.s_reference_demo_image_5" class="img img-fluid " alt="Demo Logo"/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_faq_collapse" name="Accordion">
    <section class="s_faq_collapse pt48 pb32">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_title pt16 pb16">
                    <h1 class="s_title_thin"><font style="font-size: 62px;">FAQ</font></h1>
                </div>
            </div>
            <div class="row s_col_no_bgcolor">
                <div class="col-lg-12 pt16 pb16">
                    <div id="myCollapse" class="accordion" role="tablist">
                        <div class="card bg-white">
                            <a href="#" role="tab" data-toggle="collapse" aria-expanded="true" class="card-header">Terms of service</a>
                            <div class="collapse show" role="tabpanel">
                                <div class="card-body">
                                    <p class="card-text">These terms of service ("Terms", "Agreement") are an agreement between the website ("Website operator", "us", "we" or "our") and you ("User", "you" or "your"). This Agreement sets forth the general terms and conditions of your use of this website and any of its products or services (collectively, "Website" or "Services").</p>
                                </div>
                            </div>
                        </div>
                        <div class="card bg-white">
                            <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header">Links to other Websites</a>
                            <div class="collapse" role="tabpanel">
                                <div class="card-body">
                                    <p class="card-text">Although this Website may be linked to other websites, we are not, directly or indirectly, implying any approval, association, sponsorship, endorsement, or affiliation with any linked website, unless specifically stated herein.</p>
                                    <p class="card-text">You should carefully review the legal statements and other conditions of use of any website which you access through a link from this Website. Your linking to any other off-site pages or other websites is at your own risk.</p>
                                </div>
                            </div>
                        </div>
                        <div class="card bg-white">
                            <a href="#" role="tab" data-toggle="collapse" aria-expanded="false" class="collapsed card-header">Use of Cookies</a>
                            <div class="collapse" role="tabpanel">
                                <div class="card-body">
                                    <p class="card-text">Website may use cookies to personalize and facilitate maximum navigation of the User by this site. The User may configure his / her browser to notify and reject the installation of the cookies sent by us.</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_features_grid" name="Features Grid">
    <section class="s_features_grid pt48 pb24">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 s_col_no_bgcolor pb24">
                    <div class="row">
                        <div class="col-lg-12 pb24" data-name="Box">
                            <h2>First list of Features</h2>
                            <h5 class="text-primary">Add a great slogan.</h5>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-font-awesome rounded-circle bg-primary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Change Icons</h4>
                                <p>Double click an icon to replace it with one of your choice.</p>
                            </div>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-files-o rounded-circle bg-primary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Duplicate</h4>
                                <p>Duplicate blocks and columns to add more features.</p>
                            </div>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-trash rounded-circle bg-primary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Delete Blocks</h4>
                                <p>Select and delete blocks to remove some features.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 s_col_no_bgcolor pb24">
                    <div class="row">
                        <div class="col-lg-12 pb24" data-name="Box">
                            <h2>Second list of Features</h2>
                            <h5 class="text-secondary">Add a great slogan.</h5>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-magic rounded bg-secondary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Great Value</h4>
                                <p>Turn every feature into a benefit for your reader.</p>
                            </div>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-eyedropper rounded bg-secondary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Edit Styles</h4>
                                <p>You can edit colors and background to highlight features.</p>
                            </div>
                        </div>
                        <div class="col-lg-12 pt16 pb16" data-name="Box">
                            <i class="fa fa-2x fa-picture-o rounded bg-secondary s_features_grid_icon"/>
                            <div class="s_features_grid_content">
                                <h4>Sample Icons</h4>
                                <p>All these icons are completely free for commercial use.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_tabs" name="Tabs">
    <section class="s_tabs">
        <div class="container">
            <div class="row s_col_no_bgcolor">
                <div class="col-lg-8 offset-lg-2 pt48 pb48">
                    <div class="card bg-white">
                        <div class="card-header">
                            <ul class="nav nav-tabs card-header-tabs" role="tablist">
                                <li class="nav-item">
                                    <a class="nav-link active"
                                        id="nav_tabs_link_1"
                                        data-toggle="tab"
                                        href="#nav_tabs_content_1"
                                        role="tab"
                                        aria-controls="nav_tabs_content_1"
                                        aria-selected="true">Home</a>
                                </li>
                                <li class="nav-item">
                                    <a class="nav-link"
                                        id="nav_tabs_link_2"
                                        data-toggle="tab"
                                        href="#nav_tabs_content_2"
                                        role="tab"
                                        aria-controls="nav_tabs_content_2"
                                        aria-selected="false">Profile</a>
                                </li>
                                <li class="nav-item">
                                    <a class="nav-link"
                                        id="nav_tabs_link_3"
                                        data-toggle="tab"
                                        href="#nav_tabs_content_3"
                                        role="tab"
                                        aria-controls="nav_tabs_content_3"
                                        aria-selected="false">Contact</a>
                                </li>
                            </ul>
                        </div>
                        <div class="card-body tab-content">
                            <div class="tab-pane fade show active"
                                id="nav_tabs_content_1"
                                role="tabpanel"
                                aria-labelledby="nav_tabs_link_1">
                                <h3>Details</h3>
                                <p><b>Great stories have personality.</b> Consider telling a great story that provides personality. Writing a story with personality for potential clients will assists with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.</p>
                            </div>
                            <div class="tab-pane fade"
                                id="nav_tabs_content_2"
                                role="tabpanel"
                                aria-labelledby="nav_tabs_link_2">
                                <h3>Shipping</h3>
                                <p><b>Great stories are for everyone even when only written for just one person.</b> If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.</p>
                            </div>
                            <div class="tab-pane fade"
                                id="nav_tabs_content_3"
                                role="tabpanel"
                                aria-labelledby="nav_tabs_link_3">
                                <h3>Review</h3>
                                <blockquote>
                                    <p>Write a quote here from one of your customers. Quotes are a great way to build confidence in your products or services.</p>
                                    <footer>— Jane DOE, CEO of <b>MyCompany</b></footer>
                                </blockquote>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_parallax" name="Parallax">
    <section class="s_parallax parallax s_parallax_is_fixed bg-black-50 pt160 pb160" data-scroll-background-ratio="1">
        <span class="s_parallax_bg oe_img_bg oe_custom_bg" style="background-image: url('/web/image/website.s_parallax_default_image'); background-position: 50% 75%;"/>
        <div class="oe_structure">
            <section>
                <div class="container">
                    <div class="row s_nb_column_fixed">
                        <div class="col-lg-12 s_title pt16 pb16" style="text-align: center;">
                            <h1 class="s_title_boxed"><font style="font-size: 62px;">Your Site Title</font></h1>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </section>
</template>

<template id="s_quotes_carousel" name="Quotes">
    <div id="myQuoteCarousel" class="s_quotes_carousel s_carousel_default carousel slide" data-interval="10000">
        <!-- Indicators -->
        <ol class="carousel-indicators">
            <li data-target="#myQuoteCarousel" data-slide-to="0" class="active"></li>
            <li data-target="#myQuoteCarousel" data-slide-to="1"></li>
            <li data-target="#myQuoteCarousel" data-slide-to="2"></li>
        </ol>
        <!-- Content -->
        <div class="carousel-inner">
            <!-- #01 -->
            <div class="carousel-item active bg-200 pt80 pb80" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <blockquote class="carousel-content col-lg-6 bg-white offset-lg-3">
                            <i class="fa fa-1x fa-quote-left rounded-0 bg-secondary s_quotes_carousel_icon"/>
                            <p>Write a quote here from one of your customers. Quotes are a great way to build confidence in your products or services.</p>
                            <footer>
                                <img class="img" src="/web/image/website.s_quotes_carousel_demo_image_3" alt=""/>
                                <span>— Jane DOE, CEO of <b>MyCompany</b></span>
                            </footer>
                        </blockquote>
                    </div>
                </div>
            </div>
            <!-- #02 -->
            <div class="carousel-item oe_img_bg pt80 pb80" style="background-image: url('/web/image/website.s_quotes_carousel_demo_image_1'); background-position: 50% 50%;" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <blockquote class="col-lg-6 bg-white">
                            <p>Odoo provides essential platform for our project management. Things are better organized and more visible with it.</p>
                            <footer>
                                <img class="img" src="/web/image/website.s_quotes_carousel_demo_image_4" alt=""/>
                                <span>— John DOE, CEO of <b>MyCompany</b></span>
                            </footer>
                        </blockquote>
                    </div>
                </div>
            </div>
            <!-- #03 -->
            <div class="carousel-item oe_img_bg pt80 pb80" style="background-image: url('/web/image/website.s_quotes_carousel_demo_image_2'); background-position: 50% 50%;" data-name="Slide">
                <div class="container">
                    <div class="row content">
                        <blockquote class="col-lg-6 offset-lg-6 bg-white">
                            <p>Odoo provides essential platform for our project management. Things are better organized and more visible with it.</p>
                            <footer class="blockquote-footer">
                                <img class="img" src="/web/image/website.s_quotes_carousel_demo_image_5" alt=""/>
                                <span>— Jane DOE, CEO of <b>MyCompany</b></span>
                            </footer>
                        </blockquote>
                    </div>
                </div>
            </div>
        </div>
        <!-- Controls -->
        <div class="carousel-control-prev" data-target="#myQuoteCarousel" data-slide="prev" role="img" aria-label="Previous" title="Previous">
            <span class="carousel-control-prev-icon"/>
            <span class="sr-only">Previous</span>
        </div>
        <div class="carousel-control-next" data-target="#myQuoteCarousel" data-slide="next" role="img" aria-label="Next" title="Next">
            <span class="carousel-control-next-icon"/>
            <span class="sr-only">Next</span>
        </div>
    </div>
</template>

<!-- Mega menu snippets -->

<template id="s_mega_menu_multi_menus" name="Multi-Menus">
    <section class="s_mega_menu_multi_menus py-4">
        <div class="container">
            <div class="row">
                <t t-set="menu1_title">First Menu</t>
                <t t-set="menu2_title">Second Menu</t>
                <t t-set="menu3_title">Third Menu</t>
                <t t-set="menu4_title">Last Menu</t>
                <t t-foreach="[menu1_title, menu2_title, menu3_title, menu4_title]" t-as="menu_title">
                    <div class="col-lg-3 py-2 text-center">
                        <h4 class="o_default_snippet_text" t-esc="menu_title"/>
                        <nav class="nav flex-column">
                            <t t-foreach="3" t-as="i">
                                <t t-set="text">Menu Item %s</t>
                                <t t-set="text" t-value="text % (i + 1)"/>
                                <a href="#" class="nav-link o_default_snippet_text" data-name="Menu Item"
                                   t-esc="text"/>
                            </t>
                        </nav>
                    </div>
                </t>
            </div>
        </div>
    </section>
</template>

<template id="s_mega_menu_menu_image_menu" name="Menu - Image - Menu">
    <section class="s_mega_menu_menu_image_menu py-4">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-4 py-2 text-center">
                    <h4 class="o_default_snippet_text" >Left Menu</h4>
                    <nav class="nav flex-column">
                        <t t-foreach="3" t-as="i">
                            <t t-set="text">Menu Item %s</t>
                            <t t-set="text" t-value="text % (i + 1)"/>
                            <a href="#" class="nav-link o_default_snippet_text" data-name="Menu Item"
                                t-esc="text"/>
                        </t>
                    </nav>
                </div>
                <div class="col-lg-4 py-2 text-center">
                    <img class="img-fluid" src="/web/image/website.s_mega_menu_menu_image_menu_default_image"/>
                </div>
                <div class="col-lg-4 py-2 text-center">
                    <h4 class="o_default_snippet_text">Right Menu</h4>
                    <nav class="nav flex-column">
                        <t t-foreach="3" t-as="i">
                            <t t-set="text">Menu Item %s</t>
                            <t t-set="text" t-value="text % (i + 1)"/>
                            <a href="#" class="nav-link o_default_snippet_text" data-name="Menu Item"
                                t-esc="text"/>
                        </t>
                    </nav>
                </div>
            </div>
        </div>
    </section>
</template>

<!-- Snippets menu -->
<template id="snippets" inherit_id="web_editor.snippets" primary="True">
    <xpath expr="//div[@id='o_scroll']" position="replace">
        <div id="o_scroll">
            <div id="snippet_mega_menu" class="o_panel d-none">
                <div class="o_panel_header">
                    <i class="fa fa-puzzle-piece"/> Mega Menu
                </div>
                <div class="o_panel_body">
                    <t t-snippet="website.s_mega_menu_multi_menus" t-thumbnail="/website/static/src/img/snippets_thumbs/s_mega_menu_multi_menus.png"/>
                    <t t-snippet="website.s_mega_menu_menu_image_menu" t-thumbnail="/website/static/src/img/snippets_thumbs/s_mega_menu_menu_image_menu.png"/>
                </div>
            </div>

            <div id="snippet_structure" class="o_panel">
                <div class="o_panel_header">
                    <i class="fa fa-th-large"/> Structure
                </div>
                <div class="o_panel_body">
                    <t t-snippet="website.s_banner" t-thumbnail="/website/static/src/img/snippets_thumbs/s_banner.png"/>
                    <t t-snippet="website.s_cover" t-thumbnail="/website/static/src/img/snippets_thumbs/s_cover.png"/>
                    <t t-snippet="website.s_text_image" t-thumbnail="/website/static/src/img/snippets_thumbs/s_text_image.png"/>
                    <t t-snippet="website.s_image_text" t-thumbnail="/website/static/src/img/snippets_thumbs/s_image_text.png"/>
                    <t t-snippet="website.s_title" t-thumbnail="/website/static/src/img/snippets_thumbs/s_title.png"/>
                    <t t-snippet="website.s_text_block" t-thumbnail="/website/static/src/img/snippets_thumbs/s_text_block.png"/>
                    <t t-snippet="website.s_picture" t-thumbnail="/website/static/src/img/snippets_thumbs/s_picture.png"/>
                    <t t-snippet="website.s_carousel" t-thumbnail="/website/static/src/img/snippets_thumbs/s_carousel.png"/>
                    <t t-snippet="website.s_features" t-thumbnail="/website/static/src/img/snippets_thumbs/s_features.png"/>
                    <t t-snippet="website.s_three_columns" t-thumbnail="/website/static/src/img/snippets_thumbs/s_three_columns.png"/>
                </div>
            </div>

            <div id="snippet_feature" class="o_panel">
                <div class="o_panel_header">
                    <i class="fa fa-diamond"/> Features
                </div>
                <div class="o_panel_body">
                    <t t-snippet="website.s_image_gallery" t-thumbnail="/website/static/src/img/snippets_thumbs/s_image_gallery.png"/>
                    <t t-snippet="website.s_comparisons" t-thumbnail="/website/static/src/img/snippets_thumbs/s_comparison.png"/>
                    <t t-snippet="website.s_company_team" t-thumbnail="/website/static/src/img/snippets_thumbs/s_company_team.png"/>
                    <t t-snippet="website.s_call_to_action" t-thumbnail="/website/static/src/img/snippets_thumbs/s_call_to_action.png"/>
                    <t t-snippet="website.s_references" t-thumbnail="/website/static/src/img/snippets_thumbs/s_references.png"/>
                    <t t-snippet="website.s_faq_collapse" t-thumbnail="/website/static/src/img/snippets_thumbs/s_faq.png"/>
                    <t t-snippet="website.s_features_grid" t-thumbnail="/website/static/src/img/snippets_thumbs/s_features_grid.png"/>
                    <t t-snippet="website.s_tabs" t-thumbnail="/website/static/src/img/snippets_thumbs/s_tabs.png"/>
                </div>
            </div>

            <div id="snippet_effect" class="o_panel">
                <div class="o_panel_header">
                    <i class="fa fa-magic icon-fix"/> Effects
                </div>
                <div class="o_panel_body">
                    <t t-snippet="website.s_parallax" t-thumbnail="/website/static/src/img/snippets_thumbs/s_parallax.png"/>
                    <t t-snippet="website.s_quotes_carousel" t-thumbnail="/website/static/src/img/snippets_thumbs/s_quotes_carousel.png"/>
                </div>
            </div>

            <div id="snippet_content" class="o_panel">
                <div class="o_panel_header">
                    <i class="fa fa-indent"/> Inner content
                </div>
                <div class="o_panel_body">
                    <t t-snippet="website.s_btn" t-thumbnail="/website/static/src/img/snippets_thumbs/s_button.png"/>
                    <t t-snippet="website.s_hr" t-thumbnail="/website/static/src/img/snippets_thumbs/s_separator.png"/>
                    <t t-snippet="website.s_alert" t-thumbnail="/website/static/src/img/snippets_thumbs/s_alert.png"/>
                    <t t-snippet="website.s_card" t-thumbnail="/website/static/src/img/snippets_thumbs/s_card.png"/>
                    <t t-snippet="website.s_share" t-thumbnail="/website/static/src/img/snippets_thumbs/s_share.png"/>
                    <t t-snippet="website.s_facebook_page" t-thumbnail="/website/static/src/img/snippets_thumbs/s_facebook_page.png"/>
                    <t t-snippet="website.s_rating" t-thumbnail="/website/static/src/img/snippets_thumbs/s_rating.png"/>
                </div>
            </div>
        </div>
    </xpath>

    <xpath expr="//div[@id='snippet_options']/t" position="attributes">
        <attribute name="t-call">website.snippet_options</attribute>
    </xpath>
</template>

<template id="external_snippets" inherit_id="website.snippets" priority="8">
    <xpath expr="//div[@id='snippet_feature']//t[@t-snippet][last()]" position="after">
        <t id="newsletter_popup_snippet" t-install="mass_mailing" string="Newsletter Popup" t-thumbnail="/website/static/src/img/snippets_thumbs/newsletter_subscribe_popup.png"/>
        <t id="newsletter_block_snippet" t-install="mass_mailing" string="Newsletter Block" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_block.png"/>
        <t t-install="website_twitter" string="Twitter Scroller" t-thumbnail="/website/static/src/img/snippets_thumbs/s_twitter_scroll.png"/>
        <t t-install="website_form" string="Form Builder" t-thumbnail="/website/static/src/img/s_website_form.png"/>
        <t id="s_products_searchbar" t-install="website_sale" string="Products Search" t-thumbnail="/website/static/src/img/snippets_thumbs/s_products_searchbar.png"/>
        <t id="s_products_recently_viewed" t-install="website_sale" string="Products Recently Viewed" t-thumbnail="/website/static/src/img/snippets_thumbs/s_products_recently_viewed.png"/>
    </xpath>
    <xpath expr="//div[@id='snippet_content']//t[@t-snippet][last()]" position="after">
        <t t-install="website_event" string="Local Events" t-thumbnail="/website/static/src/img/snippets_thumbs/s_local_events.png"/>
        <t t-install="website_mail_channel" string="Discussion Group" t-thumbnail="/website/static/src/img/snippets_thumbs/s_button_channel_subscribe.png"/>
        <t id="newsletter_snippet" t-install="mass_mailing" string="Newsletter" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_subscribe_form.png"/>
        <t id="s_products_searchbar_input" t-install="website_sale" string="Products Search Input" t-thumbnail="/website/static/src/img/snippets_thumbs/s_products_searchbar.png"/>
    </xpath>
</template>

<template id="snippet_options">
    <t t-call="web_editor.snippet_options"/>

    <!-- COLOR | .s_three_columns | .s_comparisons -->
    <div data-js="colorpicker"
        data-selector=".s_three_columns .row > div, .s_comparisons .row > div, .s_tabs .row > div"
        data-target=".card"
        data-palette-title="Box Color">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- COLOR | .s_hr -->
    <div data-js="colorpicker"
        data-selector=".s_hr"
        data-target="hr"
        data-color-prefix="border-"
        data-palette-exclude="transparent_grayscale, common"
        data-palette-title="Separator Color">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- COLOR | .s_cards -->
    <div data-js="colorpicker"
        data-selector=".s_card, .accordion .card"
        data-palette-title="Card Color">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- COLOR | .s_alert -->
    <div data-js="colorpicker"
        data-selector=".s_alert"
        data-color-prefix="alert-"
        data-palette-title="Box Color">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>


    <!-- STYLES | .s_title -->
    <div data-selector=".s_title" data-target="[class*='s_title_']">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-font"/> Typography</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_title_boxed">Boxed</we-button>
                <we-button data-select-class="s_title_lines">Line-On-Sides</we-button>
                <we-button data-select-class="s_title_thin">Thin</we-button>
                <we-button data-select-class="s_title_transparent">Transparent</we-button>
                <we-button data-select-class="s_title_underlined">Underlined</we-button>
                <we-button data-select-class="s_title_small_caps">Small Caps</we-button>
                <we-divider/>
                <we-button data-select-class="s_title_default">Default</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- H-ALIGN | .s_btn -->
    <div data-selector=".s_btn, .s_share">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-align-left"/> Alignment</we-toggler>
            <we-collapse>
                <we-button data-select-class="text-left">Left</we-button>
                <we-button data-select-class="text-center">Center</we-button>
                <we-button data-select-class="text-right">Right</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-selector=".s_alert">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-arrows"/> Size</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_alert_sm">Small</we-button>
                <we-button data-select-class="s_alert_md">Medium</we-button>
                <we-button data-select-class="s_alert_lg">Large</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-selector=".s_alert, .s_card">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-arrows-h"/> Width</we-toggler>
            <we-collapse>
                <we-button data-select-class="w-25">25%</we-button>
                <we-button data-select-class="w-50">50%</we-button>
                <we-button data-select-class="w-75">75%</we-button>
                <we-button data-select-class="w-100">100%</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- STYLES | .s_rating -->
    <div data-selector=".s_rating > div">
        <we-collapse-area>
            <we-toggler>Icon</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_rating_stars"><i class="fa fa-fw fa-star"/> Stars</we-button>
                <we-button data-select-class="s_rating_squares"><i class="fa fa-fw fa-square"/> Squares</we-button>
                <we-button data-select-class="s_rating_hearts"><i class="fa fa-fw fa-heart"/> Hearts</we-button>
                <we-button data-select-class="s_rating_bar"><i class="fa fa-fw fa-tasks"/> Progress Bar</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Value</we-toggler>
            <we-collapse>
                <we-button data-select-class="">0</we-button>
                <we-button data-select-class="s_rating_1">1/5</we-button>
                <we-button data-select-class="s_rating_2">2/5</we-button>
                <we-button data-select-class="s_rating_3">3/5</we-button>
                <we-button data-select-class="s_rating_4">4/5</we-button>
                <we-button data-select-class="s_rating_5">5/5</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Size</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_rating_1x">Small</we-button>
                <we-button data-select-class="s_rating_2x">Medium</we-button>
                <we-button data-select-class="s_rating_3x">Big</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- STYLES | .s_hr -->
    <div data-selector=".s_hr" data-target="hr">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-minus"/> Thickness</we-toggler>
            <we-collapse>
                <t t-foreach="range(1,6)" t-as="s_hr_thickness">
                    <we-button t-attf-data-select-class="s_hr_{{s_hr_thickness}}px"><t t-esc="s_hr_thickness"/>px</we-button>
                </t>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-pencil"/> Style</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_hr_solid">Solid</we-button>
                <we-button data-select-class="s_hr_dashed">Dashed</we-button>
                <we-button data-select-class="s_hr_dotted">Dotted</we-button>
                <we-button data-select-class="s_hr_double">Double</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-arrows-h"/> Width</we-toggler>
            <we-collapse>
                <we-button data-select-class="w-25">25%</we-button>
                <we-button data-select-class="w-50">50%</we-button>
                <we-button data-select-class="w-75">75%</we-button>
                <we-button data-select-class="w-100">100%</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-align-left"/> Alignment</we-toggler>
            <we-collapse>
                <we-button data-select-class="mr-auto">Left</we-button>
                <we-button data-select-class="mx-auto">Center</we-button>
                <we-button data-select-class="ml-auto">Right</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- Carousel | .s_carousel | .s_quotes_carousel -->
    <div data-js="carousel"
        data-selector=":not(.o_gallery > .container) > .carousel">
        <we-button data-add-slide="true" data-no-preview="true">
            <i class="fa fa-fw fa-plus-circle"/> Add Slide
        </we-button>
        <we-button data-remove-slide="true" data-no-preview="true">
            <i class="fa fa-fw fa-trash-o"/> Remove Slide
        </we-button>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-clone"/> Transition</we-toggler>
            <we-collapse>
                <we-button data-select-class="slide">Slide</we-button>
                <we-button data-select-class="carousel-fade slide">Fade</we-button>
                <we-divider/>
                <we-button data-select-class="">None</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-magic"/> Style</we-toggler>
            <we-collapse>
                <we-button data-select-class="s_carousel_bordered">Bordered</we-button>
                <we-button data-select-class="s_carousel_boxed">Boxed</we-button>
                <we-button data-select-class="s_carousel_rounded">Rounded</we-button>
                <we-divider/>
                <we-button data-select-class="s_carousel_default">Default</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-clock-o"/> Speed</we-toggler>
            <we-collapse>
                <we-button data-interval="1000">1s</we-button>
                <we-button data-interval="2000">2s</we-button>
                <we-button data-interval="3000">3s</we-button>
                <we-button data-interval="5000">5s</we-button>
                <we-button data-interval="10000">10s</we-button>
                <we-button data-interval="0">Disable autoplay</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-js="navTabs" data-selector="section .row > div" data-target=".nav-tabs">
        <we-button data-add-tab="1" data-no-preview="true">
            <i class="fa fa-fw fa-plus"/> Add Tab
        </we-button>
        <we-button data-remove-tab="1" data-no-preview="true">
            <i class="fa fa-fw fa-minus"/> Remove Tab
        </we-button>
    </div>

    <div data-js="gallery" data-selector=".o_gallery">
        <we-button data-add-images="true" data-no-preview="true">
            <i class="fa fa-fw fa-plus-circle"/> Add images
        </we-button>
        <we-button data-remove-all-images="true" data-no-preview="true">
            <i class="fa fa-fw fa-trash"/> Remove all images
        </we-button>
        <we-divider/>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-magic"/> Mode</we-toggler>
            <we-collapse>
                <we-button data-mode="nomode">Float</we-button>
                <we-button data-mode="masonry">Masonry</we-button>
                <we-button data-mode="grid">Grid</we-button>
                <we-button data-mode="slideshow">Slideshow</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-clock-o"/> Slideshow speed</we-toggler>
            <we-collapse>
                <we-button data-interval="1000">1s</we-button>
                <we-button data-interval="2000">2s</we-button>
                <we-button data-interval="3000">3s</we-button>
                <we-button data-interval="5000">5s</we-button>
                <we-button data-interval="10000">10s</we-button>
                <we-button data-interval="0">Disable autoplay</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-th"/> Columns</we-toggler>
            <we-collapse>
                <we-button data-columns="1">1</we-button>
                <we-button data-columns="2">2</we-button>
                <we-button data-columns="3">3</we-button>
                <we-button data-columns="4">4</we-button>
                <we-button data-columns="6">6</we-button>
                <we-button data-columns="12">12</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area class="o_w_image_spacing_option">
            <we-toggler><i class="fa fa-fw fa-arrows-h"/> Images spacing</we-toggler>
            <we-collapse>
                <we-button data-select-class="o_spc-none">None</we-button>
                <we-button data-select-class="o_spc-small">Small</we-button>
                <we-button data-select-class="o_spc-medium">Medium</we-button>
                <we-button data-select-class="o_spc-big">Big</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-paint-brush"/> Styling</we-toggler>
            <we-collapse>
                <we-button data-styling="">Square</we-button>
                <we-button data-styling="rounded">Rounded corners</we-button>
                <we-button data-styling="img-thumbnail">Thumbnails</we-button>
                <we-button data-styling="rounded-circle">Circle</we-button>
                <we-button data-styling="shadow">Shadows</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-js="gallery_img" data-selector=".o_gallery img">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-refresh"/> Re-order</we-toggler>
            <we-collapse data-no-preview="true">
                <we-button data-position="first">Move to first</we-button>
                <we-button data-position="prev">Move to previous</we-button>
                <we-button data-position="next">Move to next</we-button>
                <we-button data-position="last">Move to last</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- Facebook Page -->
    <div data-js="facebookPage" data-selector=".o_facebook_page">
        <we-button data-fb-page-options="true" data-no-preview="true">
            <i class="fa fa-fw fa-facebook"/> Options
        </we-button>
    </div>

    <!-- Accordion -->
    <div data-js="collapse"
         data-selector='.accordion > .card'
         data-drop-in='.accordion:has(> .card)'/>

    <div data-js="layout_column"
        data-selector="section"
        data-target="> * > .row:not(.s_nb_column_fixed)">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-columns"/> Number of columns</we-toggler>
            <we-collapse data-no-preview="true">
                <we-button data-select-count="1">1</we-button>
                <we-button data-select-count="2">2</we-button>
                <we-button data-select-count="3">3</we-button>
                <we-button data-select-count="4">4</we-button>
                <we-button data-select-count="5">5</we-button>
                <we-button data-select-count="6">6</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!--  V-ALIGN -->
    <div id="row_valign_snippet_option" data-selector=".s_text_image, .s_image_text, .s_three_columns" data-target=".row">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-arrows-v"/> Alignment</we-toggler>
            <we-collapse>
                <we-button data-select-class="align-items-start">Top</we-button>
                <we-button data-select-class="align-items-center">Middle</we-button>
                <we-button data-select-class="align-items-end">Bottom</we-button>
                <we-divider/>
                <we-button data-select-class="align-items-stretch">Equal height</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- Background Image -->
    <div data-js="background"
        data-selector="section, .parallax, :not(.o_gallery > .container) > .carousel"
        data-exclude=".s_hr, .s_image_gallery">
        <we-button data-choose-image="true" data-no-preview="true">
            <i class="fa fa-fw fa-picture-o"/> Background
        </we-button>
    </div>

    <!-- Background Image -->
    <div data-js="background_position"
        data-selector="section, .parallax, :not(.o_gallery > .container) > .carousel">
        <we-button data-background-position="true" data-no-preview="true">
            <i class="fa fa-fw fa-arrows"/> Background Image Sizing
        </we-button>
    </div>

    <!-- Parallax -->
    <div data-js="parallax" data-selector=".parallax">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-clock-o"/> Scroll Speed</we-toggler>
            <we-collapse>
                <we-button data-scroll="0">No-scroll</we-button>
                <we-divider/>
                <we-button data-scroll="1">Fixed</we-button>
                <we-divider/>
                <we-button data-scroll="0.6">Very Slow</we-button>
                <we-button data-scroll="1.2">Slow</we-button>
                <we-button data-scroll="1.6">Fast</we-button>
                <we-button data-scroll="2">Very Fast</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <!-- Color | Section -->
    <div id="so_main_colorpicker"
        data-js="colorpicker"
        data-selector="section, :not(.o_gallery > .container) > .carousel"
        data-exclude=".parallax">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Background Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- FILTER | .s_parallax -->
    <div data-js="colorpicker"
        data-selector=".parallax"
        data-palette-exclude="common, theme"
        data-palette-default="transparent_grayscale"
        data-palette-title="Overlay Color">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Filter</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- Color | Columns -->
    <div data-js="colorpicker"
        data-selector="section .row > div"
        data-exclude=".s_col_no_bgcolor, .s_col_no_bgcolor.row > div, .o_gallery .row > div">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Background Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <div data-selector="section .row > div"
        data-exclude=".s_col_no_bgcolor, .s_col_no_bgcolor.row > div, .o_gallery .row > div">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-magic"/> Styles</we-toggler>
            <we-collapse>
                <we-button data-toggle-class="border">Border</we-button>
                <we-button data-toggle-class="rounded">Rounded</we-button>
                <we-button data-toggle-class="shadow">Shadow</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-js="sizing_y"
        data-selector="section, .row > div, :not(.o_gallery > .container) > .carousel, .parallax, .s_hr, .s_btn"/>

    <div data-js="sizing_x"
        data-selector=".row > div"
        data-drop-near=".row > div"
        data-exclude=".s_col_no_resize.row > div"/>

    <div id="so_snippet_addition"
        data-selector="section, :not(.o_gallery > .container) > .carousel, .parallax"
        data-drop-in=":not(p).oe_structure:not(.oe_structure_solo), :not(p)[data-oe-type=html], :not(p).oe_structure.oe_structure_solo:not(:has(> section, > div))"/>

    <!-- Main content drop -->
    <div id="so_content_addition"
        data-selector="blockquote, .s_btn, .s_card, .s_alert, .o_facebook_page, .s_share, .s_rating, .s_hr"
        data-drop-near="p, h1, h2, h3, blockquote, .s_btn, .s_card, .s_alert, .o_facebook_page, .s_share, .s_rating, .s_hr"
        data-drop-in=".content, nav"/>

    <div data-js="ul"
         data-selector=":not(li) > ul:has(ul,ol), :not(li) > ol:has(ul,ol)">
        <we-button data-toggle-class="o_ul_folded">Folded list</we-button>
    </div>

    <div data-js="menu_data"
         data-selector="#top_menu li > a"
         data-exclude=".dropdown-toggle"
         data-no-check="true"/>

    <div data-js="company_data"
         data-selector="[data-oe-expression='res_company.partner_id']"
         data-no-check="true"/>

    <div data-js="topMenuTransparency"
        data-selector="[data-main-object^='website.page('] #wrapwrap > header"
        data-no-check="true">
        <we-button data-transparent="true" data-no-preview="true">
            <i class="fa fa-fw fa-eye-slash"/> Transparent
        </we-button>
    </div>

    <div data-js="topMenuColor"
        data-selector="[data-main-object^='website.page('] #wrapwrap > header"
        data-palette-exclude="theme,common"
        data-palette-default="transparent_grayscale"
        data-palette-title="Top Menu Color"
        data-no-check="true">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-eyedropper"/> Background Color</we-toggler>
            <we-collapse/>
        </we-collapse-area>
    </div>

    <!-- Anchor Name -->
    <div data-js="anchorName"
        data-selector=":not(p).oe_structure > *, :not(p)[data-oe-type=html] > *"
        data-exclude=".modal *">
        <we-button data-no-preview="true" data-open-anchor-dialog="">
            <i class="fa fa-fw fa-anchor"/> Link Anchor
        </we-button>
    </div>

    <!-- Mega Menu settings -->
    <div data-selector=".o_mega_menu">
        <we-collapse-area>
            <we-toggler><i class="fa fa-fw fa-arrows-h"/> Size</we-toggler>
            <we-collapse>
                <we-button data-select-class="">Full-Width</we-button>
                <we-button data-select-class="o_mega_menu_container_size">Narrow</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>

    <div data-selector=".o_mega_menu .nav-link"
         data-drop-in=".o_mega_menu nav"
         data-drop-near=".o_mega_menu .nav-link"/>

    <div data-js="CoverProperties" data-selector=".o_record_cover_container" data-no-check="true">
        <we-button data-change="true" data-no-preview="true">Change Cover</we-button>
        <we-button data-clear="true" data-no-preview="true">Remove Cover</we-button>
        <we-collapse-area>
            <we-toggler>Size</we-toggler>
            <we-collapse>
                <we-button data-cover-opt="size" data-select-class="o_record_has_cover cover_full">
                    Full Screen
                </we-button>
                <we-button data-cover-opt="size" class="o_record_cover_opt_size_default" data-select-class="o_record_has_cover cover_mid">
                    Half Screen
                </we-button>
                <we-button data-cover-opt="size" data-select-class="o_record_has_cover cover_auto">
                    Fit text
                </we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Filter Intensity</we-toggler>
            <we-collapse>
                <we-button data-cover-opt="filters" data-filter-value="0.0">None</we-button>
                <we-button data-cover-opt="filters" data-filter-value="0.2">Low</we-button>
                <we-button data-cover-opt="filters" data-filter-value="0.4">Medium</we-button>
                <we-button data-cover-opt="filters" data-filter-value="0.6">High</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Filter Color</we-toggler>
            <we-collapse>
                <we-button data-cover-opt="filters" data-filter-color="oe_black">Black</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_none">White</we-button>
                <we-button data-cover-opt="filters" data-filter-color="bg-primary">Primary</we-button>
                <we-button data-cover-opt="filters" data-filter-color="bg-secondary">Secondary</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_blue">Blue</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_yellow">Yellow</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_red">Red</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_purple">Purple</we-button>
                <we-button data-cover-opt="filters" data-filter-color="oe_green">Green</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Text Size</we-toggler>
            <we-collapse>
                <we-button data-cover-opt="text_size" data-select-class="o_record_cover_font_hero">Hero</we-button>
                <we-button data-cover-opt="text_size" data-select-class="o_record_cover_font_huge">Huge</we-button>
                <we-button data-cover-opt="text_size" data-select-class="o_record_cover_font_big">Big</we-button>
                <we-button data-cover-opt="text_size" data-select-class="">Regular</we-button>
                <we-button data-cover-opt="text_size" data-select-class="o_record_cover_font_small">Small</we-button>
                <we-button data-cover-opt="text_size" data-select-class="o_record_cover_font_tiny">Tiny</we-button>
            </we-collapse>
        </we-collapse-area>
        <we-collapse-area>
            <we-toggler>Text Alignment</we-toggler>
            <we-collapse>
                <we-button data-cover-opt="text_align" data-select-class="">Left</we-button>
                <we-button data-cover-opt="text_align" data-select-class="text-center">Centered</we-button>
                <we-button data-cover-opt="text_align" data-select-class="text-right">Right</we-button>
            </we-collapse>
        </we-collapse-area>
    </div>
</template>
</odoo>

```

## File: views\website_navbar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Front-end/Back-end integration -->
    <template id="user_navbar" inherit_id="website.layout">
        <xpath expr="//t[@t-set='html_data']" position="after">
            <t t-set="body_classname" t-value="(body_classname if body_classname else '') + (' o_connected_user' if env['ir.ui.view'].user_has_groups('base.group_user') else '')"/>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']" position="before">
            <nav groups="base.group_user" t-if="website" id="oe_main_menu_navbar" class="o_main_navbar">
                <ul id="oe_applications">
                    <li class="dropdown active">
                        <a class="dropdown-toggle full" data-toggle="dropdown" href="#">
                            <i class="fa fa-th-large"/> WEBSITE
                        </a>
                        <div class="dropdown-menu" role="menu">
                            <!-- This will be populated on hover in JS -->
                        </div>
                    </li>
                </ul>

                <button type="button" class="fa fa-bars float-right d-block d-md-none o_mobile_menu_toggle" aria-label="Menu" title="Menu"/>

                <ul class="o_menu_sections" groups="website.group_website_designer">
                    <!-- Content -->
                    <li t-if="editable" class="dropdown" id="content-menu">
                        <a id="content-menu-button" class="dropdown-toggle o-no-caret waves" data-toggle="dropdown" data-display="static" href="#">Pages</a>
                        <div class="dropdown-menu" role="menu">
                            <a role="menuitem" data-action="edit_menu" href="#" title="Edit Top Menu" class="dropdown-item">Edit Menu</a>
                            <a role="menuitem" href="/website/pages" title="Manage Your Website Pages" class="dropdown-item">Manage Pages</a>
                            <div t-if="deletable" role="separator" class="dropdown-divider"/>
                            <a role="menuitem" href="#" data-action="page_properties" class="dropdown-item" t-if="deletable">Page Properties</a>
                        </div>
                    </li>
                    <!-- Customize -->
                    <li class="dropdown" id="customize-menu">
                        <a class="dropdown-toggle o-no-caret waves" data-toggle="dropdown" data-display="static" href="#">Customize</a>
                        <div class="dropdown-menu" role="menu">
                            <a role="menuitem" href="#" data-action="customize_theme" class="dropdown-item" id="theme_customize">Customize Theme</a>
                            <a role="menuitem" href="#" data-action="ace" class="dropdown-item" id="html_editor">HTML/CSS/JS Editor</a>
                            <a role="menuitem" href="/web#action=website.action_website_add_features" class="dropdown-item" id="install_apps">Add Features</a>
                        </div>
                    </li>
                    <!-- Promote -->
                    <li class="dropdown" id="promote-menu">
                        <a class="dropdown-toggle o-no-caret waves" data-toggle="dropdown" href="#">Promote</a>
                        <div class="dropdown-menu oe_promote_menu" role="menu">
                            <a role="menuitem" data-action="promote-current-page" href="#" title="Promote page on the web" class="dropdown-item">Optimize SEO</a>
                        </div>
                    </li>
                </ul>

                <ul class="o_menu_systray d-none d-md-block" groups="website.group_website_publisher">
                    <li t-if="'website_published' in main_object.fields_get() and ('can_publish' not in main_object.fields_get() or main_object.can_publish)" t-attf-class="js_publish_management #{main_object.website_published and 'css_published' or 'css_unpublished'}" t-att-data-id="main_object.id" t-att-data-object="main_object._name" t-att-data-controller="publish_controller">
                        <label class="o_switch o_switch_danger js_publish_btn" for="id">
                            <input type="checkbox" disabled="disabled" t-att-checked="main_object.website_published" id="id"/>
                            <span/>
                            <span class="css_publish">Unpublished</span>
                            <span class="css_unpublish">Published</span>
                        </label>
                    </li>
                    <!-- Mobile preview -->
                    <li class="o_mobile_preview" id="mobile-menu">
                        <a data-action="show-mobile-preview" href="#"><span title="Mobile preview" role="img" aria-label="Mobile preview" class="fa fa-mobile"/></a>
                    </li>
                    <li groups="website.group_multi_website" t-if="multi_website_websites">
                        <a class="dropdown-toggle" data-toggle="dropdown" href="#">
                            <i class="fa fa-globe d-lg-none"/>
                            <span class="d-none d-lg-inline-block">
                                <t t-esc="multi_website_websites_current['name']"/>
                            </span>
                        </a>
                        <div class="dropdown-menu" role="menu">
                            <div class="d-lg-none dropdown-item active">
                                <span t-esc="multi_website_websites_current['name']"/>
                            </div>
                            <t t-foreach="multi_website_websites" t-as="multi_website_website">
                                <a role="menuitem" href="#"
                                    t-att-domain="multi_website_website['domain']"
                                    class="dropdown-item oe_menu_text js_multi_website_switch"
                                    t-att-website-id="str(multi_website_website['website_id'])"
                                >
                                    <span t-esc="multi_website_website['name']" />
                                </a>
                            </t>
                        </div>
                    </li>

                    <!-- Page Edition -->
                    <li class="o_new_content_menu" id="new-content-menu">
                        <a href="#"><span class="fa fa-plus mr-2"/>New</a>
                        <div id="o_new_content_menu_choices" class="o_hidden">
                            <div class="container pt32 pb32">
                                <div class="row">
                                    <div groups="website.group_website_designer" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_page" aria-label="New page" title="New page">
                                            <i class="fa fa-file-o"/>
                                            <p>New Page</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_blog" t-att-data-module-id="env.ref('base.module_website_blog').id" t-att-data-module-shortdesc="env.ref('base.module_website_blog').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_blog_post">
                                            <i class="fa fa-rss"/>
                                            <p>New Blog Post</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_event" t-att-data-module-id="env.ref('base.module_website_event').id" t-att-data-module-shortdesc="env.ref('base.module_website_event').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_event">
                                            <i class="fa fa-glass"/>
                                            <p>New Event</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_forum" t-att-data-module-id="env.ref('base.module_website_forum').id" t-att-data-module-shortdesc="env.ref('base.module_website_forum').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_forum">
                                            <i class="fa fa-comment"/>
                                            <p>New Forum</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_hr_recruitment" t-att-data-module-id="env.ref('base.module_website_hr_recruitment').id" t-att-data-module-shortdesc="env.ref('base.module_website_hr_recruitment').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#">
                                            <i class="fa fa-briefcase"/>
                                            <p>New Job Offer</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_sale" t-att-data-module-id="env.ref('base.module_website_sale').id" t-att-data-module-shortdesc="env.ref('base.module_website_sale').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_product">
                                            <i class="fa fa-shopping-cart"/>
                                            <p>New Product</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_slides" t-att-data-module-id="env.ref('base.module_website_slides').id" t-att-data-module-shortdesc="env.ref('base.module_website_slides').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_slide_channel">
                                            <i class="fa fa-graduation-cap"></i>
                                            <p>New Course</p>
                                        </a>
                                    </div>
                                    <div groups="base.group_system" name="module_website_livechat" t-att-data-module-id="env.ref('base.module_website_livechat').id" t-att-data-module-shortdesc="env.ref('base.module_website_livechat').shortdesc" class="col-md-4 mb8 o_new_content_element">
                                        <a href="#" data-action="new_channel">
                                            <i class="fa fa-hashtag"/>
                                            <p>New Livechat Channel</p>
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </li>
                    <li t-if="not translatable" id="edit-page-menu">
                        <a data-action="edit" href="#"><span class="fa fa-pencil mr-2"/>Edit</a>
                    </li>
                    <li t-if="edit_in_backend or ('website_published' in main_object.fields_get() and main_object._name != 'website.page')">
                        <a role="button" class="btn btn-primary btn-sm dropdown-toggle css_edit_dynamic" data-toggle="dropdown">
                            <span class="sr-only">Toggle Dropdown</span>
                        </a>
                        <div class="dropdown-menu" role="menu">
                            <a role="menuitem" style="text-align: left;" t-attf-href="/web#view_type=form&amp;model=#{main_object._name}&amp;id=#{main_object.id}&amp;action=#{action}&amp;menu_id=#{backend_menu_id}"
                                   class="dropdown-item" title='Edit in backend' id="edit-in-backend">Edit in backend</a>
                        </div>
                    </li>
                    <li t-if="translatable">
                        <a data-action="translate" href="#">TRANSLATE</a>
                    </li>
                    <li t-if="translatable">
                        <a data-action="edit_master" href="#">or Edit Master</a>
                    </li>
                </ul>
            </nav>
        </xpath>
    </template>
</odoo>

```

## File: views\website_rewrite.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Attachment -->
        <record id="view_website_rewrite_form" model="ir.ui.view">
            <field name="model">website.rewrite</field>
            <field name="arch" type="xml">
                <form string="Website rewrite Settings">
                    <header>
                        <button name="refresh_routes" string="Refresh route's list" type="object"
                                class="btn-primary"
                                attrs="{'invisible':[('redirect_type', '!=', '308')]}"
                        />
                    </header>
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="redirect_type"/>
                                <field name="url_from" attrs="{'invisible': [('redirect_type', '=', '308')]}"/>
                                <field name="route_id" string="URL from" options="{'no_create': True, 'no_open': True}" attrs="{'invisible': [('redirect_type', '!=', '308')]}"/>
                                <field name="url_to" attrs="{'invisible': [('redirect_type', '=', '404')]}"/>
                            </group>
                            <group>
                                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                                <field name="active"/>
                                <field name="sequence" groups="base.group_no_one"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_website_rewrite_tree" model="ir.ui.view">
            <field name="name">website.rewrite.list</field>
            <field name="model">website.rewrite</field>
            <field name="arch" type="xml">
                <tree string="Website rewrites">
                    <field name="redirect_type"/>
                    <field name="name"/>
                    <field name="url_from"/>
                    <field name="url_to"/>
                    <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                    <field name="active"/>
                    <field name="sequence" widget="handle" />
                </tree>
            </field>
        </record>


        <record id="action_website_rewrite_list" model="ir.actions.act_window">
            <field name="name">Rewrite</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">website.rewrite</field>
            <field name="view_id" eval="False"/>
        </record>

        <menuitem name="Redirects"
            id="menu_website_rewrite"
            action="action_website_rewrite_list"
            parent="menu_website_global_configuration"
            sequence="30"
            groups="base.group_no_one"/>

        <record id="view_rewrite_search" model="ir.ui.view">
            <field name="name">website.rewrite.search</field>
            <field name="model">website.rewrite</field>
            <field name="arch" type="xml">
                <search string="Search Redirect">
                    <field name="url_from"/>
                    <field name="url_to"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                    <group string="Group By">
                        <filter string="Redirection Type" name="group_by_type" domain="[]" context="{'group_by': 'redirect_type'}"/>
                    </group>
                </search>
            </field>
        </record>
</odoo>

```

## File: views\website_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<!-- Assets -->
<template id="_assets_primary_variables" inherit_id="portal._assets_primary_variables">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/primary_variables.scss"/>

        <!-- Those files will be automatically edited by users -->
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/options/user_values.scss"/>
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/options/colors/user_color_palette.scss"/>
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/options/colors/user_theme_color_palette.scss"/>
    </xpath>
</template>
<template id="_assets_secondary_variables" inherit_id="web_editor._assets_secondary_variables">
    <xpath expr="//link" position="before">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/secondary_variables.scss"/>
    </xpath>
</template>

<template id="assets_tests" name="Website Assets Tests" inherit_id="web.assets_tests">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/website/static/tests/tours/carousel_content_removal.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/reset_password.js"></script>
        <script type="text/javascript" src="/website/static/tests/tours/rte.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/focus_blur_snippets.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/html_editor.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/restricted_editor.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/dashboard_tour.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/website_navbar_menu.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/specific_website_editor.js"/>
        <script type="text/javascript" src="/website/static/tests/tours/public_user_editor.js"/>
    </xpath>
</template>

<template id="assets_backend" inherit_id="web.assets_backend" name="Website Backend Assets (used in backend interface)">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.backend.scss"/>
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website_visitor_views.scss"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website/static/src/js/backend/button.js"/>
        <script type="text/javascript" src="/website/static/src/js/backend/dashboard.js"/>
        <script type="text/javascript" src="/website/static/src/js/backend/res_config_settings.js"/>
    </xpath>
</template>

<template id="qunit_suite" inherit_id="web.qunit_suite">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website/static/tests/website_tests.js"/>
    </xpath>
</template>

<template id="_assets_frontend_helpers" inherit_id="portal._assets_frontend_helpers">
    <xpath expr="//link" position="before">
        <!-- Custom empty file for user custom bootstrap override -->
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/user_custom_bootstrap_overridden.scss"/>

        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/bootstrap_overridden.scss"/>
    </xpath>
</template>

<template id="assets_frontend" inherit_id="portal.assets_frontend" name="Website Assets">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.scss"/>
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.ui.scss"/>

        <!-- Custom empty file for user custom rules -->
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/user_custom_rules.scss"/>
    </xpath>
    <xpath expr="//script[@src='/web/static/src/js/public/public_root_instance.js']" position="replace">
        <script type="text/javascript" src="/website/static/src/js/content/website_root_instance.js"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website/static/lib/jstz.min.js"/>
        <script type="text/javascript" src="/website/static/src/js/utils.js"/>

        <script type="text/javascript" src="/website/static/src/js/content/website_root.js"/>
        <script type="text/javascript" src="/website/static/src/js/content/compatibility.js"/>
        <script type="text/javascript" src="/website/static/src/js/content/lazy_template_call.js"/>
        <script type="text/javascript" src="/website/static/src/js/content/menu.js"/>
        <script type="text/javascript" src="/website/static/src/js/content/snippets.animation.js"/>

        <script type="text/javascript" src="/website/static/src/js/menu/navbar.js"/>

        <script type="text/javascript" src="/website/static/src/js/visitor_timezone.js"/>
        <!-- Custom empty file for user javascript -->
        <script type="text/javascript" src="/website/static/src/js/user_custom_javascript.js"/>
    </xpath>
</template>

<template id="assets_frontend_compatibility_for_12_0" inherit_id="website.assets_frontend" active="False">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/compatibility/bs3_for_12_0.scss"/>
    </xpath>
</template>

<template id="website.compiled_assets_wysiwyg" name="Website Editor Assets (used in website editor)">
    <t t-call-assets="website.assets_wysiwyg"/>
</template>

<template id="website.assets_wysiwyg" name="Website Editor Assets (used in website editor)">
    <t t-call="web._assets_helpers">
        <link rel="stylesheet" type="text/scss" href="/web_editor/static/src/scss/bootstrap_overridden.scss"/>
    </t>
    <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.wysiwyg.scss"/>
    <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.edit_mode.scss"/>

    <script type="text/javascript" src="/website/static/src/js/editor/editor.js"/>
    <script type="text/javascript" src="/website/static/src/js/editor/rte.summernote.js"/>
    <script type="text/javascript" src="/website/static/src/js/editor/snippets.options.js"/>

    <script type="text/javascript" src="/website/static/src/js/editor/wysiwyg_multizone.js"/>
    <script type="text/javascript" src="/website/static/src/js/editor/wysiwyg_multizone_translate.js"/>
    <script type="text/javascript" src="/website/static/src/js/editor/widget_link.js"/>
</template>

<template id="website.assets_editor" name="Website Editor Assets (used in website editor)">
    <t t-call="web._assets_helpers"/>

    <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/website.editor.ui.scss"/>

    <script type="text/javascript" src="/website/static/src/js/set_view_track.js"/>

    <script type="text/javascript" src="/website/static/src/js/editor/editor_menu.js"/>
    <script type="text/javascript" src="/website/static/src/js/editor/editor_menu_translate.js"/>

    <script type="text/javascript" src="/website/static/src/js/menu/content.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/customize.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/debug_manager.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/edit.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/mobile_view.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/new_content.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/seo.js"/>
    <script type="text/javascript" src="/website/static/src/js/menu/translate.js"/>

    <script type="text/javascript" src="/website/static/src/js/tours/banner.js"/>
    <script type="text/javascript" src="/website/static/src/js/tours/customize.js"/>

    <script type="text/javascript" src="/website/static/src/js/widgets/ace.js"/>
    <script type="text/javascript" src="/website/static/src/js/widgets/theme.js"/>
</template>

<!-- Layout -->
<template id="submenu" name="Submenu">
    <t t-set="has_visible_submenu" t-value="(submenu.is_mega_menu and submenu.is_visible) or submenu.child_id.filtered(lambda menu: menu.is_visible)"/>
    <li t-if="submenu.is_visible and not has_visible_submenu" t-attf-class="#{item_class or ''}">
        <a t-att-href="submenu.clean_url()"
            t-attf-class="#{link_class or ''} #{'active' if submenu.clean_url() and unslug_url(request.httprequest.path) == unslug_url(submenu.clean_url()) else ''}"
            role="menuitem"
            t-ignore="true"
            t-att-target="'_blank' if submenu.new_window else None">
            <span t-field="submenu.name"/>
        </a>
    </li>
    <li t-if="has_visible_submenu" t-attf-class="#{item_class or ''} dropdown #{
        (submenu.clean_url() and submenu.clean_url() != '/' and any([request.httprequest.path == child.url for child in submenu.child_id if child.url]) or
         (submenu.clean_url() and request.httprequest.path == submenu.clean_url())) and 'active'
        } #{submenu.is_mega_menu and 'position-static'}">
        <a t-attf-class="#{link_class or ''} dropdown-toggle #{submenu.is_mega_menu and 'o_mega_menu_toggle'}" data-toggle="dropdown" href="#">
            <span t-field="submenu.name"/>
        </a>
        <div t-if="submenu.is_mega_menu"
             t-attf-class="dropdown-menu o_mega_menu #{submenu.mega_menu_classes}"
             data-name="Mega Menu"
             t-field="submenu.mega_menu_content"/>
        <ul t-else="" class="dropdown-menu" role="menu">
            <t t-foreach="submenu.child_id" t-as="submenu">
                <t t-call="website.submenu">
                    <t t-set="item_class" t-value="None"/>
                    <t t-set="link_class" t-value="'dropdown-item'"/>
                </t>
            </t>
        </ul>
    </li>
</template>

<template id="layout" name="Main layout" inherit_id="portal.frontend_layout">
    <xpath expr="//html" position="before">
        <t t-set="html_data" t-value="{
            'lang': lang and lang.replace('_', '-'),
            'data-website-id': website.id if website else None,
            'data-editable': '1' if editable else None,
            'data-translatable': '1' if translatable else None,
            'data-edit_translations': '1' if edit_translations else None,
            'data-view-xmlid': xmlid if editable or translatable else None,
            'data-viewid': viewid if editable or translatable else None,
            'data-main-object': repr(main_object) if editable or translatable else None,
            'data-seo-object': repr(seo_object) if seo_object else None,
            'data-oe-company-name': res_company.name,
        }"/>
    </xpath>
    <xpath expr="//head" position="before">
        <t t-if="not title">
            <t t-if="not additional_title and main_object and 'name' in main_object">
                <t t-set="additional_title" t-value="main_object.name"/>
            </t>
            <t t-set="default_title"> <t t-if="additional_title"><t t-raw="additional_title"/> | </t><t t-raw="(website or res_company).name"/> </t>
            <t t-set="seo_object" t-value="seo_object or main_object"/>
            <t t-if="seo_object and 'website_meta_title' in seo_object and seo_object.website_meta_title">
                <t t-set="title" t-value="seo_object.website_meta_title"/>
            </t>
            <t t-else="">
                <t t-set="title" t-value="default_title"></t>
            </t>
        </t>
        <t t-set="x_icon" t-value="website.image_url(website, 'favicon')"/>
    </xpath>
    <xpath expr="//head/meta[last()]" position="after">
        <meta name="generator" content="Odoo"/>
        <t t-set="website_meta" t-value="seo_object and seo_object.get_website_meta() or {}"/>
        <meta name="default_title" t-att-content="default_title" groups="website.group_website_designer"/>
        <meta t-if="main_object and 'website_indexed' in main_object
            and not main_object.website_indexed" name="robots" content="noindex"/>
            <t t-set="seo_object" t-value="seo_object or main_object"/>
            <t t-set="meta_description" t-value="seo_object and 'website_meta_description' in seo_object
                and seo_object.website_meta_description or website_meta_description or website_meta.get('meta_description', '')"/>
            <t t-set="meta_keywords" t-value="seo_object and 'website_meta_keywords' in seo_object
                and seo_object.website_meta_keywords or website_meta_keywords"/>
        <meta t-if="meta_description or editable" name="description" t-att-content="meta_description"/>
        <meta t-if="meta_keywords or editable" name="keywords" t-att-content="meta_keywords"/>
        <t t-if="seo_object">
            <meta name="default_description" t-att-content="website_meta_description or website_meta.get('meta_description')" groups="website.group_website_designer"/>
            <!-- OpenGraph tags for Facebook sharing -->
            <t t-set="opengraph_meta" t-value="website_meta.get('opengraph_meta')"/>
            <t t-if="opengraph_meta">
                <t t-foreach="opengraph_meta" t-as="property">
                    <t t-if="isinstance(opengraph_meta[property], list)">
                        <t t-foreach="opengraph_meta[property]" t-as="meta_content">
                            <meta t-att-property="property" t-att-content="meta_content"/>
                        </t>
                    </t>
                    <t t-else="">
                        <meta t-att-property="property" t-att-content="opengraph_meta[property]"/>
                    </t>
                </t>
            </t>
            <!-- Twitter tags for sharing -->
            <t t-set="twitter_meta" t-value="website_meta.get('twitter_meta')"/>
            <t t-if="opengraph_meta">
                <t t-foreach="twitter_meta" t-as="t_meta">
                    <meta t-att-name="t_meta" t-att-content="twitter_meta[t_meta]"/>
                </t>
            </t>
        </t>

        <t t-if="request and request.is_frontend_multilang and website">
            <t t-set="alternate_languages" t-value="website._get_alternate_languages(canonical_params=canonical_params)"/>
            <t t-foreach="alternate_languages" t-as="lg">
                <link rel="alternate" t-att-hreflang="lg['hreflang']" t-att-href="lg['href']"/>
            </t>
        </t>
        <link t-if="request and website" rel="canonical" t-att-href="website._get_canonical_url(canonical_params=canonical_params)"/>

        <link rel="preconnect" href="https://fonts.gstatic.com/" crossorigin=""/>
    </xpath>

    <xpath expr="//head/t[@t-js='false'][last()]" position="after">
        <t t-call-assets="website.assets_editor" t-js="false" groups="website.group_website_publisher"/>
    </xpath>
    <xpath expr="//head/t[@t-css='false'][last()]" position="after">
        <t t-call-assets="website.assets_editor" t-css="false" groups="website.group_website_publisher" lazy_load="True"/>
    </xpath>

    <xpath expr="//header" position="attributes">
        <attribute name="data-name">Header</attribute>
    </xpath>
    <xpath expr="//header//a[hasclass('navbar-brand')]" position="replace">
        <a class="navbar-brand" href="/" t-if="website" t-field="website.name">My Website</a>
    </xpath>
    <xpath expr="//header//ul[@id='top_menu']" position="attributes">
        <attribute name="class" separator=" " add="o_menu_loading"/>
    </xpath>
    <xpath expr="//header//ul[@id='top_menu']/li[hasclass('divider')]" position="attributes">
        <attribute name="t-if">website.user_id != user_id</attribute>
    </xpath>
    <xpath expr="//header//ul[@id='top_menu']/li[hasclass('dropdown')]" position="attributes">
        <attribute name="t-if">website.user_id != user_id</attribute>
    </xpath>
    <xpath expr="//header//ul[@id='top_menu']/li[hasclass('divider')]" position="before">
        <t t-foreach="website.menu_id.child_id" t-as="submenu">
            <t t-call="website.submenu">
                <t t-set="item_class" t-value="'nav-item'"/>
                <t t-set="link_class" t-value="'nav-link'"/>
            </t>
        </t>
    </xpath>

    <xpath expr="//div[hasclass('o_footer_copyright')]//span[@t-field='res_company.name']" position="after">
        <t t-call="website.language_selector"/>
    </xpath>
    <xpath expr="//t[@t-call='web.brand_promotion']/.." position="attributes">
        <attribute name="class" add="o_not_editable" separator=" "/>
        <attribute name="t-if">not editable</attribute>
    </xpath>

    <xpath expr="//div[@id='wrapwrap']" position="after">
        <script id='tracking_code' t-if="website and website.google_analytics_key and not editable">
            (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
            (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
            m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
            })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

            ga('create', '<t t-esc="website.google_analytics_key"/>'.trim(), 'auto');
            ga('send','pageview');
        </script>
    </xpath>

    <!-- Page options -->
    <xpath expr="//div[@id='wrapwrap']" position="before">
        <t groups="website.group_website_publisher">
            <t t-foreach="['header_overlay', 'header_color']" t-as="optionName">
                <input t-if="optionName in main_object" type="hidden" class="o_page_option_data" t-att-name="optionName" t-att-value="main_object[optionName]"/>
            </t>
        </t>
    </xpath>
    <xpath expr="//div[@id='wrapwrap']" position="attributes">
        <attribute name="t-attf-class" add="#{'o_header_overlay' if 'header_overlay' in main_object and main_object.header_overlay else ''}" separator=" "/>
    </xpath>
    <xpath expr="//header" position="attributes">
        <attribute name="t-attf-class" add="#{main_object.header_color if 'header_color' in main_object else ''}" separator=" "/>
    </xpath>
</template>

<template id="brand_promotion" inherit_id="web.brand_promotion" name="Brand Promotion">
    <xpath expr="//t[@t-call='web.brand_promotion_message']" position="replace">
        <t t-call="web.brand_promotion_message">
            <t t-set="_message">
                Create a <a target="_blank" href="http://www.odoo.com/page/website-builder?utm_source=db&amp;utm_medium=website">free website</a>
            </t>
            <t t-set="_utm_medium">website</t>
        </t>
    </xpath>
</template>

<template id="layout_logo_show" inherit_id="website.layout" name="Show Logo">
    <xpath expr="//header//a[hasclass('navbar-brand')]" position="replace">
        <a href="/" class="navbar-brand logo">
            <span t-field="website.logo" t-options="{'widget': 'image'}" role="img" t-att-aria-label="'Logo of %s' % website.name" t-att-title="website.name" />
        </a>
    </xpath>
</template>

<template id="affix_top_menu" inherit_id="website.layout" name="Fixed Top Menu">
    <xpath expr="//header" position="attributes">
        <attribute name="t-attf-class" add="#{'o_affix_enabled'}" separator=" "/>
    </xpath>
</template>

<template id="no_autohide_menu" inherit_id="website.layout" active="False">
    <xpath expr="//header" position="attributes">
        <attribute name="t-attf-class" add="#{'o_no_autohide_menu'}" separator=" "/>
    </xpath>
</template>

<!-- Features template -->
<template id="login_layout" inherit_id="web.login_layout" name="Website Login Layout" priority="20">
    <xpath expr="t" position="replace">
        <t t-call="website.layout">
            <div class="oe_website_login_container" t-raw="0"/>
        </t>
    </xpath>
</template>

<template id="footer_custom" inherit_id="website.layout" name="Footer">
    <xpath expr="//div[@id='footer']" position="replace">
        <div id="footer" class="oe_structure oe_structure_solo" t-ignore="true" t-if="not no_footer">
            <section class="s_text_block pt16 pb8">
                <div class="container">
                    <div class="row">
                        <div class="col-lg-4">
                            <h5>Our Products &amp; Services</h5>
                            <ul class="list-unstyled">
                                <li><a href="/">Home</a></li>
                            </ul>
                        </div>
                        <div class="col-lg-4" id="connect">
                            <h5>Connect with us</h5>
                            <ul class="list-unstyled">
                                <li><a href="/contactus">Contact us</a></li>
                                <li><i class="fa fa-phone"/> <span class="o_force_ltr" t-field="res_company.phone"/></li>
                                <li><i class="fa fa-envelope"/>  <span t-field="res_company.email"/></li>
                            </ul>
                            <p>
                                <a t-if="website.social_facebook" t-att-href="website.social_facebook" class="btn btn-sm btn-link"><i class="fa fa-2x fa-facebook-square"/></a>
                                <a t-if="website.social_twitter" t-att-href="website.social_twitter" class="btn btn-sm btn-link"><i class="fa fa-2x fa-twitter"/></a>
                                <a t-if="website.social_linkedin" t-att-href="website.social_linkedin" class="btn btn-sm btn-link"><i class="fa fa-2x fa-linkedin"/></a>
                                <a t-if="website.social_youtube" t-att-href="website.social_youtube" class="btn btn-sm btn-link"><i class="fa fa-2x fa-youtube-play"/></a>
                                <a t-if="website.social_github" t-att-href="website.social_github" class="btn btn-sm btn-link"><i class="fa fa-2x fa-github"/></a>
                                <a t-if="website.social_instagram" t-att-href="website.social_instagram" class="btn btn-sm btn-link"><i class="fa fa-2x fa-instagram"/></a>
                            </p>
                        </div>
                        <div class="col-lg-4">
                            <h5>
                                <span t-field="res_company.name"/>
                                <small> - <a href="/aboutus">About us</a></small>
                            </h5>
                            <p>
                                We are a team of passionate people whose goal is to improve everyone's
                                life through disruptive products. We build great products to solve your
                                business problems.
                            </p>
                            <p>
                                Our products are designed for small to medium size companies willing to optimize
                                their performance.
                            </p>
                        </div>
                    </div>
                </div>
            </section>
        </div>
    </xpath>
</template>

<template id="language_selector">
    <ul class="js_language_selector mb0 list-inline" t-if="(request and request.is_frontend_multilang and len(languages) &gt; 1) or (website and (editable or translatable))">
        <li class="list-inline-item">
            <div class="dropup">
                <button class="btn btn-sm btn-secondary dropdown-toggle" type="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
                    <span t-esc="list(filter(lambda lg : lg[0] == lang, languages))[0][2].split('/').pop()"/>
                    <span class="caret ml4"/>
                </button>
                <div class="dropdown-menu" role="menu">
                    <t t-foreach="languages" t-as="lg">
                        <a t-att-href="url_for(request.httprequest.path + '?' + keep_query(), lang_code=lg[0])"
                           class="dropdown-item js_change_lang"
                           t-att-data-url_code="lg[1]">
                            <t t-esc="lg[2].split('/').pop()"/>
                        </a>
                    </t>
                </div>
            </div>
        </li>
        <li groups="website.group_website_publisher" class="list-inline-item">
            <t t-set="url_return" t-value="quote_plus(url_for('', '[lang]') + '?' + keep_query())"/>
            <a class="d-none d-sm-block" t-attf-href="/web#action=base.action_view_base_language_install&amp;website_id=#{website.id if website else ''}&amp;url_return=#{url_return}">
                <i class="fa fa-plus-circle"/>
                Add a language...
            </a>
        </li>
    </ul>
</template>

<template id="record_cover">
    <t t-set="_cp" t-value="_cp or json.loads(_record.cover_properties)"/>
    <div t-att-data-use_size="use_size"
         t-att-data-use_filters="use_filters"
         t-att-data-use_text_size="use_text_size"
         t-att-data-use_text_align="use_text_align"
         t-att-data-res-model="request.env.user.has_group('website.group_website_publisher') and _record._name"
         t-att-data-res-id="request.env.user.has_group('website.group_website_publisher') and _record.id"
         t-attf-class="o_record_cover_container d-flex flex-column h-100 bg-secondary #{use_size and _cp.get('resize_class')} #{use_text_size and _cp.get('text_size_class')} #{use_text_align and _cp.get('text_align_class')} #{additionnal_classes}">
        <div class="o_record_cover_component o_record_cover_image" t-attf-style="background-image: #{_cp.get('background-image')};"/>
        <div t-if="use_filters" t-attf-class="o_record_cover_component o_record_cover_filter #{_cp.get('background-color')}" t-attf-style="opacity: #{_cp.get('opacity', 0.0)};"/>
        <t t-raw="0"/>
    </div>
</template>

<!-- Util template -->
<template id="publish_management">
    <div groups="website.group_website_publisher" t-ignore="true" class="float-right css_editable_mode_hidden" t-att-style="style or None">
        <div t-attf-class="btn-group #{btn_class} js_publish_management #{object.website_published and 'css_published' or 'css_unpublished'}" t-att-data-id="object.id" t-att-data-object="object._name" t-att-data-controller="publish_controller">
            <button class="btn btn-danger js_publish_btn">Unpublished</button>
            <button class="btn btn-success js_publish_btn">Published</button>
            <button type="button" t-attf-class="btn btn-default dropdown-toggle dropdown-toggle-split" t-att-id="'dopprod-%s' % object.id" data-toggle="dropdown"/>
            <div class="dropdown-menu" role="menu" t-att-aria-labelledby="'dopprod-%s' % object.id">
                <t t-raw="0"/>
                <a role="menuitem" t-attf-href="/web#view_type=form&amp;model=#{object._name}&amp;id=#{object.id}&amp;action=#{action}&amp;menu_id=#{menu or object.env.ref('website.menu_website_configuration').id}"
                    title='Edit in backend' class="dropdown-item" t-if="publish_edit">Edit</a>
            </div>
        </div>
    </div>
</template>

<template id="publish_short">
    <t groups="website.group_website_publisher" t-ignore="true">
        <div t-attf-class="float-right js_publish_management #{object.website_published and 'css_published' or 'css_unpublished'}" t-att-data-id="object.id" t-att-data-object="object._name" t-att-data-controller="publish_controller">
            <button t-attf-class="btn btn-danger js_publish_btn #{additionnal_btn_classes or ''}">Unpublished</button>
            <button t-attf-class="btn btn-success js_publish_btn #{additionnal_btn_classes or ''}">Published</button>
        </div>
    </t>
</template>

<template id="pager" name="Pager" inherit_id="portal.pager">
</template>

<!--
    XML template to be processed for theme customization modal.

    Allowed tags in the root <div/> elements:

    <content>: Declares a new set of options (a tab pane in the dialog)
        -> string = tab's nav text
        -> title = tab's heading

    In the <content/> elements:

    <opt>: Declares a new toggle option
        -> id (optional) = an ID to associate to the option

        -> string (optional) = option's text
        -> data-icon (optional) = an image URL to set as option's background
        -> data-font (optional) = indicates which fonts to use to style the
                option and adds default sample text if no 'string' is specified
                (see $o-theme-fonts in scss)
        -> data-col (optional, default 6) = bootstrap column size for the option
                (ignored for options in a <list/>)

        -> data-xmlid (optional) = template(s) to enable if the input is
                checked (list of comma-separated xml ids)
        -> data-enable (optional) = enable other options (list of
                comma-separated option ids)
        -> data-disable (optional) = disable other options (list of
                comma-separated option ids)
        -> data-reload (optional) = force the reloading of the page if the url
                matches the value. Otherwise, only the main assets are reloaded

        -> data-widget (optional) = indicates which widget to use (available
                widgets are: color_palette)

    <list>: Declares a subset of options showed as a list
        -> string = list's title
        -> data-col (optional, default 6) = bootstrap column size for the list

    <selection>: Declares a subset of options showed as a selection (dropdown)
        -> data-col (optional, default 6) = bootstrap column size for the option
                (ignored for options in a <list/>)

    Other elements will be ignored and their children will be processed as if
    that element was omitted, except that those children will be considered as
    in a new set of option. That allows to wrap an option in a fake element to
    indicate it is a standalone option for example.

    Note: if an option is the only one of its group, a toggle element will be
    rendered.
-->
<template id="theme_customize">
    <div>
        <!-- Color options -->
        <content id="theme_customize_content_colors" string="Colors" title="Choose the theme colors">
            <list string="Main">
                <opt data-widget="color" data-color-type="theme" data-color="primary" string="Primary"/>
                <opt data-widget="color" data-color-type="theme" data-color="secondary" string="Secondary"/>
                <opt data-widget="color" data-color-type="theme" data-color="alpha" string="Primary"/>
                <opt data-widget="color" data-color-type="theme" data-color="beta" string="Secondary"/>
                <opt data-widget="color" data-color-type="theme" data-color="gamma" string="Extra Color"/>
                <opt data-widget="color" data-color-type="theme" data-color="delta" string="Extra Color"/>
                <opt data-widget="color" data-color-type="theme" data-color="epsilon" string="Extra Color"/>
            </list>
            <list string="Status">
                <opt data-widget="color" data-color-type="theme" data-color="success" string="Success"/>
                <opt data-widget="color" data-color-type="theme" data-color="info" string="Info"/>
                <opt data-widget="color" data-color-type="theme" data-color="warning" string="Warning"/>
                <opt data-widget="color" data-color-type="theme" data-color="danger" string="Error"/>
            </list>
            <list string="Text">
                <opt data-widget="color" data-color="text" string="Text"/>
                <opt data-widget="color" data-color="h1" string="Headings 1"/>
                <opt data-widget="color" data-color="h2" string="Headings 2"/>
                <opt data-widget="color" data-color="h3" string="Headings 3"/>
                <opt data-widget="color" data-color="h4" string="Headings 4"/>
                <opt data-widget="color" data-color="h5" string="Headings 5"/>
                <opt data-widget="color" data-color="h6" string="Headings 6"/>
            </list>
            <list string="Background">
                <opt data-widget="color" data-color="body" string="Body"/>
                <opt data-widget="color" data-color="menu" string="Menus"/>
                <opt data-widget="color" data-color="footer" string="Footer"/>
            </list>
        </content>

        <!-- Menu options -->
        <content id="theme_customize_content_navbar" string="Navbar" title="Choose your navbar">
            <list string="Main Layout">
                <checkbox><opt data-xmlid="website.affix_top_menu" data-reload="/"/></checkbox>
                <checkbox><opt data-xmlid="portal.portal_show_sign_in" data-reload="/"/></checkbox>
                <opt data-widget="input" data-unit="rem" data-variable="header-font-size" string="Font Size"/>
            </list>
            <list string="Logo">
                <checkbox><opt data-xmlid="website.layout_logo_show" data-reload="/"/></checkbox>
                <opt data-widget="input" data-unit="rem" data-variable="logo-height" string="Logo Height"/>
            </list>
        </content>

        <!-- Layout Options -->
        <content id="theme_customize_content_layout" string="Layout" title="Choose your layout">
            <list string="Body">
                <opt string="Full" data-xmlid="" data-icon="/website/static/src/img/options/layout-full.png"/>
                <opt id="option_layout_boxed" string="Boxed" data-xmlid="website.option_layout_boxed_variables" data-icon="/website/static/src/img/options/layout-boxed.png"/>
            </list>
            <list string="Background">
                <opt id="option_no_background" string="None" data-xmlid=""/>
                <opt string="Choose an image" data-xmlid="website.option_custom_body_image"/>
                <opt string="Choose a pattern" data-xmlid="website.option_custom_body_image, website.option_custom_body_pattern"/>
            </list>
        </content>

        <!-- Font options -->
        <content id="theme_customize_content_fonts" string="Fonts" title="Choose your fonts">
            <list string="Title">
                <fontselection data-variable="headings-font-number"/>
            </list>
            <list string="Body">
                <fontselection data-variable="font-number"/>
            </list>
            <list string="Button">
                <fontselection data-variable="buttons-font-number"/>
            </list>
            <list string="Navbar">
                <fontselection data-variable="navbar-font-number"/>
            </list>
        </content>
    </div>
</template>

<!-- Layout options -->
<template id="option_layout_boxed_variables" inherit_id="website._assets_primary_variables" active="False">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/options/layout/option_layout_boxed_variables.scss"/>
    </xpath>
</template>

<template id="option_custom_body_image" inherit_id="website.assets_frontend" active="False">
    <xpath expr="//link[last()]" position="after">
        <style>
            <!-- Patched by JS option -->
        </style>
    </xpath>
</template>

<template id="option_custom_body_pattern" inherit_id="website.assets_frontend" active="False">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website/static/src/scss/options/layout/option_custom_body_pattern.scss"/>
    </xpath>
</template>

<template id="kanban">
    <t t-set="step"><t t-esc="step or 0"/></t>
    <t t-set="scope"><t t-esc="scope or 0"/></t>
    <t t-set="orderby"><t t-esc="orderby or 'name'"/></t>
    <t t-raw="website.kanban(model, domain, column, template, step=step, scope=scope, orderby=orderby)"/>
</template>

<template id="kanban_contain">
    <table class="table js_kanban">
        <thead>
            <tr>
                <t t-set="width" t-valuef="{{ round(100.0 / (len(objects) if objects else 1), 2) }}%"/>
                <t t-foreach="objects" t-as="obj">
                    <th t-att-width="width">
                        <div t-field="obj['column_id'].name" class="text-center"></div>
                    </th>
                </t>
            </tr>
        </thead>
        <tbody>
            <tr>
                <t t-foreach="objects" t-as="obj">
                    <td class="js_kanban_col" t-att-data-template="template"
                            t-att-data-domain="obj['domain']"
                            t-att-data-page_count="obj['page_count']"
                            t-att-data-model="obj['model']"
                            t-att-data-step="obj['step']"
                            t-att-data-orderby="obj['orderby']">
                        <t t-foreach="obj['object_ids']" t-as="object_id">
                            <t t-call="#{ template }"></t>
                        </t>
                        <!-- pager -->
                        <div t-if="1 != obj['page_end']" class="pagination pagination-centered"><!-- FIXME -->
                            <ul>
                                <li t-attf-class="prev #{'active' if obj['page'] == 1 else '' }">
                                    <a t-att-href=" '%s,%s-%s' % (obj['kanban_url'], obj['column_id'].id, (obj['page'] &gt; 1 and obj['page']-1 or 1)) ">Prev</a></li>
                                <t t-foreach="range(obj['page_start'], obj['page_end']+1)" t-as="p">
                                    <li t-att-class=" 'active' if obj['page'] == p else None ">
                                        <a t-att-href=" '%s,%s-%s' % (obj['kanban_url'], obj['column_id'].id, p)" t-esc="p"></a></li>
                                </t>
                                <li t-attf-class="next #{'active' if obj['page'] == obj['page_end'] else '' }">
                                    <a t-att-href=" '%s,%s-%s' % (obj['kanban_url'], obj['column_id'].id, (obj['page'] &lt; obj['page_end'] and obj['page']+1 or obj['page_end']) )">Next</a></li>
                            </ul>
                        </div>
                    </td>
                </t>
            </tr>
        </tbody>
    </table>
</template>

<!-- Error and special pages -->
<template id="website_info" name="Odoo Information">
    <t t-call="website.layout">
        <div id="wrap"/>
    </t>
</template>

<template id="show_website_info" inherit_id="website.website_info" customize_show="True" name="Show Odoo Information">
    <xpath expr="//div[@id='wrap']" position="inside">
        <div class="oe_structure">
            <section class="container">
              <t t-if="not version">
                <meta http-equiv="refresh" content="0;URL='/website/info'" />
              </t>
              <t t-if="version">
                <h1><t t-esc="res_company.name"/>
                    <small>Odoo Version <t t-raw="version.get('server_version')"/></small>
                </h1>
                <p>
                    Information about the <t t-esc="res_company.name"/> instance of Odoo, the <a target="_blank" href="https://www.odoo.com">Open Source ERP</a>.
                </p>

                <div class="alert alert-warning alert-dismissable mt16" groups="website.group_website_publisher" role="status">
                   <button type="button" class="close" data-dismiss="alert" aria-label="Close">&amp;times;</button>
                   <p>
                     Note: To hide this page, uncheck it from the top Customize menu.
                   </p>
                </div>
                <h2>Installed Applications</h2>
                <dl class="dl-horizontal" t-foreach="apps" t-as="app">
                    <dt>
                        <a t-att-href="app.website" t-if="app.website">
                            <t t-raw="app.shortdesc"/>
                        </a>
                        <span t-raw="app.shortdesc" t-if="not app.website"/>
                    </dt>
                    <dd>
                        <span t-raw="app.summary"/>
                    </dd><dd class="text-muted" groups='base.group_no_one'>
                        Technical name: <span t-field="app.name"/>, author: <span t-field="app.author"/>
                    </dd>
                </dl>

                <div t-if="l10n">
                    <h2 class='mt32'>Installed Localizations / Account Charts</h2>
                    <dl class="dl-horizontal" t-foreach="l10n" t-as="app">
                        <dt>
                            <a t-attf-href="https://www.odoo.com/page/accounting/#{app.name}">
                                <t t-raw="app.shortdesc"/>
                            </a>
                        </dt>
                        <dd>
                            <span t-raw="app.summary"/>
                        </dd><dd class="text-muted" groups='base.group_no_one'>
                            Technical name: <span t-field="app.name"/>, author: <span t-field="app.author"/>
                        </dd>
                    </dl>
                </div>
              </t>
            </section>
        </div>
    </xpath>
</template>

<template id="default_page">
    <t t-call="website.layout">
        <div id="wrap" class="oe_structure oe_empty"/>
    </t>
</template>

<template id="default_js">
    <script type="text/javascript">
        if (0 &gt; 1) {
            let it_cant_be = false;
        }
    </script>
</template>
<template id="default_xml">
    <t t-translation="off">&lt;?xml version="1.0" encoding="utf-8"?&gt;</t>
</template>
<template id="default_css">
    <style type="text/css">
        div#wrap div &gt; h1{
            color: #875A7B;
        }
    </style>
</template>
<template id="default_less">
    <style type="text/less">
        div#wrap div &gt; h1 {
            color: @o-brand-odoo;
        }
    </style>
</template>
<template id="default_scss">
    <style type="text/scss">
        div#wrap div &gt; h1 {
            color: $o-brand-odoo;
        }
    </style>
</template>
<template id="default_csv">
    <t t-translation="off">1,2,3</t>
</template>

<template id="page_404">
    <t t-call="http_routing.404">
        <div class="container">
            <div class="alert alert-info mt32">
                <p>This page does not exist, but you can create it as you are administrator of this site.</p>
                <a role="button" class="btn btn-primary js_disable_on_click" t-attf-href="/website/add/#{ path }#{ from_template and '?template=%s' % from_template }">Create Page</a>
            </div>
            <div class="text-center text-muted">Edit the content below this line to adapt the default "page not found" page.</div>
        </div>
        <hr/>
    </t>
</template>

<template id="qweb_500" inherit_id="http_routing.500">
    <!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
    <!-- This template should not use any variable except those provided by website.ir_http._handle_exception  -->
    <!--    no request.crsf_token, no theme style, no assets, ... cursor can be broken during rendering !      -->
    <!--    see test_05_reset_specific_view_controller_broken_request                                          -->
    <!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
    <xpath expr="//script[last()]" position="before">
        <script type="text/javascript" src="/web/static/lib/bootstrap/js/modal.js"/>
    </xpath>
    <xpath expr="//style" position="after">
        <t t-if='view'>
            <script>
                $(document).ready(function() {
                    var button = $('.reset_templates_button');
                    button.click(function() {
                        $('#reset_templates_mode').val($(this).data('mode'));
                        var dialog = $('#reset_template_confirmation').modal('show');
                        var input = dialog.find('input[type="text"]').val('').focus();
                        var dialog_form = dialog.find('form');
                        dialog_form.submit(function() {
                            if (input.val() == dialog.find('.confirm_word').text()) {
                                dialog.modal('hide');
                                button.prop('disabled', true).text('Working...');
                                $('#reset_templates_form').attr('action', '/website/reset_template');
                                $('#reset_templates_form').trigger('submit');
                            } else {
                                input.val('').focus();
                            }
                            return false;
                        });
                        return false;
                    });
                });
            </script>
        </t>
    </xpath>
    <xpath expr="//div[@id='wrapwrap']" position="before">
        <div t-if="view" role="dialog" id="reset_template_confirmation" class="modal" tabindex="-1" t-ignore="true">
            <div class="modal-dialog">
                <form role="form">
                    <div class="modal-content">
                        <header class="modal-header">
                            <h4 class="modal-title">Reset templates</h4>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">×</button>
                        </header>
                        <main class="modal-body">
                            <div class="form-group row mb0">
                                <label for="page-name" class="col-md-12 col-form-label">
                                    <p>The selected templates will be reset to their factory settings.</p>
                                </label>
                            </div>
                            <div class="form-group row mb0">
                                <label for="page-name" class="col-md-9 col-form-label">
                                    <p>Type '<i class="confirm_word">yes</i>' in the box below if you want to confirm.</p>
                                </label>
                                <div class="col-md-3 mt16">
                                    <input type="text" id="page-name" class="form-control" required="required" placeholder="yes"/>
                                </div>
                            </div>
                        </main>
                        <footer class="modal-footer">
                            <button type="button" class="btn" data-dismiss="modal" aria-label="Cancel">Cancel</button>
                            <input type="submit" value="Confirm" class="btn btn-primary"/>
                        </footer>
                    </div>
                </form>
            </div>
        </div>
    </xpath>
    <xpath expr="//div[@id='error_message']" position="after">
        <div class="container" t-if="view and editable">
            <div class="alert alert-danger" t-if="debug" role="alert">
                <h4>Template fallback</h4>
                <p>An error occured while rendering the template <code t-esc="qweb_exception.name"/>.</p>
                <p>If this error is caused by a change of yours in the templates, you have the possibility to reset the template to its <strong>factory settings</strong>.</p>
                <form action="#" method="post" id="reset_templates_form">
                    <ul>
                        <li>
                            <label>
                                <t t-esc="view.name"/>
                            </label>
                        </li>
                    </ul>
                    <input type="hidden" name="redirect" t-att-value="request.httprequest.path"/>
                    <input type="hidden" id="reset_templates_view_id" name="view_id" t-att-value="view.id"/>
                    <input type="hidden" id="reset_templates_mode" name="mode"/>
                    <button data-mode="soft" class="reset_templates_button btn btn-info">Restore previous version (soft reset).</button>
                    <button t-if="view.arch_fs" data-mode="hard" class="reset_templates_button btn btn-outline-danger">Reset to initial version (hard reset).</button>
                </form>
            </div>
        </div>
    </xpath>
</template>

<template id="portal_404" inherit_id="portal.portal_404">
    <xpath expr="//li" position="before">
        <li><a href="/">Homepage</a></li>
    </xpath>
</template>

<template id="robots">
<t t-translation="off">
User-agent: *
Sitemap: <t t-esc="url_root"/>sitemap.xml
</t>
</template>

<template id="sitemap_locs">
    <url t-foreach="locs" t-as="page">
        <loc><t t-esc="url_root"/><t t-esc="page['loc']"/></loc><t t-if="page.get('lastmod', False)">
        <lastmod t-esc="page['lastmod']"/></t><t t-if="page.get('priority', False)">
        <priority t-esc="page['priority']"/></t><t t-if="page.get('changefreq', False)">
        <changefreq t-esc="page['changefreq']"/></t>
    </url>
</template>

<template id="sitemap_xml"><t t-translation="off">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</t>
<urlset t-attf-xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <t t-raw="content"/>
</urlset>
</template>

<template id="sitemap_index_xml"><t t-translation="off">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
<sitemapindex t-attf-xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap t-translation="off" t-foreach="pages" t-as="page">
    <loc><t t-esc="url_root"/>sitemap-<t t-esc="page"/>.xml</loc>
  </sitemap>
</sitemapindex>
</t>
</template>

<template id="company_description" name="Company Description">
    <address itemscope="itemscope" itemtype="http://schema.org/Organization">
        <!-- TODO widget contact must add itemprop attributes -->
        <div t-field="res_company.partner_id" t-options='{
                "widget": "contact",
                "fields": ["name", "address", "phone", "mobile", "email"]}'/>
        <t t-if="not res_company.google_map_img()">
            <span class="fa fa-map-marker fa-fw mt16" role="img" aria-label="Address" title="Address"/> <a t-att-href="res_company.google_map_link()" target="_BLANK"> Google Maps</a>
        </t>
    </address>
    <t t-if="res_company.google_map_img()">
        <a t-att-href="res_company.google_map_link()" target="_BLANK">
           <img class="img-fluid" t-att-src="res_company.google_map_img()" alt="Google Maps"/>
        </a>
    </t>
</template>

<template id="website_search_box" name="Website Searchbox">
    <div t-attf-class="input-group #{_classes}" role="search">
        <input type="text" name="search" class="search-query form-control oe_search_box" placeholder="Search..." t-att-value="search"/>
        <div class="input-group-append">
            <button type="submit" class="btn btn-primary oe_search_button" aria-label="Search" title="Search"><i class="fa fa-search"/></button>
        </div>
    </div>
</template>

<template id="index_management">
    <t groups="website.group_website_publisher" t-ignore="true">
        <div t-attf-class="float-right js_index_management #{object.website_indexed and 'css_published' or 'css_unpublished'}" t-att-data-id="object.id" t-att-data-object="object._name">
            <button class="btn btn-danger js_index_btn">Unindexed</button>
            <button class="btn btn-success js_index_btn">Indexed</button>
        </div>
    </t>
</template>

<template id="list_website_pages" name="Website Pages Management">
  <t t-call="website.layout">
    <div id="wrap">
      <div class="container" id="list_website_pages">
          <form class="mt8 float-right" role="search" t-attf-action="/website/pages" method="get">
              <t t-call="website.website_search_box"/>
          </form>
          <div t-if="searchbar_sortings" class="dropdown float-right mt8 mr8">
              <button class="btn btn-secondary dropdown-toggle" type="button" data-toggle="dropdown">
                  <span class="fa fa-sort fa-lg" role="img" aria-label="Sort" title="Sort"/>
                  <span class="d-none d-xl-inline" t-esc="searchbar_sortings[sortby].get('label', 'Newest')"/>
              </button>
              <div class="dropdown-menu" aria-labelledby="portal_searchbar_sortby" role="menu">
                  <t t-foreach="searchbar_sortings" t-as="option">
                      <a role="menuitem"
                         t-att-href="request.httprequest.path + '?' + keep_query('*', sortby=option)"
                         t-attf-class="dropdown-item#{sortby == option and ' active' or ''}">
                          <span t-esc="searchbar_sortings[option].get('label')"/>
                      </a>
                  </t>
              </div>
          </div>
          <h3 class="mt16">Manage Your Pages</h3>
          <t t-if="not pages">
              <div t-if="search" class="alert alert-warning mt8" role="alert">
                  Your search '<t t-esc="search" />' did not match any pages.
              </div>
              <div t-else="" class="alert alert-warning mt8" role="alert">
                  There are currently no pages for your website.
              </div>
          </t>
          <div t-if="pages" class="table-responsive">
              <table class="table table-hover">
                  <thead>
                    <tr>
                      <th>Name</th>
                      <th>Url</th>
                      <th class="text-center"><i title="Is the page included in the main menu?" class="fa fa-thumb-tack"></i></th>
                      <th class="text-center"><i title="Is the page published?" class="fa fa-eye"></i></th>
                      <th class="text-center"><i title="Is the page indexed by search engines?" class="fa fa-globe"></i></th>
                      <th class="text-center"><i title="Is the page SEO optimized?" class="fa fa-search"></i></th>
                      <th></th>
                    </tr>
                  </thead>
                  <t t-set='prev_page' t-value='False' />
                  <t t-set='page' t-value='pages[0]' />
                  <t t-foreach="pages[1:]" t-as="next_page">
                    <t t-call='website.one_page_line'/>
                    <t t-set='prev_page' t-value='page' />
                    <t t-set='page' t-value='next_page' />
                  </t>

                  <t t-set='next_page' t-value='False'/>
                  <t t-call='website.one_page_line' />
              </table>
          </div>
          <div t-if="pager" class="o_portal_pager text-center">
              <t t-call="website.pager"/>
          </div>
      </div>
    </div>
  </t>
</template>

<template id="one_page_line">
    <t t-set='specific_page' t-value="page.website_id"/>
    <t t-set='final_page' t-value="(next_page and page.url != next_page.url) or not next_page or specific_page"/>
    <tr t-att-style='not final_page and "color:#999"'>
        <td>
            <t groups="website.group_multi_website">
                <i t-if='specific_page and prev_page and prev_page.url == page.url and not prev_page.website_id' class="fa fa-level-up fa-rotate-90 ml32 mr4"/>
                <i t-else="1" class="fa fa-globe mr4" t-att-style="'visibility:hidden;' if specific_page else ''"/>
            </t>
            <i t-if="page.is_homepage" class="fa fa-home" title="Home"/> <span t-esc="page.name"/>
        </td>
        <td>
            <a t-if='final_page' t-att-href="page.url"><t t-esc="page.url"/></a>
        </td>
        <td class="text-center">
            <i t-if="page.menu_ids" class="fa fa-check" title="In main menu"/>
            <i t-else="" class="fa fa-times text-muted" title="Not in main menu"/>
        </td>
        <td class="text-center">
            <t t-set='date_formatted'><t t-options='{"widget": "date"}' t-esc="page.date_publish"/></t>
            <i t-if="page.is_visible" class="fa fa-check" title="Visible"/>
            <i t-elif="page.website_published" class="fa fa-eye-slash" t-attf-title="This page will be visible on {{ date_formatted }}"/>
            <i t-else="" class="fa fa-times text-muted" title="Not visible"/>
        </td>
        <td class="text-center">
            <i t-if="page.website_indexed" class="fa fa-check" title="Indexed"/>
            <i t-else="" class="fa fa-times text-muted" title="Not indexed"/>
        </td>
        <td class="text-center">
            <i t-if="page.is_seo_optimized" class="fa fa-check" title="SEO optimized"/>
            <i t-else="" class="fa fa-times text-muted" title="Not SEO optimized"/>
        </td>
        <td class="text-right" style="white-space:nowrap;">
            <a class="mr4 fa fa-cog js_page_properties" href="#" t-att-data-id="page.id" title="Manage this page"/>
            <a class="mr4 fa fa-search" t-attf-href="{{ page.url}}?enable_seo" title="Optimize SEO of this page"/>
            <a groups="base.group_no_one" class="mr4 fa fa-bug" t-attf-href="/web#id=#{page.view_id.id}&amp;view_type=form&amp;model=ir.ui.view" title="Edit code in backend"/>
            <a class="mr4 fa fa-clone js_clone_page" t-att-data-id="page.id" href="#" title="Clone this page"/>
            <a class="fa fa-trash js_delete_page" t-att-data-id="page.id" href="#" title="Delete this page"/>
        </td>
    </tr>
</template>

</odoo>

```

## File: views\website_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Menu items -->
        <menuitem name="Website"
            id="menu_website_configuration"
            sequence="9"
            groups="base.group_user"
            web_icon="website,static/description/icon.png"/>

        <record id="action_website_add_features" model="ir.actions.act_window">
            <field name="name">Apps</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">ir.module.module</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="domain">['!', ('name', '=like', 'theme_%')]</field>
            <field name="context" eval="{'search_default_category_id': ref('base.module_category_website_website')}"/>
        </record>


        <!-- website views -->
        <record id="view_website_form" model="ir.ui.view">
            <field name="name">website.form</field>
            <field name="model">website</field>
            <field name="arch" type="xml">
                <form string="Website Settings">
                    <sheet>
                        <div name="domain">
                            <group name="domain">
                                <field name="name"/>
                                <field name="domain"/>
                            </group>
                        </div>
                        <div name="logo">
                            <group name="logo">
                                <field name="logo" widget="image" class="oe_avatar float-left"/>
                            </group>
                        </div>
                        <div name="other">
                            <group name="other">
                                <field name="company_id" widget="selection" groups="base.group_multi_company"/>
                                <field name="default_lang_id" widget="selection" groups="base.group_no_one"/>
                            </group>
                        </div>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="view_website_tree" model="ir.ui.view">
            <field name="name">website.tree</field>
            <field name="model">website</field>
            <field name="arch" type="xml">
                <tree string="Websites">
                    <field name="name"/>
                    <field name="domain"/>
                    <field name="country_group_ids" widget="many2many_tags"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="default_lang_id"/>
                    <field name="theme_id" groups="base.group_no_one"/>
                </tree>
            </field>
        </record>

        <record id="action_website_list" model="ir.actions.act_window">
            <field name="name">Websites</field>
            <field name="res_model">website</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_website_tree"/>
            <field name="target">current</field>
        </record>


        <!-- website.page views -->
        <record id="website_pages_form_view" model="ir.ui.view">
            <field name="name">website.page.form</field>
            <field name="model">website.page</field>
            <field name="arch" type="xml">
                <form string="Website Page Settings">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="url"/>
                                <field name="view_id" context="{'display_website': True}"/>
                                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                                <field name="track"/>
                            </group>
                            <group>
                                <field name="website_indexed"/>
                                <field name="is_published"/>
                                <field name="date_publish"/>
                            </group>
                        </group>
                        <label for="menu_ids" string="Related Menu Items"/>
                        <field name="menu_ids"/>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="website_pages_tree_view" model="ir.ui.view">
            <field name="name">website.page.list</field>
            <field name="model">website.page</field>
            <field name="arch" type="xml">
                <tree string="Website Pages" default_order="name">
                    <field name="name"/>
                    <field name="url"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <field name="website_indexed"/>
                    <field name="is_published" string="Page Published"/>
                    <field name="create_uid" invisible="1"/>
                    <field name="write_uid"/>
                    <field name="write_date"/>
                    <field name="track"/>
                </tree>
            </field>
        </record>

        <record id="website_pages_view_search" model="ir.ui.view">
            <field name="name">website.page.view.search</field>
            <field name="model">website.page</field>
            <field name="arch" type="xml">
                <search string="Website Pages" >
                    <field name="url"/>
                    <filter string="Published" name="published" domain="[('website_published', '=', True)]"/>
                    <filter string="Not published" name="not_published" domain="[('website_published', '=', False)]"/>
                    <separator/>
                    <filter string="Tracked" name="tracked" domain="[('track', '=', True)]"/>
                    <filter string="Not tracked" name="not_tracked" domain="[('track', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="action_website_pages_list" model="ir.actions.act_window">
            <field name="name">Website Pages</field>
            <field name="res_model">website.page</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="website_pages_tree_view"/>
            <field name="target">current</field>
        </record>

        <!-- website.menu views -->
        <record id="website_menus_form_view" model="ir.ui.view">
            <field name="name">website.menu.form</field>
            <field name="model">website.menu</field>
            <field name="arch" type="xml">
                <form string="Website Menus Settings">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                                <field name="url"/>
                                <field name="page_id"/>
                                <field name="is_mega_menu"/>
                            </group>
                            <group>
                                <field name="new_window"/>
                                <field name="sequence"/>
                                <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                            </group>
                            <group>
                                <field name="parent_id" context="{'display_website': True}"/>
                                <field name="group_ids"/>
                            </group>
                        </group>
                        <label for="child_id" string="Child Menus"/>
                        <field name="child_id">
                            <tree>
                                <field name="sequence" widget="handle"/>
                                <field name="name"/>
                                <field name="url"/>
                            </tree>
                        </field>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="menu_tree" model="ir.ui.view">
            <field name="name">website.menu.tree</field>
            <field name="model">website.menu</field>
            <field name="field_parent">child_id</field>
            <field name="arch" type="xml">
                <tree string="Website menu" editable="bottom">
                    <field name="sequence" widget="handle"/>
                    <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                    <field name="name"/>
                    <field name="url"/>
                    <field name="is_mega_menu"/>
                    <field name="new_window"/>
                    <field name="parent_id" context="{'display_website': True}"/>
                    <field name="group_ids" widget="many2many_tags"/>
                </tree>
            </field>
        </record>

        <record id="menu_search" model="ir.ui.view">
            <field name="name">website.menu.search</field>
            <field name="model">website.menu</field>
            <field name="arch" type="xml">
                <search string="Search Menus">
                    <field name="name"/>
                    <field name="url"/>
                    <field name="website_id" groups="website.group_multi_website"/>
                    <group string="Group By">
                        <filter string="name" name="group_by_name" domain="[]" context="{'group_by':'name'}"/>
                        <filter string="url" name="group_by_url" domain="[]" context="{'group_by':'url'}"/>
                        <filter string="website"  name="my_websites" domain="[]" context="{'group_by':'website_id'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="action_website_menu" model="ir.actions.act_window">
            <field name="name">Website Menu</field>
            <field name="res_model">website.menu</field>
            <field name="view_mode">tree,form</field>
            <field name="context">{'search_default_my_websites':1}</field>
            <field name="view_id" ref="menu_tree"/>
            <field name="target">current</field>
        </record>

        <!-- ir.ui.view views -->
        <record model="ir.ui.view" id="view_view_form_extend">
            <field name="model">ir.ui.view</field>

            <field name="inherit_id" ref="base.view_view_form"/>
            <field name="arch" type="xml">
                <field name="inherit_id" position="attributes">
                    <attribute name="context">{'display_website': True}</attribute>
                </field>
                <field name="name" position="after">
                    <field name="website_id" options="{'no_create': True}" groups="website.group_multi_website"/>
                    <field name="key"/>
                    <field name="page_ids" invisible="1" />
                    <field name="first_page_id" attrs="{'invisible': [('page_ids', '=', [])]}" />
                </field>
                <sheet position="before">
                    <header>
                        <button name="redirect_to_page_manager" string="Go to Page Manager"
                            type="object" attrs="{'invisible': [('page_ids', '=', [])]}"/>
                    </header>
                </sheet>
            </field>
        </record>
        <record id="view_view_tree_inherit_website" model="ir.ui.view">
            <field name="model">ir.ui.view</field>
            <field name="inherit_id" ref="base.view_view_tree"/>
            <field name="arch" type="xml">
                <field name="name" position="after">
                    <field name="website_id" groups="website.group_multi_website"/>
                </field>
                <field name="xml_id" position="before">
                    <field name="key" groups="website.group_multi_website"/>
                </field>
            </field>
        </record>

        <!-- Dashboard -->
        <record id="backend_dashboard" model="ir.actions.client">
            <field name="name">Analytics</field>
            <field name="tag">backend_dashboard</field>
        </record>

        <record id="ir_actions_server_website_dashboard" model="ir.actions.server">
            <field name="name">Website: Dashboard</field>
            <field name="model_id" ref="website.model_website"/>
            <field name="state">code</field>
            <field name="code">action = model.action_dashboard_redirect()</field>
        </record>

        <record id="ir_actions_server_website_google_analytics" model="ir.actions.server">
            <field name="name">Website: Dashboard</field>
            <field name="model_id" ref="website.model_website"/>
            <field name="state">code</field>
            <field name="code">action = model.env.ref('website.backend_dashboard').read()[0]</field>
        </record>

        <menuitem id="menu_dashboard"
            name="Dashboard"
            sequence="1"
            parent="website.menu_website_configuration"/>

        <!-- Force empty action, to ease upgrade -->
        <record id="menu_dashboard" model="ir.ui.menu">
            <field name="action" eval="False"/>
        </record>

        <menuitem id="menu_website_dashboard" parent="menu_dashboard"
            sequence="10" name="eCommerce Dashboard"
            action="website.ir_actions_server_website_dashboard" active="0"/>

        <menuitem id="menu_website_google_analytics" parent="menu_dashboard"
            sequence="20" name="Analytics"
            action="website.ir_actions_server_website_google_analytics"/>

    </data>
</odoo>

```

## File: views\website_visitor_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <!--page history-->
    <record id="website_visitor_page_view_tree" model="ir.ui.view">
        <field name="name">website.track.view.tree</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <tree string="Visitor Page Views History" create="0">
                <field name="visitor_id"/>
                <field name="page_id"/>
                <field name="url"/>
                <field name="visit_datetime"/>
            </tree>
        </field>
    </record>

    <record id="website_visitor_page_view_graph" model="ir.ui.view">
        <field name="name">website.track.view.graph</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <graph string="Visitor Page Views">
                <field name="url"/>
            </graph>
        </field>
    </record>

    <record id="website_visitor_page_view_search" model="ir.ui.view">
        <field name="name">website.track.view.search</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <search string="Search Visitor">
                <field name="visitor_id"/>
                <field name="page_id"/>
                <field name="url"/>
                <field name="visit_datetime"/>
                <filter string="Pages" name="type_page" domain="[('page_id', '!=', False)]"/>
                <filter string="Urls &amp; Pages" name="type_url" domain="[('url', '!=', False)]"/>
                <group string="Group By">
                    <filter string="Visitor" name="group_by_visitor" domain="[]" context="{'group_by': 'visitor_id'}"/>
                    <filter string="Page" name="group_by_page" domain="[]" context="{'group_by': 'page_id'}"/>
                    <filter string="Url" name="group_by_url" domain="[]" context="{'group_by': 'url'}"/>
                    <filter string="Date" name="group_by_date" domain="[]" context="{'group_by': 'visit_datetime'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="website_visitor_page_action" model="ir.actions.act_window">
        <field name="name">Page Views History</field>
        <field name="res_model">website.track</field>
        <field name="view_mode">tree</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('website_visitor_page_view_tree')}),
            (0, 0, {'view_mode': 'graph', 'view_id': ref('website_visitor_page_view_graph')}),
        ]"/>
        <field name="domain">[('visitor_id', '=', active_id), ('url', '!=', False)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No page views yet for this visitor
            </p>
        </field>
    </record>

    <!--Website visitor actions-->
    <record id="website.visitor_partner_action" model="ir.actions.act_window">
        <field name="name">Partners</field>
        <field name="res_model">res.partner</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('visitor_ids', 'in', [active_id])]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
              No partner linked for this visitor
            </p>
        </field>
    </record>

    <!-- website visitor views -->
    <record id="website_visitor_view_kanban" model="ir.ui.view">
        <field name="name">website.visitor.view.kanban</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <kanban class="o_wvisitor_kanban">
                <field name="id"/>
                <field name="country_id"/>
                <field name="email"/>
                <field name="is_connected"/>
                <field name="display_name"/>
                <field name="last_visited_page_id"/>
                <field name="page_ids"/>
                <field name="partner_id"/>
                <field name="partner_image"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_wvisitor_kanban_card">
                            <!-- displayed in ungrouped mode -->
                            <div class="o_kanban_detail_ungrouped row mx-0">
                                <div class="o_wvisitor_kanban_image">
                                     <img t-if="record.partner_image.raw_value"
                                        t-att-src="kanban_image('res.partner', 'image_128', record.partner_id.raw_value)"
                                        width="54px" height="54px" alt="Visitor"/>
                                     <img t-else=""
                                        t-attf-src="/base/static/img/avatar_grey.png"
                                        width="54px" height="54px" alt="Visitor"/>
                                </div>
                                <div class="col o_wvisitor_name">
                                    <div>
                                        <b><field name="display_name"/></b>
                                        <div class="float-right">
                                            <span class="fa fa-circle text-success" t-if="record.is_connected.raw_value" aria-label="Online" title="Online"/>
                                            <span class="fa fa-circle text-danger" t-else="" aria-label="Offline" title="Offline"/>
                                        </div>
                                        <div>
                                            <img t-if="record.country_id.raw_value"
                                             t-att-src="kanban_image('res.country', 'image', record.country_id.raw_value)"
                                             class="o_country_flag" alt="Country"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="col">
                                    <b><field name="time_since_last_action"/></b>
                                    <div>Last Action</div>
                                </div>
                                <div class="col">
                                    <b><field name="visit_count"/></b>
                                    <div>Visits</div>
                                </div>
                                <div class="col">
                                    <b><field name="last_visited_page_id"/></b>
                                    <div>Last Page</div>
                                </div>
                                <div id="wvisitor_visited_page" class="col">
                                    <b><field name="page_count"/></b>
                                    <div>Visited Pages</div>
                                </div>
                                <div class="col-3 w_visitor_kanban_actions_ungrouped">
                                    <button name="action_send_mail" type="object"
                                            class="btn btn-secondary border" attrs="{'invisible': [('email', '=', False)]}">
                                            Email
                                    </button>
                                </div>
                            </div>
                            <!-- displayed in grouped mode -->
                            <div class="oe_kanban_details">
                                <div class="float-right">
                                    <span class="fa fa-circle text-success" t-if="record.is_connected.raw_value" aria-label="Online" title="Online"/>
                                    <span class="fa fa-circle text-danger" t-else="" aria-label="Offline" title="Offline"/>
                                </div>
                                <strong>
                                    <img t-if="record.country_id.raw_value"
                                         t-att-src="kanban_image('res.country', 'image', record.country_id.raw_value)" class="o_country_flag" alt="Country"/>
                                    <field name="display_name"/>
                                </strong>
                                <div class="mb-2">Active <field name="time_since_last_action"/></div>
                                <div>Last Page<span class="float-right font-weight-bold"><field name="last_visited_page_id"/></span></div>
                                <div>Visits<span class="float-right font-weight-bold"><field name="visit_count"/></span></div>
                                <div id="o_page_count">Visited Pages<span class="float-right font-weight-bold"><field name="page_count"/></span></div>
                                <div class="w_visitor_kanban_actions">
                                    <button name="action_send_mail" type="object"
                                            class="btn btn-secondary" attrs="{'invisible': [('email', '=', False)]}">
                                            Email
                                    </button>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="website_visitor_view_form" model="ir.ui.view">
        <field name="name">website.visitor.view.form</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <form string="Website Visitor">
                <header>
                    <button name="action_send_mail" type="object" class="btn btn-primary"
                            attrs="{'invisible': [('email', '=', False)]}" string="Send Email"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button o_stat_button_info" attrs="{'invisible': [('is_connected', '=', False)]}">
                            <i class="fa fa-fw o_button_icon fa-circle text-success"/>
                            <span>Connected</span>
                        </button>
                        <button class="oe_stat_button o_stat_button_info" attrs="{'invisible': [('is_connected', '=', True)]}">
                            <i class="fa fa-fw o_button_icon fa-circle text-danger"/>
                            <span>Offline</span>
                        </button>
                        <button id="w_visitor_visit_counter" class="oe_stat_button o_stat_button_info" icon="fa-globe">
                            <field name="visit_count" widget="statinfo" string="Visits"/>
                        </button>
                        <button name="%(website.website_visitor_page_action)d" type="action"
                                class="oe_stat_button"
                                icon="fa-tags">
                            <field name="visitor_page_count" widget="statinfo" string="Page views"/>
                        </button>
                    </div>
                    <div class="float-right" attrs="{'invisible': [('country_id', '=', False)]}"><field name="country_flag" widget="image" options='{"size": [32, 32]}'/></div>
                    <div class="oe_title">
                        <h1><field name="display_name"/></h1>
                    </div>
                    <group id="general_info">
                        <group string="Visitor Informations">
                            <field name="is_connected" invisible="1"/>
                            <field name="partner_id" attrs="{'invisible': [('partner_id', '=', False)]}"/>
                            <field name="email"/>
                            <field name="mobile" class="o_force_ltr"/>
                            <field name="country_id" attrs="{'invisible': [('country_id', '=', False)]}"/>
                            <field name="lang_id"/>
                        </group>
                        <group id="visits" string="Visits">
                            <field name="create_date"/>
                            <field name="last_connection_datetime"/>
                            <field name="page_ids" widget="many2many_tags"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="website_visitor_view_tree" model="ir.ui.view">
        <field name="name">website.visitor.view.tree</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <tree string="Web Visitors" decoration-success="is_connected" decoration-danger="not is_connected">
                <!--TODO DBE : Handle no_label in treeview-->
                <field name="country_flag" widget="image" options='{"size": [20, 20]}' string=" "/>
                <field name="display_name"/>
                <field name="create_date"/>
                <field name="last_connection_datetime"/>
                <field name="lang_id"/>
                <field name="visit_count"/>
                <field name="page_ids" widget="many2many_tags"/>
                <field name="is_connected" invisible="1"/>
                <field name="email" invisible="1"/>
                <button string="Send Email" name="action_send_mail" type="object"
                    icon="fa-envelope" attrs="{'invisible': [('email', '=', False)]}"/>
            </tree>
        </field>
    </record>

    <record id="website_visitor_view_graph" model="ir.ui.view">
        <field name="name">website.visitor.view.graph</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <graph string="Visitors last connection">
                <field name="visit_count"/>
            </graph>
        </field>
    </record>

    <record id="website_visitor_view_search" model="ir.ui.view">
        <field name="name">website.visitor.view.search</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <search string="Search Visitor">
                <field name="name"/>
                <field name="lang_id"/>
                <field name="country_id"/>
                <field name="visit_count"/>
                <field name="page_ids"/>
                <filter string="Visitors" name="type_visitor" domain="[('partner_id', '=', False)]"/>
                <filter string="Customers" name="type_customer" domain="[('partner_id', '!=', False)]"/>
                <separator/>
                <filter string="Archived" name="is_archived" domain="[('active', '=', False)]"/>
                <separator/>
                <filter string="Is Connected" name="is_connected" domain="[('last_connection_datetime', '&gt;', datetime.datetime.now() - datetime.timedelta(minutes=5))]"/>

                <group string="Group By">
                    <filter string="Country" name="group_by_country" context="{'group_by': 'country_id'}"/>
                    <filter string="Language" name="group_by_lang" context="{'group_by': 'lang_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="website_visitor_view_graph" model="ir.ui.view">
        <field name="name">website.visitor.view.graph</field>
        <field name="model">website.visitor</field>
        <field name="arch" type="xml">
            <graph string="Visitors">
                <field name="create_date" type="row"/>
            </graph>
        </field>
    </record>

    <record id="website_visitors_action" model="ir.actions.act_window">
        <field name="name">Visitors</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">website.visitor</field>
        <field name="view_mode">kanban,tree,form,graph</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Wait for visitors to come to your website to see their history.
          </p>
        </field>
    </record>

    <record id="website_visitor_track_view_tree" model="ir.ui.view">
        <field name="name">website.track.view.tree</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <tree string="Visitor Views History" create="0" edit="0">
                <field name="visitor_id"/>
                <field name="page_id"/>
                <field name="url"/>
                <field name="visit_datetime"/>
            </tree>
        </field>
    </record>

    <record id="website_visitor_track_view_graph" model="ir.ui.view">
        <field name="name">website.track.view.graph</field>
        <field name="model">website.track</field>
        <field name="arch" type="xml">
            <graph string="Visitor Views">
                <field name="url"/>
            </graph>
        </field>
    </record>

    <record id="website_visitor_view_action" model="ir.actions.act_window">
        <field name="name">Views</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">website.track</field>
        <field name="view_mode">tree</field>
        <field name="context">{'search_default_type_url':1}</field>
        <field name="view_ids" eval="[(5, 0, 0),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('website_visitor_track_view_tree')}),
            (0, 0, {'view_mode': 'graph', 'view_id': ref('website_visitor_track_view_graph')}),
        ]"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Wait for visitors to come to your website to see the pages they viewed.
          </p>
        </field>
    </record>

    <menuitem id="website_visitor_menu"
        name="Visitors"
        sequence="80"
        parent="website.menu_website_configuration"/>

    <menuitem id="menu_visitor_sub_menu" name="Visitors"
        sequence="1"
        parent="website_visitor_menu"
        action="website.website_visitors_action"/>
    <menuitem id="menu_visitor_view_menu" name="Views"
        sequence="2"
        parent="website_visitor_menu"
        action="website.website_visitor_view_action"/>
</data></odoo>

```

## File: wizard\base_language_install.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class BaseLanguageInstall(models.TransientModel):

    _inherit = "base.language.install"

    website_ids = fields.Many2many('website', string='Websites to translate')

    @api.model
    def default_get(self, fields):
        defaults = super(BaseLanguageInstall, self).default_get(fields)
        website_id = self._context.get('params', {}).get('website_id')
        if website_id:
            if 'website_ids' not in defaults:
                defaults['website_ids'] = []
            defaults['website_ids'].append(website_id)
        return defaults

    def lang_install(self):
        action = super(BaseLanguageInstall, self).lang_install()
        lang = self.env['res.lang']._lang_get(self.lang)
        if self.website_ids and lang:
            self.website_ids.write({'language_ids': [(4, lang.id)]})
        params = self._context.get('params', {})
        if 'url_return' in params:
            return {
                'url': params['url_return'].replace('[lang]', self.lang),
                'type': 'ir.actions.act_url',
                'target': 'self'
            }
        return action

```

## File: wizard\base_language_install_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_base_language_install" model="ir.ui.view">
        <field name="name">view_base_language_install.inherit</field>
        <field name="model">base.language.install</field>
        <field name="inherit_id" ref="base.view_base_language_install"/>
        <field name="arch" type="xml">
            <group states="init" position="inside">
                <field name="website_ids" widget="many2many_checkboxes" groups="website.group_multi_website"/>
            </group>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_language_install

```

