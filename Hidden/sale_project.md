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
        'views/project_sharing_views.xml',
        'views/project_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'sale_project/static/src/components/project_right_side_panel/**/*',
        ],
        'web.assets_tests': [
            'sale_project/static/tests/tours/**/*',
        ],
    },
    'post_init_hook': '_set_allow_billable_in_project',
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _lt
from odoo.osv.expression import OR

from odoo.addons.project.controllers.portal import ProjectCustomerPortal


class SaleProjectCustomerPortal(ProjectCustomerPortal):

    def _task_get_searchbar_groupby(self, milestones_allowed, project=False):
        values = super()._task_get_searchbar_groupby(milestones_allowed, project)
        if project and not project.sudo()._get_hide_partner():
            del values['customer']
        if not project or project.sudo().allow_billable:
            values['sale_order'] = {'input': 'sale_order', 'label': _lt('Sales Order'), 'order': 8}
            values['sale_line'] = {'input': 'sale_line', 'label': _lt('Sales Order Item'), 'order': 9}
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_groupby_mapping(self):
        groupby_mapping = super()._task_get_groupby_mapping()
        groupby_mapping.update(sale_order='sale_order_id', sale_line='sale_line_id')
        return groupby_mapping

    def _task_get_searchbar_inputs(self, milestones_allowed, project=False):
        values = super()._task_get_searchbar_inputs(milestones_allowed, project)
        if project and not project.sudo()._get_hide_partner():
            del values['customer']
        if not project or project.sudo().allow_billable:
            values['sale_order'] = {'input': 'sale_order', 'label': _lt('Search in Sales Order'), 'order': 8}
            values['invoice'] = {'input': 'invoice', 'label': _lt('Search in Invoice'), 'order': 10}
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_search_domain(self, search_in, search):
        search_domain = [super()._task_get_search_domain(search_in, search)]
        if search_in in ('sale_order', 'all'):
            search_domain.append(['|', ('sale_order_id.name', 'ilike', search), ('sale_line_id.name', 'ilike', search)])
        if search_in in ('invoice', 'all'):
            search_domain.append([('sale_order_id.invoice_ids.name', 'ilike', search)])
        return OR(search_domain)

    def _prepare_project_sharing_session_info(self, project, task=None):
        session_info = super()._prepare_project_sharing_session_info(project, task)
        session_info['user_context'].update({
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

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _compute_analytic_distribution(self):
        # when a project creates an aml, it adds an analytic account to it. the following filter is to save this
        # analytic account from being overridden by analytic default rules and lack thereof
        project_amls = self.filtered(lambda aml: aml.analytic_distribution and any(aml.sale_line_ids.project_id))
        super(AccountMoveLine, self - project_amls)._compute_analytic_distribution()
        project_id = self.env.context.get('project_id')
        if project_id:
            analytic_account = self.env['project.project'].browse(project_id).analytic_account_id
            for line in project_amls:
                line.analytic_distribution = line.analytic_distribution or {analytic_account.id: 100}

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
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
        if (self.user_has_groups('project.group_project_milestone') or
                (self.env.user.has_group('base.group_public') and user.has_group('project.group_project_milestone'))
        ):
            service_policies.insert(1, ('delivered_milestones', _('Based on Milestones')))
        return service_policies

    service_tracking = fields.Selection(
        selection=[
            ('no', 'Nothing'),
            ('task_global_project', 'Task'),
            ('task_in_project', 'Project & Task'),
            ('project_only', 'Project'),
        ],
        string="Create on Order", default="no",
        help="On Sales order confirmation, this product can generate a project and/or task. \
        From those, you can track the service you are selling.\n \
        'In sale order\'s project': Will use the sale order\'s configured project if defined or fallback to \
        creating a new project based on the selected template.")
    project_id = fields.Many2one(
        'project.project', 'Project', company_dependent=True,
    )
    project_template_id = fields.Many2one(
        'project.project', 'Project Template', company_dependent=True, copy=True,
    )
    service_policy = fields.Selection('_selection_service_policy', string="Service Invoicing Policy", compute_sudo=True, compute='_compute_service_policy', inverse='_inverse_service_policy')
    service_type = fields.Selection(selection_add=[
        ('milestones', 'Project Milestones'),
    ])

    @api.depends('invoice_policy', 'service_type', 'type')
    def _compute_service_policy(self):
        for product in self:
            product.service_policy = self._get_general_to_service(product.invoice_policy, product.service_type)
            if not product.service_policy and product.type == 'service':
                product.service_policy = 'ordered_prepaid'

    @api.depends('service_tracking', 'service_policy', 'type', 'sale_ok')
    def _compute_product_tooltip(self):
        super()._compute_product_tooltip()
        for record in self.filtered(lambda record: record.type == 'service' and record.sale_ok):
            if record.service_policy == 'ordered_prepaid':
                if record.service_tracking == 'no':
                    record.product_tooltip = _(
                        "Invoice ordered quantities as soon as this service is sold."
                    )
                elif record.service_tracking == 'task_global_project':
                    record.product_tooltip = _(
                        "Invoice ordered quantities as soon as this service is sold. "
                        "Create a task in an existing project to track the time spent."
                    )
                elif record.service_tracking == 'project_only':
                    record.product_tooltip = _(
                        "Invoice ordered quantities as soon as this service is sold. "
                        "Create an empty project for the order to track the time spent."
                    )
                elif record.service_tracking == 'task_in_project':
                    record.product_tooltip = _(
                        "Invoice ordered quantities as soon as this service is sold. "
                        "Create a project for the order with a task for each sales order line "
                        "to track the time spent."
                    )
            elif record.service_policy == 'delivered_milestones':
                if record.service_tracking == 'no':
                    record.product_tooltip = _(
                        "Invoice your milestones when they are reached."
                    )
                elif record.service_tracking == 'task_global_project':
                    record.product_tooltip = _(
                        "Invoice your milestones when they are reached. "
                        "Create a task in an existing project to track the time spent."
                    )
                elif record.service_tracking == 'project_only':
                    record.product_tooltip = _(
                        "Invoice your milestones when they are reached. "
                        "Create an empty project for the order to track the time spent."
                    )
                elif record.service_tracking == 'task_in_project':
                    record.product_tooltip = _(
                        "Invoice your milestones when they are reached. "
                        "Create a project for the order with a task for each sales order line "
                        "to track the time spent."
                    )
            elif record.service_policy == 'delivered_manual':
                if record.service_tracking == 'no':
                    record.product_tooltip = _(
                        "Invoice this service when it is delivered (set the quantity by hand on your sales order lines). "
                    )
                elif record.service_tracking == 'task_global_project':
                    record.product_tooltip = _(
                        "Invoice this service when it is delivered (set the quantity by hand on your sales order lines). "
                        "Create a task in an existing project to track the time spent."
                    )
                elif record.service_tracking == 'project_only':
                    record.product_tooltip = _(
                        "Invoice this service when it is delivered (set the quantity by hand on your sales order lines). "
                        "Create an empty project for the order to track the time spent."
                    )
                elif record.service_tracking == 'task_in_project':
                    record.product_tooltip = _(
                        "Invoice this service when it is delivered (set the quantity by hand on your sales order lines). "
                        "Create a project for the order with a task for each sales order line "
                        "to track the time spent."
                    )

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

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductTemplate, self)._onchange_type()
        if self.type != 'service':
            self.service_tracking = 'no'
        return res

    def write(self, vals):
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super(ProductTemplate, self).write(vals)


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

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductProduct, self)._onchange_type()
        if self.type != 'service':
            self.service_tracking = 'no'
        return res

    def write(self, vals):
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super(ProductProduct, self).write(vals)

```

## File: models\project.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json
from collections import defaultdict

from odoo import api, fields, models, _, _lt
from odoo.exceptions import ValidationError, AccessError
from odoo.osv import expression
from odoo.tools import Query, SQL, OrderedSet
from odoo.tools.misc import unquote


class Project(models.Model):
    _inherit = 'project.project'

    def _domain_sale_line_id(self):
        domain = expression.AND([
            self.env['sale.order.line']._sellable_lines_domain(),
            [
                ('is_service', '=', True),
                ('is_expense', '=', False),
                ('state', '=', 'sale'),
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
    sale_order_id = fields.Many2one(string='Sales Order', related='sale_line_id.order_id', help="Sales order to which the project is linked.")
    has_any_so_to_invoice = fields.Boolean('Has SO to Invoice', compute='_compute_has_any_so_to_invoice')
    sale_order_line_count = fields.Integer(compute='_compute_sale_order_count', groups='sales_team.group_sale_salesman')
    sale_order_count = fields.Integer(compute='_compute_sale_order_count', groups='sales_team.group_sale_salesman')
    has_any_so_with_nothing_to_invoice = fields.Boolean('Has a SO with an invoice status of No', compute='_compute_has_any_so_with_nothing_to_invoice')
    invoice_count = fields.Integer(compute='_compute_invoice_count', groups='account.group_account_readonly')
    vendor_bill_count = fields.Integer(related='analytic_account_id.vendor_bill_count', groups='account.group_account_readonly')
    partner_id = fields.Many2one(compute="_compute_partner_id", store=True, readonly=False)
    display_sales_stat_buttons = fields.Boolean(compute='_compute_display_sales_stat_buttons')

    @api.model
    def _map_tasks_default_valeus(self, task, project):
        defaults = super()._map_tasks_default_valeus(task, project)
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
        self.env.cr.execute("""
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
               AND id in %(ids)s""", {'ids': tuple(self.ids), 'invoice_status': invoice_status})
        return self.env['project.project'].browse([x[0] for x in self.env.cr.fetchall()])

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
        sale_order_items_per_project_id = self._fetch_sale_order_items_per_project_id({'project.task': [('state', 'in', self.env['project.task'].OPEN_STATES)]})
        for project in self:
            sale_order_lines = sale_order_items_per_project_id.get(project.id, self.env['sale.order.line'])
            project.sale_order_line_count = len(sale_order_lines)

            # Use sudo to avoid AccessErrors when the SOLs belong to different companies.
            project.sale_order_count = len(sale_order_lines.sudo().order_id)

    def _compute_invoice_count(self):
        query = self.env['account.move.line']._search([('move_id.move_type', 'in', ['out_invoice', 'out_refund'])])
        query.add_where(
            SQL(
                "%s && %s",
                [str(project.analytic_account_id.id) for project in self],
                self.env['account.move.line']._query_analytic_accounts(),
            )
        )
        query.order = None
        query_string, query_param = query.select(
            r"""DISTINCT move_id, (regexp_matches(jsonb_object_keys(account_move_line.analytic_distribution), '\d+', 'g'))[1]::int as account_id"""
        )
        query_string = f"""
            SELECT account_id, count(move_id) FROM
            ({query_string}) distribution
            GROUP BY account_id
        """
        self._cr.execute(query_string, query_param)
        data = {res['account_id']: res['count'] for res in self._cr.dictfetchall()}
        for project in self:
            project.invoice_count = data.get(project.analytic_account_id.id, 0)

    @api.depends('allow_billable', 'partner_id')
    def _compute_display_sales_stat_buttons(self):
        for project in self:
            project.display_sales_stat_buttons = project.allow_billable and project.partner_id

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
        all_sale_order_lines = self._fetch_sale_order_items({'project.task': [('state', 'in', self.env['project.task'].OPEN_STATES)]})
        action_window = {
            'type': 'ir.actions.act_window',
            'res_model': 'sale.order.line',
            'name': _("%(name)s's Sales Order Items", name=self.name),
            'context': {
                'show_sale': True,
                'link_to_project': self.id,
                'form_view_ref': 'sale_project.sale_order_line_view_form_editable', # Necessary for some logic in the form view
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
                    (self.env.ref('sale_project.view_order_line_tree_with_create').id, 'tree'),
                    (self.env.ref('sale_project.sale_order_line_view_form_editable').id, 'form'),
                ],
            })
        return action_window

    def action_view_sos(self):
        self.ensure_one()
        # Use sudo to avoid AccessErrors when the SOLs belong to different companies.
        all_sale_orders = self._fetch_sale_order_items({'project.task': [('state', 'in', self.env['project.task'].OPEN_STATES)]}).sudo().order_id
        action_window = {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            'name': _("%(name)s's Sales Orders", name=self.name),
            "context": {
                "create": self.env.context.get('create_for_project_id', False),
                "show_sale": True,
                "default_partner_id": self.partner_id.id,
                "default_analytic_account_id": self.analytic_account_id.id,
            },
        }
        if len(all_sale_orders) <= 1:
            action_window.update({
                "res_id": all_sale_orders.id,
                "views": [[False, "form"]],
            })
        else:
            action_window.update({
                "domain": [('id', 'in', all_sale_orders.ids)],
                "views": [[False, "tree"], [False, "kanban"], [False, "calendar"], [False, "pivot"],
                           [False, "graph"], [False, "activity"], [False, "form"]],
            })
        return action_window

    def action_get_list_view(self):
        action = super().action_get_list_view()
        if self.allow_billable:
            action['views'] = [(self.env.ref('sale_project.project_milestone_view_tree').id, 'tree'), (False, 'form')]
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
        query = self.env['account.move.line']._search([('move_id.move_type', 'in', ['out_invoice', 'out_refund'])])
        query.add_where(
            SQL(
                "%s && %s",
                [str(self.analytic_account_id.id)],
                self.env['account.move.line']._query_analytic_accounts(),
            )
        )
        query_string, query_param = query.select('DISTINCT move_id')
        self._cr.execute(query_string, query_param)
        invoice_ids = [line.get('move_id') for line in self._cr.dictfetchall()]
        action = {
            'name': _('Invoices'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'tree'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', invoice_ids)],
            'context': {
                'default_move_type': 'out_invoice',
                'default_partner_id': self.partner_id.id,
                'project_id': self.id,
            }
        }
        if len(invoice_ids) == 1:
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
        query_str, params = self._get_sale_order_items_query(domain_per_model).select('id', 'ARRAY_AGG(DISTINCT sale_line_id) AS sale_line_ids')
        query = f"""
            {query_str}
            GROUP BY id
        """
        self._cr.execute(query, params)
        return {row['id']: self.env['sale.order.line'].browse(row['sale_line_ids']) for row in self._cr.dictfetchall()}

    def _fetch_sale_order_items(self, domain_per_model=None, limit=None, offset=None):
        return self.env['sale.order.line'].browse(self._fetch_sale_order_item_ids(domain_per_model, limit, offset))

    def _fetch_sale_order_item_ids(self, domain_per_model=None, limit=None, offset=None):
        if not self or not self.filtered('allow_billable'):
            return []
        query = self._get_sale_order_items_query(domain_per_model)
        query.limit = limit
        query.offset = offset
        query_str, params = query.select('DISTINCT sale_line_id')
        self._cr.execute(query_str, params)
        return [row[0] for row in self._cr.fetchall()]

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
            ('order_id', 'any', [('analytic_account_id', 'in', self.analytic_account_id.ids)]),
            ('display_type', '=', False),
        ]
        sale_order_line_query = SaleOrderLine._where_calc(sale_order_line_domain)
        sale_order_line_sql = sale_order_line_query.select(
            f'{SaleOrderLine._table}.project_id AS id',
            f'{SaleOrderLine._table}.id AS sale_line_id',
        )

        query = Query(self._cr, 'project_sale_order_item', SQL('(%s)', SQL(' UNION ').join([
            project_sql, task_sql, milestone_sql, sale_order_line_sql,
        ])))
        return query

    def get_panel_data(self):
        panel_data = super().get_panel_data()
        return {
            **panel_data,
            'sale_items': self._get_sale_items() if self.allow_billable else {},
        }

    def get_sale_items_data(self, domain=None, offset=0, limit=None, with_action=True):
        if not self.user_has_groups('project.group_project_user'):
            return {}
        sols = self.env['sale.order.line'].sudo().search(
            domain or self._get_sale_items_domain(),
            offset=offset,
            limit=limit,
        )
        # filter to only get the action for the SOLs that the user can read
        action_per_sol = sols.sudo(False)._filter_access_rules_python('read')._get_action_per_item() if with_action else {}

        def get_action(sol_id):
            """ Return the action vals to call it in frontend if the user can access to the SO related """
            action, res_id = action_per_sol.get(sol_id, (None, None))
            return {'action': {'name': action, 'resId': res_id, 'buttonContext': json.dumps({'active_id': sol_id, 'default_project_id': self.id})}} if action else {}

        return [{
            **sol_read,
            **get_action(sol_read['id']),
        } for sol_read in sols.with_context(with_price_unit=True).read(['display_name', 'product_uom_qty', 'qty_delivered', 'qty_invoiced', 'product_uom'])]

    def _get_sale_items_domain(self, additional_domain=None):
        sale_items = self.sudo()._get_sale_order_items()
        domain = [
            ('order_id', 'in', sale_items.sudo().order_id.ids),
            ('is_downpayment', '=', False),
            ('state', '=', 'sale'),
            ('display_type', '=', False),
            '|',
                '|',
                    ('project_id', 'in', self.ids),
                    ('project_id', '=', False),
                ('id', 'in', sale_items.ids),
        ]
        if additional_domain:
            domain = expression.AND([domain, additional_domain])
        return domain

    def _get_sale_items(self, with_action=True):
        domain = self._get_sale_items_domain()
        return {
            'total': self.env['sale.order.line'].sudo().search_count(domain),
            'data': self.get_sale_items_data(domain, limit=5, with_action=with_action),
        }

    def _show_profitability(self):
        self.ensure_one()
        return self.allow_billable and super()._show_profitability()

    def _show_profitability_helper(self):
        return True

    def _get_profitability_labels(self):
        return {
            **super()._get_profitability_labels(),
            'service_revenues': _lt('Other Services'),
            'materials': _lt('Materials'),
            'other_invoice_revenues': _lt('Customer Invoices'),
            'downpayments': _lt('Down Payments'),
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
                ('product_id', '!=', False),
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
        display_sol_action = with_action and len(self) == 1 and self.user_has_groups('sales_team.group_sale_salesman')
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
                    'to_invoice': -downpayment_amount_invoiced
                }
                if with_action and self.user_has_groups('sales_team.group_sale_salesman_all_leads, account.group_account_invoice, account.group_account_readonly'):
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
                    ._filter_access_rules_python('read')
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
        query = self.env['account.move.line'].sudo()._search(
            self._get_revenues_items_from_invoices_domain([('id', 'not in', excluded_move_line_ids)]),
        )
        query.add_where(
            SQL(
                "%s && %s",
                [str(self.analytic_account_id.id)],
                self.env['account.move.line']._query_analytic_accounts(),
            )
        )
        # account_move_line__move_id is the alias of the joined table account_move in the query
        # we can use it, because of the "move_id.move_type" clause in the domain of the query, which generates the join
        # this is faster than a search_read followed by a browse on the move_id to retrieve the move_type of each account.move.line
        query_string, query_param = query.select('price_subtotal', 'parent_state', 'account_move_line.currency_id', 'account_move_line.analytic_distribution', 'account_move_line__move_id.move_type', 'move_id', 'display_type')
        self._cr.execute(query_string, query_param)
        invoices_move_line_read = self._cr.dictfetchall()
        res = {
            'revenues': {
                'data': [], 'total': {'invoiced': 0.0, 'to_invoice': 0.0}
            },
            'costs': {
                'data': [], 'total': {'billed': 0.0, 'to_bill': 0.0}
            },
        }
        if invoices_move_line_read:
            revenues_lines = []
            cogs_lines = []
            currency_ids = OrderedSet(iml['currency_id'] for iml in invoices_move_line_read)
            for move_line in invoices_move_line_read:
                if move_line['display_type'] == 'cogs':
                    cogs_lines.append(move_line)
                else:
                    revenues_lines.append(move_line)
            for moves, ml_type in ((revenues_lines, 'revenues'), (cogs_lines, 'costs')):
                move_ids = set()
                amount_invoiced = amount_to_invoice = 0.0
                for moves_read in moves:
                    currency = self.env['res.currency'].browse(moves_read['currency_id']).with_prefetch(currency_ids)
                    price_subtotal = currency._convert(moves_read['price_subtotal'], self.currency_id, self.company_id)
                    # an analytic account can appear several time in an analytic distribution with different repartition percentage
                    analytic_contribution = sum(
                        percentage for ids, percentage in moves_read['analytic_distribution'].items()
                        if str(self.analytic_account_id.id) in ids.split(',')
                    ) / 100.
                    move_ids.add(moves_read['move_id'])
                    if moves_read['parent_state'] == 'draft':
                        if moves_read['move_type'] == 'out_invoice':
                            amount_to_invoice += price_subtotal * analytic_contribution
                        else:  # moves_read['move_type'] == 'out_refund'
                            amount_to_invoice -= price_subtotal * analytic_contribution
                    else:  # moves_read['parent_state'] == 'posted'
                        if moves_read['move_type'] == 'out_invoice':
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
                    if with_action and self.user_has_groups('sales_team.group_sale_salesman_all_leads,account.group_account_invoice,account.group_account_readonly'):
                        invoices_items['action'] = self._get_action_for_profitability_section(list(move_ids), section_id)
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
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('sales_team.group_sale_salesman_all_leads'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'dollar',
                'text': _lt('Sales Orders'),
                'number': self_sudo.sale_order_count,
                'action_type': 'object',
                'action': 'action_view_sos',
                'additional_context': json.dumps({
                    'create_for_project_id': self.id,
                }),
                'show': self_sudo.display_sales_stat_buttons and self_sudo.sale_order_count > 0,
                'sequence': 27,
            })
        if self.user_has_groups('sales_team.group_sale_salesman_all_leads'):
            buttons.append({
                'icon': 'dollar',
                'text': _lt('Sales Order Items'),
                'number': self.sale_order_line_count,
                'action_type': 'object',
                'action': 'action_view_sols',
                'show': self.display_sales_stat_buttons,
                'sequence': 28,
            })
        if self.user_has_groups('account.group_account_readonly'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'pencil-square-o',
                'text': _lt('Invoices'),
                'number': self_sudo.invoice_count,
                'action_type': 'object',
                'action': 'action_open_project_invoices',
                'show': bool(self.analytic_account_id) and self_sudo.invoice_count > 0,
                'sequence': 30,
            })
        if self.user_has_groups('account.group_account_readonly'):
            self_sudo = self.sudo()
            buttons.append({
                'icon': 'pencil-square-o',
                'text': _lt('Vendor Bills'),
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
        query = self.env['account.move.line']._search([('move_id.move_type', 'in', ['in_invoice', 'in_refund'])])
        query.add_where(
            SQL(
                "%s && %s",
                [str(self.analytic_account_id.id)],
                self.env['account.move.line']._query_analytic_accounts(),
            )
        )
        query_string, query_param = query.select('DISTINCT move_id')
        self._cr.execute(query_string, query_param)
        vendor_bill_ids = [line.get('move_id') for line in self._cr.dictfetchall()]
        action_window = {
            'name': _('Vendor Bills'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'tree'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', vendor_bill_ids)],
            'context': {
                'default_move_type': 'in_invoice',
                'project_id': self.id,
            }
        }
        if len(vendor_bill_ids) == 1:
            action_window['views'] = [[False, 'form']]
            action_window['res_id'] = vendor_bill_ids[0]
        return action_window

    def action_project_sharing(self):
        action = super().action_project_sharing()
        action['context'].update({
            'sale_show_partner_name': True,
            'hide_partner': self._get_hide_partner(),
        })
        return action

class ProjectTask(models.Model):
    _inherit = "project.task"

    def _domain_sale_line_id(self):
        domain = expression.AND([
            self.env['sale.order.line']._sellable_lines_domain(),
            [
                '|', ('order_partner_id.commercial_partner_id.id', 'parent_of', unquote('partner_id if partner_id else []')),
                    ('order_partner_id', '=?', unquote('partner_id')),
                ('is_service', '=', True), ('is_expense', '=', False), ('state', '=', 'sale'),
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
    task_to_invoice = fields.Boolean("To invoice", compute='_compute_task_to_invoice', search='_search_task_to_invoice', groups='sales_team.group_sale_salesman_all_leads')
    allow_billable = fields.Boolean(related="project_id.allow_billable")

    # Project sharing  fields
    display_sale_order_button = fields.Boolean(string='Display Sales Order', compute='_compute_display_sale_order_button')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | {'allow_billable', 'sale_order_id', 'sale_line_id', 'display_sale_order_button'}

    @api.model
    def _group_expand_sales_order(self, sales_orders, domain, order):
        start_date = self._context.get('gantt_start_date')
        scale = self._context.get('gantt_scale')
        if not (start_date and scale):
            return sales_orders
        search_on_comodel = self._search_on_comodel(domain, "sale_order_id", "sale.order", order)
        if search_on_comodel:
            return search_on_comodel
        return sales_orders

    @api.depends('sale_line_id', 'project_id', 'partner_id.commercial_partner_id', 'allow_billable')
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
            so_partners = (
                sale_order.partner_id
                | sale_order.partner_invoice_id
                | sale_order.partner_shipping_id
            )
            if task.partner_id.commercial_partner_id in so_partners.commercial_partner_id:
                task.sale_order_id = sale_order
            else:
                task.sale_order_id = False

    @api.depends('allow_billable')
    def _compute_partner_id(self):
        billable_task = self.filtered(lambda t: t.allow_billable or (not self._origin and t.parent_id.allow_billable))
        (self - billable_task).partner_id = False
        super(ProjectTask, billable_task)._compute_partner_id()

    @api.depends('partner_id', 'sale_line_id.order_partner_id', 'parent_id.sale_line_id', 'project_id.sale_line_id', 'milestone_id.sale_line_id', 'allow_billable')
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
            # check sale_line_id and customer are coherent
            if task.sale_line_id.order_partner_id.commercial_partner_id != task.partner_id.commercial_partner_id:
                task.sale_line_id = False

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
            "views": [[False, "tree"], [False, "kanban"], [False, "form"]],
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
        query = """
            SELECT so.id
            FROM sale_order so
            WHERE so.invoice_status != 'invoiced'
                AND so.invoice_status != 'no'
        """
        operator_new = 'inselect'
        if(bool(operator == '=') ^ bool(value)):
            operator_new = 'not inselect'
        return [('sale_order_id', operator_new, (query, ()))]

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

class ProjectTaskRecurrence(models.Model):
    _inherit = 'project.task.recurrence'

    @api.model
    def _get_recurring_fields_to_copy(self):
        return super(ProjectTaskRecurrence, self)._get_recurring_fields_to_copy() + ['sale_line_id']

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

    allow_billable = fields.Boolean(related='project_id.allow_billable')
    project_partner_id = fields.Many2one(related='project_id.partner_id')

    sale_line_id = fields.Many2one('sale.order.line', 'Sales Order Item', default=_default_sale_line_id, help='Sales Order Item that will be updated once the milestone is reached.',
        domain="[('order_partner_id', '=?', project_partner_id), ('qty_delivered_method', '=', 'milestones')]")
    quantity_percentage = fields.Float('Quantity (%)', compute="_compute_quantity_percentage", store=True, help='Percentage of the ordered quantity that will automatically be delivered once the milestone is reached.')

    sale_line_display_name = fields.Char("Sale Line Display Name", related='sale_line_id.display_name')
    product_uom = fields.Many2one(related="sale_line_id.product_uom")
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
from collections import defaultdict

from odoo import api, fields, models, _, Command
from odoo.exceptions import UserError
from odoo.osv.expression import AND, NEGATIVE_TERM_OPERATORS, TERM_OPERATORS_NEGATION, ANY_IN


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    tasks_ids = fields.Many2many('project.task', compute='_compute_tasks_ids', search='_search_tasks_ids', string='Tasks associated to this sale')
    tasks_count = fields.Integer(string='Tasks', compute='_compute_tasks_ids', groups="project.group_project_user")

    visible_project = fields.Boolean('Display project', compute='_compute_visible_project', readonly=True)
    project_id = fields.Many2one('project.project', 'Project',
        help='Select a non billable project on which tasks can be created.')
    project_ids = fields.Many2many('project.project', compute="_compute_project_ids", string='Projects', copy=False, groups="project.group_project_user,project.group_project_milestone", help="Projects used in this sales order.")
    project_count = fields.Integer(string='Number of Projects', compute='_compute_project_ids', groups='project.group_project_user')
    milestone_count = fields.Integer(compute='_compute_milestone_count')
    is_product_milestone = fields.Boolean(compute='_compute_is_product_milestone')
    show_create_project_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user')
    show_project_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user')
    show_task_button = fields.Boolean(compute='_compute_show_project_and_task_button', groups='project.group_project_user')

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
            ('product_id.detailed_type', '=', 'service'),
        ], aggregates=['order_id:array_agg'])[0][0]
        for order in self:
            order.show_project_button = order.id in show_button_ids and order.project_count
            order.show_task_button = order.show_project_button or order.tasks_count
            order.show_create_project_button = is_project_manager and order.id in show_button_ids and not order.project_count and order.order_line.product_template_id.filtered(lambda x: x.service_policy in ['delivered_timesheet', 'delivered_milestones'])

    def _search_tasks_ids(self, operator, value):
        if operator in NEGATIVE_TERM_OPERATORS:
            positive_operator = TERM_OPERATORS_NEGATION[operator]
        else:
            positive_operator = operator
        if operator in ANY_IN:
            new_operator = ANY_IN[operator]
            task_domain = AND([value, [('sale_order_id', '!=', False)]])
        else:
            new_operator = 'in' if positive_operator == operator else 'not in'
            task_domain = [('display_name' if isinstance(value, str) else 'id', positive_operator, value), ('sale_order_id', '!=', False)]
        query = self.env['project.task']._search(task_domain)
        return [('id', new_operator, query.subselect('sale_order_id'))]

    @api.depends('order_line.product_id.project_id')
    def _compute_tasks_ids(self):
        tasks_per_so = self.env['project.task']._read_group(
            domain=self._tasks_ids_domain(),
            groupby=['sale_order_id'],
            aggregates=['id:recordset', '__count']
        )
        so_with_tasks = self.env['sale.order']
        for order, tasks_ids, tasks_count in tasks_per_so:
            if order:
                order.tasks_ids = tasks_ids
                order.tasks_count = tasks_count
                so_with_tasks += order
            else:
                # tasks that have no sale_order_id need to be associated with the SO from their sale_line_id
                for task in tasks_ids:
                    task_so = task.sale_line_id.order_id
                    task_so.tasks_ids = [Command.link(task.id)]
                    task_so.tasks_count += 1
                    so_with_tasks += task_so
        remaining_orders = self - so_with_tasks
        if remaining_orders:
            remaining_orders.tasks_ids = [Command.clear()]
            remaining_orders.tasks_count = 0

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
        is_project_manager = self.user_has_groups('project.group_project_manager')
        projects = self.env['project.project'].search([('sale_order_id', 'in', self.ids)])
        projects_per_so = defaultdict(lambda: self.env['project.project'])
        for project in projects:
            projects_per_so[project.sale_order_id.id] |= project
        for order in self:
            projects = order.order_line.mapped('product_id.project_id')
            projects |= order.order_line.mapped('project_id')
            projects |= order.project_id
            projects |= projects_per_so[order.id or order._origin.id]
            if not is_project_manager:
                projects = projects._filter_access_rules('read')
            order.project_ids = projects
            order.project_count = len(projects)

    @api.onchange('project_id')
    def _onchange_project_id(self):
        """ Set the SO analytic account to the selected project's analytic account """
        if self.project_id.analytic_account_id:
            self.analytic_account_id = self.project_id.analytic_account_id

    def _action_confirm(self):
        """ On SO confirmation, some lines should generate a task or a project. """
        result = super()._action_confirm()
        if len(self.company_id) == 1:
            # All orders are in the same company
            self.order_line.sudo().with_company(self.company_id)._timesheet_service_generation()
        else:
            # Orders from different companies are confirmed together
            for order in self:
                order.order_line.sudo().with_company(order.company_id)._timesheet_service_generation()
        return result

    def action_view_task(self):
        self.ensure_one()
        if not self.order_line:
            return {'type': 'ir.actions.act_window_close'}

        list_view_id = self.env.ref('project.view_task_tree2').id
        form_view_id = self.env.ref('project.view_task_form2').id
        kanban_view_id = self.env.ref('project.view_task_kanban_inherit_view_default_project').id

        action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_task")
        if self.tasks_count > 1:  # cross project kanban task
            for idx, (view_id, view_type) in enumerate(action['views']):
                if view_type == 'kanban':
                    action['views'][idx] = (kanban_view_id, 'kanban')
                elif view_type == 'tree':
                    action['views'][idx] = (list_view_id, 'tree')
                elif view_type == 'form':
                    action['views'][idx] = (form_view_id, 'form')
        else:  # 1 or 0 tasks -> form view
            action['views'] = [(form_view_id, 'form')]
            action['res_id'] = self.tasks_ids.id
        # set default project
        default_line = next((sol for sol in self.order_line if sol.product_id.detailed_type == 'service'), self.env['sale.order.line'])
        default_project_id = default_line.project_id.id or self.project_id.id or self.project_ids[:1].id or self.tasks_ids.project_id[:1].id

        action['context'] = {
            'default_sale_order_id': self.id,
            'default_sale_line_id': default_line.id,
            'default_partner_id': self.partner_id.id,
            'default_project_id': default_project_id,
            'default_user_ids': [self.env.uid],
        }
        action['domain'] = AND([ast.literal_eval(action['domain']), self._tasks_ids_domain()])
        return action

    def _tasks_ids_domain(self):
        return ['&', ('project_id', '!=', False), '|', ('sale_line_id', 'in', self.order_line.ids), ('sale_order_id', 'in', self.ids)]

    def action_create_project(self):
        self.ensure_one()
        if not self.order_line:
            return {'type': 'ir.actions.act_window_close'}

        sorted_line = self.order_line.sorted('sequence')
        default_sale_line = next(sol for sol in sorted_line if sol.product_id.detailed_type == 'service')
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
        default_sale_line = next((sol for sol in sorted_line if sol.product_id.detailed_type == 'service'), None)
        action = {
            'type': 'ir.actions.act_window',
            'name': _('Projects'),
            'domain': ['|', ('sale_order_id', '=', self.id), ('id', 'in', self.with_context(active_test=False).project_ids.ids), ('active', 'in', [True, False])],
            'res_model': 'project.project',
            'views': [(False, 'kanban'), (False, 'tree'), (False, 'form')],
            'view_mode': 'kanban,tree,form',
            'context': {
                **self._context,
                'default_partner_id': self.partner_id.id,
                'default_sale_line_id': default_sale_line.id if default_sale_line else False,
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
        default_sale_line = next(sol for sol in sorted_line if sol.is_service and sol.product_id.service_policy == 'delivered_milestones')
        return {
            'type': 'ir.actions.act_window',
            'name': _('Milestones'),
            'domain': [('sale_line_id', 'in', self.order_line.ids)],
            'res_model': 'project.milestone',
            'views': [(self.env.ref('sale_project.sale_project_milestone_view_tree').id, 'tree')],
            'view_mode': 'tree',
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
            if not service_sol:
                raise UserError(_('This Sales Order must contain at least one product of type "Service".'))
            if not project.sale_line_id:
                project.sale_line_id = service_sol
        return created_records

    def write(self, values):
        if 'state' in values and values['state'] == 'cancel':
            self.project_id.sudo().sale_line_id = False
        return super(SaleOrder, self).write(values)

    def _prepare_analytic_account_data(self, prefix=None):
        result = super(SaleOrder, self)._prepare_analytic_account_data(prefix=prefix)
        project_plan, _other_plans = self.env['account.analytic.plan']._get_all_plans()
        result['plan_id'] = project_plan.id or result['plan_id']
        return result

```

## File: models\sale_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, Command, fields, models, _
from odoo.exceptions import AccessError
from odoo.tools import format_amount
from odoo.tools.sql import column_exists, create_column


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    qty_delivered_method = fields.Selection(selection_add=[('milestones', 'Milestones')])
    project_id = fields.Many2one(
        'project.project', 'Generated Project',
        index=True, copy=False)
    task_id = fields.Many2one(
        'project.task', 'Generated Task',
        index=True, copy=False)
    # used to know if generate a task and/or a project, depending on the product settings
    reached_milestones_ids = fields.One2many('project.milestone', 'sale_line_id', string='Reached Milestones', domain=[('is_reached', '=', True)])

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
                    self.env['sale.order'].browse(res['order_id']).check_access_rule('write')
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
                        project_so.check_access_rule('write')
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
        # To get the right product when creating a SOL on the fly, we need to get
        # the name that was entered in the field from the `default_get` method.
        # The easiest way of doing that is to store it in the context.
        if self.env.context.get('form_view_ref') == 'sale_project.sale_order_line_view_form_editable' and not self.env.context.get('action_view_sols'):
            self = self.with_context(sol_product_name=name)
        return super().name_create(name)

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

    @api.model_create_multi
    def create(self, vals_list):
        lines = super().create(vals_list)
        # Do not generate task/project when expense SO line, but allow
        # generate task with hours=0.
        for line in lines:
            if line.state == 'sale' and not line.is_expense:
                has_task = bool(line.task_id)
                line.sudo()._timesheet_service_generation()
                # if the SO line created a task, post a message on the order
                if line.task_id and not has_task:
                    msg_body = _("Task Created (%s): %s", line.product_id.name, line.task_id._get_html_link())
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

    ###########################################
    # Service : Project and task generation
    ###########################################

    def _convert_qty_company_hours(self, dest_company):
        return self.product_uom_qty

    def _timesheet_create_project_prepare_values(self):
        """Generate project values"""
        account = self.order_id.analytic_account_id
        if not account:
            service_products = self.order_id.order_line.product_id.filtered(
                lambda p: p.type == 'service' and p.default_code)
            default_code = service_products.default_code if len(service_products) == 1 else None
            self.order_id._create_analytic_account(prefix=default_code)
            account = self.order_id.analytic_account_id
        # create the project or duplicate one
        return {
            'name': '%s - %s' % (self.order_id.client_order_ref, self.order_id.name) if self.order_id.client_order_ref else self.order_id.name,
            'analytic_account_id': account.id,
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
        if self.product_id.project_template_id:
            values['name'] = "%s - %s" % (values['name'], self.product_id.project_template_id.name)
            # The no_create_folder context key is used in documents_project
            project = self.product_id.project_template_id.with_context(no_create_folder=True).copy(values)
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
            # The no_create_folder context key is used in documents_project
            project = self.env['project.project'].with_context(no_create_folder=True).create(values)

        # Avoid new tasks to go to 'Undefined Stage'
        if not project.type_ids:
            project.type_ids = self.env['project.task.type'].create([{
                'name': name,
                'fold': fold,
                'sequence': sequence,
            } for name, fold, sequence in [
                (_('To Do'), False, 5),
                (_('In Progress'), False, 10),
                (_('Done'), True, 15),
                (_('Canceled'), True, 20),
            ]])

        # link project as generated by current so line
        self.write({'project_id': project.id})
        return project

    def _timesheet_create_task_prepare_values(self, project):
        self.ensure_one()
        allocated_hours = 0.0
        if self.product_id.service_type not in ['milestones', 'manual']:
            allocated_hours = self._convert_qty_company_hours(self.company_id)
        sale_line_name_parts = self.name.split('\n')
        title = sale_line_name_parts[0] or self.product_id.name
        description = '<br/>'.join(sale_line_name_parts[1:])
        return {
            'name': title if project.sale_line_id else '%s - %s' % (self.order_id.name or '', title),
            'analytic_account_id': project.analytic_account_id.id,
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
        self.write({'task_id': task.id})
        # post message on task
        task_msg = _("This task has been created from: %s (%s)",
            self.order_id._get_html_link(),
            self.product_id.name
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

        def _determine_project(so_line):
            """Determine the project for this sale order line.
            Rules are different based on the service_tracking:

            - 'project_only': the project_id can only come from the sale order line itself
            - 'task_in_project': the project_id comes from the sale order line only if no project_id was configured
              on the parent sale order"""

            if so_line.product_id.service_tracking == 'project_only':
                return so_line.project_id
            elif so_line.product_id.service_tracking == 'task_in_project':
                return so_line.order_id.project_id or so_line.project_id

            return False

        # task_global_project: create task in global project
        for so_line in so_line_task_global_project:
            if not so_line.task_id:
                if map_sol_project.get(so_line.id) and so_line.product_uom_qty > 0:
                    so_line._timesheet_create_task(project=map_sol_project[so_line.id])

        # project_only, task_in_project: create a new project, based or not on a template (1 per SO). May be create a task too.
        # if 'task_in_project' and project_id configured on SO, use that one instead
        for so_line in so_line_new_project:
            project = _determine_project(so_line)
            if not project and _can_create_project(so_line):
                project = so_line._timesheet_create_project()
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
            so_line._generate_milestone()

    def _generate_milestone(self):
        if self.product_id.service_policy == 'delivered_milestones':
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
            task_analytic_account = self.task_id._get_task_analytic_account_id() if self.task_id else False
            if task_analytic_account:
                values['analytic_distribution'] = {task_analytic_account.id: 100}
            elif self.project_id.analytic_account_id:
                values['analytic_distribution'] = {self.project_id.analytic_account_id.id: 100}
            elif self.is_service and not self.is_expense:
                [task_analytic_accounts] = self.env['project.task']._read_group([
                    ('sale_line_id', '=', self.id),
                    ('analytic_account_id', '!=', False),
                ], aggregates=['analytic_account_id:recordset'])[0]
                [project_analytic_accounts] = self.env['project.project']._read_group([
                    ('analytic_account_id', '!=', False),
                    '|',
                        ('sale_line_id', '=', self.id),
                        '&',
                            ('tasks.sale_line_id', '=', self.id),
                            ('tasks.analytic_account_id', '=', False)
                ], aggregates=['analytic_account_id:recordset'])[0]
                analytic_accounts = task_analytic_accounts | project_analytic_accounts
                if len(analytic_accounts) == 1:
                    values['analytic_distribution'] = {analytic_accounts.id: 100}
        return values

    def _get_action_per_item(self):
        """ Get action per Sales Order Item

            :returns: Dict containing id of SOL as key and the action as value
        """
        return {}

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
# -*- coding: utf-8 -*-

from . import account_move
from . import product
from . import project
from . import project_milestone
from . import sale_order
from . import sale_order_line
from . import sale_order_template_line
from . import res_config_settings

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

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_report

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
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { formatFloat, formatFloatTime } from "@web/views/fields/formatters";
import { ProjectRightSidePanel } from '@project/components/project_right_side_panel/project_right_side_panel';

patch(ProjectRightSidePanel.prototype, {
    async _loadAdditionalSalesOrderItems() {
        const offset = this.state.data.sale_items.data.length;
        const totalRecords = this.state.data.sale_items.total;
        const limit = totalRecords - offset <= 5 ? totalRecords - offset : 5;
        const saleOrderItems = await this.orm.call(
            'project.project',
            'get_sale_items_data',
            [this.projectId, undefined, offset, limit],
            {
                context: this.context,
            },
        );
        this.state.data.sale_items.data = [...this.state.data.sale_items.data, ...saleOrderItems];
    },

    async onLoadSalesOrderLinesClick() {
        const saleItems = this.state.data.sale_items;
        if (saleItems && saleItems.total > saleItems.data.length) {
            await this._loadAdditionalSalesOrderItems();
        }
    },

    formatValue(value, unit) {
        return unit === 'Hours' ? formatFloatTime(value) : formatFloat(value);
    },

    //---------------------------------------------------------------------
    // Handlers
    //---------------------------------------------------------------------

    /**
     * @private
     * @param {Object} params
     */
    async onSaleItemActionClick(params) {
        if (params.resId && params.type !== 'object') {
            const action = await this.actionService.loadAction(params.name, this.context);
            this.actionService.doAction({
                ...action,
                res_id: params.resId,
                views: [[false, 'form']]
            });
        } else {
            this.onProjectActionClick(params);
        }
    },

});

```

## File: static\src\components\project_right_side_panel\project_right_side_panel.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="sale_project.ProjectRightSidePanel" t-inherit="project.ProjectRightSidePanel" t-inherit-mode="extension">
        <xpath expr="//ProjectRightSidePanelSection[@name=&quot;'profitability'&quot;]" position="before">
            <ProjectRightSidePanelSection
                name="'sales'"
                show="!!state.data.sale_items?.data"
                dataClassName="'py-3'"
            >
                <t t-set-slot="title">
                    Sales
                </t>
                <span t-if="!state.data.sale_items.data.length" class="text-muted fst-italic">
                    Track what you sold, delivered, and invoiced.
                </span>
                <table t-else="" class="table table-sm table-striped table-hover mb-0">
                    <thead class="bg-100">
                        <tr>
                            <th>Sales Order Items</th>
                            <th class="text-end">Sold</th>
                            <th class="text-end">Delivered</th>
                            <th class="text-end">Invoiced</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr t-foreach="state.data.sale_items.data" t-as="sale_item" t-key="sale_item.id">
                            <t t-set="uom_name" t-value="sale_item.product_uom and sale_item.product_uom[1]"/>
                            <td>
                                <t t-set="sol_name" t-value="sale_item.display_name"/>
                                <a t-if="sale_item.action" class="o_rightpanel_button" href="#" t-on-click="() => this.onSaleItemActionClick(sale_item.action)">
                                    <t t-esc="sol_name"/>
                                </a>
                                <t t-else="" t-esc="sol_name"/>
                            </td>
                            <td class="text-end align-middle"><t t-esc="formatValue(sale_item.product_uom_qty, uom_name)"/> <t t-esc="uom_name"/></td>
                            <td class="text-end align-middle"><t t-esc="formatValue(sale_item.qty_delivered, uom_name)"/> <t t-esc="uom_name"/></td>
                            <td class="text-end align-middle"><t t-esc="formatValue(sale_item.qty_invoiced, uom_name)"/> <t t-esc="uom_name"/></td>
                        </tr>
                    </tbody>
                    <tfoot>
                        <tr class="o_rightpanel_nohover border-0" t-if="state.data.sale_items.total &gt; state.data.sale_items.data.length">
                            <td class="py-3 border-bottom text-center" colspan="4">
                                <a class="btn btn-link w-100" t-on-click="onLoadSalesOrderLinesClick">
                                    Load more
                                </a>
                            </td>
                        </tr>
                    </tfoot>
                </table>
            </ProjectRightSidePanelSection>
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
                <attribute name="invisible">type == 'service' and sale_ok == True</attribute>
            </field>
            <field name="invoice_policy" position="after">
                <field name="service_policy" string="Invoicing Policy" invisible="type != 'service' or sale_ok == False" required="type == 'service' and sale_ok == True"/>
                <field name="service_tracking" required="1" invisible="type != 'service' or not sale_ok"/>
                <div class="o_td_label d-inline-flex" invisible="service_tracking != 'task_global_project'">
                    <label for='project_id'/>
                    <span class='fa fa-lg fa-building-o fa-fw' title="Values set here are company-specific." groups="base.group_multi_company"/>
                </div>
                <field name="project_id" context="{'default_allow_billable': True}" invisible="service_tracking != 'task_global_project'" required="service_tracking == 'task_global_project'" nolabel="1"/>
                <div class="o_td_label d-inline-flex" invisible="service_tracking not in ['task_in_project', 'project_only']">
                    <label for='project_template_id'/>
                    <span class='fa fa-lg fa-building-o fa-fw' title="Values set here are company-specific." groups="base.group_multi_company"/>
                </div>
                <field name="project_template_id" context="{'active_test': False, 'default_allow_billable': True}" invisible="service_tracking not in ['task_in_project', 'project_only']" nolabel="1"/>
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
            <xpath expr="//field[@name='child_ids']/tree/field[@name='partner_id']" position="after">
                <field name="allow_billable" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/tree/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
        </field>
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
        <field name="context">{'group_by_no_leaf':1, 'group_by':[], 'graph_measure': '__count__', 'sale_show_partner_name': True}</field>
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
        <field name="name">project.project.tree.inherit.sale.project</field>
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
            <xpath expr="//page[@name='settings']//field[@name='analytic_account_id']" position="before">
                <field name="sale_line_id"
                    groups="!sales_team.group_sale_salesman"
                    options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"
                    invisible="not allow_billable or not partner_id"/>
                <field name="sale_line_id"
                    groups="sales_team.group_sale_salesman"
                    options="{'no_edit': True, 'delete': False}"
                    invisible="not allow_billable or not partner_id"
                    context="{
                        'form_view_ref': 'sale_project.sale_order_line_view_form_editable',
                        'default_partner_id': partner_id,
                        'default_company_id': company_id,
                    }"/>
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
                <field name="sale_line_id"
                    groups="!sales_team.group_sale_salesman"
                    string="Sales Order Item"
                    options='{"no_open": True}'
                    readonly="1"
                    invisible="not allow_billable or not project_id or not sale_line_id or not partner_id"/>
                <field name="sale_line_id"
                    groups="sales_team.group_sale_salesman"
                    string="Sales Order Item"
                    context="{
                        'create': False, 'edit': False, 'delete': False,
                        'with_price_unit': True,
                        'form_view_ref': 'sale_project.sale_order_line_view_form_editable',
                        'default_partner_id': partner_id,
                        'default_company_id': company_id,
                        'default_order_id': project_sale_order_id,
                    }"
                    placeholder="Non-billable"
                    readonly="0"
                    invisible="not allow_billable or not project_id or not partner_id"/>
                <field name="allow_billable" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='child_ids']/tree/field[@name='partner_id']" position="after">
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
            <xpath expr="//field[@name='child_ids']/tree/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/tree/field[@name='partner_id']" position="after">
                <field name="sale_line_id" optional="hide" readonly="1" groups="sales_team.group_sale_salesman"/>
                <field name="sale_line_id" optional="hide" options="{'no_open': True}" readonly="1" groups="!sales_team.group_sale_salesman"/>
                <field name="allow_billable" column_invisible="True"/>
            </xpath>
            <xpath expr="//field[@name='depend_on_ids']/tree/field[@name='partner_id']" position="attributes">
                <attribute name="column_invisible">not parent.allow_billable</attribute>
                <attribute name="invisible">not allow_billable</attribute>
            </xpath>
            <field name="project_id" position="attributes">
                <attribute name="context">{'default_allow_billable': True}</attribute>
            </field>
        </field>
    </record>

    <record id="project_task_view_tree_main_base" model="ir.ui.view">
        <field name="name">project.task.main.tree.inherit</field>
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
                <field name="sale_line_id" optional="hide" readonly="1" groups="sales_team.group_sale_salesman"/>
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
        <field name="name">project.milestone.view.tree.inherit</field>
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
        <field name="name">project.milestone.view.tree.inherit</field>
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

    <record id="project.open_view_project_all_config" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True}</field>
    </record>
    <record id="project.open_view_project_all_config_group_stage" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True}</field>
    </record>
    <record id="project.open_view_project_all" model="ir.actions.act_window">
        <field name="context">{'default_allow_billable': True, 'sale_show_partner_name': True}</field>
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
        <field name="name">sale.order.line.tree.with.create</field>
        <field name="model">sale.order.line</field>
        <field name="inherit_id" ref="sale.view_order_line_tree"/>
        <field name="mode">primary</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <tree position="attributes">
                <attribute name="create">true</attribute>
            </tree>
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
                    'default_detailed_type': 'service',
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
            <button id="create_invoice_percentage" position="after">
                <button id="create_project" name="action_create_project" string="Create Project"
                    type="object" data-hotkey="q"
                    invisible="not show_create_project_button"
                    groups="project.group_project_manager"/>
            </button>
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
                <button type="object" name="action_view_task" class="oe_stat_button" icon="fa-tasks" invisible="not show_task_button" groups="project.group_project_user">
                    <field name="tasks_count" widget="statinfo" string="Tasks"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='analytic_account_id']" position="after">
                <field name="visible_project" invisible="1"/>
                <field name="project_id" options="{'no_create': True}" invisible="not visible_project" readonly="state not in ['draft', 'sent']"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_project_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="portal_tasks_list_inherit" inherit_id="project.portal_tasks_list">
        <xpath expr="//t[@t-foreach='grouped_tasks']/tbody/tr[hasclass('table-light')]" position="inside">
            <th t-if="groupby == 'sale_order'" t-attf-colspan="{{grouped_tasks_colspan}}">
                <span t-if="tasks[0].sudo().sale_order_id" class="text-truncate" t-field="tasks[0].sudo().sale_order_id"/>
                <span t-else="">No Sales Order</span>
            </th>
            <th t-if="groupby == 'sale_line'" t-attf-colspan="{{grouped_tasks_colspan}}">
                <span t-if="tasks[0].sudo().sale_line_id" class="text-truncate" t-field="tasks[0].sudo().sale_line_id"/>
                <span t-else="">Not Billed</span>
            </th>
        </xpath>
    </template>

</odoo>

```

