# Odoo Module: sale_project

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from . import report

def _set_allow_billable_in_project(env):
    Project = env['project.project']
    Task = env['project.task']
    projects = Project.search(Project._get_projects_to_make_billable_domain())
    non_billable_projects, = Task._read_group(
        Task._get_projects_to_make_billable_domain([('project_id', 'not in', projects.ids)]),
        [],
        ['project_id:recordset'],
    )[0]
    projects += non_billable_projects
    projects.allow_billable = True


def uninstall_hook(env):
    actions = env['ir.embedded.actions'].search([
        ('parent_res_model', '=', 'project.project'),
        ('python_method', 'in', ['action_open_project_invoices', 'action_view_sos'])
    ])
    actions.domain = [(0, '=', 1)]

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Sales - Project",
    'summary': "Task Generation from Sales Orders",
    'description': """
Allows to create task from your sales order
=============================================
This module allows to generate a project/task from sales orders.
""",
    'category': 'Hidden',
    'depends': ['sale_management', 'sale_service', 'project_account'],
    'auto_install': ['sale_management', 'project_account'],
    'data': [
        'security/ir.model.access.csv',
        'security/sale_project_security.xml',
        'views/product_views.xml',
        'views/project_task_views.xml',
        'views/sale_order_line_views.xml',
        'views/sale_order_views.xml',
        'views/sale_project_portal_templates.xml',
        'views/project_update_template.xml',
        'views/project_sharing_views.xml',
        'views/project_views.xml',
        'data/sale_project_data.xml',
    ],
    'demo': [
        'data/sale_project_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'sale_project/static/src/components/project_right_side_panel/**/*',
            'sale_project/static/src/views/**/*',
        ],
        'web.assets_tests': [
            'sale_project/static/tests/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'sale_project/static/tests/**/*.js',
        ],
    },
    'post_init_hook': '_set_allow_billable_in_project',
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _

from odoo.addons.project.controllers.portal import ProjectCustomerPortal


class SaleProjectCustomerPortal(ProjectCustomerPortal):

    def _task_get_searchbar_groupby(self, milestones_allowed, project=False):
        values = super()._task_get_searchbar_groupby(milestones_allowed, project)
        if project and not project.sudo()._get_hide_partner():
            del values['partner_id']
        if not project or project.sudo().allow_billable:
            values |= {
                'sale_line_id': {'label': _('Sales Order Item'), 'sequence': 80},
            }
        return values

    def _task_get_searchbar_inputs(self, milestones_allowed, project=False):
        values = super()._task_get_searchbar_inputs(milestones_allowed, project)
        if project and not project.sudo()._get_hide_partner():
            del values['partner_id']
        if not project or project.sudo().allow_billable:
            values |= {
                'sale_order':  {'input': 'sale_order', 'label': _('Search in Sales Order'), 'sequence': 90},
                'invoice': {'input': 'invoice', 'label': _('Search in Invoice'), 'sequence': 100},
            }
        return values

    def _task_get_search_domain(self, search_in, search, milestones_allowed, project):
        if search_in == 'sale_order':
            return ['|', ('sale_order_id.name', 'ilike', search), ('sale_line_id.name', 'ilike', search)]
        elif search_in == 'invoice':
            return [('sale_order_id.invoice_ids.name', 'ilike', search)]
        else:
            return super()._task_get_search_domain(search_in, search, milestones_allowed, project)

    def _prepare_project_sharing_session_info(self, project, task=None):
        session_info = super()._prepare_project_sharing_session_info(project, task)
        session_info['action_context'].update({
            'allow_billable': project.allow_billable,
        })
        return session_info

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\sale_project_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Project Task export template -->
    <record id="project_task_export_template_line_sale_line_id" model="ir.exports.line">
        <field name="export_id" ref="project.project_task_export_template"/>
        <field name="name">sale_line_id</field>
    </record>

    <!-- Product template export template -->
    <record id="services_template_export_template" model="ir.exports">
        <field name="name">Services</field>
        <field name="resource">product.template</field>
    </record>

    <record id="services_template_export_template_line_id" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">id</field>
    </record>

    <record id="services_template_export_template_line_name" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">name</field>
    </record>

    <record id="services_template_export_template_line_product_tag_ids" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">product_tag_ids</field>
    </record>

    <record id="services_template_export_template_line_list_price" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">list_price</field>
    </record>

    <record id="services_template_export_template_line_type" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">type</field>
    </record>

    <record id="services_template_export_template_line_uom_id" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">uom_id</field>
    </record>

    <record id="services_template_export_template_invoice_policy" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">service_policy</field>
    </record>

    <record id="services_template_export_template_service_tracking" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">service_tracking</field>
    </record>

    <record id="services_template_export_template_project_id" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">project_id</field>
    </record>

    <record id="services_template_export_template_project_template_id" model="ir.exports.line">
        <field name="export_id" ref="services_template_export_template"/>
        <field name="name">project_template_id</field>
    </record>
</odoo>

```

## File: data\sale_project_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Project Template -->
        <record id="so_template_project" model="project.project">
            <field name="name">Sales Order</field>
            <field name="active">False</field>
            <field name="type_ids" eval="[Command.link(ref('project.project_stage_0')), Command.link(ref('project.project_stage_1')), Command.link(ref('project.project_stage_2'))]"/>
            <field name="description">This project is used as a template for projects created from sales orders.</field>
        </record>

        <record id="product_service_create_project_and_task" model="product.product">
            <field name="name">Website Redesign Service (project &amp; task)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="type">service</field>
            <field name="list_price">66.60</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_in_project</field>
            <field name="project_template_id" ref="so_template_project"/>
        </record>

        <record id="product_service_create_project_only" model="product.product">
            <field name="name">Digital Marketing Campaign (project)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="type">service</field>
            <field name="list_price">123.00</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">project_only</field>
            <field name="project_template_id" ref="so_template_project"/>
        </record>

        <record id="product_service_create_task_only" model="product.product">
            <field name="name">Office Furniture Set (task)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="type">service</field>
            <field name="list_price">42.42</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_project_1"/>
        </record>

        <function model="project.project" name="write">
            <value model="project.project" search="[('id', '=', ref('project.project_home_construction'))]"/>
            <value eval="{
                'allow_billable': 'True',
            }"/>
        </function>

        <!-- Task in project product -->
        <record id="product_product_painting" model="product.product">
            <field name="name">Painting</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="list_price">1000.00</field>
            <field name="standard_price">1500.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>

        <record id="product.product_product_furniture" model="product.product">
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>

        <record id="product_product_flooring" model="product.product">
            <field name="name">Flooring Services</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="list_price">700.00</field>
            <field name="standard_price">1000.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>

        <record id="product_product_plumbing" model="product.product">
            <field name="name">Plumbing Services</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="list_price">500.00</field>
            <field name="standard_price">700.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>

        <record id="product_product_wiring" model="product.product">
            <field name="name">Wiring Services</field>
            <field name="categ_id" ref="product.product_category_construction"/>
            <field name="list_price">1500.00</field>
            <field name="standard_price">2000.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_prepaid</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project.project_home_construction"/>
        </record>

        <record id="product_product_screw_driver" model="product.product">
            <field name="name">Screw Driver</field>
            <field name="categ_id" ref="product.product_category_consumable"/>
            <field name="standard_price">100.00</field>
            <field name="list_price">150.00</field>
            <field name="type">consu</field>
            <field name="weight">0.75</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
        </record>

        <record id="sale_order_construction_material" model="sale.order">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <record id="sale_line_construction_material_1" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction_material"/>
            <field name="sequence" eval="1"/>
            <field name="product_id" ref="sale_project.product_product_wiring"/>
            <field name="product_uom_qty">5</field>
        </record>

        <record id="sale_line_construction_material_2" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction_material"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="sale_project.product_product_screw_driver"/>
            <field name="product_uom_qty">50</field>
        </record>

        <!-- Sale order for Project Home Construction -->
        <record id="sale_order_construction" model="sale.order">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="client_order_ref">MANUAL</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="tag_ids" eval="[Command.link(ref('sales_team.categ_oppor6'))]"/>
        </record>

        <record id="sale_line_construction_1" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="product_product_painting"/>
            <field name="product_uom_qty">7</field>
        </record>

        <record id="sale_line_construction_2" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="product.product_product_furniture"/>
            <field name="product_uom_qty">15</field>
            <field name="qty_delivered">12</field>
        </record>

        <record id="sale_line_construction_3" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="product_product_flooring"/>
            <field name="product_uom_qty">10</field>
            <field name="qty_delivered">8</field>
        </record>

        <record id="sale_line_construction_4" model="sale.order.line">
            <field name="order_id" ref="sale_project.sale_order_construction"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="product_product_plumbing"/>
            <field name="product_uom_qty">12</field>
            <field name="qty_delivered">10</field>
        </record>

        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_construction_material')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_construction')]]"/>

        <!-- Set users and state to task -->
        <function model="project.task" name="write">
            <value model="project.task" search="[('sale_line_id', '=', ref('sale_project.sale_line_construction_1'))]"/>
            <value eval="{
                'user_ids': [Command.link(ref('base.user_admin'))],
                'state': '02_changes_requested',
            }"/>
        </function>
        <function model="project.task" name="write">
            <value model="project.task" search="[('sale_line_id', '=', ref('sale_project.sale_line_construction_2'))]"/>
            <value eval="{
                'user_ids': [Command.link(ref('base.user_demo'))],
                'state': '03_approved',
            }"/>
        </function>
        <function model="project.task" name="write">
            <value model="project.task" search="[('sale_line_id', '=', ref('sale_project.sale_line_construction_3'))]"/>
            <value eval="{
                'milestone_id': ref('project.project_home_construction_milestone_1'),
                'stage_id': ref('project.project_stage_2'),
                'user_ids':[Command.link(ref('base.user_admin'))],
                'state': '1_done',
            }"/>
        </function>
        <function model="project.task" name="write">
            <value model="project.task" search="[('sale_line_id', '=', ref('sale_project.sale_line_construction_4'))]"/>
            <value eval="{
                'milestone_id': ref('project.project_home_construction_milestone_1'),
                'user_ids': [Command.link(ref('base.user_demo'))],
            }"/>
        </function>
        <function model="project.task" name="write">
            <value model="project.task" search="[('sale_line_id', '=', ref('sale_project.sale_line_construction_material_1'))]"/>
            <value eval="{
                'milestone_id': ref('project.project_home_construction_milestone_3'),
                'user_ids': [Command.link(ref('base.user_demo'))],
            }"/>
        </function>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_action_per_item(self):
        action = self.env.ref('account.action_move_out_invoice_type').id
        return {invoice.id: action for invoice in self}

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv.expression import AND, OR


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _compute_analytic_distribution(self):
        # when a project creates an aml, it adds an analytic account to it. the following filter is to save this
        # analytic account from being overridden by analytic default rules and lack thereof
        project_amls = self.filtered(lambda aml: aml.analytic_distribution and any(aml.sale_line_ids.project_id))
        super(AccountMoveLine, self - project_amls)._compute_analytic_distribution()
        project_id = self._context.get('project_id', False)
        if project_id:
            project = self.env['project.project'].browse(project_id)
            lines = self.filtered(lambda line: line.account_type not in ['asset_receivable', 'liability_payable'])
            lines.analytic_distribution = project._get_analytic_distribution()

    def _get_so_mapping_domain(self):
        return OR([
            OR([
                AND([
                    [(self.env['account.analytic.account'].browse(int(account_id)).root_plan_id._column_name(), "=", int(account_id))]
                    for account_id in key.split(",")
                ])
                for key in line.analytic_distribution
            ])
            for line in self
        ])

    def _get_so_mapping_from_project(self):
        """ Get the mapping of move.line with the sale.order record on which its analytic entries should be reinvoiced.
            A sale.order matches a move.line if the sale.order's project contains all the same analytic accounts
            as the ones in the distribution of the move.line.
            :return a dict where key is the move line id, and value is sale.order record (or None).
        """
        mapping = {}
        projects = self.env['project.project'].search(domain=self._get_so_mapping_domain())
        orders_per_project = dict(self.env['sale.order']._read_group(
            domain=[('project_id', 'in', projects.ids)],
            groupby=['project_id'],
            aggregates=['id:recordset']
        ))
        project_per_accounts = {
            next(iter(project._get_analytic_distribution())): project
            for project in projects
        }

        for move_line in self:
            analytic_distribution = move_line.analytic_distribution
            if not analytic_distribution:
                continue

            for accounts in analytic_distribution:
                project = project_per_accounts.get(accounts)
            if not project:
                continue

            orders = orders_per_project.get(project)
            if not orders:
                continue
            orders = orders.sorted('create_date')
            in_sale_state_orders = orders.filtered(lambda s: s.state == 'sale')

            mapping[move_line.id] = in_sale_state_orders[0] if in_sale_state_orders else orders[0]

        # map the move line index with the SO on which it needs to be reinvoiced. May be empty if no SO found
        return mapping

    def _sale_determine_order(self):
        mapping_from_invoice = super()._sale_determine_order()
        mapping_from_invoice.update(self._get_so_mapping_from_project())
        return mapping_from_invoice

```

## File: models\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductProduct(models.Model):
    _inherit = 'product.product'

    @api.onchange('service_tracking')
    def _onchange_service_tracking(self):
        if self.service_tracking == 'no':
            self.project_id = False
            self.project_template_id = False
        elif self.service_tracking == 'task_global_project':
            self.project_template_id = False
        elif self.service_tracking in ['task_in_project', 'project_only']:
            self.project_id = False

    def _inverse_service_policy(self):
        for product in self:
            if product.service_policy:

                product.invoice_policy, product.service_type = self.product_tmpl_id._get_service_to_general(product.service_policy)

    def write(self, vals):
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super().write(vals)

```

## File: models\product_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _, SUPERUSER_ID
from odoo.exceptions import ValidationError


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.model
    def _selection_service_policy(self):
        service_policies = [
            # (service_policy, string)
            ('ordered_prepaid', _('Prepaid/Fixed Price')),
            ('delivered_manual', _('Based on Delivered Quantity (Manual)')),
        ]

        user = self.env['res.users'].sudo().browse(SUPERUSER_ID)
        if (self.env.user.has_group('project.group_project_milestone') or
                (self.env.user.has_group('base.group_public') and user.has_group('project.group_project_milestone'))
        ):
            service_policies.insert(1, ('delivered_milestones', _('Based on Milestones')))
        return service_policies

    service_tracking = fields.Selection(
        selection_add=[
            ('task_global_project', 'Task'),
            ('task_in_project', 'Project & Task'),
            ('project_only', 'Project'),
        ], ondelete={
            'task_global_project': 'set default',
            'task_in_project': 'set default',
            'project_only': 'set default',
        },
    )
    project_id = fields.Many2one(
        'project.project', 'Project', company_dependent=True, copy=True,
    )
    project_template_id = fields.Many2one(
        'project.project', 'Project Template', company_dependent=True, copy=True,
    )
    service_policy = fields.Selection('_selection_service_policy', string="Service Invoicing Policy", compute_sudo=True, compute='_compute_service_policy', inverse='_inverse_service_policy', tracking=True)
    service_type = fields.Selection(selection_add=[
        ('milestones', 'Project Milestones'),
    ])

    @api.depends('invoice_policy', 'service_type', 'type')
    def _compute_service_policy(self):
        for product in self:
            product.service_policy = self._get_general_to_service(product.invoice_policy, product.service_type)
            if not product.service_policy and product.type == 'service':
                product.service_policy = 'ordered_prepaid'

    @api.depends('service_policy')
    def _compute_product_tooltip(self):
        super()._compute_product_tooltip()

    def _prepare_service_tracking_tooltip(self):
        if self.service_tracking == 'task_global_project':
            return _("Create a task in an existing project to track the time spent.")
        elif self.service_tracking == 'project_only':
            return _(
                "Create an empty project for the order to track the time spent."
            )
        elif self.service_tracking == 'task_in_project':
            return _(
                "Create a project for the order with a task for each sales order line "
                "to track the time spent."
            )
        elif self.service_tracking == 'no':
            return _(
                "Create projects or tasks later, and link them to order to track the time spent."
            )
        return super()._prepare_service_tracking_tooltip()

    def _prepare_invoicing_tooltip(self):
        if self.service_policy == 'delivered_milestones':
            return _("Invoice your milestones when they are reached.")
        # ordered_prepaid and delivered_manual are handled in the super call, according to the
        # corresponding value in the `invoice_policy` field (delivered/ordered quantities)
        return super()._prepare_invoicing_tooltip()

    def _get_service_to_general_map(self):
        return {
            # service_policy: (invoice_policy, service_type)
            'ordered_prepaid': ('order', 'manual'),
            'delivered_milestones': ('delivery', 'milestones'),
            'delivered_manual': ('delivery', 'manual'),
        }

    def _get_general_to_service_map(self):
        return {v: k for k, v in self._get_service_to_general_map().items()}

    def _get_service_to_general(self, service_policy):
        return self._get_service_to_general_map().get(service_policy, (False, False))

    def _get_general_to_service(self, invoice_policy, service_type):
        general_to_service = self._get_general_to_service_map()
        return general_to_service.get((invoice_policy, service_type), False)

    @api.onchange('service_policy')
    def _inverse_service_policy(self):
        for product in self:
            if product.service_policy:
                product.invoice_policy, product.service_type = self._get_service_to_general(product.service_policy)

    @api.constrains('project_id', 'project_template_id')
    def _check_project_and_template(self):
        """ NOTE 'service_tracking' should be in decorator parameters but since ORM check constraints twice (one after setting
            stored fields, one after setting non stored field), the error is raised when company-dependent fields are not set.
            So, this constraints does cover all cases and inconsistent can still be recorded until the ORM change its behavior.
        """
        for product in self:
            if product.service_tracking == 'no' and (product.project_id or product.project_template_id):
                raise ValidationError(_('The product %s should not have a project nor a project template since it will not generate project.', product.name))
            elif product.service_tracking == 'task_global_project' and product.project_template_id:
                raise ValidationError(_('The product %s should not have a project template since it will generate a task in a global project.', product.name))
            elif product.service_tracking in ['task_in_project', 'project_only'] and product.project_id:
                raise ValidationError(_('The product %s should not have a global project since it will generate a project.', product.name))

    @api.onchange('service_tracking')
    def _onchange_service_tracking(self):
        if self.service_tracking == 'no':
            self.project_id = False
            self.project_template_id = False
        elif self.service_tracking == 'task_global_project':
            self.project_template_id = False
        elif self.service_tracking in ['task_in_project', 'project_only']:
            self.project_id = False

    def write(self, vals):
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super().write(vals)

    @api.model
    def _get_saleable_tracking_types(self):
        return super()._get_saleable_tracking_types() + [
            'task_global_project',
            'task_in_project',
            'project_only',
        ]

```

## File: models\project_milestone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _

class ProjectMilestone(models.Model):
    _name = 'project.milestone'
    _inherit = 'project.milestone'

    def _default_sale_line_id(self):
        project_id = self._context.get('default_project_id')
        if not project_id:
            return []
        project = self.env['project.project'].browse(project_id)
        return self.env['sale.order.line'].search([
            ('order_id', '=', project.sale_order_id.id),
            ('qty_delivered_method', '=', 'milestones'),
        ], limit=1)

    allow_billable = fields.Boolean(related='project_id.allow_billable', export_string_translation=False)
    project_partner_id = fields.Many2one(related='project_id.partner_id', export_string_translation=False)

    sale_line_id = fields.Many2one('sale.order.line', 'Sales Order Item', default=_default_sale_line_id, help='Sales Order Item that will be updated once the milestone is reached.',
        domain="[('order_partner_id', '=?', project_partner_id), ('qty_delivered_method', '=', 'milestones')]")
    quantity_percentage = fields.Float('Quantity (%)', compute="_compute_quantity_percentage", store=True, help='Percentage of the ordered quantity that will automatically be delivered once the milestone is reached.')

    sale_line_display_name = fields.Char("Sale Line Display Name", related='sale_line_id.display_name', export_string_translation=False)
    product_uom = fields.Many2one(related="sale_line_id.product_uom", export_string_translation=False)
    product_uom_qty = fields.Float("Quantity", compute="_compute_product_uom_qty", readonly=False)

    @api.depends('sale_line_id.product_uom_qty', 'product_uom_qty')
    def _compute_quantity_percentage(self):
        for milestone in self:
            milestone.quantity_percentage = milestone.sale_line_id.product_uom_qty and milestone.product_uom_qty / milestone.sale_line_id.product_uom_qty

    @api.depends('sale_line_id', 'quantity_percentage')
    def _compute_product_uom_qty(self):
        for milestone in self:
            if milestone.quantity_percentage:
                milestone.product_uom_qty = milestone.quantity_percentage * milestone.sale_line_id.product_uom_qty
            else:
                milestone.product_uom_qty = milestone.sale_line_id.product_uom_qty

    @api.model
    def _get_fields_to_export(self):
        return super()._get_fields_to_export() + ['allow_billable', 'quantity_percentage', 'sale_line_display_name']

    def action_view_sale_order(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Sales Order'),
            'res_model': 'sale.order',
            'res_id': self.sale_line_id.order_id.id,
            'view_mode': 'form',
        }

```

## File: models\project_project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from collections import defaultdict

from odoo import api, fields, models
from odoo.osv import expression
from odoo.tools import Query, SQL
from odoo.tools.misc import unquote
from odoo.tools.translate import _


class ProjectProject(models.Model):
    _inherit = 'project.project'

    def _domain_sale_line_id(self):
        domain = expression.AND([
            self.env['sale.order.line']._sellable_lines_domain(),
            self.env['sale.order.line']._domain_sale_line_service(),
            [
                ('order_partner_id', '=?', unquote("partner_id")),
            ],
        ])
        return domain

    allow_billable = fields.Boolean("Billable")
    sale_line_id = fields.Many2one(
        'sale.order.line', 'Sales Order Item', copy=False,
        compute="_compute_sale_line_id", store=True, readonly=False, index='btree_not_null',
        domain=lambda self: str(self._domain_sale_line_id()),
        help="Sales order item that will be selected by default on the tasks and timesheets of this project,"
            " except if the employee set on the timesheets is explicitely linked to another sales order item on the project.\n"
            "It can be modified on each task and timesheet entry individually if necessary.")
    sale_order_id = fields.Many2one(related='sale_line_id.order_id', export_string_translation=False)
    has_any_so_to_invoice = fields.Boolean('Has SO to Invoice', compute='_compute_has_any_so_to_invoice', export_string_translation=False)
    sale_order_line_count = fields.Integer(compute='_compute_sale_order_count', groups='sales_team.group_sale_salesman', export_string_translation=False)
    sale_order_count = fields.Integer(compute='_compute_sale_order_count', groups='sales_team.group_sale_salesman', export_string_translation=False)
    has_any_so_with_nothing_to_invoice = fields.Boolean('Has a SO with an invoice status of No', compute='_compute_has_any_so_with_nothing_to_invoice', export_string_translation=False)
    invoice_count = fields.Integer(compute='_compute_invoice_count', groups='account.group_account_readonly', export_string_translation=False)
    vendor_bill_count = fields.Integer(related='account_id.vendor_bill_count', groups='account.group_account_readonly', export_string_translation=False)
    partner_id = fields.Many2one(compute="_compute_partner_id", store=True, readonly=False)
    display_sales_stat_buttons = fields.Boolean(compute='_compute_display_sales_stat_buttons', export_string_translation=False)
    sale_order_state = fields.Selection(related='sale_order_id.state', export_string_translation=False)
    reinvoiced_sale_order_id = fields.Many2one('sale.order', string='Sales Order', groups='sales_team.group_sale_salesman', copy=False, domain="[('partner_id', '=', partner_id)]",
        help="Products added to stock pickings, whose operation type is configured to generate analytic costs, will be re-invoiced in this sales order if they are set up for it.",
    )

    @api.model
    def _map_tasks_default_values(self, project):
        defaults = super()._map_tasks_default_values(project)
        defaults['sale_line_id'] = False
        return defaults

    @api.depends('allow_billable', 'partner_id.company_id')
    def _compute_partner_id(self):
        for project in self:
            # Ensures that the partner_id and its project do not have different companies set
            if not project.allow_billable or (project.company_id and project.partner_id.company_id and project.company_id != project.partner_id.company_id):
                project.partner_id = False

    @api.depends('partner_id')
    def _compute_sale_line_id(self):
        self.filtered(
            lambda p:
                p.sale_line_id and (
                    not p.partner_id or p.sale_line_id.order_partner_id.commercial_partner_id != p.partner_id.commercial_partner_id
                )
        ).update({'sale_line_id': False})

    def _get_projects_for_invoice_status(self, invoice_status):
        """ Returns a recordset of project.project that has any Sale Order which invoice_status is the same as the
            provided invoice_status.

            :param invoice_status: The invoice status.
        """
        result = self.env.execute_query(SQL("""
            SELECT id
              FROM project_project pp
             WHERE pp.active = true
               AND (   EXISTS(SELECT 1
                                FROM sale_order so
                                JOIN project_task pt ON pt.sale_order_id = so.id
                               WHERE pt.project_id = pp.id
                                 AND pt.active = true
                                 AND so.invoice_status = %(invoice_status)s)
                    OR EXISTS(SELECT 1
                                FROM sale_order so
                                JOIN sale_order_line sol ON sol.order_id = so.id
                               WHERE sol.id = pp.sale_line_id
                                 AND so.invoice_status = %(invoice_status)s))
               AND id in %(ids)s""", ids=tuple(self.ids), invoice_status=invoice_status))
        return self.env['project.project'].browse(id_ for id_, in result)

    @api.depends('sale_order_id.invoice_status', 'tasks.sale_order_id.invoice_status')
    def _compute_has_any_so_to_invoice(self):
        """Has any Sale Order whose invoice_status is set as To Invoice"""
        if not self.ids:
            self.has_any_so_to_invoice = False
            return

        project_to_invoice = self._get_projects_for_invoice_status('to invoice')
        project_to_invoice.has_any_so_to_invoice = True
        (self - project_to_invoice).has_any_so_to_invoice = False

    @api.depends('sale_order_id', 'task_ids.sale_order_id')
    def _compute_sale_order_count(self):
        sale_order_items_per_project_id = self._fetch_sale_order_items_per_project_id({'project.task': [('is_closed', '=', False)]})
        for project in self:
            sale_order_lines = sale_order_items_per_project_id.get(project.id, self.env['sale.order.line'])
            project.sale_order_line_count = len(sale_order_lines)

            # Use sudo to avoid AccessErrors when the SOLs belong to different companies.
            project.sale_order_count = len(sale_order_lines.sudo().order_id)

    def _compute_invoice_count(self):
        data = self.env['account.move.line']._read_group(
            [('move_id.move_type', 'in', ['out_invoice', 'out_refund']), ('analytic_distribution', 'in', self.account_id.ids)],
            groupby=['analytic_distribution'],
            aggregates=['__count'],
        )
        data = {int(account_id): move_count for account_id, move_count in data}
        for project in self:
            project.invoice_count = data.get(project.account_id.id, 0)

    @api.depends('allow_billable', 'partner_id')
    def _compute_display_sales_stat_buttons(self):
        for project in self:
            project.display_sales_stat_buttons = project.allow_billable and project.partner_id

    def action_customer_preview(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': self.get_portal_url(),
        }

    @api.onchange('reinvoiced_sale_order_id')
    def _onchange_reinvoiced_sale_order_id(self):
        if not self.sale_line_id and self.reinvoiced_sale_order_id.order_line:
            self.sale_line_id = self.reinvoiced_sale_order_id.order_line[0]

    @api.onchange('sale_line_id')
    def _onchange_sale_line_id(self):
        if not self.reinvoiced_sale_order_id and self.sale_line_id:
            self.reinvoiced_sale_order_id = self.sale_line_id.order_id

    def _ensure_sale_order_linked(self, sol_ids):
        """ Orders created from project/task are supposed to be confirmed to match the typical flow from sales, but since
        we allow SO creation from the project/task itself we want to confirm newly created SOs immediately after creation.
        However this would leads to SOs being confirmed without a single product, so we'd rather do it on record save.
        """
        quotations = self.env['sale.order.line'].sudo()._read_group(
            domain=[('state', '=', 'draft'), ('id', 'in', sol_ids)],
            aggregates=['order_id:recordset'],
        )[0][0]
        if quotations:
            quotations.action_confirm()

    @api.model_create_multi
    def create(self, vals_list):
        projects = super().create(vals_list)
        sol_ids = {
            vals['sale_line_id']
            for vals in vals_list
            if vals.get('sale_line_id')
        }
        if sol_ids:
            projects._ensure_sale_order_linked(list(sol_ids))
        return projects

    def write(self, vals):
        project = super().write(vals)
        if sol_id := vals.get('sale_line_id'):
            self._ensure_sale_order_linked([sol_id])
        return project

    def action_view_sols(self):
        self.ensure_one()
        all_sale_order_lines = self._fetch_sale_order_items({'project.task': [('is_closed', '=', False)]})
        action_window = {
            'type': 'ir.actions.act_window',
            'res_model': 'sale.order.line',
            'name': _("%(name)s's Sales Order Items", name=self.name),
            'context': {
                'show_sale': True,
                'link_to_project': self.id,
                'form_view_ref': 'sale_project.sale_order_line_view_form_editable',  # Necessary for some logic in the form view
                'action_view_sols': True,
                'default_partner_id': self.partner_id.id,
                'default_company_id': self.company_id.id,
                'default_order_id': self.sale_order_id.id,
            },
            'views': [(self.env.ref('sale_project.sale_order_line_view_form_editable').id, 'form')],
        }
        if len(all_sale_order_lines) <= 1:
            action_window['res_id'] = all_sale_order_lines.id
        else:
            action_window.update({
                'domain': [('id', 'in', all_sale_order_lines.ids)],
                'views': [
                    (self.env.ref('sale_project.view_order_line_tree_with_create').id, 'list'),
                    (self.env.ref('sale_project.sale_order_line_view_form_editable').id, 'form'),
                ],
            })
        return action_window

    def action_view_sos(self):
        self.ensure_one()
        all_sale_orders = self._fetch_sale_order_items({'project.task': [('is_closed', '=', False)]}).sudo().order_id
        embedded_action_context = self.env.context.get('from_embedded_action', False)
        action_window = {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            'name': _("%(name)s's Sales Orders", name=self.name),
            "context": {
                "create": self.env.context.get('create_for_project_id', embedded_action_context),
                "show_sale": True,
                'default_partner_id': self.partner_id.id,
                'default_project_id': self.id,
                "create_for_project_id": self.id if not embedded_action_context else False,
                "from_embedded_action": embedded_action_context
            },
            'help': "<p class='o_view_nocontent_smiling_face'>%s</p><p>%s<br/>%s</p>" %
            (_("Create a new quotation, the first step of a new sale!"),
                _("Once the quotation is confirmed by the customer, it becomes a sales order."),
                _("You will be able to create an invoice and collect the payment."))
        }
        if len(all_sale_orders) <= 1 and not embedded_action_context:
            action_window.update({
                "res_id": all_sale_orders.id,
                "views": [[False, "form"]],
            })
        else:
            action_window.update({
                "domain": [('id', 'in', all_sale_orders.ids)],
                "views": [[False, "list"], [False, "kanban"], [False, "calendar"], [False, "pivot"],
                           [False, "graph"], [False, "activity"], [False, "form"]],
            })
        return action_window

    def action_get_list_view(self):
        action = super().action_get_list_view()
        if self.allow_billable:
            action['views'] = [(self.env.ref('sale_project.project_milestone_view_tree').id, 'list'), (False, 'form')]
        return action

    def action_profitability_items(self, section_name, domain=None, res_id=False):
        if section_name in ['service_revenues', 'materials']:
            view_types = ['list', 'kanban', 'form']
            action = {
                'name': _('Sales Order Items'),
                'type': 'ir.actions.act_window',
                'res_model': 'sale.order.line',
                'context': {'create': False, 'edit': False},
            }
            if res_id:
                action['res_id'] = res_id
                view_types = ['form']
            else:
                action['domain'] = domain
            action['views'] = [(False, v) for v in view_types]
            return action

        if section_name in ['other_invoice_revenues', 'downpayments']:
            action = self.env["ir.actions.actions"]._for_xml_id("account.action_move_out_invoice_type")
            action['domain'] = domain if domain else []
            action['context'] = {
                **ast.literal_eval(action['context']),
                'default_partner_id': self.partner_id.id,
                'project_id': self.id,
            }
            if res_id:
                action['views'] = [(False, 'form')]
                action['view_mode'] = 'form'
                action['res_id'] = res_id
            return action

        return super().action_profitability_items(section_name, domain, res_id)

    @api.depends('sale_order_id.invoice_status', 'tasks.sale_order_id.invoice_status')
    def _compute_has_any_so_with_nothing_to_invoice(self):
        """Has any Sale Order whose invoice_status is set as No"""
        if not self.ids:
            self.has_any_so_with_nothing_to_invoice = False
            return

        project_nothing_to_invoice = self._get_projects_for_invoice_status('no')
        project_nothing_to_invoice.has_any_so_with_nothing_to_invoice = True
        (self - project_nothing_to_invoice).has_any_so_with_nothing_to_invoice = False

    def action_create_invoice(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_view_sale_advance_payment_inv")
        so_ids = (self.sale_order_id | self.task_ids.sale_order_id).filtered(lambda so: so.invoice_status in ['to invoice', 'no']).ids
        action['context'] = {
            'active_id': so_ids[0] if len(so_ids) == 1 else False,
            'active_ids': so_ids
        }
        if not self.has_any_so_to_invoice:
            action['context']['default_advance_payment_method'] = 'percentage'
        return action

    def action_open_project_invoices(self):
        move_lines = self.env['account.move.line'].search_fetch(
            [
                ('move_id.move_type', 'in', ['out_invoice', 'out_refund']),
                ('analytic_distribution', 'in', self.account_id.ids),
            ],
            ['move_id'],
        )
        invoice_ids = move_lines.move_id.ids
        action = {
            'name': _('Invoices'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'list'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', invoice_ids)],
            'context': {
                'default_move_type': 'out_invoice',
                'default_partner_id': self.partner_id.id,
                'project_id': self.id
            },
            'help': "<p class='o_view_nocontent_smiling_face'>%s</p><p>%s</p>" %
            (_("Create a customer invoice"),
                _("Create invoices, register payments and keep track of the discussions with your customers."))
        }
        if len(invoice_ids) == 1 and not self.env.context.get('from_embedded_action', False):
            action['views'] = [[False, 'form']]
            action['res_id'] = invoice_ids[0]
        return action

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _fetch_sale_order_items_per_project_id(self, domain_per_model=None):
        if not self:
            return {}
        if len(self) == 1:
            return {self.id: self._fetch_sale_order_items(domain_per_model)}
        sql = self._get_sale_order_items_query(domain_per_model).select('id', 'ARRAY_AGG(DISTINCT sale_line_id) AS sale_line_ids')
        sql = SQL("%s GROUP BY id", sql)
        return {
            id_: self.env['sale.order.line'].browse(sale_line_ids)
            for id_, sale_line_ids in self.env.execute_query(sql)
        }

    def _fetch_sale_order_items(self, domain_per_model=None, limit=None, offset=None):
        return self.env['sale.order.line'].browse(self._fetch_sale_order_item_ids(domain_per_model, limit, offset))

    def _fetch_sale_order_item_ids(self, domain_per_model=None, limit=None, offset=None):
        if not self or not self.filtered('allow_billable'):
            return []
        query = self._get_sale_order_items_query(domain_per_model)
        query.limit = limit
        query.offset = offset
        return [id_ for id_, in self.env.execute_query(query.select('DISTINCT sale_line_id'))]

    def _get_sale_orders(self):
        return self._get_sale_order_items().order_id

    def _get_sale_order_items(self):
        return self._fetch_sale_order_items()

    def _get_sale_order_items_query(self, domain_per_model=None):
        if domain_per_model is None:
            domain_per_model = {}
        billable_project_domain = [('allow_billable', '=', True)]
        project_domain = [('id', 'in', self.ids), ('sale_line_id', '!=', False)]
        if 'project.project' in domain_per_model:
            project_domain = expression.AND([
                domain_per_model['project.project'],
                project_domain,
                billable_project_domain,
            ])
        project_query = self.env['project.project']._where_calc(project_domain)
        self._apply_ir_rules(project_query, 'read')
        project_sql = project_query.select(f'{self._table}.id ', f'{self._table}.sale_line_id')

        Task = self.env['project.task']
        task_domain = [('project_id', 'in', self.ids), ('sale_line_id', '!=', False)]
        if Task._name in domain_per_model:
            task_domain = expression.AND([
                domain_per_model[Task._name],
                task_domain,
            ])
        task_query = Task._where_calc(task_domain)
        Task._apply_ir_rules(task_query, 'read')
        task_sql = task_query.select(f'{Task._table}.project_id AS id', f'{Task._table}.sale_line_id')

        ProjectMilestone = self.env['project.milestone']
        milestone_domain = [('project_id', 'in', self.ids), ('allow_billable', '=', True), ('sale_line_id', '!=', False)]
        if ProjectMilestone._name in domain_per_model:
            milestone_domain = expression.AND([
                domain_per_model[ProjectMilestone._name],
                milestone_domain,
                billable_project_domain,
            ])
        milestone_query = ProjectMilestone._where_calc(milestone_domain)
        ProjectMilestone._apply_ir_rules(milestone_query)
        milestone_sql = milestone_query.select(
            f'{ProjectMilestone._table}.project_id AS id',
            f'{ProjectMilestone._table}.sale_line_id',
        )

        SaleOrderLine = self.env['sale.order.line']
        sale_order_line_domain = [
            '&',
                ('display_type', '=', False),
                ('order_id', 'any', ['|',
                    ('id', 'in', self.reinvoiced_sale_order_id.ids),
                    ('project_id', 'in', self.ids),
                ]),
        ]
        sale_order_line_query = SaleOrderLine._where_calc(sale_order_line_domain)
        sale_order_line_sql = sale_order_line_query.select(
            f'{SaleOrderLine._table}.project_id AS id',
            f'{SaleOrderLine._table}.id AS sale_line_id',
        )

        return Query(self.env, 'project_sale_order_item', SQL('(%s)', SQL(' UNION ').join([
            project_sql, task_sql, milestone_sql, sale_order_line_sql,
        ])))

    def get_panel_data(self):
        panel_data = super().get_panel_data()
        foldable_sections = self._get_foldable_section()
        if self._show_profitability() and 'revenues' in panel_data['profitability_items']:
            for section in panel_data['profitability_items']['revenues']['data']:
                if section['id'] in foldable_sections:
                    section['isSectionFoldable'] = True
        return {
            **panel_data,
            'show_sale_items': self.allow_billable,
        }

    def _get_foldable_section(self):
        return ['materials', 'service_revenues']

    def get_sale_items_data(self, offset=0, limit=None, with_action=True, section_id=None):
        if not self.env.user.has_group('project.group_project_user'):
            return {}

        all_sols = self.env['sale.order.line'].sudo().search(
            self._get_domain_from_section_id(section_id),
            offset=offset,
            limit=limit + 1,
        )
        display_load_more = False
        if len(all_sols) > limit:
            all_sols = all_sols - all_sols[limit]
            display_load_more = True

        # filter to only get the action for the SOLs that the user can read
        action_per_sol = all_sols.sudo(False)._filtered_access('read')._get_action_per_item() if with_action else {}

        def get_action(sol_id):
            """ Return the action vals to call it in frontend if the user can access to the SO related """
            action, res_id = action_per_sol.get(sol_id, (None, None))
            return {'action': {'name': action, 'resId': res_id, 'buttonContext': json.dumps({'active_id': sol_id, 'default_project_id': self.id})}} if action else {}

        return {
            'sol_items': [{
                **sol_read,
                **get_action(sol_read['id']),
            } for sol_read in all_sols.with_context(with_price_unit=True)._read_format(['name', 'product_uom_qty', 'qty_delivered', 'qty_invoiced', 'product_uom', 'product_id'])],
            'displayLoadMore': display_load_more,
        }

    def _get_sale_items_domain(self, additional_domain=None):
        sale_items = self.sudo()._get_sale_order_items()
        domain = [
            ('order_id', 'in', sale_items.sudo().order_id.ids),
            ('is_downpayment', '=', False),
            ('state', '=', 'sale'),
            ('display_type', '=', False),
            '|',
                ('project_id', 'in', [*self.ids, False]),
                ('id', 'in', sale_items.ids),
        ]
        if additional_domain:
            domain = expression.AND([domain, additional_domain])
        return domain

    def _get_domain_from_section_id(self, section_id):
        #  When the sale_timesheet module is not installed, all service products are grouped under the 'service revenues' section.
        return self._get_sale_items_domain([('product_type', '!=' if section_id == 'materials' else '=', 'service')])

    def _show_profitability(self):
        self.ensure_one()
        return self.allow_billable and super()._show_profitability()

    def _show_profitability_helper(self):
        return True

    def _get_profitability_labels(self):
        return {
            **super()._get_profitability_labels(),
            'service_revenues': self.env._('Other Services'),
            'materials': self.env._('Materials'),
            'other_invoice_revenues': self.env._('Customer Invoices'),
            'downpayments': self.env._('Down Payments'),
        }

    def _get_profitability_sequence_per_invoice_type(self):
        return {
            **super()._get_profitability_sequence_per_invoice_type(),
            'service_revenues': 6,
            'materials': 7,
            'other_invoice_revenues': 9,
            'downpayments': 20,
            'cost_of_goods_sold': 21,
        }

    def _get_service_policy_to_invoice_type(self):
        return {
            'ordered_prepaid': 'service_revenues',
            'delivered_milestones': 'service_revenues',
            'delivered_manual': 'service_revenues',
        }

    def _get_profitability_sale_order_items_domain(self, domain=None):
        if domain is None:
            domain = []
        return expression.AND([
            [
                '|', ('product_id', '!=', False), ('is_downpayment', '=', True),
                ('is_expense', '=', False),
                ('state', '=', 'sale'),
                '|', ('qty_to_invoice', '>', 0), ('qty_invoiced', '>', 0),
            ],
            domain,
        ])

    def _get_revenues_items_from_sol(self, domain=None, with_action=True):
        sale_line_read_group = self.env['sale.order.line'].sudo()._read_group(
            self._get_profitability_sale_order_items_domain(domain),
            ['currency_id', 'product_id', 'is_downpayment'],
            ['id:array_agg', 'untaxed_amount_to_invoice:sum', 'untaxed_amount_invoiced:sum'],
        )
        display_sol_action = with_action and len(self) == 1 and self.env.user.has_group('sales_team.group_sale_salesman')
        revenues_dict = {}
        total_to_invoice = total_invoiced = 0.0
        data = []
        sequence_per_invoice_type = self._get_profitability_sequence_per_invoice_type()
        if sale_line_read_group:
            # Get conversion rate from currencies of the sale order lines to currency of project
            convert_company = self.company_id or self.env.company

            sols_per_product = defaultdict(lambda: [0.0, 0.0, []])
            downpayment_amount_invoiced = 0
            downpayment_sol_ids = []
            for currency, product, is_downpayment, sol_ids, untaxed_amount_to_invoice, untaxed_amount_invoiced in sale_line_read_group:
                if is_downpayment:
                    downpayment_amount_invoiced += currency._convert(untaxed_amount_invoiced, convert_company.currency_id, convert_company, round=False)
                    downpayment_sol_ids += sol_ids
                else:
                    sols_per_product[product.id][0] += currency._convert(untaxed_amount_to_invoice, convert_company.currency_id, convert_company)
                    sols_per_product[product.id][1] += currency._convert(untaxed_amount_invoiced, convert_company.currency_id, convert_company)
                    sols_per_product[product.id][2] += sol_ids
            if downpayment_amount_invoiced:
                downpayments_data = {
                    'id': 'downpayments',
                    'sequence': sequence_per_invoice_type['downpayments'],
                    'invoiced': downpayment_amount_invoiced,
                    'to_invoice': -downpayment_amount_invoiced,
                }
                if with_action and (
                    self.env.user.has_group('sales_team.group_sale_salesman_all_leads,')
                    or self.env.user.has_group('account.group_account_invoice,')
                    or self.env.user.has_group('account.group_account_readonly')
                ):
                    invoices = self.env['account.move'].search([('line_ids.sale_line_ids', 'in', downpayment_sol_ids)])
                    args = ['downpayments', [('id', 'in', invoices.ids)]]
                    if len(invoices) == 1:
                        args.append(invoices.id)
                    downpayments_data['action'] = {
                        'name': 'action_profitability_items',
                        'type': 'object',
                        'args': json.dumps(args),
                    }
                data += [downpayments_data]
                total_invoiced += downpayment_amount_invoiced
                total_to_invoice -= downpayment_amount_invoiced
            product_read_group = self.env['product.product'].sudo()._read_group(
                [('id', 'in', list(sols_per_product))],
                ['invoice_policy', 'service_type', 'type'],
                ['id:array_agg'],
            )
            service_policy_to_invoice_type = self._get_service_policy_to_invoice_type()
            general_to_service_map = self.env['product.template']._get_general_to_service_map()
            for invoice_policy, service_type, type_, product_ids in product_read_group:
                service_policy = None
                if type_ == 'service':
                    service_policy = general_to_service_map.get(
                        (invoice_policy, service_type),
                        'ordered_prepaid')
                for product_id, (amount_to_invoice, amount_invoiced, sol_ids) in sols_per_product.items():
                    if product_id in product_ids:
                        invoice_type = service_policy_to_invoice_type.get(service_policy, 'materials')
                        revenue = revenues_dict.setdefault(invoice_type, {'invoiced': 0.0, 'to_invoice': 0.0})
                        revenue['to_invoice'] += amount_to_invoice
                        total_to_invoice += amount_to_invoice
                        revenue['invoiced'] += amount_invoiced
                        total_invoiced += amount_invoiced
                        if display_sol_action and invoice_type in ['service_revenues', 'materials']:
                            revenue.setdefault('record_ids', []).extend(sol_ids)

            if display_sol_action:
                section_name = 'materials'
                materials = revenues_dict.get(section_name, {})
                sale_order_items = self.env['sale.order.line'] \
                    .browse(materials.pop('record_ids', [])) \
                    ._filtered_access('read')
                if sale_order_items:
                    args = [section_name, [('id', 'in', sale_order_items.ids)]]
                    if len(sale_order_items) == 1:
                        args.append(sale_order_items.id)
                    action_params = {
                        'name': 'action_profitability_items',
                        'type': 'object',
                        'args': json.dumps(args),
                    }
                    if len(sale_order_items) == 1:
                        action_params['res_id'] = sale_order_items.id
                    materials['action'] = action_params
        sequence_per_invoice_type = self._get_profitability_sequence_per_invoice_type()
        data += [{
            'id': invoice_type,
            'sequence': sequence_per_invoice_type[invoice_type],
            **vals,
        } for invoice_type, vals in revenues_dict.items()]
        return {
            'data': data,
            'total': {'to_invoice': total_to_invoice, 'invoiced': total_invoiced},
        }

    def _get_revenues_items_from_invoices_domain(self, domain=None):
        if domain is None:
            domain = []
        included_invoice_line_ids = self._get_already_included_profitability_invoice_line_ids()
        return expression.AND([
            domain,
            [('move_id.move_type', 'in', self.env['account.move'].get_sale_types()),
            ('parent_state', 'in', ['draft', 'posted']),
            ('price_subtotal', '!=', 0),
            ('is_downpayment', '=', False),
            ('id', 'not in', included_invoice_line_ids)],
        ])

    # TODO: rename method (+ variables and etc.) to reflect that this method now also gets `costs` items
    def _get_revenues_items_from_invoices(self, excluded_move_line_ids=None, with_action=True):
        """
        Get all revenues items from invoices, and put them into their own
        "other_invoice_revenues" section.
        If the final total is 0 for either to_invoice or invoiced (ex: invoice -> credit note),
        we don't output a new section

        :param excluded_move_line_ids a list of 'account.move.line' to ignore
        when fetching the move lines, for example a list of invoices that were
        generated from a sales order
        """
        if excluded_move_line_ids is None:
            excluded_move_line_ids = []
        invoices_move_lines = self.env['account.move.line'].sudo().search_fetch(
            expression.AND([
                self._get_revenues_items_from_invoices_domain([('id', 'not in', excluded_move_line_ids)]),
                [('analytic_distribution', 'in', self.account_id.ids)]
            ]),
            ['price_subtotal', 'parent_state', 'currency_id', 'analytic_distribution', 'move_type', 'move_id', 'display_type']
        )
        res = {
            'revenues': {
                'data': [], 'total': {'invoiced': 0.0, 'to_invoice': 0.0}
            },
            'costs': {
                'data': [], 'total': {'billed': 0.0, 'to_bill': 0.0}
            },
        }
        # TODO: invoices_move_lines.with_context(prefetch_fields=False).move_id.move_type ??
        if invoices_move_lines:
            revenues_lines = []
            cogs_lines = []
            amount_invoiced = amount_to_invoice = 0.0
            for move_line in invoices_move_lines:
                if move_line['display_type'] == 'cogs':
                    cogs_lines.append(move_line)
                else:
                    revenues_lines.append(move_line)
            for move_lines, ml_type in ((revenues_lines, 'revenues'), (cogs_lines, 'costs')):
                for move_line in move_lines:
                    currency = move_line.currency_id
                    price_subtotal = currency._convert(move_line.price_subtotal, self.currency_id, self.company_id)
                    # an analytic account can appear several time in an analytic distribution with different repartition percentage
                    analytic_contribution = sum(
                        percentage for ids, percentage in move_line.analytic_distribution.items()
                        if str(self.account_id.id) in ids.split(',')
                    ) / 100.
                    if move_line.parent_state == 'draft':
                        if move_line.move_type == 'out_invoice':
                            amount_to_invoice += price_subtotal * analytic_contribution
                        else:  # move_line.move_type == 'out_refund'
                            amount_to_invoice -= price_subtotal * analytic_contribution
                    else:  # move_line.parent_state == 'posted'
                        if move_line.move_type == 'out_invoice':
                            amount_invoiced += price_subtotal * analytic_contribution
                        else:  # moves_read['move_type'] == 'out_refund'
                            amount_invoiced -= price_subtotal * analytic_contribution
                # don't display the section if the final values are both 0 (invoice -> credit note)
                if amount_invoiced != 0 or amount_to_invoice != 0:
                    section_id = 'other_invoice_revenues' if ml_type == 'revenues' else 'cost_of_goods_sold'
                    invoices_items = {
                        'id': section_id,
                        'sequence': self._get_profitability_sequence_per_invoice_type()[section_id],
                        'invoiced' if ml_type == 'revenues' else 'billed': amount_invoiced,
                        'to_invoice' if ml_type == 'revenues' else 'to_bill': amount_to_invoice,
                    }
                    if with_action and (
                        self.env.user.has_group('sales_team.group_sale_salesman_all_leads')
                        or self.env.user.has_group('account.group_account_invoice')
                        or self.env.user.has_group('account.group_account_readonly')
                    ):
                        invoices_items['action'] = self._get_action_for_profitability_section(invoices_move_lines.move_id.ids, section_id)
                    res[ml_type] = {
                        'data': [invoices_items],
                        'total': {
                            'invoiced' if ml_type == 'revenues' else 'billed': amount_invoiced,
                            'to_invoice' if ml_type == 'revenues' else 'to_bill': amount_to_invoice,
                        },
                    }
        return res

    def _add_invoice_items(self, domain, profitability_items, with_action=True):
        sale_lines = self.env['sale.order.line'].sudo()._read_group(
            self._get_profitability_sale_order_items_domain(domain),
            [],
            ['id:recordset'],
        )[0][0]
        revenue_items_from_invoices = self._get_revenues_items_from_invoices(
            excluded_move_line_ids=sale_lines.invoice_lines.ids,
            with_action=with_action
        )
        profitability_items['revenues']['data'] += revenue_items_from_invoices['revenues']['data']
        profitability_items['revenues']['total']['to_invoice'] += revenue_items_from_invoices['revenues']['total']['to_invoice']
        profitability_items['revenues']['total']['invoiced'] += revenue_items_from_invoices['revenues']['total']['invoiced']
        profitability_items['costs']['data'] += revenue_items_from_invoices['costs']['data']
        profitability_items['costs']['total']['to_bill'] += revenue_items_from_invoices['costs']['total']['to_bill']
        profitability_items['costs']['total']['billed'] += revenue_items_from_invoices['costs']['total']['billed']

    def _get_profitability_items(self, with_action=True):
        profitability_items = super()._get_profitability_items(with_action)
        sale_items = self.sudo()._get_sale_order_items()
        domain = [
            ('order_id', 'in', sale_items.order_id.ids),
            '|',
                '|',
                    ('project_id', 'in', self.ids),
                    ('project_id', '=', False),
                ('id', 'in', sale_items.ids),
        ]
        revenue_items_from_sol = self._get_revenues_items_from_sol(
            domain,
            with_action,
        )
        profitability_items['revenues']['data'] += revenue_items_from_sol['data']
        profitability_items['revenues']['total']['to_invoice'] += revenue_items_from_sol['total']['to_invoice']
        profitability_items['revenues']['total']['invoiced'] += revenue_items_from_sol['total']['invoiced']
        self._add_invoice_items(domain, profitability_items, with_action=with_action)
        self._add_purchase_items(profitability_items, with_action=with_action)
        return profitability_items

    def _get_stat_buttons(self):
        buttons = super()._get_stat_buttons()
        if self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'dollar',
                'text': self.env._('Sales Orders'),
                'number': self_sudo.sale_order_count,
                'action_type': 'object',
                'action': 'action_view_sos',
                'additional_context': json.dumps({
                    'create_for_project_id': self.id,
                }),
                'show': self_sudo.display_sales_stat_buttons and self_sudo.sale_order_count > 0,
                'sequence': 27,
            })
        if self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
            buttons.append({
                'icon': 'dollar',
                'text': self.env._('Sales Order Items'),
                'number': self.sale_order_line_count,
                'action_type': 'object',
                'action': 'action_view_sols',
                'show': self.display_sales_stat_buttons,
                'sequence': 28,
            })
        if self.env.user.has_group('account.group_account_readonly'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'pencil-square-o',
                'text': self.env._('Invoices'),
                'number': self_sudo.invoice_count,
                'action_type': 'object',
                'action': 'action_open_project_invoices',
                'show': bool(self.account_id) and self_sudo.invoice_count > 0,
                'sequence': 30,
            })
        if self.env.user.has_group('account.group_account_readonly'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'pencil-square-o',
                'text': self.env._('Vendor Bills'),
                'number': self_sudo.vendor_bill_count,
                'action_type': 'object',
                'action': 'action_open_project_vendor_bills',
                'show': self_sudo.vendor_bill_count > 0,
                'sequence': 38,
            })
        return buttons

    # ---------------------------------------------------
    # Actions
    # ---------------------------------------------------

    def _get_hide_partner(self):
        return not self.allow_billable

    def _get_projects_to_make_billable_domain(self):
        return expression.AND([
            super()._get_projects_to_make_billable_domain(),
            [('allow_billable', '=', False)],
        ])

    def action_view_tasks(self):
        if self.env.context.get('generate_milestone'):
            line_id = self.env.context.get('default_sale_line_id')
            default_line = self.env['sale.order.line'].browse(line_id)
            milestone = self.env['project.milestone'].create({
                'name': default_line.name,
                'project_id': self.id,
                'sale_line_id': line_id,
                'quantity_percentage': 1,
            })
            if default_line.product_id.service_tracking == 'task_in_project':
                default_line.task_id.milestone_id = milestone.id

        action = super().action_view_tasks()
        action['context']['hide_partner'] = self._get_hide_partner()
        return action

    def action_open_project_vendor_bills(self):
        move_lines = self.env['account.move.line'].search_fetch(
            [
                ('move_id.move_type', 'in', ['in_invoice', 'in_refund']),
                ('analytic_distribution', 'in', self.account_id.ids),
            ],
            ['move_id'],
        )
        vendor_bill_ids = move_lines.move_id.ids
        action_window = {
            'name': _('Vendor Bills'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'list'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', vendor_bill_ids)],
            'context': {
                'default_move_type': 'in_invoice',
                'project_id': self.id,
            },
            'help': "<p class='o_view_nocontent_smiling_face'>%s</p><p>%s</p>" % (
                _("Create a vendor bill"),
                _("Create invoices, register payments and keep track of the discussions with your vendors."),
            ),
        }
        if not self.env.context.get('from_embedded_action') and len(vendor_bill_ids) == 1:
            action_window['views'] = [[False, 'form']]
            action_window['res_id'] = vendor_bill_ids[0]
        return action_window

```

## File: models\project_task.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import SQL
from odoo.tools.misc import unquote


class ProjectTask(models.Model):
    _inherit = "project.task"

    def _domain_sale_line_id(self):
        domain = expression.AND([
            self.env['sale.order.line']._sellable_lines_domain(),
            self.env['sale.order.line']._domain_sale_line_service(),
            [
                '|',
                ('order_partner_id.commercial_partner_id.id', 'parent_of', unquote('partner_id if partner_id else []')),
                ('order_partner_id', '=?', unquote('partner_id')),
            ],
        ])
        return domain

    sale_order_id = fields.Many2one('sale.order', 'Sales Order', compute='_compute_sale_order_id', store=True, help="Sales order to which the task is linked.", group_expand="_group_expand_sales_order")
    sale_line_id = fields.Many2one(
        'sale.order.line', 'Sales Order Item',
        copy=True, tracking=True, index='btree_not_null', recursive=True,
        compute='_compute_sale_line', store=True, readonly=False,
        domain=lambda self: str(self._domain_sale_line_id()),
        help="Sales Order Item to which the time spent on this task will be added in order to be invoiced to your customer.\n"
             "By default the sales order item set on the project will be selected. In the absence of one, the last prepaid sales order item that has time remaining will be used.\n"
             "Remove the sales order item in order to make this task non billable. You can also change or remove the sales order item of each timesheet entry individually.")
    project_sale_order_id = fields.Many2one('sale.order', string="Project's sale order", related='project_id.sale_order_id')
    sale_order_state = fields.Selection(related='sale_order_id.state')
    task_to_invoice = fields.Boolean("To invoice", compute='_compute_task_to_invoice', search='_search_task_to_invoice', groups='sales_team.group_sale_salesman_all_leads')
    allow_billable = fields.Boolean(related="project_id.allow_billable")
    partner_id = fields.Many2one(inverse='_inverse_partner_id')

    # Project sharing  fields
    display_sale_order_button = fields.Boolean(string='Display Sales Order', compute='_compute_display_sale_order_button')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | {'allow_billable', 'sale_order_id', 'sale_line_id', 'display_sale_order_button'}

    @api.model
    def _group_expand_sales_order(self, sales_orders, domain):
        start_date = self._context.get('gantt_start_date')
        scale = self._context.get('gantt_scale')
        if not (start_date and scale):
            return sales_orders
        search_on_comodel = self._search_on_comodel(domain, "sale_order_id", "sale.order")
        if search_on_comodel:
            return search_on_comodel
        return sales_orders

    @api.depends('sale_line_id', 'project_id', 'allow_billable')
    def _compute_sale_order_id(self):
        for task in self:
            if not task.allow_billable:
                task.sale_order_id = False
                continue
            sale_order = (
                task.sale_line_id.order_id
                or task.project_id.sale_order_id
                or task.sale_order_id
            )
            if sale_order and not task.partner_id:
                task.partner_id = sale_order.partner_id
            consistent_partners = (
                sale_order.partner_id
                | sale_order.partner_invoice_id
                | sale_order.partner_shipping_id
            ).commercial_partner_id
            if task.partner_id.commercial_partner_id in consistent_partners:
                task.sale_order_id = sale_order
            else:
                task.sale_order_id = False

    @api.depends('allow_billable')
    def _compute_partner_id(self):
        billable_task = self.filtered(lambda t: t.allow_billable or (not self._origin and t.parent_id.allow_billable))
        (self - billable_task).partner_id = False
        super(ProjectTask, billable_task)._compute_partner_id()

    def _inverse_partner_id(self):
        for task in self:
            # check that sale_line_id/sale_order_id and customer are consistent
            consistent_partners = (
                task.sale_order_id.partner_id
                | task.sale_order_id.partner_invoice_id
                | task.sale_order_id.partner_shipping_id
            ).commercial_partner_id
            if task.sale_order_id and task.partner_id.commercial_partner_id not in consistent_partners:
                task.sale_order_id = task.sale_line_id = False

    @api.depends('sale_line_id.order_partner_id', 'parent_id.sale_line_id', 'project_id.sale_line_id', 'milestone_id.sale_line_id', 'allow_billable')
    def _compute_sale_line(self):
        for task in self:
            if not (task.allow_billable or task.parent_id.allow_billable):
                task.sale_line_id = False
                continue
            if not task.sale_line_id:
                # if the project_id is set then it means the task is classic task or a subtask with another project than its parent.
                # To determine the sale_line_id, we first need to look at the parent before the project to manage the case of subtasks.
                # Two sub-tasks in the same project do not necessarily have the same sale_line_id (need to look at the parent task).
                sale_line = False
                if task.parent_id.sale_line_id and task.parent_id.partner_id.commercial_partner_id == task.partner_id.commercial_partner_id:
                    sale_line = task.parent_id.sale_line_id
                elif task.project_id.sale_line_id and task.project_id.partner_id.commercial_partner_id == task.partner_id.commercial_partner_id:
                    sale_line = task.project_id.sale_line_id
                task.sale_line_id = sale_line or task.milestone_id.sale_line_id

    @api.depends('sale_order_id')
    def _compute_display_sale_order_button(self):
        if not self.sale_order_id:
            self.display_sale_order_button = False
            return
        try:
            sale_orders = self.env['sale.order'].search([('id', 'in', self.sale_order_id.ids)])
            for task in self:
                task.display_sale_order_button = task.sale_order_id in sale_orders
        except AccessError:
            self.display_sale_order_button = False

    @api.constrains('sale_line_id')
    def _check_sale_line_type(self):
        for task in self.sudo():
            if task.sale_line_id:
                if not task.sale_line_id.is_service or task.sale_line_id.is_expense:
                    raise ValidationError(_(
                        'You cannot link the order item %(order_id)s - %(product_id)s to this task because it is a re-invoiced expense.',
                        order_id=task.sale_line_id.order_id.name,
                        product_id=task.sale_line_id.product_id.display_name,
                    ))

    def _ensure_sale_order_linked(self, sol_ids):
        """ Orders created from project/task are supposed to be confirmed to match the typical flow from sales, but since
        we allow SO creation from the project/task itself we want to confirm newly created SOs immediately after creation.
        However this would leads to SOs being confirmed without a single product, so we'd rather do it on record save.
        """
        quotations = self.env['sale.order.line'].sudo()._read_group(
            domain=[('state', '=', 'draft'), ('id', 'in', sol_ids)],
            aggregates=['order_id:recordset'],
        )[0][0]
        if quotations:
            quotations.action_confirm()

    @api.model_create_multi
    def create(self, vals_list):
        tasks = super().create(vals_list)
        sol_ids = {
            vals['sale_line_id']
            for vals in vals_list
            if vals.get('sale_line_id')
        }
        if sol_ids:
            tasks._ensure_sale_order_linked(list(sol_ids))
        return tasks

    def write(self, vals):
        task = super().write(vals)
        if sol_id := vals.get('sale_line_id'):
            self._ensure_sale_order_linked([sol_id])
        return task

    # ---------------------------------------------------
    # Actions
    # ---------------------------------------------------

    def _get_action_view_so_ids(self):
        return self.sale_order_id.ids

    def action_view_so(self):
        so_ids = self._get_action_view_so_ids()
        action_window = {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            "name": _("Sales Order"),
            "views": [[False, "list"], [False, "kanban"], [False, "form"]],
            "context": {"create": False, "show_sale": True},
            "domain": [["id", "in", so_ids]],
        }
        if len(so_ids) == 1:
            action_window["views"] = [[False, "form"]]
            action_window["res_id"] = so_ids[0]

        return action_window

    def action_project_sharing_view_so(self):
        self.ensure_one()
        if not self.display_sale_order_button:
            return {}
        return {
            "name": "Portal Sale Order",
            "type": "ir.actions.act_url",
            "url": self.sale_order_id.access_url,
        }

    def _rating_get_partner(self):
        partner = self.partner_id or self.sale_line_id.order_id.partner_id
        return partner or super()._rating_get_partner()

    @api.depends('sale_order_id.invoice_status', 'sale_order_id.order_line')
    def _compute_task_to_invoice(self):
        for task in self:
            if task.sale_order_id:
                task.task_to_invoice = bool(task.sale_order_id.invoice_status not in ('no', 'invoiced'))
            else:
                task.task_to_invoice = False

    @api.model
    def _search_task_to_invoice(self, operator, value):
        sql = SQL("""(
            SELECT so.id
            FROM sale_order so
            WHERE so.invoice_status != 'invoiced'
                AND so.invoice_status != 'no'
        )""")
        operator_new = 'in'
        if (bool(operator == '=') ^ bool(value)):
            operator_new = 'not in'
        return [('sale_order_id', operator_new, sql)]

    @api.onchange('sale_line_id')
    def _onchange_partner_id(self):
        if not self.partner_id and self.sale_line_id:
            self.partner_id = self.sale_line_id.order_partner_id

    def _get_projects_to_make_billable_domain(self, additional_domain=None):
        return expression.AND([
            super()._get_projects_to_make_billable_domain(additional_domain),
            [
                ('partner_id', '!=', False),
                ('allow_billable', '=', False),
                ('project_id', '!=', False),
            ],
        ])

```

## File: models\project_task_recurrence.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProjectTaskRecurrence(models.Model):
    _inherit = 'project.task.recurrence'

    @api.model
    def _get_recurring_fields_to_copy(self):
        return super()._get_recurring_fields_to_copy() + ['sale_line_id']

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def set_values(self):
        super().set_values()
        if self.group_project_milestone:
            # Search the milestones containing a SOL and change the qty_delivered_method field of the SOL and the
            # service_policy field set on the product to convert from manual to milestones.
            milestones = self.env['project.milestone'].search_fetch([('sale_line_id', '!=', False)], ['sale_line_id'])
            sale_lines = milestones.sale_line_id.sudo()
            sale_lines.product_id.service_policy = 'delivered_milestones'
        else:
            product_domain = [('type', '=', 'service'), ('service_type', '=', 'milestones')]
            products = self.env['product.product'].search(product_domain)
            products.service_policy = 'delivered_manual'
            self.env['sale.order.line'].sudo().search([('product_id', 'in', products.ids)]).qty_delivered_method = 'manual'

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
from collections import defaultdict

from odoo import api, fields, models, _, Command
from odoo.exceptions import UserError
from odoo.osv.expression import AND, NEGATIVE_TERM_OPERATORS, TERM_OPERATORS_NEGATION
from odoo.addons.project.models.project_task import CLOSED_STATES


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    tasks_ids = fields.Many2many('project.task', compute='_compute_tasks_ids', search='_search_tasks_ids', string='Tasks associated with this sale', export_string_translation=False)
    tasks_count = fields.Integer(string='Tasks', compute='_compute_tasks_ids', groups="project.group_project_user", export_string_translation=False)

    visible_project = fields.Boolean('Display project', compute='_compute_visible_project', readonly=True, export_string_translation=False)
    project_ids = fields.Many2many('project.project', compute="_compute_project_ids", string='Projects', copy=False, groups="project.group_project_user,project.group_project_milestone", export_string_translation=False)
    project_count = fields.Integer(string='Number of Projects', compute='_compute_project_ids', groups='project.group_project_user', export_string_translation=False)
    milestone_count = fields.Integer(compute='_compute_milestone_count', export_string_translation=False)
    is_product_milestone = fields.Boolean(compute='_compute_is_product_milestone', export_string_translation=False)
    show_create_project_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user', export_string_translation=False)
    show_project_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user', export_string_translation=False)
    show_task_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user', export_string_translation=False)
    closed_task_count = fields.Integer(compute='_compute_tasks_ids', export_string_translation=False)
    completed_task_percentage = fields.Float(compute="_compute_completed_task_percentage", export_string_translation=False)
    project_id = fields.Many2one('project.project', domain=[('allow_billable', '=', True)], copy=False, help="A task will be created for the project upon sales order confirmation. The analytic distribution of this project will also serve as a reference for newly created sales order items.")
    project_account_id = fields.Many2one('account.analytic.account', related='project_id.account_id')

    def _compute_milestone_count(self):
        read_group = self.env['project.milestone']._read_group(
            [('sale_line_id', 'in', self.order_line.ids)],
            ['sale_line_id'],
            ['__count'],
        )
        line_data = {sale_line.id: count for sale_line, count in read_group}
        for order in self:
            order.milestone_count = sum(line_data.get(line.id, 0) for line in order.order_line)

    def _compute_is_product_milestone(self):
        for order in self:
            order.is_product_milestone = order.order_line.product_id.filtered(lambda p: p.service_policy == 'delivered_milestones')

    def _compute_show_project_and_task_button(self):
        is_project_manager = self.env.user.has_group('project.group_project_manager')
        show_button_ids = self.env['sale.order.line']._read_group([
            ('order_id', 'in', self.ids),
            ('order_id.state', 'not in', ['draft', 'sent']),
            ('product_id.type', '=', 'service'),
        ], aggregates=['order_id:array_agg'])[0][0]
        for order in self:
            order.show_project_button = order.id in show_button_ids and order.project_count
            order.show_task_button = order.show_project_button or order.tasks_count
            order.show_create_project_button = (
                is_project_manager
                and order.id in show_button_ids
                and not order.project_count
            )

    @api.model
    def _search_tasks_ids(self, operator, value):
        if operator in NEGATIVE_TERM_OPERATORS:
            positive_operator = TERM_OPERATORS_NEGATION[operator]
        else:
            positive_operator = operator
        task_domain = [('display_name' if isinstance(value, str) else 'id', positive_operator, value), ('sale_order_id', '!=', False)]
        query = self.env['project.task']._search(task_domain)
        return [('id', 'in' if positive_operator == operator else 'not in', query.subselect('sale_order_id'))]

    @api.depends('order_line.product_id.project_id')
    def _compute_tasks_ids(self):
        tasks_per_so = self.env['project.task']._read_group(
            domain=self._tasks_ids_domain(),
            groupby=['sale_order_id', 'state'],
            aggregates=['id:recordset', '__count']
        )
        so_with_tasks = self.env['sale.order']
        for order, state, tasks_ids, tasks_count in tasks_per_so:
            if order:
                order.tasks_ids += tasks_ids
                order.tasks_count += tasks_count
                order.closed_task_count += state in CLOSED_STATES and tasks_count
                so_with_tasks += order
            else:
                # tasks that have no sale_order_id need to be associated with the SO from their sale_line_id
                for task in tasks_ids:
                    task_so = task.sale_line_id.order_id
                    task_so.tasks_ids = [Command.link(task.id)]
                    task_so.tasks_count += 1
                    task_so.closed_task_count += state in CLOSED_STATES
                    so_with_tasks += task_so
        remaining_orders = self - so_with_tasks
        if remaining_orders:
            remaining_orders.tasks_ids = [Command.clear()]
            remaining_orders.tasks_count = 0
            remaining_orders.closed_task_count = 0

    @api.depends('order_line.product_id.service_tracking')
    def _compute_visible_project(self):
        """ Users should be able to select a project_id on the SO if at least one SO line has a product with its service tracking
        configured as 'task_in_project' """
        for order in self:
            order.visible_project = any(
                service_tracking == 'task_in_project' for service_tracking in order.order_line.mapped('product_id.service_tracking')
            )

    @api.depends('order_line.product_id', 'order_line.project_id')
    def _compute_project_ids(self):
        is_project_manager = self.env.user.has_group('project.group_project_manager')
        projects = self.env['project.project'].search([('sale_order_id', 'in', self.ids)])
        projects_per_so = defaultdict(lambda: self.env['project.project'])
        for project in projects:
            projects_per_so[project.sale_order_id.id] |= project
        for order in self:
            projects = order.order_line.mapped('product_id.project_id')
            projects |= order.order_line.mapped('project_id')
            projects |= projects_per_so[order.id or order._origin.id]
            if not is_project_manager:
                projects = projects._filtered_access('read')
            order.project_ids = projects
            order.project_count = len(projects)

    def _action_confirm(self):
        """ On SO confirmation, some lines should generate a task or a project. """
        if len(self.company_id) == 1:
            # All orders are in the same company
            self.order_line.sudo().with_company(self.company_id)._timesheet_service_generation()
        else:
            # Orders from different companies are confirmed together
            for order in self:
                order.order_line.sudo().with_company(order.company_id)._timesheet_service_generation()
        return super()._action_confirm()

    def action_view_task(self):
        self.ensure_one()
        if not self.order_line:
            return {'type': 'ir.actions.act_window_close'}

        list_view_id = self.env.ref('project.view_task_tree2').id
        form_view_id = self.env.ref('project.view_task_form2').id
        kanban_view_id = self.env.ref('project.view_task_kanban_inherit_view_default_project').id

        project_ids = self.tasks_ids.project_id
        if len(project_ids) > 1:
            action = self.env['ir.actions.actions']._for_xml_id('project.action_view_task')
            action['domain'] = AND([ast.literal_eval(action['domain']), self._tasks_ids_domain()])
            action['context'] = {}
        else:
            # Load top bar if all the tasks linked to the SO belong to the same project
            action = self.env['ir.actions.actions'].with_context({'active_id': project_ids.id})._for_xml_id('project.act_project_project_2_project_task_all')
            action['context'] = {
                'active_id': project_ids.id,
                'search_default_sale_order_id': self.id,
            }

        if self.tasks_count > 1:  # cross project kanban task
            for idx, (view_id, view_type) in enumerate(action['views']):
                if view_type == 'kanban':
                    action['views'][idx] = (kanban_view_id, 'kanban')
                elif view_type == 'list':
                    action['views'][idx] = (list_view_id, 'list')
                elif view_type == 'form':
                    action['views'][idx] = (form_view_id, 'form')
        else:  # 1 or 0 tasks -> form view
            action['views'] = [(form_view_id, 'form')]
            action['res_id'] = self.tasks_ids.id
        # set default project
        default_line = next((sol for sol in self.order_line if sol.product_id.type == 'service'), self.env['sale.order.line'])
        default_project_id = default_line.project_id.id or self.project_ids[:1].id or self.tasks_ids.project_id[:1].id

        action['context'].update({
            'default_sale_order_id': self.id,
            'default_sale_line_id': default_line.id,
            'default_partner_id': self.partner_id.id,
            'default_project_id': default_project_id,
            'default_user_ids': [self.env.uid],
        })
        return action

    def _tasks_ids_domain(self):
        return ['&', ('project_id', '!=', False), '|', ('sale_line_id', 'in', self.order_line.ids), ('sale_order_id', 'in', self.ids)]

    def action_create_project(self):
        self.ensure_one()
        if not self.show_create_project_button:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'type': 'danger',
                    'message': _("The project couldn't be created as the Sales Order must be confirmed, is already linked to a project, or doesn't involve any services."),
                }
            }

        sorted_line = self.order_line.sorted('sequence')
        default_sale_line = next((
            sol for sol in sorted_line
            if sol.product_id.type == 'service' and not sol.is_downpayment
        ), self.env['sale.order.line'])
        return {
            **self.env["ir.actions.actions"]._for_xml_id("project.open_create_project"),
            'context': {
                'default_sale_order_id': self.id,
                'default_sale_line_id': default_sale_line.id,
                'default_partner_id': self.partner_id.id,
                'default_user_ids': [self.env.uid],
                'default_allow_billable': 1,
                'hide_allow_billable': True,
                'default_company_id': self.company_id.id,
                'generate_milestone': default_sale_line.product_id.service_policy == 'delivered_milestones',
            },
        }

    def action_view_project_ids(self):
        self.ensure_one()
        if not self.order_line:
            return {'type': 'ir.actions.act_window_close'}

        sorted_line = self.order_line.sorted('sequence')
        default_sale_line = next((
            sol for sol in sorted_line if sol.product_id.type == 'service'
        ), self.env['sale.order.line'])
        action = {
            'type': 'ir.actions.act_window',
            'name': _('Projects'),
            'domain': ['|', ('sale_order_id', '=', self.id), ('id', 'in', self.with_context(active_test=False).project_ids.ids), ('active', 'in', [True, False])],
            'res_model': 'project.project',
            'views': [(False, 'kanban'), (False, 'list'), (False, 'form')],
            'view_mode': 'kanban,list,form',
            'context': {
                **self._context,
                'default_partner_id': self.partner_id.id,
                'default_sale_line_id': default_sale_line.id,
                'default_allow_billable': 1,
            }
        }
        if len(self.with_context(active_test=False).project_ids) == 1:
            action.update({'views': [(False, 'form')], 'res_id': self.project_ids.id})
        return action

    def action_view_milestone(self):
        self.ensure_one()
        default_project = self.project_ids and self.project_ids[0]
        sorted_line = self.order_line.sorted('sequence')
        default_sale_line = next((
            sol for sol in sorted_line
                if sol.is_service and sol.product_id.service_policy == 'delivered_milestones'
        ), self.env['sale.order.line'])
        return {
            'type': 'ir.actions.act_window',
            'name': _('Milestones'),
            'domain': [('sale_line_id', 'in', self.order_line.ids)],
            'res_model': 'project.milestone',
            'views': [(self.env.ref('sale_project.sale_project_milestone_view_tree').id, 'list')],
            'view_mode': 'list',
            'help': _("""
                <p class="o_view_nocontent_smiling_face">
                    No milestones found. Let's create one!
                </p><p>
                    Track major progress points that must be reached to achieve success.
                </p>
            """),
            'context': {
                **self.env.context,
                'default_project_id': default_project.id,
                'default_sale_line_id': default_sale_line.id,
            }
        }

    @api.model_create_multi
    def create(self, vals_list):
        created_records = super().create(vals_list)
        project = self.env['project.project'].browse(self.env.context.get('create_for_project_id'))
        if project:
            service_sol = next((sol for sol in created_records.order_line if sol.is_service), False)
            if not service_sol and not self.env.context.get('from_embedded_action'):
                raise UserError(_('This Sales Order must contain at least one product of type "Service".'))
            if not project.sale_line_id:
                project.sale_line_id = service_sol
        return created_records

    def write(self, values):
        res = super().write(values)
        if 'state' in values and values['state'] == 'cancel':
            # Remove sale line field reference from all projects
            self.env['project.project'].sudo().search([('sale_line_id.order_id', 'in', self.ids)]).sale_line_id = False
        return res

    def _compute_completed_task_percentage(self):
        for so in self:
            so.completed_task_percentage = so.tasks_count and so.closed_task_count / so.tasks_count

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, Command, fields, models, _
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools import format_list
from odoo.tools.sql import column_exists, create_column


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    qty_delivered_method = fields.Selection(selection_add=[('milestones', 'Milestones')])
    project_id = fields.Many2one(
        'project.project', 'Generated Project',
        index=True, copy=False, export_string_translation=False)
    task_id = fields.Many2one(
        'project.task', 'Generated Task',
        index=True, copy=False, export_string_translation=False)
    reached_milestones_ids = fields.One2many('project.milestone', 'sale_line_id', string='Reached Milestones', domain=[('is_reached', '=', True)], export_string_translation=False)

    def _get_product_from_sol_name_domain(self, product_name):
        return [
            ('name', 'ilike', product_name),
            ('type', '=', 'service'),
            ('company_id', 'in', [False, self.env.company.id]),
        ]

    def default_get(self, fields):
        res = super().default_get(fields)
        if self.env.context.get('form_view_ref') == 'sale_project.sale_order_line_view_form_editable':
            default_values = dict()
            # If we can't add order lines to the default order, discard it
            if 'order_id' in res:
                try:
                    self.env['sale.order'].browse(res['order_id']).check_access('write')
                except AccessError:
                    del res['order_id']

            if 'order_id' in fields and not res.get('order_id'):
                assert (partner_id := self.env.context.get('default_partner_id'))
                project_id = self.env.context.get('link_to_project')
                sale_order = None
                so_create_values = {
                    'partner_id': partner_id,
                    'company_id': self.env.context.get('default_company_id') or self.env.company.id,
                }
                if project_id:
                    try:
                        project_so = self.env['project.project'].browse(project_id).sale_order_id
                        project_so.check_access('write')
                        sale_order = project_so
                    except AccessError:
                        pass
                    if not sale_order:
                        so_create_values['project_ids'] = [Command.link(project_id)]

                if not sale_order:
                    sale_order = self.env['sale.order'].create(so_create_values)
                default_values['order_id'] = sale_order.id
            if product_name := self.env.context.get('sol_product_name') or self.env.context.get('default_name'):
                product = self.env['product.product'].search(self._get_product_from_sol_name_domain(product_name), limit=1)
                if product:
                    default_values['product_id'] = product.id
                    # We need to remove the name from the defaults so that the
                    # name of the SOL is based on the full name of the product
                    # and not overwritten by what was typed in the field.
                    if "name" in res:
                        del res["name"]
            else:
                default_values['name'] = _("New Sales Order Item")
            return {**res, **default_values}
        return res

    @api.model
    def name_create(self, name):
        ensure_is_service_product = False
        # To get the right product when creating a SOL on the fly, we need to get
        # the name that was entered in the field from the `default_get` method.
        # The easiest way of doing that is to store it in the context.
        if self.env.context.get('form_view_ref') == 'sale_project.sale_order_line_view_form_editable' and not self.env.context.get('action_view_sols'):
            self = self.with_context(sol_product_name=name)
            ensure_is_service_product = True
        result = super().name_create(name)
        if ensure_is_service_product and result and not self.browse(result[0]).is_service:
            raise ValidationError(_("The Sale Order Item should contain a service product."))
        return result

    @api.model
    def _add_missing_default_values(self, values):
        # When creating a SOL through the quick create, the name_create will be
        # called with whatever was typed in the field. However, we don't want
        # that value to overwrite the computed SOL name if we find a product.
        defaults = super()._add_missing_default_values(values)
        if self.env.context.get('form_view_ref') == 'sale_project.sale_order_line_view_form_editable' and not self.env.context.get('action_view_sols'):
            if "name" in defaults and "product_id" in defaults:
                del defaults["name"]
        return defaults

    @api.depends('product_id.type')
    def _compute_product_updatable(self):
        super()._compute_product_updatable()
        for line in self:
            if line.product_id.type == 'service' and line.state == 'sale':
                line.product_updatable = False

    @api.depends('product_id')
    def _compute_qty_delivered_method(self):
        milestones_lines = self.filtered(lambda sol:
            not sol.is_expense
            and sol.product_id.type == 'service'
            and sol.product_id.service_type == 'milestones'
        )
        milestones_lines.qty_delivered_method = 'milestones'
        super(SaleOrderLine, self - milestones_lines)._compute_qty_delivered_method()

    @api.depends('qty_delivered_method', 'product_uom_qty', 'reached_milestones_ids.quantity_percentage')
    def _compute_qty_delivered(self):
        lines_by_milestones = self.filtered(lambda sol: sol.qty_delivered_method == 'milestones')
        super(SaleOrderLine, self - lines_by_milestones)._compute_qty_delivered()

        if not lines_by_milestones:
            return

        project_milestone_read_group = self.env['project.milestone']._read_group(
            [('sale_line_id', 'in', lines_by_milestones.ids), ('is_reached', '=', True)],
            ['sale_line_id'],
            ['quantity_percentage:sum'],
        )
        reached_milestones_per_sol = {sale_line.id: percentage_sum for sale_line, percentage_sum in project_milestone_read_group}
        for line in lines_by_milestones:
            sol_id = line.id or line._origin.id
            line.qty_delivered = reached_milestones_per_sol.get(sol_id, 0.0) * line.product_uom_qty

    @api.depends('order_id.partner_id', 'product_id', 'order_id.project_id')
    def _compute_analytic_distribution(self):
        super()._compute_analytic_distribution()
        for line in self:
            if line.display_type or line.analytic_distribution or not line.product_id:
                continue
            project = line.product_id.project_id or line.order_id.project_id
            distribution = project._get_analytic_distribution()
            if distribution:
                line.analytic_distribution = distribution

    @api.model_create_multi
    def create(self, vals_list):
        lines = super().create(vals_list)
        # Do not generate task/project when expense SO line, but allow
        # generate task with hours=0.
        confirmed_lines = lines.filtered(lambda sol: sol.state == 'sale' and not sol.is_expense)
        # We track the lines that already generated a task, so we know we won't have to post a message for them after calling the generation service
        has_task_lines = confirmed_lines.filtered('task_id')
        confirmed_lines.sudo()._timesheet_service_generation()
        # if the SO line created a task, post a message on the order
        for line in confirmed_lines - has_task_lines:
            if line.task_id:
                msg_body = _("Task Created (%(name)s): %(link)s", name=line.product_id.name, link=line.task_id._get_html_link())
                line.order_id.message_post(body=msg_body)

        # Set a service SOL on the project, if any is given
        if project_id := self.env.context.get('link_to_project'):
            assert (service_line := next((line for line in lines if line.is_service), False))
            project = self.env['project.project'].browse(project_id)
            if not project.sale_line_id:
                project.sale_line_id = service_line
        return lines

    def write(self, values):
        result = super().write(values)
        # changing the ordered quantity should change the allocated hours on the
        # task, whatever the SO state. It will be blocked by the super in case
        # of a locked sale order.
        if 'product_uom_qty' in values and not self.env.context.get('no_update_allocated_hours', False):
            for line in self:
                if line.task_id and line.product_id.type == 'service':
                    allocated_hours = line._convert_qty_company_hours(line.task_id.company_id or self.env.user.company_id)
                    line.task_id.write({'allocated_hours': allocated_hours})
        return result

    def copy_data(self, default=None):
        data = super().copy_data(default)
        for origin, datum in zip(self, data):
            if origin.analytic_distribution == origin.order_id.project_id.sudo()._get_analytic_distribution():
                datum['analytic_distribution'] = False
        return data

    ###########################################
    # Service : Project and task generation
    ###########################################

    def _convert_qty_company_hours(self, dest_company):
        return self.product_uom_qty

    def _timesheet_create_project_prepare_values(self):
        """Generate project values"""
        # create the project or duplicate one
        return {
            'name': '%s - %s' % (self.order_id.client_order_ref, self.order_id.name) if self.order_id.client_order_ref else self.order_id.name,
            'partner_id': self.order_id.partner_id.id,
            'sale_line_id': self.id,
            'active': True,
            'company_id': self.company_id.id,
            'allow_billable': True,
            'user_id': self.product_id.project_template_id.user_id.id,
        }

    def _timesheet_create_project(self):
        """ Generate project for the given so line, and link it.
            :param project: record of project.project in which the task should be created
            :return task: record of the created task
        """
        self.ensure_one()
        values = self._timesheet_create_project_prepare_values()
        project_template = self.product_id.project_template_id
        if project_template:
            values['name'] = "%s - %s" % (values['name'], project_template.name)
            project = project_template.copy(values)
            project.tasks.write({
                'sale_line_id': self.id,
                'partner_id': self.order_id.partner_id.id,
            })
            # duplicating a project doesn't set the SO on sub-tasks
            project.tasks.filtered('parent_id').write({
                'sale_line_id': self.id,
                'sale_order_id': self.order_id.id,
            })
        else:
            project_only_sol_count = self.env['sale.order.line'].search_count([
                ('order_id', '=', self.order_id.id),
                ('product_id.service_tracking', 'in', ['project_only', 'task_in_project']),
            ])
            if project_only_sol_count == 1:
                values['name'] = "%s - [%s] %s" % (values['name'], self.product_id.default_code, self.product_id.name) if self.product_id.default_code else "%s - %s" % (values['name'], self.product_id.name)
            values.update(self._timesheet_create_project_account_vals(self.order_id.project_id))
            project = self.env['project.project'].create(values)

        # Avoid new tasks to go to 'Undefined Stage'
        if not project.type_ids:
            project.type_ids = self.env['project.task.type'].create([{
                'name': name,
                'fold': fold,
                'sequence': sequence,
            } for name, fold, sequence in [
                (_('To Do'), False, 5),
                (_('In Progress'), False, 10),
                (_('Done'), False, 15),
                (_('Cancelled'), True, 20),
            ]])

        # link project as generated by current so line
        self.write({'project_id': project.id})
        project.reinvoiced_sale_order_id = self.order_id
        return project

    def _timesheet_create_project_account_vals(self, project):
        return {
            fname: project[fname].id for fname in project._get_plan_fnames() if fname != 'account_id' and project[fname]
        }

    def _timesheet_create_task_prepare_values(self, project):
        self.ensure_one()
        allocated_hours = 0.0
        if self.product_id.service_type not in ['milestones', 'manual']:
            allocated_hours = self._convert_qty_company_hours(self.company_id)
        sale_line_name_parts = self.name.split('\n')
        products_inside_template_line_with_name = self.order_id.sale_order_template_id.sale_order_template_line_ids.filtered(
            lambda line: line.product_id and line.name).product_id
        if self.product_id in products_inside_template_line_with_name:
            title = self.product_id.name
            description = '<br/>'.join(sale_line_name_parts)
        else:
            title = sale_line_name_parts[0]
            description = '<br/>'.join(sale_line_name_parts[1:])

        return {
            'name': title if project.sale_line_id else '%s - %s' % (self.order_id.name or '', title),
            'allocated_hours': allocated_hours,
            'partner_id': self.order_id.partner_id.id,
            'description': description,
            'project_id': project.id,
            'sale_line_id': self.id,
            'sale_order_id': self.order_id.id,
            'company_id': project.company_id.id,
            'user_ids': False,  # force non assigned task, as created as sudo()
        }

    def _timesheet_create_task(self, project):
        """ Generate task for the given so line, and link it.
            :param project: record of project.project in which the task should be created
            :return task: record of the created task
        """
        values = self._timesheet_create_task_prepare_values(project)
        task = self.env['project.task'].sudo().create(values)
        self.task_id = task
        # post message on task
        task_msg = _("This task has been created from: %(order_link)s (%(product_name)s)",
            order_link=self.order_id._get_html_link(),
            product_name=self.product_id.name,
        )
        task.message_post(body=task_msg)
        return task

    def _get_so_lines_task_global_project(self):
        return self.filtered(lambda sol: sol.is_service and sol.product_id.service_tracking == 'task_global_project')

    def _get_so_lines_new_project(self):
        return self.filtered(lambda sol: sol.is_service and sol.product_id.service_tracking in ['project_only', 'task_in_project'])

    def _timesheet_service_generation(self):
        """ For service lines, create the task or the project. If already exists, it simply links
            the existing one to the line.
            Note: If the SO was confirmed, cancelled, set to draft then confirmed, avoid creating a
            new project/task. This explains the searches on 'sale_line_id' on project/task. This also
            implied if so line of generated task has been modified, we may regenerate it.
        """
        so_line_task_global_project = self._get_so_lines_task_global_project()
        so_line_new_project = self._get_so_lines_new_project()

        # search so lines from SO of current so lines having their project generated, in order to check if the current one can
        # create its own project, or reuse the one of its order.
        map_so_project = {}
        if so_line_new_project:
            order_ids = self.mapped('order_id').ids
            so_lines_with_project = self.search([('order_id', 'in', order_ids), ('project_id', '!=', False), ('product_id.service_tracking', 'in', ['project_only', 'task_in_project']), ('product_id.project_template_id', '=', False)])
            map_so_project = {sol.order_id.id: sol.project_id for sol in so_lines_with_project}
            so_lines_with_project_templates = self.search([('order_id', 'in', order_ids), ('project_id', '!=', False), ('product_id.service_tracking', 'in', ['project_only', 'task_in_project']), ('product_id.project_template_id', '!=', False)])
            map_so_project_templates = {(sol.order_id.id, sol.product_id.project_template_id.id): sol.project_id for sol in so_lines_with_project_templates}

        # search the global project of current SO lines, in which create their task
        map_sol_project = {}
        if so_line_task_global_project:
            map_sol_project = {sol.id: sol.product_id.with_company(sol.company_id).project_id for sol in so_line_task_global_project}

        def _can_create_project(sol):
            if not sol.project_id:
                if sol.product_id.project_template_id:
                    return (sol.order_id.id, sol.product_id.project_template_id.id) not in map_so_project_templates
                elif sol.order_id.id not in map_so_project:
                    return True
            return False

        # we store the reference analytic account per SO
        map_account_per_so = {}

        # project_only, task_in_project: create a new project, based or not on a template (1 per SO). May be create a task too.
        # if 'task_in_project' and project_id configured on SO, use that one instead
        for so_line in so_line_new_project.sorted(lambda sol: (sol.sequence, sol.id)):
            project = False
            if so_line.product_id.service_tracking in ['project_only', 'task_in_project']:
                project = so_line.project_id
            if not project and _can_create_project(so_line):
                project = so_line._timesheet_create_project()

                # If the SO generates projects on confirmation and the project's SO is not set, set it to the project's SOL with the lowest (sequence, id)
                if not so_line.order_id.project_id:
                    so_line.order_id.project_id = project
                # If no reference analytic account exists, set the account of the generated project to the account of the project's SO or create a new one
                account = map_account_per_so.get(so_line.order_id.id)
                if not account:
                    account = so_line.order_id.project_account_id or self.env['account.analytic.account'].create(so_line.order_id._prepare_analytic_account_data())
                    map_account_per_so[so_line.order_id.id] = account
                project.account_id = account

                if so_line.product_id.project_template_id:
                    map_so_project_templates[(so_line.order_id.id, so_line.product_id.project_template_id.id)] = project
                else:
                    map_so_project[so_line.order_id.id] = project
            elif not project:
                # Attach subsequent SO lines to the created project
                so_line.project_id = (
                    map_so_project_templates.get((so_line.order_id.id, so_line.product_id.project_template_id.id))
                    or map_so_project.get(so_line.order_id.id)
                )
            if so_line.product_id.service_tracking == 'task_in_project':
                if not project:
                    if so_line.product_id.project_template_id:
                        project = map_so_project_templates[(so_line.order_id.id, so_line.product_id.project_template_id.id)]
                    else:
                        project = map_so_project[so_line.order_id.id]
                if not so_line.task_id:
                    so_line._timesheet_create_task(project=project)
            so_line._handle_milestones(project)

        # task_global_project: if not set, set the project's SO by looking at global projects
        for so_line in so_line_task_global_project.sorted(lambda sol: (sol.sequence, sol.id)):
            if not so_line.order_id.project_id:
                so_line.order_id.project_id = map_sol_project.get(so_line.id)

        # task_global_project: create task in global projects
        for so_line in so_line_task_global_project:
            if not so_line.task_id:
                project = map_sol_project.get(so_line.id) or so_line.order_id.project_id
                if project and so_line.product_uom_qty > 0:
                    so_line._timesheet_create_task(project)
                elif not project:
                    raise UserError(_(
                        "A project must be defined on the quotation %(order)s or on the form of products creating a task on order.\n"
                        "The following product need a project in which to put its task: %(product_name)s",
                        order=so_line.order_id.name,
                        product_name=so_line.product_id.name,
                    ))

    def _handle_milestones(self, project):
        self.ensure_one()
        if self.product_id.service_policy != 'delivered_milestones':
            return
        if (milestones := project.milestone_ids.filtered(lambda milestone: not milestone.sale_line_id)):
            milestones.write({
                'sale_line_id': self.id,
                'product_uom_qty': self.product_uom_qty / len(milestones),
            })
        else:
            milestone = self.env['project.milestone'].create({
                'name': self.name,
                'project_id': self.project_id.id or self.order_id.project_id.id,
                'sale_line_id': self.id,
                'quantity_percentage': 1,
            })
            if self.product_id.service_tracking == 'task_in_project':
                self.task_id.milestone_id = milestone.id

    def _prepare_invoice_line(self, **optional_values):
        """
            If the sale order line isn't linked to a sale order which already have a default analytic account,
            this method allows to retrieve the analytic account which is linked to project or task directly linked
            to this sale order line, or the analytic account of the project which uses this sale order line, if it exists.
        """
        values = super(SaleOrderLine, self)._prepare_invoice_line(**optional_values)
        if not values.get('analytic_distribution'):
            if self.task_id.project_id.account_id:
                values['analytic_distribution'] = {self.task_id.project_id.account_id.id: 100}
            elif self.project_id.account_id:
                values['analytic_distribution'] = {self.project_id.account_id.id: 100}
            elif self.is_service and not self.is_expense:
                [accounts] = self.env['project.project']._read_group([
                    ('account_id', '!=', False),
                    '|',
                        ('sale_line_id', '=', self.id),
                        ('tasks.sale_line_id', '=', self.id),
                ], aggregates=['account_id:recordset'])[0]
                if len(accounts) == 1:
                    values['analytic_distribution'] = {accounts.id: 100}
        return values

    def _get_action_per_item(self):
        """ Get action per Sales Order Item

            :returns: Dict containing id of SOL as key and the action as value
        """
        return {}

    def _prepare_procurement_values(self, group_id=False):
        values = super()._prepare_procurement_values(group_id=group_id)
        if self.project_id:
            values['project_id'] = self.order_id.project_id.id
        return values

```

## File: models\sale_order_template_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class SaleOrderTemplateLine(models.Model):
    _inherit = 'sale.order.template.line'

    def _prepare_order_line_values(self):
        res = super()._prepare_order_line_values()
        # prevent the association of a related task on the SOL if a task would be generated when confirming the SO.
        if 'default_task_id' in self.env.context and \
                self.product_id.service_tracking in ['task_in_project', 'task_global_project']:
            res['task_id'] = False
        return res

```

## File: models\__init__.py

```python
from . import account_move
from . import account_move_line
from . import product_product
from . import product_template
from . import project_milestone
from . import project_project
from . import project_task_recurrence
from . import project_task
from . import res_config_settings
from . import sale_order_line
from . import sale_order_template_line
from . import sale_order

```

## File: report\project_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ReportProjectTaskUser(models.Model):
    _inherit = "report.project.task.user"

    sale_line_id = fields.Many2one('sale.order.line', string='Sales Order Item', readonly=True)
    sale_order_id = fields.Many2one('sale.order', string='Sales Order', readonly=True)

    def _select(self):
        return super()._select() + ", t.sale_line_id as sale_line_id, t.sale_order_id"

    def _group_by(self):
        return super()._group_by() + ", t.sale_line_id, t.sale_order_id"

```

## File: report\sale_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class SaleReport(models.Model):
    _inherit = 'sale.report'

    project_id = fields.Many2one(comodel_name='project.project', readonly=True)

    def _select_additional_fields(self):
        res = super()._select_additional_fields()
        res['project_id'] = 's.project_id'
        return res

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_report
from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sale_order_line_project_manager,sale.order.line.project.manager,sale.model_sale_order_line,project.group_project_manager,1,0,0,0
access_sale_order_project_manager,sale.order.project.manager,sale.model_sale_order,project.group_project_manager,1,0,0,0

```

## File: security\sale_project_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="sale_order_line_rule_project_manager" model="ir.rule">
        <field name="name">Project Manager Sales Orders Line</field>
        <field name="model_id" ref="sale.model_sale_order_line"/>
        <field name="domain_force">[('state', '=', 'sale'), ('is_service', '=', True), '|', ('project_id','!=', False), ('task_id','!=', False)]</field>
        <field name="groups" eval="[(4, ref('project.group_project_manager'))]"/>
        <field name="perm_create" eval="0"/>
        <field name="perm_write" eval="0"/>
        <field name="perm_unlink" eval="0"/>
        <field name="perm_read" eval="1"/>
    </record>

</odoo>

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.js

```javascript
import { patch } from "@web/core/utils/patch";
import { ProjectRightSidePanel } from '@project/components/project_right_side_panel/project_right_side_panel';

patch(ProjectRightSidePanel.prototype, {

    get panelVisible() {
        return super.panelVisible || this.state.data.show_sale_items;
    },
});

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="sale_project.ProjectRightSidePanel" t-inherit="project.ProjectRightSidePanel" t-inherit-mode="extension">
        <xpath expr="//ProjectProfitability" position="replace">
            <ProjectProfitability
                t-if="showProjectProfitability"
                data="state.data.profitability_items"
                labels="state.data.profitability_labels"
                formatMonetary="formatMonetary.bind(this)"
                onProjectActionClick="onProjectActionClick.bind(this)"
                onClick="(params) => this.onProjectActionClick(params)"
                projectId="projectId"
                context="context"
            />
        </xpath>
    </t>
</templates>

```

## File: static\src\components\project_right_side_panel\components\project_milestone.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-inherit="project.ProjectMilestone" t-inherit-mode="extension">
        <xpath expr="//t[@t-esc='milestone.name']" position="replace">
            <span>
                <t t-esc="milestone.name"/>
                <span t-if="milestone.allow_billable &amp;&amp; milestone.quantity_percentage &amp;&amp; !milestone.sale_line_display_name" class="fst-italic text-muted">
                    (<t t-esc="(100 * milestone.quantity_percentage).toFixed(2)"/>%)
                </span>
            </span>
            <span t-if="milestone.allow_billable" t-attf-class="fst-italic {{state.colorClass || 'text-muted'}} ms-2">
                <t t-if="milestone.sale_line_display_name" t-esc="milestone.sale_line_display_name"/>
                <span t-if="milestone.quantity_percentage &amp;&amp; milestone.sale_line_display_name">
                    (<t t-esc="(100 * milestone.quantity_percentage).toFixed(2)"/>%)
                </span>
           </span>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\project_right_side_panel\components\project_profitability.js

```javascript
import { patch } from "@web/core/utils/patch";
import { ProjectProfitability } from "@project/components/project_right_side_panel/components/project_profitability";
import { ProjectProfitabilitySection } from "@sale_project/components/project_right_side_panel/components/project_profitability_section";

patch(ProjectProfitability, {
    props: {
        ...ProjectProfitability.props,
        projectId: Number,
        context: Object,
    },

    components: {
        ...ProjectProfitability.components,
        ProjectProfitabilitySection,
    },
});

```

## File: static\src\components\project_right_side_panel\components\project_profitability.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="sale_project.ProjectProfitability" t-inherit="project.ProjectProfitability" t-inherit-mode="extension">
       <xpath expr="//tr[hasclass('revenue_section')]" position="replace">
            <ProjectProfitabilitySection
                revenue="revenue"
                labels="props.labels"
                formatMonetary="props.formatMonetary.bind(this)"
                onProjectActionClick="props.onProjectActionClick.bind(this)"
                onClick="(params) => this.props.onProjectActionClick(params)"
                projectId="this.props.projectId"
                context="this.props.context"
            />
        </xpath>
    </t>
</templates>

```

## File: static\src\components\project_right_side_panel\components\project_profitability_section.js

```javascript
import { useService } from "@web/core/utils/hooks";
import { Component, useState } from "@odoo/owl";
import { formatFloat, formatFloatTime } from "@web/views/fields/formatters";

export class ProjectProfitabilitySection extends Component {

    static props = {
        revenue: Object,
        labels: Object,
        formatMonetary: Function,
        onProjectActionClick: Function,
        onClick: Function,
        projectId: Number,
        context: Object,
    };
    static template = "sale_project.ProjectProfitabilitySection";


    setup() {
        this.orm = useService("orm");
        this.actionService = useService("action");
        this.state = useState({
            isFolded: true,
            displayLoadMore: null,
        });
        this.sale_items = [];
    }

    get revenue() {
        return this.props.revenue;
    }

    async toggleSaleItems() {
        if (this.state.displayLoadMore === null) {
            // first time the section is unfold, load the 5 first items.
            await this.onLoadMoreClick()
        }
        // the state change is done at the end to ensure the loaded data are present when the component is rendered
        this.state.isFolded = !this.state.isFolded;
    }

    formatValue(value, unit) {
        return unit === "Hours" ? formatFloatTime(value) : formatFloat(value);
    }

    _getOrmValue(offset, section_id) {
        return {
            function: "get_sale_items_data",
            args: [this.props.projectId, offset, 5, true, section_id],
        };
    }

    async onLoadMoreClick() {
        const offset = this.sale_items.length;
        const orm_value = this._getOrmValue(offset, this.props.revenue.id);
        const newItems = await this.orm.call(
            "project.project",
            orm_value.function,
            orm_value.args,
            {
                context: this.props.context,
            }
        );
        this.sale_items = [...this.sale_items, ...newItems.sol_items];
        this.state.displayLoadMore = newItems.displayLoadMore;
        this.render();
    }

    async onSaleItemActionClick(params) {
        if (params.resId && params.type !== "object") {
            const action = await this.actionService.loadAction(params.name, this.props.context);
            this.actionService.doAction({
                ...action,
                res_id: params.resId,
                views: [[false, "form"]],
            });
        } else {
            this.props.onProjectActionClick(params);
        }
    }
}

```

## File: static\src\components\project_right_side_panel\components\project_profitability_section.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="sale_project.ProjectProfitabilitySection">
        <tr>
            <t t-set="revenue_label" t-value="props.labels[revenue.id] or revenue.id"/>
            <td class="align-middle">
                <t t-if="this.props.revenue.isSectionFoldable">
                    <button t-attf-class="btn o_group_caret fa fa-fw me-1 #{state.isFolded ? 'fa-caret-right' : 'fa-caret-down'}" t-on-click="() => this.toggleSaleItems()"/>
                </t>
                <a class="revenue_section" t-if="revenue.action" href="#"
                    t-on-click="() => this.props.onClick(revenue.action)"
                >
                    <t t-esc="revenue_label"/>
                </a>
                <t t-esc="revenue_label" t-else=""/>
            </td>
            <td t-attf-class="text-end align-middle {{ revenue.invoiced + revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced + revenue.to_invoice)"/></td>
            <td t-attf-class="text-end align-middle {{ revenue.to_invoice === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.to_invoice)"/></td>
            <td t-attf-class="text-end align-middle {{ revenue.invoiced === 0 ? 'text-500' : ''}}"><t t-esc="props.formatMonetary(revenue.invoiced)"/></td>
        </tr>
        <t t-if="!state.isFolded and this.props.revenue.isSectionFoldable">
            <tr>
                <td colspan="4" class="p-0 m-0">
                    <table class="table table-sm table-striped table-hover mb-0">
                        <thead>
                            <tr class="bg-light">
                                <th style="padding-left:30px">Sales Order Items</th>
                                <th class="text-end">Sold</th>
                                <th class="text-end">Delivered</th>
                                <th class="text-end">Invoiced</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr class="bg-light" t-foreach="sale_items" t-as="sale_item" t-key="sale_item.id">
                                <t t-set="uom_name" t-value="sale_item.product_uom and sale_item.product_uom[1]"/>
                                <td style="padding-left: 30px">
                                    <a t-if="sale_item.action" href="#" t-on-click="() => this.onSaleItemActionClick(sale_item.action)">
                                        <t t-esc="sale_item.name"/>
                                    </a>
                                    <t t-else="" t-esc="sale_item.name"/>
                                </td>
                                <td class="text-end align-middle"><t t-esc="formatValue(sale_item.product_uom_qty, uom_name)"/> <t t-esc="uom_name"/></td>
                                <td class="text-end align-middle"><t t-esc="formatValue(sale_item.qty_delivered, uom_name)"/> <t t-esc="uom_name"/></td>
                                <td class="text-end align-middle"><t t-esc="formatValue(sale_item.qty_invoiced, uom_name)"/> <t t-esc="uom_name"/></td>
                            </tr>
                        </tbody>
                        <tfoot>
                            <tr class="border-0 bg-light" t-if="state.displayLoadMore">
                                <td class="text-center" colspan="4">
                                    <a class="cursor-pointer btn-link w-100" t-on-click="() => this.onLoadMoreClick(revenue)">
                                        Load more
                                    </a>
                                </td>
                            </tr>
                        </tfoot>
                    </table>
                </td>
            </tr>
        </t>
    </t>
</templates>

```

## File: static\src\views\project_task_list\project_task_list_sale_line.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { ProjectTaskListRenderer } from "@project/views/project_task_list/project_task_list_renderer";
import { getRawValue } from "@web/views/kanban/kanban_record";

patch(ProjectTaskListRenderer.prototype, {
    /**
     * This method prevents from computing the selection once for each cell when
     * rendering the list. Indeed, `selection` is a getter which browses all
     * records, so computing it for each cell slows down the rendering a lot on
     * large tables. Moreover, it also prevents from iterating over the selection
     * to compare tasks' partners.
     *
     * It returns true iff the selected tasks all have the same partner.
     */
    haveSelectedTasksSamePartner() {
        if (this._haveSelectedTasksSamePartner === undefined) {
            const selection = this.props.list.selection;
            const partnerId = selection.length && getRawValue(selection[0], "partner_id");
            this._haveSelectedTasksSamePartner = selection.every(
                (task) => getRawValue(task, "partner_id") === partnerId
            );
            Promise.resolve().then(() => {
                delete this._haveSelectedTasksSamePartner;
            });
        }
        return this._haveSelectedTasksSamePartner;
    },

    isCellReadonly(column, record) {
        let readonly = false;
        if (column.name === "sale_line_id") {
            readonly = !this.haveSelectedTasksSamePartner();
        }
        return readonly || super.isCellReadonly(column, record);
    }
});

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_form_view_invoice_policy_inherit_sale_project" model="ir.ui.view">
        <field name="name">product.template.inherit.sale.projectform</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale.product_template_form_view"/>
        <field name="arch" type="xml">
            <field name="invoice_policy" position="attributes">
                <attribute name="invisible" separator="or" add="type == 'service'"/>
            </field>
            <field name="service_tracking" position="attributes">
                <attribute name="invisible" remove="1" separator="or"/>
            </field>
            <field name="service_tracking" position="after">
                <div class="o_td_label d-inline-flex" invisible="service_tracking != 'task_global_project'">
                    <label for='project_id'/>
                </div>
                <field name="project_id" context="{'default_allow_billable': True}" invisible="service_tracking != 'task_global_project'" nolabel="1" placeholder="Defined on quotation"/>
                <div class="o_td_label d-inline-flex" invisible="service_tracking not in ['task_in_project', 'project_only']">
                    <label for='project_template_id'/>
                </div>
                <field name="project_template_id" context="{'active_test': False, 'default_allow_billable': True}" invisible="service_tracking not in ['task_in_project', 'project_only']" nolabel="1" placeholder="Empty project"/>
                <field name="service_policy"
                    string="Invoicing Policy"
                    invisible="type != 'service' or sale_ok == False"
                    required="type == 'service' and sale_ok == True"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\project_sharing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_sharing_inherit_project_task_view_form" model="ir.ui.view">
        <field name="name">project.task.view.inherit</field>
        <field name="model">project.task</field>
        <field name="priority">300</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <field name="display_sale_order_button" invisible="1"/>
                <field name="allow_billable" invisible="1"/>
                <button class="oe_stat_button"
                        type="object" name="action_project_sharing_view_so" icon="fa-dollar"
                        invisible="not display_sale_order_button">
                        <div class="o_stat_info">
                            <span class="o_stat_text">Sales Order</span>
                        </div>
                </button>
            </div>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_line_id" string="Sales Order Item" options='{"no_open": True}' readonly="1" invisible="not partner_id" context="{'create': False, 'edit': False, 'delete': False}"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/list/field[@name='partner_id']" position="after">
                <field name="allow_billable" column_invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/list/field[@name='partner_id']" position="after">
                <field name="allow_billable" column_invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/list/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/list/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
        </field>
    </record>

    <record id="project_sharing_inherit_project_task_view_tree" model="ir.ui.view">
        <field name="name">project.task.view.list.inherit</field>
        <field name="model">project.task</field>
        <field name="priority">300</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_tree"/>
        <field name="arch" type="xml">
            <field name="allow_milestones" position="after">
                <field name="allow_billable" column_invisible="1"/>
            </field>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not allow_billable</attribute>
            </xpath>
        </field>
    </record>

    <record id="project.project_sharing_project_task_action" model="ir.actions.act_window">
        <field name="context">{
            'default_project_id': active_id,
            'active_id_chatter': active_id,
            'delete': false,
            'sale_show_partner_name': true,
        }</field>
    </record>

</odoo>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project.action_view_task" model="ir.actions.act_window">
        <field name="context">{'sale_show_partner_name': True, 'search_default_my_tasks': 1}</field>
    </record>

    <record id="project.action_view_my_task" model="ir.actions.act_window">
        <field name="context">{'search_default_open_tasks': 1, 'all_task': 0, 'default_user_ids': [(4, uid)], 'sale_show_partner_name': True}</field>
    </record>

    <record id="project.action_view_all_task" model="ir.actions.act_window">
        <field name="context">{'search_default_open_tasks': 1, 'default_user_ids': [(4, uid)], 'sale_show_partner_name': True}</field>
    </record>

    <record id="project.action_project_task_user_tree" model="ir.actions.act_window">
        <field name="context">{'group_by':[], 'graph_measure': '__count__', 'sale_show_partner_name': True}</field>
    </record>

    <record id="project_project_view_inherit_project_filter" model="ir.ui.view">
        <field name="name">project.project.select.inherit.project</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.view_project_project_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_order_id"/>
            </xpath>
        </field>
    </record>

    <record id="project_project_view_tree_inherit_sale_project" model="ir.ui.view">
        <field name="name">project.project.list.inherit.sale.project</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.view_project"/>
        <field name="priority">50</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_line_id" optional="hide" readonly="1"/>
                <field name="allow_billable" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
        </field>
    </record>

    <record id="view_edit_project_inherit_form" model="ir.ui.view">
        <field name="name">project.project.view.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <field name="display_sales_stat_buttons" invisible="1"/>
                <field name="allow_billable" invisible="1" />
                <field name="privacy_visibility" invisible="1" />
                <button class="oe_stat_button" type="object" name="action_customer_preview" icon="fa-globe icon" invisible="not partner_id or not allow_billable or privacy_visibility != 'portal'">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Customer</span>
                        <span class="o_stat_text">Preview</span>
                    </div>
                </button>
                <button
                    class="oe_stat_button"
                    type="object"
                    name="action_view_sos"
                    icon="fa-dollar"
                    invisible="not display_sales_stat_buttons or sale_order_count == 0"
                    groups="sales_team.group_sale_salesman_all_leads"
                    context="{
                        'create_for_project_id': id,
                        'default_project_id': id,
                        'default_partner_id': partner_id
                    }">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="sale_order_count" nolabel="1"/>
                        </span>
                        <span class="o_stat_text">
                            Sales Orders
                        </span>
                    </div>
                </button>
                <button
                    class="oe_stat_button"
                    type="object"
                    name="action_view_sos"
                    icon="fa-dollar"
                    invisible="not display_sales_stat_buttons or sale_order_count != 0"
                    groups="sales_team.group_sale_salesman_all_leads"
                    context="{
                        'create_for_project_id': id,
                        'default_project_id': id,
                        'default_partner_id': partner_id
                    }">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">
                            <span class="o_stat_value">0</span> Sales Order
                        </span>
                        <span class="o_stat_text">
                            Make Billable
                        </span>
                    </div>
                </button>
            </div>
            <xpath expr="//header" position="inside">
                <field name="has_any_so_to_invoice" invisible="1"/>
                <field name="has_any_so_with_nothing_to_invoice" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//group[@name='group_time_managment']" position="after">
                 <group name="group_sales_invoicing" string="Sales &amp; Invoicing" col="1" class="row mt16 o_settings_container col-lg-6">
                    <div>
                        <setting class="col-lg-12" help="Invoice your time and material to customers" id="allow_billable_container">
                            <field name="allow_billable"/>
                        </setting>
                    </div>
                </group>
            </xpath>
            <xpath expr="//page[@name='settings']//field[@name='privacy_visibility']" position="before">
                <field name="reinvoiced_sale_order_id" invisible="not allow_billable or not partner_id"/>
                <label for="sale_line_id" invisible="not allow_billable or not partner_id"/>
                <div 
                    class="o_row" 
                    invisible="not allow_billable or not partner_id">
                    <field name="sale_line_id"
                        groups="!sales_team.group_sale_salesman"
                        options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"/>
                    <field name="sale_line_id"
                        groups="sales_team.group_sale_salesman"
                        options="{'no_edit': True, 'delete': False}"
                        context="{
                            'form_view_ref': 'sale_project.sale_order_line_view_form_editable',
                            'default_partner_id': partner_id,
                            'default_company_id': company_id,
                        }"/>
                    <span
                        class="fa fa-exclamation-triangle text-warning"
                        title="The sales order associated with this project has been cancelled. We recommend either updating the sales order item or cancelling this project in alignment with the cancellation of the sales order."
                        invisible="sale_order_state != 'cancel'"/>
                </div>
                <field name="sale_order_state" invisible="1"/>
            </xpath>
        </field>
    </record>

    <record id="view_sale_project_quick_create_task_form" model="ir.ui.view">
        <field name="name">project.task.view.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.quick_create_task_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='project_id']" position="attributes">
                <attribute name="context">{'default_allow_billable': True, 'default_type_ids': [(4, context.get('default_stage_id', False))]}
                </attribute>
            </xpath>
        </field>
    </record>

    <record id="view_sale_project_inherit_form" model="ir.ui.view">
        <field name="name">project.task.view.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_form2"/>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <xpath expr="//span[@id='start_rating_buttons']" position="before">
                <button class="oe_stat_button"
                        type="object" name="action_view_so" icon="fa-dollar"
                        invisible="not sale_order_id"
                        groups="sales_team.group_sale_salesman">
                        <div class="o_stat_info">
                            <span class="o_stat_text">Sales Order</span>
                        </div>
                </button>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="project_sale_order_id" invisible="1"/>
                <field name="sale_order_id" invisible="True" groups="sales_team.group_sale_salesman"/>
                <label for="sale_line_id" invisible="not allow_billable or not project_id or not partner_id"/>
                <div 
                    name="sale_line_div"
                    class="o_row"
                    invisible="not allow_billable or not project_id or not partner_id">
                    <field name="sale_line_id"
                        groups="!sales_team.group_sale_salesman"
                        string="Sales Order Item"
                        options='{"no_open": True}'
                        readonly="1"
                        invisible="not sale_line_id"/>
                    <field name="sale_line_id"
                        groups="sales_team.group_sale_salesman"
                        string="Sales Order Item"
                        readonly="0"
                        context="{
                            'create': False, 'edit': False, 'delete': False,
                            'with_price_unit': True,
                            'form_view_ref': 'sale_project.sale_order_line_view_form_editable',
                            'default_partner_id': partner_id,
                            'default_company_id': company_id,
                            'default_order_id': project_sale_order_id,
                        }"
                        placeholder="Non-billable"/>
                    <span 
                        class="fa fa-exclamation-triangle text-warning"
                        title="The sales order associated with this task has been cancelled. We recommend either updating the sales order item or cancelling this task in alignment with the cancellation of the sales order." 
                        invisible="sale_order_state != 'cancel'"/>
                </div>
                <field name="allow_billable" invisible="1"/>
                <field name="sale_order_state" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/list/field[@name='partner_id']" position="after">
                <field name="sale_line_id"
                       optional="hide"
                       options='{"no_create": True}'
                       context="{'create': False, 'edit': False, 'delete': False, 'with_price_unit': True}"
                       placeholder="Non-billable"
                       groups="sales_team.group_sale_salesman"
                       invisible="not allow_billable"/>
                <field name="sale_line_id" optional="hide" options="{'no_open': True}" readonly="1" groups="!sales_team.group_sale_salesman"/>
                <field name="allow_billable" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/list/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/list/field[@name='partner_id']" position="after">
                <field name="sale_line_id" optional="hide" readonly="1" groups="sales_team.group_sale_salesman"/>
                <field name="sale_line_id" optional="hide" options="{'no_open': True}" readonly="1" groups="!sales_team.group_sale_salesman"/>
                <field name="allow_billable" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/list/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <field name="project_id" position="attributes">
                <attribute name="context">{'default_allow_billable': True}</attribute>
            </field>
        </field>
    </record>

    <record id="project_task_view_tree_main_base" model="ir.ui.view">
        <field name="name">project.task.main.list.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_task_view_tree_main_base"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="allow_billable" column_invisible="True"/>
            </field>
            <field name="partner_id" position="attributes">
                <attribute name="column_invisible">context.get('hide_partner')</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </field>
        </field>
    </record>

    <record id="view_task_tree2_inherit_sale_project" model="ir.ui.view">
        <field name="name">project.task.form.inherit.sale.project</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_task_view_tree_base"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_line_id" optional="hide" groups="sales_team.group_sale_salesman" options="{'no_create': True}"/>
                <field name="sale_line_id" optional="hide" options="{'no_open': True}" readonly="1" groups="!sales_team.group_sale_salesman"/>
            </xpath>
        </field>
    </record>

    <record id="project_task_view_search" model="ir.ui.view">
        <field name="name">project.task.search.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_search_form_project_base"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="sale_order_id" filter_domain="['|', ('sale_order_id', 'ilike', self), ('sale_line_id', 'ilike', self)]"/>
            </field>
            <field name="partner_id" position="attributes">
                <attribute name="invisible">context.get('hide_partner')</attribute>
            </field>
            <filter name="customer" position="attributes">
                <attribute name="invisible">context.get('hide_partner')</attribute>
            </filter>
        </field>
    </record>

    <record id="project_milestone_view_form" model="ir.ui.view">
        <field name="name">project.milestone.view.form.inherit</field>
        <field name="model">project.milestone</field>
        <field name="inherit_id" ref="project.project_milestone_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='main_details']" position="after">
                <field name="allow_billable" invisible="1"/>
                <group invisible="not allow_billable">
                    <field name="project_partner_id" invisible="1"/>
                    <field name="sale_line_id" groups="!sales_team.group_sale_salesman" placeholder="Non-billable" options="{'no_open': True}" readonly="1"/>
                    <field name="sale_line_id" groups="sales_team.group_sale_salesman" options="{'no_create': True}" placeholder="Non-billable" readonly="0"/>
                    <label for="quantity_percentage" invisible="not sale_line_id"/>
                    <div class="col-6" invisible="not sale_line_id">
                        <field name="quantity_percentage" groups="!sales_team.group_sale_salesman" widget="percentage" readonly="1" class="mw-25"/>
                        <field name="quantity_percentage" class="mw-25" groups="sales_team.group_sale_salesman"
                            widget="percentage" decoration-danger="quantity_percentage &lt; 0 or 1 &lt; quantity_percentage" readonly="0"/>
                        <span>
                            (<field name="product_uom_qty" class="mw-25"  decoration-danger="quantity_percentage &lt; 0 or 1 &lt; quantity_percentage"/>
                            <field name="product_uom" class="w-auto text-end" groups="uom.group_uom" options="{'no_open': True}"/>
                            <span>)</span>
                        </span>
                    </div>
                </group>
            </xpath>
            <xpath expr="//button[@name='%(project.action_view_task_from_milestone)d']" position="before">
                <button name="action_view_sale_order" type="object" class="oe_stat_button" icon="fa-dollar" invisible="not sale_line_id">
                    <div class="o_stat_info">
                        <span class="o_stat_text">Sales Order</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <record id="project_milestone_view_tree" model="ir.ui.view">
        <field name="name">project.milestone.view.list.inherit</field>
        <field name="model">project.milestone</field>
        <field name="inherit_id" ref="project.project_milestone_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="project_partner_id" column_invisible="True"/>
                <field name="allow_billable" column_invisible="True"/>
                <field name="sale_line_id" optional="hide" options="{'no_open': True}" placeholder="Non-billable" readonly="1" groups="!sales_team.group_sale_salesman"/>
                <field name="sale_line_id" optional="hide" options="{'no_create': True}" placeholder="Non-billable" groups="sales_team.group_sale_salesman"/>
                <field name="quantity_percentage" string="Quantity (%)" optional="hide" widget="percentage" groups="!sales_team.group_sale_salesman"/>
                <field name="quantity_percentage" string="Quantity (%)" optional="hide" widget="percentage" readonly="0" groups="sales_team.group_sale_salesman"/>
                <field name="product_uom_qty" optional="hide" groups="!sales_team.group_sale_salesman" readonly="1"/>
                <field name="product_uom_qty" optional="hide" groups="sales_team.group_sale_salesman"/>
            </xpath>
            <xpath expr="//button[@name='action_view_tasks']" position="after">
                <button name="action_view_sale_order" type="object" string="View Sales Order"
                    class="btn btn-link float-end" invisible="not sale_line_id"/>
            </xpath>
        </field>
    </record>

    <record id="sale_project_milestone_view_tree" model="ir.ui.view">
        <field name="name">project.milestone.view.list.inherit</field>
        <field name="model">project.milestone</field>
        <field name="inherit_id" ref="sale_project.project_milestone_view_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sale_line_id'][1]" position="attributes">
                <attribute name="optional">show</attribute>
            </xpath>
            <xpath expr="//field[@name='sale_line_id'][2]" position="attributes">
                <attribute name="optional">show</attribute>
            </xpath>
            <xpath expr="//field[@name='quantity_percentage'][1]" position="attributes">
                <attribute name="optional">show</attribute>
            </xpath>
            <xpath expr="//field[@name='quantity_percentage'][2]" position="attributes">
                <attribute name="optional">show</attribute>
            </xpath>
        </field>
    </record>

    <!-- Views for 'Tasks' stat button via Contact form -->
    <record id="view_task_form_res_partner" model="ir.ui.view">
        <field name="name">project.task.form.res.partner.inherit.sale_project</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_form_res_partner"/>
        <field name="arch" type="xml">
            <xpath expr="//group//field[@name='project_id']" position="attributes">
                <attribute name="domain">['&amp;', ('allow_billable', '=', True), '&amp;',('active', '=', True), '|', ('company_id', '=', False), ('company_id', '=?', company_id)]</attribute>
            </xpath>
        </field>
    </record>

    <record id="quick_create_task_form_res_partner" model="ir.ui.view">
        <field name="name">project.task.form.quick_create.res.partner.inherit.sale_project</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.quick_create_task_form_res_partner"/>
        <field name="arch" type="xml">
            <xpath expr="//group//field[@name='project_id']" position="attributes">
                <attribute name="domain">[('allow_billable', '=', True), ('type_ids', 'in', context['default_stage_id'])] if context.get('default_stage_id') else []</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\project_update_template.xml

```xml
<?xml version="1.0"?>
<odoo>

    <template id="sale_project.milestone_deadline_inherit" inherit_id="project.milestone_deadline">
        <xpath expr="//t[@t-if=&quot;milestone['deadline']&quot;]" position="before">
            <t t-if="milestone['quantity_percentage']">
                <font style="color: rgb(190, 190, 190);">
                <t t-if="milestone['sale_line_display_name']">
                    (<t t-out="milestone['sale_line_display_name']"/> -
                </t>
                <t t-out="100 * milestone['quantity_percentage']"/>%)</font>
            </t>
        </xpath>
    </template>

</odoo>

```

## File: views\project_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="project_project_view_form_simplified_inherit" model="ir.ui.view">
        <field name="name">project.project.view.form.simplified.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.project_project_view_form_simplified"/>
        <field name="priority">25</field>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_settings_container')]" position="inside">
                <field name="company_id" invisible="1"/>
                <setting class="col-lg-12" help="Invoice your time and material to customers" invisible="context.get('hide_allow_billable', False)">
                    <field name="allow_billable"/>
                    <div invisible="not allow_billable">
                        <label for="partner_id"/>
                        <field name="partner_id" class="ms-1" widget="res_partner_many2one" context="{'res_partner_search_mode': 'customer'}"
                            options="{'no_create_edit': True, 'no_open': True}" placeholder="Select who to bill..."/>
                    </div>
                </setting>
            </xpath>
        </field>
    </record>

    <record id="project_embedded_action_invoices" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">60</field>
        <field name="name">Invoices</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_project_invoices</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('account.group_account_invoice')), (4, ref('sales_team.group_sale_salesman'))]" />
        <field name="domain">[('allow_billable', '=', True)]</field>
    </record>

    <record id="project_embedded_action_sales_orders" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">50</field>
        <field name="name">Sales Orders</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_view_sos</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('sales_team.group_sale_salesman'))]" />
        <field name="domain">[('allow_billable', '=', True)]</field>
    </record>

    <record id="project_embedded_action_invoices_dashboard" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">70</field>
        <field name="name">Invoices</field>
        <field name="parent_action_id" ref="project.project_update_all_action"/>
        <field name="python_method">action_open_project_invoices</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('account.group_account_invoice')), (4, ref('sales_team.group_sale_salesman'))]" />
        <field name="domain">[('allow_billable', '=', True)]</field>
    </record>

    <record id="project_embedded_action_sales_orders_dashboard" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">50</field>
        <field name="name">Sales Orders</field>
        <field name="parent_action_id" ref="project.project_update_all_action"/>
        <field name="python_method">action_view_sos</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('sales_team.group_sale_salesman'))]" />
        <field name="domain">[('allow_billable', '=', True)]</field>
    </record>

    <record id="project_embedded_action_vendor_bills" model="ir.embedded.actions">
        <field name="parent_res_model">project.project</field>
        <field name="sequence">75</field>
        <field name="name">Vendor Bills</field>
        <field name="parent_action_id" ref="project.act_project_project_2_project_task_all"/>
        <field name="python_method">action_open_project_vendor_bills</field>
        <field name="context">{"from_embedded_action": true}</field>
        <field name="groups_ids" eval="[(4, ref('account.group_account_invoice')), (4, ref('sales_team.group_sale_salesman'))]"/>
        <field name="domain">[('allow_billable', '=', True)]</field>
    </record>

    <record id="project.open_view_project_all_config" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True, 'display_milestone_deadline': True}</field>
    </record>
    <record id="project.open_view_project_all_config_group_stage" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True}</field>
    </record>
    <record id="project.open_view_project_all" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True, 'display_milestone_deadline': True}</field>
    </record>
    <record id="project.open_view_project_all_group_stage" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True}</field>
    </record>
</odoo>

```

## File: views\sale_order_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_order_line_tree_with_create" model="ir.ui.view">
        <field name="name">sale.order.line.list.with.create</field>
        <field name="model">sale.order.line</field>
        <field name="inherit_id" ref="sale.view_order_line_tree"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <list position="attributes">
                <attribute name="create">true</attribute>
            </list>
        </field>
    </record>

    <record id="sale_order_line_view_form_editable" model="ir.ui.view">
        <field name="name">sale.order.line.view.form.editable</field>
        <field name="model">sale.order.line</field>
        <field name="inherit_id" ref="sale.sale_order_line_view_form_readonly"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <form position="attributes">
                <attribute name="edit">true</attribute>
            </form>
            <field name="product_id" position="attributes">
                <attribute name="domain">[('type', '=', 'service')]</attribute>
                <attribute name="context">{
                    'default_type': 'service',
                    'default_service_policy': 'ordered_prepaid',
                }</attribute>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_order_form_inherit_sale_project" model="ir.ui.view">
        <field name="name">sale.order.form.sale.project</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_view_invoice']" position="before">
                <field name="show_task_button" invisible="1"/>
                <field name="show_project_button" invisible="1"/>
                <field name="show_create_project_button" invisible="1"/>
                <field name="project_ids" invisible="1"/>
                <field name="is_product_milestone" invisible="1"/>
                <button type="object" name="action_view_project_ids" class="oe_stat_button" icon="fa-puzzle-piece" invisible="not show_project_button" groups="project.group_project_user">
                    <field name="project_count" widget="statinfo" string="Projects"/>
                </button>
                <button class="oe_stat_button" name="action_view_milestone" type="object" icon="fa-check-square-o" invisible="not is_product_milestone or not project_ids or state == 'draft'" groups="project.group_project_milestone">
                    <field name="milestone_count" widget="statinfo" string="Milestones"/>
                </button>
                <button type="object" name="action_view_task" class="oe_stat_button" icon="fa-check" invisible="not show_task_button" groups="project.group_project_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Tasks</span>
                        <span class="o_stat_value">
                            <field name="closed_task_count"/> / <field name="tasks_count"/>
                            (<field name="completed_task_percentage" widget="percentage" options="{'digits': [1, 0]}"/>)
                        </span>
                    </div>
                </button>
            </xpath>
            <field name="journal_id" position="before">
                <field name="project_id" groups="project.group_project_user" context="{'default_allow_billable': True}"/>
            </field>
        </field>
    </record>

    <record id="view_sales_order_filter_inherit_sale_project" model="ir.ui.view">
        <field name="name">sale.order.list.select.inherit.sale_project</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="arch" type="xml">
            <field name="order_line" position="after">
                <field name="project_id" groups="project.group_project_user"/>
            </field>
        </field>
    </record>

    <record id="model_sale_order_action_create_project" model="ir.actions.server">
        <field name="name">Create Project</field>
        <field name="model_id" ref="sale.model_sale_order"/>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">action = records.action_create_project()</field>
    </record>

</odoo>

```

## File: views\sale_project_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_tasks_list_inherit" inherit_id="project.portal_tasks_list">
        <xpath expr="//t[@t-foreach='grouped_tasks']/tbody/tr[hasclass('table-light')]" position="inside">
            <th t-if="groupby == 'sale_order_id'" t-attf-colspan="{{grouped_tasks_colspan}}">
                <span t-if="tasks[0].sudo().sale_order_id" class="text-truncate" t-field="tasks[0].sudo().sale_order_id"/>
                <span t-else="">Not Billed</span>
            </th>
            <th t-if="groupby == 'sale_line_id'" t-attf-colspan="{{grouped_tasks_colspan}}">
                <span t-if="tasks[0].sudo().sale_line_id" class="text-truncate" t-field="tasks[0].sudo().sale_line_id"/>
                <span t-else="">Not Billed</span>
            </th>
        </xpath>
    </template>

</odoo>

```

