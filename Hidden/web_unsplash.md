# Odoo Module: web_unsplash

Category: Hidden

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
    'name': 'Unsplash Image Library',
    'category': 'Hidden',
    'summary': 'Find free high-resolution images from Unsplash',
    'version': '1.1',
    'description': """Explore the free high-resolution image library of Unsplash.com and find images to use in Odoo. An Unsplash search bar is added to the image library modal.""",
    'depends': ['base_setup', 'web_editor'],
    'data': [
        'views/res_config_settings_view.xml',
        ],
    'auto_install': True,
    'assets': {
        'web_editor.assets_wysiwyg': [
            'web_unsplash/static/src/js/unsplashapi.js',
            'web_unsplash/static/src/js/unsplash_image_widget.js',
        ],
        'web.assets_frontend': [
            'web_unsplash/static/src/js/unsplash_beacon.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import logging
import mimetypes
import requests
import werkzeug.utils

from odoo import http, tools, _
from odoo.http import request
from odoo.tools.mimetypes import guess_mimetype
from werkzeug.urls import url_encode

logger = logging.getLogger(__name__)


class Web_Unsplash(http.Controller):

    def _get_access_key(self):
        if request.env.user._has_unsplash_key_rights(mode='read'):
            return request.env['ir.config_parameter'].sudo().get_param('unsplash.access_key')
        raise werkzeug.exceptions.NotFound()

    def _notify_download(self, url):
        ''' Notifies Unsplash from an image download. (API requirement)
            :param url: the download_url of the image to be notified

            This method won't return anything. This endpoint should just be
            pinged with a simple GET request for Unsplash to increment the image
            view counter.
        '''
        try:
            if not url.startswith('https://api.unsplash.com/photos/') and not request.env.registry.in_test_mode():
                raise Exception(_("ERROR: Unknown Unsplash notify URL!"))
            access_key = self._get_access_key()
            requests.get(url, params=url_encode({'client_id': access_key}))
        except Exception as e:
            logger.exception("Unsplash download notification failed: " + str(e))

    # ------------------------------------------------------
    # add unsplash image url
    # ------------------------------------------------------
    @http.route('/web_unsplash/attachment/add', type='json', auth='user', methods=['POST'])
    def save_unsplash_url(self, unsplashurls=None, **kwargs):
        """
            unsplashurls = {
                image_id1: {
                    url: image_url,
                    download_url: download_url,
                },
                image_id2: {
                    url: image_url,
                    download_url: download_url,
                },
                .....
            }
        """
        def slugify(s):
            ''' Keeps only alphanumeric characters, hyphens and spaces from a string.
                The string will also be truncated to 1024 characters max.
                :param s: the string to be filtered
                :return: the sanitized string
            '''
            return "".join([c for c in s if c.isalnum() or c in list("- ")])[:1024]

        if not unsplashurls:
            return []

        uploads = []
        Attachments = request.env['ir.attachment']

        query = kwargs.get('query', '')
        query = slugify(query)

        res_model = kwargs.get('res_model', 'ir.ui.view')
        if res_model != 'ir.ui.view' and kwargs.get('res_id'):
            res_id = int(kwargs['res_id'])
        else:
            res_id = None

        for key, value in unsplashurls.items():
            url = value.get('url')
            try:
                if not url.startswith(('https://images.unsplash.com/', 'https://plus.unsplash.com/')) and not request.env.registry.in_test_mode():
                    logger.exception("ERROR: Unknown Unsplash URL!: " + url)
                    raise Exception(_("ERROR: Unknown Unsplash URL!"))

                req = requests.get(url)
                if req.status_code != requests.codes.ok:
                    continue

                # get mime-type of image url because unsplash url dosn't contains mime-types in url
                image_base64 = base64.b64encode(req.content)
            except requests.exceptions.ConnectionError as e:
                logger.exception("Connection Error: " + str(e))
                continue
            except requests.exceptions.Timeout as e:
                logger.exception("Timeout: " + str(e))
                continue

            image_base64 = tools.image_process(image_base64, verify_resolution=True)
            mimetype = guess_mimetype(base64.b64decode(image_base64))
            # append image extension in name
            query += mimetypes.guess_extension(mimetype) or ''

            # /unsplash/5gR788gfd/lion
            url_frags = ['unsplash', key, query]

            attachment = Attachments.create({
                'name': '_'.join(url_frags),
                'url': '/' + '/'.join(url_frags),
                'mimetype': mimetype,
                'datas': image_base64,
                'public': res_model == 'ir.ui.view',
                'res_id': res_id,
                'res_model': res_model,
                'description': value.get('description'),
            })
            attachment.generate_access_token()
            uploads.append(attachment._get_media_info())

            # Notifies Unsplash from an image download. (API requirement)
            self._notify_download(value.get('download_url'))

        return uploads

    @http.route("/web_unsplash/fetch_images", type='json', auth="user")
    def fetch_unsplash_images(self, **post):
        access_key = self._get_access_key()
        app_id = self.get_unsplash_app_id()
        if not access_key or not app_id:
            return {'error': 'key_not_found'}
        post['client_id'] = access_key
        response = requests.get('https://api.unsplash.com/search/photos/', params=url_encode(post))
        if response.status_code == requests.codes.ok:
            return response.json()
        else:
            return {'error': response.status_code}

    @http.route("/web_unsplash/get_app_id", type='json', auth="public")
    def get_unsplash_app_id(self, **post):
        return request.env['ir.config_parameter'].sudo().get_param('unsplash.app_id')

    @http.route("/web_unsplash/save_unsplash", type='json', auth="user")
    def save_unsplash(self, **post):
        if request.env.user._has_unsplash_key_rights(mode='write'):
            request.env['ir.config_parameter'].sudo().set_param('unsplash.app_id', post.get('appId'))
            request.env['ir.config_parameter'].sudo().set_param('unsplash.access_key', post.get('key'))
            return True
        raise werkzeug.exceptions.NotFound()

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main

```

## File: models\ir_qweb.py

```python
from werkzeug import urls

from odoo import models, api


class Image(models.AbstractModel):
    _inherit = 'ir.qweb.field.image'

    @api.model
    def from_html(self, model, field, element):
        if element.find('.//img') is None:
            return False
        url = element.find('.//img').get('src')
        url_object = urls.url_parse(url)

        if url_object.path.startswith('/unsplash/'):
            res_id = element.get('data-oe-id')
            if res_id:
                res_id = int(res_id)
                res_model = model._name
                attachment = self.env['ir.attachment'].search([
                    '&', '|', '&',
                    ('res_model', '=', res_model),
                    ('res_id', '=', res_id),
                    ('public', '=', True),
                    ('url', '=', url_object.path),
                ], limit=1)
                return attachment.datas

        return super(Image, self).from_html(model, field, element)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    unsplash_access_key = fields.Char("Access Key", config_parameter='unsplash.access_key')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class ResUsers(models.Model):
    _inherit = 'res.users'

    def _has_unsplash_key_rights(self, mode='write'):
        self.ensure_one()
        # Website has no dependency to web_unsplash, we cannot warranty the order of the execution
        # of the overwrite done in 5ef8300.
        # So to avoid to create a new module bridge, with a lot of code, we prefer to make a check
        # here for website's user.
        assert mode in ('read', 'write')
        website_group_required = (mode == 'write') and 'website.group_website_designer' or 'website.group_website_publisher'
        return self.has_group('base.group_erp_manager') or self.has_group(website_group_required)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import res_users
from . import ir_qweb

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#7CC098"/><stop offset="100%" stop-color="#5F8A71"/></linearGradient><path id="d" d="M35 24c1.653 0 3.066.538 4.24 1.614C40.413 26.69 41 27.985 41 29.5c0 1.515-.587 2.81-1.76 3.886C38.066 34.462 36.653 35 35 35s-3.066-.538-4.24-1.614C29.587 32.31 29 31.015 29 29.5c0-1.515.587-2.81 1.76-3.886C31.934 24.538 33.347 24 35 24zm11.733-5.154c1.178 0 2.184.376 3.017 1.127.833.751 1.25 1.658 1.25 2.72v13.46c0 1.063-.417 1.969-1.25 2.72-.833.751-1.839 1.127-3.017 1.127H23.267c-1.178 0-2.184-.376-3.017-1.127-.833-.751-1.25-1.657-1.25-2.72v-13.46c0-1.062.417-1.969 1.25-2.72.833-.751 1.839-1.127 3.017-1.127H27l.85-2.043c.211-.49.597-.914 1.158-1.27.561-.355 1.136-.533 1.725-.533h8.534c.589 0 1.164.178 1.725.533.56.356.947.78 1.158 1.27l.85 2.043h3.733zM35 37c2.202 0 4.086-.734 5.652-2.201C42.217 33.33 43 31.565 43 29.5c0-2.065-.783-3.83-2.348-5.299C39.086 22.734 37.202 22 35 22s-4.086.734-5.652 2.201C27.783 25.67 27 27.435 27 29.5c0 2.065.783 3.83 2.348 5.299C30.914 36.266 32.798 37 35 37zM19.699 50.884c0 1.063-.297 1.853-.891 2.37-.594.518-1.415.776-2.464.776-1.063 0-1.886-.257-2.47-.77-.583-.513-.874-1.305-.874-2.376V46h1.727v4.884c0 .213.018.422.055.627.037.205.114.387.231.544.117.158.28.286.49.386.209.099.489.148.841.148.616 0 1.041-.137 1.276-.413.235-.275.352-.705.352-1.292V46h1.727v4.884zm.956-2.717h1.485v.792h.033c.198-.33.454-.57.77-.72.315-.15.638-.226.968-.226.418 0 .76.057 1.028.17.268.114.479.272.633.474.154.201.262.447.324.737.063.29.094.61.094.962v3.498h-1.562v-3.212c0-.47-.074-.82-.22-1.05-.147-.232-.407-.347-.781-.347-.426 0-.734.126-.924.38-.191.253-.286.669-.286 1.248v2.981h-1.562v-5.687zm7.368 3.839a.857.857 0 0 0 .374.731c.11.078.237.134.38.171a1.67 1.67 0 0 0 .792.017c.12-.026.23-.066.33-.122a.75.75 0 0 0 .247-.22.578.578 0 0 0 .1-.346c0-.235-.156-.41-.468-.528a9.336 9.336 0 0 0-1.304-.352 7.71 7.71 0 0 1-.665-.181 2.274 2.274 0 0 1-.578-.276 1.337 1.337 0 0 1-.407-.428 1.216 1.216 0 0 1-.154-.633c0-.367.072-.667.215-.902.143-.235.332-.42.566-.555.235-.136.5-.231.792-.286.294-.056.594-.083.902-.083.308 0 .607.03.897.088.29.059.548.158.775.297.228.14.417.324.567.555.15.232.24.523.27.875h-1.486c-.022-.3-.135-.504-.34-.61a1.559 1.559 0 0 0-.727-.16c-.088 0-.183.005-.286.017a.955.955 0 0 0-.28.071.578.578 0 0 0-.215.16.421.421 0 0 0-.088.28c0 .14.052.253.154.341.103.088.237.16.402.215.165.055.354.104.566.148.213.044.43.092.65.143.227.051.449.114.665.187.216.073.409.17.577.291.17.122.305.272.407.451.103.18.154.402.154.666 0 .374-.075.687-.225.94-.15.254-.347.457-.589.611a2.412 2.412 0 0 1-.83.325 4.84 4.84 0 0 1-1.92-.006 2.555 2.555 0 0 1-.841-.33 1.887 1.887 0 0 1-.605-.61c-.158-.254-.244-.57-.259-.952h1.485zm7.204.825c.257 0 .471-.051.644-.154.172-.103.311-.236.418-.401.106-.166.181-.358.225-.578.044-.22.066-.444.066-.671a3.16 3.16 0 0 0-.071-.671 1.786 1.786 0 0 0-.237-.589 1.317 1.317 0 0 0-.423-.418 1.183 1.183 0 0 0-.633-.159c-.257 0-.471.053-.643.16a1.284 1.284 0 0 0-.418.412 1.712 1.712 0 0 0-.226.583 3.47 3.47 0 0 0-.066.682c0 .227.024.451.072.671.047.22.124.412.23.578.107.165.248.298.424.401.176.103.389.154.638.154zm-2.87-4.664h1.484v.726h.022c.19-.308.433-.532.726-.671.293-.14.616-.209.968-.209.447 0 .832.084 1.155.253.323.169.59.392.803.671.213.279.37.603.473.974.103.37.154.757.154 1.16 0 .381-.051.748-.154 1.1a2.8 2.8 0 0 1-.467.935c-.21.271-.47.488-.781.649-.312.161-.677.242-1.095.242-.352 0-.676-.071-.973-.215a1.843 1.843 0 0 1-.732-.632h-.022v2.695h-1.562v-7.678zm6.4-2.167h1.562v7.854h-1.562V46zm2.364 3.916c.022-.367.113-.671.275-.913.16-.242.366-.436.616-.583.249-.147.53-.251.841-.314.312-.062.625-.093.94-.093.287 0 .576.02.87.06.293.04.56.12.803.237.242.117.44.28.594.49.154.209.23.485.23.83v2.959c0 .257.015.502.045.737.029.235.08.41.154.528h-1.584a2.241 2.241 0 0 1-.11-.55c-.25.257-.543.436-.88.539a3.532 3.532 0 0 1-1.034.154c-.272 0-.525-.033-.76-.099a1.738 1.738 0 0 1-.615-.308 1.434 1.434 0 0 1-.413-.528 1.785 1.785 0 0 1-.148-.759c0-.323.056-.588.17-.797.114-.21.26-.376.44-.501s.385-.218.616-.28c.231-.063.464-.112.699-.149.234-.037.465-.066.693-.088.227-.022.429-.055.605-.099.176-.044.315-.108.418-.193.102-.084.15-.207.143-.368a.829.829 0 0 0-.083-.402.607.607 0 0 0-.22-.23.865.865 0 0 0-.319-.11 2.61 2.61 0 0 0-.39-.028c-.308 0-.55.066-.726.198-.176.132-.28.352-.308.66H41.12zm3.608 1.155a.7.7 0 0 1-.248.138c-.099.033-.205.06-.319.082-.114.022-.233.04-.357.055-.125.015-.25.033-.374.055a2.82 2.82 0 0 0-.347.088 1.024 1.024 0 0 0-.297.149.706.706 0 0 0-.203.236.76.76 0 0 0-.077.363c0 .14.025.257.077.352a.59.59 0 0 0 .209.226c.088.055.19.093.308.115.117.022.238.033.363.033.308 0 .546-.051.715-.154a1.03 1.03 0 0 0 .374-.368c.08-.144.13-.288.148-.435.018-.147.028-.264.028-.352v-.583zm3.617.935a.857.857 0 0 0 .374.731c.11.078.237.134.38.171a1.67 1.67 0 0 0 .792.017c.12-.026.23-.066.33-.122a.75.75 0 0 0 .247-.22.578.578 0 0 0 .1-.346c0-.235-.157-.41-.468-.528a9.336 9.336 0 0 0-1.304-.352 7.71 7.71 0 0 1-.665-.181 2.274 2.274 0 0 1-.578-.276 1.337 1.337 0 0 1-.407-.428 1.216 1.216 0 0 1-.154-.633c0-.367.072-.667.215-.902.143-.235.332-.42.566-.555.235-.136.499-.231.792-.286.294-.056.594-.083.902-.083.308 0 .607.03.897.088.29.059.548.158.775.297.228.14.416.324.567.555.15.232.24.523.27.875H50.49c-.022-.3-.135-.504-.34-.61a1.559 1.559 0 0 0-.727-.16c-.088 0-.183.005-.286.017a.955.955 0 0 0-.28.071.578.578 0 0 0-.215.16.421.421 0 0 0-.088.28c0 .14.052.253.154.341.103.088.237.16.402.215.165.055.354.104.566.148.213.044.43.092.65.143.227.051.448.114.665.187.216.073.409.17.577.291.169.122.305.272.407.451.103.18.154.402.154.666 0 .374-.075.687-.225.94-.15.254-.347.457-.589.611a2.412 2.412 0 0 1-.83.325 4.84 4.84 0 0 1-1.92-.006 2.555 2.555 0 0 1-.841-.33 1.887 1.887 0 0 1-.605-.61c-.158-.254-.244-.57-.259-.952h1.485zM52.68 46h1.562v2.959h.033c.198-.33.451-.57.759-.72.308-.15.609-.226.902-.226.418 0 .76.057 1.028.17.268.114.479.272.633.474.154.201.262.447.324.737.063.29.094.61.094.962v3.498h-1.562v-3.212c0-.47-.073-.82-.22-1.05-.147-.232-.407-.347-.781-.347-.425 0-.733.126-.924.38-.19.253-.286.669-.286 1.248v2.981h-1.562V46zM14.036 57.642H13V57h2.833v.642h-1.037v2.832h-.76v-2.832zM16 57h1.07l.809 2.389h.01L18.652 57h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H16V57z"/><path id="e" d="M35 22c1.653 0 3.066.538 4.24 1.614C40.413 24.69 41 25.985 41 27.5c0 1.515-.587 2.81-1.76 3.886C38.066 32.462 36.653 33 35 33s-3.066-.538-4.24-1.614C29.587 30.31 29 29.015 29 27.5c0-1.515.587-2.81 1.76-3.886C31.934 22.538 33.347 22 35 22zm11.733-5.154c1.178 0 2.184.376 3.017 1.127.833.751 1.25 1.658 1.25 2.72v13.46c0 1.063-.417 1.969-1.25 2.72-.833.751-1.839 1.127-3.017 1.127H23.267c-1.178 0-2.184-.376-3.017-1.127-.833-.751-1.25-1.657-1.25-2.72v-13.46c0-1.062.417-1.969 1.25-2.72.833-.751 1.839-1.127 3.017-1.127H27l.85-2.043c.211-.49.597-.914 1.158-1.27.561-.355 1.136-.533 1.725-.533h8.534c.589 0 1.164.178 1.725.533.56.356.947.78 1.158 1.27l.85 2.043h3.733zM35 35c2.202 0 4.086-.734 5.652-2.201C42.217 31.33 43 29.565 43 27.5c0-2.065-.783-3.83-2.348-5.299C39.086 20.734 37.202 20 35 20s-4.086.734-5.652 2.201C27.783 23.67 27 25.435 27 27.5c0 2.065.783 3.83 2.348 5.299C30.914 34.266 32.798 35 35 35zM19.699 48.884c0 1.063-.297 1.853-.891 2.37-.594.518-1.415.776-2.464.776-1.063 0-1.886-.257-2.47-.77-.583-.513-.874-1.305-.874-2.376V44h1.727v4.884c0 .213.018.422.055.627.037.205.114.387.231.544.117.158.28.286.49.386.209.099.489.148.841.148.616 0 1.041-.137 1.276-.413.235-.275.352-.705.352-1.292V44h1.727v4.884zm.956-2.717h1.485v.792h.033c.198-.33.454-.57.77-.72.315-.15.638-.226.968-.226.418 0 .76.057 1.028.17.268.114.479.272.633.474.154.201.262.447.324.737.063.29.094.61.094.962v3.498h-1.562v-3.212c0-.47-.074-.82-.22-1.05-.147-.232-.407-.347-.781-.347-.426 0-.734.126-.924.38-.191.253-.286.669-.286 1.248v2.981h-1.562v-5.687zm7.368 3.839a.857.857 0 0 0 .374.731c.11.078.237.134.38.171a1.67 1.67 0 0 0 .792.017c.12-.026.23-.066.33-.122a.75.75 0 0 0 .247-.22.578.578 0 0 0 .1-.346c0-.235-.156-.41-.468-.528a9.336 9.336 0 0 0-1.304-.352 7.71 7.71 0 0 1-.665-.181 2.274 2.274 0 0 1-.578-.276 1.337 1.337 0 0 1-.407-.428 1.216 1.216 0 0 1-.154-.633c0-.367.072-.667.215-.902.143-.235.332-.42.566-.555.235-.136.5-.231.792-.286.294-.056.594-.083.902-.083.308 0 .607.03.897.088.29.059.548.158.775.297.228.14.417.324.567.555.15.232.24.523.27.875h-1.486c-.022-.3-.135-.504-.34-.61a1.559 1.559 0 0 0-.727-.16c-.088 0-.183.005-.286.017a.955.955 0 0 0-.28.071.578.578 0 0 0-.215.16.421.421 0 0 0-.088.28c0 .14.052.253.154.341.103.088.237.16.402.215.165.055.354.104.566.148.213.044.43.092.65.143.227.051.449.114.665.187.216.073.409.17.577.291.17.122.305.272.407.451.103.18.154.402.154.666 0 .374-.075.687-.225.94-.15.254-.347.457-.589.611a2.412 2.412 0 0 1-.83.325 4.84 4.84 0 0 1-1.92-.006 2.555 2.555 0 0 1-.841-.33 1.887 1.887 0 0 1-.605-.61c-.158-.254-.244-.57-.259-.952h1.485zm7.204.825c.257 0 .471-.051.644-.154.172-.103.311-.236.418-.401.106-.166.181-.358.225-.578.044-.22.066-.444.066-.671a3.16 3.16 0 0 0-.071-.671 1.786 1.786 0 0 0-.237-.589 1.317 1.317 0 0 0-.423-.418 1.183 1.183 0 0 0-.633-.159c-.257 0-.471.053-.643.16a1.284 1.284 0 0 0-.418.412 1.712 1.712 0 0 0-.226.583 3.47 3.47 0 0 0-.066.682c0 .227.024.451.072.671.047.22.124.412.23.578.107.165.248.298.424.401.176.103.389.154.638.154zm-2.87-4.664h1.484v.726h.022c.19-.308.433-.532.726-.671.293-.14.616-.209.968-.209.447 0 .832.084 1.155.253.323.169.59.392.803.671.213.279.37.603.473.974.103.37.154.757.154 1.16 0 .381-.051.748-.154 1.1a2.8 2.8 0 0 1-.467.935c-.21.271-.47.488-.781.649-.312.161-.677.242-1.095.242-.352 0-.676-.071-.973-.215a1.843 1.843 0 0 1-.732-.632h-.022v2.695h-1.562v-7.678zm6.4-2.167h1.562v7.854h-1.562V44zm2.364 3.916c.022-.367.113-.671.275-.913.16-.242.366-.436.616-.583.249-.147.53-.251.841-.314.312-.062.625-.093.94-.093.287 0 .576.02.87.06.293.04.56.12.803.237.242.117.44.28.594.49.154.209.23.485.23.83v2.959c0 .257.015.502.045.737.029.235.08.41.154.528h-1.584a2.241 2.241 0 0 1-.11-.55c-.25.257-.543.436-.88.539a3.532 3.532 0 0 1-1.034.154c-.272 0-.525-.033-.76-.099a1.738 1.738 0 0 1-.615-.308 1.434 1.434 0 0 1-.413-.528 1.785 1.785 0 0 1-.148-.759c0-.323.056-.588.17-.797.114-.21.26-.376.44-.501s.385-.218.616-.28c.231-.063.464-.112.699-.149.234-.037.465-.066.693-.088.227-.022.429-.055.605-.099.176-.044.315-.108.418-.193.102-.084.15-.207.143-.368a.829.829 0 0 0-.083-.402.607.607 0 0 0-.22-.23.865.865 0 0 0-.319-.11 2.61 2.61 0 0 0-.39-.028c-.308 0-.55.066-.726.198-.176.132-.28.352-.308.66H41.12zm3.608 1.155a.7.7 0 0 1-.248.138c-.099.033-.205.06-.319.082-.114.022-.233.04-.357.055-.125.015-.25.033-.374.055a2.82 2.82 0 0 0-.347.088 1.024 1.024 0 0 0-.297.149.706.706 0 0 0-.203.236.76.76 0 0 0-.077.363c0 .14.025.257.077.352a.59.59 0 0 0 .209.226c.088.055.19.093.308.115.117.022.238.033.363.033.308 0 .546-.051.715-.154a1.03 1.03 0 0 0 .374-.368c.08-.144.13-.288.148-.435.018-.147.028-.264.028-.352v-.583zm3.617.935a.857.857 0 0 0 .374.731c.11.078.237.134.38.171a1.67 1.67 0 0 0 .792.017c.12-.026.23-.066.33-.122a.75.75 0 0 0 .247-.22.578.578 0 0 0 .1-.346c0-.235-.157-.41-.468-.528a9.336 9.336 0 0 0-1.304-.352 7.71 7.71 0 0 1-.665-.181 2.274 2.274 0 0 1-.578-.276 1.337 1.337 0 0 1-.407-.428 1.216 1.216 0 0 1-.154-.633c0-.367.072-.667.215-.902.143-.235.332-.42.566-.555.235-.136.499-.231.792-.286.294-.056.594-.083.902-.083.308 0 .607.03.897.088.29.059.548.158.775.297.228.14.416.324.567.555.15.232.24.523.27.875H50.49c-.022-.3-.135-.504-.34-.61a1.559 1.559 0 0 0-.727-.16c-.088 0-.183.005-.286.017a.955.955 0 0 0-.28.071.578.578 0 0 0-.215.16.421.421 0 0 0-.088.28c0 .14.052.253.154.341.103.088.237.16.402.215.165.055.354.104.566.148.213.044.43.092.65.143.227.051.448.114.665.187.216.073.409.17.577.291.169.122.305.272.407.451.103.18.154.402.154.666 0 .374-.075.687-.225.94-.15.254-.347.457-.589.611a2.412 2.412 0 0 1-.83.325 4.84 4.84 0 0 1-1.92-.006 2.555 2.555 0 0 1-.841-.33 1.887 1.887 0 0 1-.605-.61c-.158-.254-.244-.57-.259-.952h1.485zM52.68 44h1.562v2.959h.033c.198-.33.451-.57.759-.72.308-.15.609-.226.902-.226.418 0 .76.057 1.028.17.268.114.479.272.633.474.154.201.262.447.324.737.063.29.094.61.094.962v3.498h-1.562v-3.212c0-.47-.073-.82-.22-1.05-.147-.232-.407-.347-.781-.347-.425 0-.733.126-.924.38-.19.253-.286.669-.286 1.248v2.981h-1.562V44zM14.036 55.642H13V55h2.833v.642h-1.037v2.832h-.76v-2.832zM16 55h1.07l.809 2.389h.01L18.652 55h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H16V55z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V33.916L25 18l11.4-3 11.016 3.802 3.209 19.927L46.5 47.19l1.345-.833h2.78l2.135-2.198L54 47.19l3-.19 1.014 4.854L39.224 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\description\index.html

```html
<section class="oe_container">
    <div class="oe_row oe_spaced">
        <h2 class="oe_slogan" style="color:#875A7B;">Free high-resolution image library</h2>
        <h3 class="oe_slogan">An Unsplash search bar is added to the image library modal...</h3>
        <div class="oe_span12">
            <div class="oe_demo oe_picture oe_screenshot">
                <img src="unsplash_interface_bar.png">
            </div>
        </div>
    </div>
</section>
<section class="oe_container">
    <div class="oe_row oe_spaced">
        <h3 class="oe_slogan">...to explore <a href="https://unsplash.com/">Unsplash.com</a> and find images to use in Odoo.</h3>
        <div class="oe_span12">
            <div class="oe_demo oe_picture oe_screenshot">
                <img src="unsplash_interface.png">
            </div>
        </div>
    </div>
</section>


```

## File: static\src\js\unsplashapi.js

```javascript
odoo.define('unsplash.api', function (require) {
'use strict';

var Class = require('web.Class');
var rpc = require('web.rpc');
var Mixins = require('web.mixins');
var ServicesMixin = require('web.ServicesMixin');

var UnsplashCore = Class.extend(Mixins.EventDispatcherMixin, ServicesMixin, {
    /**
     * @constructor
     */
    init: function (parent) {
        Mixins.EventDispatcherMixin.init.call(this, arguments);
        this.setParent(parent);

        this._cache = {};
        this.clientId = false;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Gets unsplash images from query string.
     *
     * @param {String} query search terms
     * @param {Integer} pageSize number of image to display per page
     * @returns {Promise}
     */
    getImages: function (query, pageSize) {
        var from = 0;
        var to = pageSize;
        var cachedData = this._cache[query];

        if (cachedData && (cachedData.images.length >= to || (cachedData.totalImages !== 0 && cachedData.totalImages < to))) {
            return Promise.resolve({ images: cachedData.images.slice(from, to), isMaxed: to > cachedData.totalImages });
        }
        return this._fetchImages(query).then(function (cachedData) {
            return { images: cachedData.images.slice(from, to), isMaxed: to > cachedData.totalImages };
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Fetches images from unsplash and stores it in cache
     *
     * @param {String} query search terms
     * @returns {Promise}
     * @private
     */
    _fetchImages: function (query) {
        if (!this._cache[query]) {
            this._cache[query] = {
                images: [],
                maxPages: 0,
                totalImages: 0,
                pageCached: 0
            };
        }
        var cachedData = this._cache[query];
        var payload = {
            query: query,
            page: cachedData.pageCached + 1,
            per_page: 30, // max size from unsplash API
        };
        return this._rpc({
            route: '/web_unsplash/fetch_images',
            params: payload,
        }).then(function (result) {
            if (result.error) {
                return Promise.reject(result.error);
            }
            cachedData.pageCached++;
            cachedData.images.push.apply(cachedData.images, result.results);
            cachedData.maxPages = result.total_pages;
            cachedData.totalImages = result.total;
            return cachedData;
        });
    },
});

return UnsplashCore;

});

```

## File: static\src\js\unsplash_beacon.js

```javascript
odoo.define('web_unsplash.beacon', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.UnsplashBeacon = publicWidget.Widget.extend({
    // /!\ To adapt the day the beacon makes sense for backend customizations
    selector: '#wrapwrap',

    /**
     * @override
     */
    start: function () {
        var unsplashImages = _.map(this.$('img[src*="/unsplash/"]'), function (img) {
            // get image id from URL (`http://www.domain.com:1234/unsplash/xYdf5feoI/lion.jpg` -> `xYdf5feoI`)
            return img.src.split('/unsplash/')[1].split('/')[0];
        });
        if (unsplashImages.length) {
            this._rpc({
                route: '/web_unsplash/get_app_id',
            }).then(function (appID) {
                if (!appID) {
                    return;
                }
                $.get('https://views.unsplash.com/v', {
                    'photo_id': unsplashImages.join(','),
                    'app_id': appID,
                });
            });
        }
        return this._super.apply(this, arguments);
    },
});
});

```

## File: static\src\js\unsplash_image_widget.js

```javascript
odoo.define('web_unsplash.image_widgets', function (require) {
'use strict';

var core = require('web.core');
var UnsplashAPI = require('unsplash.api');
const {UploadProgressToast} = require('@web_editor/js/wysiwyg/widgets/upload_progress_toast');
var widgetsMedia = require('wysiwyg.widgets.media');
const {_t} = require('web.core');

var unsplashAPI = null;

// Prevent base class from treating unsplash images like regular attachments
const originalEvents = widgetsMedia.ImageWidget.prototype.events;
const clickHandler = originalEvents['click .o_existing_attachment_cell'];
if (!clickHandler) {
    throw new Error(`Couldn't find a handler for o_existing_attachment_cell clicks.
The unsplash image widget needs to prevent this handler from executing on unsplash attachments.`);
}
_.extend(originalEvents, {
    'click .o_existing_attachment_cell:not(.o_unsplash_attachment_cell)': clickHandler,
});
delete originalEvents['click .o_existing_attachment_cell'];

widgetsMedia.ImageWidget.include({
    xmlDependencies: widgetsMedia.ImageWidget.prototype.xmlDependencies.concat(
        ['/web_unsplash/static/src/xml/unsplash_image_widget.xml']
    ),
    events: _.extend({}, widgetsMedia.ImageWidget.prototype.events, {
        'click .o_unsplash_attachment_cell[data-imgid]': '_onUnsplashImgClick',
        'click button.save_unsplash': '_onSaveUnsplashCredentials',
    }),

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);

        this._unsplash = {
            selectedImages: {},
            isMaxed: false,
            query: false,
            error: false,
            records: [],
        };

        // TODO improve this
        //
        // This is a `hack` to prevent the UnsplashAPI to be destroyed every
        // time the media dialog is closed. Indeed, UnsplashAPI has a cache
        // system to recude unsplash call, it is then better to keep its state
        // to take advantage from it from one media dialog call to another.
        //
        // Unsplash API will either be (it's still being discussed):
        //  * a service (ideally coming with an improvement to not auto load
        //    the service)
        //  * initialized in the website_root (trigger_up)
        if (unsplashAPI === null) {
            this.unsplashAPI = new UnsplashAPI(this);
            unsplashAPI = this.unsplashAPI;
        } else {
            this.unsplashAPI = unsplashAPI;
            this.unsplashAPI.setParent(this);
        }
    },
    /**
     * @override
     */
    destroy: function () {
        // TODO See `hack` explained in `init`. This prevent the media dialog destroy
        //      to destroy unsplashAPI when destroying the children
        this.unsplashAPI.setParent(undefined);
        this._super.apply(this, arguments);
    },

    // --------------------------------------------------------------------------
    // Public
    // --------------------------------------------------------------------------

    /**
     * @override
     */
    _save: async function () {
        const _super = this._super;
        const selectedImages = this._unsplash.selectedImages;
        const imagesCount = Object.keys(selectedImages).length;
        if (imagesCount) {
            this.saved = true;
            await this._setUpProgressToast([{
                name: imagesCount > 1 ?
                    _.str.sprintf(_t("Uploading %s '%s' images."), imagesCount, this._unsplash.query) :
                    _.str.sprintf(_t("Uploading '%s' image."), this._unsplash.query),
                size: null,
            }]);
            const images = await this.uploader.rpcShowProgress({
                route: '/web_unsplash/attachment/add',
                params: {
                    unsplashurls: selectedImages,
                    res_model: this.options.res_model,
                    res_id: this.options.res_id,
                    query: this._unsplash.query,
                },
            }, 0);
            this.uploader.close(3000);
            this.attachments.push(...images);
            this.selectedAttachments.push(...images);
        }
        return _super.apply(this, arguments);
    },
    /**
     * @override
     */
    search: async function (needle) {
        var self = this;
        await this._super(...arguments);

        this._unsplash.query = needle;
        if (!needle) {
            this._unsplash.records = [];
            return;
        }

        await this.unsplashAPI.getImages(needle, this.numberOfAttachmentsToDisplay).then(function (res) {
            self._unsplash.isMaxed = res.isMaxed;
            self._unsplash.records = res.images;
            self._unsplash.error = false;
        }, function (err) {
            self._unsplash.error = err;
        });
    },
    /**
     * @override
     */
    hasContent() {
        if (this.searchService === 'all') {
            return this._super(...arguments) || (this.unsplashRecords && this.unsplashRecords.length);
        } else if (this.searchService === 'unsplash') {
            return (this.unsplashRecords && this.unsplashRecords.length);
        }
        return this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _highlightSelected: function () {
        this._super.apply(this, arguments);

        const $select = this.$('.o_unsplash_attachment_cell[data-imgid]').filter((i, el) => {
            return $(el).data('imgid') in this._unsplash.selectedImages;
        }).addClass('o_we_attachment_selected');
        return $select;
    },
    /**
     * @private
     */
    _loadMoreImages: function (forceSearch) {
        if (!this.$('.o_we_search').val() || this.searchService === "database") {
            return this._super(forceSearch);
        }
        this.numberOfAttachmentsToDisplay += 10;
        this.search(this.$('.o_we_search').val()).then(() => this._renderThumbnails());
    },
    /**
     * @override
     */
    _renderThumbnails: function () {
        this._super(...arguments);
        this.$('.unsplash_error').empty();
        if (!['all', 'unsplash'].includes(this.searchService)) {
            return;
        }
        if (this._unsplash.query && this._unsplash.error) {
            this.$('.unsplash_error').html(
                core.qweb.render('web_unsplash.dialog.error.content', {
                    status: this._unsplash.error,
                })
            );
            return;
        }

        if (['all', 'unsplash'].includes(this.searchService) && this._unsplash.query && !this._unsplash.isMaxed) {
            this.$('.o_load_more').removeClass('d-none');
            this.$('.o_load_done_msg').addClass('d-none');
        }
    },
    /**
     * @override
     */
    _renderExisting: function (attachments) {
        this.unsplashRecords = this._unsplash.records.map(record => {
            const url = new URL(record.urls.regular);
            // In small windows, row height could get quite a bit larger than the min, so we keep some leeway.
            url.searchParams.set('h', 2 * this.MIN_ROW_HEIGHT);
            url.searchParams.delete('w');
            return Object.assign({}, record, {
                url: url.toString(),
            });
        });
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _selectAttachement: function (attachment, save) {
        if (!this.options.multiImages) {
            this._unsplash.selectedImages = {};
        }
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSaveUnsplashCredentials: function () {
        var self = this;
        var key = this.$('#accessKeyInput').val().trim();
        var appId = this.$('#appIdInput').val().trim();

        this.$('#accessKeyInput').toggleClass('is-invalid', !key);
        this.$('#appIdInput').toggleClass('is-invalid', !appId);

        if (key && appId) {
            if (!this.$el.find('.is-invalid').length) {
                this._rpc({
                    route: '/web_unsplash/save_unsplash',
                    params: {key: key, appId: appId},
                }).then(function () {
                    self.unsplashAPI.clientId = key;
                    self._unsplash.error = false;
                    self.search(self._unsplash.query).then(() => self._renderThumbnails());
                });
            }
        }
    },
    /**
     * @private
     */
    _onUnsplashImgClick: function (ev) {
        if (this.saved) {
            // already saved, probably a double click. Ignore.
            return;
        }
        const {imgid, url, downloadUrl, description} = ev.currentTarget.dataset;
        if (!this.options.multiImages) {
            this._unsplash.selectedImages = {};
            this.selectedAttachments = [];
        }
        if (imgid in this._unsplash.selectedImages) {
            delete this._unsplash.selectedImages[imgid];
        } else {
            const _1920Url = new URL(url);
            _1920Url.searchParams.set('w', '1920');
            this._unsplash.selectedImages[imgid] = {url: _1920Url.href, download_url: downloadUrl, description: description};
        }
        this._highlightSelected();
        if (!this.options.multiImages) {
            this.trigger_up('save_request');
        }
    },
});
});

```

## File: static\src\xml\unsplash_image_widget.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

<t t-extend="wysiwyg.widgets.file">
    <t t-jquery=".o_we_load_more" t-operation="after">
        <div class="unsplash_error"></div>
    </t>
</t>
<t t-extend="wysiwyg.widgets.image">
    <t t-jquery="option[value='media-library']" t-operation="after">
        <option value="unsplash">Photos (via Unsplash)</option>
    </t>
</t>
<t t-extend="wysiwyg.widgets.image.existing.attachments">
    <t t-jquery="t[t-foreach='libraryMedia']" t-operation="after">
        <t t-if="['all', 'unsplash'].includes(widget.searchService)" t-foreach="widget.unsplashRecords" t-as="record">
            <t t-call="web_unsplash.dialog.image.content"/>
        </t>
    </t>
</t>

<t t-name="web_unsplash.dialog.image.content">
    <div class="o_existing_attachment_cell o_unsplash_attachment_cell position-relative align-items-center justify-content-center bg-light" t-att-data-imgid="record.id" t-att-data-id="record.id" t-att-data-url="record.urls.regular" t-att-data-download-url="record.links.download_location" t-att-data-description="record.alt_description">
        <img class="img img-fluid o_we_attachment_highlight" t-att-src="record.url" t-att-alt="record.alt_description" style="max-height: 100%;"/>
        <a class="o_we_media_author" t-att-href="record.user.links.html" target="_blank" t-esc="record.user.name" t-att-title="record.user.name"/>
    </div>
</t>

<t t-name="web_unsplash.dialog.error.credentials">
    <div class="alert alert-info w-100">
        <h4><t t-esc="title"/></h4>
        <p><t t-esc="subtitle"/></p>
        <t t-if="showUnsplashSetup">
            <div class="form-inline">
                <div class="form-group">
                    <span>
                        <a href="https://www.odoo.com/documentation/15.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-access-key" target="_blank">Get an Access key</a>
                        and paste it here:
                        <input type="text" class="form-control w-auto" id="accessKeyInput" placeholder="Paste your access key here"/>
                    </span>
                </div>
                <div class="form-group my-md-3">
                    <span class="ml-xl-1">
                        and paste
                        <a href="https://www.odoo.com/documentation/15.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-application-id" target="_blank">Application ID</a>
                        here:
                        <input type="text" class="form-control w-auto" id="appIdInput" placeholder="Paste your application ID here"/>
                    </span>
                </div>
                <button type="button" class="btn btn-primary btn-block w-auto ml-2 save_unsplash">Apply</button>
            </div>
        </t>
    </div>
</t>

<t t-name="web_unsplash.dialog.error.content">
    <div class="d-flex mt-2 unsplash_error">
        <t t-call="web_unsplash.dialog.error.credentials">
            <t t-if="status == 'key_not_found'">
                <t t-set="title">
                    Setup Unsplash to access royalty free photos.
                </t>
                <t t-set="showUnsplashSetup" t-value="true"/>
            </t>
            <t t-elif="status == 403">
                <t t-set="title">
                    Search is temporarily unavailable
                </t>
                <t t-set="subtitle">
                    The max number of searches is exceeded. Please retry in an hour or extend to a better account.
                </t>
            </t>
            <t t-elif="status == 401">
                <t t-set="title">
                    Unauthorized Key
                </t>
                <t t-set="subtitle">
                    Please check your Unsplash access key and application ID.
                </t>
                <t t-set="showUnsplashSetup" t-value="true"/>
            </t>
            <t t-else="">
                <t t-set="title">
                    Something went wrong
                </t>
                <t t-set="subtitle">
                    Please check your internet connection or contact administrator.
                </t>
            </t>
        </t>
    </div>
</t>

</templates>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.web.unsplash</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="web_unsplash_warning" position="replace">
                <div attrs="{'invisible': [('module_web_unsplash', '=', False)]}">
                    <div class="content-group mt16">
                        <label for="unsplash_access_key" class="o_light_label"/>
                        <field name="unsplash_access_key"/>
                    </div>
                    <div>
                        <a href="https://www.odoo.com/documentation/15.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-access-key" class="oe_link" target="_blank">
                            <i class="fa fa-arrow-right"/> Generate an Access Key
                        </a>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

