# Odoo Module: cloud_storage

Category: Technical Settings

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Cloud Storage",
    "summary": """Store chatter attachments in the cloud""",
    "category": "Technical Settings",
    "version": "1.0",
    "depends": ["mail"],
    "data": [
        "views/settings.xml",
    ],
    'assets': {
        'web.assets_backend': [
            'cloud_storage/static/src/core/common/**/*',
            'cloud_storage/static/src/**/web_portal/**/*',
        ],
        'mail.assets_public': [
            'cloud_storage/static/src/core/common/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.http import route, request
from odoo.addons.mail.models.discuss.mail_guest import add_guest_to_context
from odoo.addons.mail.controllers.attachment import AttachmentController


class CloudAttachmentController(AttachmentController):
    @route()
    @add_guest_to_context
    def mail_attachment_upload(self, ufile, thread_id, thread_model, is_pending=False, **kwargs):
        is_cloud_storage = kwargs.get('cloud_storage')
        if (is_cloud_storage and not request.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider')):
            return request.make_json_response({
                'error': _('Cloud storage configuration has been changed. Please refresh the page.')
            })

        response = super().mail_attachment_upload(ufile, thread_id, thread_model, is_pending, **kwargs)

        if not is_cloud_storage:
            return response

        data = response.json
        if data.get("error"):
            return response

        # append upload url to the response to allow the client to directly
        # upload files to the cloud storage
        attachment = request.env["ir.attachment"].browse(data["data"]["ir.attachment"][0]["id"]).sudo()
        data["upload_info"] = attachment._generate_cloud_storage_upload_info()
        return request.make_json_response(data)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import attachment

```

## File: data\neutralize.sql

```sql
DELETE FROM ir_config_parameter WHERE key = 'cloud_storage_provider';

```

## File: models\ir_attachment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import uuid

from odoo import models, fields, _
from odoo.exceptions import UserError
from odoo.http import Stream


class CloudStorageAttachment(models.Model):
    _inherit = 'ir.attachment'
    _cloud_storage_upload_url_time_to_expiry = 300  # 300 seconds
    _cloud_storage_download_url_time_to_expiry = 300  # 300 seconds

    type = fields.Selection(
        selection_add=[('cloud_storage', 'Cloud Storage')],
        ondelete={'cloud_storage': 'set url'}
    )

    def _to_http_stream(self):
        if (self.type == 'cloud_storage' and
              self.env['res.config.settings']._get_cloud_storage_configuration()):
            self.ensure_one()
            info = self._generate_cloud_storage_download_info()
            stream = Stream(type='url', url=info['url'])
            if 'time_to_expiry' in info:
                # cache the redirection until 10 seconds before the expiry
                stream.max_age = max(info['time_to_expiry'] - 10, 0)
            return stream
        return super()._to_http_stream()

    def _post_add_create(self, **kwargs):
        super()._post_add_create(**kwargs)
        if kwargs.get('cloud_storage'):
            if not self.env['ir.config_parameter'].sudo().get_param('cloud_storage_provider'):
                raise UserError(_('Cloud Storage is not enabled'))
            for record in self:
                record.write({
                    'raw': False,
                    'type': 'cloud_storage',
                    'url': record._generate_cloud_storage_url(),
                })

    def _generate_cloud_storage_blob_name(self):
        """
        Generate a unique blob name for the attachment
        :param attachment: an ir.attachment record
        :return: A unique blob name str
        """
        return f'{self.id}/{uuid.uuid4()}/{self.name}'

    # Implement the following methods for each cloud storage provider.
    def _generate_cloud_storage_url(self):
        """
        Generate a cloud blob url without signature or token for the attachment.
        This url is only used to identify the cloud blob.
        :param attachment: an ir.attachment record
        :return: A cloud blob url str
        """
        raise NotImplementedError()

    def _generate_cloud_storage_download_info(self):
        """
        Generate the download info for the public client to directly download
        the attachment's blob from the cloud storage.
        :param attachment: an ir.attachment record
        :return: An download_info dictionary containing:
            * download_url: cloud storage url with permission to download the file
            * time_to_expiry: the time in seconds before the download url expires
        """
        raise NotImplementedError()

    def _generate_cloud_storage_upload_info(self):
        """
        Generate the upload info for the public client to directly upload a
        file to the cloud storage.
        :param attachment: an ir.attachment record
        :return: An upload_info dictionary containing:
            * upload_url: cloud storage url with permission to upload the file
            * method: the request method used to upload the file
            * response_status: the status of the response for a successful
                upload request
            * [Optionally] headers: a dictionary of headers to be added to the
                upload request
        """
        raise NotImplementedError()

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from .res_config_settings import DEFAULT_CLOUD_STORAGE_MIN_FILE_SIZE


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        res = super().session_info()
        ICP = self.env['ir.config_parameter'].sudo()
        if ICP.get_param('cloud_storage_provider'):
            res['cloud_storage_min_file_size'] = ICP.get_param('cloud_storage_min_file_size', DEFAULT_CLOUD_STORAGE_MIN_FILE_SIZE)
        return res

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _
from odoo.exceptions import UserError


DEFAULT_CLOUD_STORAGE_MIN_FILE_SIZE = 20_000_000


class CloudStorageSettings(models.TransientModel):
    """
    Instructions:
    cloud_storage_provider: Once set, new attachments from the web client can
        be created as cloud storage attachments. Once changed, all attachments
        stored in the old cloud storage provider cannot be fetched. Please
        migrate those cloud storage blobs and the url field of their
        ir.attachment records before change.
    cloud_storage_mim_file_size: a soft limit for the file size that can be
        uploaded as the cloud storage attachments for web client.
    """
    _inherit = 'res.config.settings'

    cloud_storage_provider = fields.Selection(
        selection=[],
        string='Cloud Storage Provider for new attachments',
        config_parameter='cloud_storage_provider',
    )

    cloud_storage_min_file_size = fields.Integer(
        string='Minimum File Size (bytes)',
        help='''webclient can upload files larger than the minimum file size
        (in bytes) as url attachments to the server and then upload the file to
        the cloud storage.''',
        config_parameter='cloud_storage_min_file_size',
        default=DEFAULT_CLOUD_STORAGE_MIN_FILE_SIZE,
    )

    def _setup_cloud_storage_provider(self):
        """
        Setup the cloud storage provider and check the validity of the account
        info after saving the config in settings.
        return: None
        """
        pass

    def _get_cloud_storage_configuration(self):
        """
        Return the configuration for the cloud storage provider. If the cloud
        storage provider is not fully configured, return an empty dict.
        :return: A configuration dict
        """
        return {}

    def _check_cloud_storage_uninstallable(self):
        """
        Check if the cloud storages provider is used by any attachments
        :raise UserError: when the cloud storage provider cannot be uninstalled
        """
        pass

    def set_values(self):
        ICP = self.env['ir.config_parameter']
        cloud_storage_configuration_before = self._get_cloud_storage_configuration()
        cloud_storage_provider_before = ICP.get_param('cloud_storage_provider')
        if cloud_storage_provider_before and self.cloud_storage_provider != cloud_storage_provider_before:
            self._check_cloud_storage_uninstallable()
        super().set_values()
        cloud_storage_configuration = self._get_cloud_storage_configuration()
        if not cloud_storage_configuration and self.cloud_storage_provider:
            raise UserError(_('Please configure the Cloud Storage before enabling it'))
        if cloud_storage_configuration and cloud_storage_configuration != cloud_storage_configuration_before:
            self._setup_cloud_storage_provider()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_attachment
from . import ir_http
from . import res_config_settings

```

## File: static\src\chatter\web_portal\chatter_patch.js

```javascript
import { Chatter } from "@mail/chatter/web_portal/chatter";

import { patch } from "@web/core/utils/patch";
import { session } from "@web/session";

patch(Chatter.prototype, {
    setup() {
        super.setup();
        this.cloudStorageUsable = session.cloud_storage_min_file_size !== undefined;
    },
});

```

## File: static\src\chatter\web_portal\chatter_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.Chatter" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o-mail-Chatter-topbar')]//FileUploader" position="attributes">
            <attribute name="checkSize">!cloudStorageUsable</attribute>
        </xpath>
        
        <xpath expr="//div[hasclass('o-mail-AttachmentBox')]//FileUploader" position="attributes">
            <attribute name="checkSize">!cloudStorageUsable</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\common\attachment_upload_service_patch.js

```javascript
import { AttachmentUploadService } from "@mail/core/common/attachment_upload_service";

import { patch } from "@web/core/utils/patch";
import { session } from "@web/session";
import { _t } from "@web/core/l10n/translation";

patch(AttachmentUploadService.prototype, {
    setup(env, services) {
        super.setup(env, services);
        this.uploadingCloudFiles = new Map();
        window.addEventListener('beforeunload', () => 
            this.abortByAttachmentId.forEach(abort => abort())
        );
    },

    _processLoaded(thread, composer, { data, upload_info }, tmpId, def) {
        if (!upload_info) {
            super._processLoaded(...arguments);
            return;
        }
        const removeAttachment = () => {
            const { Attachment } = this.store.insert(data);
            const [attachment] = Attachment;
            attachment.remove();
        }
        const xhr = new window.XMLHttpRequest();
        this.abortByAttachmentId.set(tmpId, xhr.abort.bind(xhr));
        const file = this.uploadingCloudFiles.get(tmpId);

        xhr.open(upload_info.method, upload_info.url);
        for (const [key, value] of Object.entries(upload_info.headers || {})) {
            xhr.setRequestHeader(key, value);
        }

        xhr.onload = () => {
            if (!this.uploadingAttachmentIds.has(tmpId)) {
                return;
            }
            if (xhr.status === 403) {
                // usually it is because the token of the server for the cloud storage is expired
                this.notificationService.add(
                    _t("You are not allowed to upload file to the cloud storage"),
                    { type: "danger" }
                );
                removeAttachment();
                def.resolve();
                this._cleanupUploading(tmpId);
                return;
            }
            // google returns 200, azure returns 201
            if (xhr.status !== upload_info.response_status) {
                this.notificationService.add(_t("Cloud storage error"), { type: "danger" });
                removeAttachment();
                def.resolve();
                this._cleanupUploading(tmpId);
                return;
            }
            super._processLoaded(...arguments);
        };

        xhr.onerror = () => {
            if (!this.uploadingAttachmentIds.has(tmpId)) {
                return;
            }
            // usually it is because the CORS config for PUT is disallowed for the cloud storage
            this.notificationService.add(_t("Cloud storage error"), { type: "danger" });
            removeAttachment();
            this._cleanupUploading(tmpId);
        };

        xhr.onabort = () => {
            removeAttachment();
            this._cleanupUploading(tmpId);
        };

        xhr.send(file);
    },

    _cleanupUploading(tmpId) {
        super._cleanupUploading(tmpId);
        this.uploadingCloudFiles.delete(tmpId);
    },

    async _upload(thread, composer, file, options, tmpId, tmpURL) {
        if (
            session.cloud_storage_min_file_size !== undefined &&
            file.size > session.cloud_storage_min_file_size
        ) {
            // store the file in the this.uploadingCloudFiles map
            this.uploadingCloudFiles.set(tmpId, file);
            // replace the file to a dummy file with the same name and type
            // and send the dummy file to the server without real content overhead
            file = new File([new Blob([])], file.name, { type: file.type });
            options = options ? { ...options, cloud_storage: true } : { cloud_storage: true };
        }
        return super._upload(thread, composer, file, options, tmpId, tmpURL);
    },

    _buildFormData(formData, tmpURL, thread, composer, tmpId, options) {
        super._buildFormData(...arguments);
        if (options?.cloud_storage) {
            formData.append("cloud_storage", true);
        }
        return formData;
    },
});

```

## File: static\src\core\common\composer_patch.js

```javascript
import { Composer } from "@mail/core/common/composer";

import { patch } from "@web/core/utils/patch";
import { session } from "@web/session";

patch(Composer.prototype, {
    setup() {
        super.setup();
        this.cloudStorageUsable = session.cloud_storage_min_file_size !== undefined;
    },
});

```

## File: static\src\core\common\composer_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.Composer.attachFiles" t-inherit-mode="extension">
        <xpath expr="//FileUploader" position="attributes">
            <attribute name="checkSize">!cloudStorageUsable</attribute>
        </xpath>
    </t>
</templates>

```

## File: views\settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="cloud_storage_config_settings_view_form" model="ir.ui.view">
        <field name="name">cloud_storage_config_settings_view_form</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="70"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside" >
                <app data-string="Cloud Storage" string="Cloud Storage" name="cloud_storage">
                    <block title="Cloud Storage Settings">
                        <setting id="cloud_storage_provider" help="Select the cloud storage provider to store new attachments.">
                            <field name="cloud_storage_provider"/>
                        </setting>
                        <setting help="Minimum size(bytes) for attachments to be stored in the cloud storage"
                                 id="cloud_storage_min_file_size">
                            <field name="cloud_storage_min_file_size"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

</odoo>

```

