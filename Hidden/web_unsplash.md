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
        'web.assets_frontend': [
            'web_unsplash/static/src/js/unsplash_beacon.js',
        ],
        'web_editor.assets_media_dialog': [
            'web_unsplash/static/src/components/media_dialog/*.js',
            'web_unsplash/static/src/components/media_dialog/*.xml',
            'web_unsplash/static/src/services/unsplash_service.js',
        ],
        'web.qunit_suite_tests': [
            'web_unsplash/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import mimetypes
import requests
import werkzeug.utils
from werkzeug.urls import url_encode

from odoo import http, tools, _
from odoo.http import request
from odoo.tools.mimetypes import guess_mimetype

from odoo.addons.web_editor.controllers.main import Web_Editor

logger = logging.getLogger(__name__)


class Web_Unsplash(http.Controller):

    def _get_access_key(self):
        """ Use this method to get the key, needed for internal reason """
        return request.env['ir.config_parameter'].sudo().get_param('unsplash.access_key')

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
                image = req.content
            except requests.exceptions.ConnectionError as e:
                logger.exception("Connection Error: " + str(e))
                continue
            except requests.exceptions.Timeout as e:
                logger.exception("Timeout: " + str(e))
                continue

            image = tools.image_process(image, verify_resolution=True)
            mimetype = guess_mimetype(image)
            # append image extension in name
            query += mimetypes.guess_extension(mimetype) or ''

            # /unsplash/5gR788gfd/lion
            url_frags = ['unsplash', key, query]

            attachment_data = {
                'name': '_'.join(url_frags),
                'url': '/' + '/'.join(url_frags),
                'data': image,
                'res_id': res_id,
                'res_model': res_model,
            }
            attachment = Web_Editor._attachment_create(self, **attachment_data)
            if value.get('description'):
                attachment.description = value.get('description')
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
            if not request.env.user._can_manage_unsplash_settings():
                return {'error': 'no_access'}
            return {'error': 'key_not_found'}
        post['client_id'] = access_key
        response = requests.get('https://api.unsplash.com/search/photos/', params=url_encode(post))
        if response.status_code == requests.codes.ok:
            return response.json()
        else:
            if not request.env.user._can_manage_unsplash_settings():
                return {'error': 'no_access'}
            return {'error': response.status_code}

    @http.route("/web_unsplash/get_app_id", type='json', auth="public")
    def get_unsplash_app_id(self, **post):
        return request.env['ir.config_parameter'].sudo().get_param('unsplash.app_id')

    @http.route("/web_unsplash/save_unsplash", type='json', auth="user")
    def save_unsplash(self, **post):
        if request.env.user._can_manage_unsplash_settings():
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

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Attachment(models.Model):

    _inherit = "ir.attachment"

    def _can_bypass_rights_on_media_dialog(self, **attachment_data):
        # We need to allow and sudo the case of an "url + file" attachment,
        # which is by default forbidden for non admin.
        # See `_check_serving_attachments`
        forbidden = 'url' in attachment_data and attachment_data.get('type', 'binary') == 'binary'
        if forbidden and attachment_data['url'].startswith('/unsplash/'):
            return True
        return super()._can_bypass_rights_on_media_dialog(**attachment_data)

```

## File: models\ir_qweb_fields.py

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
    unsplash_app_id = fields.Char("Application ID", config_parameter='unsplash.app_id')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class ResUsers(models.Model):
    _inherit = 'res.users'

    def _can_manage_unsplash_settings(self):
        self.ensure_one()
        # Website has no dependency to web_unsplash, we cannot warranty the order of the execution
        # of the overwrite done in 5ef8300.
        # So to avoid to create a new module bridge, with a lot of code, we prefer to make a check
        # here for website's user.
        return self.has_group('base.group_erp_manager') or self.has_group('website.group_website_restricted_editor')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import ir_qweb_fields
from . import res_config_settings
from . import res_users

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/><path d="M17.125 15.813V4h15.75v11.813h-15.75Zm15.75 6.562H46V46H4V22.375h13.125v11.813h15.75V22.374Z"/></svg>

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

## File: static\src\components\media_dialog\image_selector.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { KeepLast } from "@web/core/utils/concurrency";
import { MediaDialog, TABS } from '@web_editor/components/media_dialog/media_dialog';
import { ImageSelector } from '@web_editor/components/media_dialog/image_selector';
import { useService } from '@web/core/utils/hooks';
import { uploadService, AUTOCLOSE_DELAY } from '@web_editor/components/upload_progress_toast/upload_service';

import { useState, Component } from "@odoo/owl";

class UnsplashCredentials extends Component {
    setup() {
        this.state = useState({
            key: '',
            appId: '',
            hasKeyError: this.props.hasCredentialsError,
            hasAppIdError: this.props.hasCredentialsError,
        });
    }

    submitCredentials() {
        if (this.state.key === '') {
            this.state.hasKeyError = true;
        } else if (this.state.appId === '') {
            this.state.hasAppIdError = true;
        } else {
            this.props.submitCredentials(this.state.key, this.state.appId);
        }
    }
}
UnsplashCredentials.template = 'web_unsplash.UnsplashCredentials';

export class UnsplashError extends Component {}
UnsplashError.template = 'web_unsplash.UnsplashError';
UnsplashError.components = {
    UnsplashCredentials,
};

patch(ImageSelector.prototype, {
    setup() {
        super.setup();
        this.unsplash = useService('unsplash');
        this.keepLastUnsplash = new KeepLast();

        this.state.unsplashRecords = [];
        this.state.isFetchingUnsplash = false;
        this.state.isMaxed = false;
        this.state.unsplashError = null;
        this.state.useUnsplash = true;
        this.NUMBER_OF_RECORDS_TO_DISPLAY = 30;

        this.errorMessages = {
            'key_not_found': {
                title: _t("Setup Unsplash to access royalty free photos."),
                subtitle: "",
            },
            401: {
                title: _t("Unauthorized Key"),
                subtitle: _t("Please check your Unsplash access key and application ID."),
            },
            403: {
                title: _t("Search is temporarily unavailable"),
                subtitle: _t("The max number of searches is exceeded. Please retry in an hour or extend to a better account."),
            },
        };
    },

    get canLoadMore() {
        if (this.state.searchService === 'all') {
            return super.canLoadMore || this.state.needle && !this.state.isMaxed && !this.state.unsplashError;
        } else if (this.state.searchService === 'unsplash') {
            return this.state.needle && !this.state.isMaxed && !this.state.unsplashError;
        }
        return super.canLoadMore;
    },

    get hasContent() {
        if (this.state.searchService === 'all') {
            return super.hasContent || !!this.state.unsplashRecords.length;
        } else if (this.state.searchService === 'unsplash') {
            return !!this.state.unsplashRecords.length;
        }
        return super.hasContent;
    },

    get errorTitle() {
        if (this.errorMessages[this.state.unsplashError]) {
            return this.errorMessages[this.state.unsplashError].title;
        }
        return _t("Something went wrong");
    },

    get errorSubtitle() {
        if (this.errorMessages[this.state.unsplashError]) {
            return this.errorMessages[this.state.unsplashError].subtitle;
        }
        return _t("Please check your internet connection or contact administrator.");
    },

    get selectedRecordIds() {
        return this.props.selectedMedia[this.props.id].filter(media => media.mediaType === 'unsplashRecord').map(({ id }) => id);
    },

    get isFetching() {
        return super.isFetching || this.state.isFetchingUnsplash;
    },

    get combinedRecords() {
        /**
         * Creates an array with alternating elements from two arrays.
         *
         * @param {Array} a
         * @param {Array} b
         * @returns {Array} alternating elements from a and b, starting with
         *     an element of a
         */
        function alternate(a, b) {
            return [
                a.map((v, i) => i < b.length ? [v, b[i]] : v),
                b.slice(a.length),
            ].flat(2);
        }
        return alternate(this.state.unsplashRecords, this.state.libraryMedia);
    },

    get allAttachments() {
        return [...super.allAttachments, ...this.state.unsplashRecords];
    },

    // It seems that setters are mandatory when patching a component that
    // extends another component.
    set canLoadMore(_) {},
    set hasContent(_) {},
    set isFetching(_) {},
    set selectedMediaIds(_) {},
    set attachmentsDomain(_) {},
    set errorTitle(_) {},
    set errorSubtitle(_) {},
    set selectedRecordIds(_) {},

    async fetchUnsplashRecords(offset) {
        if (!this.state.needle) {
            return { records: [], isMaxed: false };
        }
        this.state.isFetchingUnsplash = true;
        try {
            const { isMaxed, images } = await this.unsplash.getImages(this.state.needle, offset, this.NUMBER_OF_RECORDS_TO_DISPLAY, this.props.orientation);
            this.state.isFetchingUnsplash = false;
            this.state.unsplashError = false;
            // Ignore duplicates.
            const existingIds = this.state.unsplashRecords.map(existing => existing.id);
            const newImages = images.filter(record => !existingIds.includes(record.id));
            const records = newImages.map(record => {
                const url = new URL(record.urls.regular);
                // In small windows, row height could get quite a bit larger than the min, so we keep some leeway.
                url.searchParams.set('h', 2 * this.MIN_ROW_HEIGHT);
                url.searchParams.delete('w');
                return Object.assign({}, record, {
                    url: url.toString(),
                    mediaType: 'unsplashRecord',
                });
            });
            return { isMaxed, records };
        } catch (e) {
            this.state.isFetchingUnsplash = false;
            if (e === 'no_access') {
                this.state.useUnsplash = false;
            } else {
                this.state.unsplashError = e;
            }
            return { records: [], isMaxed: true };
        }
    },

    async loadMore(...args) {
        await super.loadMore(...args);
        return this.keepLastUnsplash.add(this.fetchUnsplashRecords(this.state.unsplashRecords.length)).then(({ records, isMaxed }) => {
            // This is never reached if another search or loadMore occurred.
            this.state.unsplashRecords.push(...records);
            this.state.isMaxed = isMaxed;
        });
    },

    async search(...args) {
        await super.search(...args);
        await this.searchUnsplash();
    },

    async searchUnsplash() {
        if (!this.state.needle) {
            this.state.unsplashError = false;
            this.state.unsplashRecords = [];
            this.state.isMaxed = false;
        }
        return this.keepLastUnsplash.add(this.fetchUnsplashRecords(0)).then(({ records, isMaxed }) => {
            // This is never reached if a new search occurred.
            this.state.unsplashRecords = records;
            this.state.isMaxed = isMaxed;
        });
    },

    async onClickRecord(media) {
        this.props.selectMedia({ ...media, mediaType: 'unsplashRecord', query: this.state.needle });
        if (!this.props.multiSelect) {
            await this.props.save();
        }
    },

    async submitCredentials(key, appId) {
        this.state.unsplashError = null;
        await this.rpc('/web_unsplash/save_unsplash', { key, appId });
        await this.searchUnsplash();
    },
});
ImageSelector.components = {
    ...ImageSelector.components,
    UnsplashError,
};

patch(MediaDialog.prototype, {
    setup() {
        super.setup();

        this.uploadService = useService('upload');
    },

    async save() {
        const selectedImages = this.selectedMedia[TABS.IMAGES.id];
        if (selectedImages) {
            const unsplashRecords = selectedImages.filter(media => media.mediaType === 'unsplashRecord');
            if (unsplashRecords.length) {
                await this.uploadService.uploadUnsplashRecords(unsplashRecords, { resModel: this.props.resModel, resId: this.props.resId }, (attachments) => {
                    this.selectedMedia[TABS.IMAGES.id] = this.selectedMedia[TABS.IMAGES.id].filter(media => media.mediaType !== 'unsplashRecord');
                    this.selectedMedia[TABS.IMAGES.id] = this.selectedMedia[TABS.IMAGES.id].concat(attachments.map(attachment => ({...attachment, mediaType: 'attachment'})));
                });
            }
        }
        return super.save(...arguments);
    },
});

patch(uploadService, {
    start(env, { rpc }) {
        const service = super.start(...arguments);
        return {
            ...service,
            async uploadUnsplashRecords(records, { resModel, resId }, onUploaded) {
                service.incrementId();
                const file = service.addFile({
                    id: service.fileId,
                    name: records.length > 1 ?
                    _t("Uploading %s '%s' images.", records.length, records[0].query) :
                    _t("Uploading '%s' image.", records[0].query),
                });

                try {
                    const urls = {};
                    for (const record of records) {
                        const _1920Url = new URL(record.urls.regular);
                        _1920Url.searchParams.set('w', '1920');
                        urls[record.id] = {
                            url: _1920Url.href,
                            download_url: record.links.download_location,
                            description: record.alt_description,
                        };
                    }

                    const xhr = new XMLHttpRequest();
                    xhr.upload.addEventListener('progress', ev => {
                        const rpcComplete = ev.loaded / ev.total * 100;
                        file.progress = rpcComplete;
                    });
                    xhr.upload.addEventListener('load', function () {
                        // Don't show yet success as backend code only starts now
                        file.progress = 100;
                    });
                    const attachments = await rpc('/web_unsplash/attachment/add', {
                        'res_id': resId,
                        'res_model': resModel,
                        'unsplashurls': urls,
                        'query': records[0].query,
                    }, {xhr});

                    if (attachments.error) {
                        file.hasError = true;
                        file.errorMessage = attachments.error;
                    } else {
                        file.uploaded = true;
                        await onUploaded(attachments);
                    }
                    setTimeout(() => service.deleteFile(file.id), AUTOCLOSE_DELAY);
                } catch (error) {
                    file.hasError = true;
                    setTimeout(() => service.deleteFile(file.id), AUTOCLOSE_DELAY);
                    throw error;
                }
            }
        };
    }
});

```

## File: static\src\components\media_dialog\image_selector.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
<t t-name="web_unsplash.UnsplashError">
    <div class="alert alert-info w-100">
        <h4><t t-esc="props.title"/></h4>
        <p><t t-esc="props.subtitle"/></p>
        <UnsplashCredentials t-if="props.showCredentials" submitCredentials="props.submitCredentials" hasCredentialsError="props.hasCredentialsError"/>
    </div>
</t>

<t t-name="web_unsplash.UnsplashCredentials">
    <div class="d-flex align-items-center flex-wrap">
        <a href="https://www.odoo.com/documentation/17.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-access-key"
           class="me-1" target="_blank">Get an Access key</a>
        and paste it here:
        <input type="text"
            class="o_input o_required_modifier form-control w-auto mx-2"
            id="accessKeyInput"
            placeholder="Paste your access key here"
            t-model="state.key"
            t-on-input="() => this.state.hasKeyError = false"
            t-att-class="{ 'is-invalid': state.hasKeyError }"/>
        and paste
        <a href="https://www.odoo.com/documentation/17.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-application-id"
           class="mx-1" target="_blank">Application ID</a>
        here:
        <input type="text"
            class="o_input o_required_modifier form-control w-auto ms-2"
            placeholder="Paste your application ID here"
            t-model="state.appId"
            t-on-input="() => this.state.hasAppIdError = false"
            t-att-class="{ 'is-invalid': state.hasAppIdError }"/>
        <button type="button" class="btn btn-primary w-auto ms-3 p-auto save_unsplash" t-on-click="() => this.submitCredentials()">Apply</button>
    </div>
</t>

<t t-inherit="web_editor.ExternalImage" t-inherit-mode="extension">
    <xpath expr="//t[@t-if]" position="after">
        <t t-elif="record.mediaType == 'unsplashRecord'">
            <AutoResizeImage src="record.url"
                author="record.user.name"
                authorLink="record.user.links.html"
                name="record.user.name"
                title="record.user.name"
                altDescription="record.alt_description"
                selected="this.selectedRecordIds.includes(record.id)"
                onImageClick="() => this.onClickRecord(record)"
                minRowHeight="MIN_ROW_HEIGHT"
                onLoaded="(imgEl) => this.onImageLoaded(imgEl, record)"/>
        </t>
    </xpath>
</t>

<t t-name="web_unsplash.ImagesListTemplate" t-inherit="web_editor.ImagesListTemplate" t-inherit-mode="extension">
    <xpath expr="//t[@id='o_we_media_library_images']" position="replace">
        <t id='o_we_media_library_images' t-if="['all', 'unsplash', 'media-library'].includes(state.searchService)">
            <t t-foreach="combinedRecords" t-as="record" t-key="record.id">
                <t t-call="web_editor.ExternalImage"/>
            </t>
        </t>
    </xpath>
</t>

<t t-inherit="web_editor.FileSelector" t-inherit-mode="extension">
    <xpath expr="//div[@name='load_more_attachments']" position="before">
        <div t-if="state.unsplashError" class="d-flex mt-2 unsplash_error">
            <UnsplashError
                title="errorTitle"
                subtitle="errorSubtitle"
                showCredentials="['key_not_found', 401].includes(state.unsplashError)"
                submitCredentials="(key, appId)  => this.submitCredentials(key, appId)"
                hasCredentialsError="state.unsplashError === 401"/>
        </div>
    </xpath>
</t>

<t t-inherit="web_editor.FileSelectorControlPanel" t-inherit-mode="extension">
    <xpath expr="//option[@value='media-library']" position="after">
        <option t-if="props.useUnsplash" t-att-selected="props.searchService === 'unsplash'" value="unsplash">Photos (via Unsplash)</option>
    </xpath>
</t>

<t t-inherit="web_editor.FileSelector" t-inherit-mode="extension">
    <xpath expr="//FileSelectorControlPanel" position="attributes">
        <attribute name="useUnsplash">state.useUnsplash</attribute>
    </xpath>
</t>
</templates>

```

## File: static\src\js\unsplash_beacon.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.UnsplashBeacon = publicWidget.Widget.extend({
    // /!\ To adapt the day the beacon makes sense for backend customizations
    selector: '#wrapwrap',

    init() {
        this._super(...arguments);
        this.rpc = this.bindService("rpc");
    },

    /**
     * @override
     */
    start: function () {
        var unsplashImages = Array.from(this.$('img[src*="/unsplash/"]')).map((img) => {
            // get image id from URL (`http://www.domain.com:1234/unsplash/xYdf5feoI/lion.jpg` -> `xYdf5feoI`)
            return img.src.split('/unsplash/')[1].split('/')[0];
        });
        if (unsplashImages.length) {
            this.rpc('/web_unsplash/get_app_id').then(function (appID) {
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

```

## File: static\src\services\unsplash_service.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';

export const unsplashService = {
    dependencies: ['rpc'],
    async start(env, { rpc }) {
        const _cache = {};
        return {
            async getImages(query, offset = 0, pageSize = 30, orientation) {
                const from = offset;
                const to = offset + pageSize;
                // Use orientation in the cache key to not show images in cache
                // when using the same query word but changing the orientation
                let cachedData = orientation ? _cache[query + orientation] : _cache[query];

                if (cachedData && (cachedData.images.length >= to || (cachedData.totalImages !== 0 && cachedData.totalImages < to))) {
                    return { images: cachedData.images.slice(from, to), isMaxed: to > cachedData.totalImages };
                }
                cachedData = await this._fetchImages(query, orientation);
                return { images: cachedData.images.slice(from, to), isMaxed: to > cachedData.totalImages };
            },
            /**
             * Fetches images from unsplash and stores it in cache
             */
            async _fetchImages(query, orientation) {
                const key = orientation ? query + orientation : query;
                if (!_cache[key]) {
                    _cache[key] = {
                        images: [],
                        maxPages: 0,
                        totalImages: 0,
                        pageCached: 0
                    };
                }
                const cachedData = _cache[key];
                const payload = {
                    query: query,
                    page: cachedData.pageCached + 1,
                    per_page: 30, // max size from unsplash API
                };
                if (orientation) {
                    payload.orientation = orientation;
                }
                const result = await rpc('/web_unsplash/fetch_images', payload);
                if (result.error) {
                    return Promise.reject(result.error);
                }
                cachedData.pageCached++;
                cachedData.images.push(...result.results);
                cachedData.maxPages = result.total_pages;
                cachedData.totalImages = result.total;
                return cachedData;
            },
        };
    },
};

registry.category('services').add('unsplash', unsplashService);

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
                <div invisible="not module_web_unsplash">
                    <div class="content-group mt16">
                        <label for="unsplash_access_key" class="o_light_label mr8"/>
                        <field name="unsplash_access_key"/>
                    </div>
                    <div class="content-group">
                        <label for="unsplash_app_id" class="o_light_label mr8"/>
                        <field name="unsplash_app_id"/>
                    </div>
                    <div>
                        <a href="https://www.odoo.com/documentation/17.0/applications/websites/website/optimize/unsplash.html#generate-an-unsplash-access-key" class="oe_link" target="_blank">
                            <i class="oi oi-arrow-right"/> Generate an Access Key
                        </a>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

