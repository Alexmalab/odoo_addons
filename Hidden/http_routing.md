# Odoo Module: http_routing

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

from odoo.http import request


def _post_init_hook(cr, registry):
    if request:
        request.is_frontend = False
        request.is_frontend_multilang = False

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Web Routing',
    'summary': 'Web Routing',
    'sequence': '9100',
    'category': 'Hidden',
    'description': """
Proposes advanced routing options not available in web or base to keep
base modules simple.
""",
    'data': [
        'views/http_routing_template.xml',
        'views/res_lang_views.xml',
    ],
    'post_init_hook': '_post_init_hook',
    'depends': ['web'],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.web.controllers.home import Home
from odoo.addons.web.controllers.session import Session
from odoo.addons.web.controllers.webclient import WebClient


class Routing(Home):

    @http.route('/website/translations/<string:unique>', type='http', auth="public", website=True)
    def get_website_translations(self, unique, lang=None, mods=None):
        IrHttp = request.env['ir.http'].sudo()
        modules = IrHttp.get_translation_frontend_modules()
        if mods:
            modules += mods.split(',')
        return WebClient().translations(unique, mods=','.join(modules), lang=lang)


class SessionWebsite(Session):

    @http.route('/web/session/logout', type='http', auth="none", website=True, multilang=False, sitemap=False)
    def logout(self, redirect='/web'):
        return super().logout(redirect=redirect)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import contextlib
import logging
import os
import re
import traceback
import threading
import unicodedata
import werkzeug.exceptions
import werkzeug.routing
import werkzeug.urls
from werkzeug.exceptions import HTTPException, NotFound

# optional python-slugify import (https://github.com/un33k/python-slugify)
try:
    import slugify as slugify_lib
except ImportError:
    slugify_lib = None

import odoo
from odoo import api, models, exceptions, tools, http
from odoo.addons.base.models import ir_http
from odoo.addons.base.models.ir_http import RequestUID
from odoo.addons.base.models.ir_qweb import QWebException
from odoo.http import request, HTTPRequest, Response
from odoo.osv import expression
from odoo.tools import config, ustr, pycompat

_logger = logging.getLogger(__name__)

# ------------------------------------------------------------
# Slug API
# ------------------------------------------------------------

def _guess_mimetype(ext=False, default='text/html'):
    exts = {
        '.css': 'text/css',
        '.less': 'text/less',
        '.scss': 'text/scss',
        '.js': 'text/javascript',
        '.xml': 'text/xml',
        '.csv': 'text/csv',
        '.html': 'text/html',
    }
    return ext is not False and exts.get(ext, default) or exts


def slugify_one(s, max_length=0):
    """ Transform a string to a slug that can be used in a url path.
        This method will first try to do the job with python-slugify if present.
        Otherwise it will process string by stripping leading and ending spaces,
        converting unicode chars to ascii, lowering all chars and replacing spaces
        and underscore with hyphen "-".
        :param s: str
        :param max_length: int
        :rtype: str
    """
    s = ustr(s)
    if slugify_lib:
        # There are 2 different libraries only python-slugify is supported
        try:
            return slugify_lib.slugify(s, max_length=max_length)
        except TypeError:
            pass
    uni = unicodedata.normalize('NFKD', s).encode('ascii', 'ignore').decode('ascii')
    slug_str = re.sub(r'[\W_]', ' ', uni).strip().lower()
    slug_str = re.sub(r'[-\s]+', '-', slug_str)
    return slug_str[:max_length] if max_length > 0 else slug_str


def slugify(s, max_length=0, path=False):
    if not path:
        return slugify_one(s, max_length=max_length)
    else:
        res = []
        for u in s.split('/'):
            if slugify_one(u, max_length=max_length) != '':
                res.append(slugify_one(u, max_length=max_length))
        # check if supported extension
        path_no_ext, ext = os.path.splitext(s)
        if ext and ext in _guess_mimetype():
            res[-1] = slugify_one(path_no_ext) + ext
        return '/'.join(res)


def slug(value):
    try:
        if not value.id:
            raise ValueError("Cannot slug non-existent record %s" % value)
        # [(id, name)] = value.name_get()
        identifier, name = value.id, getattr(value, 'seo_name', False) or value.display_name
    except AttributeError:
        # assume name_search result tuple
        identifier, name = value
    slugname = slugify(name or '').strip().strip('-')
    if not slugname:
        return str(identifier)
    return f"{slugname}-{identifier}"


# NOTE: the second pattern is used for the ModelConverter, do not use nor flags nor groups
_UNSLUG_RE = re.compile(r'(?:(\w{1,2}|\w[A-Za-z0-9-_]+?\w)-)?(-?\d+)(?=$|/)')
_UNSLUG_ROUTE_PATTERN = r'(?:(?:\w{1,2}|\w[A-Za-z0-9-_]+?\w)-)?(?:-?\d+)(?=$|/)'


def unslug(s):
    """Extract slug and id from a string.
        Always return un 2-tuple (str|None, int|None)
    """
    m = _UNSLUG_RE.match(s)
    if not m:
        return None, None
    return m.group(1), int(m.group(2))


def unslug_url(s):
    """ From /blog/my-super-blog-1" to "blog/1" """
    parts = s.split('/')
    if parts:
        unslug_val = unslug(parts[-1])
        if unslug_val[1]:
            parts[-1] = str(unslug_val[1])
            return '/'.join(parts)
    return s


# ------------------------------------------------------------
# Language tools
# ------------------------------------------------------------

def url_lang(path_or_uri, lang_code=None):
    ''' Given a relative URL, make it absolute and add the required lang or
        remove useless lang.
        Nothing will be done for absolute or invalid URL.
        If there is only one language installed, the lang will not be handled
        unless forced with `lang` parameter.

        :param lang_code: Must be the lang `code`. It could also be something
                          else, such as `'[lang]'` (used for url_return).
    '''
    Lang = request.env['res.lang']
    location = pycompat.to_text(path_or_uri).strip()
    force_lang = lang_code is not None
    try:
        url = werkzeug.urls.url_parse(location)
    except ValueError:
        # e.g. Invalid IPv6 URL, `werkzeug.urls.url_parse('http://]')`
        url = False
    # relative URL with either a path or a force_lang
    if url and not url.netloc and not url.scheme and (url.path or force_lang):
        location = werkzeug.urls.url_join(request.httprequest.path, location)
        lang_url_codes = [url_code for _, url_code, *_ in Lang.get_available()]
        lang_code = pycompat.to_text(lang_code or request.context['lang'])
        lang_url_code = Lang._lang_code_to_urlcode(lang_code)
        lang_url_code = lang_url_code if lang_url_code in lang_url_codes else lang_code
        if (len(lang_url_codes) > 1 or force_lang) and is_multilang_url(location, lang_url_codes):
            loc, sep, qs = location.partition('?')
            ps = loc.split(u'/')
            default_lg = request.env['ir.http']._get_default_lang()
            if ps[1] in lang_url_codes:
                # Replace the language only if we explicitly provide a language to url_for
                if force_lang:
                    ps[1] = lang_url_code
                # Remove the default language unless it's explicitly provided
                elif ps[1] == default_lg.url_code:
                    ps.pop(1)
            # Insert the context language or the provided language
            elif lang_url_code != default_lg.url_code or force_lang:
                ps.insert(1, lang_url_code)

            location = u'/'.join(ps) + sep + qs
    return location


def url_for(url_from, lang_code=None, no_rewrite=False):
    ''' Return the url with the rewriting applied.
        Nothing will be done for absolute URL, invalid URL, or short URL from 1 char.

        :param url_from: The URL to convert.
        :param lang_code: Must be the lang `code`. It could also be something
                          else, such as `'[lang]'` (used for url_return).
        :param no_rewrite: don't try to match route with website.rewrite.
    '''
    new_url = False

    # don't try to match route if we know that no rewrite has been loaded.
    routing = getattr(request, 'website_routing', None)  # not modular, but not overridable
    if not getattr(request.env['ir.http'], '_rewrite_len', {}).get(routing):
        no_rewrite = True

    path, _, qs = (url_from or '').partition('?')

    if (not no_rewrite and path and (
            len(path) > 1
            and path.startswith('/')
            and '/static/' not in path
            and not path.startswith('/web/')
    )):
        new_url, _ = request.env['ir.http'].url_rewrite(path)
        new_url = new_url if not qs else new_url + '?%s' % qs

    return url_lang(new_url or url_from, lang_code=lang_code)


def is_multilang_url(local_url, lang_url_codes=None):
    ''' Check if the given URL content is supposed to be translated.
        To be considered as translatable, the URL should either:
        1. Match a POST (non-GET actually) controller that is `website=True` and
           either `multilang` specified to True or if not specified, with `type='http'`.
        2. If not matching 1., everything not under /static/ or /web/ will be translatable
    '''
    if not lang_url_codes:
        lang_url_codes = [url_code for _, url_code, *_ in request.env['res.lang'].get_available()]
    spath = local_url.split('/')
    # if a language is already in the path, remove it
    if spath[1] in lang_url_codes:
        spath.pop(1)
        local_url = '/'.join(spath)

    url = local_url.partition('#')[0].split('?')
    path = url[0]

    # Consider /static/ and /web/ files as non-multilang
    if '/static/' in path or path.startswith('/web/'):
        return False

    query_string = url[1] if len(url) > 1 else None

    # Try to match an endpoint in werkzeug's routing table
    try:
        _, func = request.env['ir.http'].url_rewrite(path, query_args=query_string)

        # /page/xxx has no endpoint/func but is multilang
        return (not func or (
            func.routing.get('website', False)
            and func.routing.get('multilang', func.routing['type'] == 'http')
        ))
    except Exception as exception:
        _logger.warning(exception)
        return False


class ModelConverter(ir_http.ModelConverter):

    def __init__(self, url_map, model=False, domain='[]'):
        super(ModelConverter, self).__init__(url_map, model)
        self.domain = domain
        self.regex = _UNSLUG_ROUTE_PATTERN

    def to_url(self, value):
        return slug(value)

    def to_python(self, value):
        matching = _UNSLUG_RE.match(value)
        _uid = RequestUID(value=value, match=matching, converter=self)
        record_id = int(matching.group(2))
        env = api.Environment(request.cr, _uid, request.context)

        if record_id < 0:
            # limited support for negative IDs due to our slug pattern, assume abs() if not found
            if not env[self.model].browse(record_id).exists():
                record_id = abs(record_id)
        return env[self.model].with_context(_converter_value=value).browse(record_id)


class IrHttp(models.AbstractModel):
    _inherit = ['ir.http']

    rerouting_limit = 10

    @classmethod
    def _get_converters(cls):
        """ Get the converters list for custom url pattern werkzeug need to
            match Rule. This override adds the website ones.
        """
        return dict(
            super(IrHttp, cls)._get_converters(),
            model=ModelConverter,
        )

    @classmethod
    def _get_default_lang(cls):
        lang_code = request.env['ir.default'].sudo().get('res.partner', 'lang')
        if lang_code:
            return request.env['res.lang']._lang_get(lang_code)
        return request.env['res.lang'].search([], limit=1)

    @api.model
    def get_frontend_session_info(self):
        session_info = super(IrHttp, self).get_frontend_session_info()

        IrHttpModel = request.env['ir.http'].sudo()
        modules = IrHttpModel.get_translation_frontend_modules()
        user_context = request.session.context if request.session.uid else {}
        lang = user_context.get('lang')
        translation_hash = request.env['ir.http'].get_web_translations_hash(modules, lang)

        session_info.update({
            'translationURL': '/website/translations',
            'cache_hashes': {
                'translations': translation_hash,
            },
        })
        return session_info

    @api.model
    def get_translation_frontend_modules(self):
        Modules = request.env['ir.module.module'].sudo()
        extra_modules_domain = self._get_translation_frontend_modules_domain()
        extra_modules_name = self._get_translation_frontend_modules_name()
        if extra_modules_domain:
            new = Modules.search(
                expression.AND([extra_modules_domain, [('state', '=', 'installed')]])
            ).mapped('name')
            extra_modules_name += new
        return extra_modules_name

    @classmethod
    def _get_translation_frontend_modules_domain(cls):
        """ Return a domain to list the domain adding web-translations and
            dynamic resources that may be used frontend views
        """
        return []

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        """ Return a list of module name where web-translations and
            dynamic resources may be used in frontend views
        """
        return ['web']

    @classmethod
    def _get_frontend_langs(cls):
        return [code for code, _ in request.env['res.lang'].get_installed()]

    @classmethod
    def get_nearest_lang(cls, lang_code):
        """ Try to find a similar lang. Eg: fr_BE and fr_FR
            :param lang_code: the lang `code` (en_US)
        """
        if not lang_code:
            return None

        lang_codes = cls._get_frontend_langs()
        if lang_code in lang_codes:
            return lang_code

        short = lang_code.partition('_')[0]
        return next((code for code in lang_codes if code.startswith(short)), None)

    @classmethod
    def _match(cls, path):
        """
        Grant multilang support to URL matching by using http 3xx
        redirections and URL rewrite. This method also grants various
        attributes such as ``lang`` and ``is_frontend`` on the current
        ``request`` object.

        1/ Use the URL as-is when it matches a non-multilang compatible
           endpoint.

        2/ Use the URL as-is when the lang is not present in the URL and
           that the default lang has been requested.

        3/ Use the URL as-is saving the requested lang when the user is
           a bot and that the lang is missing from the URL.

        4) Use the url as-is when the lang is missing from the URL, that
           another lang than the default one has been requested but that
           it is forbidden to redirect (e.g. POST)

        5/ Redirect the browser when the lang is missing from the URL
           but another lang than the default one has been requested. The
           requested lang is injected before the original path.

        6/ Redirect the browser when the lang is present in the URL but
           it is the default lang. The lang is removed from the original
           URL.

        7/ Redirect the browser when the lang present in the URL is an
           alias of the preferred lang url code (e.g. fr_FR -> fr)

        8/ Redirect the browser when the requested page is the homepage
           but that there is a trailing slash.

        9/ Rewrite the URL when the lang is present in the URL, that it
           matches and that this lang is not the default one. The URL is
           rewritten to remove the lang.

        Note: The "requested lang" is (in order) either (1) the lang in
              the URL or (2) the lang in the ``frontend_lang`` request
              cookie or (3) the lang in the context or (4) the default
              lang of the website.
        """

        # The URL has been rewritten already
        if hasattr(request, 'is_frontend'):
            return super()._match(path)

        # See /1, match a non website endpoint
        try:
            rule, args = super()._match(path)
            routing = rule.endpoint.routing
            request.is_frontend = routing.get('website', False)
            request.is_frontend_multilang = request.is_frontend and routing.get('multilang', routing['type'] == 'http')
            if not request.is_frontend:
                return rule, args
        except NotFound:
            _, url_lang_str, *rest = path.split('/', 2)
            path_no_lang = '/' + (rest[0] if rest else '')
        else:
            url_lang_str = ''
            path_no_lang = path

        allow_redirect = (
            request.httprequest.method != 'POST'
            and getattr(request, 'is_frontend_multilang', True)
        )

        # Some URLs in website are concatenated, first url ends with /,
        # second url starts with /, resulting url contains two following
        # slashes that must be merged.
        if allow_redirect and '//' in path:
            new_url = path.replace('//', '/')
            werkzeug.exceptions.abort(request.redirect(new_url, code=301, local=True))

        # There is no user on the environment yet but the following code
        # requires one to set the lang on the request. Temporary grant
        # the public user. Don't try it at home!
        real_env = request.env
        try:
            request.registry['ir.http']._auth_method_public()  # it calls update_env
            nearest_url_lang = cls.get_nearest_lang(request.env['res.lang']._lang_get_code(url_lang_str))
            cookie_lang = cls.get_nearest_lang(request.httprequest.cookies.get('frontend_lang'))
            context_lang = cls.get_nearest_lang(real_env.context.get('lang'))
            default_lang = cls._get_default_lang()
            request.lang = request.env['res.lang']._lang_get(
                nearest_url_lang or cookie_lang or context_lang or default_lang._get_cached('code')
            )
            request_url_code = request.lang._get_cached('url_code')
        finally:
            request.env = real_env

        if not nearest_url_lang:
            url_lang_str = None

        # See /2, no lang in url and default website
        if not url_lang_str and request.lang == default_lang:
            _logger.debug("%r (lang: %r) no lang in url and default website, continue", path, request_url_code)

        # See /3, missing lang in url but user-agent is a bot
        elif not url_lang_str and request.env['ir.http'].is_a_bot():
            _logger.debug("%r (lang: %r) missing lang in url but user-agent is a bot, continue", path, request_url_code)
            request.lang = default_lang

        # See /4, no lang in url and should not redirect (e.g. POST), continue
        elif not url_lang_str and not allow_redirect:
            _logger.debug("%r (lang: %r) no lang in url and should not redirect (e.g. POST), continue", path, request_url_code)

        # See /5, missing lang in url, /home -> /fr/home
        elif not url_lang_str:
            _logger.debug("%r (lang: %r) missing lang in url, redirect", path, request_url_code)
            redirect = request.redirect_query(f'/{request_url_code}{path}', request.httprequest.args)
            redirect.set_cookie('frontend_lang', request.lang._get_cached('code'), max_age=365 * 24 * 3600)
            werkzeug.exceptions.abort(redirect)

        # See /6, default lang in url, /en/home -> /home
        elif url_lang_str == default_lang.url_code and allow_redirect:
            _logger.debug("%r (lang: %r) default lang in url, redirect", path, request_url_code)
            redirect = request.redirect_query(path_no_lang, request.httprequest.args)
            redirect.set_cookie('frontend_lang', default_lang._get_cached('code'), max_age=365 * 24 * 3600)
            werkzeug.exceptions.abort(redirect)

        # See /7, lang alias in url, /fr_FR/home -> /fr/home
        elif url_lang_str != request_url_code and allow_redirect:
            _logger.debug("%r (lang: %r) lang alias in url, redirect", path, request_url_code)
            redirect = request.redirect_query(f'/{request_url_code}{path_no_lang}', request.httprequest.args, code=301)
            redirect.set_cookie('frontend_lang', request.lang._get_cached('code'), max_age=365 * 24 * 3600)
            werkzeug.exceptions.abort(redirect)

        # See /8, homepage with trailing slash. /fr_BE/ -> /fr_BE
        elif path == f'/{url_lang_str}/' and allow_redirect:
            _logger.debug("%r (lang: %r) homepage with trailing slash, redirect", path, request_url_code)
            redirect = request.redirect_query(path[:-1], request.httprequest.args, code=301)
            redirect.set_cookie('frontend_lang', default_lang._get_cached('code'), max_age=365 * 24 * 3600)
            werkzeug.exceptions.abort(redirect)

        # See /9, valid lang in url
        elif url_lang_str == request_url_code:
            # Rewrite the URL to remove the lang
            _logger.debug("%r (lang: %r) valid lang in url, rewrite url and continue", path, request_url_code)
            cls.reroute(path_no_lang)
            path = path_no_lang

        else:
            _logger.warning("%r (lang: %r) couldn't not correctly route this frontend request, url used as-is.", path, request_url_code)

        # Re-match using rewritten route and really raise for 404 errors
        try:
            rule, args = super()._match(path)
            routing = rule.endpoint.routing
            request.is_frontend = routing.get('website', False)
            request.is_frontend_multilang = request.is_frontend and routing.get('multilang', routing['type'] == 'http')
            return rule, args
        except NotFound:
            # Use website to render a nice 404 Not Found html page
            request.is_frontend = True
            request.is_frontend_multilang = True
            raise

    @classmethod
    def reroute(cls, path, query_string=None):
        """
        Rewrite the current request URL using the new path and query
        string. This act as a light redirection, it does not return a
        3xx responses to the browser but still change the current URL.
        """
        # WSGI encoding dance https://peps.python.org/pep-3333/#unicode-issues
        if isinstance(path, str):
            path = path.encode('utf-8')
        path = path.decode('latin1', 'replace')

        if query_string is None:
            query_string = request.httprequest.environ['QUERY_STRING']

        # Change the WSGI environment
        environ = request.httprequest._HTTPRequest__environ.copy()
        environ['PATH_INFO'] = path
        environ['QUERY_STRING'] = query_string
        environ['RAW_URI'] = f'{path}?{query_string}'
        # REQUEST_URI left as-is so it still contains the original URI

        # Create and expose a new request from the modified WSGI env
        httprequest = HTTPRequest(environ)
        threading.current_thread().url = httprequest.url
        request.httprequest = httprequest

    @classmethod
    def _pre_dispatch(cls, rule, args):
        super()._pre_dispatch(rule, args)

        if request.is_frontend:
            cls._frontend_pre_dispatch()

        # update the context of "<model(...):...>" args
        for key, val in list(args.items()):
            if isinstance(val, models.BaseModel):
                args[key] = val.with_context(request.context)

        if request.is_frontend_multilang:
            # A product with id 1 and named 'egg' is accessible via a
            # frontend multilang enpoint 'foo' at the URL '/foo/1'.
            # The preferred URL to access the product (and to generate
            # URLs pointing it) should instead be the sluggified URL
            # '/foo/egg-1'. This code is responsible of redirecting the
            # browser from '/foo/1' to '/foo/egg-1', or '/fr/foo/1' to
            # '/fr/foo/oeuf-1'. While it is nice (for humans) to have a
            # pretty URL, the real reason of this redirection is SEO.
            if request.httprequest.method in ('GET', 'HEAD'):
                try:
                    _, path = rule.build(args)
                except odoo.exceptions.MissingError:
                    raise werkzeug.exceptions.NotFound()
                assert path is not None
                generated_path = werkzeug.urls.url_unquote_plus(path)
                current_path = werkzeug.urls.url_unquote_plus(request.httprequest.path)
                if generated_path != current_path:
                    if request.lang != cls._get_default_lang():
                        path = f'/{request.lang.url_code}{path}'
                    redirect = request.redirect_query(path, request.httprequest.args, code=301)
                    werkzeug.exceptions.abort(redirect)

    @classmethod
    def _frontend_pre_dispatch(cls):
        request.update_context(lang=request.lang._get_cached('code'))
        if request.httprequest.cookies.get('frontend_lang') != request.lang._get_cached('code'):
            request.future_response.set_cookie('frontend_lang', request.lang._get_cached('code'), max_age=365 * 24 * 3600)

    @classmethod
    def _get_exception_code_values(cls, exception):
        """ Return a tuple with the error code following by the values matching the exception"""
        code = 500  # default code
        values = dict(
            exception=exception,
            traceback=traceback.format_exc(),
        )
        if isinstance(exception, exceptions.AccessDenied):
            code = 403
        elif isinstance(exception, exceptions.UserError):
            values['error_message'] = exception.args[0]
            code = 400
            if isinstance(exception, exceptions.AccessError):
                code = 403

        elif isinstance(exception, QWebException):
            values.update(qweb_exception=exception)

            if isinstance(exception.__context__, exceptions.UserError):
                code = 400
                values['error_message'] = exception.__context__.args[0]
                if isinstance(exception.__context__, exceptions.AccessError):
                    code = 403

        elif isinstance(exception, werkzeug.exceptions.HTTPException):
            code = exception.code

        values.update(
            status_message=werkzeug.http.HTTP_STATUS_CODES.get(code, ''),
            status_code=code,
        )

        return (code, values)

    @classmethod
    def _get_values_500_error(cls, env, values, exception):
        values['view'] = env["ir.ui.view"]
        return values

    @classmethod
    def _get_error_html(cls, env, code, values):
        return code, env['ir.ui.view']._render_template('http_routing.%s' % code, values)

    @classmethod
    def _handle_error(cls, exception):
        response = super()._handle_error(exception)

        is_frontend_request = bool(getattr(request, 'is_frontend', False))
        if not is_frontend_request or not isinstance(response, HTTPException):
            # neither handle backend requests nor plain responses
            return response

        # minimal setup to serve frontend pages
        if not request.uid:
            cls._auth_method_public()
        cls._handle_debug()
        cls._frontend_pre_dispatch()
        request.params = request.get_http_params()

        code, values = cls._get_exception_code_values(exception)

        request.cr.rollback()
        if code in (404, 403):
            try:
                response = cls._serve_fallback()
                if response:
                    cls._post_dispatch(response)
                    return response
            except werkzeug.exceptions.Forbidden:
                # Rendering does raise a Forbidden if target is not visible.
                pass # Use default error page handling.
        elif code == 500:
            values = cls._get_values_500_error(request.env, values, exception)
        try:
            code, html = cls._get_error_html(request.env, code, values)
        except Exception:
            code, html = 418, request.env['ir.ui.view']._render_template('http_routing.http_error', values)

        response = Response(html, status=code, content_type='text/html;charset=utf-8')
        cls._post_dispatch(response)
        return response

    @api.model
    @tools.ormcache('path', 'query_args')
    def url_rewrite(self, path, query_args=None):
        new_url = False
        router = http.root.get_db_router(request.db).bind('')
        endpoint = False
        try:
            endpoint = router.match(path, method='POST', query_args=query_args)
        except werkzeug.exceptions.MethodNotAllowed:
            endpoint = router.match(path, method='GET', query_args=query_args)
        except werkzeug.routing.RequestRedirect as e:
            new_url = e.new_url.split('?')[0][7:]  # remove scheme
            _, endpoint = self.url_rewrite(new_url, query_args)
            endpoint = endpoint and [endpoint]
        except werkzeug.exceptions.NotFound:
            new_url = path
        return new_url or path, endpoint and endpoint[0]

```

## File: models\ir_qweb.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import slug, unslug_url, url_for


class IrQweb(models.AbstractModel):
    _inherit = "ir.qweb"

    def _prepare_environment(self, values):
        irQweb = super()._prepare_environment(values)
        values['slug'] = slug
        values['unslug_url'] = unslug_url

        if (not irQweb.env.context.get('minimal_qcontext') and
                request and request.is_frontend):
            return irQweb._prepare_frontend_environment(values)

        return irQweb

    def _prepare_frontend_environment(self, values):
        values['url_for'] = url_for
        return self

```

## File: models\res_lang.py

```python

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import ir_qweb

```

## File: static\shapes\404.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="310" height="120" viewBox="0 0 310 120">
    <g>
        <path fill="#7C6576" d="M205.104,48.417c-0.688-26.971-22.805-48.624-49.983-48.624c-26.929,0-48.883,21.258-49.954,47.883h-0.045V120c8.337,0,8.337-14.977,16.674-14.977c8.338,0,8.338,14.977,16.674,14.977c8.332,0,8.332-14.977,16.664-14.977c8.335,0,8.335,14.977,16.668,14.977c8.332,0,8.332-14.977,16.661-14.977c8.33,0,8.33,14.977,16.659,14.977V48.418L205.104,48.417zM146.152,64.963c-6.698,0-12.126-5.419-12.126-12.107s5.429-12.106,12.126-12.106c5.839,0,10.715,4.122,11.866,9.612c-1.011-1.197-2.521-1.958-4.21-1.958c-3.044,0-5.511,2.462-5.511,5.501c0,3.04,2.466,5.505,5.511,5.505c1.147,0,2.21-0.351,3.093-0.95C154.875,62.325,150.822,64.963,146.152,64.963z M180.425,64.963c-6.696,0-12.126-5.419-12.126-12.107s5.43-12.106,12.126-12.106c5.841,0,10.716,4.122,11.867,9.612c-1.011-1.197-2.522-1.958-4.212-1.958c-3.044,0-5.512,2.462-5.512,5.501c0,3.04,2.468,5.505,5.512,5.505c1.146,0,2.211-0.351,3.093-0.95C189.148,62.325,185.096,64.963,180.425,64.963z"/>
    </g>
    <g>
        <path fill="#21323A" d="M308.002,84.182h-24.3V120h-5.04V84.182h-59.758v-3.96l58.139-79.017h6.659v78.656h24.3V84.182zM224.484,79.861h54.178V18.304l0.18-12.419h-0.359c-2.52,3.96-5.76,8.64-8.459,12.419L224.484,79.861z"/>
    </g>
    <g>
        <path fill="#21323A" d="M90.002,84.182H65.703V120h-5.04V84.182H0.906v-3.96L59.043,1.205h6.66v78.656h24.299V84.182zM6.486,79.861h54.178V18.304l0.18-12.419h-0.36c-2.52,3.96-5.76,8.64-8.459,12.419L6.486,79.861z"/>
    </g>
    <desc>
        <!-- Monkey HTML formatted-text fallback -->
        <h1 style="font-size:12em;line-height:1;">404</h1>
    </desc>
</svg>

```

## File: views\http_routing_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="http_error">
        <t t-call="web.frontend_layout">
            <div id="wrap">
                <div class="oe_structure">
                    <h1 class="container mt32"><t t-esc="status_code"/>: <t t-esc="status_message"/></h1>
                </div>
                <t t-if="editable or debug">
                    <t t-call="http_routing.http_error_debug"/>
                </t>
            </div>
        </t>
    </template>

    <template id="error_message">
        <p t-if="error_message">
            <strong>Error message:</strong>
            <pre t-esc="error_message"/>
        </p>
    </template>

    <template id="http_error_debug">
        <div class="container accordion mb32 mt32" id="debug_infos">
            <div class="card" t-if="error_message">
                <h4 class="card-header">
                    <a data-bs-toggle="collapse" href="#error_main">Error</a>
                </h4>
                <div id="error_main" class="collapse show">
                    <div class="card-body">
                        <t t-call="http_routing.error_message"/>
                    </div>
                </div>
            </div>
            <div class="card" t-if="qweb_exception">
                <h4 class="card-header">
                    <a data-bs-toggle="collapse" href="#error_qweb">QWeb</a>
                </h4>
                <div id="error_qweb" class="collapse show">
                    <div class="card-body">
                        <p>
                            <strong>Error message:</strong>
                            <pre t-esc="exception"/>
                        </p>
                        <p>
                            The error occurred while rendering the template <code t-esc="qweb_exception.name"/> 
                            <t t-if="qweb_exception.html">and evaluating the following expression: <code t-esc="qweb_exception.html"/></t>
                        </p>
                    </div>
                </div>
            </div>
            <div class="card" t-if="traceback">
                <h4 class="card-header">
                    <a data-bs-toggle="collapse" href="#error_traceback">Traceback</a>
                </h4>
                <div id="error_traceback" t-attf-class="collapse #{not error_message and not qweb_exception and 'show'}">
                    <div class="card-body">
                        <pre id="exception_traceback" t-esc="traceback"/>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="400">
        <t t-call="web.frontend_layout">
            <div id="wrap">
                <div class="container">
                    <h1 class="mt-5">Oops! Something went wrong.</h1>
                    <p>Take a look at the error message below.</p>
                </div>
                <t t-if="editable or debug">
                    <t t-call="http_routing.http_error_debug"/>
                </t>
                <t t-else="">
                    <div class="container">
                        <t t-call="http_routing.error_message"/>
                    </div>
                </t>
        </div>
        </t>
    </template>

    <template id="403">
        <t t-call="web.frontend_layout">
            <div id="wrap">
                <div class="container">
                    <h1 class="mt-5">403: Forbidden</h1>
                    <p>The page you were looking for could not be authorized.</p>
                </div>
                    <t t-if="editable or debug">
                        <t t-call="http_routing.http_error_debug"/>
                    </t>
                    <t t-else="">
                        <div class="container">
                            <t t-call="http_routing.error_message"/>
                        </div>
                    </t>
            </div>
        </t>
    </template>

    <template id="404" name="Page Not Found">
        <t t-call="web.frontend_layout">
            <div id="wrap">
                <t t-out="0"/>
                <div class="oe_structure oe_empty">
                    <section class="s_text_image pt104 pb104" data-snippet="s_image_text" data-name="Image - Text">
                        <div class="container">
                            <div class="row align-items-center">
                                <div class="col-lg-4 pb32">
                                    <img class="img img-fluid mx-auto" src="/web_editor/shape/http_routing/404.svg?c2=o-color-2"/>
                                </div>
                                <div class="col-lg-8 text-lg-start text-center my-auto">
                                    <h1 class="visually-hidden">Error 404</h1>
                                    <h2>We couldn't find the page you're looking for!</h2>
                                    <p></p>
                                    <p><b>Don't panic.</b> If you think it's our mistake, please send us a message on <a href="/contactus">this page</a>.</p>
                                </div>
                                <div class="col-lg-12">
                                    <div class="s_hr pt32 pb48" data-snippet="s_hr" data-name="Separator">
                                        <hr class="w-100 mx-auto" style="border-top-width: 1px; border-top-style: solid; border-top-color: var(--400);"/>
                                    </div>
                                </div>
                                <div class="col-lg-12">
                                    <p class="text-center">Maybe you were looking for one of these <b>popular pages?</b></p>
                                    <ul class="list-inline text-center">
                                        <li class="list-inline-item mb-2"><a href="/" class="btn btn-primary">Home</a></li>
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </section>
                </div>
            </div>
        </t>
    </template>

    <template id="500">
        <!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
        <!-- This template should not use any variable except those provided by http_routing.ir_http._handle_exception  -->
        <!--    no request.crsf_token, no theme style, no assets, ... cursor can be broken during rendering !      -->
        <!--    see test_05_reset_specific_view_controller_broken_request                                          -->
        <!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
        <html>
            <head>
                <title t-esc="status_message">Internal Server Error</title>

                <link rel="stylesheet" href="/web/static/lib/bootstrap/dist/css/bootstrap.css"/>
                <script src="/web/static/lib/jquery/jquery.js" type="text/javascript"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/dom/data.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/dom/event-handler.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/dom/manipulator.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/dom/selector-engine.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/base-component.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/dist/collapse.js"/>
                <style>
                    html {
                        font-size: 14px;
                    }
                </style>
            </head>
            <body>
                <div id="wrapwrap">
                    <header>
                        <div class="navbar navbar-expand-md navbar-light bg-light">
                            <div class="container">
                                <div class="collapse navbar-collapse navbar-top-collapse">
                                    <ul class="navbar-nav ms-auto" id="top_menu">
                                        <li class="nav-item"><a href="/" class="nav-link">Home</a></li>
                                        <li class="nav-item"><a href="javascript: window.history.back()" class="nav-link">Back</a></li>
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </header>
                    <main>
                        <div id="error_message" class="oe_structure">
                            <h2 class="container mt32"><t t-esc="status_code"/>: <t t-esc="status_message"/></h2>
                        </div>

                        <t t-if="editable or debug">
                            <t t-call="http_routing.http_error_debug"/>
                        </t>
                    </main>
                </div>
            </body>
        </html>
    </template>
</odoo>

```

## File: views\res_lang_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_lang_form_inherit_model" model="ir.ui.view">
        <field name="name">res.lang.form.http_routing.inherit</field>
        <field name="model">res.lang</field>
        <field name="inherit_id" ref="base.res_lang_form"/>
        <field name="arch" type="xml">
            <field name="url_code" position="attributes">
                <attribute name="invisible">0</attribute>
            </field>
        </field>
    </record>

    <record id="res_lang_tree_inherit_model" model="ir.ui.view">
        <field name="name">res.lang.tree.model.inherit</field>
        <field name="model">res.lang</field>
        <field name="inherit_id" ref="base.res_lang_tree"/>
        <field name="arch" type="xml">
            <field name="url_code" position="attributes">
                <attribute name="invisible">0</attribute>
            </field>
        </field>
    </record>
</odoo>

```

