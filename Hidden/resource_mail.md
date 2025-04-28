# Odoo Module: resource_mail

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Resource Mail',
    'version': '1.0',
    'category': 'Hidden',
    'description': """Integrate features developped in Mail in use case involving resources instead of users""",
    'depends': ['resource', 'mail'],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'resource_mail/static/src/**/*',
        ],
        'web.assets_unit_tests': [
            'resource_mail/static/tests/**/*',
            ('remove', 'resource_mail/static/tests/legacy/**/*'),
        ],
        'web.qunit_suite_tests': [
            'resource_mail/static/tests/legacy/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\resource_resource.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResourceResource(models.Model):
    _inherit = 'resource.resource'

    im_status = fields.Char(related='user_id.im_status')

    def get_avatar_card_data(self, fields):
        return self._read_format(fields)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import resource_resource

```

## File: static\src\components\avatar_card_resource\avatar_card_resource_popover.js

```javascript
/** @odoo-module **/

import { onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";
import { useOpenChat } from "@mail/core/web/open_chat_hook";
import { AvatarCardPopover } from "@mail/discuss/web/avatar_card/avatar_card_popover";

export class AvatarCardResourcePopover extends AvatarCardPopover {
    static template = "resource_mail.AvatarCardResourcePopover";

    static props = {
        ...AvatarCardPopover.props,
        recordModel: {
            type: String,
            optional: true,
        },
    };

    static defaultProps = {
        ...AvatarCardPopover.defaultProps,
        recordModel: "resource.resource",
    };

    setup() {
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.openChat = useOpenChat("res.users");
        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        [this.record] = await this.orm.call('resource.resource', 'get_avatar_card_data', [[this.props.id], this.fieldNames], {});
        await Promise.all(this.loadAdditionalData());
    }

    loadAdditionalData() {
        // To use when overriden in other modules to load additional data, returns promise(s)
        return [];
    }

    get fieldNames() {
        const excludedFields = new Set(["partner_id"]);
        return super.fieldNames
            .concat(["user_id", "resource_type"])
            .filter((field) => !excludedFields.has(field));
    }

    get email() {
        return this.record.email;
    }

    get phone() {
        return this.record.phone;
    }

    get displayAvatar() {
        return this.record.user_id?.length;
    }

    get showViewProfileBtn() {
        return false;
    }

    get userId() {
        return this.record.user_id[0];
    }
}

```

## File: static\src\components\avatar_card_resource\avatar_card_resource_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="resource_mail.AvatarCardResourcePopover" t-inherit="mail.AvatarCardPopover">
        <xpath expr="//span[hasclass('o_avatar')]/img" position="attributes">
            <attribute name="t-attf-src" t-if="displayAvatar">/web/image/{{ props.recordModel }}/{{ props.id }}/avatar_128</attribute>
        </xpath>
        <xpath expr="//span[hasclass('o_card_avatar_im_status')]" position="replace">
            <span t-if="record.user_id" name="icon" class="o_card_avatar_im_status position-absolute d-flex align-items-center justify-content-center o_user_im_status bg-inherit">
                <i t-if="record.im_status === 'online'" class="fa fa-fw fa-circle text-success" title="Online" role="img" aria-label="User is online"/>
                <i t-elif="record.im_status === 'away'" class="fa fa-fw fa-circle text-warning" title="Idle" role="img" aria-label="User is idle"/>
                <i t-elif="record.im_status === 'offline'" class="fa fa-fw fa-circle-o text-700" title="Offline" role="img" aria-label="User is offline"/>
                <i t-elif="record.im_status === 'bot'" class="fa fa-fw fa-heart text-success" title="Bot" role="img" aria-label="User is a bot"/>
                <i t-elif="!record.im_status" class="fa fa-fw fa-question-circle" title="No IM status available"/>
            </span>
        </xpath>
        <xpath expr="//div[hasclass('o_card_user_infos')]/span" position="attributes">
            <attribute name="t-esc">record.name</attribute>
        </xpath>
        <xpath expr="//div[hasclass('o_avatar_card_buttons')]/button" position="attributes">
            <attribute name="t-if">record.user_id and !record.share</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\many2many_avatar_resource\many2many_avatar_resource_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { usePopover } from "@web/core/popover/popover_hook";
import {
    Many2ManyTagsAvatarUserField,
    many2ManyTagsAvatarUserField,
    Many2ManyAvatarUserTagsList,
} from "@mail/views/web/fields/many2many_avatar_user_field/many2many_avatar_user_field";
import { AvatarMany2XAutocomplete } from "@web/views/fields/relational_utils";
import { AvatarCardResourcePopover } from "@resource_mail/components/avatar_card_resource/avatar_card_resource_popover";
import { Domain } from "@web/core/domain";


export class AvatarResourceMany2XAutocomplete extends AvatarMany2XAutocomplete {
    get optionsSource() {
        return {
            ...super.optionsSource,
            optionTemplate: "resource_mail.AvatarResourceMany2XAutocomplete",
        };
    }

    /**
     * @override
     */
    search(request) {
        return this.orm.call(
            this.props.resModel,
            "search_read",
            [this.getDomain(request), ["id", "display_name", "resource_type", "color"]],
            {
                context: this.props.context,
                limit: this.props.searchLimit + 1,
            }
        );
    }

    /**
     * @override
     */
    getDomain(request) {
        return Domain.and([[["name", "ilike", request]], this.props.getDomain()]).toList(
            this.props.context
        );
    }

    /**
     * @override
     */
    mapRecordToOption(result) {
        return {
            resModel: this.props.resModel,
            value: result.id,
            resourceType: result.resource_type,
            label: result.display_name,
            color: result.color,
        };
    }
}

class Many2ManyAvatarResourceTagsList extends Many2ManyAvatarUserTagsList {
    static template = "resource_mail.Many2ManyAvatarResourceTagsList";
}

export class Many2ManyAvatarResourceField extends Many2ManyTagsAvatarUserField {
    setup() {
        super.setup(...arguments);
        if (this.relation == "resource.resource") {
            this.avatarCard = usePopover(AvatarCardResourcePopover);
        }
    }

    static components = {
        ...super.components,
        Many2XAutocomplete: AvatarResourceMany2XAutocomplete,
        TagsList: Many2ManyAvatarResourceTagsList,
    };

    displayAvatarCard(record) {
        return !this.env.isSmall && this.relation === "resource.resource" && record.data.resource_type === "user";
    }

    getTagProps(record) {
        return {
            ...super.getTagProps(...arguments),
            icon: record.data.resource_type === "user" ? null : "fa-wrench",
            img: record.data.resource_type === "user"
                ? `/web/image/${this.relation}/${record.resId}/avatar_128`
                : null,
        };
    }
}

export const many2ManyAvatarResourceField = {
    ...many2ManyTagsAvatarUserField,
    component: Many2ManyAvatarResourceField,
    additionalClasses: ["o_field_many2many_tags_avatar"],
    relatedFields: (fieldInfo) => {
        return [
            ...many2ManyTagsAvatarUserField.relatedFields(fieldInfo),
            {
                name: "resource_type",
                type: "selection",
            },
        ];
    },
};

registry.category("fields").add("many2many_avatar_resource", many2ManyAvatarResourceField);

```

## File: static\src\views\fields\many2many_avatar_resource\many2many_avatar_resource_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="resource_mail.AvatarResourceMany2XAutocomplete" t-inherit="web.AvatarMany2XAutocomplete">
        <xpath expr="//span[hasclass('o_avatar_many2x_autocomplete')]/img" position="before">
            <i t-if="option.resourceType === 'material'" class="o_material_resource fa fa-wrench rounded text-center me-2
            d-flex align-items-center justify-content-center" t-attf-class="o_colorlist_item_color_{{ option.color }}"/>
        </xpath>
        <xpath expr="//span[hasclass('o_avatar_many2x_autocomplete')]/img" position="attributes">
            <attribute name="t-if" add="&amp;&amp; option.resourceType !== 'material'" separator=" "/>
        </xpath>
    </t>

    <t t-name="resource_mail.Many2ManyAvatarResourceTagsList" t-inherit="mail.Many2ManyAvatarUserTagsList">
        <xpath expr="//span[hasclass('o_tag')]/i" position="attributes">
            <attribute name="t-on-click.stop.prevent">tag.onImageClicked</attribute>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\many2one_avatar_resource\many2one_avatar_resource_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { usePopover } from "@web/core/popover/popover_hook";
import {
    Many2OneAvatarUserField,
    many2OneAvatarUserField,
} from "@mail/views/web/fields/many2one_avatar_user_field/many2one_avatar_user_field";
import { AvatarCardResourcePopover } from "@resource_mail/components/avatar_card_resource/avatar_card_resource_popover";
import { AvatarResourceMany2XAutocomplete } from "@resource_mail/views/fields/many2many_avatar_resource/many2many_avatar_resource_field";


const ExtendMany2OneAvatarToResource = (T) => class extends T {
    // We choose to extend Many2One_avatar_user instead of patching it as field dependencies need to be added on the widget to manage resources
    setup() {
        super.setup();
        this.avatarCard = usePopover(AvatarCardResourcePopover);
    }

    get displayAvatarCard() {
        return !this.env.isSmall && this.relation === "resource.resource" && this.props.record.data.resource_type === "user";
    }
};


export class Many2OneAvatarResourceField extends ExtendMany2OneAvatarToResource(Many2OneAvatarUserField) {
    static template = "resource_mail.Many2OneAvatarResourceField";
    static components = {
        ...super.components,
        Many2XAutocomplete: AvatarResourceMany2XAutocomplete,
    };
}

export const many2OneAvatarResourceField = {
    ...many2OneAvatarUserField,
    component: Many2OneAvatarResourceField,
    fieldDependencies: [
        {
            name: "resource_type", //to add in model that will use this widget for m2o field related to resource.resource record (as related field is only supported for x2m)
            type: "selection",
        },
    ],
};

registry.category("fields").add("many2one_avatar_resource", many2OneAvatarResourceField);

export class KanbanMany2OneAvatarResourceField extends ExtendMany2OneAvatarToResource(Many2OneAvatarResourceField) {
    static template = "resource_mail.KanbanMany2OneAvatarResourceField";
}

export const kanbanMany2OneAvatarResourceField = {
    ...many2OneAvatarResourceField,
    component: KanbanMany2OneAvatarResourceField,
};

registry.category("fields").add("kanban.many2one_avatar_resource", kanbanMany2OneAvatarResourceField);

```

## File: static\src\views\fields\many2one_avatar_resource\many2one_avatar_resource_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="resource_mail.Many2OneAvatarResourceField" t-inherit="mail.Many2OneAvatarUserField" t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_m2o_avatar')]" position="attributes">
            <attribute name="t-attf-class" add="{{ displayResourcePopover ? 'o_field_many2one_avatar': '' }}" separator=" "/>
        </xpath>
        <xpath expr="//span[hasclass('o_m2o_avatar')]/img" position="attributes">
            <attribute name="t-if">props.record.data.resource_type !== 'material' &amp;&amp; props.record.data[props.name] !== false</attribute>
        </xpath>
        <xpath expr="//span[hasclass('o_m2o_avatar')]/img" position="after">
            <t t-if="props.record.data.resource_type === 'material'">
                <span t-if="props.record.data[props.name] !== false"
                      t-att-title="props.record.data[props.name][1]"
                      class="d-inline-flex align-items-center justify-content-center rounded o_material_resource cursor-default me-1">
                    <i class="fa fa-wrench"/>
                </span>
                <span t-elif="!props.readonly" class="o_m2o_avatar_empty"></span>
            </t>
        </xpath>
    </t>

    <t t-name="resource_mail.KanbanMany2OneAvatarResourceField" t-inherit="mail.KanbanMany2OneAvatarUserField" t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_m2o_avatar')]/img" position="attributes">
            <attribute name="t-if">props.record.data.resource_type !== 'material' &amp;&amp; props.record.data[props.name] !== false</attribute>
        </xpath>
        <xpath expr="//span[hasclass('o_m2o_avatar')]/img" position="after">
            <t t-if="props.record.data.resource_type === 'material'">
                <span t-if="props.record.data[props.name] !== false"
                      t-att-title="props.record.data[props.name][1]"
                      class="d-inline-flex align-items-center justify-content-center rounded o_material_resource cursor-default me-1">
                    <i class="fa fa-wrench"/>
                </span>
                <span t-elif="!props.readonly" class="o_m2o_avatar_empty"></span>
            </t>
        </xpath>
    </t>
</templates>

```

