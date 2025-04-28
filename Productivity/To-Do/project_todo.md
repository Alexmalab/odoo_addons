# Odoo Module: project_todo

Category: Productivity/To-Do

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

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
        'data/todo_template.xml',
        'views/project_task_views.xml',
        'views/project_todo_menus.xml',
        'wizard/mail_activity_todo_create.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'project_todo/static/src/components/**/*',
            'project_todo/static/src/scss/todo.scss',
            'project_todo/static/src/views/**/*',
            'project_todo/static/src/web/**/*',
        ],
        'web.assets_tests': [
            'project_todo/static/tests/tours/**/*',
        ],
        'web.assets_unit_tests': [
            'project_todo/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\todo_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <template id="todo_user_onboarding">
<h1>
    <span style="font-size: 36px; color: #374151">Hey <t t-out="object.name"/> &amp;#128075; <br />
    Welcome to the To-do app! </span>
</h1>
<p>
    <span style="font-size: 14px;">
        Use it to manage your work, take notes on the go, and create tasks based on them.
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
    <li id="checkId-948702291173">
        <span style="font-size: 14px;">Check this box to indicate it's done</span>
    </li>
    <li id="checkId-140908257281">
        <span style="font-size: 14px;">Click anywhere, and just start typing</span>
    </li>
    <li id="checkId-441216037148">
        <span style="font-size: 14px;">Press Ctrl+Z/⌘+Z to undo any change</span>
    </li>
    <li id="checkId-980463482772">
        <span style="font-size: 14px;">
            Select text to
            <font style="background-color: #017E84; color: white">Highlight</font>,
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
                <font style="color: #017E84">typing</font>
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

<h3>Who has access to what?</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        By default, to-dos are only visible to you. You can share them with other users by adding them as
        <span>
            <font style="font-weight: bolder;" class="text-o-color-2">assignees</font>.
        </span>
    </span>
    <br/>
    <img class="img-fluid d-none d-sm-block" src="/project_todo/static/img/todo_access.png" alt="todo-access"/>
</p>
<p><br /></p>

<h3>Organize your to-dos however you want</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        Customize the stages from the
        <span style="font-weight: bolder;">
            <font class="text-o-color-2">Kanban view</font>
        </span>
        to reflect your preferred workflow.
    </span>
</p>
<p><br /></p>

<h3 class="fw-bolder">Manage your to-dos and assigned tasks from a single place</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        Access your personal pipeline with your to-dos and assigned tasks by going to the Project app and clicking
        <span>
            <font style="font-weight: bolder;" class="text-o-color-2">My Tasks</font>.
        </span>
    </span>
    <br/>
    <br/>
    <span style="font-size: 14px;">
        There, your to-dos are listed as
        <span style="font-weight: bolder;">
            <font class="text-o-color-2">private tasks.</font>
        </span>
        Any task you create privately will also be included in your to-dos. Essentially, they are interchangeable.
    </span>
</p>
<p><br /></p>

<h3>Convert to-dos into tasks</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        If you want to assign your to-do to a specific project, open the ⚙️ menu and click
        <span>
            <font style="font-weight: bolder;" class="text-o-color-2">Convert to Task</font>.
        </span>
        This action will make it visible to other users.
    </span>
    <br/>
    <img class="img-fluid d-none d-sm-block" src="/project_todo/static/img/convert_todo.png" alt="convert-todo"/>
</p>
<p><br /></p>

<h3>Create to-dos from anywhere</h3>
<hr />
<p>
    <span style="font-size: 14px;">
        Wherever you are, use the magic keyboard shortcut to add yourself a reminder &amp;#128161;
    </span>
    <br/>
    <ul>
        <li>
            <span style="font-weight: bolder;">
                <font class="text-o-color-2">Alt + Shift + T</font>
            </span>
            (Windows/Linux)
        </li>
        <li>
            <span style="font-weight: bolder;">
                <font class="text-o-color-2">Ctrl + Shift + T</font>
            </span>
            (MacOs)
        </li>
    </ul>
</p>
    </template>

</data></odoo>

```

## File: models\project_task.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, Command
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
                    vals['name'] = self.env._('Untitled to-do')
        return super().create(vals_list)

    def _ensure_onboarding_todo(self):
        if not self.env.user.has_group('project_todo.group_onboarding_todo'):
            self._generate_onboarding_todo(self.env.user)
            onboarding_group = self.env.ref('project_todo.group_onboarding_todo').sudo()
            onboarding_group.write({'users': [Command.link(self.env.user.id)]})

    def _generate_onboarding_todo(self, user):
        user.ensure_one()
        self_lang = self.with_context(lang=user.lang or self.env.user.lang)
        body = self_lang.env['ir.qweb']._render(
            'project_todo.todo_user_onboarding',
            {'object': user},
            minimal_qcontext=True,
            raise_if_not_found=False
        )
        if not body:
            return
        title = self_lang.env._('Welcome %s!', user.name)
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

    @api.model
    def get_todo_views_id(self):
        """ Returns the ids of the main views used in the To-Do app.

        :return: a list of views id and views type
                 e.g. [(kanban_view_id, "kanban"), (list_view_id, "list"), ...]
        :rtype: list(tuple())
        """
        return [
            (self.env['ir.model.data']._xmlid_to_res_id("project_todo.project_task_view_todo_kanban"), "kanban"),
            (self.env['ir.model.data']._xmlid_to_res_id("project_todo.project_task_view_todo_tree"), "list"),
            (self.env['ir.model.data']._xmlid_to_res_id("project_todo.project_task_view_todo_form"), "form"),
            (self.env['ir.model.data']._xmlid_to_res_id("project_todo.project_task_view_todo_activity"), "activity"),
        ]

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _, api, models, modules


class Users(models.Model):
    _inherit = 'res.users'

    @api.model
    def _get_activity_groups(self):
        """ Split To-do and Project activities in systray by removing
            the single project.task activity represented and doing a
            new query to split them between private/non-private tasks.
        """
        activity_groups = super()._get_activity_groups()
        # 1. removing project.task activity group
        to_remove = next((g for g in activity_groups if g.get('model') == 'project.task'), None)
        if to_remove:
            activity_groups.remove(to_remove)

        # 2. creating groups for todo and task seperately
        query = """SELECT BOOL(t.project_id) as is_task, count(*), act.res_model, act.res_id,
                       CASE
                           WHEN CURRENT_DATE - act.date_deadline::date = 0 THEN 'today'
                           WHEN CURRENT_DATE - act.date_deadline::date > 0 THEN 'overdue'
                           WHEN CURRENT_DATE - act.date_deadline::date < 0 THEN 'planned'
                        END AS states
                     FROM mail_activity AS act
                     JOIN project_task AS t ON act.res_id = t.id
                    WHERE act.res_model = 'project.task' AND act.user_id = %(user_id)s AND act.active in (TRUE, %(active)s)
                 GROUP BY is_task, states, act.res_model, act.res_id
                """
        self.env.cr.execute(query, {
            'user_id': self.env.uid,
            'active': self._context.get('active_test', True),
        })
        activity_data = self.env.cr.dictfetchall()
        view_type = self.env['project.task']._systray_view

        user_activities = {}
        for activity in activity_data:
            is_task = activity['is_task']
            if is_task not in user_activities:
                if not is_task:
                    module = 'project_todo'
                    name = _('To-Do')
                else:
                    module = 'project'
                    name = _('Task')
                icon = modules.module.get_module_icon(module)
                user_activities[is_task] = {
                    'id': self.env['ir.model']._get('project.task').id,
                    'name': name,
                    'is_todo': not is_task,
                    'model': 'project.task',
                    'type': 'activity',
                    'icon': icon,
                    'total_count': 0, 'today_count': 0, 'overdue_count': 0, 'planned_count': 0,
                    'res_ids': set(),
                    'view_type': view_type,
                }
            user_activities[is_task]['res_ids'].add(activity['res_id'])
            user_activities[is_task][f"{activity['states']}_count"] += activity['count']
            if activity['states'] in ('today', 'overdue'):
                user_activities[is_task]['total_count'] += activity['count']

        for group in user_activities.values():
            group.update({
                'domain': json.dumps([['activity_ids.res_id', 'in', list(group['res_ids'])]])
            })
        activity_groups.extend(list(user_activities.values()))

        return activity_groups

```

## File: models\__init__.py

```python
from . import project_task
from . import res_users

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

    <record model="ir.rule" id="task_edition_rule_internal">
        <field name="name">Project/Task: employees: Full access to own private task only</field>
        <field name="model_id" ref="project.model_project_task"/>
        <field name="domain_force">[('project_id', '=', False), ('user_ids', 'in', user.id), ('parent_id', '=', False)]</field>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

</data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M10 41h36a3 3 0 0 1-3 3H10v-3Z" fill="#1AD3BB"/><path d="M9 41h19c0 1.657-1.418 3-3.167 3H9v-3Z" fill="#03AF89"/><path d="M40.706 7.282a4.35 4.35 0 0 1 0 6.193L9.906 44H4v-6.192L34.457 7.282a4.447 4.447 0 0 1 6.249 0Z" fill="#005E7A"/></svg>

```

## File: static\src\components\todo_chatter_panel\todo_chatter_panel.js

```javascript
import { Chatter } from "@mail/chatter/web_portal/chatter";

import { Component, useState, useRef } from "@odoo/owl";

import { registry } from "@web/core/registry";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { useBus } from "@web/core/utils/hooks";

export class TodoChatterPanel extends Component {
    static template = "project_todo.TodoChatterPanel";
    static components = { Chatter };
    static props = {
        ...standardWidgetProps,
    };

    setup() {
        this.state = useState({
            displayChatter: this.env.isSmall,
        });
        this.rootRef = useRef("root");
        useBus(this.env.bus, "TODO:TOGGLE_CHATTER", this.toggleChatter);
    }

    toggleChatter(ev) {
        this.state.displayChatter = ev.detail.displayChatter;
        this.rootRef.el?.parentElement?.classList.toggle('d-none', !this.state.displayChatter);
    }
}

export const todoChatterPanel = {
    component: TodoChatterPanel,
    additionalClasses: ["o_todo_chatter", "d-none", "position-relative", "p-0", "overflow-y-auto"],
};

registry.category("view_widgets").add("todo_chatter_panel", todoChatterPanel);

```

## File: static\src\components\todo_chatter_panel\todo_chatter_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<templates xml:space="preserve">

    <div t-name="project_todo.TodoChatterPanel" t-ref="root">
        <chatter t-if="state.displayChatter" class="o_FormRenderer_chatterContainer">
            <Chatter
                threadModel="'project.task'"
                threadId="props.record.resId"
                webRecord="props.record"
                hasParentReloadOnAttachmentsChanged="true"
                hasParentReloadOnFollowersUpdate="true"
                hasParentReloadOnMessagePosted="true"
                saveRecord="props.record.save.bind(props.record, { reload: false })"
            />
        </chatter>
    </div>

</templates>

```

## File: static\src\components\todo_done_checkmark\todo_done_checkmark.js

```javascript
/** @odoo-module */

import { useState, onRendered, onMounted } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { StateSelectionField, stateSelectionField } from "@web/views/fields/state_selection/state_selection_field";

export class TodoDoneCheckmark extends StateSelectionField {
    static template = "project_todo.TodoDoneCheckmark";
    static props = {
        ...stateSelectionField.component.props,
        viewType: { type: String },
    };
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
            <a t-att-title="stateDone.isDone ? 'Mark as to-do' : 'Mark as done'"
               t-on-click.stop="onDoneToggled"
               t-on-mouseleave="actualizeDoneState"
               t-on-mouseover="freezeDoneState"
               t-attf-class="o_todo_done_button fa fa-lg fa-check-circle{{!stateDone.isDone ? '-o' : ' done_button_enabled'}}"/>
        </t>
        <t t-else="">
            <a t-att-title="stateDone.isDone ? 'Mark as to-do' : 'Mark as done'"
               t-on-click.stop="onDoneToggled"
               t-attf-class="o_todo_done_button_mobile fa fa-lg fa-check-circle{{!stateDone.isDone ? '-o' : ' done_button_enabled'}}"/>
        </t>
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
import { onWillStart } from "@odoo/owl";

import { _t } from "@web/core/l10n/translation";
import { user } from "@web/core/user";
import { FormControllerWithHTMLExpander } from "@resource/views/form_with_html_expander/form_controller_with_html_expander";

/**
 *  The FormController is overridden to be able to manage the edition of the name of a to-do directly
 *  in the breadcrumb as well as the mark as done button next to it.
 */

export class TodoFormController extends FormControllerWithHTMLExpander {
    setup() {
        super.setup();
        onWillStart(async () => {
            this.projectAccess = await user.hasGroup("project.group_project_user");
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

```

## File: static\src\views\todo_form\todo_form_control_panel.js

```javascript
import { onMounted, useEffect } from "@odoo/owl";
import { browser } from "@web/core/browser/browser";
import { router } from "@web/core/browser/router";
import { ControlPanel } from "@web/search/control_panel/control_panel";

export class TodoFormControlPanel extends ControlPanel {
    static template = "project_todo.TodoFormControlPanel";

    setup() {
        super.setup();
        useEffect(
            (isSmall) => {
                if (isSmall && !this.state.displayChatter) {
                    this.toggleChatter();
                }
            },
            () => [this.env.isSmall]
        );
        onMounted(() => {
            // We check if we have come from activity view using router action stack and toggle chatter
            const isFromActivityView =
                router.current.actionStack?.[router.current.actionStack?.length - 1]?.view_type ===
                "activity";
            if (
                !this.env.isSmall &&
                !this.state.displayChatter &&
                (isFromActivityView || JSON.parse(browser.localStorage.getItem("isChatterOpened")))
            ) {
                this.toggleChatter();
            }
        });
    }

    toggleChatter(ev) {
        this.state.displayChatter = !this.state.displayChatter;
        if (ev) {
            browser.localStorage.setItem("isChatterOpened", this.state.displayChatter);
        }
        this.env.bus.trigger("TODO:TOGGLE_CHATTER", { displayChatter: this.state.displayChatter });
    }
}

```

## File: static\src\views\todo_form\todo_form_control_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="project_todo.TodoFormControlPanel" t-inherit="web.ControlPanel" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_cp_pager')]" position="before">
            <a
                t-if="!env.isSmall"
                role="button"
                title="Toggle chatter"
                data-hotkey="d"
                t-attf-class="btn btn-light btn-chatter todo_toggle_chatter #{state.displayChatter ? 'active' : '' }"
                t-on-click="toggleChatter"
            >
                <i class="fa fa-comments"/>
            </a>
        </xpath>
    </t>

</templates>

```

## File: static\src\views\todo_form\todo_form_renderer.js

```javascript
import { FormRendererWithHtmlExpander } from "@resource/views/form_with_html_expander/form_renderer_with_html_expander";
import { useBus } from "@web/core/utils/hooks";

export class TodoFormRenderer extends FormRendererWithHtmlExpander {
    setup() {
        super.setup();
        useBus(this.env.bus, "TODO:TOGGLE_CHATTER", this.toggleChatter);
        this.sizeToExpandHTMLField = 1;
    }

    toggleChatter(ev) {
        this.sizeToExpandHTMLField = ev.detail.displayChatter ? 6 : 1;
    }

    _canExpandHTMLField(size) {
        return size >= this.sizeToExpandHTMLField;
    }
}

```

## File: static\src\views\todo_form\todo_form_view.js

```javascript
import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { TodoFormController } from "./todo_form_controller";
import { TodoFormControlPanel } from "./todo_form_control_panel";
import { TodoFormRenderer } from "./todo_form_renderer";

export const todoFormView = {
    ...formView,
    Controller: TodoFormController,
    ControlPanel: TodoFormControlPanel,
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
import { TaskListRenderer } from "@project/components/task_list_renderer";

export const todoListView = {
    ...listView,
    Controller: TodoListController,
    Renderer: TaskListRenderer,
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

    availableViews(group) {
        if (group.is_todo) {
            return this.todoViews;
        }
        return super.availableViews(group);
    },

    async loadTodoViews() {
        this.todoViews = await this.orm.call(
            "project.task",
            "get_todo_views_id",
            [],
        );
    },

    async onClickAction(action, group) {
        if (group.is_todo) {
            await this.loadTodoViews();
        }
        return super.onClickAction(...arguments);
    },

    async openActivityGroup(group, filter = "all") {
        if (group.is_todo) {
            await this.loadTodoViews();
        }
        return super.openActivityGroup(...arguments);
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
            <kanban highlight_color="color"
                    default_group_by="personal_stage_type_id"
                    class="o_kanban_small_column"
                    on_create="quick_create"
                    quick_create_view="project_todo.project_task_view_todo_quick_create_form"
                    sample="1"
                    js_class="project_task_kanban"
                    default_order="state, priority desc, date_deadline asc, sequence, id desc">
                <field name="color"/>
                <field name="sequence"/>
                <field name="active"/>
                <field name="state"/>
                <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                <templates>
                    <t t-name="menu">
                        <a t-if="widget.editable" role="menuitem" type="set_cover" class="dropdown-item" data-field="displayed_image_id">Set Cover Image</a>
                        <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                        <field name="color" widget="kanban_color_picker"/>
                    </t>
                    <t t-name="card">
                        <t t-set="todoHasAssignees" t-value="record.user_ids.raw_value.length &gt; 1"/>
                        <div t-att-class="{'opacity-50': ['1_done', '1_canceled'].includes(record.state.raw_value)}">
                            <field name="name" class="fw-bolder fs-5" widget="name_with_subtask_count"/>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                            <field t-if="record.displayed_image_id.value" name="displayed_image_id" widget="attachment_image"/>
                        </div>
                        <div class="d-flex pt-2">
                            <div class="d-flex" t-att-class="{'opacity-50': ['1_done', '1_canceled'].includes(record.state.raw_value)}">
                                <field name="priority" class="me-2" widget="priority"/>
                                <field name="activity_ids" widget="kanban_activity"/>
                            </div>
                            <div class="d-flex ms-auto">
                                <div t-att-class="{'opacity-50': ['1_done', '1_canceled'].includes(record.state.raw_value), 'o_todo_hide_avatar': !todoHasAssignees}">
                                    <field name="user_ids" widget="many2many_avatar_user" readonly="True"/>
                                </div>
                                <field name="state" widget="todo_done_checkmark"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- To-do List View -->
    <record id="project_task_view_todo_tree" model="ir.ui.view">
        <field name="name">project.task.todo.list</field>
        <field name="model">project.task</field>
        <field name="arch" type="xml">
            <list string="To-dos"
                  default_group_by="personal_stage_type_id"
                  editable="bottom"
                  multi_edit="1"
                  open_form_view="True"
                  js_class="todo_list">
                <field name="priority" widget="priority" nolabel="1"/>
                <field name="state" widget="todo_done_checkmark" nolabel="1"/>
                <field name="name"/>
                <field name="user_ids" optional="show" required="1" widget="many2many_avatar_user" options="{'no_quick_create': True}"/>
                <field name="tag_ids" optional="show" widget="many2many_tags" options="{'color_field': 'color', 'no_create_edit': True}"/>
                <field name="personal_stage_type_id" string="Stage" optional="hide"/>
                <field name="activity_ids" optional="show" widget="list_activity"/>
            </list>
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
                <header>
                    <field name="personal_stage_type_id"
                           domain="[('user_id','=',uid)]"
                           widget="statusbar"
                           options="{'clickable': '1', 'fold_field': 'fold'}"/>
                    <field name="active" invisible="1"/>
                </header>
                <sheet class="o_todo_form_sheet_bg">
                    <widget name="web_ribbon" class="todo_archived" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title pe-0">
                        <h1 class="d-flex justify-content-between align-items-center">
                            <div class="d-flex w-100">
                                <field name="priority" widget="priority_switch" class="me-3"/>
                                <field name="name" options="{'line_breaks': False}" widget="text" class="o_task_name text-truncate w-md-75 w-100 pe-2" placeholder="To-do..."/>
                            </div>
                            <div class="d-flex justify-content-end o_state_container">
                                <field name="state" widget="todo_done_checkmark" class="o_task_state_widget"/>
                            </div>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="user_ids"
                                widget="many2many_avatar_user"
                                required="1"
                                options="{'no_quick_create': True}"
                                placeholder="Assignees"/>
                        </group>
                        <group>
                            <field name="tag_ids"
                                widget="many2many_tags"
                                options="{'color_field': 'color', 'no_create_edit': True}"
                                class="me-2"/>
                        </group>
                    </group>
                    <field name="description" type="html" class="oe_description" default_focus="1" options="{'resizable': false, 'collaborative': true}"
                        placeholder="Type Here..."/>
                </sheet>
                <widget name="todo_chatter_panel"/>
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
            <activity string="To-dos">
                <field name="user_ids"/>
                <templates>
                    <div class="w-100" t-name="activity-box" style="display: inline-grid; grid-template-columns: auto max-content;">
                        <div>
                            <span t-att-title="record.name.value">
                                <field name="name" display="full" class="w-100 o_text_block align-middle"/>
                            </span>
                        </div>
                        <field name="user_ids" widget="many2many_avatar_user" readonly="True"/>
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
                <filter string="Starred" name="starred_tasks" domain="[('priority', '=', '1')]"/>
                <separator/>
                <filter name="open_tasks" string="Open" domain="[('is_closed', '=', False)]"/>
                <filter name="closed_tasks" string="Closed" domain="[('is_closed', '=', True)]"/>
                <filter string="Closed On" name="closed_on" domain="[('is_closed', '=', True)]" date="date_last_stage_update"/>
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
        <field name="view_mode">kanban,form,list,activity</field>
        <field name="search_view_id" ref="project_task_view_todo_search"/>
        <field name="context">{'search_default_open_tasks': 1, 'list_view_ref': 'project_todo.project_task_view_todo_tree', 'default_project_id': False}</field>
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
        <field name="view_mode">list</field>
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
        <field name="path">to-do</field>
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
        <field name="context">{'dialog_size': 'medium'}</field>
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

