# Odoo Module: website_profile

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website profile',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Access the website profile of the users',
    'description': "Allows to access the website profile of the users and see their statistics (karma, badges, etc..)",
    'depends': [
        'website_partner',
        'gamification'
    ],
    'data': [
        'data/profile_data.xml',
        'views/gamification_badge_views.xml',
        'views/website_profile.xml',
        'security/ir.model.access.csv',
    ],
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import werkzeug
import werkzeug.exceptions
import werkzeug.urls
import werkzeug.wrappers
import math

from odoo import http, modules, tools
from odoo.http import request
from odoo.osv import expression


class WebsiteProfile(http.Controller):
    _users_per_page = 30
    _pager_max_pages = 5

    # Profile
    # ---------------------------------------------------

    def _check_avatar_access(self, user_id, **post):
        """ Base condition to see user avatar independently form access rights
        is to see published users having karma, meaning they participated to
        frontend applications like forum or elearning. """
        try:
            user = request.env['res.users'].sudo().browse(user_id).exists()
        except:
            return False
        if user:
            return user.website_published and user.karma > 0
        return False

    def _get_default_avatar(self):
        img_path = modules.get_module_resource('web', 'static/src/img', 'placeholder.png')
        with open(img_path, 'rb') as f:
            return base64.b64encode(f.read())

    def _check_user_profile_access(self, user_id):
        user_sudo = request.env['res.users'].sudo().browse(user_id)
        # User can access - no matter what - his own profile
        if user_sudo.id == request.env.user.id:
            return user_sudo
        if user_sudo.karma == 0 or not user_sudo.website_published or \
            (user_sudo.id != request.session.uid and request.env.user.karma < request.website.karma_profile_min):
            return False
        return user_sudo

    def _prepare_user_values(self, **kwargs):
        kwargs.pop('edit_translations', None) # avoid nuking edit_translations
        values = {
            'user': request.env.user,
            'is_public_user': request.website.is_public_user(),
            'validation_email_sent': request.session.get('validation_email_sent', False),
            'validation_email_done': request.session.get('validation_email_done', False),
        }
        values.update(kwargs)
        return values

    def _prepare_user_profile_parameters(self, **post):
        return post

    def _prepare_user_profile_values(self, user, **post):
        return {
            'uid': request.env.user.id,
            'user': user,
            'main_object': user,
            'is_profile_page': True,
            'edit_button_url_param': '',
        }

    @http.route([
        '/profile/avatar/<int:user_id>',
    ], type='http', auth="public", website=True, sitemap=False)
    def get_user_profile_avatar(self, user_id, field='image_256', width=0, height=0, crop=False, **post):
        if field not in ('image_128', 'image_256'):
            return werkzeug.exceptions.Forbidden()

        can_sudo = self._check_avatar_access(user_id, **post)
        if can_sudo:
            status, headers, image_base64 = request.env['ir.http'].sudo().binary_content(
                model='res.users', id=user_id, field=field,
                default_mimetype='image/png')
        else:
            status, headers, image_base64 = request.env['ir.http'].binary_content(
                model='res.users', id=user_id, field=field,
                default_mimetype='image/png')
        if status == 301:
            return request.env['ir.http']._response_by_status(status, headers, image_base64)
        if status == 304:
            return werkzeug.wrappers.Response(status=304)

        if not image_base64:
            image_base64 = self._get_default_avatar()
            if not (width or height):
                width, height = tools.image_guess_size_from_field_name(field)

        image_base64 = tools.image_process(image_base64, size=(int(width), int(height)), crop=crop)

        content = base64.b64decode(image_base64)
        headers = http.set_safe_image_headers(headers, content)
        response = request.make_response(content, headers)
        response.status_code = status
        return response

    @http.route(['/profile/user/<int:user_id>'], type='http', auth="public", website=True)
    def view_user_profile(self, user_id, **post):
        user = self._check_user_profile_access(user_id)
        if not user:
            return request.render("website_profile.private_profile")
        values = self._prepare_user_values(**post)
        params = self._prepare_user_profile_parameters(**post)
        values.update(self._prepare_user_profile_values(user, **params))
        return request.render("website_profile.user_profile_main", values)

    # Edit Profile
    # ---------------------------------------------------
    @http.route('/profile/edit', type='http', auth="user", website=True)
    def view_user_profile_edition(self, **kwargs):
        user_id = int(kwargs.get('user_id', 0))
        countries = request.env['res.country'].search([])
        if user_id and request.env.user.id != user_id and request.env.user._is_admin():
            user = request.env['res.users'].browse(user_id)
            values = self._prepare_user_values(searches=kwargs, user=user, is_public_user=False)
        else:
            values = self._prepare_user_values(searches=kwargs)
        values.update({
            'email_required': kwargs.get('email_required'),
            'countries': countries,
            'url_param': kwargs.get('url_param'),
        })
        return request.render("website_profile.user_profile_edit_main", values)

    def _profile_edition_preprocess_values(self, user, **kwargs):
        values = {
            'name': kwargs.get('name'),
            'website': kwargs.get('website'),
            'email': kwargs.get('email'),
            'city': kwargs.get('city'),
            'country_id': int(kwargs.get('country')) if kwargs.get('country') else False,
            'website_description': kwargs.get('description'),
        }

        if 'clear_image' in kwargs:
            values['image_1920'] = False
        elif kwargs.get('ufile'):
            image = kwargs.get('ufile').read()
            values['image_1920'] = base64.b64encode(image)

        if request.uid == user.id:  # the controller allows to edit only its own privacy settings; use partner management for other cases
            values['website_published'] = kwargs.get('website_published') == 'True'
        return values

    @http.route('/profile/user/save', type='http', auth="user", methods=['POST'], website=True)
    def save_edited_profile(self, **kwargs):
        user_id = int(kwargs.get('user_id', 0))
        if user_id and request.env.user.id != user_id and request.env.user._is_admin():
            user = request.env['res.users'].browse(user_id)
        else:
            user = request.env.user
        values = self._profile_edition_preprocess_values(user, **kwargs)
        whitelisted_values = {key: values[key] for key in type(user).SELF_WRITEABLE_FIELDS if key in values}
        user.write(whitelisted_values)
        if kwargs.get('url_param'):
            return werkzeug.utils.redirect("/profile/user/%d?%s" % (user.id, kwargs['url_param']))
        else:
            return werkzeug.utils.redirect("/profile/user/%d" % user.id)

    # Ranks and Badges
    # ---------------------------------------------------
    def _prepare_badges_domain(self, **kwargs):
        """
        Hook for other modules to restrict the badges showed on profile page, depending of the context
        """
        domain = [('website_published', '=', True)]
        if 'badge_category' in kwargs:
            domain = expression.AND([[('challenge_ids.category', '=', kwargs.get('badge_category'))], domain])
        return domain

    @http.route('/profile/ranks_badges', type='http', auth="public", website=True)
    def view_ranks_badges(self, **kwargs):
        ranks = []
        if 'badge_category' not in kwargs:
            Rank = request.env['gamification.karma.rank']
            ranks = Rank.sudo().search([], order='karma_min DESC')

        Badge = request.env['gamification.badge']
        badges = Badge.sudo().search(self._prepare_badges_domain(**kwargs))
        badges = sorted(badges, key=lambda b: b.stat_count_distinct, reverse=True)
        values = self._prepare_user_values(searches={'badges': True})

        values.update({
            'ranks': ranks,
            'badges': badges,
            'user': request.env.user,
        })
        return request.render("website_profile.rank_badge_main", values)

    # All Users Page
    # ---------------------------------------------------
    def _prepare_all_users_values(self, users):
        user_values = []
        for user in users:
            user_values.append({
                'id': user.id,
                'name': user.name,
                'company_name': user.company_id.name,
                'rank': user.rank_id.name,
                'karma': user.karma,
                'badge_count': len(user.badge_ids),
                'website_published': user.website_published
            })
        return user_values

    @http.route(['/profile/users',
                 '/profile/users/page/<int:page>'], type='http', auth="public", website=True)
    def view_all_users_page(self, page=1, **searches):
        User = request.env['res.users']
        dom = [('karma', '>', 1), ('website_published', '=', True)]

        # Searches
        search_term = searches.get('search')
        if search_term:
            dom = expression.AND([['|', ('name', 'ilike', search_term), ('company_id.name', 'ilike', search_term)], dom])

        user_count = User.sudo().search_count(dom)

        if user_count:
            page_count = math.ceil(user_count / self._users_per_page)
            pager = request.website.pager(url="/profile/users", total=user_count, page=page, step=self._users_per_page,
                                          scope=page_count if page_count < self._pager_max_pages else self._pager_max_pages)

            users = User.sudo().search(dom, limit=self._users_per_page, offset=pager['offset'], order='karma DESC')
            user_values = self._prepare_all_users_values(users)

            # Get karma position for users (only website_published)
            position_domain = [('karma', '>', 1), ('website_published', '=', True)]
            position_map = self._get_users_karma_position(position_domain, users.ids)
            for user in user_values:
                user['position'] = position_map.get(user['id'], 0)

            values = {
                'top3_users': user_values[:3] if not search_term and page == 1 else None,
                'users': user_values[3:] if not search_term and page == 1 else user_values,
                'pager': pager
            }
        else:
            values = {'top3_users': [], 'users': [], 'search': search_term, 'pager': dict(page_count=0)}

        return request.render("website_profile.users_page_main", values)

    def _get_users_karma_position(self, domain, user_ids):
        if not user_ids:
            return {}

        Users = request.env['res.users']
        where_query = Users._where_calc(domain)
        from_clause, where_clause, where_clause_params = where_query.get_sql()

        # we search on every user in the DB to get the real positioning (not the one inside the subset)
        # then, we filter to get only the subset.
        query = """
            SELECT sub.id, sub.karma_position
            FROM (
                SELECT "res_users"."id", row_number() OVER (ORDER BY res_users.karma DESC) AS karma_position
                FROM {from_clause}
                WHERE {where_clause}
            ) sub
            WHERE sub.id IN %s
            """.format(from_clause=from_clause, where_clause=where_clause)

        request.env.cr.execute(query, where_clause_params + [tuple(user_ids)])

        return {item['id']: item['karma_position'] for item in request.env.cr.dictfetchall()}

    # User and validation
    # --------------------------------------------------

    @http.route('/profile/send_validation_email', type='json', auth='user', website=True)
    def send_validation_email(self, **kwargs):
        if request.env.uid != request.website.user_id.id:
            request.env.user._send_profile_validation_email(**kwargs)
        request.session['validation_email_sent'] = True
        return True

    @http.route('/profile/validate_email', type='http', auth='public', website=True, sitemap=False)
    def validate_email(self, token, user_id, email, **kwargs):
        done = request.env['res.users'].sudo().browse(int(user_id))._process_profile_validation_token(token, email)
        if done:
            request.session['validation_email_done'] = True
        url = kwargs.get('redirect_url', '/')
        return request.redirect(url)

    @http.route('/profile/validate_email/close', type='json', auth='public', website=True)
    def validate_email_done(self, **kwargs):
        request.session['validation_email_done'] = False
        return True

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\profile_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Email template for email validation (for karma purpose) -->
        <record id="validation_email" model="mail.template">
            <field name="name">Profile: Email Verification</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="subject">${object.company_id.name} Profile validation</field>
            <field name="email_from">${user.email_formatted | safe}</field>
            <field name="email_to">${object.email_formatted | safe}</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="middle">
                        <span style="font-size: 20px; font-weight: bold;">
                            ${object.company_id.name} Profile validation
                        </span>
                    </td>
                    <td valign="middle" align="right">
                        <img src="/logo.png?company=${user.company_id.id}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" alt="${user.company_id.name}"/>
                    </td>
                </tr>
                <tr>
                    <td colspan="2" style="text-align:center;">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="top" style="font-size: 13px;">
                        <p style="margin: 0px; padding: 0px; font-size: 13px;">
                            Hello ${object.name},<br /><br />
                            You have been invited to validate your email in order to get access to "${object.company_id.name}" website.
                            To validate your email, please click on the following link:
                            <div style="margin: 16px 0px 16px 0px;">
                                <a href="${ctx.get('token_url')}"
                                    style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                                    Validate my account
                                </a>
                            </div>
                            Thanks for your participation!
                        </p>
                    </td>
                </tr>
                <tr>
                    <td style="text-align:center;">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style=" font-family: 'Verdana Regular'; color: #454748; min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="middle" align="left">
                         ${user.company_id.name}
                    </td>
                    <td valign="middle" align="right" style="opacity: 0.7;">
                        ${user.company_id.phone}
                        % if user.company_id.email:
                            | <a href="'mailto:%s' % ${user.company_id.email}" style="text-decoration:none; color: #454748;">
                                ${user.company_id.email}
                            </a>
                        % endif
                        % if user.company_id.website:
                            | <a href="${user.company_id.website}" style="text-decoration:none; color: #454748;">
                                ${user.company_id.website}
                            </a>
                        % endif
                    </td>
                </tr>
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
            Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=forum" style="color: #875A7B;">Odoo</a>
        </td></tr>
    </table>
</td></tr>
</table>
            </field>
            <field name="lang">${object.lang}</field>
            <field name="user_signature" eval="False"/>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: models\gamification_badge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class GamificationBadge(models.Model):
    _name = 'gamification.badge'
    _inherit = ['gamification.badge', 'website.published.mixin']

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import uuid

from datetime import datetime
from werkzeug import urls
from odoo import api, models

VALIDATION_KARMA_GAIN = 3


class Users(models.Model):
    _inherit = 'res.users'

    def __init__(self, pool, cr):
        init_res = super(Users, self).__init__(pool, cr)
        type(self).SELF_WRITEABLE_FIELDS = list(
            set(
                self.SELF_WRITEABLE_FIELDS +
                ['country_id', 'city', 'website', 'website_description', 'website_published']))
        type(self).SELF_READABLE_FIELDS = type(self).SELF_READABLE_FIELDS + ['karma']
        return init_res

    @api.model
    def _generate_profile_token(self, user_id, email):
        """Return a token for email validation. This token is valid for the day
        and is a hash based on a (secret) uuid generated by the forum module,
        the user_id, the email and currently the day (to be updated if necessary). """
        profile_uuid = self.env['ir.config_parameter'].sudo().get_param('website_profile.uuid')
        if not profile_uuid:
            profile_uuid = str(uuid.uuid4())
            self.env['ir.config_parameter'].sudo().set_param('website_profile.uuid', profile_uuid)
        return hashlib.sha256((u'%s-%s-%s-%s' % (
            datetime.now().replace(hour=0, minute=0, second=0, microsecond=0),
            profile_uuid,
            user_id,
            email
        )).encode('utf-8')).hexdigest()

    def _send_profile_validation_email(self, **kwargs):
        if not self.email:
            return False
        token = self._generate_profile_token(self.id, self.email)
        activation_template = self.env.ref('website_profile.validation_email')
        if activation_template:
            params = {
                'token': token,
                'user_id': self.id,
                'email': self.email
            }
            params.update(kwargs)
            base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
            token_url = base_url + '/profile/validate_email?%s' % urls.url_encode(params)
            with self._cr.savepoint():
                activation_template.sudo().with_context(token_url=token_url).send_mail(
                    self.id, force_send=True, raise_exception=True)
        return True

    def _process_profile_validation_token(self, token, email):
        self.ensure_one()
        validation_token = self._generate_profile_token(self.id, email)
        if token == validation_token and self.karma == 0:
            return self.write({'karma': VALIDATION_KARMA_GAIN})
        return False

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Website(models.Model):
    _inherit = 'website'

    karma_profile_min = fields.Integer(string="Minimal karma to see other user's profile", default=150)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gamification_badge
from . import website
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
gamification_karma_rank_access_website_publisher,gamification.karma.rank.access.website.publisher,gamification.model_gamification_karma_rank,website.group_website_publisher,1,1,1,1

```

## File: static\src\img\badge_bronze.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#C37933" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\img\badge_gold.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#E2BE00" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\img\badge_silver.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><g fill="none"><circle cx="150" cy="150" r="150" fill="#FFF"/><path fill="#838997" d="M150 300C67.157 300 0 232.843 0 150S67.157 0 150 0s150 67.157 150 150-67.157 150-150 150zm0-9.375c77.665 0 140.625-62.96 140.625-140.625 0-77.665-62.96-140.625-140.625-140.625C72.335 9.375 9.375 72.335 9.375 150c0 77.665 62.96 140.625 140.625 140.625zm0-14.063C80.101 276.563 23.437 219.9 23.437 150S80.102 23.437 150 23.437 276.563 80.102 276.563 150 219.899 276.563 150 276.563zm56.757-126.57l12.882-12.602c1.867-1.743 2.49-3.922 1.867-6.536-.747-2.551-2.365-4.139-4.854-4.76l-17.55-4.482 4.947-17.365c.747-2.552.156-4.73-1.773-6.535-1.805-1.93-3.983-2.52-6.535-1.774l-17.363 4.948-4.48-17.551c-.623-2.552-2.21-4.14-4.761-4.761-2.552-.685-4.73-.094-6.535 1.773L150 93.324l-12.602-12.977c-1.805-1.93-3.983-2.52-6.535-1.773-2.551.622-4.138 2.209-4.76 4.76l-4.481 17.552-17.363-4.948c-2.552-.747-4.73-.155-6.535 1.774-1.929 1.805-2.52 3.983-1.773 6.535l4.947 17.365-17.55 4.481c-2.489.623-4.107 2.21-4.854 4.761-.622 2.614 0 4.793 1.867 6.536l12.882 12.603-12.882 12.603c-1.867 1.743-2.49 3.921-1.867 6.535.747 2.552 2.365 4.14 4.854 4.762l17.55 4.481-4.947 17.365c-.747 2.552-.156 4.73 1.773 6.535 1.805 1.93 3.983 2.52 6.535 1.774l17.363-4.948 4.48 17.551c.623 2.552 2.21 4.17 4.761 4.855 2.614.622 4.792 0 6.535-1.867L150 206.755l12.602 12.884c1.245 1.369 2.832 2.053 4.761 2.053.436 0 1.027-.062 1.774-.186 2.551-.747 4.138-2.365 4.76-4.855l4.481-17.551 17.363 4.948c2.552.747 4.73.155 6.535-1.774 1.929-1.805 2.52-3.983 1.773-6.535l-4.947-17.365 17.55-4.481c2.489-.623 4.107-2.21 4.854-4.762.622-2.614 0-4.792-1.867-6.535l-12.882-12.603z"/></g></svg>
```

## File: static\src\img\rank_1.svg

```svg
<svg height="37" viewBox="0 0 35 37" width="35" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><linearGradient id="a" x1="50%" x2="50%" y1="0%" y2="100%"><stop offset="0" stop-color="#ffe897"/><stop offset="1" stop-color="#f1c363"/></linearGradient><circle id="b" cx="17.5" cy="17.5" r="17.5"/><filter id="c" height="111.4%" width="105.7%" x="-2.9%" y="-2.9%"><feOffset dy="2" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0.564705882   0 0 0 0 0.37254902   0 0 0 0 0.192156863  0 0 0 1 0"/></filter></defs><g fill="none" fill-rule="evenodd"><g fill-rule="nonzero"><use fill="#000" filter="url(#c)" xlink:href="#b"/><use fill="url(#a)" xlink:href="#b"/></g><text fill="#905f31" font-family="Montserrat-Bold, Montserrat" font-size="11" font-weight="bold"><tspan x="10.031" y="22">1st</tspan></text></g></svg>
```

## File: static\src\img\rank_2.svg

```svg
<svg height="37" viewBox="0 0 35 37" width="35" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><linearGradient id="a" x1="50%" x2="50%" y1="0%" y2="100%"><stop offset="0" stop-color="#d7dde1"/><stop offset="1" stop-color="#b7c3c8"/></linearGradient><circle id="b" cx="17.5" cy="17.5" r="17.5"/><filter id="c" height="111.4%" width="105.7%" x="-2.9%" y="-2.9%"><feOffset dy="2" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0.360784314   0 0 0 0 0.388235294   0 0 0 0 0.403921569  0 0 0 1 0"/></filter></defs><g fill="none" fill-rule="evenodd"><g fill-rule="nonzero"><use fill="#000" filter="url(#c)" xlink:href="#b"/><use fill="url(#a)" xlink:href="#b"/></g><text fill="#5c6367" font-family="Montserrat-Bold, Montserrat" font-size="11" font-weight="bold"><tspan x="6.615" y="22">2nd</tspan></text></g></svg>
```

## File: static\src\img\rank_3.svg

```svg
<svg height="37" viewBox="0 0 35 37" width="35" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><defs><linearGradient id="a" x1="50%" x2="50%" y1="0%" y2="100%"><stop offset="0" stop-color="#ecc29f"/><stop offset="1" stop-color="#d09e7d"/></linearGradient><circle id="b" cx="17.5" cy="17.5" r="17.5"/><filter id="c" height="111.4%" width="105.7%" x="-2.9%" y="-2.9%"><feOffset dy="2" in="SourceAlpha" result="shadowOffsetOuter1"/><feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0.62745098   0 0 0 0 0.462745098   0 0 0 0 0.329411765  0 0 0 1 0"/></filter></defs><g fill="none" fill-rule="evenodd"><g fill-rule="nonzero"><use fill="#000" filter="url(#c)" xlink:href="#b"/><use fill="url(#a)" xlink:href="#b"/></g><text fill="#5e4129" font-family="Montserrat-Bold, Montserrat" font-size="11" font-weight="bold"><tspan x="8.117" y="22">3rd</tspan></text></g></svg>
```

## File: static\src\js\website_profile.js

```javascript
odoo.define('website_profile.website_profile', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
var wysiwygLoader = require('web_editor.loader');

publicWidget.registry.websiteProfile = publicWidget.Widget.extend({
    selector: '.o_wprofile_email_validation_container',
    read_events: {
        'click .send_validation_email': '_onSendValidationEmailClick',
        'click .validated_email_close': '_onCloseValidatedEmailClick',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    /**
     * @private
     * @param {Event} ev
     */
    _onSendValidationEmailClick: function (ev) {
        ev.preventDefault();
        var self = this;
        var $element = $(ev.currentTarget);
        this._rpc({
            route: '/profile/send_validation_email',
            params: {'redirect_url': $element.data('redirect_url')},
        }).then(function (data) {
            if (data) {
                self.$('button.validation_email_close').click();
            }
        });
    },

    /**
     * @private
     */
    _onCloseValidatedEmailClick: function () {
        this._rpc({
            route: '/profile/validate_email/close',
        });
    },
});

publicWidget.registry.websiteProfileEditor = publicWidget.Widget.extend({
    selector: '.o_wprofile_editor_form',
    read_events: {
        'click .o_forum_profile_pic_edit': '_onEditProfilePicClick',
        'change .o_forum_file_upload': '_onFileUploadChange',
        'click .o_forum_profile_pic_clear': '_onProfilePicClearClick',
        'click .o_wprofile_submit_btn': '_onSubmitClick',
    },

    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        if (this.editableMode) {
            return def;
        }

        // Warning: Do not activate any option that adds inline style.
        // Because the style is deleted after save.
        var toolbar = [
            ['style', ['style']],
            ['font', ['bold', 'italic', 'underline', 'clear']],
            ['para', ['ul', 'ol', 'paragraph']],
            ['table', ['table']],
            ['insert', ['link', 'picture']],
            ['history', ['undo', 'redo']],
        ];

        var $textarea = this.$('textarea.o_wysiwyg_loader');
        var loadProm = wysiwygLoader.load(this, $textarea[0], {
            toolbar: toolbar,
            recordInfo: {
                context: this._getContext(),
                res_model: 'res.users',
                res_id: parseInt(this.$('input[name=user_id]').val()),
            },
            disableResizeImage: true,
        }).then(wysiwyg => {
            this._wysiwyg = wysiwyg;
        });

        return Promise.all([def, loadProm]);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onEditProfilePicClick: function (ev) {
        ev.preventDefault();
        $(ev.currentTarget).closest('form').find('.o_forum_file_upload').trigger('click');
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onFileUploadChange: function (ev) {
        if (!ev.currentTarget.files.length) {
            return;
        }
        var $form = $(ev.currentTarget).closest('form');
        var reader = new window.FileReader();
        reader.readAsDataURL(ev.currentTarget.files[0]);
        reader.onload = function (ev) {
            $form.find('.o_forum_avatar_img').attr('src', ev.target.result);
        };
        $form.find('#forum_clear_image').remove();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onProfilePicClearClick: function (ev) {
        var $form = $(ev.currentTarget).closest('form');
        $form.find('.o_forum_avatar_img').attr('src', '/web/static/src/img/placeholder.png');
        $form.append($('<input/>', {
            name: 'clear_image',
            id: 'forum_clear_image',
            type: 'hidden',
        }));
    },
    /**
     * @private
     */
    _onSubmitClick: function () {
        if (this._wysiwyg) {
            this._wysiwyg.save();
        }
    },
});

return publicWidget.registry.websiteProfile;

});

```

## File: views\gamification_badge_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="gamification_badge_view_form" model="ir.ui.view">
        <field name="name">gamification.badge.view.form.inherit.website</field>
        <field name="model">gamification.badge</field>
        <field name="inherit_id" ref="gamification.badge_form_view"/>
        <field name="arch" type="xml">
        	<xpath expr="//div[@name='button_box']" position="inside">
                <button name="website_publish_button" type="object" class="oe_stat_button" icon="fa-globe">
                    <field name="is_published" widget="website_publish_button"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_profile.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <template id="assets_frontend" inherit_id="website.assets_frontend">
        <xpath expr="link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/website_profile/static/src/scss/website_profile.scss"/>
        </xpath>
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/website_profile/static/src/js/website_profile.js"/>
        </xpath>
    </template>

    <!-- Sub nav -->
    <template id="user_profile_sub_nav" name="User profile subnav">
        <div class="o_wprofile_all_users_nav">
            <div class="container">
                <div class="row align-items-center justify-content-between">
                    <!-- Desktop Mode -->
                    <nav aria-label="breadcrumb" class="col d-none d-md-flex">
                        <ol class="breadcrumb bg-transparent mb-0 pl-0 py-0">
                            <li t-attf-class="breadcrumb-item #{'active' if not view_user else ''}">
                                <a href="/profile/users">Users</a>
                            </li>
                            <li t-if="view_user" class="breadcrumb-item active">
                                <a><t t-esc="view_user"/></a>
                            </li>
                        </ol>
                    </nav>

                    <div class="col d-none d-md-flex flex-row align-items-center justify-content-end">
                        <!-- search -->
                        <form t-attf-action="/profile/users" role="search" method="get">
                            <div class="input-group o_wprofile_course_nav_search ml-1 position-relative">
                                <span class="input-group-prepend">
                                    <button class="btn btn-link text-white rounded-0 pr-1" type="submit" aria-label="Search" title="Search">
                                        <i class="fa fa-search"></i>
                                    </button>
                                </span>
                                <input type="text" class="form-control border-0 rounded-0 bg-transparent text-white" name="search" placeholder="Search users"/>
                            </div>
                        </form>
                    </div>

                    <!-- Mobile Mode -->
                    <div class="col d-md-none py-1 o_wprofile_user_profile_sub_nav_mobile_col">
                        <div class="btn-group w-100 position-relative" role="group" aria-label="Mobile sub-nav">
                            <div class="btn-group w-100 ml-2">
                                <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">Nav</a>

                                <ul class="dropdown-menu">
                                    <a class="dropdown-item" t-att-href="home_url or '/'">Home</a>
                                    <a class="dropdown-item" href="/profile/users">&#9492; Users</a>
                                    <a t-if="view_user" class="dropdown-item">&#9492; <t t-esc="view_user"/></a>
                                </ul>
                            </div>

                            <div class="btn-group ml-1 position-static mr-2">
                                <a class="btn bg-black-25 text-white dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false"><i class="fa fa-search"></i></a>
                                <div class="dropdown-menu dropdown-menu-right w-100" style="right: 10px;">
                                    <form class="px-3" t-attf-action="/profile/users" role="search" method="get">
                                        <div class="input-group">
                                            <input type="text" class="form-control" name="search" placeholder="Search courses"/>
                                            <span class="input-group-append">
                                                <button class="btn btn-primary" type="submit" aria-label="Search" title="Search">
                                                    <i class="fa fa-search"/>
                                                </button>
                                            </span>
                                        </div>
                                    </form>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <!--
    Single User Profile Page
    -->

    <!-- Edit Profile Page -->
    <template id="user_profile_edit_main" name="Edit Profile">
        <t t-set="body_classname" t-value="'o_wprofile_body'"/>
        <t t-call="website.layout">
            <div id="wrap" class="o_wprofile_wrap">
                <div class="container pt-4 pb-5">
                    <t t-call="website_profile.user_profile_edit_content"/>
                </div>
            </div>
        </t>
    </template>

    <template id="user_profile_edit_content" name="Edit Profile">
        <h1 class="o_page_header">Edit Profile</h1>
        <div>
            <form t-attf-action="/profile/user/save" method="post" role="form" class="o_wprofile_editor_form js_website_submit_form row" enctype="multipart/form-data">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="file" class="d-none o_forum_file_upload" name="ufile" accept="image/*"/>
                <input type="hidden" name="url_param" t-att-value="request.params.get('url_param')"/>
                <div class="col-3">
                    <div class="card o_card_people">
                        <div class="card-body">
                            <img class="o_forum_avatar_img w-100 mb-3" t-att-src="website.image_url(user, 'image_128')"/>
                            <div class="text-center">
                                <a href="#" class="o_forum_profile_pic_edit btn btn-primary" aria-label="Edit">
                                    <i class="fa fa-pencil fa-1g float-sm-none float-md-left" title="Edit"></i>
                                </a>
                                <a href="#" title="Clear" aria-label="Clear" class="btn border-primary o_forum_profile_pic_clear">
                                    <i class="fa fa-trash-o float-sm-none float-md-right"></i>
                                </a>
                            </div>
                            <div class="form-group mt-3 mb-0 pt-2 border-top">
                                <label class="text-primary" for="user_website_published" t-if="user.id == uid"><span class="font-weight-bold">Public profile</span></label>
                                <div class=" mb-0 float-right" t-if="user.id == uid">
                                    <input type="checkbox" class="mt8" name="website_published" id="user_website_published" value="True" t-if="not user.website_published"/>
                                    <input type="checkbox" class="mt8" name="website_published" id="user_website_published" value="True" checked="checked" t-if="user.website_published"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-9 mb-3">
                    <div class="card">
                        <div class="card-body">
                            <div class="row">
                                <input name="user_id" t-att-value="user.id" type="hidden"/>
                                <div class="form-group col-12">
                                    <label class="text-primary mb-1 d-block" for="user_name"><span class="font-weight-bold">Real name</span></label>
                                    <div>
                                        <input type="text" class="form-control" name="name" id="user_name" required="True" t-attf-value="#{user.name}"/>
                                    </div>
                                </div>

                                <div class="form-group col-6">
                                    <label class="mb-1 text-primary" for="user_website"><span class="font-weight-bold">Website</span></label>
                                    <div>
                                        <input type="text" class="form-control" name="website" id="user_website" t-attf-value="#{user.partner_id.website or ''}"/>
                                    </div>
                                </div>
                                <div class="form-group col-6">
                                    <div t-if="email_required" class="alert alert-danger alert-dismissable oe_email_required" role="alert">
                                        <button type="button" class="close" data-dismiss="alert">x</button>
                                        <p>Please enter a valid email address in order to receive notifications from answers or comments.</p>
                                    </div>
                                    <label class="mb-1 text-primary" for="user_email"><span class="font-weight-bold">Email</span></label>
                                    <div>
                                        <input type="text" class="form-control" name="email" id="user_email" required="True" t-attf-value="#{user.partner_id.email}"/>
                                    </div>
                                </div>
                                <div class="form-group col-6">
                                    <label class="mb-1 text-primary" for="user_city"><span class="font-weight-bold">City</span></label>
                                    <div>
                                        <input type="text" class="form-control" name="city" id="user_city" t-attf-value="#{user.partner_id.city or ''}"/>
                                    </div>
                                </div>
                                <div class="form-group col-6">
                                    <label class="mb-1 text-primary"><span class="font-weight-bold">Country</span></label>
                                    <div>
                                        <select class="form-control" name="country">
                                            <option value="">Country...</option>
                                            <t t-foreach="countries or []" t-as="country">
                                                <option t-att-value="country.id" t-att-selected="country.id == user.partner_id.country_id.id"><t t-esc="country.name"/></option>
                                            </t>
                                         </select>
                                    </div>
                                </div>
                                <div class="form-group col-12">
                                    <label class="mb-1 text-primary" for="description"><span class="font-weight-bold">Biography</span></label>
                                    <textarea name="description" id="description" style="min-height: 120px"
                                        class="form-control o_wysiwyg_loader"><t t-esc="user.partner_id.website_description"/></textarea>
                                </div>
                                <div class="col">
                                    <button type="submit" class="btn btn-primary o_wprofile_submit_btn">Update</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </template>

    <!-- Profile Page -->
    <template id="user_profile_main" name="Profile Page">
        <t t-set="body_classname" t-value="'o_wprofile_body'"/>
        <t t-call="website.layout">
            <div id="wrap" class="o_wprofile_wrap mt-0">
                <t t-call="website_profile.user_profile_header"/>
                <t t-call="website_profile.user_profile_content"/>
            </div>
        </t>
    </template>

    <template id="user_profile_header" name="Profile Page Header">
        <div class="o_wprofile_header o_wprofile_gradient position-relative text-white">
            <t t-call="website_profile.user_profile_sub_nav">
                <t t-set="view_user"><t t-esc="user.name"/></t>
            </t>

            <div class="container pb-3 pb-md-0 pt-2 pt-md-5">
                <div class="row">
                    <!-- ==== Header Left ==== -->
                    <div class="col-12 col-md-4 col-lg-3">
                        <div t-attf-class="d-flex align-items-start h-100 #{'justify-content-between' if (request.env.user == user) else 'justify-content-around' }">
                            <div class="o_wprofile_pict d-inline-block mb-3 mb-md-0" t-attf-style="background-image: url(#{website.image_url(user, 'image_1024')});"/>
                            <a class="btn btn-primary d-inline-block d-md-none"
                                t-if="request.env.user == user and user.karma != 0"
                                t-attf-href="/profile/edit?url_param=#{edit_button_url_param}&amp;user_id=#{user.id}">
                                <i class="fa fa-pencil mr-1"/>EDIT
                            </a>
                        </div>
                    </div>

                    <!-- ==== Header Right ==== -->
                    <div class="col-12 col-md-8 col-lg-9 d-flex flex-column">
                        <div class="d-flex justify-content-between align-items-start">
                            <h1 class="o_card_people_name">
                                <span t-field="user.name"/><small t-if="user.karma == 0"> (not verified)</small>
                            </h1>
                            <a class="btn btn-primary d-none d-md-inline-block" t-if="request.env.user == user and user.karma != 0 or request.env.user._is_admin()" t-attf-href="/profile/edit?url_param=#{edit_button_url_param}&amp;user_id=#{user.id}">
                                <i class="fa fa-pencil mr-2"/>EDIT PROFILE
                            </a>
                        </div>

                        <div class="d-flex flex-column justify-content-center flex-grow-1 mb-0 mb-md-5">
                            <div t-if="user.partner_id.company_name" class="lead mb-2">
                                <i class="fa fa-building-o fa-fw mr-1"/><span t-field="user.partner_id.company_name"/>
                            </div>
                            <div t-if="user.city or user.country_id" class="lead mb-2">
                                <i class="fa fa-map-marker fa-fw mr-1"/>
                                <span t-field="user.city"/><span class="text-nowrap ml-1" t-if="user.country_id">(<span t-field="user.country_id"/>)</span>
                            </div>
                            <div t-if="user.website" class="lead mb-2">
                                <i class="fa fa-globe fa-fw mr-1"/><span t-field="user.website"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="user_profile_content" name="Profile Page Content">
        <div class="container">
            <div class="row">

                <!-- ========== SIDEBAR ========== -->
                <div class="col-12 col-md-4 col-lg-3 mt-3 mt-md-0">
                    <div class="o_wprofile_sidebar bg-white px-3 py-2 py-md-3 mb-3 mb-md-5">
                        <div class="o_wprofile_sidebar_top d-flex justify-content-between">
                            <div t-if="user.rank_id" class="d-flex align-items-center">
                                <small class="font-weight-bold mr-2">Current rank:</small>
                                <img t-att-src="website.image_url(user.rank_id, 'image_128')" width="16" height="16" alt="" class="o_object_fit_cover mr-1"/>
                                <a href="/profile/ranks_badges" t-field="user.rank_id"/>
                            </div>
                            <button class="btn btn-sm d-md-none bg-white border" type="button" data-toggle="collapse" data-target="#o_wprofile_sidebar_collapse" aria-expanded="false" aria-controls="o_wprofile_sidebar_collapse">More info</button>
                        </div>
                        <div class="collapse d-md-block" id="o_wprofile_sidebar_collapse">
                            <t t-set="next_rank_id" t-value="user._get_next_rank()"/>
                            <small t-if="next_rank_id" class="font-weight-bold mt-1">Next rank:</small>
                            <t t-if="next_rank_id or user.rank_id" t-call="website_profile.profile_next_rank_card">
                                <t t-set="img_max_width">40%</t>
                            </t>

                            <table class="table table-sm w-100" id="o_wprofile_sidebar_table">
                                <tbody>
                                    <tr>
                                        <th><small class="font-weight-bold">Joined</small></th>
                                        <td><span t-field="user.create_date" t-options='{"format": "d MMM Y"}'/></td>
                                    </tr>
                                    <tr>
                                        <th><small class="font-weight-bold">Badges</small></th>
                                        <td t-if="user.badge_ids" t-esc="len(user.badge_ids.filtered(lambda b: b.badge_id.website_published))"/>
                                        <td t-else="">0</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>

                <!-- ========== PROFILE CONTENT ========== -->
                <div class="col-12 col-md-8 col-lg-9">
                    <ul class="nav nav-tabs o_wprofile_nav_tabs flex-nowrap" role="tablist" id="profile_extra_info_tablist">
                        <li class="nav-item">
                            <a role="tab" aria-controls="about" href="#profile_tab_content_about" class="nav-link active" data-toggle="tab">About</a>
                        </li>
                    </ul>
                    <div class="tab-content py-4 o_wprofile_tabs_content mb-4" id="profile_extra_info_tabcontent">
                        <div role="tabpanel" class="tab-pane active" id="profile_tab_content_about">
                            <div class="o_wprofile_email_validation_container mb16 mt16">
                                <t t-call="website_profile.email_validation_banner">
                                    <t t-set="redirect_url" t-value="'/profile/user/%s' % user.id"/>
                                    <t t-set="send_validation_email_message" t-value="'Click here to send a verification email.'"/>
                                </t>
                            </div>
                            <div id="profile_about_badge" class="mb32">
                                <h5 class="border-bottom pb-1">Badges</h5>
                                <t t-call="website_profile.user_badges"></t>
                            </div>
                            <div t-if="user.partner_id.website_description" class="mb32">
                                <h5 class="border-bottom pb-1">Biography</h5>
                                <span t-field="user.partner_id.website_description"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="profile_next_rank_card" name="Profile Next Rank Card">
        <div class="o_wprofile_progress_circle">
            <svg viewBox="0 0 36 36" class="o_pc_circular_chart">
                <t t-set="next_rank_id" t-value="next_rank_id or user._get_next_rank()"/>
                <t t-if="next_rank_id and user.rank_id">
                    <t t-if="(next_rank_id.karma_min - user.rank_id.karma_min) > 0">
                        <t t-set="user_points" t-value="int(100*(user.karma - user.rank_id.karma_min)/(next_rank_id.karma_min - user.rank_id.karma_min))"/>
                    </t>
                    <t t-else="">
                        <t t-set="user_points" t-value="0"/>
                    </t>
                </t>
                <t t-elif="user.rank_id">
                    <t t-set="user_points" t-value="100"/>
                </t>
                <t t-else="">
                    <t t-set="user_points" t-value="0"/>
                </t>
                <path class="o_pc_circle_bg" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" />
                <path class="o_pc_circle" t-attf-stroke-dasharray="#{user_points}, 100" d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831" stroke="url(#gradient)" mask="url(#mask)"/>
                <mask id="mask">
                    <polygon points="0,0 17.2,0 17.2,10 18,10 18,0 36,0 36,36 0,36" fill="white"/>
                </mask>
                <linearGradient id="gradient">
                    <stop offset="0%" stop-color="var(--o-pc-color-stop-1)"/>
                    <stop offset="100%" stop-color="var(--o-pc-color-stop-2)"/>
                </linearGradient>
            </svg>
            <div class="o_pc_overlay d-flex flex-column align-items-center justify-content-center">
                <img class="img-fluid"
                    t-att-src="website.image_url(next_rank_id if next_rank_id else user.rank_id, 'image_128')"
                    t-att-alt="(next_rank_id.name if next_rank_id else user.rank_id.name) + ' badge'"
                    t-att-style="'max-width: ' + (img_max_width if img_max_width else '50%;')"/>
                <h4 class=" mb-0">
                    "
                    <span t-if="next_rank_id" t-field="next_rank_id.name"/>
                    <span t-else="" t-field="user.rank_id.name"/>
                    "
                </h4>
                <small>
                    <span class="font-weight-bold text-primary" t-field="user.karma"/>/
                    <span t-if="next_rank_id" class="font-weight-bold" t-field="next_rank_id.karma_min"/>
                    <span t-else="" class="font-weight-bold" t-field="user.rank_id.karma_min"/>
                     xp
                </small>
            </div>
        </div>
    </template>

    <template id="user_badges" name="User Bagdes">
        <div t-if="user.badge_ids" class="row mx-n1">
            <t t-foreach="user.badge_ids" t-as="badge">
                <t t-if="badge.badge_id.website_published">
                    <div class="col px-1 mb-2 col-xl-4">
                        <div class="card">
                            <div class="card-body p-2 pr-3">
                                <div class="media align-items-center">
                                  <img t-if="not badge.badge_id.image_128 and badge.level" t-attf-src="/website_profile/static/src/img/badge_#{badge.badge_id.level}.svg" class="m-1" style="height:2.5em" t-att-alt="badge.badge_id.name"/>
                                  <img t-else="" width="38" height="38" t-att-src="website.image_url(badge.badge_id, 'image_128')" class="o_object_fit_cover mr-1"/>
                                  <div class="media-body col-md-10 p-0">
                                    <h6 class="my-0 text-truncate" t-field="badge.badge_id.name"/>
                                  </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </t>
            </t>
        </div>
        <div class="mb-3" t-if="not user.badge_ids">
            <p class="text-muted">No badge yet</p>
        </div>
        <div class="text-right">
            <a t-if="not user.badge_ids and badge_category" t-attf-href="/profile/ranks_badges?badge_category=#{badge_category}"><i class="fa fa-arrow-right"/> Check available badges</a>
            <a t-else="" href="/profile/ranks_badges" class="btn btn-link btn-sm"><i class="fa fa-arrow-right mr-1"/>Check all available badges</a>
        </div>
    </template>

    <!-- About Ranks and badges Page -->
    <template id="rank_badge_main" name="Ranks Page">
        <t t-call="website.layout">
            <div class="container mb32 mt48">
                <div class="row justify-content-between" t-if="ranks">
                    <div class="col-12 col-md-6 col-lg-5">
                        <h1>Ranks</h1>
                        <p class="lead mb-4">Keep learning with <t t-esc="website.company_id.name"/>. Collect points on the forum or on the eLearning platform. Those points will make you reach new ranks.</p>
                        <h5>How do I earn badges?</h5>
                        <p>When you finish a course or reach milestones, you're awarded badges.</p>
                        <h5>How do I score more points?</h5>
                        <p>You can score more points by answering quizzes at the end of each course content. Points can also be earned on the forum. Follow this link to the guidelines of the forum.</p>
                    </div>
                    <div class="col-12 col-md-5 col-lg-4">
                        <div class="card">
                            <div class="card-header border-bottom-0">Ranks</div>
                            <ul class="list-group list-group-flush">
                                <t t-foreach="ranks" t-as="rank">
                                    <li t-attf-class="list-group-item">
                                        <div class="media align-items-center">
                                            <img t-att-src="website.image_url(rank, 'image_128')" class="mr-2 o_image_40_cover" alt="Rank badge"/>
                                            <div class="media-body">
                                                <h5 class="mt-0 mb-0" t-field="rank.name"/>
                                                <span class="badge badge-success"><span t-field="rank.karma_min"/></span> point<span t-if="rank.karma_min">s</span>
                                            </div>
                                        </div>
                                    </li>
                                </t>
                            </ul>
                        </div>
                    </div>
                </div>
                <t t-call="website_profile.badge_content"/>
            </div>
        </t>
    </template>

    <template id="badge_content" name="Badges Page content">
        <div class="row">
            <div class="col-12">
                <h1 class="mt-4 mt-lg-2">Badges</h1>
                <p class="lead">
                    Besides gaining reputation with your questions and answers,
                    you receive badges for being especially helpful.<br class="d-none d-lg-inline-block"/>Badges
                    appear on your profile page, and your posts.
                </p>
            </div>
        </div>
        <table class="table table-sm mb64">
            <tr t-foreach="badges" t-as="badge">
                <td class="align-middle">
                    <img t-if="not badge.image_1920 and badge.level" t-attf-src="/website_profile/static/src/img/badge_#{badge.level}.svg"
                         class="my-1" style="height:2.5em" t-att-alt="badge.name"/>
                    <img t-else="" t-att-src="website.image_url(badge, 'image_1024')" class="my-1" style="height:2.5em" t-att-alt="badge.name"/>
                    <h6 t-field="badge.name" class="d-inline my-0"/>
                </td>
                <td class="align-middle d-none d-md-table-cell">
                    <b t-esc="badge.stat_count_distinct"/>
                    <i class="text-muted"> awarded users</i>
                </td>
                <td class="align-middle">
                    <span t-field="badge.description"/>
                </td>
            </tr>
        </table>
    </template>

    <!--Private profile-->
    <template id="private_profile" name="Private Profile Page">
        <t t-call="website.layout">
            <div class="container mb32 mt48">
                <h1 class="mt32">This profile is private!</h1>
                <div id="private_profile_return_link_container">
                    <p><a t-attf-href="/">Return to the website.</a></p>
                </div>
            </div>
        </t>
    </template>

    <!--
    All Users Page
    -->
    <template id="users_page_main" name="Users Page">
        <t t-set="body_classname" t-value="'o_wprofile_body'"/>
         <t t-call="website.layout">
            <div id="wrap" class="o_wprofile_wrap mt-0 pb-5">
                <t t-call="website_profile.users_page_header"/>
                <t t-call="website_profile.users_page_content"/>
            </div>
        </t>
    </template>

    <template id="users_page_header" name="Users Page Header">
        <div class="o_wprofile_all_users_header o_wprofile_gradient mb-n5 pb-5">
            <t t-call="website_profile.user_profile_sub_nav"/>
            <div class="container">
                <h1 class="py-4 text-white">All Users</h1>
            </div>
        </div>
    </template>

    <template id="users_page_content">
        <div class="container mb32">
            <div class="row mb-3">
                <div class="col-md-4" t-foreach="top3_users" t-as="user" t-attf-onclick="location.href='/profile/user/#{user['id']}';">
                    <t t-call="website_profile.top3_user_card"></t>
                </div>
            </div>
            <table class="table table-sm" t-if='users'>
                <tr t-foreach="users" t-as="user" t-attf-onclick="location.href='/profile/user/#{user['id']}';" class="o_wprofile_pointer bg-white">
                    <t t-call="website_profile.all_user_card"/>
                </tr>
            </table>
            <t t-if='search'>
                <div class='alert alert-warning'>No user found for <strong><t t-esc="search"/></strong>. Try another search.</div>
            </t>
            <div class="form-inline justify-content-center">
                <t t-call="website_profile.pager_nobox"/>
            </div>
        </div>
    </template>

    <template id="top3_user_card" name="Top 3 User Card">
        <div class="card text-center mb-2 border-bottom-0 o_wprofile_pointer">
            <div class="card-body">
                <div class="d-inline-block position-relative">
                    <img class="rounded-circle img-fluid"
                        style="width: 128px; height: 128px; object-fit: cover;"
                        t-att-src="'/profile/avatar/%s?field=image_256%s' % (user['id'], '&amp;res_model=%s&amp;res_id=%s' % (record._name, record.id) if record else '')"/>
                    <img class="position-absolute" t-attf-src="/website_profile/static/src/img/rank_#{user_index + 1}.svg" alt="User rank" style="bottom: 0; right: -10px"/>
                </div>
                <h3 class="mt-2 mb-0" t-esc="user['name']"></h3>
                <span class="badge badge-danger font-weight-normal px-2" t-if="not user['website_published']">Unpublished</span>
                <strong class="text-muted" t-esc="user['rank']"/>
            </div>
            <div class="row mx-0 o_wprofile_top3_card_footer text-nowrap">
                <div class="col py-3"><b t-esc="user['karma']"/> <span class="text-muted">XP</span></div>
                <div class="col py-3"><b t-esc="user['badge_count']"/> <span class="text-muted">Badges</span></div>
            </div>
        </div>
    </template>

    <template id="all_user_card" name="All User Card">
        <td class="align-middle text-right text-muted" style="width: 0">
            <span t-esc="user['position']"/>
        </td>
        <td class="align-middle d-none d-sm-table-cell">
            <img class="o_object_fit_cover rounded-circle o_wprofile_img_small" width="30" height="30" t-att-src="'/profile/avatar/%s?field=image_128%s' % (user['id'], '&amp;res_model=%s&amp;res_id=%s' % (record._name, record.id) if record else '')"/>
        </td>
        <td class="align-middle w-md-75">
            <span class="font-weight-bold" t-esc="user['name']"/><br/>
            <span class="text-muted font-weight-bold" t-esc="user['rank']"></span>
        </td>
         <td t-if="not user['website_published']" class="align-middle font-weight-bold text-right text-nowrap">
            <span class="badge badge-danger font-weight-normal px-2 py-1 m-1">Unpublished</span>
        </td>
        <td class="align-middle font-weight-bold text-right text-nowrap">
            <b t-esc="user['karma']"/> <span class="text-muted small font-weight-bold">XP</span>
        </td>
        <td class="align-middle font-weight-bold text-right pr-3 text-nowrap all_user_badge_count">
            <b t-esc="user['badge_count']"/> <span class="text-muted small font-weight-bold">Badges</span>
        </td>
    </template>

    <!-- Custom Pager: lighter than existing one, no box around number, first / end displayed -->
    <template id="pager_nobox" name="Pager (not box display)">
        <ul t-if="pager['page_count'] > 1" t-attf-class="o_wprofile_pager font-weight-bold pagination m-0">
            <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == 1 else ''}">
                <a t-att-href=" pager['page_first']['url'] if pager['page']['num'] != 1 else None" class="page-link"><i class="fa fa-step-backward"/></a>
            </li>
            <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == 1 else ''}">
                <a t-att-href=" pager['page_previous']['url'] if pager['page']['num'] != 1 else None" class="page-link"><i class="fa fa-caret-left"/></a>
            </li>
            <t t-foreach="pager['pages']" t-as="page">
                <li t-attf-class="page-item #{'active disabled bg-primary rounded-circle' if page['num'] == pager['page']['num'] else ''}"> <a t-att-href="page['url']" class="page-link" t-raw="page['num']"></a></li>
            </t>
            <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == pager['page_count'] else ''}">
                <a t-att-href="pager['page_next']['url'] if pager['page']['num'] != pager['page_count'] else None" class="page-link"><i class="fa fa-caret-right"/></a>
            </li>
            <li t-attf-class="page-item o_wprofile_pager_arrow #{'disabled' if pager['page']['num'] == pager['page_count'] else ''}">
                <a t-att-href=" pager['page_last']['url'] if pager['page']['num'] != pager['page_count'] else None" class="page-link"><i class="fa fa-step-forward"/></a>
            </li>
        </ul>
    </template>

    <template id="email_validation_banner">
        <t t-set="send_alert_classes" t-value="send_alert_classes if send_alert_classes else 'alert alert-danger alert-dismissable'"/>
        <t t-set="done_alert_classes" t-value="done_alert_classes if done_alert_classes else 'alert alert-success alert-dismissable'"/>

        <div t-if="not validation_email_sent and not is_public_user and user.karma == 0" t-att-class="send_alert_classes" role="alert">
            <button type="button" class="close validation_email_close" data-dismiss="alert" aria-label="Close">&amp;times;</button>
            It appears your email has not been verified.<br/>
            <a class="send_validation_email alert-link" href="#" t-att-data-redirect_url="redirect_url">
                <span t-esc="send_validation_email_message"/>
            </a>
        </div>
        <div t-if="validation_email_done" t-att-class="done_alert_classes" role="status">
            <button type="button" class="close validated_email_close" data-dismiss="alert" aria-label="Close">&amp;times;</button>
            <span id="email_validated_message">Congratulations! Your email has just been validated.</span>
            <span t-esc="additional_validated_email_message"/>
        </div>
    </template>

</data></odoo>

```

