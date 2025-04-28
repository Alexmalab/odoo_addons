# Odoo Module: project_todo

Category: Productivity/To-Do

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

def _todo_uninstall(env):
    # The record rule project.task_visibility_rule needs to apply to all rights and not just read after uninstallation
    project_task_visibility_rule_rec = env.ref("project.task_visibility_rule")
    project_task_visibility_rule_rec.write({
        'perm_create': True,
        'perm_unlink': True,
        'perm_write': True,
    })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'To-Do',
    'version': '1.0',
    'category': 'Productivity/To-Do',
    'summary': 'Organize your work with memos and to-do lists',
    'sequence': 260,
    'depends': [
        'project',
    ],
    'auto_install': True,
    'data': [
        'security/ir.model.access.csv',
        'security/project_todo_security.xml',
        'data/mail_activity_type_data.xml',
        'data/todo_template.xml',
        'views/project_task_views.xml',
        'views/project_todo_menus.xml',
        'wizard/mail_activity_todo_create.xml',
    ],
    'installable': True,
    'application': True,
    'uninstall_hook': '_todo_uninstall',
    'assets': {
        'web.assets_backend': [
            'project_todo/static/src/components/**/*',
            'project_todo/static/src/scss/todo.scss',
            'project_todo/static/src/views/**/*',
            'project_todo/static/src/web/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_activity_data_reminder" model="mail.activity.type">
            <field name="name">Reminder</field>
            <field name="category">reminder</field>
            <field name="icon">fa-tasks</field>
            <field name="delay_count">0</field>
            <field name="sequence">20</field>
        </record>
        <record id="mail.mail_activity_data_todo" model="mail.activity.type">
            <field name="category">reminder</field>
        </record>
    </data>
</odoo>

```

## File: data\todo_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <template id="todo_user_onboarding">
<h1>
    <span style="font-size: 36px;">Hey <t t-out="object.name"/> &amp;#128075; <br />
    Welcome to the To-do app! </span>
</h1>
<p>
    <span style="font-size: 14px;">
        Use it to manage your urgent work, take notes on the go, and create tasks based on them. 
    </span>
</p>
<p><br /></p>
<h3>Using the editor</h3>
<hr />
<p>
    <span style="font-size: 14px;">This private to-do is for you to play around with.</span>
    <br />
    <span style="font-size: 14px;">Ready to give it a spin?</span>
</p>
<p>
    <span style="font-size: 14px;">Try the following</span>
</p>
<ul class="o_checklist">
    <li id="checkId-140908257281">
        <span style="font-size: 14px;">Click anywhere, and just start typing</span>
    </li>
    <li id="checkId-441216037148">
        <span style="font-size: 14px;">Press Ctrl-Z to undo any change</span>
    </li>
    <li id="checkId-948702291173">
        <span style="font-size: 14px;">Check this box to indicate it's done</span>
    </li>
    <li id="checkId-980463482772">
        <span style="font-size: 14px;">
            Select text to
            <font class="bg-o-color-2">Highlight</font>,
            <span style="text-decoration-line: line-through;">strikethrough</span>
            or
            <span style="font-weight: bolder;">style</span>
            <span style="font-style: italic; text-decoration-line: underline;">it</span>
        </span>
    </li>
    <li id="checkId-152369821198">
        <span style="font-size: 14px;">
            Below this list, try
            <span style="font-weight: bolder;">commands</span>
            by
            <span style="font-weight: bolder;">
                <font class="text-o-color-2">typing</font>
            </span>
            "<span style="font-weight: bolder;">/</span>"
        </span>
    </li>
    <li class="oe-nested" id="checkId-947916154443">
        <ul class="o_checklist">
            <li id="checkId-1422510558483">
                <span style="font-size: 14px;">
                    Add a checklist
                    (/<span style="font-style: italic;">checklist</span>)
                </span>
            </li>
            <li id="checkId-109605733655">
                <span style="font-size: 14px;">
                    Add a separator
                    (/<span style="font-style: italic;">separator</span>)
                </span>
            </li>
            <li id="checkId-1518315885641">
                <span style="font-size: 14px;">
                    Use
                    /<span style="font-style: italic;">heading</span>
                    to convert a text into a title
                </span>
            </li>
        </ul>
    </li>
</ul>
<p><br /></p>

<h3 id="">Create a task</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        You can convert your to-dos into tasks and assign them to a project. To do this, you need to access the
        <span style="font-weight: bolder;">
            <font class="text-o-color-2">Convert to Task</font>
        </span>
        pop-up from the action menu of the to-do.
    </span>
</p>
<p>
    <span style="font-size: 14px;">
        To-dos are now like 
        <span style="font-weight: bolder;">
            <font class="text-o-color-2">private tasks</font>
        </span>
        and will appear in the Project app under 
        <span>
            <font style="font-weight: bolder;" class="text-o-color-2">My Tasks</font>.
        </span>
        Similarly, private tasks created in the Project app will also appear in your to-dos.
    </span>
</p>
<p><br /></p>

<h3>Who has access to what?</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        To-dos, like private tasks, are only accessible to the users specified as assignees. You can use the
        <span style="font-weight: bolder;">
            <font class="text-o-color-2">assignees</font>
        </span>
        field above to share this to-do with other users.
    </span>
</p>
<p>
    <span style="font-size: 14px;">
        During the conversion of a to-do to a task, you will be able to add or remove assignees to control who has access to the created task. Note that its visibility will also depend on the visibility settings of the project you are assigning it to.
    </span>
</p>
    </template>

</data></odoo>

```

## File: models\mail_activity_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class MailActivityType(models.Model):
    _inherit = "mail.activity.type"

    category = fields.Selection(selection_add=[('reminder', 'Reminder')])

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _, _lt, Command
from odoo.tools import html2plaintext

class Task(models.Model):
    _inherit = 'project.task'

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('name') and not vals.get('project_id') and not vals.get('parent_id'):
                if vals.get('description'):
                    # Generating name from first line of the description
                    text = html2plaintext(vals['description'])
                    name = text.strip().replace('*', '').partition("\n")[0]
                    vals['name'] = (name[:97] + '...') if len(name) > 100 else name
                else:
                    vals['name'] = _('Untitled to-do')
        return super().create(vals_list)

    def _ensure_onboarding_todo(self):
        if not self.env.user.has_group('project_todo.group_onboarding_todo'):
            self._generate_onboarding_todo(self.env.user)
            onboarding_group = self.env.ref('project_todo.group_onboarding_todo').sudo()
            onboarding_group.write({'users': [Command.link(self.env.user.id)]})

    def _generate_onboarding_todo(self, user):
        user.ensure_one()
        body = self.with_context(lang=user.lang or self.env.user.lang).env['ir.qweb']._render(
            'project_todo.todo_user_onboarding',
            {'object': user},
            minimal_qcontext=True,
            raise_if_not_found=False
        )
        if not body:
            return
        title = _lt('Welcome %s!', user.name)
        self.env['project.task'].create([{
            'user_ids': user.ids,
            'description': body,
            'name': title,
        }])

    def action_convert_to_task(self):
        self.ensure_one()
        self.company_id = self.project_id.company_id
        return {
            'view_mode': 'form',
            'res_model': 'project.task',
            'res_id': self.id,
            'type': 'ir.actions.act_window',
        }

```

## File: models\__init__.py

```python
from . import mail_activity_type
from . import project_task

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_project_task_type_user,project.task.type.user,project.model_project_task_type,base.group_user,1,1,1,1
access_task_on_partner,project.task on partners,project.model_project_task,base.group_user,1,1,1,1
access_project_tags_user,project.project_tags_user,project.model_project_tags,base.group_user,1,1,1,1
access_mail_activity_todo_create,mail.activity.todo.create,model_mail_activity_todo_create,base.group_user,1,1,1,0

```

## File: security\project_todo_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="group_onboarding_todo" model="res.groups">
        <field name="name">Onboarding todo already generated for those users</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

<data noupdate="1">

    <record model="ir.rule" id="project.task_visibility_rule">
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record model="ir.rule" id="task_edition_rule_internal">
        <field name="name">Project/Task: employees: Full access to own private task only</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[('project_id', '=', False), ('user_ids', 'in', user.id), ('parent_id', '=', False)]</field>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record model="ir.rule" id="task_visibility_rule_project_user">
        <field name="name">Project/Task: project users: follow required for follower-only projects</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[
            '|',
                '&amp;',
                    ('project_id', '!=', False),
                    '|',
                        ('project_id.privacy_visibility', '!=', 'followers'),
                        ('project_id.message_partner_ids', 'in', [user.partner_id.id]),
                '|',
                    ('message_partner_ids', 'in', [user.partner_id.id]),
                    # to subscribe check access to the record, follower is not enough at creation
                    ('user_ids', 'in', user.id)
        ]</field>
        <field name="perm_read" eval="False"/>
        <field name="groups" eval="[(4,ref('project.group_project_user'))]"/>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M10 41h36a3 3 0 0 1-3 3H10v-3Z" fill="#1AD3BB"/><path d="M9 41h19c0 1.657-1.418 3-3.167 3H9v-3Z" fill="#03AF89"/><path d="M40.706 7.282a4.35 4.35 0 0 1 0 6.193L9.906 44H4v-6.192L34.457 7.282a4.447 4.447 0 0 1 6.249 0Z" fill="#005E7A"/></svg>

```

## File: static\src\components\todo_done_checkmark\todo_done_checkmark.js

```javascript
/** @odoo-module */

import { useState, onRendered, onMounted } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { StateSelectionField, stateSelectionField } from "@web/views/fields/state_selection/state_selection_field";

export class TodoDoneCheckmark extends StateSelectionField {
    setup() {
        super.setup();
        this.stateDone = useState({
            isDone: false, //This state determines the appearance of the done checkmark and should only be actualized when the mouse leaves it (and atfer the form is loaded)
            notReloadState: false, //used to avoid a change of the checkmark when re-rendering the form
        });
        onMounted(() => {
            const fieldValue = this.props.record.data[this.props.name]
            this.notDoneState = fieldValue == '1_done' ? '01_in_progress' : fieldValue;
        });
        onRendered(() => {
            if (!this.stateDone.notReloadState) {
                this.stateDone.isDone = this.props.record.data[this.props.name] == '1_done';
            }
        });
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    actualizeDoneState(ev) {
        this.stateDone.notReloadState = false;
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    freezeDoneState(ev) {
        this.stateDone.notReloadState = true;
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    async onDoneToggled(ev) {
        const value = this.props.record.data[this.props.name] != '1_done' ? '1_done' : this.notDoneState;
        if (['kanban', 'list'].includes(this.props.viewType)) {
            await super.updateRecord(value);
        }
        else {
            await this.props.record.update({
                [this.props.name]: value,
            });
        }
    }
}

TodoDoneCheckmark.template = 'project_todo.TodoDoneCheckmark';

TodoDoneCheckmark.props = {
    ...stateSelectionField.component.props,
    viewType: { type: String },
}

export const todoDoneCheckmark = {
    ...stateSelectionField,
    component: TodoDoneCheckmark,
    extractProps: (fieldInfo, dynamicInfo) => {
        const props = stateSelectionField.extractProps(fieldInfo, dynamicInfo);
        props.viewType = fieldInfo.viewType;
        return props;
    },
}

registry.category("fields").add("todo_done_checkmark", todoDoneCheckmark);

```

## File: static\src\components\todo_done_checkmark\todo_done_checkmark.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoDoneCheckmark">
        <t t-if="!env.isSmall">
            <a title="Mark as done"
               t-on-click.stop="onDoneToggled"
               t-on-mouseleave="actualizeDoneState"
               t-on-mouseover="freezeDoneState"
               t-attf-class="o_todo_done_button fa fa-lg fa-check-circle{{!stateDone.isDone ? '-o' : ' done_button_enabled'}}"/>
        </t>
        <t t-else="">
            <a title="Mark as done"
               t-on-click.stop="onDoneToggled"
               t-attf-class="o_todo_done_button_mobile fa fa-lg fa-check-circle{{!stateDone.isDone ? '-o' : ' done_button_enabled'}}"/>
        </t>
    </t>

</templates>

```

## File: static\src\components\todo_editable_breadcrumb_name\todo_editable_breadcrumb_name.js

```javascript
/** @odoo-module */

import { useState, onRendered, useRef } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { CharField } from "@web/views/fields/char/char_field";
import { useAutoresize } from "@web/core/utils/autoresize";

export class TodoEditableBreadcrumbName extends CharField {
    setup() {
        super.setup();
        this.placeholder = _t("Untitled to-do");;
        this.input = useRef("input");

        this.stateTodo = useState({
            isUntitled: true,
        });

        onRendered(() => {
            this.stateTodo.isUntitled = this._isUntitled(this.props.record.data[this.props.name]);
        });
        useAutoresize(this.input);
    }

    /**
     * Check if the name is empty or is the generic name
     * for untitled todos.
     * @param {string} name
     * @returns {boolean}
     */
    _isUntitled(name) {
        name = name && name.trim();
        return !name || name === this.placeholder.toString();
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    _onFocus(ev) {
        if (this._isUntitled(ev.target.value)) {
            ev.target.value = this.placeholder;
            ev.target.select();
        }
    }

    /**
     * @private
     * @param {InputEvent} ev
     */
    _onInput(ev) {
        const value = ev.target.value;
        this.stateTodo.isUntitled = this._isUntitled(value);
    }
}

TodoEditableBreadcrumbName.template = 'project_todo.TodoEditableBreadcrumbName';

```

## File: static\src\components\todo_editable_breadcrumb_name\todo_editable_breadcrumb_name.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoEditableBreadcrumbName" t-inherit="web.CharField" t-inherit-mode="primary">
        <xpath expr="//input[hasclass('o_input')]" position="attributes">
            <attribute name="class">o_input o_todo_breadcrumb_name_input</attribute>
            <attribute name="t-on-focus">_onFocus</attribute>
            <attribute name="t-att-placeholder">placeholder</attribute>
            <attribute name="t-on-input">_onInput</attribute>
            <attribute name="t-att-class">stateTodo.isUntitled ? 'o-todo-untitled' : ''</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\todo_activity_wizard\todo_activity_wizard_controller.js

```javascript
/** @odoo-module **/

import { onMounted } from "@odoo/owl";
import { FormController } from "@web/views/form/form_controller";

export class TodoActivityWizardController extends FormController {
    setup() {
        super.setup();
        onMounted(() => {
            const firstInput = document.querySelector('div.o_field_widget input');
            firstInput.focus();
        });
    }
}

```

## File: static\src\views\todo_activity_wizard\todo_activity_wizard_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { TodoActivityWizardController } from "./todo_activity_wizard_controller";

export const todoActivityWizardView = {
    ...formView,
    Controller: TodoActivityWizardController,
};

registry.category("views").add("todo_activity_wizard", todoActivityWizardView);

```

## File: static\src\views\todo_conversion_form\todo_conversion_form_controller.js

```javascript
/** @odoo-module **/

import { onMounted } from "@odoo/owl";
import { FormController } from "@web/views/form/form_controller";

export class TodoConversionFormController extends FormController {
    /**
     * Allows to autofocus the first element of the conversion form
     *
     * @override
     * @private
     */
    setup() {
        super.setup();
        onMounted(() => {
            const firstConversionInput = document.querySelector('div.o_todo_conversion_form_view input');
            firstConversionInput.focus();
        });
    }
}

```

## File: static\src\views\todo_conversion_form\todo_conversion_form_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { TodoConversionFormController } from "./todo_conversion_form_controller";

export const todoConversionFormView = {
    ...formView,
    Controller: TodoConversionFormController,
};

registry.category("views").add("todo_conversion_form", todoConversionFormView);

```

## File: static\src\views\todo_form\todo_form_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { FormController } from "@web/views/form/form_controller";
import { TodoEditableBreadcrumbName } from "@project_todo/components/todo_editable_breadcrumb_name/todo_editable_breadcrumb_name";
import { TodoDoneCheckmark } from "@project_todo/components/todo_done_checkmark/todo_done_checkmark";

import { onWillStart } from "@odoo/owl";

/**
 *  The FormController is overridden to be able to manage the edition of the name of a to-do directly
 *  in the breadcrumb as well as the mark as done button next to it.
 */

export class TodoFormController extends FormController {
    static template = "project_todo.TodoFormView";

    setup() {
        super.setup();
        onWillStart(async () => {
            this.projectAccess = await this.user.hasGroup("project.group_project_user");
        });
    }

    get actionMenuItems() {
        const actionToKeep = ["archive", "unarchive", "duplicate", "delete"];
        const menuItems = super.actionMenuItems;
        const filteredActions =
            menuItems.action?.filter((action) => actionToKeep.includes(action.key)) || [];

        if (this.projectAccess) {
            filteredActions.push({
                description: _t("Convert to Task"),
                callback: () => {
                    this.model.action.doAction(
                        "project_todo.project_task_action_convert_todo_to_task",
                        {
                            props: {
                                resId: this.model.root.resId,
                            },
                        }
                    );
                },
            });
        }
        menuItems.action = filteredActions;
        menuItems.print = [];
        return menuItems;
    }
}

Object.assign(TodoFormController.components, {
    TodoEditableBreadcrumbName,
    TodoDoneCheckmark,
});

```

## File: static\src\views\todo_form\todo_form_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoFormView" t-inherit="web.FormView" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_form_view_container')]/Layout" position="inside">
            <t t-set-slot="control-panel-todo-name">
                <t t-set="record" t-value="model.root"/>
                <TodoEditableBreadcrumbName
                    name="'name'"
                    record="record"
                    id="'name'"
                />
            </t>
            <t t-set-slot="todo-mark-as-done-button">
                <t t-set="record" t-value="model.root"/>
                <TodoDoneCheckmark
                    name="'state'"
                    record="record"
                    id="'state'"
                    viewType="'form'"
                />
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\todo_form\todo_form_control_panel.js

```javascript
/** @odoo-module **/

import { ControlPanel } from "@web/search/control_panel/control_panel";

export class TodoFormControlPanel extends ControlPanel {
    static template = "project_todo.TodoFormControlPanel";
}

```

## File: static\src\views\todo_form\todo_form_control_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoFormControlPanel" t-inherit="web.ControlPanel" t-inherit-mode="primary">
        <xpath expr="//div/t/t[@t-slot='control-panel-breadcrumb']" position="replace">
            <t t-slot="control-panel-breadcrumb">
                <t t-call="project_todo.TodoBreadcrumbs"/>
            </t>
        </xpath>

    </t>

    <t t-name="project_todo.TodoBreadcrumbs" t-inherit="web.Breadcrumbs" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_last_breadcrumb_item')]//span" position="replace">
            <t t-slot="control-panel-todo-name" />
            <t t-slot="todo-mark-as-done-button" />
        </xpath>
    </t>

</templates>

```

## File: static\src\views\todo_form\todo_form_renderer.js

```javascript
/* @odoo-module */

import { FormRenderer } from "@web/views/form/form_renderer";

import { TodoFormStatusBarButtons } from "./todo_form_status_bar_button";

export class TodoFormRenderer extends FormRenderer {
    static components = {
        ...FormRenderer.components,
        StatusBarButtons: TodoFormStatusBarButtons,
    };
}

```

## File: static\src\views\todo_form\todo_form_status_bar_button.js

```javascript
/* @odoo-module */
import { StatusBarButtons } from "@web/views/form/status_bar_buttons/status_bar_buttons";

export class TodoFormStatusBarButtons extends StatusBarButtons {
    static template = "project_todo.TodoFormStatusBarButtons";
}

```

## File: static\src\views\todo_form\todo_form_status_bar_buttons.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoFormStatusBarButtons" t-inherit="web.StatusBarButtons">
        <xpath expr="//DropdownItem" position="attributes">
            <attribute name="parentClosingMode">'none'</attribute>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\todo_form\todo_form_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { TodoFormControlPanel } from "./todo_form_control_panel";
import { TodoFormController } from "./todo_form_controller";
import { TodoFormRenderer } from "./todo_form_renderer";

export const todoFormView = {
    ...formView,
    ControlPanel: TodoFormControlPanel,
    Controller: TodoFormController,
    Renderer: TodoFormRenderer,
};

registry.category("views").add("todo_form", todoFormView);

```

## File: static\src\views\todo_list\todo_list_controller.js

```javascript
/** @odoo-module */

import { ListController } from "@web/views/list/list_controller";

export class TodoListController extends ListController {
    get actionMenuItems() {
        this.archiveEnabled = true;
        const actionToKeep = ["export", "archive", "unarchive", "duplicate", "delete"];
        const menuItems = super.actionMenuItems;
        const filteredActions = menuItems.action?.filter(action => actionToKeep.includes(action.key)) || [];
        menuItems.action = filteredActions;
        menuItems.print = [];
        return menuItems;
    }
}

```

## File: static\src\views\todo_list\todo_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { TodoListController } from "./todo_list_controller";

export const todoListView = {
    ...listView,
    Controller: TodoListController,
};

registry.category("views").add("todo_list", todoListView);

```

## File: static\src\web\activity\activity_menu_patch.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ActivityMenu } from "@mail/core/web/activity_menu";
import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";
import { useCommand } from "@web/core/commands/command_hook";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";
import { registry } from "@web/core/registry";

// Add a to-do category for the command palette
registry.category("command_categories").add("to-do", {}, { sequence: 105 });

patch(ActivityMenu.prototype, {
    setup() {
        super.setup(...arguments);
        this.orm = useService("orm");
        this.dialogService = useService("dialog");
        useCommand(
            _t("Add a To-Do"),
            () => {
                document.body.click(); // hack to close command palette
                this.createActivityTodo();
            },
            {
                category: "to-do",
                hotkey: "alt+shift+t",
                global: true,
            }
        );
    },

    async createActivityTodo() {
        const wizard = await this.orm.call("mail.activity.todo.create", "create", [{
            "user_id": this.userId,
        }]);
        this.dialogService.add(FormViewDialog, {
            title: _t("Add a To-Do"),
            resModel: "mail.activity.todo.create",
            resId: wizard,
            preventCreate: true,
            size: "md",
        });
    },
});

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Todo Kanban view -->
    <record id="project_task_view_todo_kanban" model="ir.ui.view">
        <field name="name">project.task.kanban</field>
        <field name="model">project.task</field>
        <field name="priority">800</field>
        <field name="arch" type="xml">
            <kanban default_group_by="personal_stage_type_id"
                    class="o_kanban_small_column"
                    on_create="quick_create"
                    quick_create_view="project_todo.project_task_view_todo_quick_create_form"
                    sample="1"
                    js_class="project_task_kanban"
                    default_order="state, priority desc, date_deadline asc, sequence, id desc">
                <field name="color"/>
                <field name="sequence"/>
                <field name="name"/>
                <field name="active"/>
                <field name="description"/>
                <field name="message_partner_ids"/>
                <field name="activity_ids" />
                <field name="activity_state" />
                <field name="state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="kanban-menu">
                        <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                    </t>
                    <t t-name="kanban-box">
                        <t t-set="todoHasAssignees" t-value="record.user_ids.raw_value.length &gt; 1"/>
                        <div t-attf-class="#{kanban_color(record.color.raw_value)} oe_kanban_global_click_edit oe_semantic_html_override oe_kanban_card">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <div class="d-flex justify-content-start">
                                            <div>
                                                <field name="state" widget="todo_done_checkmark"/>
                                            </div>
                                            <div t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                                <strong class="o_kanban_record_title align-middle">
                                                    <field name="name" widget="name_with_subtask_count"/>
                                                </strong>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body o_todo_kanban_card_body" t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                    <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left"/>
                                    <div class="oe_kanban_bottom_right"  t-att-class="['1_done', '1_canceled'].includes(record.state.raw_value) ? 'opacity-50' : ''">
                                        <div t-attf-class="d-flex #{todoHasAssignees ? 'w-100 align-items-center justify-content-end' : 'align-items-end'}">
                                            <div class="d-flex align-items-center me-2 mt-2">
                                                <div t-if="todoHasAssignees">
                                                    <field name="user_ids" widget="many2many_avatar_user"/>
                                                </div>
                                                <field name="activity_ids" widget="kanban_activity"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- To-do Tree View -->
    <record id="project_task_view_todo_tree" model="ir.ui.view">
        <field name="name">project.task.todo.tree</field>
        <field name="model">project.task</field>
        <field name="arch" type="xml">
            <tree string="To-dos"
                  default_group_by="personal_stage_type_id"
                  editable="bottom"
                  multi_edit="1"
                  open_form_view="True"
                  js_class="todo_list">
                <field name="state" widget="todo_done_checkmark" nolabel="1"/>
                <field name="name"/>
                <field name="user_ids" optional="show" widget="many2many_avatar_user" options="{'no_quick_create': True}"/>
                <field name="tag_ids" optional="show" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                <field name="personal_stage_type_id" string="Stage" optional="hide"/>
                <field name="activity_ids" optional="show" widget="list_activity"/>
            </tree>
        </field>
    </record>

    <!-- Todo Form view -->
    <record id="project_task_view_todo_form" model="ir.ui.view">
        <field name="name">project.task.view.todo.form</field>
        <field name="model">project.task</field>
        <field name="arch" type="xml">
            <form string="To-do"
                  class="o_todo_form_view"
                  js_class="todo_form">
                <field name="name" invisible="1"/>
                <header>
                    <div class="py-1 px-2 border rounded bg-view">
                        <field name="tag_ids"
                               widget="many2many_tags"
                               options="{'color_field': 'color', 'no_create_edit': True}"
                               class="me-2"
                               placeholder="Tags"/>
                    </div>
                    <div class="py-1 px-2 border rounded bg-view">
                        <field name="user_ids"
                               widget="many2many_avatar_user"
                               options="{'no_quick_create': True}"
                               placeholder="Assignees"/>
                    </div>
                    <field name="personal_stage_type_id"
                           domain="[('user_id','=',uid)]"
                           widget="statusbar"
                           options="{'clickable': '1', 'fold_field': 'fold'}"/>
                    <field name="active" invisible="1"/>
                    <field name="state" invisible="1"/>
                </header>
                <sheet class="o_todo_form_sheet_bg">
                    <widget name="web_ribbon" class="todo_archived" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="description" type="html" class="oe_description" default_focus="1" options="{'resizable': false, 'collaborative': true}"/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <!-- To-do Quick create form  view -->
    <record id="project_task_view_todo_quick_create_form" model="ir.ui.view">
        <field name="name">project.task.view.todo.quick.create.todo</field>
        <field name="model">project.task</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form class="o_form_project_tasks">
                <group>
                    <field name="name" string="To-do Title" placeholder="e.g. Send Invitations"/>
                </group>
            </form>
        </field>
    </record>

    <!-- Todo conversion form (used by an action added in TodoFormView (contoller) -->
    <record id="project_task_view_todo_conversion_form" model="ir.ui.view">
        <field name="model">project.task</field>
        <field name="name">project.task.view.todo.conversion.form</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <form string="Convert to Task"
                  js_class="todo_conversion_form">
                <sheet>
                    <group>
                        <!-- company_id field is used in the domain filtering project_id in
                             hr_timesheet. Creating a bridge module just to add that field is
                             overkill so it is added here. -->
                        <field name="company_id" invisible="1"/>
                        <field name="project_id"
                               required="1"
                               placeholder="Select an existing project"
                               default_focus="1"/>
                        <field name="user_ids"
                               class="o_task_user_field"
                               options="{'no_open': True, 'no_quick_create': True}"
                               widget="many2many_avatar_user"/>
                        <field name="tag_ids" widget="many2many_tags"
                               options="{'color_field': 'color', 'no_create_edit': True}"
                               context="{'project_id': project_id}"
                               placeholder="Choose tags from the selected project"/>
                    </group>
                </sheet>
                <footer>
                    <button name="action_convert_to_task" string="Convert to Task" type="object" class="btn-primary"/>
                    <button string="Discard" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <!-- Todo Activity  -->
    <record id="project_task_view_todo_activity" model="ir.ui.view">
        <field name="name">project.task.view.todo.activity</field>
        <field name="model">project.task</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <activity string="To-dos" js_class="project_activity">
                <field name="user_ids"/>
                <templates>
                    <div class="w-100" t-name="activity-box" style="display: inline-grid; grid-template-columns: auto max-content;">
                        <div>
                            <span t-att-title="record.name.value">
                                <field name="name" display="full" class="w-100 o_text_block align-middle"/>
                            </span>
                        </div>
                        <field name="user_ids" widget="many2many_avatar_user"/>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <!-- Todo Search  -->
    <record id="project_task_view_todo_search" model="ir.ui.view">
        <field name="name">project.task.view.todo.search</field>
        <field name="model">project.task</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <search string="Todos">
                <field name="name"/>
                <field name="tag_ids"/>
                <field name="user_ids"/>
                <field name="personal_stage_type_ids" string="Stage"/>
                <filter name="open_tasks" string="Open" domain="[('state', 'in', ['01_in_progress', '02_changes_requested', '03_approved', '04_waiting_normal'])]"/>
                <filter name="closed_tasks" string="Closed" domain="[('state', 'in', ['1_done','1_canceled'])]"/>
                <filter string="Closed On" name="closed_on" domain="[('state', 'in', ['1_done','1_canceled'])]" date="date_last_stage_update" invisible="1"/>
                <separator/>
                <filter name="active_false" string="Archived" domain="[('active', '=', False)]"/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                <group expand="0" string="Group By">
                    <filter string="Tags" name="tags" help="By assigned tags" context="{'group_by':'tag_ids'}"/>
                    <filter string="Assignees" name="user_ids" context="{'group_by': 'user_ids'}"/>
                    <filter string="Stage" name="stage" help="By personal stages" context="{'group_by':'personal_stage_type_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- Todo main action + view links-->
    <record id="project_task_action_todo" model="ir.actions.act_window">
        <field name="name">To-dos</field>
        <field name="res_model">project.task</field>
        <field name="domain">[('user_ids', 'in', [uid]), ('project_id', '=', False), ('parent_id', '=', False)]</field>
        <field name="view_mode">kanban,form,tree,activity</field>
        <field name="search_view_id" ref="project_task_view_todo_search"/>
        <field name="context">{'search_default_open_tasks': 1, 'tree_view_ref': 'project_todo.project_task_view_todo_tree', 'default_project_id': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No to-do found. Let's create one!
            </p><p>
                Keep your work organized by using memos and to-do lists.
                Your to-do items are private by default, but you can choose to share them with others by adding them as assignees.
            </p>
        </field>
    </record>

    <record model="ir.actions.act_window.view" id="project_task_action_todo_kanban_view">
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="project_task_view_todo_kanban"/>
        <field name="act_window_id" ref="project_task_action_todo"/>
    </record>

    <record model="ir.actions.act_window.view" id="project_task_action_todo_form_view">
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_task_view_todo_form"/>
        <field name="act_window_id" ref="project_task_action_todo"/>
    </record>

    <record model="ir.actions.act_window.view" id="project_task_action_todo_tree_view">
        <field name="view_mode">tree</field>
        <field name="view_id" ref="project_task_view_todo_tree"/>
        <field name="act_window_id" ref="project_task_action_todo"/>
    </record>

    <record model="ir.actions.act_window.view" id="project_task_action_todo_activity_view">
        <field name="view_mode">activity</field>
        <field name="view_id" ref="project_task_view_todo_activity"/>
        <field name="act_window_id" ref="project_task_action_todo"/>
    </record>

    <!-- Todo pre-loading action -->
    <record id="project_task_preload_action_todo" model="ir.actions.server">
        <field name="name">menu load To-dos</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="state">code</field>
        <field name="code">
            model._ensure_onboarding_todo(); action = env["ir.actions.actions"]._for_xml_id("project_todo.project_task_action_todo")
        </field>
    </record>

    <!-- Conversion actions-->
    <record id="project_task_action_convert_todo_to_task" model="ir.actions.act_window">
        <field name="name">Convert to Task</field>
        <field name="res_model">project.task</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

    <record model="ir.actions.act_window.view" id="project_task_action_convert_todo_to_task_form_view">
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_task_view_todo_conversion_form"/>
        <field name="act_window_id" ref="project_task_action_convert_todo_to_task"/>
    </record>

</odoo>

```

## File: views\project_todo_menus.xml

```xml
<?xml version="1.0"?>
<odoo>
    <menuitem
        id="menu_todo_todos"
        name="To-do"
        action="project_todo.project_task_preload_action_todo"
        web_icon="project_todo,static/description/icon.png">
    </menuitem>
</odoo>

```

## File: wizard\mail_activity_todo_create.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _

class MailActivityTodoCreate(models.TransientModel):
    _name = 'mail.activity.todo.create'
    _description = 'Create activity and todo at the same time'

    summary = fields.Char()
    date_deadline = fields.Date('Due Date', index=True, required=True, default=fields.Date.context_today)
    user_id = fields.Many2one('res.users', 'Assigned to', default=lambda self: self.env.user, required=True, readonly=True)
    note = fields.Html(sanitize_style=True)

    def create_todo_activity(self):
        todo = self.env['project.task'].create({
            'name': self.summary,
            'description': self.note,
            'date_deadline': self.date_deadline,
            'user_ids': self.user_id.ids,
        })
        self.env['mail.activity'].create({
            'res_model_id': self.env['ir.model']._get('project.task').id,
            'res_id': todo.id,
            'summary': self.summary,
            'user_id': self.user_id.id,
            'date_deadline': self.date_deadline,
            'activity_type_id': self.env['mail.activity']._default_activity_type_for_model('project.task').id,
        })

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _("Your to-do has been successfully added to your pipeline."),
            },
        }

```

## File: wizard\mail_activity_todo_create.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_activity_todo_create_popup" model="ir.ui.view">
        <field name="name">mail.activity.todo.create.popup</field>
        <field name="model">mail.activity.todo.create</field>
        <field name="arch" type="xml">
            <form js_class="todo_activity_wizard">
                <group>
                    <field name="summary" placeholder="Reminder to..." required="1"/>
                    <field name="date_deadline"/>
                    <field name="user_id" widget="many2one_avatar_user" options="{'no_open': 1}"/>
                </group>
                <field name="note" class="oe-bordered-editor" placeholder="Add details about your to-do..."/>
                <footer>
                    <button class="btn btn-primary" type="object" name="create_todo_activity" close="1">Add To-Do</button>
                    <button class="btn btn-secondary" special="cancel" close="1">Discard</button>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_activity_todo_create

```

