# Odoo Module: http_routing

Category: Hidden

This file contains the source code of the Odoo module.

## File: geoipresolver.py

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import os.path

try:
    import GeoIP    # Legacy
except ImportError:
    GeoIP = None

try:
    import geoip2
    import geoip2.database
except ImportError:
    geoip2 = None

class GeoIPResolver(object):
    def __init__(self, fname):
        self.fname = fname
        try:
            self._db = geoip2.database.Reader(fname)
            self.version = 2
        except Exception:
            try:
                self._db = GeoIP.open(fname, GeoIP.GEOIP_STANDARD)
                self.version = 1
                assert self._db.database_info is not None
            except Exception:
                raise ValueError('Invalid GeoIP database: %r' % fname)

    def __del__(self):
        if self.version == 2:
            self._db.close()

    @classmethod
    def open(cls, fname):
        if not GeoIP and not geoip2:
            return None
        if not os.path.exists(fname):
            return None
        return GeoIPResolver(fname)

    def resolve(self, ip):
        if self.version == 1:
            return self._db.record_by_addr(ip) or {}
        elif self.version == 2:
            try:
                r = self._db.city(ip)
            except (ValueError, geoip2.errors.AddressNotFoundError):
                return {}
            # Compatibility with Legacy database.
            # Some ips cannot be located to a specific country. Legacy DB used to locate them in
            # continent instead of country. Do the same to not change behavior of existing code.
            country, attr = (r.country, 'iso_code') if r.country.geoname_id else (r.continent, 'code')
            return {
                'city': r.city.name,
                'country_code': getattr(country, attr),
                'country_name': country.name,
                'latitude': r.location.latitude,
                'longitude': r.location.longitude,
                'region': r.subdivisions[0].iso_code if r.subdivisions else None,
                'time_zone': r.location.time_zone,
            }

    # compat
    def record_by_addr(self, addr):
        return self.resolve(addr)

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
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
    'depends': ['web'],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.web.controllers.main import WebClient, Home, Session


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
# -*- coding: utf-8 -*-

import logging
import os
import re
import traceback
import unicodedata
import werkzeug.exceptions
import werkzeug.routing
import werkzeug.urls

# optional python-slugify import (https://github.com/un33k/python-slugify)
try:
    import slugify as slugify_lib
except ImportError:
    slugify_lib = None

import odoo
from odoo import api, models, registry, exceptions, tools, http
from odoo.addons.base.models import ir_http
from odoo.addons.base.models.ir_http import RequestUID
from odoo.addons.base.models.qweb import QWebException
from odoo.http import request, HTTPRequest
from odoo.osv import expression
from odoo.tools import config, ustr, pycompat

from ..geoipresolver import GeoIPResolver

_logger = logging.getLogger(__name__)

# global resolver (GeoIP API is thread-safe, for multithreaded workers)
# This avoids blowing up open files limit
odoo._geoip_resolver = None

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
    if isinstance(value, models.BaseModel):
        if not value.id:
            raise ValueError("Cannot slug non-existent record %s" % value)
        # [(id, name)] = value.name_get()
        identifier, name = value.id, getattr(value, 'seo_name', False) or value.display_name
    else:
        # assume name_search result tuple
        identifier, name = value
    slugname = slugify(name or '').strip().strip('-')
    if not slugname:
        return str(identifier)
    return "%s-%d" % (slugname, identifier)


# NOTE: as the pattern is used as it for the ModelConverter (ir_http.py), do not use any flags
_UNSLUG_RE = re.compile(r'(?:(\w{1,2}|\w[A-Za-z0-9-_]+?\w)-)?(-?\d+)(?=$|/)')


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
        self.regex = _UNSLUG_RE.pattern

    def to_url(self, value):
        return slug(value)

    def to_python(self, value):
        matching = re.match(self.regex, value)
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
        user_context = request.session.get_context() if request.session.uid else {}
        lang = user_context.get('lang')
        translation_hash = request.env['ir.translation'].get_web_translations_hash(modules, lang)

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

    bots = "bot|crawl|slurp|spider|curl|wget|facebookexternalhit".split("|")

    @classmethod
    def is_a_bot(cls):
        # We don't use regexp and ustr voluntarily
        # timeit has been done to check the optimum method
        user_agent = request.httprequest.environ.get('HTTP_USER_AGENT', '').lower()
        try:
            return any(bot in user_agent for bot in cls.bots)
        except UnicodeDecodeError:
            return any(bot in user_agent.encode('ascii', 'ignore') for bot in cls.bots)

    @classmethod
    def _get_frontend_langs(cls):
        return [code for code, _ in request.env['res.lang'].get_installed()]

    @classmethod
    def get_nearest_lang(cls, lang_code):
        """ Try to find a similar lang. Eg: fr_BE and fr_FR
            :param lang_code: the lang `code` (en_US)
        """
        if not lang_code:
            return False
        short_match = False
        short = lang_code.partition('_')[0]
        for code in cls._get_frontend_langs():
            if code == lang_code:
                return code
            if not short_match and code.startswith(short):
                short_match = code
        return short_match

    @classmethod
    def _geoip_setup_resolver(cls):
        # Lazy init of GeoIP resolver
        if odoo._geoip_resolver is not None:
            return
        geofile = config.get('geoip_database')
        try:
            odoo._geoip_resolver = GeoIPResolver.open(geofile) or False
        except Exception as e:
            _logger.warning('Cannot load GeoIP: %s', ustr(e))

    @classmethod
    def _geoip_resolve(cls):
        if 'geoip' not in request.session:
            record = {}
            if odoo._geoip_resolver and request.httprequest.remote_addr:
                record = odoo._geoip_resolver.resolve(request.httprequest.remote_addr) or {}
            request.session['geoip'] = record

    @classmethod
    def _add_dispatch_parameters(cls, func):
        Lang = request.env['res.lang']
        # only called for is_frontend request
        if request.routing_iteration == 1:
            context = dict(request.context)
            path = request.httprequest.path.split('/')
            is_a_bot = cls.is_a_bot()

            lang_codes = [code for code, *_ in Lang.get_available()]
            nearest_lang = not func and cls.get_nearest_lang(Lang._lang_get_code(path[1]))
            cook_lang = request.httprequest.cookies.get('frontend_lang')
            cook_lang = cook_lang in lang_codes and cook_lang

            if nearest_lang:
                lang = Lang._lang_get(nearest_lang)
            else:
                nearest_ctx_lg = not is_a_bot and cls.get_nearest_lang(request.env.context.get('lang'))
                nearest_ctx_lg = nearest_ctx_lg in lang_codes and nearest_ctx_lg
                preferred_lang = Lang._lang_get(cook_lang or nearest_ctx_lg)
                lang = preferred_lang or cls._get_default_lang()

            request.lang = lang
            context['lang'] = lang._get_cached('code')

            # bind modified context
            request.context = context

    @classmethod
    def _dispatch(cls):
        """ Before executing the endpoint method, add website params on request, such as
                - current website (record)
                - multilang support (set on cookies)
                - geoip dict data are added in the session
            Then follow the parent dispatching.
            Reminder :  Do not use `request.env` before authentication phase, otherwise the env
                        set on request will be created with uid=None (and it is a lazy property)
        """
        request.routing_iteration = getattr(request, 'routing_iteration', 0) + 1

        func = None
        routing_error = None

        # handle // in url
        if request.httprequest.method == 'GET' and '//' in request.httprequest.path:
            new_url = request.httprequest.path.replace('//', '/') + '?' + request.httprequest.query_string.decode('utf-8')
            return request.redirect(new_url, code=301)

        # locate the controller method
        try:
            rule, arguments = cls._match(request.httprequest.path)
            func = rule.endpoint
            request.is_frontend = func.routing.get('website', False)
        except werkzeug.exceptions.NotFound as e:
            # either we have a language prefixed route, either a real 404
            # in all cases, website processes them exept if second element is static
            # Checking static will avoid to generate an expensive 404 web page since
            # most of the time the browser is loading and inexisting assets or image. A standard 404 is enough.
            # Earlier check would be difficult since we don't want to break data modules
            path_components = request.httprequest.path.split('/')
            request.is_frontend = len(path_components) < 3 or path_components[2] != 'static' or '.' not in path_components[-1]
            routing_error = e

        request.is_frontend_multilang = not func or (func and request.is_frontend and func.routing.get('multilang', func.routing['type'] == 'http'))

        # check authentication level
        try:
            if func:
                cls._authenticate(func)
            elif request.uid is None and request.is_frontend:
                cls._auth_method_public()
        except Exception as e:
            return cls._handle_exception(e)

        cls._geoip_setup_resolver()
        cls._geoip_resolve()

        # For website routes (only), add website params on `request`
        if request.is_frontend:
            cls._add_dispatch_parameters(func)

            path = request.httprequest.path.split('/')
            default_lg_id = cls._get_default_lang()
            if request.routing_iteration == 1:
                is_a_bot = cls.is_a_bot()
                nearest_lang = not func and cls.get_nearest_lang(request.env['res.lang']._lang_get_code(path[1]))
                url_lg = nearest_lang and path[1]

                # The default lang should never be in the URL, and a wrong lang
                # should never be in the URL.
                wrong_url_lg = url_lg and (url_lg != request.lang.url_code or url_lg == default_lg_id.url_code)
                # The lang is missing from the URL if multi lang is enabled for
                # the route and the current lang is not the default lang.
                # POST requests are excluded from this condition.
                missing_url_lg = not url_lg and request.is_frontend_multilang and request.lang != default_lg_id and request.httprequest.method != 'POST'
                # Bots should never be redirected when the lang is missing
                # because it is the only way for them to index the default lang.
                if wrong_url_lg or (missing_url_lg and not is_a_bot):
                    if url_lg:
                        path.pop(1)
                    if request.lang != default_lg_id:
                        path.insert(1, request.lang.url_code)
                    path = '/'.join(path) or '/'
                    routing_error = None
                    redirect = request.redirect(path + '?' + request.httprequest.query_string.decode('utf-8'))
                    redirect.set_cookie('frontend_lang', request.lang.code)
                    return redirect
                elif url_lg:
                    request.uid = None
                    if request.httprequest.path == '/%s/' % url_lg:
                        # special case for homepage controller, mimick `_postprocess_args()` redirect
                        path = request.httprequest.path[:-1]
                        if request.httprequest.query_string:
                            path += '?' + request.httprequest.query_string.decode('utf-8')
                        return request.redirect(path, code=301)
                    path.pop(1)
                    routing_error = None
                    return cls.reroute('/'.join(path) or '/')
                elif missing_url_lg and is_a_bot:
                    # Ensure that if the URL without lang is not redirected, the
                    # current lang is indeed the default lang, because it is the
                    # lang that bots should index in that case.
                    request.lang = default_lg_id
                    request.context = dict(request.context, lang=default_lg_id.code)

            if request.lang == default_lg_id:
                context = dict(request.context)
                context['edit_translations'] = False
                request.context = context

        if routing_error:
            return cls._handle_exception(routing_error)

        # removed cache for auth public
        result = super(IrHttp, cls)._dispatch()

        cook_lang = request.httprequest.cookies.get('frontend_lang')
        if request.is_frontend and cook_lang != request.lang._get_cached('code') and hasattr(result, 'set_cookie'):
            result.set_cookie('frontend_lang', request.lang._get_cached('code'))

        return result

    @classmethod
    def _redirect(cls, location, code=303):
        if request and request.db and getattr(request, 'is_frontend', False):
            location = url_for(location)
        return super()._redirect(location, code)

    @classmethod
    def reroute(cls, path):
        if isinstance(path, str):
            path = path.encode("utf-8")
        path = path.decode("latin1", "replace")

        if not hasattr(request, 'rerouting'):
            request.rerouting = [request.httprequest.path]
        if path in request.rerouting:
            raise Exception("Rerouting loop is forbidden")
        request.rerouting.append(path)
        if len(request.rerouting) > cls.rerouting_limit:
            raise Exception("Rerouting limit exceeded")
        environ = dict(request.httprequest._HTTPRequest__environ, PATH_INFO=path)
        request.httprequest = HTTPRequest(environ)
        return cls._dispatch()

    @classmethod
    def _postprocess_args(cls, arguments, rule):
        super(IrHttp, cls)._postprocess_args(arguments, rule)

        try:
            _, path = rule.build(arguments)
            assert path is not None
        except odoo.exceptions.MissingError:
            return cls._handle_exception(werkzeug.exceptions.NotFound())
        except Exception as e:
            return cls._handle_exception(e)

        if getattr(request, 'is_frontend_multilang', False) and request.httprequest.method in ('GET', 'HEAD'):
            generated_path = werkzeug.urls.url_unquote_plus(path)
            current_path = werkzeug.urls.url_unquote_plus(request.httprequest.path)
            if generated_path != current_path:
                if request.lang != cls._get_default_lang():
                    path = '/' + request.lang.url_code + path
                if request.httprequest.query_string:
                    path += '?' + request.httprequest.query_string.decode('utf-8')
                return request.redirect(path, code=301)

    @classmethod
    def _get_exception_code_values(cls, exception):
        """ Return a tuple with the error code following by the values matching the exception"""
        code = 500  # default code
        values = dict(
            exception=exception,
            traceback=traceback.format_exc(),
        )
        if isinstance(exception, exceptions.UserError):
            values['error_message'] = exception.args[0]
            code = 400
            if isinstance(exception, exceptions.AccessError):
                code = 403

        elif isinstance(exception, QWebException):
            values.update(qweb_exception=exception)

            if isinstance(exception.error, exceptions.AccessError):
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
    def _handle_exception(cls, exception):
        is_frontend_request = bool(getattr(request, 'is_frontend', False))
        if not is_frontend_request:
            # Don't touch non frontend requests exception handling
            return super(IrHttp, cls)._handle_exception(exception)
        try:
            response = super(IrHttp, cls)._handle_exception(exception)

            if isinstance(response, Exception):
                exception = response
            else:
                # if parent excplicitely returns a plain response, then we don't touch it
                return response
        except Exception as e:
            if 'werkzeug' in config['dev_mode']:
                raise e
            exception = e

        code, values = cls._get_exception_code_values(exception)

        if code is None:
            # Hand-crafted HTTPException likely coming from abort(),
            # usually for a redirect response -> return it directly
            return exception

        if not request.uid:
            cls._auth_method_public()

        # We rollback the current transaction before initializing a new
        # cursor to avoid potential deadlocks.

        # If the current (failed) transaction was holding a lock, the new
        # cursor might have to wait for this lock to be released further
        # down the line. However, this will only happen after the
        # request is done (and in fact it won't happen). As a result, the
        # current thread/worker is frozen until its timeout is reached.

        # So rolling back the transaction will release any potential lock
        # and, since we are in a case where an exception was raised, the
        # transaction shouldn't be committed in the first place.
        request.env.cr.rollback()

        with registry(request.env.cr.dbname).cursor() as cr:
            env = api.Environment(cr, request.uid, request.env.context)
            if code == 500:
                _logger.error("500 Internal Server Error:\n\n%s", values['traceback'])
                values = cls._get_values_500_error(env, values, exception)
            elif code == 403:
                _logger.warning("403 Forbidden:\n\n%s", values['traceback'])
            elif code == 400:
                _logger.warning("400 Bad Request:\n\n%s", values['traceback'])
            try:
                code, html = cls._get_error_html(env, code, values)
            except Exception:
                code, html = 418, env['ir.ui.view']._render_template('http_routing.http_error', values)

        return werkzeug.wrappers.Response(html, status=code, content_type='text/html;charset=utf-8')

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
            # get path from http://{path}?{current query string}
            new_url = e.new_url.split('?')[0][7:]
            _, endpoint = self.url_rewrite(new_url, query_args)
            endpoint = endpoint and [endpoint]
        except werkzeug.exceptions.NotFound:
            new_url = path
        return new_url or path, endpoint and endpoint[0]

```

## File: models\ir_ui_view.py

```python
# -*- coding: ascii -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.addons.http_routing.models.ir_http import slug, unslug_url


class IrUiView(models.Model):
    _inherit = ["ir.ui.view"]

    @api.model
    def _prepare_qcontext(self):
        qcontext = super(IrUiView, self)._prepare_qcontext()
        qcontext['slug'] = slug
        qcontext['unslug_url'] = unslug_url
        return qcontext

```

## File: models\res_lang.py

```python

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import ir_ui_view

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
                    <a data-toggle="collapse" href="#error_main">Error</a>
                </h4>
                <div id="error_main" class="collapse show">
                    <div class="card-body">
                        <t t-call="http_routing.error_message"/>
                    </div>
                </div>
            </div>
            <div class="card" t-if="qweb_exception">
                <h4 class="card-header">
                    <a data-toggle="collapse" href="#error_qweb">QWeb</a>
                </h4>
                <div id="error_qweb" class="collapse show">
                    <div class="card-body">
                        <p t-if="exception.message">
                            <strong>Error message:</strong>
                            <pre t-esc="exception.message"/>
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
                    <a data-toggle="collapse" href="#error_traceback">Traceback</a>
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
                                <div class="col-lg-8 text-lg-left text-center my-auto">
                                    <h1 class="sr-only">Error 404</h1>
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

                <link rel="stylesheet" href="/web/static/lib/bootstrap/css/bootstrap.css"/>
                <script src="/web/static/lib/jquery/jquery.js" type="text/javascript"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/util.js"/>
                <script type="text/javascript" src="/web/static/lib/bootstrap/js/collapse.js"/>
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
                                    <ul class="navbar-nav ml-auto" id="top_menu">
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

