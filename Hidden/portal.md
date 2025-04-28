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
    'sequence': '9000',
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
        'web.assets_frontend': [
            'portal/static/src/scss/bootstrap.extend.scss',
            'portal/static/src/scss/portal.scss',
            'portal/static/src/js/portal.js',
            'portal/static/src/js/portal_chatter.js',
            'portal/static/src/js/portal_composer.js',
            'portal/static/src/js/portal_signature.js',
            'portal/static/src/js/portal_sidebar.js',
        ],
        'web.assets_tests': [
            'portal/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'portal/static/src/xml/portal_chatter.xml',
            'portal/static/src/xml/portal_signature.xml',
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
        return consteq(_hash, record._sign_token(pid))
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

    @http.route(['/mail/chatter_post'], type='json', methods=['POST'], auth='public', website=True)
    def portal_chatter_post(self, res_model, res_id, message, attachment_ids=None, attachment_tokens=None, **kw):
        """Create a new `mail.message` with the given `message` and/or `attachment_ids` and return new message values.

        The message will be associated to the record `res_id` of the model
        `res_model`. The user must have access rights on this target document or
        must provide valid identifiers through `kw`. See `_message_post_helper`.
        """
        res_id = int(res_id)

        self._portal_post_check_attachments(attachment_ids or [], attachment_tokens or [])

        if message or attachment_ids:
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
                # sudo write the attachment to bypass the read access
                # verification in mail message
                record = request.env[res_model].browse(res_id)
                message_values = {'res_id': res_id, 'model': res_model}
                attachments = record._message_post_process_attachments([], attachment_ids, message_values)

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
                'is_user_employee': request.env.user.has_group('base.group_user'),
                'is_user_publisher': request.env.user.has_group('website.group_website_publisher'),
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
            'messages': Message.search(domain, limit=limit, offset=offset).portal_message_format(),
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
        a valid access token, redirect him to the front-end view.
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
                    record_action = record_sudo.with_context(force_website=True).get_access_action()
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

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import functools
import json
import logging
import math
import re

from werkzeug import urls

from odoo import fields as odoo_fields, http, tools, _, SUPERUSER_ID
from odoo.exceptions import ValidationError, AccessError, MissingError, UserError, AccessDenied
from odoo.http import content_disposition, Controller, request, route
from odoo.tools import consteq

# --------------------------------------------------
# Misc tools
# --------------------------------------------------

_logger = logging.getLogger(__name__)
def pager(url, total, page=1, step=30, scope=5, url_args=None):
    """ Generate a dict with required value to render `website.pager` template. This method compute
        url, page range to display, ... in the pager.
        :param url : base url of the page link
        :param total : number total of item to be splitted into pages
        :param page : current page
        :param step : item per page
        :param scope : number of page to display on pager
        :param url_args : additionnal parameters to add as query params to page url
        :type url_args : dict
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
        sales_user = False
        partner = request.env.user.partner_id
        if partner.user_id and not partner.user_id._is_public():
            sales_user = partner.user_id

        return {
            'sales_user': sales_user,
            'page_name': 'home',
        }

    def _prepare_home_portal_values(self, counters):
        """Values for /my & /my/home routes template rendering.

        Includes the record count for the displayed badges.
        where 'coutners' is the list of the displayed badges
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
            error, error_message = self.details_form_validate(post)
            values.update({'error': error, 'error_message': error_message})
            values.update(post)
            if not error:
                values = {key: post[key] for key in self.MANDATORY_BILLING_FIELDS}
                values.update({key: post[key] for key in self.OPTIONAL_BILLING_FIELDS if key in post})
                for field in set(['country_id', 'state_id']) & set(values.keys()):
                    try:
                        values[field] = int(values[field])
                    except:
                        values[field] = False
                values.update({'zip': values.pop('zipcode', '')})
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
            'redirect': redirect,
            'page_name': 'my_details',
        })

        response = request.render("portal.portal_my_details", values)
        response.headers['X-Frame-Options'] = 'DENY'
        return response

    @route('/my/security', type='http', auth='user', website=True, methods=['GET', 'POST'])
    def security(self, **post):
        values = self._prepare_portal_layout_values()
        values['get_error'] = get_error
        values['allow_api_keys'] = bool(request.env['ir.config_parameter'].sudo().get_param('portal.allow_api_keys'))

        if request.httprequest.method == 'POST':
            values.update(self._update_password(
                post['old'].strip(),
                post['new1'].strip(),
                post['new2'].strip()
            ))

        return request.render('portal.portal_my_security', values, headers={
            'X-Frame-Options': 'DENY'
        })

    def _update_password(self, old, new1, new2):
        for k, v in [('old', old), ('new1', new1), ('new2', new2)]:
            if not v:
                return {'errors': {'password': {k: _("You cannot leave any password empty.")}}}

        if new1 != new2:
            return {'errors': {'password': {'new2': _("The new password and its confirmation must be identical.")}}}

        try:
            request.env['res.users'].change_password(old, new1)
        except UserError as e:
            return {'errors': {'password': e.name}}
        except AccessDenied as e:
            msg = e.args[0]
            if msg == AccessDenied().args[0]:
                msg = _('The old password you provided is incorrect, your password was not changed.')
            return {'errors': {'password': {'old': msg}}}

        # update session token so the user does not get logged out (cache cleared by passwd change)
        new_token = request.env.user._compute_session_token(request.session.sid)
        request.session.session_token = new_token

        return {'success': {'password': True}}

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
        if not request.env.user.has_group('base.group_user'):
            IrAttachment = IrAttachment.sudo().with_context(binary_field_real_user=IrAttachment.env.user)

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

    def details_form_validate(self, data):
        error = dict()
        error_message = []

        # Validation
        for field_name in self.MANDATORY_BILLING_FIELDS:
            if not data.get(field_name):
                error[field_name] = 'missing'

        # email validation
        if data.get('email') and not tools.single_email_re.match(data.get('email')):
            error["email"] = 'error'
            error_message.append(_('Invalid Email! Please enter a valid email address.'))

        # vat validation
        partner = request.env.user.partner_id
        if data.get("vat") and partner and partner.vat != data.get("vat"):
            if partner.can_edit_vat():
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
                    except ValidationError:
                        error["vat"] = 'error'
            else:
                error_message.append(_('Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.'))

        # error message for empty required fields
        if [err for err in error.values() if err == 'missing']:
            error_message.append(_('Some required fields are empty.'))

        unknown = [k for k in data if k not in self.MANDATORY_BILLING_FIELDS + self.OPTIONAL_BILLING_FIELDS]
        if unknown:
            error['common'] = 'Unknown field'
            error_message.append("Unknown field '%s'" % ','.join(unknown))

        return error, error_message

    def _document_check_access(self, model_name, document_id, access_token=None):
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

        report_sudo = request.env.ref(report_ref).with_user(SUPERUSER_ID)

        if not isinstance(report_sudo, request.env.registry['ir.actions.report']):
            raise UserError(_("%s is not the reference of a report", report_ref))

        if hasattr(model, 'company_id'):
            report_sudo = report_sudo.with_company(model.company_id)

        method_name = '_render_qweb_%s' % (report_type)
        report = getattr(report_sudo, method_name)([model.id], data={'report_type': report_type})[0]
        reporthttpheaders = [
            ('Content-Type', 'application/pdf' if report_type == 'pdf' else 'text/html'),
            ('Content-Length', len(report)),
        ]
        if report_type == 'pdf' and download:
            filename = "%s.pdf" % (re.sub(r'\W+', '-', model._get_report_base_filename()))
            reporthttpheaders.append(('Content-Disposition', content_disposition(filename)))
        return request.make_response(report, headers=reporthttpheaders)

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
from odoo.addons.web.controllers import main
from odoo.http import request


class Home(main.Home):

    @http.route()
    def index(self, *args, **kw):
        if request.session.uid and not request.env['res.users'].sudo().browse(request.session.uid).has_group('base.group_user'):
            return request.redirect_query('/my', query=request.params)
        return super(Home, self).index(*args, **kw)

    def _login_redirect(self, uid, redirect=None):
        if not redirect and not request.env['res.users'].sudo().browse(uid).has_group('base.group_user'):
            redirect = '/my'
        return super(Home, self)._login_redirect(uid, redirect=redirect)

    @http.route('/web', type='http', auth="none")
    def web_client(self, s_action=None, **kw):
        if request.session.uid and not request.env['res.users'].sudo().browse(request.session.uid).has_group('base.group_user'):
            return request.redirect_query('/my', query=request.params)
        return super(Home, self).web_client(s_action, **kw)

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
            <p>Dear <span t-esc="partner.name"/>,</p>
            <p>You have been invited to access the following document:</p>
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
                </td><td valign="middle" align="right">
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
                            <a href="/web/login" style="display: inline-block; padding: 10px; text-decoration: none; font-size: 12px;">
                                <strong>Log in</strong>
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
# -*- coding: utf-8 -*-
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
        if request and request.is_frontend:
            return [lang[0] for lang in filter(lambda l: l[3], request.env['res.lang'].get_available())]
        return super()._get_frontend_langs()

```

## File: models\ir_ui_view.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import url_for
from odoo.tools import is_html_empty


class View(models.Model):
    _inherit = "ir.ui.view"

    customize_show = fields.Boolean("Show As Optional Inherit", default=False)

    @api.model
    def _prepare_qcontext(self):
        """ Returns the qcontext : rendering context with portal specific value (required
            to render portal layout template)
        """
        qcontext = super(View, self)._prepare_qcontext()
        if request and getattr(request, 'is_frontend', False):
            Lang = request.env['res.lang']
            portal_lang_code = request.env['ir.http']._get_frontend_langs()
            qcontext.update(dict(
                self._context.copy(),
                languages=[lang for lang in Lang.get_available() if lang[0] in portal_lang_code],
                url_for=url_for,
                is_html_empty=is_html_empty,
            ))
        return qcontext

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailMessage(models.Model):
    _inherit = 'mail.message'

    def portal_message_format(self):
        return self._portal_message_format([
            'id', 'body', 'date', 'author_id', 'email_from',  # base message fields
            'message_type', 'subtype_id', 'is_internal', 'subject',  # message specific
            'model', 'res_id', 'record_name',  # document related
        ])

    def _portal_message_format(self, fields_list):
        vals_list = self._message_format(fields_list)
        message_subtype_note_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note')
        IrAttachmentSudo = self.env['ir.attachment'].sudo()
        for vals in vals_list:
            vals['is_message_subtype_note'] = message_subtype_note_id and (vals.get('subtype_id') or [False])[0] == message_subtype_note_id
            for attachment in vals.get('attachment_ids', []):
                if not attachment.get('access_token'):
                    attachment['access_token'] = IrAttachmentSudo.browse(attachment['id']).generate_access_token()[0]
        return vals_list

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import hmac

from odoo import api, fields, models, _


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    _mail_post_token_field = 'access_token' # token field for external posts, to be overridden

    website_message_ids = fields.One2many('mail.message', 'res_id', string='Website Messages',
        domain=lambda self: [('model', '=', self._name), '|', ('message_type', '=', 'comment'), ('message_type', '=', 'email')], auto_join=True,
        help="Website communication history")

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

```

## File: models\portal_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import uuid
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
        params = {
            'model': self._name,
            'res_id': self.id,
        }
        if share_token and hasattr(self, 'access_token'):
            params['access_token'] = self._portal_ensure_token()
        if pid:
            params['pid'] = pid
            params['hash'] = self._sign_token(pid)
        if signup_partner and hasattr(self, 'partner_id') and self.partner_id:
            params.update(self.partner_id.signup_get_auth_param()[self.partner_id.id])

        return '%s?%s' % ('/mail/view' if redirect else self.access_url, url_encode(params))

    def _notify_get_groups(self, msg_vals=None):
        access_token = self._portal_ensure_token()
        groups = super(PortalMixin, self)._notify_get_groups(msg_vals=msg_vals)
        local_msg_vals = dict(msg_vals or {})

        if access_token and 'partner_id' in self._fields and self['partner_id']:
            customer = self['partner_id']
            local_msg_vals['access_token'] = self.access_token
            local_msg_vals.update(customer.signup_get_auth_param()[customer.id])
            access_link = self._notify_get_action_link('view', **local_msg_vals)

            new_group = [
                ('portal_customer', lambda pdata: pdata['id'] == customer.id, {
                    'has_button_access': False,
                    'button_access': {
                        'url': access_link,
                    },
                    'notification_is_customer': True,
                })
            ]
        else:
            new_group = []
        return new_group + groups

    def get_access_action(self, access_uid=None):
        """ Instead of the classic form view, redirect to the online document for
        portal users or if force_website=True in the context. """
        self.ensure_one()

        user, record = self.env.user, self
        if access_uid:
            try:
                record.check_access_rights('read')
                record.check_access_rule("read")
            except exceptions.AccessError:
                return super(PortalMixin, self).get_access_action(access_uid)
            user = self.env['res.users'].sudo().browse(access_uid)
            record = self.with_user(user)
        if user.share or self.env.context.get('force_website'):
            try:
                record.check_access_rights('read')
                record.check_access_rule('read')
            except exceptions.AccessError:
                if self.env.context.get('force_website'):
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
        return super(PortalMixin, self).get_access_action(access_uid)

    @api.model
    def action_share(self):
        action = self.env["ir.actions.actions"]._for_xml_id("portal.portal_share_action")
        action['context'] = {'active_id': self.env.context['active_id'],
                             'active_model': self.env.context['active_model']}
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

    def can_edit_vat(self):
        ''' `vat` is a commercial field, synced between the parent (commercial
        entity) and the children. Only the commercial entity should be able to
        edit it (as in backend). '''
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

## File: static\src\js\portal.js

```javascript
odoo.define('portal.portal', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
const Dialog = require('web.Dialog');
const {_t, qweb} = require('web.core');
const ajax = require('web.ajax');
const session = require('web.session');

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

publicWidget.registry.PortalHomeCounters = publicWidget.Widget.extend({
    selector: '.o_portal_my_home',

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
     * @private
     */
    async _updateCounters(elem) {
        const numberRpc = 3;
        const needed = this.$('[data-placeholder_count]')
                                .map((i, o) => $(o).data('placeholder_count'))
                                .toArray();
        const counterByRpc = Math.ceil(needed.length / numberRpc);  // max counter, last can be less

        const proms = [...Array(Math.min(numberRpc, needed.length)).keys()].map(async i => {
            await this._rpc({
                route: "/my/counters",
                params: {
                    counters: needed.slice(i * counterByRpc, (i + 1) * counterByRpc)
                },
            }).then(data => {
                Object.keys(data).map(k => this.$("[data-placeholder_count='" + k + "']").text(data[k]));
            });
        });
        return Promise.all(proms);
    },
});

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
        var search = $.deparam(window.location.search.substring(1));
        const item = this.$('.dropdown-item.active').attr('href');
        search['search_in'] = item && item.replace('#', '') || '';
        search['search'] = this.$('input[name="search"]').val();
        window.location.search = $.param(search);
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

publicWidget.registry.NewAPIKeyButton = publicWidget.Widget.extend({
    selector: '.o_portal_new_api_key',
    events: {
        click: '_onClick'
    },

    async _onClick(e){
        e.preventDefault();
        // This call is done just so it asks for the password confirmation before starting displaying the
        // dialog forms, to mimic the behavior from the backend, in which it asks for the password before
        // displaying the wizard.
        // The result of the call is unused. But it's required to call a method with the decorator `@check_identity`
        // in order to use `handleCheckIdentity`.
        await handleCheckIdentity(this.proxy('_rpc'), this._rpc({
            model: 'res.users',
            method: 'api_key_wizard',
            args: [session.user_id],
        }));
        await ajax.loadXML('/portal/static/src/xml/portal_security.xml', qweb);
        const self = this;
        const d_description = new Dialog(self, {
            title: _t('New API Key'),
            $content: qweb.render('portal.keydescription'),
            buttons: [{text: _t('Confirm'), classes: 'btn-primary', close: true, click: async () => {
                var description = d_description.el.querySelector('[name="description"]').value;
                var wizard_id = await this._rpc({
                    model: 'res.users.apikeys.description',
                    method: 'create',
                    args: [{name: description}],
                });
                var res = await handleCheckIdentity(
                    this.proxy('_rpc'),
                    this._rpc({
                        model: 'res.users.apikeys.description',
                        method: 'make_key',
                        args: [wizard_id],
                    })
                );
                const d_show = new Dialog(self, {
                    title: _t('API Key Ready'),
                    $content: qweb.render('portal.keyshow', {key: res.context.default_key}),
                    buttons: [{text: _t('Close'), clases: 'btn-primary', close: true}],
                });
                d_show.on('closed', this, () => {
                    window.location = window.location;
                });
                d_show.open();
            }}, {text: _t('Discard'), close: true}],
        });
        d_description.opened(() => {
            const input = d_description.el.querySelector('[name="description"]');
            input.focus();
            d_description.el.addEventListener('submit', (e) => {
                e.preventDefault();
                d_description.$footer.find('.btn-primary').click();
            });
        });
        d_description.open();
    }
});

publicWidget.registry.RemoveAPIKeyButton = publicWidget.Widget.extend({
    selector: '.o_portal_remove_api_key',
    events: {
        click: '_onClick'
    },

    async _onClick(e){
        e.preventDefault();
        await handleCheckIdentity(
            this.proxy('_rpc'),
            this._rpc({
                model: 'res.users.apikeys',
                method: 'remove',
                args: [parseInt(this.target.id)]
            })
        );
        window.location = window.location;
    }
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
 * @param {Function} rpc Widget#_rpc bound do the widget
 * @param {Promise} wrapped promise to check for an identity check request
 * @returns {Promise} result of the original call
 */
function handleCheckIdentity(rpc, wrapped) {
    return wrapped.then((r) => {
        if (!_.isMatch(r, {type: 'ir.actions.act_window', res_model: 'res.users.identitycheck'})) {
            return r;
        }
        const check_id = r.res_id;
        return ajax.loadXML('/portal/static/src/xml/portal_security.xml', qweb).then(() => new Promise((resolve, reject) => {
            const d = new Dialog(null, {
                title: _t("Security Control"),
                $content: qweb.render('portal.identitycheck'),
                buttons: [{
                    text: _t("Confirm Password"), classes: 'btn btn-primary',
                    // nb: if click & close, waits for click to resolve before closing
                    click() {
                        const password_input = this.el.querySelector('[name=password]');
                        if (!password_input.reportValidity()) {
                            password_input.classList.add('is-invalid');
                            return;
                        }
                        return rpc({
                            model: 'res.users.identitycheck',
                            method: 'write',
                            args: [check_id, {password: password_input.value}]
                        }).then(() => rpc({
                            model: 'res.users.identitycheck',
                            method: 'run_check',
                            args: [check_id]
                        })).then((r) => {
                            this.close();
                            resolve(r);
                        }, (err) => {
                            err.event.preventDefault(); // suppress crashmanager
                            password_input.classList.add('is-invalid');
                            password_input.setCustomValidity(_t("Check failed"));
                            password_input.reportValidity();
                        });
                    }
                }, {
                    text: _t('Cancel'), close: true
                }]
            }).on('close', null, () => {
                // unlink wizard object?
                reject();
            });
            d.opened(() => {
                const pw = d.el.querySelector('[name="password"]');
                pw.focus();
                pw.addEventListener('input', () => {
                    pw.classList.remove('is-invalid');
                    pw.setCustomValidity('');
                });
                d.el.addEventListener('submit', (e) => {
                    e.preventDefault();
                    d.$footer.find('.btn-primary').click();
                });
            });
            d.open();
        }));
    });
}
return {
    handleCheckIdentity,
}
});

```

## File: static\src\js\portal_chatter.js

```javascript
odoo.define('portal.chatter', function (require) {
'use strict';

var core = require('web.core');
const dom = require('web.dom');
var publicWidget = require('web.public.widget');
var time = require('web.time');
var portalComposer = require('portal.composer');
const {Markup} = require('web.utils');

var qweb = core.qweb;
var _t = core._t;

/**
 * Widget PortalChatter
 *
 * - Fetch message fron controller
 * - Display chatter: pager, total message, composer (according to access right)
 * - Provider API to filter displayed messages
 */
var PortalChatter = publicWidget.Widget.extend({
    template: 'portal.Chatter',
    xmlDependencies: ['/portal/static/src/xml/portal_chatter.xml'],
    events: {
        'click .o_portal_chatter_pager_btn': '_onClickPager',
        'click .o_portal_chatter_js_is_internal': 'async _onClickUpdateIsInternal',
    },

    /**
     * @constructor
     */
    init: function (parent, options) {
        var self = this;
        this.options = {};
        this._super.apply(this, arguments);

        this._setOptions(options);

        this.set('messages', []);
        this.set('message_count', this.options['message_count']);
        this.set('pager', {});
        this.set('domain', this.options['domain']);
        this._currentPage = this.options['pager_start'];
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
        core.bus.on('reload_chatter_content', this, this._reloadChatterContent);

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
        return this._rpc({
            route: '/mail/chatter_fetch',
            params: self._messageFetchPrepareParams(),
        }).then(function (result) {
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
        _.each(messages, function (m) {
            m['author_avatar_url'] = _.str.sprintf('/web/image/%s/%s/author_avatar/50x50', 'mail.message', m.id);
            m['published_date_str'] = _.str.sprintf(_t('Published on %s'), moment(time.str_to_datetime(m.date)).format('MMMM Do YYYY, h:mm:ss a'));
            m['body'] = Markup(m.body);
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

        this.options = Object.entries(options).reduce(
            (acc, [key, value]) => {
                acc[_.str.underscored(key)] = value;
                return acc;
            },
            defaultOptions);
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
        return this._rpc({
            route: '/mail/chatter_init',
            params: this._messageFetchPrepareParams()
        }).then(function (result) {
            self.result = result;
            self.options = _.extend(self.options, self.result['options'] || {});
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
        var d = domain ? domain : _.clone(this.get('domain'));
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
        _.each(_.range(pmin, pmax + 1), function (index) {
            pages.push(index);
        });

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
        this.$('.o_portal_chatter_messages').html(qweb.render("portal.chatter_messages", {widget: this}));
    },
    _renderMessageCount: function () {
        this.$('.o_message_counter').replaceWith(qweb.render("portal.chatter_message_count", {widget: this}));
    },
    _renderPager: function () {
        this.$('.o_portal_chatter_pager').replaceWith(qweb.render("portal.pager", {widget: this}));
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onChangeDomain: function () {
        var self = this;
        this.messageFetch().then(function () {
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
        return this._rpc({
            route: '/mail/update_is_internal',
            params: {
                message_id: $elem.data('message-id'),
                is_internal: ! $elem.data('is-internal'),
            },
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

return {
    PortalChatter: PortalChatter,
};
});

```

## File: static\src\js\portal_composer.js

```javascript
odoo.define('portal.composer', function (require) {
'use strict';

var ajax = require('web.ajax');
var core = require('web.core');
var publicWidget = require('web.public.widget');

var qweb = core.qweb;
var _t = core._t;

/**
 * Widget PortalComposer
 *
 * Display the composer (according to access right)
 *
 */
var PortalComposer = publicWidget.Widget.extend({
    template: 'portal.Composer',
    xmlDependencies: ['/portal/static/src/xml/portal_chatter.xml'],
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
        this.options = _.defaults(options || {}, {
            'allow_composer': true,
            'display_composer': false,
            'csrf_token': odoo.csrf_token,
            'token': false,
            'res_model': false,
            'res_id': false,
        });
        this.attachments = [];
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
                _.each(self.attachments, function(attachment) {
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
        var accessToken = _.find(this.attachments, {'id': attachmentId}).access_token;
        ev.preventDefault();
        ev.stopPropagation();

        this.$sendButton.prop('disabled', true);

        return this._rpc({
            route: '/portal/attachment/remove',
            params: {
                'attachment_id': attachmentId,
                'access_token': accessToken,
            },
        }).then(function () {
            self.attachments = _.reject(self.attachments, {'id': attachmentId});
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

        return Promise.all(_.map(this.$fileInput[0].files, function (file) {
            return new Promise(function (resolve, reject) {
                var data = self._prepareAttachmentData(file);
                ajax.post('/portal/attachment/add', data).then(function (attachment) {
                    attachment.state = 'pending';
                    self.attachments.push(attachment);
                    self._updateAttachments();
                    resolve();
                }).guardedCatch(function (error) {
                    self.displayNotification({
                        message: _.str.sprintf(_t("Could not save file <strong>%s</strong>"),
                            _.escape(file.name)),
                        type: 'warning',
                        sticky: true,
                    });
                    resolve();
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
            'attachment_ids': _.pluck(this.attachments, 'id'),
            'attachment_tokens': _.pluck(this.attachments, 'access_token'),
        });
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onSubmitButtonClick: function (ev) {
        ev.preventDefault();
        if (!this.$inputTextarea.val().trim() && !this.attachments.length) {
            this.$inputTextarea.addClass('border-danger');
            const error = _t('Some fields are required. Please make sure to write a message or attach a document');
            this.$(".o_portal_chatter_composer_error").text(error).removeClass('d-none');
            return Promise.reject();
        } else {
            return this._chatterPostMessage(ev.currentTarget.getAttribute('data-action'));
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _updateAttachments: function () {
        this.$attachments.html(qweb.render('portal.Chatter.Attachments', {
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
        const result = await this._rpc({
            route: route,
            params: this._prepareMessageData(),
        });
        core.bus.trigger('reload_chatter_content', result);
        return result;
    },
});

return {
    PortalComposer: PortalComposer,
};
});

```

## File: static\src\js\portal_sidebar.js

```javascript
odoo.define('portal.PortalSidebar', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');
var time = require('web.time');
var session = require('web.session');

var _t = core._t;

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
        var $sidebarTimeago = this.$el.find('.o_portal_sidebar_timeago');
        _.each($sidebarTimeago, function (el) {
            var dateTime = moment(time.auto_str_to_date($(el).attr('datetime'))),
                today = moment().startOf('day'),
                diff = dateTime.diff(today, 'days', true),
                displayStr;

            session.is_bound.then(function (){
                if (diff === 0) {
                    displayStr = _t('Due today');
                } else if (diff > 0) {
                    // Workaround: force uniqueness of these two translations. We use %1d because the string
                    // with %d is already used in mail and mail's translations are not sent to the frontend.
                    displayStr = _.str.sprintf(_t('Due in %1d days'), Math.abs(diff));
                } else {
                    displayStr = _.str.sprintf(_t('%1d days overdue'), Math.abs(diff));
                }
                $(el).text(displayStr);
            });
        });
    },
    /**
     * @private
     * @param {string} href
     */
    _printIframeContent: function (href) {
        // due to this issue : https://bugzilla.mozilla.org/show_bug.cgi?id=911444
        // open a new window with pdf for print in Firefox (in other system: http://printjs.crabbly.com)
        if ($.browser.mozilla) {
            window.open(href, '_blank');
            return;
        }
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
return PortalSidebar;
});

```

## File: static\src\js\portal_signature.js

```javascript
odoo.define('portal.signature_form', function (require) {
'use strict';

var core = require('web.core');
var publicWidget = require('web.public.widget');
var NameAndSignature = require('web.name_and_signature').NameAndSignature;
var qweb = core.qweb;

var _t = core._t;

/**
 * This widget is a signature request form. It uses
 * @see NameAndSignature for the input fields, adds a submit
 * button, and handles the RPC to save the result.
 */
var SignatureForm = publicWidget.Widget.extend({
    template: 'portal.portal_signature',
    xmlDependencies: ['/portal/static/src/xml/portal_signature.xml'],
    events: {
        'click .o_portal_sign_submit': 'async _onClickSignSubmit',
    },
    custom_events: {
        'signature_changed': '_onChangeSignature',
    },

    /**
     * Overridden to allow options.
     *
     * @constructor
     * @param {Widget} parent
     * @param {Object} options
     * @param {string} options.callUrl - make RPC to this url
     * @param {string} [options.sendLabel='Accept & Sign'] - label of the
     *  send button
     * @param {Object} [options.rpcParams={}] - params for the RPC
     * @param {Object} [options.nameAndSignatureOptions={}] - options for
     *  @see NameAndSignature.init()
     */
    init: function (parent, options) {
        this._super.apply(this, arguments);

        this.csrf_token = odoo.csrf_token;

        this.callUrl = options.callUrl || '';
        this.rpcParams = options.rpcParams || {};
        this.sendLabel = options.sendLabel || _t("Accept & Sign");

        this.nameAndSignature = new NameAndSignature(this,
            options.nameAndSignatureOptions || {});
    },
    /**
     * Overridden to get the DOM elements
     * and to insert the name and signature.
     *
     * @override
     */
    start: function () {
        var self = this;
        this.$confirm_btn = this.$('.o_portal_sign_submit');
        this.$controls = this.$('.o_portal_sign_controls');
        var subWidgetStart = this.nameAndSignature.replace(this.$('.o_web_sign_name_and_signature'));
        return Promise.all([subWidgetStart, this._super.apply(this, arguments)]).then(function () {
            self.nameAndSignature.resetSignature();
        });
    },

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * Focuses the name.
     *
     * @see NameAndSignature.focusName();
     */
    focusName: function () {
        this.nameAndSignature.focusName();
    },
    /**
     * Resets the signature.
     *
     * @see NameAndSignature.resetSignature();
     */
    resetSignature: function () {
        return this.nameAndSignature.resetSignature();
    },

    //----------------------------------------------------------------------
    // Handlers
    //----------------------------------------------------------------------

    /**
     * Handles click on the submit button.
     *
     * This will get the current name and signature and validate them.
     * If they are valid, they are sent to the server, and the reponse is
     * handled. If they are invalid, it will display the errors to the user.
     *
     * @private
     * @param {Event} ev
     * @returns {Deferred}
     */
    _onClickSignSubmit: function (ev) {
        var self = this;
        ev.preventDefault();

        if (!this.nameAndSignature.validateSignature()) {
            return;
        }

        var name = this.nameAndSignature.getName();
        var signature = this.nameAndSignature.getSignatureImage()[1];

        return this._rpc({
            route: this.callUrl,
            params: _.extend(this.rpcParams, {
                'name': name,
                'signature': signature,
            }),
        }).then(function (data) {
            if (data.error) {
                self.$('.o_portal_sign_error_msg').remove();
                self.$controls.prepend(qweb.render('portal.portal_signature_error', {widget: data}));
            } else if (data.success) {
                var $success = qweb.render('portal.portal_signature_success', {widget: data});
                self.$el.empty().append($success);
            }
            if (data.force_refresh) {
                if (data.redirect_url) {
                    window.location = data.redirect_url;
                } else {
                    window.location.reload();
                }
                // no resolve if we reload the page
                return new Promise(function () { });
            }
        });
    },
    /**
     * Toggles the submit button depending on the signature state.
     *
     * @private
     */
    _onChangeSignature: function () {
        var isEmpty = this.nameAndSignature.isSignatureEmpty();
        this.$confirm_btn.prop('disabled', isEmpty);
    },
});

publicWidget.registry.SignatureForm = publicWidget.Widget.extend({
    selector: '.o_portal_signature_form',

    /**
     * @private
     */
    start: function () {
        var hasBeenReset = false;

        var callUrl = this.$el.data('call-url');
        var nameAndSignatureOptions = {
            defaultName: this.$el.data('default-name'),
            mode: this.$el.data('mode'),
            displaySignatureRatio: this.$el.data('signature-ratio'),
            signatureType: this.$el.data('signature-type'),
            fontColor: this.$el.data('font-color')  || 'black',
        };
        var sendLabel = this.$el.data('send-label');

        var form = new SignatureForm(this, {
            callUrl: callUrl,
            nameAndSignatureOptions: nameAndSignatureOptions,
            sendLabel: sendLabel,
        });

        // Correctly set up the signature area if it is inside a modal
        this.$el.closest('.modal').on('shown.bs.modal', function (ev) {
            if (!hasBeenReset) {
                // Reset it only the first time it is open to get correct
                // size. After we want to keep its content on reopen.
                hasBeenReset = true;
                form.resetSignature();
            } else {
                form.focusName();
            }
        });

        return Promise.all([
            this._super.apply(this, arguments),
            form.appendTo(this.$el)
        ]);
    },
});

return {
    SignatureForm: SignatureForm,
};
});

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
                <t t-if="count == 1">comment</t>
                <t t-else="">comments</t>
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
                <div class="media">
                    <img alt="Avatar" class="o_portal_chatter_avatar o_object_fit_cover" t-attf-src="/web/image/res.partner/#{widget.options['partner_id']}/avatar_128"
                         t-if="!widget.options['is_user_public'] or !widget.options['token']"/>
                    <div class="media-body">
                        <div class="o_portal_chatter_composer_input">
                            <div class="o_portal_chatter_composer_body mb32">
                                <textarea rows="4" name="message" class="form-control" placeholder="Write a message..."></textarea>
                                <div class="o_portal_chatter_attachments mt-3"/>
                                <div class="mt8">
                                    <button t-attf-data-action="/mail/chatter_post" class="o_portal_chatter_composer_btn btn btn-primary" type="submit">Send</button>
                                    <button class="o_portal_chatter_attachment_btn btn btn-secondary" type="button" title="Add attachment">
                                        <i class="fa fa-paperclip"/>
                                    </button>
                                </div>
                            </div>
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
        <div t-if="attachments.length" class="row">
            <div t-foreach="attachments" t-as="attachment" class="col-lg-2 col-md-3 col-sm-6">
                <div class="o_portal_chatter_attachment mb-2 position-relative text-center" t-att-data-id="attachment.id">
                    <button t-if="showDelete and attachment.state == 'pending'" class="o_portal_chatter_attachment_delete btn btn-sm btn-outline-danger" title="Delete">
                        <i class="fa fa-times"/>
                    </button>
                    <a t-attf-href="/web/content/#{attachment.id}?download=true&amp;access_token=#{attachment.access_token}" target="_blank">
                        <div class='oe_attachment_embedded o_image' t-att-title="attachment.name" t-att-data-mimetype="attachment.mimetype"/>
                        <div class='o_portal_chatter_attachment_name'><t t-esc='attachment.name'/></div>
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
            <t t-foreach="widget.get('messages') || []" t-as="message">
                <div class="media o_portal_chatter_message" t-att-id="'message-' + message.id">
                    <img class="o_portal_chatter_avatar" t-att-src="message.author_avatar_url" alt="avatar"/>
                    <div class="media-body">
                        <t t-call="portal.chatter_internal_toggle" t-if="widget.options['is_user_employee']"/>

                        <div class="o_portal_chatter_message_title">
                            <h5 class='mb-1'><t t-esc="message.author_id[1]"/></h5>
                            <p class="o_portal_chatter_puslished_date"><t t-esc="message.published_date_str"/></p>
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
        <div t-if="message.is_message_subtype_note" class="float-right">
            <button class="btn btn-secondary" title="Internal notes are only displayed to internal users." disabled="true">Internal Note</button>
        </div>
        <div t-else="" t-attf-class="float-right o_portal_chatter_js_is_internal #{message.is_internal and 'o_portal_message_internal_on' or 'o_portal_message_internal_off'}"
                t-att-data-message-id="message.id"
                t-att-data-is-internal="message.is_internal">
            <button class="btn btn-danger"
                title="Currently restricted to internal employees, click to make it available to everyone viewing this document.">Employees Only</button>
            <button class="btn btn-success"
                title="Currently available to everyone viewing this document, click to restrict to internal employees.">Visible</button>
        </div>
    </t>

    <t t-name="portal.pager">
        <div class="o_portal_chatter_pager">
            <t t-if="!_.isEmpty(widget.get('pager'))">
                <ul class="pagination" t-if="widget.get('pager')['pages'].length &gt; 1">
                    <li t-if="widget.get('pager')['page'] != widget.get('pager')['page_previous']" t-att-data-page="widget.get('pager')['page_previous']" class="page-item o_portal_chatter_pager_btn">
                        <a href="#" class="page-link"><i class="fa fa-chevron-left" role="img" aria-label="Previous" title="Previous"/></a>
                    </li>
                    <t t-foreach="widget.get('pager')['pages']" t-as="page">
                        <li t-att-data-page="page" t-attf-class="page-item #{page == widget.get('pager')['page'] ? 'o_portal_chatter_pager_btn active' : 'o_portal_chatter_pager_btn'}">
                            <a href="#" class="page-link"><t t-esc="page"/></a>
                        </li>
                    </t>
                    <li t-if="widget.get('pager')['page'] != widget.get('pager')['page_next']" t-att-data-page="widget.get('pager')['page_next']" class="page-item o_portal_chatter_pager_btn">
                        <a href="#" class="page-link"><i class="fa fa-chevron-right" role="img" aria-label="Next" title="Next"/></a>
                    </li>
                </ul>
            </t>
        </div>
    </t>

    <t t-name="portal.Chatter">
        <t t-set="two_columns" t-value="widget.options['two_columns']"/>
        <div t-attf-class="o_portal_chatter p-0 #{two_columns and 'row' or ''}">
            <div t-attf-class="#{two_columns and 'col-lg-5' or ''}">
                <div class="o_portal_chatter_header">
                    <t t-call="portal.chatter_message_count"/>
                </div>
                <hr t-if="widget.options['allow_composer']"/>
                <div class="o_portal_chatter_composer"/>
            </div>
            <hr t-if="!two_columns"/>
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
</templates>

```

## File: static\src\xml\portal_signature.xml

```xml
<templates id="template" xml:space="preserve">

    <!-- Template for the widget SignatureForm. -->
    <t t-name="portal.portal_signature">
        <form method="POST">
            <input type="hidden" name="csrf_token" t-att-value="widget.csrf_token"/>
            <div class="o_web_sign_name_and_signature"/>
            <div class="o_portal_sign_controls my-3">
                <div class="text-right my-3">
                    <button type="submit" class="o_portal_sign_submit btn btn-primary" disabled="disabled">
                        <i class="fa fa-check"/>
                        <t t-esc="widget.sendLabel"/>
                    </button>
                </div>
            </div>
        </form>
    </t>
    <!-- Template when the sign rpc is successful. -->
    <t t-name="portal.portal_signature_success">
        <div class="alert alert-success" role="status">
            <span t-if="widget.message" t-esc="widget.message"/>
            <span t-else="">Thank You!</span>
            <a t-if="widget.redirect_url" t-att-href="widget.redirect_url">
                <t t-if="widget.redirect_message" t-esc="widget.redirect_message"/>
                <t t-else="">Click here to see your document.</t>
            </a>
        </div>
    </t>
    <!-- Template when the sign rpc returns an error. -->
    <t t-name="portal.portal_signature_error">
        <div class="o_portal_sign_error_msg alert alert-danger" role="status">
            <t t-esc="widget.error"/>
        </div>
    </t>
</templates>

```

## File: views\portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="frontend_layout" name="Main Frontend Layout" inherit_id="web.frontend_layout">
        <xpath expr="//div[@id='wrapwrap']" position="attributes">
            <attribute name="t-attf-class" add="#{request.env['res.lang']._lang_get(request.env.lang).direction == 'rtl' and 'o_rtl' or ''}" separator=" "/>
            <attribute name="t-attf-class" add="#{'o_portal' if is_portal else ''}" separator=" "/>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']/header/img" position="replace">
            <nav class="navbar navbar-expand navbar-light bg-light">
                <div class="container">
                    <a href="/" class="navbar-brand logo">
                        <img t-att-src="'/logo.png?company=%s' % res_company.id" t-att-alt="'Logo of %s' % res_company.name" t-att-title="res_company.name"/>
                    </a>
                    <ul id="top_menu" class="nav navbar-nav ml-auto">
                        <t t-call="portal.placeholder_user_sign_in">
                            <t t-set="_item_class" t-value="'nav-item'"/>
                            <t t-set="_link_class" t-value="'nav-link'"/>
                        </t>
                        <t t-call="portal.user_dropdown">
                            <t t-set="_user_name" t-value="true"/>
                            <t t-set="_item_class" t-value="'nav-item dropdown'"/>
                            <t t-set="_link_class" t-value="'nav-link'"/>
                            <t t-set="_dropdown_menu_class" t-value="'dropdown-menu-right'"/>
                        </t>
                    </ul>
                </div>
            </nav>
        </xpath>
        <xpath expr="//div[@id='wrapwrap']/main/t[@t-out='0']" position="before">
            <div t-if="o_portal_fullwidth_alert" class="alert alert-info alert-dismissible rounded-0 fade show d-print-none css_editable_mode_hidden">
                <div class="container">
                    <t t-out="o_portal_fullwidth_alert"/>
                </div>
            </div>
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
        <t t-set="active_lang" t-value="list(filter(lambda lg : lg[0] == lang, languages))[0]"/>
        <t t-set="language_selector_visible" t-value="len(languages) &gt; 1"/>
        <div t-attf-class="js_language_selector #{_div_classes}" t-if="language_selector_visible">
            <button t-attf-class="btn btn-sm btn-outline-secondary border-0 dropdown-toggle #{_btn_class}" type="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
                <span t-if="not no_text"
                        class="align-middle"
                        t-esc="active_lang[2].split('/').pop()"/>
            </button>
            <div t-attf-class="dropdown-menu #{_dropdown_menu_class}" role="menu">
                <t t-foreach="languages" t-as="lg">
                    <a t-att-href="url_for(request.httprequest.path + '?' + keep_query(), lang_code=lg[0])"
                       t-attf-class="dropdown-item js_change_lang #{active_lang == lg and 'active'}"
                       t-att-data-url_code="lg[1]">
                        <span t-if="not no_text" t-esc="lg[2].split('/').pop()"/>
                    </a>
                </t>
            </div>
        </div>
    </template>

    <template id="user_dropdown" name="Portal User Dropdown">
        <t t-set="is_connected" t-value="not user_id._is_public()"/>
        <li t-if="is_connected" t-attf-class="#{_item_class} o_no_autohide_item">
            <a href="#" role="button" data-toggle="dropdown" t-attf-class="dropdown-toggle #{_link_class}">
                <t t-if="_avatar">
                    <t t-set="avatar_source" t-value="image_data_uri(user_id.avatar_256)"/>
                    <img t-att-src="avatar_source" t-attf-class="rounded-circle o_object_fit_cover #{_avatar_class}" width="24" height="24" alt="" loading="eager"/>
                </t>
                <i t-if="_icon" t-attf-class="fa fa-1x fa-fw fa-user-circle-o #{_icon_class}"/>
                <span t-if="_user_name" t-attf-class="#{_user_name_class}" t-esc="user_id.name[:23] + '...' if user_id.name and len(user_id.name) &gt; 25 else user_id.name"/>
            </a>
            <div t-attf-class="dropdown-menu js_usermenu #{_dropdown_menu_class}" role="menu">
                <div id="o_logout_divider" class="dropdown-divider"/>
                <a t-attf-href="/web/session/logout?redirect=/" role="menuitem" id="o_logout" class="dropdown-item">Logout</a>
            </div>
        </li>
    </template>

    <template id="portal_breadcrumbs" name="Portal Breadcrumbs">
        <ol t-if="page_name != 'home'" class="o_portal_submenu breadcrumb mb-0 py-2 flex-grow-1 row">
            <li class="breadcrumb-item ml-1"><a href="/my/home" aria-label="Home" title="Home"><i class="fa fa-home"/></a></li>
            <li t-if="page_name == 'my_details'" class="breadcrumb-item">Details</li>
        </ol>
    </template>

    <template id="portal_back_in_edit_mode" name="Back to edit mode">
        <div t-ignore="true" class="text-center">
            <t t-if="custom_html" t-out="custom_html"/>
            <t t-else="">This is a preview of the customer portal.</t>
            <a t-att-href="backend_url"><i class="fa fa-arrow-right mr-1"/>Back to edit mode</a>
        </div>
        <button type="button" class="close" data-dismiss="alert" aria-label="Close"> &#215; </button>
    </template>

    <template id="portal_layout" name="Portal Layout">
        <t t-call="portal.frontend_layout">
            <t t-set="is_portal" t-value="True"/>

            <div t-if="not no_breadcrumbs and not my_details and not breadcrumbs_searchbar" class="o_portal container mt-3">
                <div class="row align-items-center bg-white no-gutters border rounded">
                    <div class="col-10">
                        <t t-call="portal.portal_breadcrumbs"></t>
                    </div>
                    <div t-if="prev_record or next_record" class="col-2 flex-grow-0 text-center">
                        <t t-call='portal.record_pager'/>
                    </div>
                </div>
            </div>
            <div id="wrap" class='o_portal_wrap'>
                <div class="container mb64">
                    <t t-if="my_details">
                        <div class="row justify-content-between mt-4">
                            <div t-attf-class="col-12 col-md col-lg-6">
                                <t t-out="0"/>
                            </div>
                            <div id="o_my_sidebar" class="pt-3 pt-lg-0 col-12 col-md col-lg-4 col-xl-3 o_my_sidebar">
                                <div class="o_my_contact" t-if="sales_user">
                                    <t t-call="portal.portal_contact"/>
                                </div>
                                <div class="o_portal_my_details">
                                    <h4>Details <a role="button" href="/my/account" class="btn btn-sm btn-link"><i class="fa fa-pencil"/> Edit</a></h4>
                                    <hr class="mt-1 mb-0"/>
                                    <div t-field="user_id.partner_id" t-options='{"widget": "contact", "fields": ["email", "phone", "address", "name"]}'/>
                                </div>
                                <div class="o_portal_my_security mt-3">
                                    <h4>Account Security </h4>
                                    <hr class="mt-1 mb-1"/>
                                    <a href="/my/security"><i class="fa fa-pencil mx-1"/>Edit Security Settings</a>
                                </div>
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
            <li groups="base.group_public" t-attf-class="#{_item_class} o_no_autohide_item">
                <a t-attf-href="/web/login" t-attf-class="#{_link_class}">Sign in<span t-if="request.session.profile_session" class="text-danger fa fa-circle"/></a>
            </li>
        </xpath>
    </template>

    <template id="portal_my_home" name="My Portal">
        <t t-call="portal.portal_layout">
            <t t-set="my_details" t-value="True"/>
            <div class="o_portal_my_home">
                <div class="oe_structure" id="oe_structure_portal_my_home_1"/>
                <h3>Documents</h3>
                <div class="o_portal_docs list-group">
                </div>
            </div>
            <div class="oe_structure" id="oe_structure_portal_my_home_2"/>
        </t>
    </template>

    <template id="portal_docs_entry" name="My Portal Docs Entry">
        <a t-att-href="url" t-att-title="title" class="list-group-item list-group-item-action d-flex align-items-center justify-content-between">
            <t t-esc="title"/>
            <t t-if='count'>
                <span class="badge badge-secondary badge-pill" t-esc="count"/>
            </t>
            <t t-elif="placeholder_count">
                <span class="badge badge-secondary badge-pill" t-att-data-placeholder_count="placeholder_count">
                    <i class="fa fa-spin fa-circle-o-notch"></i>
                </span>
            </t>
        </a>
    </template>

    <template id="portal_table" name="My Portal Table">
        <div t-attf-class="table-responsive border rounded border-top-0 #{classes if classes else ''}">
            <table class="table rounded mb-0 bg-white o_portal_my_doc_table">
                <t t-out="0"/>
            </table>
        </div>
        <div t-if="pager" class="o_portal_pager text-center">
            <t t-call="portal.pager"/>
        </div>
    </template>

    <template id="portal_record_sidebar" name="My Portal Record Sidebar">
        <div t-attf-class="#{classes}">
            <div class="card bg-white mb-4 sticky-top" id="sidebar_content">
                <div t-if="title" class="card-body text-center pb-2 pt-3">
                    <t t-out="title"/>
                </div>
                <t t-if="entries" t-out="entries"/>
                <div class="card-footer small text-center text-muted border-top-0 pt-1 pb-1 d-none d-lg-block">
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
        <nav t-attf-class="navbar navbar-light navbar-expand-lg border py-0 mb-2 o_portal_navbar {{classes if classes else ''}} {{'mt-3 rounded' if breadcrumbs_searchbar else 'border-top-0' }}">
            <!--  Navbar breadcrumb or title  -->
            <t t-if="breadcrumbs_searchbar">
                <t t-call="portal.portal_breadcrumbs"/>
            </t>
            <span t-else="" class="navbar-brand mb-0 h1 mr-auto" t-esc="title or 'No title'"/>

            <!--  Collapse button -->
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#o_portal_navbar_content" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle filters">
                <span class="navbar-toggler-icon small"/>
            </button>

            <!--  Collapsable content  -->
            <div class="collapse navbar-collapse" id="o_portal_navbar_content">
                <div class="nav flex-column flex-lg-row ml-auto p-0 mb-3 mb-lg-0 mt-1 mt-lg-0">
                    <div t-if="searchbar_sortings" class="form-inline">
                        <span class="small mr-1 navbar-text">Sort By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_sortby" data-toggle="dropdown" class="btn btn-secondary btn-sm dropdown-toggle">
                                <t t-esc="searchbar_sortings[sortby].get('label', 'Newest')"/>
                            </button>
                            <div class="dropdown-menu" aria-labelledby="portal_searchbar_sortby">
                                <t t-foreach="searchbar_sortings" t-as="option">
                                    <a t-att-href="request.httprequest.path + '?' + keep_query('*', sortby=option)"
                                       t-attf-class="dropdown-item#{sortby == option and ' active' or ''}">
                                        <span t-esc="searchbar_sortings[option].get('label')"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div t-if="searchbar_filters" class="form-inline ml-lg-2">
                        <span class="small mr-1 navbar-text">Filter By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_filters" data-toggle="dropdown" class="btn btn-secondary btn-sm dropdown-toggle">
                                <t t-esc="searchbar_filters.get(filterby,searchbar_filters.get('all')).get('label', 'All')"/>
                            </button>
                            <div class="dropdown-menu" aria-labelledby="portal_searchbar_filters">
                                <t t-foreach="searchbar_filters" t-as="option">
                                    <a t-att-href="default_url + '?' + keep_query('*', filterby=option)"
                                       t-attf-class="dropdown-item#{filterby == option and ' active' or ''}">
                                        <span t-esc="searchbar_filters[option].get('label')"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                    </div>
                    <div t-if="searchbar_groupby" class="form-inline ml-lg-2">
                        <span class="small mr-1 navbar-text">Group By:</span>
                        <div class="btn-group">
                            <button id="portal_searchbar_groupby" data-toggle="dropdown" class="btn btn-secondary btn-sm dropdown-toggle">
                                <t t-esc="searchbar_groupby[groupby].get('label', 'None')"/>
                            </button>
                            <div class="dropdown-menu" aria-labelledby="portal_searchbar_groupby">
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
                <form t-if="searchbar_inputs" class="form-inline o_portal_search_panel ml-lg-4 col-xl-4 col-md-5">
                    <div class="input-group input-group-sm w-100">
                        <div class="input-group-prepend">
                            <button type="button" class="btn btn-secondary dropdown-toggle" data-toggle="dropdown"/>
                            <div class="dropdown-menu" role="menu">
                                <t t-foreach='searchbar_inputs' t-as='input'>
                                    <a t-att-href="'#' + input_value['input']"
                                        t-attf-class="dropdown-item#{search_in == input_value['input'] and ' active' or ''}">
                                        <span t-out="input_value['label']"/>
                                    </a>
                                </t>
                            </div>
                        </div>
                        <input type="text" class="form-control form-control-sm" placeholder="Search" t-att-value='search' name="search"/>
                        <span class="input-group-append">
                            <button class="btn btn-secondary o_wait_lazy_js" type="submit">
                                <span class="fa fa-search"/>
                            </button>
                        </span>
                    </div>
                </form>
            </div>
        </nav>
    </template>

    <template id="portal_record_layout" name="Portal single record layout">
        <div t-attf-class="card mt-0 border-top-0 rounded-0 rounded-bottom #{classes if classes else ''}">
            <div t-if="card_header" t-attf-class="card-header #{header_classes if header_classes else ''}">
                <t t-out="card_header"/>
            </div>
            <div t-if="card_body" t-attf-class="card-body #{body_classes if body_classes else ''}">
                <t t-out="card_body"/>
            </div>
        </div>
    </template>

    <!--
    TODO adapt in master: remove the use of the "title" variable in this
    template (or rename the variable). Using it at controller level actually
    also controls the browser tab title, which makes no sense.
    -->
    <template id="portal_contact" name="Contact">
        <div class="o_portal_contact_details mb-5">
            <h4><t t-if="title" t-esc="title"/><t t-else="">Your contact</t></h4>
            <hr class="mt-1 mb0"/>
            <h6 class="mb-1"><b t-esc="sales_user.name"/></h6>
            <div class="d-flex align-items-center mb-1">
                <div class="fa fa-envelope fa-fw mr-1"></div>
                <a t-att-href="'mailto:'+sales_user.email" t-esc="sales_user.email"/>
            </div>
            <div class="d-flex flex-nowrap align-items-center mb-1">
                <div class="fa fa-phone fa-fw mr-1"></div>
                <span t-esc="sales_user.phone"/>
            </div>
            <div class="d-flex flex-nowrap align-items-center mb-1">
                <div class="fa fa-map-marker fa-fw mr-1"></div>
                <span t-esc="sales_user.city"/>
            </div>
        </div>
    </template>

    <template id="portal_my_details">
        <t t-call="portal.portal_layout">
            <t t-set="additional_title">Contact Details</t>
            <form action="/my/account" method="post">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <div class="row o_portal_details">
                    <div class="col-lg-8">
                        <div class="row">
                            <t t-set="partner_can_edit_vat" t-value="partner.can_edit_vat()"/>
                            <div class="col-lg-12">
                              <div t-if="error_message" class="alert alert-danger" role="alert">
                                  <t t-foreach="error_message" t-as="err"><t t-esc="err"/><br /></t>
                              </div>
                            </div>
                            <div t-attf-class="form-group #{error.get('name') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="name">Name</label>
                                <input type="text" name="name" t-attf-class="form-control #{error.get('name') and 'is-invalid' or ''}" t-att-value="name or partner.name" />
                            </div>
                            <div t-attf-class="form-group #{error.get('email') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="email">Email</label>
                                <input type="email" name="email" t-attf-class="form-control #{error.get('email') and 'is-invalid' or ''}" t-att-value="email or partner.email" />
                            </div>

                            <div class="clearfix" />
                            <div t-attf-class="form-group mb-1 #{error.get('company_name') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label label-optional" for="company_name">Company Name</label>
                                <!-- The <input> use "disabled" attribute to avoid sending an unauthorized value on form submit.
                                     The user might not have rights to change company_name but should still be able to see it.
                                -->
                                <input type="text" name="company_name" t-attf-class="form-control #{error.get('company_name') and 'is-invalid' or ''}" t-att-value="company_name or partner.commercial_company_name" t-att-disabled="None if partner_can_edit_vat else '1'" />
                                <small t-if="not partner_can_edit_vat" class="form-text text-muted d-block d-xl-none">
                                    Changing company name is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.
                                </small>
                            </div>
                            <div t-attf-class="form-group mb-1 #{error.get('vat') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label label-optional" for="vat">VAT Number</label>
                                <!-- The <input> use "disabled" attribute to avoid sending an unauthorized value on form submit.
                                     The user might not have rights to change company_name but should still be able to see it.
                                -->
                                <input type="text" name="vat" t-attf-class="form-control #{error.get('vat') and 'is-invalid' or ''}" t-att-value="vat or partner.vat" t-att-disabled="None if partner_can_edit_vat else '1'" />
                                <small t-if="not partner_can_edit_vat" class="form-text text-muted d-block d-xl-none">Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.</small>
                            </div>
                            <div t-if="not partner_can_edit_vat" class="col-12 d-none d-xl-block">
                                <small class="form-text text-muted">Changing company name or VAT number is not allowed once document(s) have been issued for your account. <br/>Please contact us directly for this operation.</small>
                            </div>
                            <div t-attf-class="form-group #{error.get('phone') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="phone">Phone</label>
                                <input type="tel" name="phone" t-attf-class="form-control #{error.get('phone') and 'is-invalid' or ''}" t-att-value="phone or partner.phone" />
                            </div>

                            <div class="clearfix" />
                            <div t-attf-class="form-group #{error.get('street') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="street">Street</label>
                                <input type="text" name="street" t-attf-class="form-control #{error.get('street') and 'is-invalid' or ''}" t-att-value="street or partner.street"/>
                            </div>
                            <div t-attf-class="form-group #{error.get('city') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="city">City</label>
                                <input type="text" name="city" t-attf-class="form-control #{error.get('city') and 'is-invalid' or ''}" t-att-value="city or partner.city" />
                            </div>
                            <div t-attf-class="form-group #{error.get('zip') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label label-optional" for="zipcode">Zip / Postal Code</label>
                                <input type="text" name="zipcode" t-attf-class="form-control #{error.get('zip') and 'is-invalid' or ''}" t-att-value="zipcode or partner.zip" />
                            </div>
                            <div t-attf-class="form-group #{error.get('country_id') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label" for="country_id">Country</label>
                                <select name="country_id" t-attf-class="form-control #{error.get('country_id') and 'is-invalid' or ''}">
                                    <option value="">Country...</option>
                                    <t t-foreach="countries or []" t-as="country">
                                        <option t-att-value="country.id" t-att-selected="country.id == int(country_id) if country_id else country.id == partner.country_id.id">
                                            <t t-esc="country.name" />
                                        </option>
                                    </t>
                                </select>
                            </div>
                            <div t-attf-class="form-group #{error.get('state_id') and 'o_has_error' or ''} col-xl-6">
                                <label class="col-form-label label-optional" for="state_id">State / Province</label>
                                <select name="state_id" t-attf-class="form-control #{error.get('state_id') and 'is-invalid' or ''}">
                                    <option value="">select...</option>
                                    <t t-foreach="states or []" t-as="state">
                                        <option t-att-value="state.id" style="display:none;" t-att-data-country_id="state.country_id.id" t-att-selected="state.id == partner.state_id.id">
                                            <t t-esc="state.name" />
                                        </option>
                                    </t>
                                </select>
                            </div>
                            <input type="hidden" name="redirect" t-att-value="redirect"/>
                        </div>
                        <div class="clearfix">
                            <button type="submit" class="btn btn-primary float-right mb32 ">
                                Confirm
                                <span class="fa fa-long-arrow-right" />
                            </button>
                        </div>
                    </div>
                </div>
            </form>
        </t>
    </template>

    <template id="portal_my_security">
        <t t-call="portal.portal_layout"><div class="o_portal_security_body">
            <t t-set="additional_title">Security</t>
            <t t-set="no_breadcrumbs" t-value="1"/>
            <div class="alert alert-danger" role="alert" t-if="get_error(errors)">
                <t t-esc="errors"/>
            </div>
            <section>
                <h3>Change Password</h3>
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
                    <div class="form-group">
                        <label for="current">Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.old') else '' }}"
                               id="current" name="old"
                               autocomplete="current-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.old')"/>
                        </div>
                    </div>
                    <div class="form-group">
                        <label for="new">New Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.new1') else '' }}"
                               id="new" name="new1"
                               autocomplete="new-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.new1')"/>
                        </div>
                    </div>
                    <div class="form-group">
                        <label for="new2">Verify New Password:</label>
                        <input type="password" t-attf-class="form-control form-control-sm {{ 'is-invalid' if get_error(errors, 'password.new2') else '' }}"
                               id="new2" name="new2"
                               autocomplete="new-password" required="required"/>
                        <div class="invalid-feedback">
                            <t t-esc="get_error(errors, 'password.new2')"/>
                        </div>
                    </div>
                    <button type="submit" class="btn btn-danger">Change Password</button>
                </form>
            </section>
            <section t-if="debug and allow_api_keys">
                <h3>
                Developer API Keys
                    <a href="https://www.odoo.com/documentation/15.0/developer/misc/api/external_api.html#api-keys" target="_blank">
                        <i title="Documentation" class="fa fa-fw o_button_icon fa-info-circle"></i>
                    </a>
                </h3>
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
                    <button type="submit" class="btn btn-primary o_portal_new_api_key">New API Key</button>
                </div>
            </section>
        </div></t>
    </template>

    <template id="record_pager" name="Portal Record Pager">
        <t t-if='prev_record or next_record'>
            <div class="record_pager btn-group" role="group">
                <a role="button" t-att-class="'btn btn-link %s' % ('disabled' if not prev_record else '')" t-att-href="prev_record or '#'" ><i class="fa fa-chevron-left" role="img" aria-label="Previous" title="Previous"></i></a>
                <a role="button" t-att-class="'btn btn-link %s' % ('disabled' if not next_record else '')" t-att-href="next_record or '#'" ><i class="fa fa-chevron-right" role="img" aria-label="Next" title="Next"></i></a>
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
            <a href="/my/home" role="menuitem" class="dropdown-item">My Account</a>
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
    -->
    <template id="message_thread">
        <div id="discussion" data-anchor="true" class="d-print-none o_portal_chatter o_not_editable p-0"
            t-att-data-token="token" t-att-data-res_model="object._name" t-att-data-pid="pid" t-att-data-hash="hash" t-att-data-res_id="object.id" t-att-data-pager_step="message_per_page or 10" t-att-data-allow_composer="'0' if disable_composer else '1'" t-att-data-two_columns="'true' if two_columns else 'false'">
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

        For the default values and more information, see init() of the widgets
        SignatureForm and NameAndSignature.
    -->
    <template id="portal.signature_form" name="Ask Signature">
        <div class="o_portal_signature_form"
            t-att-data-call-url="call_url"
            t-att-data-default-name="default_name"
            t-att-data-mode="mode"
            t-att-data-send-label="send_label"
            t-att-data-signature-ratio="signature_ratio"
            t-att-data-signature-type="signature_type"
            t-att-data-font-color="font_color"
            />
    </template>

    <template id="portal_sidebar" name="Sidebar">
        <t t-call="portal.portal_layout">
            <body data-spy="scroll" data-target=".navspy" data-offset="50">
                <div class="container o_portal_sidebar"></div>
                <div class="oe_structure mb32" id="oe_structure_portal_sidebar_1"/>
            </body>
        </t>
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
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name=%(base.action_apikeys_admin)d]//ancestor::div[hasclass('o_setting_box')]" position="inside">
                    <div groups="base.group_no_one">
                        <div class="o_setting_left_pane">
                            <field name="portal_allow_api_keys"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="portal_allow_api_keys"/>
                            <div class="text-muted">
                                Let your customers create developer API keys
                            </div>
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

    @api.model
    def _get_note(self):
        return self.env.ref('mail.mt_note')

    def _send_public_link(self, note, partners=None):
        if partners is None:
            partners = self.partner_ids
        for partner in partners:
            share_link = self.resource_ref.get_base_url() + self.resource_ref._get_share_url(redirect=True, pid=partner.id)
            saved_lang = self.env.lang
            self = self.with_context(lang=partner.lang)
            template = self.env.ref('portal.portal_share_template', False)
            self.resource_ref.message_post_with_view(template,
                values={'partner': partner, 'note': self.note, 'record': self.resource_ref,
                        'share_link': share_link},
                subject=_("You are invited to access %s", self.resource_ref.display_name),
                subtype_id=note.id,
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=[(6, 0, partner.ids)])
            self = self.with_context(lang=saved_lang)

    def _send_signup_link(self, note, partners=None):
        if partners is None:
            partners = self.partner_ids.filtered(lambda partner: not partner.user_ids)
        for partner in partners:
            #  prepare partner for signup and send singup url with redirect url
            partner.signup_get_auth_param()
            share_link = partner._get_signup_url_for_action(action='/mail/view', res_id=self.res_id, model=self.res_model)[partner.id]
            saved_lang = self.env.lang
            self = self.with_context(lang=partner.lang)
            template = self.env.ref('portal.portal_share_template', False)
            self.resource_ref.message_post_with_view(template,
                values={'partner': partner, 'note': self.note, 'record': self.resource_ref,
                        'share_link': share_link},
                subject=_("You are invited to access %s", self.resource_ref.display_name),
                subtype_id=note.id,
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=[(6, 0, partner.ids)])
            self = self.with_context(lang=saved_lang)

    def action_send_mail(self):
        note = self._get_note()
        signup_enabled = self.env['ir.config_parameter'].sudo().get_param('auth_signup.invitation_scope') == 'b2c'

        if getattr(self.resource_ref, 'access_token', False) or not signup_enabled:
            partner_ids = self.partner_ids
        else:
            partner_ids = self.partner_ids.filtered(lambda x: x.user_ids)
        # if partner already user or record has access token send common link in batch to all user
        self._send_public_link(note, partner_ids)
        # when partner not user send individual mail with signup token
        self._send_signup_link(note, self.partner_ids - partner_ids)

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
                <p class="alert alert-warning" attrs="{'invisible': [('access_warning', '=', '')]}" role="alert"><field name="access_warning"/></p>
                <group name="share_link">
                    <field name="res_model" invisible="1"/>
                    <field name="res_id" invisible="1"/>
                    <field name="share_link" widget="CopyClipboardChar" options="{'string': 'Copy Link'}"/>
                </group>
                <group>
                    <field name="partner_ids" widget="many2many_tags_email" placeholder="Add contacts to share the document..."/>
                </group>
                <group>
                    <field name="note" placeholder="Add a note"/>
                </group>
                <footer>
                    <button string="Send" name="action_send_mail" attrs="{'invisible': [('access_warning', '!=', '')]}" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-default" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>

    <record id="portal_share_action" model="ir.actions.act_window">
        <field name="name">Share Document</field>
        <field name="res_model">portal.share</field>
        <field name="binding_model_id" ref="model_portal_share"/>
        <field name="view_mode">form</field>
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
        self.refresh()
        return {
            'name': _('Portal Access Management'),
            'type': 'ir.actions.act_window',
            'res_model': 'portal.wizard',
            'view_type': 'form',
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

    @api.depends('partner_id')
    def _compute_user_id(self):
        for portal_wizard_user in self:
            user = portal_wizard_user.partner_id.with_context(active_test=False).user_ids
            portal_wizard_user.user_id = user[0] if user else False

    @api.depends('user_id', 'user_id.groups_id')
    def _compute_group_details(self):
        for portal_wizard_user in self:
            user = portal_wizard_user.user_id

            if user and user.has_group('base.group_user'):
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

        # update partner email, if a new one was introduced
        if self.partner_id.email != self.email:
            self.partner_id.write({'email': self.email})

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

        return self.wizard_id._action_open_modal()

    def action_revoke_access(self):
        """Remove the user of the partner from the portal group.

        If the user was only in the portal group, we archive it.
        """
        self.ensure_one()
        self._assert_user_email_uniqueness()

        if not self.is_portal:
            raise UserError(_('The partner "%s" has no portal access.', self.partner_id.name))

        group_portal = self.env.ref('base.group_portal')
        group_public = self.env.ref('base.group_public')

        # update partner email, if a new one was introduced
        if self.partner_id.email != self.email:
            self.partner_id.write({'email': self.email})

        # Remove the sign up token, so it can not be used
        self.partner_id.sudo().signup_token = False

        user_sudo = self.user_id.sudo()

        # remove the user from the portal group
        if user_sudo and user_sudo.has_group('base.group_portal'):
            user_sudo.write({'groups_id': [(3, group_portal.id), (4, group_public.id)], 'active': False})

        return self.wizard_id._action_open_modal()

    def action_invite_again(self):
        """Re-send the invitation email to the partner."""
        self.ensure_one()

        if not self.is_portal:
            raise UserError(_('You should first grant the portal access to the partner "%s".', self.partner_id.name))

        # update partner email, if a new one was introduced
        if self.partner_id.email != self.email:
            self.partner_id.write({'email': self.email})

        self.with_context(active_test=True)._send_email()

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

        email = email_normalize(self.email)

        if not email:
            raise UserError(_('The contact "%s" does not have a valid email.', self.partner_id.name))

        user = self.env['res.users'].sudo().with_context(active_test=False).search(
            self._get_similar_user_domain(email),
            limit=1
        )

        if user:
            raise UserError(self._get_same_email_error_message(user.name))

    def _get_similar_user_domain(self, email):
        """ Returns the domain needed to find the users that have the same email
        as the current partner.
        :param string email: the email of the current partner
        """
        return [('id', '!=', self.user_id.id),
                ('login', '=ilike', email)]

    def _get_same_email_error_message(self, user_name):
        """ Returns the error message in case the current partner has the same
        email as an existing user.
        :param string user_name: the name of the user that has the same email
        as the current partner
        """
        return _('The contact "%s" has the same email has an existing user (%s).', self.partner_id.name, user_name)

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
                    <field name="user_ids">
                        <tree string="Contacts" editable="bottom" create="false" delete="false">
                            <field name="partner_id" force_save="1"/>
                            <field name="email" attrs="{'readonly': [('is_internal', '=', True)]}"/>
                            <field name="login_date"/>
                            <field name="is_portal" invisible="1"/>
                            <field name="is_internal" invisible="1"/>
                            <button string="Grant Access" name="action_grant_access" type="object" class="btn-secondary"
                                attrs="{'invisible': ['|', ('is_portal', '=', True), ('is_internal', '=', True)]}"/>
                            <button string="Revoke Access" name="action_revoke_access" type="object" class="btn-secondary"
                                attrs="{'invisible': ['|', ('is_portal', '=', False), ('is_internal', '=', True)]}"/>
                            <button string="Re-Invite" name="action_invite_again" type="object" class="btn-secondary"
                                attrs="{'invisible': ['|', ('is_portal', '=', False), ('is_internal', '=', True)]}"/>
                            <button string="Internal User" attrs="{'invisible': [('is_internal', '=', False)]}"
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

