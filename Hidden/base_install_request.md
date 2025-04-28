# Odoo Module: base_install_request

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

from odoo import api, tools, SUPERUSER_ID


def _auto_install_apps(cr, registry):
    if not tools.config.get('default_productivity_apps', False):
        return
    env = api.Environment(cr, SUPERUSER_ID, {})
    env['ir.module.module'].sudo().search([
        ('name', 'in', [
            # Community
            'hr', 'mass_mailing', 'project', 'survey',
            # Enterprise
            'appointment', 'knowledge', 'planning', 'sign',
        ]),
        ('state', '=', 'uninstalled')
    ]).button_install()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Base - Module Install Request',
    'category': 'Hidden',
    'depends': ['mail'],
    'description': """
Allow internal users requesting a module installation
=====================================================
    """,
    'auto_install': True,
    'data':[
        'security/ir.model.access.csv',
        'wizard/base_module_install_request_views.xml',
        'data/mail_template_data.xml',
        'data/mail_templates_module_install.xml',
        'views/ir_module_module_views.xml',
    ],
    'license': 'LGPL-3',
    'post_init_hook': '_auto_install_apps'
}

```

## File: data\mail_templates_module_install.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="base_install_request.base_module_install_review_description">
        <p t-if="any(app.application for app in apps)">The following apps will be installed:</p>
        <div class="container-fluid">
            <ul class="list-unstyled row">
                <t t-foreach="apps" t-as="app">
                    <li class="mt8 col-lg-6" t-if="app.application">
                        <div>
                            <img width="24px" height="24px" class="img-fluid" t-att-src="app.icon"/>
                            <t t-esc="app.shortdesc"/>
                        </div>
                    </li>
                </t>
            </ul>
        </div>
    </template>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_base_install_request" model="mail.template">
            <field name="name">Mail: Install Request</field>
            <field name="model_id" ref="base_install_request.model_base_module_install_request"/>
            <field name="subject">Module Activation Request for "{{ object.module_id.shortdesc }}"</field>
            <field name="email_from">{{ object.user_id.email_formatted or user.email_formatted }}</field>
            <field name="partner_to" >{{ ctx['partner'].id}}</field>
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Hello,
        <br/><br/>
        <span style="font-weight: bold;" t-out="object.user_id.name"/> has requested to activate the <span style="font-weight: bold;" t-out="object.module_id.shortdesc"/> module. This module is included in your subscription. It has <span style="color: #875A7B; font-weight: bold;">no extra cost</span>, but an administrator role is required to activate it.
        <br/><br/>
        <blockquote>
            <t t-out="object.body_html"/>
        </blockquote>
        <br/><br/>
        <a style="background-color:#875A7B; padding:8px 16px 8px 16px; text-decoration:none; color:#fff; border-radius:5px" t-attf-href="/web?#action=base_install_request.action_base_module_install_review&amp;active_id={{ object.module_id.id }}&amp;menu_id={{ ctx['menu_id'] }}">Review Request</a>
        <br/><br/>
        Thanks,
        <t t-if="not is_html_empty(object.user_id.signature)">
            <br/><br/>
            <t t-out="object.user_id.signature or ''">--<br/>Mitchell Admin</t>
        </t>
        <br/><br/>
    </p>
</div>
            </field>
            <field name="lang">{{ ctx['partner'].lang or user.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: models\ir_module_module.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class IrModuleModule(models.Model):
    _inherit = 'ir.module.module'

    def action_open_install_request(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'name': _('Activation Request of "%s"', self.shortdesc),
            'view_mode': 'form',
            'res_model': 'base.module.install.request',
            'context': {'default_module_id': self.id},
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_module_module

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_module_install_request,access_base_module_install_request,model_base_module_install_request,base.group_user,1,1,1,0
access_base_module_install_review,access_base_module_install_review,model_base_module_install_review,base.group_system,1,1,1,0
base.access_ir_module_category_group_user,ir_module_category group_user,base.model_ir_module_category,base.group_user,1,0,0,0
access_ir_module_module_group_user,ir_module_module group_user,base.model_ir_module_module,base.group_user,1,0,0,0
```

## File: views\ir_module_module_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_module_module_view_kanban" model="ir.ui.view">
        <field name="name">ir.module.module.view.kanban.inherit.mail</field>
        <field name="model">ir.module.module</field>
        <field name="inherit_id" ref="base.module_view_kanban"/>
        <field name="arch" type="xml">
            <button name="button_immediate_install" position="after">
                <button type="object" class="btn btn-primary btn-sm" name="action_open_install_request" states="uninstalled" t-if="!record.to_buy.raw_value" groups="!base.group_system">Request Access</button>
            </button>
        </field>
    </record>

    <record id="base.menu_management" model="ir.ui.menu">
        <field name="groups_id" eval="[(5, 0, 0)]"/>
    </record>
</odoo>

```

## File: wizard\base_module_install_request.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class BaseModuleInstallRequest(models.TransientModel):
    _name = "base.module.install.request"
    _description = "Module Activation Request"
    _rec_name = "module_id"

    module_id = fields.Many2one(
        'ir.module.module', string="Module", required=True,
        domain=[('state', '=', "uninstalled")],
        ondelete='cascade', readonly=True,
    )
    user_id = fields.Many2one('res.users', default=lambda self: self.env.user, required=True)
    user_ids = fields.Many2many('res.users', string="Send to:", compute='_compute_user_ids')
    body_html = fields.Html('Body')

    @api.depends('module_id')
    def _compute_user_ids(self):
        users = self.env.ref('base.group_system').users
        self.user_ids = [(6, 0, users.ids)]

    def action_send_request(self):
        mail_template = self.env.ref('base_install_request.mail_template_base_install_request')
        menu_id = self.env.ref('base.menu_apps').id
        for user in self.user_ids:
            render_ctx = dict(self.env.context, partner=user.partner_id, menu_id=menu_id)
            mail_template.with_context(render_ctx).send_mail(
                self.id,
                force_send=True,
                email_layout_xmlid='mail.mail_notification_light')
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _('Your request has been successfully sent'),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }


class BaseModuleInstallReview(models.TransientModel):
    _name = "base.module.install.review"
    _description = "Module Activation Review"
    _rec_name = "module_id"

    module_id = fields.Many2one(
        'ir.module.module', string="Module", required=True,
        domain=[('state', '=', "uninstalled")],
        ondelete='cascade', readonly=True,
    )
    module_ids = fields.Many2many(
        'ir.module.module', string="Depending Apps", compute='_compute_modules_description')
    modules_description = fields.Html(compute='_compute_modules_description')

    @api.depends('module_id')
    def _compute_modules_description(self):
        for wizard in self:
            apps = wizard._get_depending_apps(wizard.module_id)
            wizard.module_ids = [(6, 0, apps.ids)]
            wizard.modules_description = self.env["ir.qweb"]._render(
                "base_install_request.base_module_install_review_description", {'apps': apps})

    @api.model
    def _get_depending_apps(self, module):
        if not module:
            raise UserError(_('No module selected.'))
        if module.state == "installed":
            raise UserError(_('The module is already installed.'))
        deps = module.upstream_dependencies()
        apps = module | deps.filtered(lambda d: d.application)
        for dep in deps:
            apps |= dep.upstream_dependencies()
        return apps

    def action_install_module(self):
        self.ensure_one()
        self.module_id.button_immediate_install()
        return {
            'type': 'ir.actions.client',
            'tag': 'home',
            'params': {'wait': True},
        }

```

## File: wizard\base_module_install_request_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="base_module_install_request_view_form" model="ir.ui.view">
        <field name="name">base.module.install.request.view.form</field>
        <field name="model">base.module.install.request</field>
        <field name="arch" type="xml">
            <form>
                <div class="alert alert-warning oe_button_box" role="alert">
                    <p class="mt-3">
                        This app is included in your subscription. It's free to activate, but only an administrator can do it. Fill this form to send an activation request.
                    </p>
                </div>
                <field name="module_id" invisible="1"/>
                <group string="Send to:">
                    <field colspan="2" name="user_ids" widget="many2many_tags" nolabel="1"/>
                </group>
                <group string="Why do you need this module ?">
                    <field colspan="2" name="body_html" widget="html" nolabel="1" placeholder="e.g. I'd like to use the SMS Marketing module to organize the promotion of our internal events, and exhibitions. I need access for 3 people of my team."/>
                </group>
                <footer>
                    <button string="Request Activation" class="btn-primary" type="object" name="action_send_request" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="base_module_install_review_view_form" model="ir.ui.view">
        <field name="name">base.module.install.review.view.form</field>
        <field name="model">base.module.install.review</field>
        <field name="arch" type="xml">
            <form>
                <div class="alert alert-success oe_button_box" role="alert">
                    <p class="mt-3">
                        No extra cost, this application is free.
                    </p>
                </div>
                <field name="module_id" invisible="1"/>
                <field name="module_ids" widget="many2many_tags" invisible="1"/>
                <field name="modules_description"/>
                <footer>
                    <button string="Install App" class="btn-primary" type="object" name="action_install_module" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="action_base_module_install_review" model="ir.actions.act_window">
        <field name="name">You are about to install an extra application</field>
        <field name="res_model">base.module.install.review</field>
        <field name="view_mode">form</field>
        <field name="context">{ 'default_module_id': active_id }</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_module_install_request

```

