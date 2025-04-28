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
    'depends': ['website', 'mass_mailing', 'google_recaptcha'],
    'data': [
        'security/ir.model.access.csv',
        'data/ir_model_data.xml',
        'views/snippets/s_popup.xml',
        'views/snippets_templates.xml',
    ],
    'auto_install': ['website', 'mass_mailing'],
    'assets': {
        'web.assets_frontend': [
            'website_mass_mailing/static/src/scss/website_mass_mailing_popup.scss',
            'website_mass_mailing/static/src/js/website_mass_mailing.js',
            'website_mass_mailing/static/src/xml/*.xml',
        ],
        'website.assets_wysiwyg': [
            'website_mass_mailing/static/src/js/website_mass_mailing.editor.js',
            'website_mass_mailing/static/src/scss/website_mass_mailing_edit_mode.scss',
            'website_mass_mailing/static/src/snippets/s_popup/options.js',
        ],
        'website.assets_editor': [
            'website_mass_mailing/static/src/js/mass_mailing_form_editor.js',
        ],
        'web.assets_tests': [
            'website_mass_mailing/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.http import route, request
from odoo.addons.mass_mailing.controllers import main


class MassMailController(main.MassMailController):

    @route('/website_mass_mailing/is_subscriber', type='json', website=True, auth='public')
    def is_subscriber(self, list_id, subscription_type, **post):
        value = self._get_value(subscription_type)
        fname = self._get_fname(subscription_type)
        is_subscriber = False
        if value and fname:
            contacts_count = request.env['mailing.contact.subscription'].sudo().search_count(
                [('list_id', 'in', [int(list_id)]), (f'contact_id.{fname}', '=', value), ('opt_out', '=', False)])
            is_subscriber = contacts_count > 0

        return {'is_subscriber': is_subscriber, 'value': value}

    def _get_value(self, subscription_type):
        value = None
        if subscription_type == 'email':
            if not request.env.user._is_public():
                value = request.env.user.email
            elif request.session.get('mass_mailing_email'):
                value = request.session['mass_mailing_email']
        return value

    def _get_fname(self, subscription_type):
        return 'email' if subscription_type == 'email' else ''

    @route('/website_mass_mailing/subscribe', type='json', website=True, auth='public')
    def subscribe(self, list_id, value, subscription_type, **post):
        if not request.env['ir.http']._verify_request_recaptcha_token('website_mass_mailing_subscribe'):
            return {
                'toast_type': 'danger',
                'toast_content': _("Suspicious activity detected by Google reCaptcha."),
            }

        ContactSubscription = request.env['mailing.contact.subscription'].sudo()
        Contacts = request.env['mailing.contact'].sudo()
        if subscription_type == 'email':
            name, value = Contacts.get_name_email(value)
        elif subscription_type == 'mobile':
            name = value

        fname = self._get_fname(subscription_type)
        subscription = ContactSubscription.search(
            [('list_id', '=', int(list_id)), (f'contact_id.{fname}', '=', value)], limit=1)
        if not subscription:
            # inline add_to_list as we've already called half of it
            contact_id = Contacts.search([(fname, '=', value)], limit=1)
            if not contact_id:
                contact_id = Contacts.create({'name': name, fname: value})
            ContactSubscription.create({'contact_id': contact_id.id, 'list_id': int(list_id)})
        elif subscription.opt_out:
            subscription.opt_out = False
        # add email to session
        request.session[f'mass_mailing_{fname}'] = value
        return {
            'toast_type': 'success',
            'toast_content': _("Thanks for subscribing!"),
        }

```

## File: controllers\website_form.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _
from odoo.http import request
from odoo.addons.website.controllers.form import WebsiteForm


class WebsiteNewsletterForm(WebsiteForm):

    def _handle_website_form(self, model_name, **kwargs):
        if model_name == 'mailing.contact':
            list_ids = kwargs.get('list_ids')
            if not list_ids:
                return json.dumps({'error': _('Mailing List(s) not found!')})
            list_ids = [int(x) for x in list_ids.split(',')]
            private_list_ids = request.env['mailing.list'].sudo().search([
                ('id', 'in', list_ids), ('is_public', '=', False)])
            if private_list_ids:
                return json.dumps({
                    'error': _('You cannot subscribe to the following list anymore : %s',
                               ', '.join(private_list_ids.mapped('name')))
                })
        return super()._handle_website_form(model_name, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import website_form

```

## File: data\ir_model_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mass_mailing.model_mailing_contact" model="ir.model">
            <field name="website_form_key">create_mailing_contact</field>
            <field name="website_form_access">True</field>
            <field name="website_form_label">Subscribe to Newsletter</field>
        </record>

        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>mailing.contact</value>
            <value eval="[
                'name',
                'company_name',
                'title_id',
                'email',
                'list_ids',
                'country_id',
                'tag_ids',
            ]"/>
        </function>
    </data>
</odoo>

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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mailing_list_website_designer,access_mailing_list_website_designer,mass_mailing.model_mailing_list,website.group_website_designer,1,0,0,0

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

## File: static\src\js\mass_mailing_form_editor.js

```javascript
/** @odoo-module **/

import core from "web.core";
import FormEditorRegistry from "website.form_editor_registry";

const _lt = core._lt;

FormEditorRegistry.add('create_mailing_contact', {
    formFields: [{
        name: 'name',
        required: true,
        fillWith: "name",
        string: _lt('Your Name'),
        type: 'char',
    }, {
        name: 'email',
        modelRequired: true,
        fillWith: "email",
        string: _lt('Your Email'),
        type: 'email',
    }, {
        name: 'list_ids',
        relation: 'mailing.list',
        modelRequired: true,
        string: _lt('Subscribe to'),
        type: 'many2many',
        fieldName: "name",
    }],
});

```

## File: static\src\js\website_mass_mailing.editor.js

```javascript
odoo.define('website_mass_mailing.editor', function (require) {
'use strict';

var core = require('web.core');
const Dialog = require('web.Dialog');
var options = require('web_editor.snippets.options');

const qweb = core.qweb;
var _t = core._t;


options.registry.mailing_list_subscribe = options.Class.extend({
    /**
     * @override
     */
    onBuilt() {
        this._super(...arguments);
        if (this.mailingLists.length) {
            this.$target.attr("data-list-id", this.mailingLists[0][0]);
        } else {
            Dialog.confirm(this, _t("No mailing list found, do you want to create a new one? This will save all your changes, are you sure you want to proceed?"), {
                confirm_callback: () => {
                    this.trigger_up('request_save', {
                        reload: false,
                        onSuccess: () => {
                            window.location.href = '/web#action=mass_mailing.action_view_mass_mailing_lists';
                        },
                    });
                },
                cancel_callback: () => {
                    this.trigger_up('remove_snippet', {
                        $snippet: this.$target,
                    });
                },
            });
        }
    },
    /**
     * @override
     */
    cleanForSave() {
        const previewClasses = ['o_disable_preview', 'o_enable_preview'];
        const toCleanElsSelector =
            ".js_subscribe_btn, .js_subscribed_btn, #newsletter_form, .s_website_form_end_message";
        const toCleanEls = this.$target[0].querySelectorAll(toCleanElsSelector);
        toCleanEls.forEach(element => {
            element.classList.remove(...previewClasses);
        });
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    toggleThanksButton(previewMode, widgetValue, params) {
        const toSubscribeEl = this.$target[0].querySelector(".js_subscribe_btn, #newsletter_form");
        const thanksMessageEl =
            this.$target[0].querySelector(".js_subscribed_btn, .s_website_form_end_message");

        thanksMessageEl.classList.toggle("o_disable_preview", !widgetValue);
        thanksMessageEl.classList.toggle("o_enable_preview", widgetValue);
        toSubscribeEl.classList.toggle("o_enable_preview", !widgetValue);
        toSubscribeEl.classList.toggle("o_disable_preview", widgetValue);
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
        const toSubscribeElSelector =
            ".js_subscribe_btn.o_disable_preview, #newsletter_form.o_disable_preview";
        return this.$target[0].querySelector(toSubscribeElSelector) ? "true" : "";
    },
    /**
     * @override
     */
    async _renderCustomXML(uiFragment) {
        this.mailingLists = await this._rpc({
            model: 'mailing.list',
            method: 'name_search',
            args: ['', [['is_public', '=', true]]],
            context: this.options.recordInfo.context,
        });
        if (this.mailingLists.length) {
            const selectEl = uiFragment.querySelector('we-select[data-attribute-name="listId"]');
            for (const mailingList of this.mailingLists) {
                const button = document.createElement('we-button');
                button.dataset.selectDataAttribute = mailingList[0];
                button.textContent = mailingList[1];
                selectEl.appendChild(button);
            }
        }
        const checkboxEl = document.createElement('we-checkbox');
        checkboxEl.setAttribute('string', _t("Display Thanks Button"));
        checkboxEl.dataset.toggleThanksButton = 'true';
        checkboxEl.dataset.noPreview = 'true';
        checkboxEl.dataset.dependencies = "!form_opt";
        uiFragment.appendChild(checkboxEl);
    },
});

options.registry.recaptchaSubscribe = options.Class.extend({
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
});

```

## File: static\src\js\website_mass_mailing.js

```javascript
odoo.define('mass_mailing.website_integration', function (require) {
"use strict";

var core = require('web.core');
var publicWidget = require('web.public.widget');
const {ReCaptcha} = require('google_recaptcha.ReCaptchaV3');

var _t = core._t;

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
        this._recaptcha = new ReCaptcha();
    },
    /**
     * @override
     */
    willStart: function () {
        this._recaptcha.loadLibs();
        return this._super(...arguments);
    },
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        if (this.editableMode) {
            // Since there is an editor option to choose whether "Thanks" button
            // should be visible or not, we should not vary its visibility here.
            return def;
        }
        const always = this._updateView.bind(this);
        const inputName = this.$target[0].querySelector('input').name;
        return Promise.all([def, this._rpc({
            route: '/website_mass_mailing/is_subscriber',
            params: {
                'list_id': this._getListId(),
                'subscription_type': inputName,
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
        const valueInputEl = this.$target[0].querySelector('input.js_subscribe_value, input.js_subscribe_email'); // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)

        subscribeBtnEl.disabled = isSubscriber;
        valueInputEl.value = data.value || '';
        valueInputEl.disabled = isSubscriber;
        // Compat: remove d-none for DBs that have the button saved with it.
        this.$target[0].classList.remove('d-none');

        subscribeBtnEl.classList.toggle('d-none', !!isSubscriber);
        thanksBtnEl.classList.toggle('d-none', !isSubscriber);
    },

    _getListId: function () {
        return this.$target.closest('[data-snippet=s_newsletter_block').data('list-id') || this.$target.data('list-id');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubscribeClick: async function () {
        var self = this;
        const inputName = this.$('input').attr('name');
        const $input = this.$(".js_subscribe_value:visible, .js_subscribe_email:visible"); // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        if (inputName === 'email' && $input.length && !$input.val().match(/.+@.+/)) {
            this.$target.addClass('o_has_error').find('.form-control').addClass('is-invalid');
            return false;
        }
        this.$target.removeClass('o_has_error').find('.form-control').removeClass('is-invalid');
        const tokenObj = await this._recaptcha.getToken('website_mass_mailing_subscribe');
        if (tokenObj.error) {
            self.displayNotification({
                type: 'danger',
                title: _t("Error"),
                message: tokenObj.error,
                sticky: true,
            });
            return false;
        }
        this._rpc({
            route: '/website_mass_mailing/subscribe',
            params: {
                'list_id': this._getListId(),
                'value': $input.length ? $input.val() : false,
                'subscription_type': inputName,
                recaptcha_token_response: tokenObj.token,
            },
        }).then(function (result) {
            let toastType = result.toast_type;
            if (toastType === 'success') {
                self.$(".js_subscribe_btn").addClass('d-none');
                self.$(".js_subscribed_btn").removeClass('d-none');
                self.$('input.js_subscribe_value, input.js_subscribe_email').prop('disabled', !!result); // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
                const $popup = self.$target.closest('.o_newsletter_modal');
                if ($popup.length) {
                    $popup.modal('hide');
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

/**
 * This widget tries to fix snippets that were malformed because of a missing
 * upgrade script. Without this, some newsletter snippets coming from users
 * upgraded from a version lower than 16.0 may not be able to update their
 * newsletter block.
 *
 * TODO an upgrade script should be made to fix databases and get rid of this.
 */
publicWidget.registry.fixNewsletterListClass = publicWidget.Widget.extend({
    selector: '.s_newsletter_subscribe_form:not(.s_subscription_list), .s_newsletter_block',

    /**
     * @override
     */
    start() {
        this.$target[0].classList.add('s_newsletter_list');
        return this._super(...arguments);
    },
});

});

```

## File: static\src\snippets\s_popup\000.js

```javascript
/** @odoo-module **/

import PopupWidget from 'website.s_popup';

PopupWidget.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Prevents the (newsletter) popup to be shown if the user is subscribed.
     *
     * @override
     */
    _canShowPopup() {
        if (
            this.$el.is('.o_newsletter_popup') &&
            this.$el.find('input.js_subscribe_value, input.js_subscribe_email').prop('disabled') // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        ) {
            return false;
        }
        return this._super(...arguments);
    },
});

export default PopupWidget;

```

## File: static\src\snippets\s_popup\options.js

```javascript
/** @odoo-module **/

import options from 'web_editor.snippets.options';

options.registry.NewsletterLayout = options.registry.SelectTemplate.extend({
    /**
     * @constructor
     */
    init() {
        this._super(...arguments);
        this.containerSelector = '> .container, > .container-fluid, > .o_container_small';
        this.selectTemplateWidgetName = 'newsletter_template_opt';
    },
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

## File: views\snippets_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="iframe_css_assets_edit" name="CSS assets for wysiwyg iframe content for popup" groups="base.group_user">
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
        <t t-snippet="website_mass_mailing.s_newsletter_block" string="Newsletter Block" t-thumbnail="/website_mass_mailing/static/src/img/snippets_thumbs/s_newsletter_block.svg" t-forbid-sanitize="form"/>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_popup_hook']" position="replace">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_popup" string="Newsletter Popup" t-thumbnail="/website/static/src/img/snippets_thumbs/newsletter_subscribe_popup.svg" t-forbid-sanitize="form"/>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_hook']" position="replace">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_form" string="Newsletter" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_subscribe_form.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

<!--
Users upgraded from a version lower than 16.0 may have those blocks in their
database, without the s_newsletter_list class. See fixNewsletterListClass.
-->
<template id="s_newsletter_subscribe_form" name="Newsletter">
    <div class="s_newsletter_subscribe_form s_newsletter_list js_subscribe" data-vxml="001" data-list-id="0" data-name="Newsletter Form">
        <div class="input-group">
            <input type="email" name="email" class="js_subscribe_value form-control" placeholder="your email..."/>
            <a role="button" href="#" class="btn btn-primary js_subscribe_btn o_submit">Subscribe</a>
            <a role="button" href="#" class="btn btn-success js_subscribed_btn d-none o_submit" disabled="disabled">Thanks</a>
        </div>
    </div>
</template>

<!--
Users upgraded from a version lower than 16.0 may have those blocks in their
database, without the s_newsletter_list class. See fixNewsletterListClass.
-->
<template id="s_newsletter_block" name="Newsletter Block">
    <section class="s_newsletter_block s_newsletter_list pt32 pb32" data-list-id="0">
        <div class="container">
            <t t-call="website_mass_mailing.s_newsletter_block_default_template"/>
        </div>
    </section>
</template>

<template id="s_newsletter_block_default_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-8 offset-lg-2 pt24 pb24">
            <h2>Always First.</h2>
            <p>Be the first to find out all the latest news, products, and trends.</p>
            <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
        </div>
    </div>
</template>

<template id="s_newsletter_block_form_template" groups="base.group_user">
    <section class="s_website_form" data-vcss="001" data-snippet="s_website_form" data-name="Form">
        <div class="container">
            <form id="newsletter_form" action="/website/form/" method="post" enctype="multipart/form-data" class="o_mark_required"
                  data-mark="*" data-model_name="mailing.contact" data-success-mode="message" hide-change-model="true">
                <div class="s_website_form_rows row s_col_no_bgcolor">
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="char" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 250px" for="mailing_sub1">
                                <span class="s_website_form_label_content">Your Name</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <input id="mailing_sub1" type="text" class="form-control s_website_form_input" name="name" required="1"/>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="email" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 250px" for="mailing_sub2">
                                <span class="s_website_form_label_content">Your Email</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <input id="mailing_sub2" type='email' class='form-control s_website_form_input' name="email"/>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="many2many" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-sm-auto s_website_form_label" style="width: 250px" for="mailing_list_">
                                <span class="s_website_form_label_content">Subscribe to</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <div class="row s_col_no_resize s_col_no_bgcolor s_website_form_multiple" data-name="list_ids" data-display="horizontal">
                                    <t t-foreach="request.env['mailing.list'].search([('is_public', '=', True)])" t-as="record">
                                        <div class="checkbox col-12 col-lg-4 col-md-6">
                                            <div class="form-check">
                                                <input type="checkbox" class="s_website_form_input form-check-input" name="list_ids"
                                                    t-attf-id="mailing_list_#{record.id}" t-att-value="record.id" required="1"/>
                                                <label class="form-check-label s_website_form_check_label" t-attf-for="mailing_list_#{record.id}">
                                                    <t t-esc="record.name"/>
                                                </label>
                                            </div>
                                        </div>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_custom s_website_form_required" data-type="boolean" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-sm-auto s_website_form_label" style="width: 250px;" for="mailing_sub3">
                                <span class="s_website_form_label_content">I agree to receive updates</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <div class="form-check">
                                    <input type="checkbox" value="Yes" class="s_website_form_input" name="I want to be kept updated"
                                           id="mailing_sub3" required="1"/>
                                </div>
                                <div class="s_website_form_field_description small form-text text-muted">
                                    We send one weekly newsletter per list and always try to keep it interesting. You can unsubscribe at any time.
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_submit" data-name="Submit Button">
                        <div style="width: 250px;" class="s_website_form_label"/>
                        <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send">Subscribe</a>
                        <span id="s_website_form_result"></span>
                    </div>
                </div>
            </form>
            <div class="s_website_form_end_message d-none">
                <div class="oe_structure">
                    <section class="s_text_block pt64 pb64 o_colored_level o_cc o_cc2" data-snippet="s_text_block">
                        <div class="container">
                            <h2 class="text-center"><span class="fa fa-check-circle"></span>
                                Thank you for subscribing !
                            </h2>
                            <p class="text-center">
                                You will now be informed about the latest news.<br/>
                            </p>
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_newsletter_subscribe_popup" name="Newsletter Popup">
    <div class="s_popup o_newsletter_popup o_snippet_invisible" data-name="Newsletter Popup" data-vcss="001" data-invisible="1">
        <div class="modal fade s_popup_middle o_newsletter_modal"
             style="background-color: var(--black-50) !important;"
             data-show-after="5000" data-display="afterDelay" data-consents-duration="7"
             data-bs-focus="false" data-bs-backdrop="false" tabindex="-1" role="dialog">
            <div class="modal-dialog d-flex">
                <div class="modal-content oe_structure">
                    <div class="s_popup_close js_close_popup o_we_no_overlay o_not_editable" aria-label="Close">&#215;</div>
                    <section class="s_text_block oe_img_bg o_bg_img_center pt88 pb64" data-snippet="s_text_block" data-name="Text"
                             style="background-image: url('/web/image/website.s_cover_default_image'); background-position: 0 100%;">
                        <div class="container s_allow_columns">
                            <h1 style="text-align: center;">Always <b>First</b>.</h1>
                            <p style="text-align: center;">Be the first to find out all the latest news,<br/> products, and trends.</p>
                        </div>
                    </section>
                    <section class="s_text_block" data-snippet="s_text_block" data-name="Text">
                        <div class="container">
                            <div class="row s_nb_column_fixed g-0">
                                <div class="col-lg-8 offset-lg-2 pt32 pb32">
                                    <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
                                </div>
                            </div>
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="newsletter_subscribe_options" name="Newsletter Subscribe Options" inherit_id="website.snippet_options">
    <xpath expr="//*[@t-set='so_snippet_addition_selector']" position="inside" t-translation="off">, .o_newsletter_popup</xpath>
    <xpath expr="//div[1]" position="before">
        <div data-js="NewsletterLayout" data-selector=".s_newsletter_block">
            <we-select string="Template"
                       data-name="newsletter_template_opt"
                       data-attribute-name="newsletterTemplate"
                       data-attribute-default-value="email">
                <we-button title="Email Subscription" string="Email Subscription"
                           data-select-template="website_mass_mailing.s_newsletter_block_default_template"
                           data-select-data-attribute="email" data-name="email_opt"/>
                <we-button title="Form Subscription" string="Form Subscription"
                           data-select-template="website_mass_mailing.s_newsletter_block_form_template"
                           data-select-data-attribute="form" data-name="form_opt"/>
            </we-select>
        </div>
        <t t-call="website_mass_mailing.newsletter_subscribe_options_common">
            <t t-set="_selector">.o_newsletter_popup</t>
            <t t-set="_target">.s_newsletter_list</t>
        </t>
        <t t-call="website_mass_mailing.newsletter_subscribe_options_common">
            <t t-set="_selector">.s_newsletter_list</t>
            <t t-set="_exclude">.s_newsletter_block .s_newsletter_list, .o_newsletter_popup .s_newsletter_list</t>
        </t>
        <div data-selector=".js_subscribe" data-drop-near="p, h1, h2, h3, blockquote, .card"/>
    </xpath>
</template>

<template id="newsletter_subscribe_options_common">
    <div data-js="mailing_list_subscribe"
         t-att-data-selector="_selector"
         t-att-data-exclude="_exclude"
         t-att-data-target="_target">
        <we-select string="Newsletter" data-attribute-name="listId" data-dependencies="!form_opt"></we-select>
    </div>
    <div data-js="recaptchaSubscribe"
         t-att-data-selector="_selector"
         t-att-data-exclude="_exclude"
         t-att-data-target="_target">
        <t t-set="recaptcha_public_key" t-value="request.env['ir.config_parameter'].sudo().get_param('recaptcha_public_key')"/>
        <we-checkbox t-if="recaptcha_public_key" string="Show reCaptcha Policy" data-toggle-recaptcha-legal="" data-no-preview="true"/>
    </div>
</template>

<!-- Extend default mass_mailing snippets with website feature -->

<template id="s_mail_block_footer_social" inherit_id="mass_mailing.s_mail_block_footer_social">
    <xpath expr="//div[hasclass('o_mail_footer_links')]" position="inside">
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

## File: views\snippets\s_popup.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="website_mass_mailing.s_popup_000_js" model="ir.asset">
    <field name="name">Popup 000 JS Website Mass Mailing Override</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_mass_mailing/static/src/snippets/s_popup/000.js</field>
</record>

</odoo>

```

