# Odoo Module: website_mass_mailing

Category: Website/Website

This file contains the source code of the Odoo module.

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
    'name': 'Newsletter Subscribe Button',
    'summary': 'Attract visitors to subscribe to mailing lists',
    'description': """
This module brings a new building block with a mailing list widget to drop on any page of your website.
On a simple click, your visitors can subscribe to mailing lists managed in the Email Marketing app.
    """,
    'version': '1.0',
    'category': 'Website/Website',
    'depends': ['website', 'mass_mailing'],
    'data': [
        'security/ir.model.access.csv',
        'views/website_mass_mailing_templates.xml',
        'views/snippets_templates.xml',
        'views/mailing_list_views.xml',
        'views/website_mass_mailing_views.xml',
    ],
    'qweb': [
        'static/src/xml/*.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.http import route, request
from odoo.osv import expression
from odoo.addons.mass_mailing.controllers.main import MassMailController


class MassMailController(MassMailController):

    @route('/website_mass_mailing/is_subscriber', type='json', website=True, auth="public")
    def is_subscriber(self, list_id, **post):
        email = None
        if not request.env.user._is_public():
            email = request.env.user.email
        elif request.session.get('mass_mailing_email'):
            email = request.session['mass_mailing_email']

        is_subscriber = False
        if email:
            contacts_count = request.env['mailing.contact.subscription'].sudo().search_count([('list_id', 'in', [int(list_id)]), ('contact_id.email', '=', email), ('opt_out', '=', False)])
            is_subscriber = contacts_count > 0

        return {'is_subscriber': is_subscriber, 'email': email}

    @route('/website_mass_mailing/subscribe', type='json', website=True, auth="public")
    def subscribe(self, list_id, email, **post):
        # FIXME the 14.0 was released with this but without the google_recaptcha
        # module being added as a dependency of the website_mass_mailing module.
        # This is to be fixed in master of course but in stable, we'll have to
        # use this workaround.
        if hasattr(request.env['ir.http'], '_verify_request_recaptcha_token') \
                and not request.env['ir.http']._verify_request_recaptcha_token('website_mass_mailing_subscribe'):
            return {
                'toast_type': 'danger',
                'toast_content': _("Suspicious activity detected by Google reCaptcha."),
            }
        ContactSubscription = request.env['mailing.contact.subscription'].sudo()
        Contacts = request.env['mailing.contact'].sudo()
        name, email = Contacts.get_name_email(email)

        subscription = ContactSubscription.search([('list_id', '=', int(list_id)), ('contact_id.email', '=', email)], limit=1)
        if not subscription:
            # inline add_to_list as we've already called half of it
            contact_id = Contacts.search([('email', '=', email)], limit=1)
            if not contact_id:
                contact_id = Contacts.create({'name': name, 'email': email})
            ContactSubscription.create({'contact_id': contact_id.id, 'list_id': int(list_id)})
        elif subscription.opt_out:
            subscription.opt_out = False
        # add email to session
        request.session['mass_mailing_email'] = email
        mass_mailing_list = request.env['mailing.list'].sudo().browse(list_id)
        return {
            'toast_type': 'success',
            'toast_content': mass_mailing_list.toast_content
        }

    @route(['/website_mass_mailing/get_content'], type='json', website=True, auth="public")
    def get_mass_mailing_content(self, newsletter_id, **post):
        PopupModel = request.env['website.mass_mailing.popup'].sudo()
        data = self.is_subscriber(newsletter_id, **post)
        domain = expression.AND([request.website.website_domain(), [('mailing_list_id', '=', newsletter_id)]])
        mass_mailing_popup = PopupModel.search(domain, limit=1)
        if mass_mailing_popup:
            data['popup_content'] = mass_mailing_popup.popup_content
        else:
            data.update(PopupModel.default_get(['popup_content']))
        return data

    @route(['/website_mass_mailing/set_content'], type='json', website=True, auth="user")
    def set_mass_mailing_content(self, newsletter_id, content, **post):
        PopupModel = request.env['website.mass_mailing.popup']
        domain = expression.AND([request.website.website_domain(), [('mailing_list_id', '=', newsletter_id)]])
        mass_mailing_popup = PopupModel.search(domain, limit=1)
        if mass_mailing_popup:
            mass_mailing_popup.write({'popup_content': content})
        else:
            PopupModel.create({
                'mailing_list_id': newsletter_id,
                'popup_content': content,
                'website_id': request.website.id,
            })
        return True

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.tools.translate import _


class MailingList(models.Model):
    _inherit = 'mailing.list'

    def _default_toast_content(self):
        return _('<p>Thanks for subscribing!</p>')

    website_popup_ids = fields.One2many('website.mass_mailing.popup', 'mailing_list_id', string="Website Popups")
    toast_content = fields.Html(default=_default_toast_content, translate=True)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCompany(models.Model):
    _inherit = "res.company"

    def _get_social_media_links(self):
        social_media_links = super()._get_social_media_links()
        website_id = self.env['website'].get_current_website()
        social_media_links.update({
            'social_facebook': website_id.social_facebook or social_media_links.get('social_facebook'),
            'social_linkedin': website_id.social_linkedin or social_media_links.get('social_linkedin'),
            'social_twitter': website_id.social_twitter or social_media_links.get('social_twitter'),
            'social_instagram': website_id.social_instagram or social_media_links.get('social_instagram')
        })
        return social_media_links

```

## File: models\website_mass_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MassMailingPopup(models.Model):
    _name = 'website.mass_mailing.popup'
    _description = "Mailing list popup"

    def _default_popup_content(self):
        return self.env['ir.ui.view']._render_template('website_mass_mailing.s_newsletter_subscribe_popup_content')

    mailing_list_id = fields.Many2one('mailing.list')
    website_id = fields.Many2one('website')
    popup_content = fields.Html(string="Website Popup Content", default=_default_popup_content, translate=True, sanitize=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_list
from . import website_mass_mailing
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mass_mailing_popup,website.mail.popup,model_website_mass_mailing_popup,mass_mailing.group_mass_mailing_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M29.423 69H4c-2 0-4-.146-4-4.078V40.993l22.445-18.95 7.33-3.392H53.22l1.443 1.761.245 15.702L29.423 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M48.592 44.35c-.31.781-.682 1.543-1.116 2.285a16.757 16.757 0 0 1-6.14 6.112C38.753 54.25 35.93 55 32.87 55c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 44.06 16 41.25 16 38.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c1.72-1 3.546-1.667 5.478-2.001v4.28l-.042-.005c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.092.058.176.107.25.147v.302h1.75c.025.027.05.056.076.087.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a26.286 26.286 0 0 1-.017-.02h14.246zm-13.285 6.632c1.95-.34 3.714-1.036 5.291-2.088l-3.676-3.675c-.105.08-.168.135-.187.164a1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35zM55 27.7v10.785c0 .598-.201 1.11-.603 1.535-.402.426-.886.639-1.45.639H34.053c-.565 0-1.049-.213-1.45-.639A2.156 2.156 0 0 1 32 38.486V27.701c.376.443.809.837 1.296 1.181 3.098 2.228 5.224 3.79 6.38 4.687.487.38.883.677 1.186.89.304.212.708.43 1.213.652.505.221.976.332 1.412.332h.026c.436 0 .907-.11 1.412-.332.505-.222.909-.44 1.213-.652.303-.213.7-.51 1.187-.89 1.454-1.114 3.585-2.676 6.392-4.687A7.216 7.216 0 0 0 55 27.701zm0-4.177c0 .715-.21 1.399-.629 2.051a6.297 6.297 0 0 1-1.566 1.67c-3.217 2.364-5.22 3.836-6.006 4.416-.086.063-.268.201-.546.414-.278.213-.509.385-.693.516-.184.131-.406.279-.667.442a3.793 3.793 0 0 1-.738.366 1.94 1.94 0 0 1-.642.123h-.026c-.197 0-.41-.041-.642-.123a3.793 3.793 0 0 1-.738-.366c-.26-.163-.483-.31-.667-.442a26.904 26.904 0 0 1-.693-.516c-.278-.213-.46-.351-.546-.414-.778-.58-1.9-1.406-3.362-2.48a592.34 592.34 0 0 1-2.631-1.935c-.53-.38-1.031-.904-1.502-1.57-.47-.665-.706-1.283-.706-1.853 0-.707.178-1.295.533-1.766.355-.471.862-.707 1.52-.707h18.893c.557 0 1.038.213 1.444.639.407.425.61.937.61 1.535z" opacity=".3"/><path fill="#FFF" d="M48.592 41.35c-.31.781-.682 1.543-1.116 2.285a16.757 16.757 0 0 1-6.14 6.112C38.753 51.25 35.93 52 32.87 52c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 41.06 16 38.25 16 35.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c1.72-1 3.546-1.667 5.478-2.001v4.28l-.042-.005c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.092.058.176.107.25.147v.302h1.75c.025.027.05.056.076.087.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a26.286 26.286 0 0 1-.017-.02h14.246zm-13.285 6.632c1.95-.34 3.714-1.036 5.291-2.088l-3.676-3.675c-.105.08-.168.135-.187.164a1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35zM55 24.7v10.785c0 .598-.201 1.11-.603 1.535-.402.426-.886.639-1.45.639H34.053c-.565 0-1.049-.213-1.45-.639A2.156 2.156 0 0 1 32 35.486V24.701c.376.443.809.837 1.296 1.181 3.098 2.228 5.224 3.79 6.38 4.687.487.38.883.677 1.186.89.304.212.708.43 1.213.652.505.221.976.332 1.412.332h.026c.436 0 .907-.11 1.412-.332.505-.222.909-.44 1.213-.652.303-.213.7-.51 1.187-.89 1.454-1.114 3.585-2.676 6.392-4.687A7.216 7.216 0 0 0 55 24.701zm0-4.177c0 .715-.21 1.399-.629 2.051a6.297 6.297 0 0 1-1.566 1.67c-3.217 2.364-5.22 3.836-6.006 4.416-.086.063-.268.201-.546.414-.278.213-.509.385-.693.516-.184.131-.406.279-.667.442a3.793 3.793 0 0 1-.738.366 1.94 1.94 0 0 1-.642.123h-.026c-.197 0-.41-.041-.642-.123a3.793 3.793 0 0 1-.738-.366c-.26-.163-.483-.31-.667-.442a26.904 26.904 0 0 1-.693-.516c-.278-.213-.46-.351-.546-.414-.778-.58-1.9-1.406-3.362-2.48a592.34 592.34 0 0 1-2.631-1.935c-.53-.38-1.031-.904-1.502-1.57-.47-.665-.706-1.283-.706-1.853 0-.707.178-1.295.533-1.766.355-.471.862-.707 1.52-.707h18.893c.557 0 1.038.213 1.444.639.407.425.61.937.61 1.535z"/></g></g></svg>
```

## File: static\src\img\snippets_thumbs\s_newsletter_block.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="44.579%" y2="55.421%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="22" height="2" x="33" y="7"/>
    <filter id="filter-3" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M13 8v10a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1V8a1 1 0 0 0-1-1H14a1 1 0 0 0-1 1zm7.445 3.63L15 8h12l-5.445 3.63a1 1 0 0 1-1.11 0zM14 18V8.5l6.428 4.485a1 1 0 0 0 1.144 0L28 8.5V18H14z"/>
    <filter id="filter-5" width="106.2%" height="116.7%" x="-3.1%" y="-4.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <rect id="path-6" width="13" height="1" x="33" y="12"/>
    <filter id="filter-7" width="107.7%" height="300%" x="-3.8%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-8" width="10" height="1" x="33" y="15"/>
    <filter id="filter-9" width="110%" height="300%" x="-5%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <path id="path-10" d="M72 23.571V22.43a.55.55 0 0 0-.17-.402.55.55 0 0 0-.401-.17h-2.286v-2.286a.55.55 0 0 0-.17-.401.55.55 0 0 0-.402-.17H67.43a.55.55 0 0 0-.402.17.55.55 0 0 0-.17.401v2.286h-2.286a.55.55 0 0 0-.401.17.55.55 0 0 0-.17.402v1.142a.55.55 0 0 0 .17.402.55.55 0 0 0 .401.17h2.286v2.286a.55.55 0 0 0 .17.401.55.55 0 0 0 .402.17h1.142a.55.55 0 0 0 .402-.17.55.55 0 0 0 .17-.401v-2.286h2.286a.55.55 0 0 0 .401-.17.55.55 0 0 0 .17-.402zM75 23c0 1.27-.313 2.441-.939 3.514a6.969 6.969 0 0 1-2.547 2.547A6.848 6.848 0 0 1 68 30a6.848 6.848 0 0 1-3.514-.939 6.969 6.969 0 0 1-2.547-2.547A6.848 6.848 0 0 1 61 23c0-1.27.313-2.441.939-3.514a6.969 6.969 0 0 1 2.547-2.547A6.848 6.848 0 0 1 68 16c1.27 0 2.441.313 3.514.939a6.969 6.969 0 0 1 2.547 2.547A6.848 6.848 0 0 1 75 23z"/>
    <filter id="filter-12" width="107.1%" height="114.3%" x="-3.6%" y="-3.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_newsletter_block">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(0 16)">
        <g fill="url(#linearGradient-1)" class="image_1" opacity=".4">
          <rect width="82" height="27" class="rectangle"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-4"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-6"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-8"/>
        </g>
        <mask id="mask-11" fill="#fff">
          <use xlink:href="#path-10"/>
        </mask>
        <g class="plus_circle">
          <use fill="#000" filter="url(#filter-12)" xlink:href="#path-10"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-10"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\website_mass_mailing.editor.js

```javascript
odoo.define('website_mass_mailing.editor', function (require) {
'use strict';

var core = require('web.core');
var rpc = require('web.rpc');
var WysiwygMultizone = require('web_editor.wysiwyg.multizone');
var WysiwygTranslate = require('web_editor.wysiwyg.multizone.translate');
var options = require('web_editor.snippets.options');
var wUtils = require('website.utils');

const qweb = core.qweb;
var _t = core._t;


options.registry.mailing_list_subscribe = options.Class.extend({
    popup_template_id: "editor_new_mailing_list_subscribe_button",
    popup_title: _t("Add a Newsletter Subscribe Button"),

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Allows to select mailing list.
     *
     * @see this.selectClass for parameters
     */
    select_mailing_list: function (previewMode, value) {
        var self = this;
        var def = wUtils.prompt({
            'id': this.popup_template_id,
            'window_title': this.popup_title,
            'select': _t("Newsletter"),
            'init': function (field, dialog) {
                return rpc.query({
                    model: 'mailing.list',
                    method: 'name_search',
                    args: ['', [['is_public', '=', true]]],
                    context: self.options.recordInfo.context,
                }).then(function (data) {
                    $(dialog).find('.btn-primary').prop('disabled', !data.length);
                    var list_id = self.$target.attr("data-list-id");
                    $(dialog).on('show.bs.modal', function () {
                        if (list_id !== "0"){
                            $(dialog).find('select').val(list_id);
                        };
                    });
                    return data;
                });
            },
        });
        def.then(function (result) {
            self.$target.attr("data-list-id", result.val);
        });
        return def;
    },
    /**
     * @see this.selectClass for parameters
     */
    toggleThanksButton(previewMode, widgetValue, params) {
        const subscribeBtnEl = this.$target[0].querySelector('.js_subscribe_btn');
        const thanksBtnEl = this.$target[0].querySelector('.js_subscribed_btn');

        thanksBtnEl.classList.toggle('o_disable_preview', !widgetValue);
        thanksBtnEl.classList.toggle('o_enable_preview', widgetValue);
        subscribeBtnEl.classList.toggle('o_enable_preview', !widgetValue);
        subscribeBtnEl.classList.toggle('o_disable_preview', widgetValue);
    },
    /**
     * @override
     */
    onBuilt: function () {
        var self = this;
        this._super();
        this.select_mailing_list('click').guardedCatch(function () {
            self.getParent()._onRemoveClick($.Event( "click" ));
        });
    },
    /**
     * @override
     */
    cleanForSave() {
        const previewClasses = ['o_disable_preview', 'o_enable_preview'];
        this.$target[0].querySelector('.js_subscribe_btn').classList.remove(...previewClasses);
        this.$target[0].querySelector('.js_subscribed_btn').classList.remove(...previewClasses);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        if (methodName !== 'toggleThanksButton') {
            return this._super(...arguments);
        }
        const subscribeBtnEl = this.$target[0].querySelector('.js_subscribe_btn');
        return subscribeBtnEl && subscribeBtnEl.classList.contains('o_disable_preview') ?
            'true' : '';
    },
    /**
     * @override
     */
    _renderCustomXML(uiFragment) {
        const checkboxEl = document.createElement('we-checkbox');
        checkboxEl.setAttribute('string', _t("Display Thanks Button"));
        checkboxEl.dataset.toggleThanksButton = 'true';
        checkboxEl.dataset.noPreview = 'true';
        // Prevent this option from triggering a refresh of the public widget.
        checkboxEl.dataset.noWidgetRefresh = 'true';
        uiFragment.appendChild(checkboxEl);
    },
});

options.registry.recaptchaSubscribe = options.Class.extend({
    xmlDependencies: ['/google_recaptcha/static/src/xml/recaptcha.xml'],

    /**
     * Toggle the recaptcha legal terms
     */
    toggleRecaptchaLegal: function (previewMode, value, params) {
        const recaptchaLegalEl = this.$target[0].querySelector('.o_recaptcha_legal_terms');
        if (recaptchaLegalEl) {
            recaptchaLegalEl.remove();
        } else {
            const template = document.createElement('template');
            template.innerHTML = qweb.render("google_recaptcha.recaptcha_legal_terms");
            this.$target[0].appendChild(template.content.firstElementChild);
        }
    },

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'toggleRecaptchaLegal':
                return !this.$target[0].querySelector('.o_recaptcha_legal_terms') || '';
        }
        return this._super(...arguments);
    },
});

options.registry.newsletter_popup = options.registry.mailing_list_subscribe.extend({
    popup_template_id: "editor_new_mailing_list_subscribe_popup",
    popup_title: _t("Add a Newsletter Subscribe Popup"),

    /**
     * @override
     */
    start: function () {
        this.$target.on('hidden.bs.modal.newsletter_popup_option', () => {
            this.trigger_up('snippet_option_visibility_update', {show: false});
        });
        return this._super(...arguments);
    },
    /**
     * @override
     */
    onTargetShow: function () {
        // Open the modal
        this.$target.data('quick-open', true);
        return this._refreshPublicWidgets();
    },
    /**
     * @override
     */
    onTargetHide: function () {
        // Close the modal
        const $modal = this.$('.modal');
        if ($modal.length && $modal.is('.modal_shown')) {
            $modal.modal('hide');
        }
    },
    /**
     * @override
     */
    cleanForSave: function () {
        var self = this;
        var content = this.$target.data('content');
        if (content) {
            const $layout = $('<div/>', {html: content});
            const previewClasses = ['o_disable_preview', 'o_enable_preview'];
            $layout[0].querySelector('.js_subscribe_btn').classList.remove(...previewClasses);
            $layout[0].querySelector('.js_subscribed_btn').classList.remove(...previewClasses);
            this.trigger_up('get_clean_html', {
                $layout: $layout,
                callback: function (html) {
                    self.$target.data('content', html);
                },
            });
        }
    },
    /**
     * @override
     */
    destroy: function () {
        this.$target.off('.newsletter_popup_option');
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    select_mailing_list: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$target.data('quick-open', true);
            self.$target.removeData('content');
            return self._refreshPublicWidgets();
        });
    },
});

WysiwygMultizone.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _saveElement: function (outerHTML, recordInfo, editable) {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        var $popups = $(editable).find('.o_newsletter_popup');
        _.each($popups, function (popup) {
            var $popup = $(popup);
            var content = $popup.data('content');
            if (content) {
                defs.push(self._rpc({
                    route: '/website_mass_mailing/set_content',
                    params: {
                        'newsletter_id': parseInt($popup.attr('data-list-id')),
                        'content': content,
                    },
                }));
            }
        });
        return Promise.all(defs);
    },
});

WysiwygTranslate.include({
    /**
     * @override
     */
    start: function () {
        this.$target.on('click.newsletter_popup_option', '.o_edit_popup', function (ev) {
            alert(_t('Website popups can only be translated through mailing list configuration in the Email Marketing app.'));
        });
        this._super.apply(this, arguments);
    },
});

options.registry.Parallax.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _computeVisibility() {
        // Hides parallax options for snippets that are inside a "Newsletter"
        // popup because the parallax effect is not working in this case. This
        // is due to the scrollbar which is on the "modal-body" element and not
        // on the "modal" element.
        // TODO in master: Make sure that the scrollbar is in the same place as
        // for the "s_popup" snippet for the "Newsletter" popup and make the
        // parallax effect work.
        if (this.$target[0].closest('.o_newsletter_popup')) {
            return false;
        }
        return this._super.apply(this, arguments);
    },
});

});

```

## File: static\src\js\website_mass_mailing.js

```javascript
odoo.define('mass_mailing.website_integration', function (require) {
"use strict";

var config = require('web.config');
var core = require('web.core');
const dom = require('web.dom');
var Dialog = require('web.Dialog');
var utils = require('web.utils');
var publicWidget = require('web.public.widget');
const session = require('web.session');

// FIXME the 14.0 was released with this but without the google_recaptcha
// module being added as a dependency of the website_mass_mailing module. This
// is to be fixed in master of course but in stable, we'll have to use a
// workaround.
// const {ReCaptcha} = require('google_recaptcha.ReCaptchaV3');

var _t = core._t;

let alertReCaptchaDisplayed;

publicWidget.registry.subscribe = publicWidget.Widget.extend({
    selector: ".js_subscribe",
    disabledInEditableMode: false,
    read_events: {
        'click .js_subscribe_btn': '_onSubscribeClick',
    },

    /**
     * @constructor
     */
    init: function () {
        this._super(...arguments);
        const ReCaptchaService = odoo.__DEBUG__.services['google_recaptcha.ReCaptchaV3'];
        this._recaptcha = ReCaptchaService && new ReCaptchaService.ReCaptcha() || null;
    },
    /**
     * @override
     */
    willStart: function () {
        if (this._recaptcha) {
            this._recaptcha.loadLibs();
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        if (!this._recaptcha && this.editableMode && session.is_admin && !alertReCaptchaDisplayed) {
            this.displayNotification({
                type: 'info',
                message: _t("Do you want to install Google reCAPTCHA to secure your newsletter subscriptions?"),
                sticky: true,
                buttons: [{text: _t("Install now"), primary: true, click: async () => {
                    dom.addButtonLoadingEffect($('.o_notification .btn-primary')[0]);

                    const record = await this._rpc({
                        model: 'ir.module.module',
                        method: 'search_read',
                        domain: [['name', '=', 'google_recaptcha']],
                        fields: ['id'],
                        limit: 1,
                    });
                    await this._rpc({
                        model: 'ir.module.module',
                        method: 'button_immediate_install',
                        args: [[record[0]['id']]],
                    });

                    this.displayNotification({
                        type: 'info',
                        message: _t("Google reCAPTCHA is now installed! You can configure it from your website settings."),
                        sticky: true,
                        buttons: [{text: _t("Website settings"), primary: true, click: async () => {
                            window.open('/web#action=website.action_website_configuration', '_blank');
                        }}],
                    });
                }}],
            });
            alertReCaptchaDisplayed = true;
        }

        this.$popup = this.$target.closest('.o_newsletter_modal');
        if (this.$popup.length) {
            // No need to check whether the user subscribed or not if the input
            // is in a popup as the popup won't open if he did subscribe.
            return def;
        }

        if (this.editableMode) {
            // Since there is an editor option to choose whether "Thanks" button
            // should be visible or not, we should not vary its visibility here.
            return def;
        }
        const always = this._updateView.bind(this);
        return Promise.all([def, this._rpc({
            route: '/website_mass_mailing/is_subscriber',
            params: {
                'list_id': this.$target.data('list-id'),
            },
        }).then(always).guardedCatch(always)]);
    },
    /**
     * @override
     */
    destroy() {
        this._updateView({is_subscriber: false});
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Modifies the elements to have the view of a subscriber/non-subscriber.
     *
     * @param {Object} data
     */
    _updateView(data) {
        const isSubscriber = data.is_subscriber;
        const subscribeBtnEl = this.$target[0].querySelector('.js_subscribe_btn');
        const thanksBtnEl = this.$target[0].querySelector('.js_subscribed_btn');
        const emailInputEl = this.$target[0].querySelector('input.js_subscribe_email');

        subscribeBtnEl.disabled = isSubscriber;
        emailInputEl.value = data.email || '';
        emailInputEl.disabled = isSubscriber;
        // Compat: remove d-none for DBs that have the button saved with it.
        this.$target[0].classList.remove('d-none');

        subscribeBtnEl.classList.toggle('d-none', !!isSubscriber);
        thanksBtnEl.classList.toggle('d-none', !isSubscriber);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubscribeClick: async function () {
        var self = this;
        var $email = this.$(".js_subscribe_email:visible");

        if ($email.length && !$email.val().match(/.+@.+/)) {
            this.$target.addClass('o_has_error').find('.form-control').addClass('is-invalid');
            return false;
        }
        this.$target.removeClass('o_has_error').find('.form-control').removeClass('is-invalid');
        let tokenObj = null;
        if (this._recaptcha) {
            tokenObj = await this._recaptcha.getToken('website_mass_mailing_subscribe');
            if (tokenObj.error) {
                self.displayNotification({
                    type: 'danger',
                    title: _t("Error"),
                    message: tokenObj.error,
                    sticky: true,
                });
                return false;
            }
        }
        const params = {
            'list_id': this.$target.data('list-id'),
            'email': $email.length ? $email.val() : false,
        };
        if (this._recaptcha) {
            params['recaptcha_token_response'] = tokenObj.token;
        }
        this._rpc({
            route: '/website_mass_mailing/subscribe',
            params: params,
        }).then(function (result) {
            let toastType = result.toast_type;
            if (toastType === 'success') {
                self.$(".js_subscribe_btn").addClass('d-none');
                self.$(".js_subscribed_btn").removeClass('d-none');
                self.$('input.js_subscribe_email').prop('disabled', !!result);
                if (self.$popup.length) {
                    self.$popup.modal('hide');
                }
            }
            self.displayNotification({
                type: toastType,
                title: toastType === 'success' ? _t('Success') : _t('Error'),
                message: result.toast_content,
                sticky: true,
            });
        });
    },
});

publicWidget.registry.newsletter_popup = publicWidget.Widget.extend({
    selector: ".o_newsletter_popup",
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        this.websiteID = this._getContext().website_id;
        this.listID = parseInt(this.$target.attr('data-list-id'));
        if (!this.listID || (utils.get_cookie(_.str.sprintf("newsletter-popup-%s-%s", this.listID, this.websiteID)) && !self.editableMode)) {
            return Promise.all(defs);
        }
        if (this.$target.data('content') && this.editableMode) {
            // To avoid losing user changes.
            this._dialogInit(this.$target.data('content'));
            this.$target.removeData('quick-open');
            this.massMailingPopup.open();
        } else {
            defs.push(this._rpc({
                route: '/website_mass_mailing/get_content',
                params: {
                    newsletter_id: self.listID,
                },
            }).then(function (data) {
                self._dialogInit(data.popup_content, data.email || '');
                if (!self.editableMode && !data.is_subscriber) {
                    if (config.device.isMobile) {
                        setTimeout(function () {
                            self._showBanner();
                        }, 5000);
                    } else {
                        $(document).on('mouseleave.open_popup_event', self._showBanner.bind(self));
                    }
                } else {
                    $(document).off('mouseleave.open_popup_event');
                }
                // show popup after choosing a newsletter
                if (self.$target.data('quick-open')) {
                    self.massMailingPopup.open();
                    self.$target.removeData('quick-open');
                }
            }));
        }

        return Promise.all(defs);
    },
    /**
     * @override
     */
    destroy: function () {
        if (this.massMailingPopup) {
            this.massMailingPopup.close();
        }
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @param {string} content
     * @private
     */
    _dialogInit: function (content, email) {
        var self = this;
        this.massMailingPopup = new Dialog(this, {
            technical: false,
            $content: $('<div/>').html(content),
            $parentNode: this.$target,
            backdrop: !this.editableMode,
            dialogClass: 'p-0' + (this.editableMode ? ' oe_structure oe_empty' : ''),
            renderFooter: false,
            size: 'medium',
        });
        this.massMailingPopup.opened().then(function () {
            var $modal = self.massMailingPopup.$modal;
            $modal.find('header button.close').on('mouseup', function (ev) {
                ev.stopPropagation();
            });
            $modal.addClass('o_newsletter_modal');
            $modal.find('.oe_structure').attr('data-editor-message', _t('DRAG BUILDING BLOCKS HERE'));
            $modal.find('.modal-dialog').addClass('modal-dialog-centered');
            $modal.find('.js_subscribe').data('list-id', self.listID)
                  .find('input.js_subscribe_email').val(email);
            self.trigger_up('widgets_start_request', {
                editableMode: self.editableMode,
                $target: $modal,
            });
        });
        this.massMailingPopup.on('closed', this, function () {
            var $modal = self.massMailingPopup.$modal;
            if ($modal) { // The dialog might have never been opened
                self.$el.data('content', $modal.find('.modal-body').html());
            }
        });
    },
    /**
     * @private
     */
    _showBanner: function () {
        this.massMailingPopup.open();
        utils.set_cookie(_.str.sprintf("newsletter-popup-%s-%s", this.listID, this.websiteID), true);
        $(document).off('mouseleave.open_popup_event');
    },
});
});

```

## File: static\src\xml\website_mass_mailing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
<t t-name="website_mass_mailing.edition.wrapper">
    <div class="modal fade show d-block o_newsletter_modal">
        <div role="dialog" class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <header class="modal-header">
                    <button type="button" class="close" aria-label="Close" tabindex="-1">×</button>
                </header>
                <div id="wrapper" class="modal-body p-0 oe_structure oe_empty"></div>
            </div>
        </div>
    </div>
</t>
</templates>

```

## File: views\mailing_list_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_list_view_form" model="ir.ui.view">
        <field name="name">mailing.list.view.form.inherit.website</field>
        <field name="model">mailing.list</field>
        <field name="inherit_id" ref="mass_mailing.mailing_list_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="inside">
                <notebook>
                    <page string="Website popups" name="website_popups">
                        <field name="website_popup_ids">
                            <tree>
                                <field name="website_id"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </xpath>
            <xpath expr="//div[hasclass('oe_title')]" position="after">
                <group>
                    <field name="toast_content"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="iframe_css_assets_edit" name="CSS assets for wysiwyg iframe content for popup" groups="base.group_user">
    <t t-call-assets="web.assets_common" t-js="false"/>
    <t t-call-assets="web.assets_frontend" t-js="false"/>
    <t t-call-assets="web_editor.assets_wysiwyg" t-js="false"/>
    <t t-call-assets="website.assets_editor" t-js="false"/>
</template>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@id='newsletter_snippet']" position="replace"/>
    <xpath expr="//t[@id='newsletter_popup_snippet']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//t[@id='mass_mailing_newsletter_block_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields -->
        <!-- because it contains inputs that would be removed -->
        <t t-snippet="website_mass_mailing.s_newsletter_block" t-thumbnail="/website_mass_mailing/static/src/img/snippets_thumbs/s_newsletter_block.svg" t-forbid-sanitize="form"/>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_popup_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields -->
        <!-- because it contains inputs that would be removed -->
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_popup" t-thumbnail="/website/static/src/img/snippets_thumbs/newsletter_subscribe_popup.svg" t-forbid-sanitize="form"/>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields -->
        <!-- because it contains inputs that would be removed -->
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_form" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_subscribe_form.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

<template id="s_newsletter_subscribe_form" name="Newsletter">
    <div class="s_newsletter_subscribe_form js_subscribe" data-vxml="001" data-list-id="0">
        <div class="input-group">
            <input type="email" name="email" class="js_subscribe_email form-control" placeholder="your email..."/>
            <span class="input-group-append">
                <a role="button" href="#" class="btn btn-primary js_subscribe_btn o_submit">Subscribe</a>
                <a role="button" href="#" class="btn btn-success js_subscribed_btn d-none o_submit" disabled="disabled">Thanks</a>
            </span>
        </div>
    </div>
</template>

<template id="s_newsletter_block" name="Newsletter block">
    <section class="s_newsletter_block bg-200 pt32 pb32">
        <div class="container">
            <div class="row">
                <div class="col-lg-8 offset-lg-2 pt24 pb24">
                    <h2>Always First.</h2>
                    <p>Be the first to find out all the latest news, products, and trends.</p>
                    <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_newsletter_subscribe_popup" name="Newsletter Popup">
    <div class="o_newsletter_popup o_snippet_invisible" data-list-id="0"/>
</template>

<template id="s_newsletter_subscribe_popup_content" name="Newsletter Popup Content">
    <section class="s_text_block oe_img_bg pt88 pb64" data-snippet="s_text_block"
             style="background-image: url('/web/image/website.s_cover_default_image'); background-position: 0 100%;">
        <div class="container s_allow_columns">
            <h1 style="text-align: center;">Always <b>First</b>.</h1>
            <p style="text-align: center;">Be the first to find out all the latest news,<br/> products, and trends.</p>
        </div>
    </section>
    <section class="s_text_block" data-snippet="s_text_block">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-8 offset-lg-2 pt32 pb32">
                    <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="newsletter_subscribe_options" name="Newsletter Subscribe Options" inherit_id="website.snippet_options">
    <xpath expr="//*[@t-set='so_snippet_addition_selector']" position="inside">, .o_newsletter_popup</xpath>
    <xpath expr="//div" position="after">
        <t t-set="selector" t-value="'.js_subscribe'"/>
        <div data-js="mailing_list_subscribe"
            t-att-data-selector="selector"
            t-attf-data-exclude=".o_newsletter_modal #{selector}">
            <we-button data-select_mailing_list="" data-no-preview="true">Change Newsletter</we-button>
        </div>
        <div data-js="recaptchaSubscribe"
            t-att-data-selector="selector">
            <t t-set="recaptcha_public_key" t-value="request.env['ir.config_parameter'].sudo().get_param('recaptcha_public_key')"/>
            <we-checkbox t-if="recaptcha_public_key" string="Show reCaptcha Policy" data-toggle-recaptcha-legal="" data-no-preview="true"/>
        </div>
        <div t-att-data-selector="selector" data-drop-near="p, h1, h2, h3, blockquote, .card"/>
        <div data-js="newsletter_popup"
            data-selector=".o_newsletter_popup">
            <we-button data-select_mailing_list="" data-no-preview="true">Change Newsletter</we-button>
        </div>
    </xpath>
    <xpath expr="//div[@data-js='anchor']" position="attributes">
        <attribute name="data-exclude" add=".o_newsletter_popup" separator=","/>
    </xpath>
</template>

<!-- Extend default mass_mailing snippets with website feature -->

<template id="s_mail_block_header_social" inherit_id="mass_mailing.s_mail_block_header_social">
    <xpath expr="//td[hasclass('o_mail_logo_container')]" position="after">
        <td width="30%" class="text-right o_mail_no_resize">
            <div class="o_mail_header_social">
                <t t-call="mass_mailing.social_links"/>
            </div>
        </td>
    </xpath>
</template>

<template id="s_mail_block_header_text_social" inherit_id="mass_mailing.s_mail_block_header_text_social">
    <xpath expr="//table//td" position="after">
        <td width="30%" class="text-right o_mail_no_resize">
            <div class="o_mail_header_social">
                <t t-call="mass_mailing.social_links"/>
            </div>
        </td>
    </xpath>
</template>

<template id="s_mail_block_footer_social" inherit_id="mass_mailing.s_mail_block_footer_social">
    <xpath expr="//td[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
</template>

<template id="s_mail_block_footer_social_left" inherit_id="mass_mailing.s_mail_block_footer_social_left">
    <xpath expr="//div[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
</template>

</odoo>

```

## File: views\website_mass_mailing_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_mass_mailing/static/src/scss/website_mass_mailing_popup.scss"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mass_mailing/static/src/js/website_mass_mailing.js"/>
    </xpath>
</template>

<template id="assets_editor" inherit_id="website.assets_wysiwyg">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mass_mailing/static/src/js/website_mass_mailing.editor.js"/>
    </xpath>
</template>

<template id="assets_tests" inherit_id="website.assets_tests">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mass_mailing/static/tests/tours/newsletter_popup.js"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_mass_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_mail_mass_mailing_popup_form" model="ir.ui.view">
        <field name="name">website.mass_mailing.popup.form</field>
        <field name="model">website.mass_mailing.popup</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="mailing_list_id"/>
                        <field name="website_id" groups="website.group_multi_website"/>
                    </group>
                    <label for="popup_content"/>
                    <field name="popup_content" widget="html" options="{
                            'snippets': 'website.snippets',
                            'cssEdit': 'website_mass_mailing.iframe_css_assets_edit',
                            'wrapper': 'website_mass_mailing.edition.wrapper',
                        }"/>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

