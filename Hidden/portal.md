# Odoo Module: portal

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Updating mako environement in order to be able to use slug
try:
    from odoo.tools.rendering_tools import template_env_globals
    from odoo.addons.http_routing.models.ir_http import slug

    template_env_globals.update({
        'slug': slug
    })
except ImportError:
    pass

from . import controllers
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Customer Portal',
    'summary': 'Customer Portal',
    'sequence': 9000,
    'category': 'Hidden',
    'description': """
This module adds required base code for a fully integrated customer portal.
It contains the base controller class and base templates. Business addons
will add their specific templates and controllers to extend the customer
portal.

This module contains most code coming from odoo v10 website_portal. Purpose
of this module is to allow the display of a customer portal without having
a dependency towards website editing and customization capabilities.""",
    'depends': ['web', 'web_editor', 'http_routing', 'mail', 'auth_signup'],
    'data': [
        'security/ir.model.access.csv',
        'data/mail_template_data.xml',
        'data/mail_templates.xml',
        'views/mail_templates_public.xml',
        'views/portal_templates.xml',
        'views/res_config_settings_views.xml',
        'wizard/portal_share_views.xml',
        'wizard/portal_wizard_views.xml',
    ],
    'assets': {
        'web._assets_primary_variables': [
            'portal/static/src/scss/primary_variables.scss',
        ],
        'web._assets_frontend_helpers': [
            ('prepend', 'portal/static/src/scss/bootstrap_overridden.scss'),
        ],
        'web.assets_backend': [
            'portal/static/src/views/**/*',
        ],
        'web.assets_frontend': [
            'portal/static/src/scss/portal.scss',
            'portal/static/src/js/portal.js',
            'portal/static/src/js/portal_chatter.js',
            'portal/static/src/xml/portal_chatter.xml',
            'portal/static/src/js/portal_composer.js',
            'portal/static/src/js/portal_security.js',
            'portal/static/src/js/portal_sidebar.js',
            'portal/static/src/xml/portal_security.xml',
            'portal/static/src/js/components/**/*',
            'portal/static/src/signature_form/**/*',
        ],
        'web.assets_tests': [
            'portal/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug import urls
from werkzeug.exceptions import NotFound, Forbidden

from odoo import http
from odoo.http import request
from odoo.osv import expression
from odoo.tools import consteq, plaintext2html
from odoo.addons.mail.controllers import mail
from odoo.exceptions import AccessError


def _check_special_access(res_model, res_id, token='', _hash='', pid=False):
    record = request.env[res_model].browse(res_id).sudo()
    if _hash and pid:  # Signed Token Case: hash implies token is signed by partner pid
        can_access = consteq(_hash, record._sign_token(pid))
        if not can_access:
            parent_sign_token = record._portal_get_parent_hash_token(pid)
            can_access = parent_sign_token and consteq(_hash, parent_sign_token)
        return can_access
    elif token:  # Token Case: token is the global one of the document
        token_field = request.env[res_model]._mail_post_token_field
        return (token and record and consteq(record[token_field], token))
    else:
        raise Forbidden()


def _message_post_helper(res_model, res_id, message, token='', _hash=False, pid=False, nosubscribe=True, **kw):
    """ Generic chatter function, allowing to write on *any* object that inherits mail.thread. We
        distinguish 2 cases:
            1/ If a token is specified, all logged in users will be able to write a message regardless
            of access rights; if the user is the public user, the message will be posted under the name
            of the partner_id of the object (or the public user if there is no partner_id on the object).

            2/ If a signed token is specified (`hash`) and also a partner_id (`pid`), all post message will
            be done under the name of the partner_id (as it is signed). This should be used to avoid leaking
            token to all users.

        Required parameters
        :param string res_model: model name of the object
        :param int res_id: id of the object
        :param string message: content of the message

        Optional keywords arguments:
        :param string token: access token if the object's model uses some kind of public access
                             using tokens (usually a uuid4) to bypass access rules
        :param string hash: signed token by a partner if model uses some token field to bypass access right
                            post messages.
        :param string pid: identifier of the res.partner used to sign the hash
        :param bool nosubscribe: set False if you want the partner to be set as follower of the object when posting (default to True)

        The rest of the kwargs are passed on to message_post()
    """
    record = request.env[res_model].browse(res_id)

    # check if user can post with special token/signed token. The "else" will try to post message with the
    # current user access rights (_mail_post_access use case).
    if token or (_hash and pid):
        pid = int(pid) if pid else False
        if _check_special_access(res_model, res_id, token=token, _hash=_hash, pid=pid):
            record = record.sudo()
        else:
            raise Forbidden()
    else:  # early check on access to avoid useless computation
        record.check_access_rights('read')
        record.check_access_rule('read')

    # deduce author of message
    author_id = request.env.user.partner_id.id if request.env.user.partner_id else False

    # Signed Token Case: author_id is forced
    if _hash and pid:
        author_id = pid
    # Token Case: author is document customer (if not logged) or itself even if user has not the access
    elif token:
        if request.env.user._is_public():
            # TODO : After adding the pid and sign_token in access_url when send invoice by email, remove this line
            # TODO : Author must be Public User (to rename to 'Anonymous')
            author_id = record.partner_id.id if hasattr(record, 'partner_id') and record.partner_id.id else author_id
        else:
            if not author_id:
                raise NotFound()

    email_from = None
    if author_id and 'email_from' not in kw:
        partner = request.env['res.partner'].sudo().browse(author_id)
        email_from = partner.email_formatted if partner.email else None

    message_post_args = dict(
        body=message,
        message_type=kw.pop('message_type', "comment"),
        subtype_xmlid=kw.pop('subtype_xmlid', "mail.mt_comment"),
        author_id=author_id,
        **kw
    )

    # This is necessary as mail.message checks the presence
    # of the key to compute its default email from
    if email_from:
        message_post_args['email_from'] = email_from

    return record.with_context(mail_create_nosubscribe=nosubscribe).message_post(**message_post_args)


class PortalChatter(http.Controller):

    def _portal_post_filter_params(self):
        return ['token', 'pid']

    def _portal_post_check_attachments(self, attachment_ids, attachment_tokens):
        request.env['ir.attachment'].browse(attachment_ids)._check_attachments_access(attachment_tokens)

    def _portal_post_has_content(self, res_model, res_id, message, attachment_ids=None, **kw):
        """ Tells if we can effectively post on the model based on content. """
        return bool(message) or bool(attachment_ids)

    @http.route('/mail/avatar/mail.message/<int:res_id>/author_avatar/<int:width>x<int:height>', type='http', auth='public')
    def portal_avatar(self, res_id=None, height=50, width=50, access_token=None, _hash=None, pid=None):
        """ Get the avatar image in the chatter of the portal """
        if access_token or (_hash and pid):
            message = request.env['mail.message'].browse(int(res_id)).exists().filtered(
                lambda msg: _check_special_access(msg.model, msg.res_id, access_token, _hash, pid and int(pid))
            ).sudo()
        else:
            message = request.env.ref('web.image_placeholder').sudo()
        # in case there is no message, it creates a stream with the placeholder image
        stream = request.env['ir.binary']._get_image_stream_from(
            message, field_name='author_avatar', width=int(width), height=int(height),
        )
        return stream.get_response()

    @http.route(['/mail/chatter_post'], type='json', methods=['POST'], auth='public', website=True)
    def portal_chatter_post(self, res_model, res_id, message, attachment_ids=None, attachment_tokens=None, **kw):
        """Create a new `mail.message` with the given `message` and/or `attachment_ids` and return new message values.

        The message will be associated to the record `res_id` of the model
        `res_model`. The user must have access rights on this target document or
        must provide valid identifiers through `kw`. See `_message_post_helper`.
        """
        if not self._portal_post_has_content(res_model, res_id, message,
                                             attachment_ids=attachment_ids, attachment_tokens=attachment_tokens,
                                             **kw):
            return

        res_id = int(res_id)

        self._portal_post_check_attachments(attachment_ids or [], attachment_tokens or [])

        result = {'default_message': message}
        # message is received in plaintext and saved in html
        if message:
            message = plaintext2html(message)
        post_values = {
            'res_model': res_model,
            'res_id': res_id,
            'message': message,
            'send_after_commit': False,
            'attachment_ids': False,  # will be added afterward
        }
        post_values.update((fname, kw.get(fname)) for fname in self._portal_post_filter_params())
        post_values['_hash'] = kw.get('hash')
        message = _message_post_helper(**post_values)
        result.update({'default_message_id': message.id})

        if attachment_ids:
            # _message_post_helper already checks for pid/hash/token -> use message
            # environment to keep the sudo mode when activated
            record = message.env[res_model].browse(res_id)
            attachments = record._process_attachments_for_post(
                [], attachment_ids,
                {'res_id': res_id, 'model': res_model}
            )
            # sudo write the attachment to bypass the read access verification in
            # mail message
            if attachments.get('attachment_ids'):
                message.sudo().write(attachments)

            result.update({'default_attachment_ids': message.attachment_ids.sudo().read(['id', 'name', 'mimetype', 'file_size', 'access_token'])})
        return result

    @http.route('/mail/chatter_init', type='json', auth='public', website=True)
    def portal_chatter_init(self, res_model, res_id, domain=False, limit=False, **kwargs):
        is_user_public = request.env.user.has_group('base.group_public')
        message_data = self.portal_message_fetch(res_model, res_id, domain=domain, limit=limit, **kwargs)
        display_composer = False
        if kwargs.get('allow_composer'):
            display_composer = kwargs.get('token') or not is_user_public
        return {
            'messages': message_data['messages'],
            'options': {
                'message_count': message_data['message_count'],
                'is_user_public': is_user_public,
                'is_user_employee': request.env.user._is_internal(),
                'is_user_publisher': request.env.user.has_group('website.group_website_restricted_editor'),
                'display_composer': display_composer,
                'partner_id': request.env.user.partner_id.id
            }
        }

    @http.route('/mail/chatter_fetch', type='json', auth='public', website=True)
    def portal_message_fetch(self, res_model, res_id, domain=False, limit=10, offset=0, **kw):
        # Only search into website_message_ids, so apply the same domain to perform only one search
        # extract domain from the 'website_message_ids' field
        model = request.env[res_model]
        field = model._fields['website_message_ids']
        field_domain = field.get_domain_list(model)
        domain = expression.AND([
            self._setup_portal_message_fetch_extra_domain(kw),
            field_domain,
            [('res_id', '=', res_id), '|', ('body', '!=', ''), ('attachment_ids', '!=', False)]
        ])

        # Check access
        Message = request.env['mail.message']
        if kw.get('token'):
            access_as_sudo = _check_special_access(res_model, res_id, token=kw.get('token'))
            if not access_as_sudo:  # if token is not correct, raise Forbidden
                raise Forbidden()
            # Non-employee see only messages with not internal subtype (aka, no internal logs)
            if not request.env['res.users'].has_group('base.group_user'):
                domain = expression.AND([Message._get_search_domain_share(), domain])
            Message = request.env['mail.message'].sudo()
        return {
            'messages': Message.search(domain, limit=limit, offset=offset).portal_message_format(options=kw),
            'message_count': Message.search_count(domain)
        }

    def _setup_portal_message_fetch_extra_domain(self, data):
        return []

    @http.route(['/mail/update_is_internal'], type='json', auth="user", website=True)
    def portal_message_update_is_internal(self, message_id, is_internal):
        message = request.env['mail.message'].browse(int(message_id))
        message.write({'is_internal': is_internal})
        return message.is_internal


class MailController(mail.MailController):

    @classmethod
    def _redirect_to_record(cls, model, res_id, access_token=None, **kwargs):
        """ If the current user doesn't have access to the document, but provided
        a valid access token, redirect them to the front-end view.
        If the partner_id and hash parameters are given, add those parameters to the redirect url
        to authentify the recipient in the chatter, if any.

        :param model: the model name of the record that will be visualized
        :param res_id: the id of the record
        :param access_token: token that gives access to the record
            bypassing the rights and rules restriction of the user.
        :param kwargs: Typically, it can receive a partner_id and a hash (sign_token).
            If so, those two parameters are used to authentify the recipient in the chatter, if any.
        :return:
        """
        # no model / res_id, meaning no possible record -> direct skip to super
        if not model or not res_id or model not in request.env:
            return super(MailController, cls)._redirect_to_record(model, res_id, access_token=access_token, **kwargs)

        if isinstance(request.env[model], request.env.registry['portal.mixin']):
            uid = request.session.uid or request.env.ref('base.public_user').id
            record_sudo = request.env[model].sudo().browse(res_id).exists()
            try:
                record_sudo.with_user(uid).check_access_rights('read')
                record_sudo.with_user(uid).check_access_rule('read')
            except AccessError:
                if record_sudo.access_token and access_token and consteq(record_sudo.access_token, access_token):
                    record_action = record_sudo._get_access_action(force_website=True)
                    if record_action['type'] == 'ir.actions.act_url':
                        pid = kwargs.get('pid')
                        hash = kwargs.get('hash')
                        url = record_action['url']
                        if pid and hash:
                            url = urls.url_parse(url)
                            url_params = url.decode_query()
                            url_params.update([("pid", pid), ("hash", hash)])
                            url = url.replace(query=urls.url_encode(url_params)).to_url()
                        return request.redirect(url)
        return super(MailController, cls)._redirect_to_record(model, res_id, access_token=access_token, **kwargs)

    # Add website=True to support the portal layout
    @http.route('/mail/unfollow', type='http', website=True)
    def mail_action_unfollow(self, model, res_id, pid, token, **kwargs):
        return super().mail_action_unfollow(model, res_id, pid, token, **kwargs)

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import math
import re

from werkzeug import urls

from odoo import http, tools, _, SUPERUSER_ID
from odoo.exceptions import AccessDenied, AccessError, MissingError, UserError, ValidationError
from odoo.http import content_disposition, Controller, request, route
from odoo.tools import consteq

# --------------------------------------------------
# Misc tools
# --------------------------------------------------

def pager(url, total, page=1, step=30, scope=5, url_args=None):
    """ Generate a dict with required value to render `website.pager` template.

    This method computes url, page range to display, ... in the pager.

    :param str url : base url of the page link
    :param int total : number total of item to be splitted into pages
    :param int page : current page
    :param int step : item per page
    :param int scope : number of page to display on pager
    :param dict url_args : additionnal parameters to add as query params to page url
    :returns dict
    """
    # Compute Pager
    page_count = int(math.ceil(float(total) / step))

    page = max(1, min(int(page if str(page).isdigit() else 1), page_count))
    scope -= 1

    pmin = max(page - int(math.floor(scope/2)), 1)
    pmax = min(pmin + scope, page_count)

    if pmax - pmin < scope:
        pmin = pmax - scope if pmax - scope > 0 else 1

    def get_url(page):
        _url = "%s/page/%s" % (url, page) if page > 1 else url
        if url_args:
            _url = "%s?%s" % (_url, urls.url_encode(url_args))
        return _url

    return {
        "page_count": page_count,
        "offset": (page - 1) * step,
        "page": {
            'url': get_url(page),
            'num': page
        },
        "page_first": {
            'url': get_url(1),
            'num': 1
        },
        "page_start": {
            'url': get_url(pmin),
            'num': pmin
        },
        "page_previous": {
            'url': get_url(max(pmin, page - 1)),
            'num': max(pmin, page - 1)
        },
        "page_next": {
            'url': get_url(min(pmax, page + 1)),
            'num': min(pmax, page + 1)
        },
        "page_end": {
            'url': get_url(pmax),
            'num': pmax
        },
        "page_last": {
            'url': get_url(page_count),
            'num': page_count
        },
        "pages": [
            {'url': get_url(page_num), 'num': page_num} for page_num in range(pmin, pmax+1)
        ]
    }


def get_records_pager(ids, current):
    if current.id in ids and (hasattr(current, 'website_url') or hasattr(current, 'access_url')):
        attr_name = 'access_url' if hasattr(current, 'access_url') else 'website_url'
        idx = ids.index(current.id)
        prev_record = idx != 0 and current.browse(ids[idx - 1])
        next_record = idx < len(ids) - 1 and current.browse(ids[idx + 1])

        if prev_record and prev_record[attr_name] and attr_name == "access_url":
            prev_url = '%s?access_token=%s' % (prev_record[attr_name], prev_record._portal_ensure_token())
        elif prev_record and prev_record[attr_name]:
            prev_url = prev_record[attr_name]
        else:
            prev_url = prev_record

        if next_record and next_record[attr_name] and attr_name == "access_url":
            next_url = '%s?access_token=%s' % (next_record[attr_name], next_record._portal_ensure_token())
        elif next_record and next_record[attr_name]:
            next_url = next_record[attr_name]
        else:
            next_url = next_record

        return {
            'prev_record': prev_url,
            'next_record': next_url,
        }
    return {}


def _build_url_w_params(url_string, query_params, remove_duplicates=True):
    """ Rebuild a string url based on url_string and correctly compute query parameters
    using those present in the url and those given by query_params. Having duplicates in
    the final url is optional. For example:

     * url_string = '/my?foo=bar&error=pay'
     * query_params = {'foo': 'bar2', 'alice': 'bob'}
     * if remove duplicates: result = '/my?foo=bar2&error=pay&alice=bob'
     * else: result = '/my?foo=bar&foo=bar2&error=pay&alice=bob'
    """
    url = urls.url_parse(url_string)
    url_params = url.decode_query()
    if remove_duplicates:  # convert to standard dict instead of werkzeug multidict to remove duplicates automatically
        url_params = url_params.to_dict()
    url_params.update(query_params)
    return url.replace(query=urls.url_encode(url_params)).to_url()


class CustomerPortal(Controller):

    MANDATORY_BILLING_FIELDS = ["name", "phone", "email", "street", "city", "country_id"]
    OPTIONAL_BILLING_FIELDS = ["zipcode", "state_id", "vat", "company_name"]

    _items_per_page = 80

    def _prepare_portal_layout_values(self):
        """Values for /my/* templates rendering.

        Does not include the record counts.
        """
        # get customer sales rep
        sales_user_sudo = request.env['res.users']
        partner_sudo = request.env.user.partner_id
        if partner_sudo.user_id and not partner_sudo.user_id._is_public():
            sales_user_sudo = partner_sudo.user_id
        else:
            fallback_sales_user = partner_sudo.commercial_partner_id.user_id
            if fallback_sales_user and not fallback_sales_user._is_public():
                sales_user_sudo = fallback_sales_user

        return {
            'sales_user': sales_user_sudo,
            'page_name': 'home',
        }

    def _prepare_home_portal_values(self, counters):
        """Values for /my & /my/home routes template rendering.

        Includes the record count for the displayed badges.
        where 'counters' is the list of the displayed badges
        and so the list to compute.
        """
        return {}

    @route(['/my/counters'], type='json', auth="user", website=True)
    def counters(self, counters, **kw):
        return self._prepare_home_portal_values(counters)

    @route(['/my', '/my/home'], type='http', auth="user", website=True)
    def home(self, **kw):
        values = self._prepare_portal_layout_values()
        return request.render("portal.portal_my_home", values)

    @route(['/my/account'], type='http', auth='user', website=True)
    def account(self, redirect=None, **post):
        values = self._prepare_portal_layout_values()
        partner = request.env.user.partner_id
        values.update({
            'error': {},
            'error_message': [],
        })

        if post and request.httprequest.method == 'POST':
            if not partner.can_edit_vat():
                post['country_id'] = str(partner.country_id.id)

            error, error_message = self.details_form_validate(post)
            values.update({'error': error, 'error_message': error_message})
            values.update(post)
            if not error:
                values = {key: post[key] for key in self._get_mandatory_fields()}
                values.update({key: post[key] for key in self._get_optional_fields() if key in post})
                for field in set(['country_id', 'state_id']) & set(values.keys()):
                    try:
                        values[field] = int(values[field])
                    except:
                        values[field] = False
                values.update({'zip': values.pop('zipcode', '')})
                self.on_account_update(values, partner)
                partner.sudo().write(values)
                if redirect:
                    return request.redirect(redirect)
                return request.redirect('/my/home')

        countries = request.env['res.country'].sudo().search([])
        states = request.env['res.country.state'].sudo().search([])

        values.update({
            'partner': partner,
            'countries': countries,
            'states': states,
            'has_check_vat': hasattr(request.env['res.partner'], 'check_vat'),
            'partner_can_edit_vat': partner.can_edit_vat(),
            'redirect': redirect,
            'page_name': 'my_details',
        })

        response = request.render("portal.portal_my_details", values)
        response.headers['X-Frame-Options'] = 'SAMEORIGIN'
        response.headers['Content-Security-Policy'] = "frame-ancestors 'self'"
        return response

    def on_account_update(self, values, partner):
        pass

    @route('/my/security', type='http', auth='user', website=True, methods=['GET', 'POST'])
    def security(self, **post):
        values = self._prepare_portal_layout_values()
        values['get_error'] = get_error
        values['allow_api_keys'] = bool(request.env['ir.config_parameter'].sudo().get_param('portal.allow_api_keys'))
        values['open_deactivate_modal'] = False

        if request.httprequest.method == 'POST':
            values.update(self._update_password(
                post['old'].strip(),
                post['new1'].strip(),
                post['new2'].strip()
            ))

        return request.render('portal.portal_my_security', values, headers={
            'X-Frame-Options': 'SAMEORIGIN',
            'Content-Security-Policy': "frame-ancestors 'self'"
        })

    def _update_password(self, old, new1, new2):
        for k, v in [('old', old), ('new1', new1), ('new2', new2)]:
            if not v:
                return {'errors': {'password': {k: _("You cannot leave any password empty.")}}}

        if new1 != new2:
            return {'errors': {'password': {'new2': _("The new password and its confirmation must be identical.")}}}

        try:
            request.env['res.users'].change_password(old, new1)
        except AccessDenied as e:
            msg = e.args[0]
            if msg == AccessDenied().args[0]:
                msg = _('The old password you provided is incorrect, your password was not changed.')
            return {'errors': {'password': {'old': msg}}}
        except UserError as e:
            return {'errors': {'password': str(e)}}

        # update session token so the user does not get logged out (cache cleared by passwd change)
        new_token = request.env.user._compute_session_token(request.session.sid)
        request.session.session_token = new_token

        return {'success': {'password': True}}

    @route('/my/deactivate_account', type='http', auth='user', website=True, methods=['POST'])
    def deactivate_account(self, validation, password, **post):
        values = self._prepare_portal_layout_values()
        values['get_error'] = get_error
        values['open_deactivate_modal'] = True

        if validation != request.env.user.login:
            values['errors'] = {'deactivate': 'validation'}
        else:
            try:
                request.env['res.users']._check_credentials(password, {'interactive': True})
                request.env.user.sudo()._deactivate_portal_user(**post)
                request.session.logout()
                return request.redirect('/web/login?message=%s' % urls.url_quote(_('Account deleted!')))
            except AccessDenied:
                values['errors'] = {'deactivate': 'password'}
            except UserError as e:
                values['errors'] = {'deactivate': {'other': str(e)}}

        return request.render('portal.portal_my_security', values, headers={
            'X-Frame-Options': 'SAMEORIGIN',
            'Content-Security-Policy': "frame-ancestors 'self'",
        })

    @http.route('/portal/attachment/add', type='http', auth='public', methods=['POST'], website=True)
    def attachment_add(self, name, file, res_model, res_id, access_token=None, **kwargs):
        """Process a file uploaded from the portal chatter and create the
        corresponding `ir.attachment`.

        The attachment will be created "pending" until the associated message
        is actually created, and it will be garbage collected otherwise.

        :param name: name of the file to save.
        :type name: string

        :param file: the file to save
        :type file: werkzeug.FileStorage

        :param res_model: name of the model of the original document.
            To check access rights only, it will not be saved here.
        :type res_model: string

        :param res_id: id of the original document.
            To check access rights only, it will not be saved here.
        :type res_id: int

        :param access_token: access_token of the original document.
            To check access rights only, it will not be saved here.
        :type access_token: string

        :return: attachment data {id, name, mimetype, file_size, access_token}
        :rtype: dict
        """
        try:
            self._document_check_access(res_model, int(res_id), access_token=access_token)
        except (AccessError, MissingError) as e:
            raise UserError(_("The document does not exist or you do not have the rights to access it."))

        IrAttachment = request.env['ir.attachment']

        # Avoid using sudo when not necessary: internal users can create attachments,
        # as opposed to public and portal users.
        if not request.env.user._is_internal():
            IrAttachment = IrAttachment.sudo()

        # At this point the related message does not exist yet, so we assign
        # those specific res_model and res_is. They will be correctly set
        # when the message is created: see `portal_chatter_post`,
        # or garbage collected otherwise: see  `_garbage_collect_attachments`.
        attachment = IrAttachment.create({
            'name': name,
            'datas': base64.b64encode(file.read()),
            'res_model': 'mail.compose.message',
            'res_id': 0,
            'access_token': IrAttachment._generate_access_token(),
        })
        return request.make_response(
            data=json.dumps(attachment.read(['id', 'name', 'mimetype', 'file_size', 'access_token'])[0]),
            headers=[('Content-Type', 'application/json')]
        )

    @http.route('/portal/attachment/remove', type='json', auth='public')
    def attachment_remove(self, attachment_id, access_token=None):
        """Remove the given `attachment_id`, only if it is in a "pending" state.

        The user must have access right on the attachment or provide a valid
        `access_token`.
        """
        try:
            attachment_sudo = self._document_check_access('ir.attachment', int(attachment_id), access_token=access_token)
        except (AccessError, MissingError) as e:
            raise UserError(_("The attachment does not exist or you do not have the rights to access it."))

        if attachment_sudo.res_model != 'mail.compose.message' or attachment_sudo.res_id != 0:
            raise UserError(_("The attachment %s cannot be removed because it is not in a pending state.", attachment_sudo.name))

        if attachment_sudo.env['mail.message'].search([('attachment_ids', 'in', attachment_sudo.ids)]):
            raise UserError(_("The attachment %s cannot be removed because it is linked to a message.", attachment_sudo.name))

        return attachment_sudo.unlink()

    def details_form_validate(self, data, partner_creation=False):
        error = dict()
        error_message = []

        request.update_context(portal_form_country_id=data['country_id'])
        partner = request.env.user.partner_id

        # Validation
        for field_name in self._get_mandatory_fields():
            if not data.get(field_name):
                if field_name == 'country_id' and not partner.can_edit_vat():
                    # you cannot edit the country ID in that case so we skip the error, see the XML form for details
                    continue
                error[field_name] = 'missing'
                if field_name == 'zipcode':
                    error['zip'] = 'missing'

        # email validation
        if data.get('email') and not tools.single_email_re.match(data.get('email')):
            error["email"] = 'error'
            error_message.append(_('Invalid Email! Please enter a valid email address.'))

        # vat validation
        if data.get("vat") and partner and partner.vat != data.get("vat"):
            # Check the VAT if it is the public user too.
            if partner_creation or partner.can_edit_vat():
                if hasattr(partner, "check_vat"):
                    if data.get("country_id"):
                        data["vat"] = request.env["res.partner"].fix_eu_vat_number(int(data.get("country_id")), data.get("vat"))
                    partner_dummy = partner.new({
                        'vat': data['vat'],
                        'country_id': (int(data['country_id'])
                                       if data.get('country_id') else False),
                    })
                    try:
                        partner_dummy.check_vat()
                    except ValidationError as e:
                        error["vat"] = 'error'
                        error_message.append(e.args[0])
            else:
                error_message.append(_('Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.'))

        # error message for empty required fields
        if [err for err in error.values() if err == 'missing']:
            error_message.append(_('Some required fields are empty.'))

        unknown = [k for k in data if k not in self._get_mandatory_fields() + self._get_optional_fields()]
        if unknown:
            error['common'] = 'Unknown field'
            error_message.append("Unknown field '%s'" % ','.join(unknown))

        return error, error_message

    def _get_mandatory_fields(self):
        """ This method is there so that we can override the mandatory fields """
        return list(self.MANDATORY_BILLING_FIELDS)

    def _get_optional_fields(self):
        """ This method is there so that we can override the optional fields """
        return list(self.OPTIONAL_BILLING_FIELDS)

    def _document_check_access(self, model_name, document_id, access_token=None):
        """Check if current user is allowed to access the specified record.

        :param str model_name: model of the requested record
        :param int document_id: id of the requested record
        :param str access_token: record token to check if user isn't allowed to read requested record
        :return: expected record, SUDOED, with SUPERUSER context
        :raise MissingError: record not found in database, might have been deleted
        :raise AccessError: current user isn't allowed to read requested document (and no valid token was given)
        """
        document = request.env[model_name].browse([document_id])
        document_sudo = document.with_user(SUPERUSER_ID).exists()
        if not document_sudo:
            raise MissingError(_("This document does not exist."))
        try:
            document.check_access_rights('read')
            document.check_access_rule('read')
        except AccessError:
            if not access_token or not document_sudo.access_token or not consteq(document_sudo.access_token, access_token):
                raise
        return document_sudo

    def _get_page_view_values(self, document, access_token, values, session_history, no_breadcrumbs, **kwargs):
        """Include necessary values for portal chatter & pager setup (see template portal.message_thread).

        :param document: record to display on portal
        :param str access_token: provided document access token
        :param dict values: base dict of values where chatter rendering values should be added
        :param str session_history: key used to store latest records browsed on the portal in the session
        :param bool no_breadcrumbs:
        :return: updated values
        :rtype: dict
        """
        values['object'] = document

        if access_token:
            # if no_breadcrumbs = False -> force breadcrumbs even if access_token to `invite` users to register if they click on it
            values['no_breadcrumbs'] = no_breadcrumbs
            values['access_token'] = access_token
            values['token'] = access_token  # for portal chatter

        # Those are used notably whenever the payment form is implied in the portal.
        if kwargs.get('error'):
            values['error'] = kwargs['error']
        if kwargs.get('warning'):
            values['warning'] = kwargs['warning']
        if kwargs.get('success'):
            values['success'] = kwargs['success']
        # Email token for posting messages in portal view with identified author
        if kwargs.get('pid'):
            values['pid'] = kwargs['pid']
        if kwargs.get('hash'):
            values['hash'] = kwargs['hash']

        history = request.session.get(session_history, [])
        values.update(get_records_pager(history, document))

        return values

    def _show_report(self, model, report_type, report_ref, download=False):
        if report_type not in ('html', 'pdf', 'text'):
            raise UserError(_("Invalid report type: %s", report_type))

        ReportAction = request.env['ir.actions.report'].sudo()

        if hasattr(model, 'company_id'):
            if len(model.company_id) > 1:
                raise UserError(_('Multi company reports are not supported.'))
            ReportAction = ReportAction.with_company(model.company_id)

        method_name = '_render_qweb_%s' % (report_type)
        report = getattr(ReportAction, method_name)(report_ref, list(model.ids), data={'report_type': report_type})[0]
        headers = self._get_http_headers(model, report_type, report, download)
        return request.make_response(report, headers=list(headers.items()))

    def _get_http_headers(self, model, report_type, report, download):
        headers = {
            'Content-Type': 'application/pdf' if report_type == 'pdf' else 'text/html',
            'Content-Length': len(report),
        }
        if report_type == 'pdf' and download:
            filename = "%s.pdf" % (re.sub(r'\W+', '-', model._get_report_base_filename()))
            headers['Content-Disposition'] = content_disposition(filename)
        return headers

def get_error(e, path=''):
    """ Recursively dereferences `path` (a period-separated sequence of dict
    keys) in `e` (an error dict or value), returns the final resolution IIF it's
    an str, otherwise returns None
    """
    for k in (path.split('.') if path else []):
        if not isinstance(e, dict):
            return None
        e = e.get(k)

    return e if isinstance(e, str) else None

```

## File: controllers\web.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.addons.web.controllers.home import Home as WebHome
from odoo.addons.web.controllers.utils import is_user_internal
from odoo.http import request


class Home(WebHome):

    @http.route()
    def index(self, *args, **kw):
        if request.session.uid and not is_user_internal(request.session.uid):
            return request.redirect_query('/my', query=request.params)
        return super().index(*args, **kw)

    def _login_redirect(self, uid, redirect=None):
        if not redirect and not is_user_internal(uid):
            redirect = '/my'
        return super()._login_redirect(uid, redirect=redirect)

    @http.route()
    def web_client(self, s_action=None, **kw):
        if request.session.uid and not is_user_internal(request.session.uid):
            return request.redirect_query('/my', query=request.params)
        return super().web_client(s_action, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import web
from . import portal
from . import mail

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <template id="portal_share_template">
        <div>
            <br/>
            <p>Dear <span t-esc="partner.name"/>,</p>
            <p><span t-out="user.name"/> has invited you to access the following <span t-out="model_description"/>:</p>
            <br/>
            <a t-attf-href="#{share_link}" style="background-color: #875A7B; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px; font-size: 12px;"><strong>Open </strong><strong t-esc="record.display_name"/></a><br/>
            <br/>
            <p t-if="note" style="white-space: pre-wrap;" t-esc="note"/>
        </div>
    </template>

</data></odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mail_template_data_portal_welcome" model="mail.template">
            <field name="name">Portal: User Invite</field>
            <field name="model_id" ref="portal.model_portal_wizard_user"/>
            <field name="subject">Your account at {{ object.user_id.company_id.name }}</field>
            <field name="email_to">{{ object.user_id.email_formatted }}</field>
            <field name="description">Invitation email to contacts to create a user account</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Account</span><br/>
                    <span style="font-size: 20px; font-weight: bold;" t-out="object.user_id.name or ''">Marc Demo</span>
                </td><td valign="middle" align="right" t-if="not object.user_id.company_id.uses_default_logo">
                    <img t-attf-src="/logo.png?company={{ object.user_id.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="object.user_id.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Dear <t t-out="object.user_id.name or ''">Marc Demo</t>,<br/> <br/>
                        Welcome to <t t-out="object.user_id.company_id.name">YourCompany</t>'s Portal!<br/><br/>
                        An account has been created for you with the following login: <t t-out="object.user_id.login">demo</t><br/><br/>
                        Click on the button below to pick a password and activate your account.
                        <div style="margin: 16px 0px 16px 0px; text-align: center;">
                            <a t-att-href="object.user_id.signup_url" style="display: inline-block; padding: 10px; text-decoration: none; font-size: 12px; background-color: #875A7B; color: #fff; border-radius: 5px;">
                                <strong>Activate Account</strong>
                            </a>
                        </div>
                        <t t-out="object.wizard_id.welcome_message or ''">Welcome to our company's portal.</t>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="object.user_id.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="object.user_id.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="object.user_id.company_id.email">
                        | <a t-attf-href="'mailto:%s' % {{ object.user_id.company_id.email }}" style="text-decoration:none; color: #454748;" t-out="object.user_id.company_id.email or ''">info@yourcompany.com</a>
                    </t>
                    <t t-if="object.user_id.company_id.website">
                        | <a t-attf-href="'%s' % {{ object.user_id.company_id.website }}" style="text-decoration:none; color: #454748;" t-out="object.user_id.company_id.website or ''">http://www.example.com</a>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=portalinvite" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super(IrHttp, cls)._get_translation_frontend_modules_name()
        return mods + ['portal']

    @classmethod
    def _get_frontend_langs(cls):
        # _get_frontend_langs() is used by @http_routing:IrHttp._match
        # where is_frontend is not yet set and when no backend endpoint
        # matched. We have to assume we are going to match a frontend
        # route, hence the default True. Elsewhere, request.is_frontend
        # is set.
        if request and getattr(request, 'is_frontend', True):
            return [lang[0] for lang in filter(lambda l: l[3], request.env['res.lang'].get_available())]
        return super()._get_frontend_langs()

```

## File: models\ir_qweb.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import is_html_empty, lazy


class IrQWeb(models.AbstractModel):
    _inherit = "ir.qweb"

    def _prepare_frontend_environment(self, values):
        """ Returns ir.qweb with context and update values with portal specific
            value (required to render portal layout template)
        """
        irQweb = super()._prepare_frontend_environment(values)
        values.update(
            is_html_empty=is_html_empty,
            languages=lazy(lambda: [lang for
                    lang in irQweb.env['res.lang'].get_available()
                    if lang[0] in irQweb.env['ir.http']._get_frontend_langs()])
        )
        for key in irQweb.env.context:
            if key not in values:
                values[key] = irQweb.env.context[key]

        return irQweb

```

## File: models\ir_ui_view.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class View(models.Model):
    _inherit = "ir.ui.view"

    customize_show = fields.Boolean("Show As Optional Inherit", default=False)

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request
from odoo.tools import format_datetime


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def portal_message_format(self, options=None):
        """ Simpler and portal-oriented version of 'message_format'. Purpose
        is to prepare, organize and format values required by frontend widget
        (frontend Chatter).

        This public API asks for read access on messages before doing the
        actual computation in the private implementation.

        :param dict options: options, used notably for inheritance and adding
          specific fields or properties to compute;

        :return list: list of dict, one per message in self. Each dict contains
          values for either fields, either properties derived from fields.
        """
        self.check_access_rule('read')
        return self._portal_message_format(
            self._portal_get_default_format_properties_names(options=options),
            options=options,
        )

    def _portal_get_default_format_properties_names(self, options=None):
        """ Fields and values to compute for portal format.

        :param dict options: options, used notably for inheritance and adding
          specific fields or properties to compute;

        :return set: fields or properties derived from fields
        """
        return {
            'attachment_ids',
            'author_avatar_url',
            'author_id',
            'body',
            'date',
            'id',
            'is_internal',
            'is_message_subtype_note',
            'published_date_str',
            'subtype_id',
        }

    def _portal_message_format(self, properties_names, options=None):
        """ Format messages for portal frontend. This private implementation
        does not check for access that should be checked beforehand.

        Notes:
          * when asking for attachments: ensure an access token is present then
            access them (using sudo);

        :param set properties_names: fields or properties derived from fields
          for which we are going to compute values;

        :return list: list of dict, one per message in self. Each dict contains
          values for either fields, either properties derived from fields.
        """
        message_to_attachments = {}
        if 'attachment_ids' in properties_names:
            properties_names.remove('attachment_ids')
            attachments_sudo = self.sudo().attachment_ids
            attachments_sudo.generate_access_token()
            related_attachments = {
                att_read_values['id']: att_read_values
                for att_read_values in attachments_sudo.read(
                    ["access_token", "checksum", "id", "mimetype", "name", "res_id", "res_model"]
                )
            }
            message_to_attachments = {
                message.id: [
                    self._portal_message_format_attachments(related_attachments[att_id])
                    for att_id in message.attachment_ids.ids
                ]
                for message in self.sudo()
            }

        fnames = {
            property_name for property_name in properties_names
            if property_name in self._fields
        }
        vals_list = self._read_format(fnames)

        note_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note')
        for message, values in zip(self, vals_list):
            if message_to_attachments:
                values['attachment_ids'] = message_to_attachments.get(message.id, {})
            if 'author_avatar_url' in properties_names:
                if options and 'token' in options:
                    values['author_avatar_url'] = f'/mail/avatar/mail.message/{message.id}/author_avatar/50x50?access_token={options["token"]}'
                elif options and options.keys() >= {"hash", "pid"}:
                    values['author_avatar_url'] = f'/mail/avatar/mail.message/{message.id}/author_avatar/50x50?_hash={options["hash"]}&pid={options["pid"]}'
                else:
                    values['author_avatar_url'] = f'/web/image/mail.message/{message.id}/author_avatar/50x50'
            if 'is_message_subtype_note' in properties_names:
                values['is_message_subtype_note'] = (values.get('subtype_id') or [False, ''])[0] == note_id
            if 'published_date_str' in properties_names:
                values['published_date_str'] = format_datetime(self.env, values['date']) if values.get('date') else ''
        return vals_list

    def _portal_message_format_attachments(self, attachment_values):
        """ From 'attachment_values' get an updated version formatted for
        frontend display.

        :param dict attachment_values: values coming from reading attachments
          in database;

        :return dict: updated attachment_values
        """
        safari = request and request.httprequest.user_agent and request.httprequest.user_agent.browser == 'safari'
        attachment_values['filename'] = attachment_values['name']
        attachment_values['mimetype'] = (
            'application/octet-stream' if safari and
            'video' in (attachment_values["mimetype"] or "")
            else attachment_values["mimetype"])
        return attachment_values

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac

from odoo import fields, models, _


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    _mail_post_token_field = 'access_token' # token field for external posts, to be overridden

    website_message_ids = fields.One2many('mail.message', 'res_id', string='Website Messages',
        domain=lambda self: [('model', '=', self._name), ('message_type', 'in', ('comment', 'email', 'email_outgoing'))],
        auto_join=True,
        help="Website communication history")

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        portal_enabled = isinstance(self, self.env.registry['portal.mixin'])
        if not portal_enabled:
            return groups

        customer = self._mail_get_partners(introspect_fields=False)[self.id]
        if customer:
            access_token = self._portal_ensure_token()
            local_msg_vals = dict(msg_vals or {})
            local_msg_vals['access_token'] = access_token
            local_msg_vals['pid'] = customer.id
            local_msg_vals['hash'] = self._sign_token(customer.id)
            local_msg_vals.update(customer.signup_get_auth_param()[customer.id])
            access_link = self._notify_get_action_link('view', **local_msg_vals)

            new_group = [
                ('portal_customer', lambda pdata: pdata['id'] == customer.id, {
                    'active': True,
                    'button_access': {
                        'url': access_link,
                    },
                    'has_button_access': True,
                })
            ]
        else:
            new_group = []

        # enable portal users that should have access through portal (if not access rights
        # will do their duty)
        portal_group = next(group for group in groups if group[0] == 'portal')
        portal_group[2]['active'] = True
        portal_group[2]['has_button_access'] = True

        return new_group + groups

    def _sign_token(self, pid):
        """Generate a secure hash for this record with the email of the recipient with whom the record have been shared.

        This is used to determine who is opening the link
        to be able for the recipient to post messages on the document's portal view.

        :param str email:
            Email of the recipient that opened the link.
        """
        self.ensure_one()
        # check token field exists
        if self._mail_post_token_field not in self._fields:
            raise NotImplementedError(_(
                "Model %(model_name)s does not support token signature, as it does not have %(field_name)s field.",
                model_name=self._name,
                field_name=self._mail_post_token_field
            ))
        # sign token
        secret = self.env["ir.config_parameter"].sudo().get_param("database.secret")
        token = (self.env.cr.dbname, self[self._mail_post_token_field], pid)
        return hmac.new(secret.encode('utf-8'), repr(token).encode('utf-8'), hashlib.sha256).hexdigest()

    def _portal_get_parent_hash_token(self, pid):
        """ Overridden in models which have M2o 'parent' field and can be shared on
        either an individual basis or indirectly in a group via the M2o record.

        :return: False or logical parent's _sign_token() result
        """
        return False

```

## File: models\portal_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import uuid
from ast import literal_eval
from werkzeug.urls import url_encode
from odoo import api, exceptions, fields, models, _


class PortalMixin(models.AbstractModel):
    _name = "portal.mixin"
    _description = 'Portal Mixin'

    access_url = fields.Char(
        'Portal Access URL', compute='_compute_access_url',
        help='Customer Portal URL')
    access_token = fields.Char('Security Token', copy=False)

    # to display the warning from specific model
    access_warning = fields.Text("Access warning", compute="_compute_access_warning")

    def _compute_access_warning(self):
        for mixin in self:
            mixin.access_warning = ''

    def _compute_access_url(self):
        for record in self:
            record.access_url = '#'

    def _portal_ensure_token(self):
        """ Get the current record access token """
        if not self.access_token:
            # we use a `write` to force the cache clearing otherwise `return self.access_token` will return False
            self.sudo().write({'access_token': str(uuid.uuid4())})
        return self.access_token

    def _get_share_url(self, redirect=False, signup_partner=False, pid=None, share_token=True):
        """
        Build the url of the record  that will be sent by mail and adds additional parameters such as
        access_token to bypass the recipient's rights,
        signup_partner to allows the user to create easily an account,
        hash token to allow the user to be authenticated in the chatter of the record portal view, if applicable
        :param redirect : Send the redirect url instead of the direct portal share url
        :param signup_partner: allows the user to create an account with pre-filled fields.
        :param pid: = partner_id - when given, a hash is generated to allow the user to be authenticated
            in the portal chatter, if any in the target page,
            if the user is redirected to the portal instead of the backend.
        :return: the url of the record with access parameters, if any.
        """
        self.ensure_one()
        if redirect:
            # model / res_id used by mail/view to check access on record
            params = {
                'model': self._name,
                'res_id': self.id,
            }
        else:
            params = {}
        if share_token and hasattr(self, 'access_token'):
            params['access_token'] = self._portal_ensure_token()
        if pid:
            params['pid'] = pid
            params['hash'] = self._sign_token(pid)
        if signup_partner and hasattr(self, 'partner_id') and self.partner_id:
            params.update(self.partner_id.signup_get_auth_param()[self.partner_id.id])

        return '%s?%s' % ('/mail/view' if redirect else self.access_url, url_encode(params))

    def _get_access_action(self, access_uid=None, force_website=False):
        """ Instead of the classic form view, redirect to the online document for
        portal users or if force_website=True. """
        self.ensure_one()

        user, record = self.env.user, self
        if access_uid:
            try:
                record.check_access_rights('read')
                record.check_access_rule("read")
            except exceptions.AccessError:
                return super(PortalMixin, self)._get_access_action(
                    access_uid=access_uid, force_website=force_website
                )
            user = self.env['res.users'].sudo().browse(access_uid)
            record = self.with_user(user)
        if user.share or force_website:
            try:
                record.check_access_rights('read')
                record.check_access_rule('read')
            except exceptions.AccessError:
                if force_website:
                    return {
                        'type': 'ir.actions.act_url',
                        'url': record.access_url,
                        'target': 'self',
                        'res_id': record.id,
                    }
                else:
                    pass
            else:
                return {
                    'type': 'ir.actions.act_url',
                    'url': record._get_share_url(),
                    'target': 'self',
                    'res_id': record.id,
                }
        return super(PortalMixin, self)._get_access_action(
            access_uid=access_uid, force_website=force_website
        )

    @api.model
    def action_share(self):
        action = self.env["ir.actions.actions"]._for_xml_id("portal.portal_share_action")
        action['context'] = {'active_id': self.env.context['active_id'],
                             'active_model': self.env.context['active_model'],
                             **literal_eval(action['context'])}
        return action

    def get_portal_url(self, suffix=None, report_type=None, download=None, query_string=None, anchor=None):
        """
            Get a portal url for this model, including access_token.
            The associated route must handle the flags for them to have any effect.
            - suffix: string to append to the url, before the query string
            - report_type: report_type query string, often one of: html, pdf, text
            - download: set the download query string to true
            - query_string: additional query string
            - anchor: string to append after the anchor #
        """
        self.ensure_one()
        url = self.access_url + '%s?access_token=%s%s%s%s%s' % (
            suffix if suffix else '',
            self._portal_ensure_token(),
            '&report_type=%s' % report_type if report_type else '',
            '&download=true' if download else '',
            query_string if query_string else '',
            '#%s' % anchor if anchor else ''
        )
        return url

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    portal_allow_api_keys = fields.Boolean(
        string='Customer API Keys',
        compute='_compute_portal_allow_api_keys',
        inverse='_inverse_portal_allow_api_keys',
    )

    def _compute_portal_allow_api_keys(self):
        for setting in self:
            setting.portal_allow_api_keys = self.env['ir.config_parameter'].sudo().get_param('portal.allow_api_keys')

    def _inverse_portal_allow_api_keys(self):
        self.env['ir.config_parameter'].sudo().set_param('portal.allow_api_keys', self.portal_allow_api_keys)

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        res['portal_allow_api_keys'] = bool(self.env['ir.config_parameter'].sudo().get_param('portal.allow_api_keys'))
        return res

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _can_edit_name(self):
        """ Name can be changed more often than the VAT """
        self.ensure_one()
        return True

    def can_edit_vat(self):
        """ `vat` is a commercial field, synced between the parent (commercial
        entity) and the children. Only the commercial entity should be able to
        edit it (as in backend)."""
        self.ensure_one()
        return not self.parent_id

```

## File: models\res_users_apikeys_description.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.exceptions import AccessError


class APIKeyDescription(models.TransientModel):
    _inherit = 'res.users.apikeys.description'

    def check_access_make_key(self):
        try:
            return super().check_access_make_key()
        except AccessError:
            if self.env['ir.config_parameter'].sudo().get_param('portal.allow_api_keys'):
                if self.user_has_groups('base.group_portal'):
                    return
                else:
                    raise AccessError(_("Only internal and portal users can create API keys"))
            raise

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import ir_ui_view
from . import ir_qweb
from . import mail_thread
from . import mail_message
from . import portal_mixin
from . import res_config_settings
from . import res_partner
from . import res_users_apikeys_description

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_portal_share","access.portal.share","model_portal_share","base.group_partner_manager",1,1,1,0
"access_portal_wizard","access.portal.wizard","model_portal_wizard","base.group_partner_manager",1,1,1,0
"access_portal_wizard_user","access.portal.wizard.user","model_portal_wizard_user","base.group_partner_manager",1,1,1,0

```

## File: static\src\img\portal-addresses.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M42.7366 3.2766C39.6798 1.50012 35.4465 1.75104 30.7766 4.44716C21.4986 9.80391 14.0014 22.7894 14.0317 33.4603C14.0619 44.1214 30.0964 58.0101 30.9403 58.9143L35.984 61.8578L47.7803 6.22019L42.7366 3.2766Z" fill="#FBDBD0"/>
<path d="M35.8202 7.3908C26.5422 12.7475 19.045 25.733 19.0753 36.4039C19.1054 47.065 35.14 60.9538 35.984 61.8579C36.8204 59.9835 52.7133 27.6616 52.6831 17.0005C52.6655 10.8105 50.116 6.76586 46.1671 5.50602C43.3085 4.594 39.7163 5.14141 35.8202 7.3908Z" fill="white"/>
<path d="M47.1275 25.7108C44.1739 34.1529 36.8024 41.3561 30.6627 41.7993C24.5231 42.2427 21.9404 35.7584 24.894 27.3162C27.8476 18.8742 35.2191 11.671 41.3587 11.2277C47.4984 10.7844 50.0811 17.2687 47.1275 25.7108Z" fill="#C1DBF6" stroke="#374874" stroke-linecap="square" stroke-linejoin="round"/>
<path d="M38.3178 2.14219C39.9616 2.14219 41.4529 2.53069 42.7366 3.27668L47.7803 6.22032L47.7798 6.22256C50.796 7.97579 52.668 11.7008 52.6831 17.0005C52.7132 27.6615 36.8204 59.9836 35.984 61.8579L30.9403 58.9143C30.0964 58.0101 14.0619 44.1214 14.0317 33.4603C14.0014 22.7895 21.4986 9.80404 30.7765 4.44724C33.4855 2.88329 36.0471 2.14219 38.3178 2.14219ZM38.3179 1.22793C35.8006 1.22793 33.1095 2.04466 30.3194 3.65538C25.6949 6.32546 21.363 10.8885 18.1217 16.5039C14.8796 22.1209 13.1023 28.1437 13.1174 33.4629C13.1296 37.7625 15.5482 42.9875 20.3062 48.993C24.0297 53.6928 28.1483 57.5443 29.6988 58.9942C29.9636 59.2418 30.2136 59.4757 30.2719 59.5381C30.3326 59.6032 30.4026 59.6591 30.4795 59.7039L35.5231 62.6476C35.6644 62.73 35.8237 62.7722 35.984 62.7722C36.0782 62.7722 36.1726 62.7577 36.2641 62.7282C36.5113 62.6487 36.7131 62.4678 36.8189 62.2305C36.8929 62.0648 37.1126 61.6026 37.4167 60.9629C48.0204 38.6569 53.6156 23.4541 53.5974 16.9979C53.5819 11.569 51.6999 7.4768 48.2959 5.4653C48.2782 5.45325 48.2599 5.44164 48.2412 5.43069L43.1975 2.48706C41.7593 1.65124 40.118 1.22793 38.3179 1.22793Z" fill="#374874"/>
</svg>

```

## File: static\src\img\portal-connection.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M33.6091 58.1611C38.9578 58.1611 43.2937 56.3324 43.2937 54.0766C43.2937 51.8207 38.9578 49.992 33.6091 49.992C28.2605 49.992 23.9246 51.8207 23.9246 54.0766C23.9246 56.3324 28.2605 58.1611 33.6091 58.1611Z" fill="#C1DBF6"/>
<path d="M45.7715 21.9257C44.6789 18.3486 42.3429 15.4983 39.1612 13.9006C39.0151 13.8271 38.8539 13.7887 38.6903 13.7886C38.3191 13.7866 37.9732 13.9768 37.7761 14.2914C37.5063 14.7272 37.6408 15.2992 38.0766 15.569C38.1036 15.5857 38.1315 15.601 38.1601 15.6149C40.9029 16.9589 42.9235 19.4046 43.8743 22.504C44.6369 25.0794 44.644 27.8196 43.8949 30.3989C43.1675 33.0055 41.7722 35.377 39.8469 37.2789C39.4809 37.6267 39.4662 38.2054 39.814 38.5714C39.8158 38.5733 39.8177 38.5753 39.8195 38.5772L39.8995 38.6617C40.0772 38.851 40.3256 38.9578 40.5852 38.9566C40.8282 38.958 41.0618 38.8626 41.2343 38.6914C45.7441 34.248 47.5246 27.6674 45.7715 21.9257Z" fill="#C1DBF6"/>
<path d="M27.2754 40.7326C26.8674 40.6043 26.4674 40.4517 26.0777 40.2754C23.2046 38.9611 21.0811 36.4697 20.1028 33.2606C18.7314 28.776 19.8537 23.5783 23.0286 19.6949C23.3567 19.3111 23.3116 18.7339 22.9278 18.4058C22.8655 18.3526 22.7964 18.308 22.7223 18.2731C22.5729 18.1996 22.4087 18.1613 22.2423 18.1611C21.9152 18.1599 21.6054 18.308 21.4011 18.5634C17.8903 22.9406 16.6674 28.7943 18.2011 33.8389C19.344 37.5966 21.8583 40.5223 25.2457 42.0674C25.747 42.2958 26.2623 42.492 26.7886 42.6549C26.8774 42.6822 26.9699 42.6961 27.0628 42.696C27.4779 42.6944 27.8424 42.4201 27.9588 42.0217C28.1199 41.4776 27.8162 40.9046 27.2754 40.7326Z" fill="#C1DBF6"/>
<path d="M53.3463 18.392C51.6549 12.856 48.0252 8.44458 43.1223 5.97372C42.9463 5.8857 42.7523 5.83955 42.5555 5.83887C42.113 5.84104 41.7024 6.06961 41.4675 6.44458C41.1546 6.94275 41.3048 7.60022 41.803 7.91308C41.8356 7.93359 41.8694 7.95231 41.904 7.96915C46.3178 10.136 49.5886 14.0812 51.1132 19.0732C52.3395 23.2079 52.3522 27.6079 51.1498 31.7497C49.9818 35.9289 47.7426 39.7305 44.6538 42.7783C44.2364 43.1841 44.2221 43.8498 44.6218 44.2732L44.7475 44.408C44.9516 44.6231 45.2349 44.7453 45.5315 44.7463C45.8122 44.7479 46.0819 44.6377 46.2812 44.44C49.6643 41.1012 52.1177 36.9374 53.3989 32.36C54.7178 27.6469 54.7041 22.8194 53.3463 18.392Z" fill="#C1DBF6"/>
<path d="M24.4411 48.3188C23.7823 48.1091 23.1366 47.8603 22.5074 47.5737C17.872 45.4525 14.448 41.4365 12.8754 36.2663C10.6766 29.0685 12.4731 20.7234 17.5657 14.5017C17.9416 14.0475 17.8782 13.3747 17.424 12.9988C17.3608 12.9465 17.2918 12.9016 17.2183 12.8651C17.037 12.7755 16.8377 12.7286 16.6354 12.728C16.2393 12.7268 15.8641 12.906 15.616 13.2148C10.1486 20.0171 8.24686 29.112 10.64 36.9474C12.4183 42.7645 16.288 47.2903 21.5383 49.6925C22.3124 50.0484 23.1088 50.3538 23.9223 50.6068C24.0244 50.6387 24.1307 50.6549 24.2377 50.6548C24.7161 50.6529 25.1363 50.3368 25.2709 49.8777C25.4699 49.2183 25.0993 48.522 24.4411 48.3188Z" fill="#C1DBF6"/>
<path d="M32.6058 52.7509C32.6058 52.5589 33.0972 32.1269 33.1498 30.0103H32.5326C32.5326 30.0103 31.9863 52.728 31.9863 52.9315C31.9863 53.3086 32.7109 53.6172 33.6023 53.6172C34.0131 53.6255 34.4212 53.5476 34.8 53.3886C34.6066 53.4205 34.4109 53.4365 34.2149 53.4366C33.3303 53.4343 32.6058 53.128 32.6058 52.7509Z" fill="#C1DBF6"/>
<path d="M35.2251 52.9315C35.2251 52.728 34.6789 30.0103 34.6789 30.0103H33.1497C33.0971 32.1269 32.6057 52.5589 32.6057 52.7509C32.6057 53.128 33.3303 53.4366 34.2217 53.4366C34.4216 53.4371 34.6211 53.4211 34.8183 53.3886C35.0651 53.2652 35.2251 53.1075 35.2251 52.9315Z" fill="white"/>
<path d="M34.6789 30.0103C34.6789 30.0103 35.2252 52.728 35.2252 52.9314C35.2252 53.1074 35.0652 53.2652 34.8183 53.3886C34.8091 53.3901 34.7998 53.3905 34.7906 53.392C34.4325 53.5408 34.0488 53.6177 33.6613 53.6177C33.6417 53.6177 33.622 53.6175 33.6023 53.6171C32.7109 53.6171 31.9863 53.3086 31.9863 52.9314C31.9863 52.728 32.5326 30.0103 32.5326 30.0103H34.6789ZM35.5715 29.096H31.64L31.6186 29.9883C31.5273 33.7837 31.072 52.7273 31.072 52.9314C31.072 53.8716 32.1082 54.5288 33.5932 54.5314C33.6159 54.5318 33.6386 54.5321 33.6613 54.5321C34.1317 54.5321 34.5921 54.4473 35.0313 54.2801L35.1035 54.2682L35.2272 54.2064C36.0209 53.8095 36.1394 53.2372 36.1394 52.9315C36.1394 52.7274 35.6842 33.7838 35.5929 29.9883L35.5715 29.096Z" fill="#374874"/>
<path d="M33.6023 21.4343C36.4853 21.4353 38.8222 23.7722 38.8232 26.6553C38.8243 29.5398 36.4868 31.879 33.6023 31.88C33.2678 31.8795 32.9341 31.8474 32.6057 31.784C30.5122 31.3794 28.8752 29.7424 28.4706 27.649C27.9235 24.8181 29.7749 22.0797 32.6057 21.5326C32.9339 21.4673 33.2677 21.4344 33.6023 21.4343ZM33.6026 20.52H33.602C33.2085 20.5201 32.8133 20.5591 32.4273 20.6359C29.1109 21.2768 26.931 24.5012 27.573 27.8225C28.0511 30.2962 29.9584 32.2036 32.4322 32.6817C32.8166 32.7559 33.2097 32.7938 33.6009 32.7943C36.9866 32.7931 39.7387 30.039 39.7375 26.655C39.7363 23.2734 36.9842 20.5213 33.6026 20.52Z" fill="#374874"/>
<path d="M33.6137 31.9221C36.5214 31.9221 38.8786 29.565 38.8786 26.6572C38.8786 23.7495 36.5214 21.3923 33.6137 21.3923C30.7059 21.3923 28.3488 23.7495 28.3488 26.6572C28.3488 29.565 30.7059 31.9221 33.6137 31.9221Z" fill="white"/>
<path d="M33.6023 21.4343C33.2677 21.4344 32.9339 21.4673 32.6057 21.5326C35.4366 22.0797 37.2879 24.8181 36.7408 27.6489C36.3361 29.7424 34.6991 31.3794 32.6057 31.784C32.9341 31.8474 33.2678 31.8795 33.6023 31.88C36.4868 31.879 38.8243 29.5398 38.8233 26.6552C38.8222 23.7722 36.4853 21.4353 33.6023 21.4343Z" fill="#FBDBD0"/>
</svg>

```

## File: static\src\js\portal.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.portalDetails = publicWidget.Widget.extend({
    selector: '.o_portal_details',
    events: {
        'change select[name="country_id"]': '_onCountryChange',
    },

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        this.$state = this.$('select[name="state_id"]');
        this.$stateOptions = this.$state.filter(':enabled').find('option:not(:first)');
        this._adaptAddressForm();

        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptAddressForm: function () {
        var $country = this.$('select[name="country_id"]');
        var countryID = ($country.val() || 0);
        this.$stateOptions.detach();
        var $displayedState = this.$stateOptions.filter('[data-country_id=' + countryID + ']');
        var nb = $displayedState.appendTo(this.$state).show().length;
        this.$state.parent().toggle(nb >= 1);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onCountryChange: function () {
        this._adaptAddressForm();
    },
});

export const PortalHomeCounters = publicWidget.Widget.extend({
    selector: '.o_portal_my_home',

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this._updateCounters();
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Return a list of counters name linked to a line that we want to keep
     * regardless of the number of documents present
     * @private
     * @returns {Array}
     */
    _getCountersAlwaysDisplayed() {
        return [];
    },

    /**
     * @private
     */
    async _updateCounters(elem) {
        const numberRpc = 3;
        const needed = Object.values(this.el.querySelectorAll('[data-placeholder_count]'))
                                .map(documentsCounterEl => documentsCounterEl.dataset['placeholder_count']);
        const counterByRpc = Math.ceil(needed.length / numberRpc);  // max counter, last can be less
        const countersAlwaysDisplayed = this._getCountersAlwaysDisplayed();

        const proms = [...Array(Math.min(numberRpc, needed.length)).keys()].map(async i => {
            const documentsCountersData = await this.rpc("/my/counters", {
                counters: needed.slice(i * counterByRpc, (i + 1) * counterByRpc)
            });
            Object.keys(documentsCountersData).forEach(counterName => {
                const documentsCounterEl = this.el.querySelector(`[data-placeholder_count='${counterName}']`);
                documentsCounterEl.textContent = documentsCountersData[counterName];
                // The element is hidden by default, only show it if its counter is > 0 or if it's in the list of counters always shown
                if (documentsCountersData[counterName] !== 0 || countersAlwaysDisplayed.includes(counterName)) {
                    documentsCounterEl.closest('.o_portal_index_card').classList.remove('d-none');
                }
            });
            return documentsCountersData;
        });
        return Promise.all(proms).then((results) => {
            this.el.querySelector('.o_portal_doc_spinner').remove();
        });
    },
});

publicWidget.registry.PortalHomeCounters = PortalHomeCounters;

publicWidget.registry.portalSearchPanel = publicWidget.Widget.extend({
    selector: '.o_portal_search_panel',
    events: {
        'click .dropdown-item': '_onDropdownItemClick',
        'submit': '_onSubmit',
    },

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        this._adaptSearchLabel(this.$('.dropdown-item.active'));
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _adaptSearchLabel: function (elem) {
        var $label = $(elem).clone();
        $label.find('span.nolabel').remove();
        this.$('input[name="search"]').attr('placeholder', $label.text().trim());
    },
    /**
     * @private
     */
    _search: function () {
        var search = new URL(window.location).searchParams;
        search.set("search_in", this.$('.dropdown-item.active').attr('href')?.replace('#', '') || "");
        search.set("search", this.$('input[name="search"]').val());
        window.location.search = search.toString();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onDropdownItemClick: function (ev) {
        ev.preventDefault();
        var $item = $(ev.currentTarget);
        $item.closest('.dropdown-menu').find('.dropdown-item').removeClass('active');
        $item.addClass('active');

        this._adaptSearchLabel(ev.currentTarget);
    },
    /**
     * @private
     */
    _onSubmit: function (ev) {
        ev.preventDefault();
        this._search();
    },
});

```

## File: static\src\js\portal_chatter.js

```javascript
/** @odoo-module **/

import { renderToElement } from "@web/core/utils/render";
import dom from "@web/legacy/js/core/dom";
import publicWidget from "@web/legacy/js/public/public_widget";
import portalComposer from "@portal/js/portal_composer";
import { range } from "@web/core/utils/numbers";

import { Component, markup } from "@odoo/owl";

/**
 * Widget PortalChatter
 *
 * - Fetch message fron controller
 * - Display chatter: pager, total message, composer (according to access right)
 * - Provider API to filter displayed messages
 */
var PortalChatter = publicWidget.Widget.extend({
    template: 'portal.Chatter',
    events: {
        'click .o_portal_chatter_pager_btn': '_onClickPager',
        'click .o_portal_chatter_js_is_internal': 'async _onClickUpdateIsInternal',
    },

    /**
     * @constructor
     */
    init: function (parent, options) {
        this.options = {};
        this._super.apply(this, arguments);

        this._setOptions(options);

        this.set('messages', []);
        this.set('message_count', this.options['message_count']);
        this.set('pager', {});
        this.set('domain', this.options['domain']);
        this._currentPage = this.options['pager_start'];

        this.rpc = this.bindService("rpc");
    },
    /**
     * @override
     */
    willStart: function () {
        return Promise.all([
            this._super.apply(this, arguments),
            this._chatterInit()
        ]);
    },
    /**
     * @override
     */
    start: function () {
        // bind events
        this.on("change:messages", this, this._renderMessages);
        this.on("change:message_count", this, function () {
            this._renderMessageCount();
            this.set('pager', this._pager(this._currentPage));
        });
        this.on("change:pager", this, this._renderPager);
        this.on("change:domain", this, this._onChangeDomain);
        // set options and parameters
        this.set('message_count', this.options['message_count']);
        this.set('messages', this.preprocessMessages(this.result['messages']));
        // bind bus event: this (portal.chatter) and 'portal.rating.composer' in portal_rating
        // are separate and sibling widgets, this event is to be triggered from portal.rating.composer,
        // hence bus event is bound to achieve usage of the event in another widget.
        Component.env.bus.addEventListener('reload_chatter_content', (ev) => this._reloadChatterContent(ev.detail));

        return Promise.all([this._super.apply(this, arguments), this._reloadComposer()]);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Fetch the messages and the message count from the server for the
     * current page and current domain.
     *
     * @param {Array} domain
     * @returns {Promise}
     */
    messageFetch: function (domain) {
        var self = this;
        return this.rpc('/mail/chatter_fetch', self._messageFetchPrepareParams()).then(function (result) {
            self.set('messages', self.preprocessMessages(result['messages']));
            self.set('message_count', result['message_count']);
            return result;
        });
    },
    /**
     * Update the messages format
     *
     * @param {Array<Object>} messages
     * @returns {Array}
     */
    preprocessMessages(messages) {
        messages.forEach((m) => {
            m['body'] = markup(m.body);
        });
        return messages;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Set options
     *
     * @param {Array<string>} options: new options to set
     */
    _setOptions: function (options) {
        // underscorize the camelcased option keys
        const defaultOptions = Object.assign({
            'allow_composer': true,
            'display_composer': false,
            'csrf_token': odoo.csrf_token,
            'message_count': 0,
            'pager_step': 10,
            'pager_scope': 5,
            'pager_start': 1,
            'is_user_public': true,
            'is_user_employee': false,
            'is_user_publisher': false,
            'hash': false,
            'pid': false,
            'domain': [],
            'two_columns': false,
        }, this.options || {});

        this.options = Object.entries(options).reduce((acc, [key, value]) => {
            acc[
                //Camelized to Underscored key
                key
                    .split(/\.?(?=[A-Z])/)
                    .join("_")
                    .toLowerCase()
            ] = value;
            return acc;
        }, defaultOptions);
    },

    /**
     * Reloads chatter and message count after posting message
     *
     * @private
     */
    _reloadChatterContent: function (data) {
        this.messageFetch();
        this._reloadComposer();
    },
    _createComposerWidget: function () {
        return new portalComposer.PortalComposer(this, this.options);
    },
    /**
     * Destroy current composer widget and initialize and insert new widget
     *
     * @private
     */
    _reloadComposer: async function () {
        if (this._composer) {
            this._composer.destroy();
        }
        if (this.options.display_composer) {
            this._composer = this._createComposerWidget();
            await this._composer.appendTo(this.$('.o_portal_chatter_composer'));
        }
    },
    /**
     * @private
     * @returns {Deferred}
     */
    _chatterInit: function () {
        var self = this;
        return this.rpc('/mail/chatter_init', this._messageFetchPrepareParams()).then(function (result) {
            self.result = result;
            self.options = Object.assign(self.options, self.result['options'] || {});
            return result;
        });
    },
    /**
     * Change the current page by refreshing current domain
     *
     * @private
     * @param {Number} page
     * @param {Array} domain
     */
    _changeCurrentPage: function (page, domain) {
        this._currentPage = page;
        var d = domain ? domain : Object.assign({}, this.get("domain"));
        this.set('domain', d); // trigger fetch message
    },
    _messageFetchPrepareParams: function () {
        var self = this;
        var data = {
            'res_model': this.options['res_model'],
            'res_id': this.options['res_id'],
            'limit': this.options['pager_step'],
            'offset': (this._currentPage - 1) * this.options['pager_step'],
            'allow_composer': this.options['allow_composer'],
        };
        // add token field to allow to post comment without being logged
        if (self.options['token']) {
            data['token'] = self.options['token'];
        }
        if (self.options['hash'] && self.options['pid']) {
            Object.assign(data, {
                'hash': self.options['hash'],
                'pid': self.options['pid'],
            });
        }
        // add domain
        if (this.get('domain')) {
            data['domain'] = this.get('domain');
        }
        return data;
    },
    /**
     * Generate the pager data for the given page number
     *
     * @private
     * @param {Number} page
     * @returns {Object}
     */
    _pager: function (page) {
        page = page || 1;
        var total = this.get('message_count');
        var scope = this.options['pager_scope'];
        var step = this.options['pager_step'];

        // Compute Pager
        var pageCount = Math.ceil(parseFloat(total) / step);

        page = Math.max(1, Math.min(parseInt(page), pageCount));
        scope -= 1;

        var pmin = Math.max(page - parseInt(Math.floor(scope / 2)), 1);
        var pmax = Math.min(pmin + scope, pageCount);

        if (pmax - scope > 0) {
            pmin = pmax - scope;
        } else {
            pmin = 1;
        }

        var pages = [];
        range(pmin, pmax + 1).forEach(index => pages.push(index));

        return {
            "page_count": pageCount,
            "offset": (page - 1) * step,
            "page": page,
            "page_start": pmin,
            "page_previous": Math.max(pmin, page - 1),
            "page_next": Math.min(pmax, page + 1),
            "page_end": pmax,
            "pages": pages
        };
    },
    _renderMessages: function () {
        this.$('.o_portal_chatter_messages').empty().append(renderToElement("portal.chatter_messages", {widget: this}));
    },
    _renderMessageCount: function () {
        this.$('.o_message_counter').replaceWith(renderToElement("portal.chatter_message_count", {widget: this}));
    },
    _renderPager: function () {
        this.$('.o_portal_chatter_pager').replaceWith(renderToElement("portal.pager", {widget: this}));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onChangeDomain: function () {
        var self = this;
        return this.messageFetch().then(function () {
            var p = self._currentPage;
            self.set('pager', self._pager(p));
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickPager: function (ev) {
        ev.preventDefault();
        var page = $(ev.currentTarget).data('page');
        this._changeCurrentPage(page);
    },

    /**
     * Toggle is_internal state of message. Update both node data and
     * classes to ensure DOM is updated accordingly to RPC call result.
     * @private
     * @returns {Promise}
     */
    _onClickUpdateIsInternal: function (ev) {
        ev.preventDefault();

        var $elem = $(ev.currentTarget);
        return this.rpc('/mail/update_is_internal', {
            message_id: $elem.data('message-id'),
            is_internal: ! $elem.data('is-internal'),
        }).then(function (result) {
            $elem.data('is-internal', result);
            if (result === true) {
                $elem.addClass('o_portal_message_internal_on');
                $elem.removeClass('o_portal_message_internal_off');
            } else {
                $elem.addClass('o_portal_message_internal_off');
                $elem.removeClass('o_portal_message_internal_on');
            }
        });
    },
});

publicWidget.registry.portalChatter = publicWidget.Widget.extend({
    selector: '.o_portal_chatter',

    /**
     * @override
     */
    async start() {
        const proms = [this._super.apply(this, arguments)];
        const chatter = new PortalChatter(this, this.$el.data());
        proms.push(chatter.appendTo(this.$el));
        await Promise.all(proms);
        // scroll to the right place after chatter loaded
        if (window.location.hash === `#${this.el.id}`) {
            dom.scrollTo(this.el, {duration: 0});
        }
    },
});

export default PortalChatter

```

## File: static\src\js\portal_composer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { escape } from "@web/core/utils/strings";
import { renderToElement } from "@web/core/utils/render";
import publicWidget from "@web/legacy/js/public/public_widget";
import { post } from "@web/core/network/http_service";
import { Component } from "@odoo/owl";
import { RPCError } from "@web/core/network/rpc_service";

/**
 * Widget PortalComposer
 *
 * Display the composer (according to access right)
 *
 */
var PortalComposer = publicWidget.Widget.extend({
    template: 'portal.Composer',
    events: {
        'change .o_portal_chatter_file_input': '_onFileInputChange',
        'click .o_portal_chatter_attachment_btn': '_onAttachmentButtonClick',
        'click .o_portal_chatter_attachment_delete': 'async _onAttachmentDeleteClick',
        'click .o_portal_chatter_composer_btn': 'async _onSubmitButtonClick',
    },

    /**
     * @constructor
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.options = Object.assign({
            'allow_composer': true,
            'display_composer': false,
            'csrf_token': odoo.csrf_token,
            'token': false,
            'res_model': false,
            'res_id': false,
        }, options || {});
        this.attachments = [];
        this.rpc = this.bindService("rpc");
        this.notification = this.bindService("notification");
    },
    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$attachmentButton = this.$('.o_portal_chatter_attachment_btn');
        this.$fileInput = this.$('.o_portal_chatter_file_input');
        this.$sendButton = this.$('.o_portal_chatter_composer_btn');
        this.$attachments = this.$('.o_portal_chatter_composer_input .o_portal_chatter_attachments');
        this.$inputTextarea = this.$('.o_portal_chatter_composer_input textarea[name="message"]');

        return this._super.apply(this, arguments).then(function () {
            if (self.options.default_attachment_ids) {
                self.attachments = self.options.default_attachment_ids || [];
                self.attachments.forEach((attachment) => {
                    attachment.state = 'done';
                });
                self._updateAttachments();
            }
            return Promise.resolve();
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onAttachmentButtonClick: function () {
        this.$fileInput.click();
    },
    /**
     * @private
     * @param {Event} ev
     * @returns {Promise}
     */
    _onAttachmentDeleteClick: function (ev) {
        var self = this;
        var attachmentId = $(ev.currentTarget).closest('.o_portal_chatter_attachment').data('id');
        var accessToken = this.attachments.find(attachment => attachment.id === attachmentId).access_token;
        ev.preventDefault();
        ev.stopPropagation();

        this.$sendButton.prop('disabled', true);

        return this.rpc('/portal/attachment/remove', {
            'attachment_id': attachmentId,
            'access_token': accessToken,
        }).then(function () {
            self.attachments = self.attachments.filter(attachment => attachment.id !== attachmentId);
            self._updateAttachments();
            self.$sendButton.prop('disabled', false);
        });
    },
    _prepareAttachmentData: function (file) {
        return {
            'name': file.name,
            'file': file,
            'res_id': this.options.res_id,
            'res_model': this.options.res_model,
            'access_token': this.options.token,
        };
    },
    /**
     * @private
     * @returns {Promise}
     */
    _onFileInputChange: function () {
        var self = this;

        this.$sendButton.prop('disabled', true);

        return Promise.all([...this.$fileInput[0].files].map((file) => {
            return new Promise(function (resolve, reject) {
                var data = self._prepareAttachmentData(file);
                if (odoo.csrf_token) {
                    data.csrf_token = odoo.csrf_token;
                }
                post('/portal/attachment/add', data).then(function (attachment) {
                    attachment.state = 'pending';
                    self.attachments.push(attachment);
                    self._updateAttachments();
                    resolve();
                }).catch(function (error) {
                    if (error instanceof RPCError) {
                        self.notification.add(
                            _t("Could not save file <strong>%s</strong>", escape(file.name)),
                            { type: 'warning', sticky: true }
                        );
                        resolve();
                    }
                });
            });
        })).then(function () {
            // ensures any selection triggers a change, even if the same files are selected again
            self.$fileInput[0].value = null;
            self.$sendButton.prop('disabled', false);
        });
    },
    /**
     * prepares data to send message
     *
     * @private
     */
    _prepareMessageData: function () {
        return Object.assign(this.options || {}, {
            'message': this.$('textarea[name="message"]').val(),
            attachment_ids: this.attachments.map((a) => a.id),
            attachment_tokens: this.attachments.map((a) => a.access_token),
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onSubmitButtonClick: function (ev) {
        ev.preventDefault();
        const error = this._onSubmitCheckContent();
        if (error) {
            this.$inputTextarea.addClass('border-danger');
            this.$(".o_portal_chatter_composer_error").text(error).removeClass('d-none');
            return Promise.reject();
        } else {
            return this._chatterPostMessage(ev.currentTarget.getAttribute('data-action'));
        }
    },

    /**
     * @private
     */
    _onSubmitCheckContent: function () {
        if (!this.$inputTextarea.val().trim() && !this.attachments.length) {
            return _t('Some fields are required. Please make sure to write a message or attach a document');
        };
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _updateAttachments: function () {
        this.$attachments.empty().append(renderToElement('portal.Chatter.Attachments', {
            attachments: this.attachments,
            showDelete: true,
        }));
    },
    /**
     * post message using rpc call and display new message and message count
     *
     * @private
     * @param {String} route
     * @returns {Promise}
     */
    _chatterPostMessage: async function (route) {
        const result = await this.rpc(route, this._prepareMessageData());
        Component.env.bus.trigger('reload_chatter_content', result);
        return result;
    },
});

export default {
    PortalComposer: PortalComposer,
};

```

## File: static\src\js\portal_security.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from '@web/core/confirmation_dialog/confirmation_dialog';
import { renderToMarkup } from "@web/core/utils/render";
import publicWidget from '@web/legacy/js/public/public_widget';
import { session } from "@web/session";
import { InputConfirmationDialog } from './components/input_confirmation_dialog/input_confirmation_dialog';
import { _t } from "@web/core/l10n/translation";

publicWidget.registry.NewAPIKeyButton = publicWidget.Widget.extend({
    selector: '.o_portal_new_api_key',
    events: {
        click: '_onClick'
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
        this.dialog = this.bindService("dialog");
    },

    async _onClick(e){
        e.preventDefault();
        // This call is done just so it asks for the password confirmation before starting displaying the
        // dialog forms, to mimic the behavior from the backend, in which it asks for the password before
        // displaying the wizard.
        // The result of the call is unused. But it's required to call a method with the decorator `@check_identity`
        // in order to use `handleCheckIdentity`.
        await handleCheckIdentity(
            this.orm.call("res.users", "api_key_wizard", [session.user_id]),
            this.orm,
            this.dialog
        );

        this.call("dialog", "add", InputConfirmationDialog, {
            title: _t("New API Key"),
            body: renderToMarkup("portal.keydescription"),
            confirmLabel: _t("Confirm"),
            confirm: async ({ inputEl }) => {
                const description = inputEl.value;
                const wizard_id = await this.orm.create("res.users.apikeys.description", [{ name: description }]);
                const res = await handleCheckIdentity(
                    this.orm.call("res.users.apikeys.description", "make_key", [wizard_id]),
                    this.orm,
                    this.dialog
                );

                this.call("dialog", "add", ConfirmationDialog, {
                    title: _t("API Key Ready"),
                    body: renderToMarkup("portal.keyshow", { key: res.context.default_key }),
                    confirmLabel: _t("Close"),
                }, {
                    onClose: () => {
                        window.location = window.location;
                    },
                })
            }
        });
    }
});

publicWidget.registry.RemoveAPIKeyButton = publicWidget.Widget.extend({
    selector: '.o_portal_remove_api_key',
    events: {
        click: '_onClick'
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
        this.dialog = this.bindService("dialog");
    },

    async _onClick(e){
        e.preventDefault();
        await handleCheckIdentity(
            this.orm.call("res.users.apikeys", "remove", [parseInt(this.el.id)]),
            this.orm,
            this.dialog
        );
        window.location = window.location;
    }
});

publicWidget.registry.portalSecurity = publicWidget.Widget.extend({
    selector: '.o_portal_security_body',

    /**
     * @override
     */
    init: function () {
        // Show the "deactivate your account" modal if needed
        $('.modal.show#portal_deactivate_account_modal').removeClass('d-block').modal('show');

        // Remove the error messages when we close the modal,
        // so when we re-open it again we get a fresh new form
        $('.modal#portal_deactivate_account_modal').on('hide.bs.modal', (event) => {
            const $target = $(event.currentTarget);
            $target.find('.alert').remove();
            $target.find('.invalid-feedback').remove();
            $target.find('.is-invalid').removeClass('is-invalid');
        });

        return this._super(...arguments);
    },

});

/**
 * Defining what happens when you click the "Log out from all devices"
 * on the "/my/security" page.
 */
publicWidget.registry.RevokeSessionsButton = publicWidget.Widget.extend({
    selector: '#portal_revoke_all_sessions_popup',
    events: {
        click: '_onClick',
    },

    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    async _onClick() {
        const { res_id: checkId } = await this.orm.call("res.users", "api_key_wizard", [
            session.user_id,
        ]);
        this.call("dialog", "add", InputConfirmationDialog, {
            title: _t("Log out from all devices?"),
            body: renderToMarkup("portal.revoke_all_devices_popup_template"),
            confirmLabel: _t("Log out from all devices"),
            confirm: async ({ inputEl }) => {
                if (!inputEl.reportValidity()) {
                    inputEl.classList.add("is-invalid");
                    return false;
                }

                await this.orm.write("res.users.identitycheck", [checkId], { password: inputEl.value });
                try {
                    await this.orm.call(
                        "res.users.identitycheck",
                        "revoke_all_devices",
                        [checkId]
                    );
                } catch {
                    inputEl.classList.add("is-invalid");
                    inputEl.setCustomValidity(_t("Check failed"));
                    inputEl.reportValidity();
                    return false;
                }

                window.location.href = "/web/session/logout?redirect=/";
                return true;
            },
            cancel: () => {},
            onInput: ({ inputEl }) => {
                inputEl.classList.remove("is-invalid");
                inputEl.setCustomValidity("");
            },
        });
    },
});

/**
 * Wraps an RPC call in a check for the result being an identity check action
 * descriptor. If no such result is found, just returns the wrapped promise's
 * result as-is; otherwise shows an identity check dialog and resumes the call
 * on success.
 *
 * Warning: does not in and of itself trigger an identity check, a promise which
 * never triggers and identity check internally will do nothing of use.
 *
 * @param {Promise} wrapped promise to check for an identity check request
 * @param {Function} ormService bound do the widget
 * @param {Function} dialogService dialog service
 * @returns {Promise} result of the original call
 */
export async function handleCheckIdentity(wrapped, ormService, dialogService) {
    return wrapped.then((r) => {
        if (!(r.type === "ir.actions.act_window" && r.res_model === "res.users.identitycheck")) {
            return r;
        }
        const checkId = r.res_id;
        return new Promise((resolve) => {
            dialogService.add(InputConfirmationDialog, {
                title: _t("Security Control"),
                body: renderToMarkup("portal.identitycheck"),
                confirmLabel: _t("Confirm Password"),
                confirm: async ({ inputEl }) => {
                    if (!inputEl.reportValidity()) {
                        inputEl.classList.add("is-invalid");
                        return false;
                    }
                    let result;
                    await ormService.write("res.users.identitycheck", [checkId], { password: inputEl.value });
                    try {
                        result = await ormService.call("res.users.identitycheck", "run_check", [checkId]);
                    } catch {
                        inputEl.classList.add("is-invalid");
                        inputEl.setCustomValidity(_t("Check failed"));
                        inputEl.reportValidity();
                        return false;
                    }
                    resolve(result);
                    return true;
                },
                cancel: () => {},
                onInput: ({ inputEl }) => {
                    inputEl.classList.remove("is-invalid");
                    inputEl.setCustomValidity("");
                },
            });
        });
    });
}

```

## File: static\src\js\portal_sidebar.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { deserializeDateTime } from "@web/core/l10n/dates";

const { DateTime } = luxon;

var PortalSidebar = publicWidget.Widget.extend({
    /**
     * @override
     */
    start: function () {
        this._setDelayLabel();
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //---------------------------------------------------------------------------

    /**
     * Set the due/delay information according to the given date
     * like : <span class="o_portal_sidebar_timeago" t-att-datetime="invoice.date_due"/>
     *
     * @private
     */
    _setDelayLabel: function () {
        var $sidebarTimeago = this.$el.find('.o_portal_sidebar_timeago').toArray();
        $sidebarTimeago.forEach((el) => {
            var dateTime = deserializeDateTime($(el).attr('datetime')),
                today = DateTime.now().startOf('day'),
                diff = dateTime.diff(today).as("days"),
                displayStr;

                if (diff === 0) {
                    displayStr = _t('Due today');
                } else if (diff > 0) {
                    // Workaround: force uniqueness of these two translations. We use %1d because the string
                    // with %d is already used in mail and mail's translations are not sent to the frontend.
                    displayStr = _t('Due in %s days', Math.abs(diff).toFixed());
                } else {
                    displayStr = _t('%s days overdue', Math.abs(diff).toFixed());
                }
                $(el).text(displayStr);
        });
    },
    /**
     * @private
     * @param {string} href
     */
    _printIframeContent: function (href) {
        if (!this.printContent) {
            this.printContent = $('<iframe id="print_iframe_content" src="' + href + '" style="display:none"></iframe>');
            this.$el.append(this.printContent);
            this.printContent.on('load', function () {
                $(this).get(0).contentWindow.print();
            });
        } else {
            this.printContent.get(0).contentWindow.print();
        }
    },
});
export default PortalSidebar;

```

## File: static\src\js\components\input_confirmation_dialog\input_confirmation_dialog.js

```javascript
/** @odoo-module */

import { useEffect } from "@odoo/owl";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

export class InputConfirmationDialog extends ConfirmationDialog {
    static props = {
        ...ConfirmationDialog.props,
        onInput: { type: Function, optional: true },
    };
    static template = "portal.InputConfirmationDialog";

    setup() {
        super.setup();

        const onInput = () => {
            if (this.props.onInput) {
                this.props.onInput({ inputEl: this.inputEl });
            }
        };
        const onKeydown = (ev) => {
            if (ev.key && ev.key.toLowerCase() === "enter") {
                ev.preventDefault();
                this._confirm();
            }
        };
        useEffect(
            (inputEl) => {
                this.inputEl = inputEl;
                if (this.inputEl) {
                    this.inputEl.focus();
                    this.inputEl.addEventListener("keydown", onKeydown);
                    this.inputEl.addEventListener("input", onInput);
                    return () => {
                        this.inputEl.removeEventListener("keydown", onKeydown);
                        this.inputEl.removeEventListener("input", onInput);
                    };
                }
            },
            () => [this.modalRef.el?.querySelector("input")]
        );
    }

    _confirm() {
        this.execButton(() => this.props.confirm({ inputEl: this.inputEl }));
    }
}

```

## File: static\src\js\components\input_confirmation_dialog\input_confirmation_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

  <t t-name="portal.InputConfirmationDialog" t-inherit="web.ConfirmationDialog">
    <xpath expr="//p[hasclass('text-prewrap')]" position="attributes">
        <attribute name="class"></attribute>
    </xpath>
  </t>

</templates>

```

## File: static\src\signature_form\signature_form.js

```javascript
/** @odoo-module **/

import { Component, onMounted, useRef, useState } from "@odoo/owl";
import dom from "@web/legacy/js/core/dom";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { redirect } from "@web/core/utils/urls";
import { NameAndSignature } from "@web/core/signature/name_and_signature";
import { useService } from "@web/core/utils/hooks";

/**
 * This Component is a signature request form. It uses
 * @see NameAndSignature for the input fields, adds a submit
 * button, and handles the RPC to save the result.
 */
class SignatureForm extends Component {
    static template = "portal.SignatureForm"
    static components = { NameAndSignature }

    setup() {
        this.rootRef = useRef("root");
        this.rpc = useService("rpc");

        this.csrfToken = odoo.csrf_token;
        this.state = useState({
            error: false,
            success: false,
        });
        this.signature = useState({
            name: this.props.defaultName,
            getSignatureImage: () => "",
            resetSignature: () => {},
        });
        this.nameAndSignatureProps = {
            signature: this.signature,
            fontColor: this.props.fontColor || "black",
        };
        if (this.props.signatureRatio) {
            this.nameAndSignatureProps.displaySignatureRatio = this.props.signatureRatio;
        }
        if (this.props.signatureType) {
            this.nameAndSignatureProps.signatureType = this.props.signatureType;
        }
        if (this.props.mode) {
            this.nameAndSignatureProps.mode = this.props.mode;
        }

        // Correctly set up the signature area if it is inside a modal
        onMounted(() => {
            this.rootRef.el.closest('.modal').addEventListener('shown.bs.modal', () => {
                this.signature.resetSignature();
                this.toggleSignatureFormVisibility();
            });
        });
    }

    toggleSignatureFormVisibility() {
        this.rootRef.el.classList.toggle('d-none', document.querySelector('.editor_enable'));
    }

    get sendLabel() {
        return this.props.sendLabel || _t("Accept & Sign");
    }

     /**
     * Handles click on the submit button.
     *
     * This will get the current name and signature and validate them.
     * If they are valid, they are sent to the server, and the reponse is
     * handled. If they are invalid, it will display the errors to the user.
     *
     * @returns {Promise}
     */
    async onClickSubmit() {
        const button = document.querySelector('.o_portal_sign_submit')
        const icon = button.removeChild(button.firstChild)
        const restoreBtnLoading = dom.addButtonLoadingEffect(button);

        const name = this.signature.name;
        const signature = this.signature.getSignatureImage()[1];
        const data = await this.rpc(this.props.callUrl, { name, signature });
        if (data.force_refresh) {
            restoreBtnLoading();
            button.prepend(icon)
            if (data.redirect_url) {
                redirect(data.redirect_url);
            } else {
                window.location.reload();
            }
            // do not resolve if we reload the page
            return new Promise(() => {});
        }
        this.state.error = data.error || false;
        this.state.success = !data.error && {
            message: data.message,
            redirectUrl: data.redirect_url,
            redirectMessage: data.redirect_message,
        };
    }
}

registry.category("public_components").add("portal.signature_form", SignatureForm);

```

## File: static\src\signature_form\signature_form.xml

```xml
<templates id="template" xml:space="preserve">

    <!-- Template for the widget SignatureForm. -->
    <t t-name="portal.SignatureForm">
        <div t-ref="root">
            <div t-if="state.success" class="alert alert-success" role="status">
                <span t-if="state.success.message" t-esc="state.success.message"/>
                <span t-else="">Thank You!</span>
                <a t-if="state.success.redirect_url" t-att-href="state.success.redirect_url">
                    <t t-if="state.success.redirect_message" t-esc="state.success.redirect_message"/>
                    <t t-else="">Click here to see your document.</t>
                </a>
            </div>
            <t t-else="">
                <NameAndSignature t-props="nameAndSignatureProps"/>
                <form method="POST">
                    <input type="hidden" name="csrf_token" t-att-value="csrfToken"/>
                    <div class="o_web_sign_name_and_signature"/>
                    <div class="o_portal_sign_controls my-3">
                        <div t-if="state.error" class="o_portal_sign_error_msg alert alert-danger" role="status">
                            <t t-esc="state.error"/>
                        </div>
                        <div class="text-end my-3">
                            <button type="submit" class="o_portal_sign_submit btn btn-primary" t-on-click.prevent="onClickSubmit" t-att-disabled="signature.isSignatureEmpty ? 'disabled' : ''">
                                <i class="fa fa-check me-1"/>
                                <t t-esc="sendLabel"/>
                            </button>
                        </div>
                    </div>
                </form>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\views\fields\portal_wizard_user_one2many.js

```javascript
/** @odoo-module **/

import { PortalWizardUserListController } from "../list/portal_wizard_user_list_controller";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";
import { registry } from "@web/core/registry";

export class PortalUserX2ManyField extends X2ManyField {}
PortalUserX2ManyField.components = {
    ...X2ManyField.components,
    Controller: PortalWizardUserListController,
};

export const portalUserX2ManyField = {
    ...x2ManyField,
    component: PortalUserX2ManyField,
};

registry.category("fields").add("portal_wizard_user_one2many", portalUserX2ManyField);

```

## File: static\src\views\list\portal_wizard_user_list_controller.js

```javascript
/** @odoo-module **/

import { ListController } from "@web/views/list/list_controller";

export class PortalWizardUserListController extends ListController {
    setup() {
        super.setup();
        this.isPortalActionOngoing = false;
    }

    /**
     * @override
     */
     async beforeExecuteActionButton(clickParams) {
        if (clickParams.name === 'action_refresh_modal' || this.isPortalActionOngoing) {
            return false;
        }
        this.isPortalActionOngoing = true;
        return super.beforeExecuteActionButton(clickParams);
    }
    
    /**
     * @override
     */
    async afterExecuteActionButton(clickParams) {
        this.isPortalActionOngoing = false;
    }
}

```

## File: static\src\xml\portal_chatter.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="portal.chatter_message_count">
        <t t-set="count" t-value="widget.get('message_count')"/>
        <div class="o_message_counter">
            <t t-if="count">
                <span class="fa fa-comments" />
                <span class="o_message_count"> <t t-esc="count"/></span>
                <t t-if="count == 1"> comment</t>
                <t t-else=""> comments</t>
            </t>
            <t t-else="">
                There are no comments for now.
            </t>
        </div>
    </t>

    <!--
        Widget PortalComposer (standalone)

        required many options: token, res_model, res_id, ...
    -->
    <t t-name="portal.Composer">
        <div class="o_portal_chatter_composer" t-if="widget.options['allow_composer']">
            <t t-set="discussion_url" t-value="window.encodeURI(window.location.href.split('#')[0] + '#discussion')"/>
            <t t-if="!widget.options['display_composer']">
                <h4>Leave a comment</h4>
                <p>You must be <a t-attf-href="/web/login?redirect=#{discussion_url}">logged in</a> to post a comment.</p>
            </t>
            <t t-if="widget.options['display_composer']">
                <div class="alert alert-danger mb8 d-none o_portal_chatter_composer_error" role="alert">
                    Oops! Something went wrong. Try to reload the page and log in.
                </div>
                <div class="d-flex">
                    <img alt="Avatar" class="o_avatar o_portal_chatter_avatar align-self-start me-3 rounded" t-attf-src="/web/image/res.partner/#{widget.options['partner_id']}/avatar_128"
                         t-if="!widget.options['is_user_public'] or !widget.options['token']"/>
                    <div class="flex-grow-1">
                        <div class="o_portal_chatter_composer_input">
                            <div class="o_portal_chatter_composer_body d-flex flex-nowrap align-items-start flex-grow-1 mb-4">
                                <div class="d-flex flex-column flex-grow-1 rounded-3">
                                    <div class="position-relative flex-grow-1">
                                        <textarea rows="4" name="message" class="form-control rounded-3 shadow-none" placeholder="Write a message..." style="resize:none;"/>
                                    </div>
                                    <div class="d-flex flex-row align-self-end p-2">
                                        <div class="d-flex px-1">
                                            <button class="o_portal_chatter_attachment_btn btn fa fa-paperclip border-0" type="button" title="Add attachment"/>
                                        </div>
                                        <button t-attf-data-action="/mail/chatter_post" class="o_portal_chatter_composer_btn btn btn-primary o-last rounded-3" type="submit">Send</button>
                                    </div>
                                </div>
                            </div>
                            <div class="o_portal_chatter_attachments mt-3"/>
                        </div>
                        <div class="d-none">
                            <input type="file" class="o_portal_chatter_file_input" multiple="multiple"/>
                        </div>
                    </div>
                </div>
            </t>
        </div>
    </t>

    <t t-name="portal.Chatter.Attachments">
        <div t-if="attachments.length" class="d-flex flex-grow-1 flex-wrap gap-1">
            <div t-foreach="attachments" t-as="attachment" t-key="attachment_index" class="bg-light p-2 rounded position-relative">
                <div class="o_portal_chatter_attachment text-center" t-att-data-id="attachment.id">
                    <button t-if="showDelete and attachment.state == 'pending'" class="o_portal_chatter_attachment_delete btn btn-sm btn-outline-danger" title="Delete">
                        <i class="fa fa-times"/>
                    </button>
                    <a t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank" class="d-flex flex-row">
                        <div class='oe_attachment_embedded o_image' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype"/>
                        <div class='o_portal_chatter_attachment_name align-self-center text-truncate' t-att-data-tooltip="attachment.name" data-tooltip-position="top">
                            <t t-esc='attachment.name'/>
                        </div>
                    </a>
                </div>
            </div>
        </div>
    </t>

    <!--
        Widget PortalChatter, and subtemplates
    -->

    <t t-name="portal.chatter_messages">
        <div class="o_portal_chatter_messages">
            <t t-foreach="widget.get('messages') || []" t-as="message" t-key="message_index">
                <div class="d-flex o_portal_chatter_message mb-4" t-att-id="'message-' + message.id">
                    <img class="o_avatar o_portal_chatter_avatar rounded me-3" t-att-src="message.author_avatar_url" alt="avatar"/>
                    <div class="flex-grow-1">
                        <t t-call="portal.chatter_internal_toggle" t-if="widget.options['is_user_employee']"/>

                        <div class="o_portal_chatter_message_title">
                            <h5 class='mb-1'><t t-esc="message.author_id[1]"/></h5>
                            <p class="o_portal_chatter_puslished_date">Published on <t t-esc="message.published_date_str"/></p>
                        </div>
                        <t t-out="message.body"/>

                        <div class="o_portal_chatter_attachments">
                            <t t-call="portal.Chatter.Attachments">
                                <t t-set="attachments" t-value="message.attachment_ids"/>
                            </t>
                        </div>
                    </div>
                </div>
            </t>
        </div>
    </t>

    <!-- Chatter: internal toggle widget -->
    <t t-name="portal.chatter_internal_toggle">
        <div t-if="message.is_message_subtype_note" class="float-end">
            <span class="badge rounded-pill px-3 py-2 text-bg-light" title="Internal notes are only displayed to internal users.">Internal Note</span>
        </div>
        <div t-else="" t-attf-class="float-end o_portal_chatter_js_is_internal d-flex #{message.is_internal and 'o_portal_message_internal_on' or 'o_portal_message_internal_off'}"
                t-att-data-message-id="message.id"
                t-att-data-is-internal="message.is_internal">
            <div class="form-check form-switch o_portal_chatter_visibility_on" title="Currently restricted to internal employees, click to make it available to everyone viewing this document.">
                <input class="form-check-input" type="checkbox" role="switch"/>
                <label class="form-check-label small">Public</label>
            </div>
            <div class="form-check form-switch o_portal_chatter_visibility_off" title="Currently available to everyone viewing this document, click to restrict to internal employees.">
                <input class="form-check-input" type="checkbox" role="switch" checked="true"/>
                <label class="form-check-label small">Public</label>
            </div>
        </div>
    </t>

    <t t-name="portal.pager">
        <div class="o_portal_chatter_pager">
            <t t-if="Object.keys(widget.get('pager') || {}).length > 0">
                <ul class="pagination" t-if="widget.get('pager')['pages'].length &gt; 1">
                    <li t-if="widget.get('pager')['page'] != widget.get('pager')['page_previous']" t-att-data-page="widget.get('pager')['page_previous']" class="page-item o_portal_chatter_pager_btn">
                        <a href="#" class="page-link"><i class="oi oi-chevron-left" role="img" aria-label="Previous" title="Previous"/></a>
                    </li>
                    <t t-foreach="widget.get('pager')['pages']" t-as="page" t-key="page_index">
                        <li t-att-data-page="page" t-attf-class="page-item #{page == widget.get('pager')['page'] ? 'o_portal_chatter_pager_btn active' : 'o_portal_chatter_pager_btn'}">
                            <a href="#" class="page-link"><t t-esc="page"/></a>
                        </li>
                    </t>
                    <li t-if="widget.get('pager')['page'] != widget.get('pager')['page_next']" t-att-data-page="widget.get('pager')['page_next']" class="page-item o_portal_chatter_pager_btn">
                        <a href="#" class="page-link"><i class="oi oi-chevron-right" role="img" aria-label="Next" title="Next"/></a>
                    </li>
                </ul>
            </t>
        </div>
    </t>

    <t t-name="portal.Chatter">
        <t t-set="two_columns" t-value="widget.options['two_columns']"/>
        <div t-attf-class="o_portal_chatter p-0 #{two_columns and 'row' or ''}">
            <div t-attf-class="#{two_columns and 'col-lg-5' or ''}">
                <div class="o_portal_chatter_header pb-2 text-muted">
                    <t t-call="portal.chatter_message_count"/>
                </div>
                <div class="o_portal_chatter_composer"/>
            </div>
            <div t-attf-class="#{two_columns and 'offset-lg-1 col-lg-6' or ''}">
                <t t-call="portal.chatter_messages"/>
                <div class="o_portal_chatter_footer">
                    <t t-call="portal.pager"/>
                </div>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\portal_security.xml

```xml
<templates xml:space="preserve">
    <t t-name="portal.identitycheck">
        <form string="Security Control">
            <h3><strong>Please enter your password to confirm you own this account</strong></h3>
            <br/>
            <div>
                <input class="form-control col-10 col-md-6" autocomplete="current-password"
                       name="password" type="password" required="required"/>
            </div>
            <a href="/web/reset_password/" class="btn btn-link" role="button">Forgot password?</a>
        </form>
    </t>
    <t t-name="portal.keydescription">
        <form string="Key Description">
            <h3><strong>Name your key</strong></h3>
            <p>Enter a description of and purpose for the key.</p>
            <input type="text" class="form-control col-10 col-md-6" placeholder="What's this key for?"
                name="description" required="required"/>
            <p>
                It is very important that this description be clear
                and complete, <strong>it will be the only way to
                identify the key once created</strong>.
            </p>
        </form>
    </t>
    <t t-name="portal.keyshow">
        <div>
            <h3><strong>Write down your key</strong></h3>
            <p>
                Here is your new API key, use it instead of a password for RPC access.
                Your login is still necessary for interactive usage.
            </p>
            <p><code><span t-out="key"/></code></p>
            <p class="alert alert-warning" role="alert">
                <strong>Important:</strong>
                The key cannot be retrieved later and provides <b>full access</b>
                to your user account, it is very important to store it securely.
            </p>
        </div>
    </t>
    <!-- Popup's view !-->
    <t t-name="portal.revoke_all_devices_popup_template">
        <section>
            <div>
                You are about to log out from all devices that currently have access to this user's account.
            </div><br/>
            <form action="/my/security" method="post" class="oe_reset_password_form" >
                <div class="mb-3">
                    <label for="current">Type in your password to confirm :</label>
                    <input type="password" t-attf-class="form-control form-control-sm"
                            id="current" name="password"
                            autocomplete="current-password" required="required"/>
                </div>
                <input type="hidden" name="op" value="revoke"/>
            </form>
        </section>
    </t>
</templates>

```

## File: views\mail_templates_public.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="message_document_unfollowed" inherit_id="mail.message_document_unfollowed">
        <xpath expr="//t[@t-call='mail.public_layout']" position="attributes">
            <attribute name="t-call">portal.portal_layout</attribute>
        </xpath>
    </template>
</odoo>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="frontend_layout" name="Main Frontend Layout" inherit_id="web.frontend_layout">
        <xpath expr="//div[@id='wrapwrap']" position="attributes">
            <attribute name="t-attf-class" add="#{request.env['res.lang']._lang_get_direction(request.env.lang) == 'rtl' and 'o_rtl' or ''}" separator=" "/>
            <attribute name="t-attf-class" add="#{'o_portal' if is_portal else ''}" separator=" "/>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']/header/img" position="replace">
            <t t-cache="res_company">
                <nav class="navbar navbar-expand navbar-light bg-light">
                    <div class="container">
                        <a href="/" class="navbar-brand logo">
                            <img t-att-src="'/logo.png?company=%s' % res_company.id" t-att-alt="'Logo of %s' % res_company.name" t-att-title="res_company.name"/>
                        </a>
                        <ul id="top_menu" class="nav navbar-nav ms-auto">
                            <t t-call="portal.placeholder_user_sign_in">
                                <t t-set="_item_class" t-value="'nav-item'"/>
                                <t t-set="_link_class" t-value="'nav-link'"/>
                            </t>
                            <t t-call="portal.user_dropdown">
                                <t t-set="_user_name" t-value="true"/>
                                <t t-set="_item_class" t-value="'nav-item dropdown'"/>
                                <t t-set="_link_class" t-value="'nav-link'"/>
                                <t t-set="_dropdown_menu_class" t-value="'dropdown-menu-end'"/>
                            </t>
                        </ul>
                    </div>
                </nav>
            </t>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']/main/t[@t-out='0']" position="before">
            <div t-if="o_portal_fullwidth_alert" class="container mt-3">
                <div class="alert alert-info alert-dismissible fade show d-print-none css_editable_mode_hidden">
                    <t t-out="o_portal_fullwidth_alert"/>
                </div>
            </div>
        </xpath>
        <xpath expr="//head/meta" position="after">
            <t t-if="preview_object">
                <!-- Remove seo_object to not define og and twitter tags twice when wesbite is installed -->
                <t t-set="seo_object" t-value="False"/>
                <t t-set="company" t-value="preview_object.company_id or request.env.company"/>
                <t t-set="not_uses_default_logo" t-value="company and not company.uses_default_logo"/>
                <meta property="og:title" t-att-content="preview_object.name"/>
                <meta property="og:description" t-att-content="preview_object.description.striptags() if preview_object.description else ''"/>
                <meta property="og:site_name" t-att-content="company.name if company else ''"/>
                <t t-if="not_uses_default_logo">
                    <meta property="og:image" t-attf-content="/web/binary/company_logo?company={{ company.id }}"/>
                </t>
                <meta property="og:image:width" content="300"/>
                <meta property="og:image:height" content="200"/>
                <meta name="twitter:card" content="summary_large_image"/>
                <meta property="twitter:title" t-att-content="preview_object.name"/>
                <meta property="twitter:description" t-att-content="preview_object.description.striptags() if preview_object.description else ''"/>
                <t t-if="not_uses_default_logo">
                    <meta property="twitter:image" t-attf-content="/web/binary/company_logo?company={{ company.id }}"/>
                </t>
            </t>
        </xpath>
    </template>

    <!-- Added by another template so that it can be disabled if needed -->
    <template id="footer_language_selector" inherit_id="portal.frontend_layout" name="Footer Language Selector">
        <xpath expr="//*[hasclass('o_footer_copyright_name')]" position="after">
            <t id="language_selector_call" t-call="portal.language_selector">
                <t t-set="_div_classes" t-value="(_div_classes or '') + ' dropup'"/>
            </t>
        </xpath>
    </template>

    <template id="language_selector" name="Language Selector">
        <t t-nocache="The query strings can change for the same page and the same rendering."
           t-nocache-no_text="no_text"
           t-nocache-codes="codes"
           t-nocache-_div_classes="_div_classes"
           t-nocache-_btn_class="_btn_class"
           t-nocache-_txt_class="_txt_class"
           t-nocache-_dropdown_menu_class="_dropdown_menu_class">
            <t t-set="active_lang" t-value="list(filter(lambda lg : lg[0] == lang, languages))[0]"/>
            <t t-set="language_selector_visible" t-value="len(languages) &gt; 1"/>
            <div t-attf-class="js_language_selector #{_div_classes} d-print-none" t-if="language_selector_visible">
                <button t-attf-class="btn border-0 dropdown-toggle #{_btn_class or 'btn-sm btn-outline-secondary'}" type="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
                    <span t-if="not no_text"
                            t-attf-class="align-middle #{_txt_class}"
                            t-esc="active_lang[2].split('/').pop()"/>
                    <span t-elif="codes" class="align-middle" t-esc="active_lang[1].split('_').pop(0).upper()"/>
                </button>
                <div t-attf-class="dropdown-menu #{_dropdown_menu_class}" role="menu">
                    <t t-foreach="languages" t-as="lg">
                        <a class="dropdown-item" t-att-href="url_for(request.httprequest.path + '?' + keep_query(), lang_code=lg[0])"
                        t-attf-class="dropdown-item js_change_lang #{active_lang == lg and 'active'}"
                        t-att-data-url_code="lg[1]" t-att-title="lg[2].split('/').pop()"
                        role="menuitem">
                            <span t-if="not no_text" t-esc="lg[2].split('/').pop()" t-attf-class="#{_txt_class}"/>
                            <span t-elif="codes" t-esc="lg[1].split('_').pop(0).upper()" t-attf-class="align-middle #{_txt_class}"/>
                        </a>
                    </t>
                </div>
            </div>
        </t>
    </template>

    <template id="user_dropdown" name="Portal User Dropdown">
        <t t-nocache="Each user is different regardless of the page visited."
           t-nocache-_avatar="_avatar"
           t-nocache-_icon="_icon"
           t-nocache-_icon_class="_icon_class"
           t-nocache-_icon_wrap_class="_icon_wrap_class"
           t-nocache-_no_caret="_no_caret"
           t-nocache-_user_name="_user_name"
           t-nocache-_user_name_class="_user_name_class"
           t-nocache-_item_class="_item_class"
           t-nocache-_link_class="_link_class"
           t-nocache-_dropdown_menu_class="_dropdown_menu_class">
            <t t-set="is_connected" t-value="not user_id._is_public()"/>
            <li t-if="is_connected" t-attf-class="#{_item_class} o_no_autohide_item">
                <a href="#" role="button" data-bs-toggle="dropdown" t-attf-class="#{'' if _no_caret else 'dropdown-toggle'} btn #{_link_class}">
                    <t t-if="_avatar">
                        <t t-set="avatar_source" t-value="image_data_uri(user_id.avatar_256)"/>
                        <img t-att-src="avatar_source" t-attf-class="rounded-circle o_object_fit_cover #{_avatar_class}" width="24" height="24" alt="" loading="eager"/>
                    </t>
                    <div t-if="_icon" t-attf-class="#{_icon_wrap_class}">
                        <i t-attf-class="fa fa-1x fa-fw fa-user #{_icon_class}"/>
                    </div>
                    <span t-if="_user_name" t-attf-class="#{_user_name_class}" t-esc="user_id.name[:23] + '...' if user_id.name and len(user_id.name) &gt; 25 else user_id.name"/>
                </a>
                <div t-attf-class="dropdown-menu js_usermenu #{_dropdown_menu_class}" role="menu">
                    <a groups="base.group_user" href="/web" role="menuitem" class="dropdown-item ps-3" id="o_backend_user_dropdown_link">
                        <i class="fa fa-fw fa-th me-1 small text-primary"/> Apps
                    </a>
                    <div id="o_logout_divider" class="dropdown-divider"/>
                    <a t-attf-href="/web/session/logout?redirect=/" role="menuitem" id="o_logout" class="dropdown-item ps-3">
                        <i class="fa fa-fw fa-sign-out me-1 small text-primary"/> Logout
                    </a>
                </div>
            </li>
        </t>
    </template>

    <template id="portal_breadcrumbs" name="Portal Breadcrumbs">
        <ol t-if="page_name != 'home'" class="o_portal_submenu breadcrumb mb-0 flex-grow-1 px-0">
            <li class="breadcrumb-item ms-1"><a href="/my/home" aria-label="Home" title="Home"><i class="fa fa-home"/></a></li>
            <li t-if="page_name == 'my_details'" class="breadcrumb-item">Details</li>
        </ol>
    </template>

    <template id="portal_back_in_edit_mode" name="Back to edit mode">
        <div t-ignore="true" class="text-center">
            <t t-if="custom_html" t-out="custom_html"/>
            <t t-else="">This is a preview of the customer portal.</t>
            <a t-att-href="backend_url" class="alert-link"><i class="oi oi-arrow-right me-1"/>Back to edit mode</a>
        </div>
        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
    </template>

    <template id="portal_layout" name="Portal Layout">
        <t t-call="portal.frontend_layout">
            <t t-set="is_portal" t-value="True"/>

            <div t-if="not no_breadcrumbs and not my_details and not breadcrumbs_searchbar" class="o_portal container mt-3">
                <div class="d-flex justify-content-between align-items-center">
                    <t t-call="portal.portal_breadcrumbs"/>
                    <t t-if="prev_record or next_record" t-call='portal.record_pager'/>
                </div>
            </div>
            <div id="wrap" class='o_portal_wrap'>
                <div class="container pt-3">
                    <t t-if="my_details">
                        <div class="wrapper col-12 d-flex flex-wrap justify-content-between align-items-center">
                            <h3 class="my-3">My account</h3>
                            <button class="btn py-0 d-flex align-items-center gap-2 d-lg-none ms-auto"
                            data-bs-toggle="offcanvas"
                            data-bs-target="#accountOffCanvas">
                                <img class="o_avatar rounded"
                                t-att-src="image_data_uri(user_id.partner_id.avatar_1024)" alt="Contact"/>
                            </button>
                        </div>
                        <div class="row justify-content-between">
                            <div t-attf-class="o_portal_content col-12 col-lg-8 mb-5">
                                <t t-out="0"/>
                            </div>
                            <div class="d-none d-lg-flex justify-content-end col-lg-4">
                                <t t-call="portal.side_content"/>
                            </div>
                            <div class="offcanvas offcanvas-start d-lg-none" id="accountOffCanvas">
                                <t t-call="portal.side_content">
                                    <t t-set="isOffcanvas" t-value="true"/>
                                </t>
                            </div>
                        </div>
                    </t>
                    <t t-else="">
                        <t t-out="0"/>
                    </t>
                </div>
            </div>
        </t>
    </template>

    <template id="placeholder_user_sign_in" name="User Sign In Placeholder"/>

    <template id="user_sign_in" name="User Sign In" inherit_id="portal.placeholder_user_sign_in">
        <xpath expr="." position="inside">
            <li t-nocache="Profile session and user group can change unrelated to parent caches."
                t-nocache-_item_class="_item_class"
                t-nocache-_link_class="_link_class"
                groups="base.group_public" t-attf-class="#{_item_class} o_no_autohide_item">
                <a t-attf-href="/web/login" t-attf-class="#{_link_class}">Sign in<span t-if="request.session.profile_session" class="text-danger fa fa-circle"/></a>
            </li>
        </xpath>
    </template>

    <template id="portal.user_sign_in_redirect" name="User Sign In redirect" inherit_id="portal.user_sign_in" primary="True">
        <xpath expr="//a" position="attributes">
            <attribute name="t-attf-href">/web/login?redirect={{request.httprequest.path}}</attribute>
        </xpath>
    </template>

    <template id="portal_my_home" name="My Portal">
        <t t-call="portal.portal_layout">
            <t t-set="my_details" t-value="True"/>
            <div class="o_portal_my_home">
                <div class="oe_structure" id="oe_structure_portal_my_home_1"/>
                <div class="o_portal_docs row g-2">
                    <div class="o_portal_doc_spinner spinner-border text-o-color-2 align-self-center mt-5"/>
                    <div t-if="portal_alert_category_enable" class="o_portal_category row g-2 mt-3" id="portal_alert_category"/>
                    <div t-if="portal_client_category_enable" class="o_portal_category row g-2 mt-3" id="portal_client_category"/>
                    <div t-if="portal_service_category_enable" class="o_portal_category row g-2 mt-3" id="portal_service_category"/>
                    <div t-if="portal_vendor_category_enable" class="o_portal_category row g-2 mt-3" id="portal_vendor_category"/>
                    <div class="o_portal_category row g-2 mt-3" id="portal_common_category">
                        <t t-call="portal.portal_docs_entry" t-if="False"/>
                        <t t-call="portal.portal_docs_entry">
                            <t t-set="icon" t-value="'/portal/static/src/img/portal-connection.svg'"/>
                            <t t-set="title">Connection &amp; Security</t>
                            <t t-set="text">Configure your connection parameters</t>
                            <t t-set="url" t-value="'/my/security'"/>
                            <t t-set="config_card" t-value="True"/>
                        </t>
                    </div>
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_portal_my_home_2"/>
        </t>
    </template>

    <template id="portal_docs_entry" name="My Portal Docs Entry">
        <div t-att-class="'o_portal_index_card ' +  ('' if config_card else 'd-none ') + ('col-12 order-0' if show_count else 'col-md-6 order-2')">
            <a t-att-href="url" t-att-title="title" t-attf-class="d-flex justify-content-start gap-2 gap-md-3 align-items-center py-3 pe-2 px-md-3 h-100 rounded text-decoration-none text-reset #{bg_color if bg_color else 'text-bg-light'}">
                <div t-if="icon" class="o_portal_icon align-self-start">
                    <img t-attf-src="#{icon}"/>
                </div>
                <div>
                    <h5 t-attf-class="mt-0 mb-1 #{'d-flex gap-2' if placeholder_count or count else ''}">
                        <t t-out="count"/>
                        <span t-if="placeholder_count" t-att-class="'' if show_count else 'd-none'" t-att-data-placeholder_count="placeholder_count"/>
                        <span t-out="title"/>
                    </h5>
                    <p class="m-0 text-600">
                        <t t-out="text"/>
                    </p>
                </div>
            </a>
        </div>
    </template>

    <template id="portal_table" name="My Portal Table">
        <div t-attf-class="table-responsive border-0 #{classes if classes else ''}">
            <table class="o_list_table position-relative table table-sm o_list_table_ungrouped table-striped o_portal_my_doc_table mb-0">
                <t t-out="0"/>
            </table>
        </div>
        <div t-if="pager" class="o_portal_pager d-flex justify-content-center my-3">
            <t t-call="portal.pager"/>
        </div>
    </template>

    <template id="portal_record_sidebar" name="My Portal Record Sidebar">
        <div t-attf-class="#{classes}">
            <div class="o_portal_sidebar_content d-lg-inline-block mb-4 mb-lg-0 p-3 p-lg-0" id="sidebar_content">
                <div t-if="title" class="position-relative d-flex align-items-center justify-content-md-center justify-content-lg-between flex-wrap gap-2">
                    <t t-out="title"/>
                </div>
                <t t-if="entries" t-out="entries"/>
                <div class="d-none d-lg-block mt-5 small text-center text-muted">
                    Powered by <a target="_blank" href="http://www.odoo.com?utm_source=db&amp;utm_medium=portal" title="odoo"><img src="/web/static/img/logo.png" alt="Odoo Logo" height="15"/></a>
                </div>
            </div>
        </div>
    </template>

    <!--
        The search bar is composed of 2 buttons : a "filter by" and a "sort by". Changing the 'sortby'
        criteria will keep the number of page, query params, ... Changing the 'filterby' param will
        redirect the user to the beginning of document list, keeping query parameters.

        These 2 buttons can be prepended by a advanced search input, to activate it, search_input need
        to be initialized at 'True' and the content of the t-call is the list of li elements searchable.

        :param dict searchbar_sortings : containing the sort criteria like
            {'date': {'label': _('Newest'), 'order': 'create_date desc'}}
        :param string sortby : name of the sort criteria
        :param dict searchbar_filters : containing the filter criteria like
            {'open': {'label': _('In Progress'), 'domain': [('state', '=', 'open')]}}
        :param string filterby : name of the filter criteria
        :param default_url : the base url of the pages (like '/my/orders')
        :param boolean breadcrumbs_searchbar : set to True to show breadcrumbs rather than the title
        :param boolean o_portal_search_panel : set to True to active the input search
        :param html $0 : content of the t-call
        :param title : bavbar title
        :param classes : navbar classes
    -->
    <template id="portal_searchbar" name="Portal Search Bar">
        <nav t-attf-class="navbar navbar-expand-lg flex-wrap mb-4 p-0 o_portal_navbar {{classes if classes else ''}}">
            <!--  Navbar breadcrumb or title  -->
            <t t-if="breadcrumbs_searchbar">
                <t t-call="portal.portal_breadcrumbs"/>
            </t>
            <span t-else="" class="navbar-brand mb-0 h1 me-auto" t-esc="title or 'No title'"/>

            <!--  Collapse button -->
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#o_portal_navbar_content" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle filters">
                <span class="fa fa-fw fa-bars"/>
            </button>

            <!--  Collapsable content  -->
            <div class="collapse navbar-collapse flex-wrap-reverse justify-content-end gap-3" id="o_portal_navbar_content">
                <div class="nav flex-column flex-sm-row gap-2 ms-auto p-0 mb-3 mb-lg-0 mt-1 mt-lg-0">
                    <div t-if="searchbar_sortings">
                        <span class="small me-1 navbar-text">Sort By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_sortby" data-bs-toggle="dropdown" class="btn btn-secondary dropdown-toggle">
                                <t t-esc="searchbar_sortings[sortby].get('label', 'Newest')"/>
                            </button>
                            <div class="dropdown-menu dropdown-menu-end" aria-labelledby="portal_searchbar_sortby">
                                <t t-foreach="searchbar_sortings" t-as="option">
                                    <a t-att-href="request.httprequest.path + '?' + keep_query('*', sortby=option)"
                                       t-attf-class="dropdown-item#{sortby == option and ' active' or ''}">
                                        <span t-esc="searchbar_sortings[option].get('label')"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div t-if="searchbar_filters" class="ms-lg-2">
                        <span class="small me-1 navbar-text">Filter By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_filters" data-bs-toggle="dropdown" class="btn btn-secondary dropdown-toggle">
                                <t t-esc="searchbar_filters.get(filterby,searchbar_filters.get('all')).get('label', 'All')"/>
                            </button>
                            <div class="dropdown-menu dropdown-menu-end" aria-labelledby="portal_searchbar_filters">
                                <t t-foreach="searchbar_filters" t-as="option">
                                    <a t-att-href="default_url + '?' + keep_query('*', filterby=option)"
                                       t-attf-class="dropdown-item#{filterby == option and ' active' or ''}">
                                        <span t-esc="searchbar_filters[option].get('label')"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div t-if="searchbar_groupby" class="ms-lg-2">
                        <span class="small me-1 navbar-text">Group By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_groupby" data-bs-toggle="dropdown" class="btn btn-secondary dropdown-toggle">
                                <t t-esc="searchbar_groupby[groupby].get('label', 'None')"/>
                            </button>
                            <div class="dropdown-menu dropdown-menu-end" aria-labelledby="portal_searchbar_groupby">
                                <t t-foreach="searchbar_groupby" t-as="option">
                                    <a t-att-href="default_url + '?' + keep_query('*', groupby=option)"
                                       t-attf-class="dropdown-item#{groupby == option and ' active' or ''}">
                                        <span t-esc="searchbar_groupby[option].get('label')"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </div>
                    <t t-out="0"/>
                </div>
                <form t-if="searchbar_inputs" class="o_portal_search_panel col-md-5 col-xl-4 ms-lg-2">
                    <div class="input-group w-100">
                        <button type="button" class="btn btn-secondary border-end dropdown-toggle" data-bs-toggle="dropdown"/>
                        <div class="dropdown-menu dropdown-menu-end" role="menu">
                            <t t-foreach='searchbar_inputs' t-as='input'>
                                <a t-att-href="'#' + input_value['input']"
                                    t-attf-class="dropdown-item#{search_in == input_value['input'] and ' active' or ''}">
                                    <span t-out="input_value['label']"/>
                                </a>
                            </t>
                        </div>
                        <input type="text" class="form-control" placeholder="Search" t-att-value='search' name="search"/>
                        <button class="btn btn-secondary o_wait_lazy_js" type="submit">
                            <span class="oi oi-search"/>
                        </button>
                    </div>
                </form>
            </div>
        </nav>
    </template>

    <template id="portal_record_layout" name="Portal single record layout">
        <div t-attf-class="card mt-0 rounded #{classes if classes else ''}">
            <div t-if="card_header" t-attf-class="card-header #{header_classes if header_classes else ''}">
                <t t-out="card_header"/>
            </div>
            <div t-if="card_body" t-attf-class="card-body #{body_classes if body_classes else ''}">
                <t t-out="card_body"/>
            </div>
        </div>
    </template>

    <template id="portal_contact" name="Contact">
        <div class="o_portal_contact_details mb-5">
            <h4>Your contact</h4>
            <hr class="mt-1 mb0"/>
            <h6 class="mb-1"><b t-esc="sales_user.name"/></h6>
            <div class="d-flex align-items-center mb-1">
                <div class="fa fa-envelope fa-fw me-1"></div>
                <a t-att-href="'mailto:'+sales_user.email" t-esc="sales_user.email"/>
            </div>
            <div class="d-flex flex-nowrap align-items-center mb-1">
                <div class="fa fa-phone fa-fw me-1"></div>
                <span t-esc="sales_user.phone"/>
            </div>
            <div class="d-flex flex-nowrap align-items-center mb-1">
                <div class="fa fa-map-marker fa-fw me-1"></div>
                <span t-esc="sales_user.city"/>
            </div>
        </div>
    </template>

    <template id="portal_my_details_fields">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <div t-if="error_message" class="alert alert-danger" role="alert">
            <div class="col-lg-12">
                <t t-foreach="error_message" t-as="err"><t t-esc="err"/><br /></t>
            </div>
        </div>
        <div t-if="not partner_can_edit_vat" class="col-12 d-none d-xl-block">
            <small class="form-text text-muted">
                Company name, VAT Number and country can not be changed once document(s) have been issued for your account.
                <br/>Please contact us directly for that operation.
            </small>
        </div>
        <div t-attf-class="mb-3 #{error.get('name') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="name">Name</label>
            <input type="text" name="name" t-attf-class="form-control #{error.get('name') and 'is-invalid' or ''}" t-att-value="name or partner.name" />
        </div>
        <div t-attf-class="mb-3 #{error.get('email') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="email">Email</label>
            <input type="email" name="email" t-attf-class="form-control #{error.get('email') and 'is-invalid' or ''}" t-att-value="email or partner.email" />
        </div>

        <div class="clearfix" />
        <div t-attf-class="mb-1 #{error.get('company_name') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label label-optional" for="company_name">Company Name</label>
            <!-- The <input> use "disabled" attribute to avoid sending an unauthorized value on form submit.
                 The user might not have rights to change company_name but should still be able to see it.
            -->
            <input type="text" name="company_name" t-attf-class="form-control #{error.get('company_name') and 'is-invalid' or ''}" t-att-value="company_name or partner.commercial_company_name" t-att-disabled="None if partner_can_edit_vat else '1'" />
            <small t-if="not partner_can_edit_vat" class="form-text text-muted d-block d-xl-none">
                Changing company name is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.
            </small>
        </div>
        <div t-attf-class="mb-1 #{error.get('vat') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label label-optional" for="vat">VAT Number</label>
            <!-- The <input> use "disabled" attribute to avoid sending an unauthorized value on form submit.
                 The user might not have rights to change company_name but should still be able to see it.
            -->
            <input type="text" name="vat" t-attf-class="form-control #{error.get('vat') and 'is-invalid' or ''}" t-att-value="vat or partner.vat" t-att-disabled="None if partner_can_edit_vat else '1'" />
            <small t-if="not partner_can_edit_vat" class="form-text text-muted d-block d-xl-none">Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
        </div>
        <div t-attf-class="mb-3 #{error.get('phone') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="phone">Phone</label>
            <input type="tel" name="phone" t-attf-class="form-control #{error.get('phone') and 'is-invalid' or ''}" t-att-value="phone or partner.phone" />
        </div>

        <div class="clearfix" />
        <div t-attf-class="mb-3 #{error.get('street') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="street">Street</label>
            <input type="text" name="street" t-attf-class="form-control #{error.get('street') and 'is-invalid' or ''}" t-att-value="street or partner.street"/>
        </div>
        <div t-attf-class="mb-3 #{error.get('city') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="city">City</label>
            <input type="text" name="city" t-attf-class="form-control #{error.get('city') and 'is-invalid' or ''}" t-att-value="city or partner.city" />
        </div>
        <div t-attf-class="mb-3 #{error.get('zip') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label label-optional" for="zipcode">Zip / Postal Code</label>
            <input type="text" name="zipcode" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="zipcode or partner.zip" />
        </div>
        <div t-attf-class="mb-3 #{error.get('country_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label" for="country_id">Country</label>
            <select name="country_id" t-attf-class="form-select #{error.get('country_id') and 'is-invalid' or ''}" t-att-disabled="None if partner_can_edit_vat else '1'">
                <option value="">Country...</option>
                <t t-foreach="countries or []" t-as="country">
                    <option t-att-value="country.id" t-att-selected="country.id == int(country_id) if country_id else country.id == partner.country_id.id">
                        <t t-esc="country.name" />
                    </option>
                </t>
            </select>
            <small t-if="not partner_can_edit_vat" class="form-text text-muted d-block d-xl-none">Changing the country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
        </div>
        <div t-attf-class="mb-3 #{error.get('state_id') and 'o_has_error' or ''} col-xl-6">
            <label class="col-form-label label-optional" for="state_id">State / Province</label>
            <select name="state_id" t-attf-class="form-select #{error.get('state_id') and 'is-invalid' or ''}">
                <option value="">select...</option>
                <t t-foreach="states or []" t-as="state">
                    <option t-att-value="state.id" style="display:none;" t-att-data-country_id="state.country_id.id" t-att-selected="state.id == int(state_id) if state_id else state.id == partner.state_id.id">
                        <t t-esc="state.name" />
                    </option>
                </t>
            </select>
        </div>
    </template>

    <template id="portal_my_details">
        <t t-call="portal.portal_layout">
            <t t-set="additional_title">Contact Details</t>
            <form action="/my/account" method="post">
                <div class="row o_portal_details">
                    <div class="col-lg-8">
                        <div class="row">
                            <t t-call="portal.portal_my_details_fields"/>
                            <input type="hidden" name="redirect" t-att-value="redirect"/>
                        </div>
                        <div class="clearfix text-end mb-5">
                            <a href="/my/" class="btn btn-secondary me-2">
                                Discard
                            </a>
                            <button type="submit" class="btn btn-primary float-end">
                                Save
                            </button>
                        </div>
                    </div>
                </div>
            </form>
        </t>
    </template>

    <template id="portal_my_security">
        <t t-call="portal.portal_layout"><div class="o_portal_security_body w-md-75 w-lg-50 pb-5">
            <t t-set="additional_title">Security</t>
            <t t-set="no_breadcrumbs" t-value="1"/>
            <div class="alert alert-danger" role="alert" t-if="get_error(errors)">
                <t t-esc="errors"/>
            </div>
            <div class="d-flex gap-2 my-3">
                <a href="/my/" title="Go Back" class="btn btn-light px-2"><i class="oi oi-chevron-left"/></a>
                <h3 class="my-0">Connection &amp; Security</h3>
            </div>
            <section name="portal_change_password">
                <h4>Change Password</h4>
                <t t-set="path">password</t>
                <div class="alert alert-success" role="alert" t-if="success and success.get('password')">
                    Password Updated!
                </div>
                <div class="alert alert-danger" role="alert" t-if="get_error(errors, 'password')">
                    <t t-esc="errors['password']"/>
                </div>
                <form action="/my/security" method="post" class="oe_reset_password_form">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input type="hidden" name="op" value="password"/>
                    <div class="mb-3">
                        <label for="current">Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.old') else '' }}"
                               id="current" name="old"
                               autocomplete="current-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.old')"/>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label for="new">New Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.new1') else '' }}"
                               id="new" name="new1"
                               autocomplete="new-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.new1')"/>
                        </div>
                    </div>
                    <div class="mb-3">
                        <label for="new2">Verify New Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.new2') else '' }}"
                               id="new2" name="new2"
                               autocomplete="new-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.new2')"/>
                        </div>
                    </div>
                    <button type="submit" class="btn btn-secondary">Change Password</button>
                </form>
            </section>
            <section name="portal_revoke_all_devices_popup">
            <h4>Revoke All Sessions</h4>
                <t t-set="path"/>
                    <button type="button" class="btn btn-secondary" id="portal_revoke_all_sessions_popup">
                        Log out from all devices
                    </button>
            </section>
            <section t-if="debug and allow_api_keys">
                <h4>
                Developer API Keys
                    <a href="https://www.odoo.com/documentation/17.0/developer/misc/api/external_api.html#api-keys" target="_blank">
                        <i title="Documentation" class="fa fa-fw o_button_icon fa-info-circle"></i>
                    </a>
                </h4>
                <div>
                    <table class="table o_main_table">
                        <thead>
                            <tr>
                                <th>Description</th>
                                <th>Scope</th>
                                <th>Added On</th>
                                <th/>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="request.env.user.api_key_ids" t-as="key">
                                <tr>
                                    <td><span t-field="key.name"/></td>
                                    <td><span t-field="key.scope"/></td>
                                    <td><span t-field="key.create_date"/></td>
                                    <td>
                                        <i class="fa fa-trash text-danger o_portal_remove_api_key" type="button" t-att-id="key.id"/>
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                    </table>
                </div>
                <div>
                    <button type="submit" class="btn btn-secondary o_portal_new_api_key">New API Key</button>
                </div>
            </section>
            <section name="portal_deactivate_account" groups="base.group_portal">
                <h4>Delete Account</h4>
                <t t-set="deactivate_error" t-value="get_error(errors, 'deactivate')"/>
                <button class="btn btn-secondary" data-bs-toggle="modal"
                    data-bs-target="#portal_deactivate_account_modal">
                    Delete Account
                </button>
                <div t-attf-class="modal #{'show d-block' if open_deactivate_modal else ''}"
                    id="portal_deactivate_account_modal" tabindex="-1" role="dialog">
                    <div class="modal-dialog" role="document">
                        <div class="modal-content">
                            <div class="modal-header bg-danger">
                                <h5 class="modal-title">Are you sure you want to do this?</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                            </div>
                            <form action="/my/deactivate_account" method="post" class="modal-body"
                                id="portal_deactivate_account_form">
                                <div>
                                    <div class="alert alert-danger"
                                        t-esc="get_error(errors, 'deactivate.other')"/>
                                    <p class="text-muted">
                                        Disable your account, preventing any further login.<br/>
                                        <b>
                                            <i class="fa fa-exclamation-triangle text-danger"></i>
                                            This action cannot be undone.
                                        </b>
                                    </p>
                                    <hr/>
                                    <p>1. Enter your password to confirm you own this account</p>
                                    <input name="password" type="password" required="1"
                                        t-attf-class="form-control #{'is-invalid' if deactivate_error == 'password' else ''}"
                                        placeholder="Password"/>
                                    <div t-if="deactivate_error == 'password'" class="invalid-feedback">
                                        Wrong password.
                                    </div>
                                    <hr/>
                                    <p>
                                        2. Confirm you want to delete your account by
                                        copying down your login (<t t-esc="env.user.login"/>).
                                    </p>
                                    <input name="validation" type="text" required="1"
                                        t-attf-class="form-control #{'is-invalid' if deactivate_error == 'validation' else ''}"/>
                                    <div t-if="deactivate_error == 'validation'" class="invalid-feedback">
                                        You should enter "<t t-esc="env.user.login"/>" to validate your action.
                                    </div>
                                    <div class="d-flex flex-row align-items-center">
                                        <input type="checkbox" name="request_blacklist" id="request_blacklist" checked="1"/>
                                        <label for="request_blacklist" class="ms-2 mw-100 fw-normal mt-3">
                                            Put my email and phone in a block list to make sure I'm never contacted again
                                        </label>
                                    </div>
                                </div>
                                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                            </form>
                            <div class="modal-footer justify-content-start">
                                <input type="submit" class="btn btn-danger" form="portal_deactivate_account_form"
                                    value="Delete Account"/>
                                <button type="button" class="btn" data-bs-dismiss="modal">
                                    Cancel
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </section>
        </div></t>
    </template>

    <template id="record_pager" name="Portal Record Pager">
        <t t-if='prev_record or next_record'>
            <div class="record_pager btn-group" role="group">
                <a role="button" t-att-class="'btn btn-light %s' % ('disabled' if not prev_record else '')" t-att-href="prev_record or '#'" ><i class="oi oi-chevron-left" role="img" aria-label="Previous" title="Previous"></i></a>
                <a role="button" t-att-class="'btn btn-light %s' % ('disabled' if not next_record else '')" t-att-href="next_record or '#'" ><i class="oi oi-chevron-right" role="img" aria-label="Next" title="Next"></i></a>
            </div>
        </t>
    </template>

    <template id="pager" name="Pager">
        <ul t-if="pager['page_count'] > 1" t-attf-class="#{ classname or '' } pagination m-0 #{_classes}" t-att-style="style or None">
            <li t-attf-class="page-item #{'disabled' if pager['page']['num'] == 1 else ''}">
                <a t-att-href=" pager['page_previous']['url'] if pager['page']['num'] != 1 else None" t-attf-class="page-link #{extraLinkClass}">
                    <span class="fa fa-chevron-left" role="img" aria-label="Previous" title="Previous"/>
                </a>
            </li>
            <t t-foreach="pager['pages']" t-as="page">
                <li t-attf-class="page-item #{'active' if page['num'] == pager['page']['num'] else ''}"> <a t-att-href="page['url']" t-attf-class="page-link #{extraLinkClass}" t-out="page['num']"/></li>
            </t>
            <li t-attf-class="page-item #{'disabled' if pager['page']['num'] == pager['page_count'] else ''}">
                <a t-att-href="pager['page_next']['url'] if pager['page']['num'] != pager['page_count'] else None" t-attf-class="page-link #{extraLinkClass}">
                    <span class="fa fa-chevron-right" role="img" aria-label="Next" title="Next"/>
                </a>
            </li>
        </ul>
    </template>

    <template id="my_account_link" name="Link to frontend portal" inherit_id="portal.user_dropdown">
        <xpath expr="//*[@id='o_logout_divider']" position="before">
            <a href="/my/home" role="menuitem" class="dropdown-item ps-3">
                <i class="fa fa-fw fa-id-card-o me-1 small text-primary"/> My Account
            </a>
        </xpath>
    </template>

    <!--
        Generic chatter template for the frontend
        This template provide the container of the chatter. The rest is done in js.
        To use this template, you need to call it after setting the following variable in your template or in your controller:
            :object browserecord : the mail_thread object
            :message_per_page int (optional): number of message per chatter page
            :token string (optional): if you want your chatter to be available for non-logged user,
                     you can use a token to verify the identity of the user;
                     the message will be posted with the identity of the partner_id of the object
            :hash : signed token with the partner_id using `_sign_token` method (on mail.thread)
            :pid : identifier of the partner signing the token

        NOTE: for standard portal flows, _get_page_view_values should be used to properly setup the template parameters
    -->
    <template id="message_thread">
        <div id="discussion" data-anchor="true"
            class="d-print-none o_portal_chatter o_not_editable p-0"
            t-att-data-token="token"
            t-att-data-res_model="object._name"
            t-att-data-pid="pid"
            t-att-data-hash="hash"
            t-att-data-res_id="object.id"
            t-att-data-pager_step="message_per_page or 10"
            t-att-data-allow_composer="'0' if disable_composer else '1'"
            t-att-data-two_columns="'true' if two_columns else 'false'">
        </div>
    </template>

    <!--
        Snippet to request user signature in the portal. The feature comes with
        the JS file `portal_signature.js`.

        The following variable has to be set:
            - {string} call_url: url where to send the name and signature by RPC
                The url should contain a query string if additional parameters
                have to be sent, such as an access token.

        The following variables are optional:
            - {string} default_name: the default name to display
            - {string} mode: 'draw', 'auto', or 'load'
            - {string} send_label: label of the send button
            - {number} signature_ratio: ratio of the signature area
            - {string} signature_type: 'signature' or 'initial'

        For default values and more information, see components SignatureForm and NameAndSignature.
    -->
    <template id="portal.signature_form" name="Ask Signature">
        <t t-set="signature_form_props" t-value="{
            'callUrl': call_url,
            'defaultName': default_name,
            'mode': mode,
            'sendLabel': send_label,
            'signatureRatio': signature_ratio,
            'signatureType': signature_type,
            'fontColor': font_color
        }"/>
        <owl-component name="portal.signature_form" t-att-props="json.dumps(signature_form_props)"/>
    </template>

    <template id="portal_sidebar" name="Sidebar">
        <t t-call="portal.portal_layout">
            <body data-bs-spy="scroll" data-target=".navspy" data-offset="50">
                <div class="container o_portal_sidebar"></div>
                <div class="oe_structure mb32" id="oe_structure_portal_sidebar_1"/>
            </body>
        </t>
    </template>

    <template id="side_content">
        <div t-if="isOffcanvas" class="offcanvas-header justify-content-end">
            <button type="button" class="btn-close text-reset" data-bs-dismiss="offcanvas" aria-label="Close"/>
        </div>
        <div t-attf-class="{{'offcanvas-body' if isOffcanvas else 'mt-3'}}">
            <div class="d-flex justify-content-start align-items-center gap-3 mb-4">
                <img class="o_portal_contact_img rounded o_object_fit_cover" t-att-src="image_data_uri(user_id.partner_id.avatar_128)" alt="Contact" width="50"/>
                <div class="d-flex flex-column justify-content-center">
                    <h5 class="mb-0" t-out="user_id.name"/>
                    <p class="mb-0 text-muted" t-out="user_id.company_name"/>
                </div>
            </div>
            <div class="o_portal_my_details">
                <div t-field="user_id.partner_id" t-options='{"widget": "contact", "fields": ["email", "phone", "address"]}'/>
            </div>
            <a role="button" href="/my/account" class="btn btn-link p-0 mt-3"><i class="fa fa-pencil"/> Edit information</a>
            <hr t-if="sales_user"/>
            <div class="o_my_contact" t-if="sales_user">
                <t t-call="portal.portal_contact"/>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.portal</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="40"/>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name=%(base.action_apikeys_admin)d]//ancestor::setting" position="inside">
                    <div groups="base.group_no_one">
                        <div class="mt16 content-group">
                            <field name="portal_allow_api_keys"/>
                            <label string="Customers can generate API Keys" for="portal_allow_api_keys"/>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: wizard\portal_share.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models, _


class PortalShare(models.TransientModel):
    _name = 'portal.share'
    _description = 'Portal Sharing'

    @api.model
    def default_get(self, fields):
        result = super(PortalShare, self).default_get(fields)
        result['res_model'] = self._context.get('active_model', False)
        result['res_id'] = self._context.get('active_id', False)
        if result['res_model'] and result['res_id']:
            record = self.env[result['res_model']].browse(result['res_id'])
            result['share_link'] = record.get_base_url() + record._get_share_url(redirect=True)
        return result

    @api.model
    def _selection_target_model(self):
        return [(model.model, model.name) for model in self.env['ir.model'].sudo().search([])]

    res_model = fields.Char('Related Document Model', required=True)
    res_id = fields.Integer('Related Document ID', required=True)
    resource_ref = fields.Reference('_selection_target_model', 'Related Document', compute='_compute_resource_ref')
    partner_ids = fields.Many2many('res.partner', string="Recipients", required=True)
    note = fields.Text(help="Add extra content to display in the email")
    share_link = fields.Char(string="Link", compute='_compute_share_link')
    access_warning = fields.Text("Access warning", compute="_compute_access_warning")

    @api.depends('res_model', 'res_id')
    def _compute_resource_ref(self):
        for wizard in self:
            if wizard.res_model and wizard.res_model in self.env:
                wizard.resource_ref = '%s,%s' % (wizard.res_model, wizard.res_id or 0)
            else:
                wizard.resource_ref = None

    @api.depends('res_model', 'res_id')
    def _compute_share_link(self):
        for rec in self:
            rec.share_link = False
            if rec.res_model:
                res_model = self.env[rec.res_model]
                if isinstance(res_model, self.pool['portal.mixin']) and rec.res_id:
                    record = res_model.browse(rec.res_id)
                    rec.share_link = record.get_base_url() + record._get_share_url(redirect=True)

    @api.depends('res_model', 'res_id')
    def _compute_access_warning(self):
        for rec in self:
            rec.access_warning = False
            if rec.res_model:
                res_model = self.env[rec.res_model]
                if isinstance(res_model, self.pool['portal.mixin']) and rec.res_id:
                    record = res_model.browse(rec.res_id)
                    rec.access_warning = record.access_warning

    def _send_public_link(self, partners=None):
        if partners is None:
            partners = self.partner_ids
        for partner in partners:
            share_link = self.resource_ref.get_base_url() + self.resource_ref._get_share_url(redirect=True, pid=partner.id)
            saved_lang = self.env.lang
            self = self.with_context(lang=partner.lang)
            self.resource_ref.message_post_with_source(
                'portal.portal_share_template',
                render_values={'partner': partner, 'note': self.note, 'record': self.resource_ref,
                        'share_link': share_link,
                        'model_description': self.env['ir.model']._get(self.resource_ref._name).display_name.lower()},
                subject=_("Invitation to access %s", self.resource_ref.display_name),
                subtype_xmlid='mail.mt_note',
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=partner.ids)
            self = self.with_context(lang=saved_lang)

    def _send_signup_link(self, partners=None):
        if partners is None:
            partners = self.partner_ids.filtered(lambda partner: not partner.user_ids)
        for partner in partners:
            #  prepare partner for signup and send singup url with redirect url
            partner.signup_get_auth_param()
            share_link = partner._get_signup_url_for_action(action='/mail/view', res_id=self.res_id, model=self.res_model)[partner.id]
            saved_lang = self.env.lang
            self = self.with_context(lang=partner.lang)
            self.resource_ref.message_post_with_source(
                'portal.portal_share_template',
                render_values={'partner': partner, 'note': self.note, 'record': self.resource_ref,
                        'share_link': share_link,
                        'model_description': self.env['ir.model']._get(self.resource_ref._name).display_name.lower()},
                subject=_("Invitation to access %s", self.resource_ref.display_name),
                subtype_xmlid='mail.mt_note',
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=partner.ids)
            self = self.with_context(lang=saved_lang)

    def action_send_mail(self):
        signup_enabled = self.env['ir.config_parameter'].sudo().get_param('auth_signup.invitation_scope') == 'b2c'

        if getattr(self.resource_ref, 'access_token', False) or not signup_enabled:
            partner_ids = self.partner_ids
        else:
            partner_ids = self.partner_ids.filtered(lambda x: x.user_ids)
        # if partner already user or record has access token send common link in batch to all user
        self._send_public_link(partner_ids)
        # when partner not user send individual mail with signup token
        self._send_signup_link(self.partner_ids - partner_ids)

        # subscribe all recipients so that they receive future communication (better than
        # using autofollow as more precise)
        self.resource_ref.message_subscribe(partner_ids=self.partner_ids.ids)

        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\portal_share_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="portal_share_wizard" model="ir.ui.view">
        <field name="name">portal.share.wizard</field>
        <field name="model">portal.share</field>
        <field name="arch" type="xml">
            <form string="Share Document">
                <p class="alert alert-warning" invisible="access_warning == ''" role="alert"><field name="access_warning"/></p>
                <group name="share_link">
                    <field name="res_model" invisible="1"/>
                    <field name="res_id" invisible="1"/>
                    <field name="share_link" widget="CopyClipboardChar" options="{'string': 'Copy Link'}"/>
                    <field name="partner_ids" widget="many2many_tags_email" placeholder="Add contacts to share the document..."/>
                    <field name="note" placeholder="Add a note"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_mail" invisible="access_warning != ''" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-default" special="cancel" data-hotkey="x" />
                </footer>
            </form>
        </field>
    </record>

    <record id="portal_share_action" model="ir.actions.act_window">
        <field name="name">Share Document</field>
        <field name="res_model">portal.share</field>
        <field name="binding_model_id" ref="model_portal_share"/>
        <field name="view_mode">form</field>
        <field name="context">{'dialog_size': 'medium'}</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\portal_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo.tools.translate import _
from odoo.tools import email_normalize
from odoo.exceptions import UserError

from odoo import api, fields, models, Command


_logger = logging.getLogger(__name__)


class PortalWizard(models.TransientModel):
    """
        A wizard to manage the creation/removal of portal users.
    """

    _name = 'portal.wizard'
    _description = 'Grant Portal Access'

    def _default_partner_ids(self):
        partner_ids = self.env.context.get('default_partner_ids', []) or self.env.context.get('active_ids', [])
        contact_ids = set()
        for partner in self.env['res.partner'].sudo().browse(partner_ids):
            contact_partners = partner.child_ids.filtered(lambda p: p.type in ('contact', 'other')) | partner
            contact_ids |= set(contact_partners.ids)

        return [Command.link(contact_id) for contact_id in contact_ids]

    partner_ids = fields.Many2many('res.partner', string='Partners', default=_default_partner_ids)
    user_ids = fields.One2many('portal.wizard.user', 'wizard_id', string='Users', compute='_compute_user_ids', store=True, readonly=False)
    welcome_message = fields.Text('Invitation Message', help="This text is included in the email sent to new users of the portal.")

    @api.depends('partner_ids')
    def _compute_user_ids(self):
        for portal_wizard in self:
            portal_wizard.user_ids = [
                Command.create({
                    'partner_id': partner.id,
                    'email': partner.email,
                })
                for partner in portal_wizard.partner_ids
            ]

    @api.model
    def action_open_wizard(self):
        """Create a "portal.wizard" and open the form view.

        We need a server action for that because the one2many "user_ids" records need to
        exist to be able to execute an a button action on it. If they have no ID, the
        buttons will be disabled and we won't be able to click on them.

        That's why we need a server action, to create the records and then open the form
        view on them.
        """
        portal_wizard = self.create({})
        return portal_wizard._action_open_modal()

    def _action_open_modal(self):
        """Allow to keep the wizard modal open after executing the action."""
        return {
            'name': _('Portal Access Management'),
            'type': 'ir.actions.act_window',
            'res_model': 'portal.wizard',
            'view_mode': 'form',
            'res_id': self.id,
            'target': 'new',
        }


class PortalWizardUser(models.TransientModel):
    """
        A model to configure users in the portal wizard.
    """

    _name = 'portal.wizard.user'
    _description = 'Portal User Config'

    wizard_id = fields.Many2one('portal.wizard', string='Wizard', required=True, ondelete='cascade')
    partner_id = fields.Many2one('res.partner', string='Contact', required=True, readonly=True, ondelete='cascade')
    email = fields.Char('Email')

    user_id = fields.Many2one('res.users', string='User', compute='_compute_user_id', compute_sudo=True)
    login_date = fields.Datetime(related='user_id.login_date', string='Latest Authentication')
    is_portal = fields.Boolean('Is Portal', compute='_compute_group_details')
    is_internal = fields.Boolean('Is Internal', compute='_compute_group_details')
    email_state = fields.Selection([
        ('ok', 'Valid'),
        ('ko', 'Invalid'),
        ('exist', 'Already Registered')],
        string='Status', compute='_compute_email_state', default='ok')

    @api.depends('email')
    def _compute_email_state(self):
        portal_users_with_email = self.filtered(lambda user: email_normalize(user.email))
        (self - portal_users_with_email).email_state = 'ko'

        existing_users = self.env['res.users'].with_context(active_test=False).sudo().search_read(
            self._get_similar_users_domain(portal_users_with_email),
            self._get_similar_users_fields()
        )
        for portal_user in portal_users_with_email:
            if next((user for user in existing_users if self._is_portal_similar_than_user(user, portal_user)), None):
                portal_user.email_state = 'exist'
            else:
                portal_user.email_state = 'ok'

    @api.depends('partner_id')
    def _compute_user_id(self):
        for portal_wizard_user in self:
            user = portal_wizard_user.partner_id.with_context(active_test=False).user_ids
            portal_wizard_user.user_id = user[0] if user else False

    @api.depends('user_id', 'user_id.groups_id')
    def _compute_group_details(self):
        for portal_wizard_user in self:
            user = portal_wizard_user.user_id

            if user and user._is_internal():
                portal_wizard_user.is_internal = True
                portal_wizard_user.is_portal = False
            elif user and user.has_group('base.group_portal'):
                portal_wizard_user.is_internal = False
                portal_wizard_user.is_portal = True
            else:
                portal_wizard_user.is_internal = False
                portal_wizard_user.is_portal = False

    def action_grant_access(self):
        """Grant the portal access to the partner.

        If the partner has no linked user, we will create a new one in the same company
        as the partner (or in the current company if not set).

        An invitation email will be sent to the partner.
        """
        self.ensure_one()
        self._assert_user_email_uniqueness()

        if self.is_portal or self.is_internal:
            raise UserError(_('The partner "%s" already has the portal access.', self.partner_id.name))

        group_portal = self.env.ref('base.group_portal')
        group_public = self.env.ref('base.group_public')

        self._update_partner_email()
        user_sudo = self.user_id.sudo()

        if not user_sudo:
            # create a user if necessary and make sure it is in the portal group
            company = self.partner_id.company_id or self.env.company
            user_sudo = self.sudo().with_company(company.id)._create_user()

        if not user_sudo.active or not self.is_portal:
            user_sudo.write({'active': True, 'groups_id': [(4, group_portal.id), (3, group_public.id)]})
            # prepare for the signup process
            user_sudo.partner_id.signup_prepare()

        self.with_context(active_test=True)._send_email()

        return self.action_refresh_modal()

    def action_revoke_access(self):
        """Remove the user of the partner from the portal group.

        If the user was only in the portal group, we archive it.
        """
        self.ensure_one()
        if not self.is_portal:
            raise UserError(_('The partner "%s" has no portal access or is internal.', self.partner_id.name))

        group_portal = self.env.ref('base.group_portal')
        group_public = self.env.ref('base.group_public')

        self._update_partner_email()

        # Remove the sign up token, so it can not be used
        self.partner_id.sudo().signup_token = False

        user_sudo = self.user_id.sudo()

        # remove the user from the portal group
        if user_sudo and user_sudo.has_group('base.group_portal'):
            user_sudo.write({'groups_id': [(3, group_portal.id), (4, group_public.id)], 'active': False})

        return self.action_refresh_modal()

    def action_invite_again(self):
        """Re-send the invitation email to the partner."""
        self.ensure_one()
        self._assert_user_email_uniqueness()

        if not self.is_portal:
            raise UserError(_('You should first grant the portal access to the partner "%s".', self.partner_id.name))

        self._update_partner_email()
        self.with_context(active_test=True)._send_email()

        return self.action_refresh_modal()

    def action_refresh_modal(self):
        """Refresh the portal wizard modal and keep it open. Used as fallback action of email state icon buttons,
        required as they must be non-disabled buttons to fire mouse events to show tooltips on email state."""
        return self.wizard_id._action_open_modal()

    def _create_user(self):
        """ create a new user for wizard_user.partner_id
            :returns record of res.users
        """
        return self.env['res.users'].with_context(no_reset_password=True)._create_user_from_template({
            'email': email_normalize(self.email),
            'login': email_normalize(self.email),
            'partner_id': self.partner_id.id,
            'company_id': self.env.company.id,
            'company_ids': [(6, 0, self.env.company.ids)],
        })

    def _send_email(self):
        """ send notification email to a new portal user """
        self.ensure_one()

        # determine subject and body in the portal user's language
        template = self.env.ref('portal.mail_template_data_portal_welcome')
        if not template:
            raise UserError(_('The template "Portal: new user" not found for sending email to the portal user.'))

        lang = self.user_id.sudo().lang
        partner = self.user_id.sudo().partner_id

        portal_url = partner.with_context(signup_force_type_in_url='', lang=lang)._get_signup_url_for_action()[partner.id]
        partner.signup_prepare()

        template.with_context(dbname=self._cr.dbname, portal_url=portal_url, lang=lang).send_mail(self.id, force_send=True)

        return True

    def _assert_user_email_uniqueness(self):
        """Check that the email can be used to create a new user."""
        self.ensure_one()
        if self.email_state == 'ko':
            raise UserError(_('The contact "%s" does not have a valid email.', self.partner_id.name))
        if self.email_state == 'exist':
            raise UserError(_('The contact "%s" has the same email as an existing user', self.partner_id.name))

    def _update_partner_email(self):
        """Update partner email on portal action, if a new one was introduced and is valid."""
        email_normalized = email_normalize(self.email)
        if self.email_state == 'ok' and email_normalize(self.partner_id.email) != email_normalized:
            self.partner_id.write({'email': email_normalized})

    def _get_similar_users_domain(self, portal_users_with_email):
        """ Returns the domain needed to find the users that have the same email
        as portal users.
        :param portal_users_with_email: portal users that have an email address.
        """
        normalized_emails = [email_normalize(portal_user.email) for portal_user in portal_users_with_email]
        return [('login', 'in', normalized_emails)]

    def _get_similar_users_fields(self):
        """ Returns a list of field elements to extract from users.
        """
        return ['id', 'login']

    def _is_portal_similar_than_user(self, user, portal_user):
        """ Checks if the credentials of a portal user and a user are the same
        (users are distinct and their emails are similar).
        """
        return user['login'] == email_normalize(portal_user.email) and user['id'] != portal_user.user_id.id

```

## File: wizard\portal_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- wizard action on res.partner -->
        <record id="partner_wizard_action_create_and_open" model="ir.actions.server">
            <field name="name">Grant portal access</field>
            <field name="model_id" ref="portal.model_portal_wizard"/>
            <field name="binding_model_id" ref="base.model_res_partner"/>
            <field name="state">code</field>
            <field name="code">action = model.action_open_wizard()</field>
        </record>

        <record id="partner_wizard_action" model="ir.actions.act_window">
            <field name="name">Grant portal access</field>
            <field name="res_model">portal.wizard</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="binding_model_id" eval="False"/>
        </record>

        <!-- wizard view -->
        <record id="wizard_view" model="ir.ui.view">
            <field name="name">Grant portal access</field>
            <field name="model">portal.wizard</field>
            <field name="arch" type="xml">
                <form string="Portal Access Management">
                    <div class="mb-3">
                        Select which contacts should belong to the portal in the list below.
                        The email address of each selected contact must be valid and unique.
                        If necessary, you can fix any contact's email address directly in the list.
                    </div>
                    <field name="welcome_message"
                        placeholder="This text is included at the end of the email sent to new portal users."
                        class="mb-3"/>
                    <field name="user_ids" class="o_portal_wizard_user_one2many" widget="portal_wizard_user_one2many">
                        <tree string="Contacts" editable="bottom" create="false" delete="false">
                            <field name="partner_id" force_save="1"/>
                            <field name="email" readonly="is_internal"/>
                            <field name="email_state" column_invisible="True"/>
                            <button name="action_refresh_modal" type="object" icon="fa-check text-success"
                                invisible="email_state != 'ok'" title="Valid Email Address"/>
                            <button name="action_refresh_modal" type="object" icon="fa-times text-danger"
                                invisible="email_state != 'ko'" title="Invalid Email Address"/>
                            <button name="action_refresh_modal" type="object" icon="fa-user-times text-danger"
                                invisible="email_state != 'exist'" title="Email Address already taken by another user"/>
                            <field name="login_date"/>
                            <field name="is_portal" column_invisible="True"/>
                            <field name="is_internal" column_invisible="True"/>
                            <button string="Grant Access" name="action_grant_access" type="object" class="btn-secondary"
                                invisible="is_portal or is_internal or email_state != 'ok'"/>
                            <button string="Revoke Access" name="action_revoke_access" type="object" class="btn-secondary"
                                invisible="not is_portal or is_internal"/>
                            <button string="Re-Invite" name="action_invite_again" type="object" class="btn-secondary"
                                invisible="not is_portal or is_internal or email_state != 'ok'"/>
                            <button string="Internal User" invisible="not is_internal"
                                disabled="True" title="This partner is linked to an internal User and already has access to the Portal."/>
                        </tree>
                    </field>
                    <footer>
                        <button string="Close" class="btn-primary" special="save" data-hotkey="v" />
                    </footer>
                </form>
            </field>
        </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal_share
from . import portal_wizard

```

