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
    'depends': ['sale_management', 'project'],
    'data': [
        'security/ir.model.access.csv',
        'security/sale_project_security.xml',
        'report/project_report_views.xml',
        'views/product_views.xml',
        'views/project_task_views.xml',
        'views/sale_order_views.xml',
        'views/sale_project_portal_templates.xml',
        'views/project_sharing_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from operator import itemgetter

from odoo import _
from odoo.http import request
from odoo.tools import groupby as groupbyelem

from odoo.osv.expression import OR

from odoo.addons.project.controllers.portal import ProjectCustomerPortal


class SaleProjectCustomerPortal(ProjectCustomerPortal):

    def _task_get_searchbar_groupby(self):
        values = super()._task_get_searchbar_groupby()
        values['sale_order'] = {'input': 'sale_order', 'label': _('Sales Order'), 'order': 7}
        values['sale_line'] = {'input': 'sale_line', 'label': _('Sales Order Item'), 'order': 8}
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_groupby_mapping(self):
        groupby_mapping = super()._task_get_groupby_mapping()
        groupby_mapping.update(sale_order='sale_order_id', sale_line='sale_line_id')
        return groupby_mapping

    def _task_get_searchbar_inputs(self):
        values = super()._task_get_searchbar_inputs()
        values['sale_order'] = {'input': 'sale_order', 'label': _('Search in Sales Order'), 'order': 7}
        values['sale_line'] = {'input': 'sale_line', 'label': _('Search in Sales Order Item'), 'order': 8}
        values['invoice'] = {'input': 'invoice', 'label': _('Search in Invoice'), 'order': 9}
        return dict(sorted(values.items(), key=lambda item: item[1]["order"]))

    def _task_get_search_domain(self, search_in, search):
        search_domain = [super()._task_get_search_domain(search_in, search)]
        if search_in in ('sale_order', 'all'):
            search_domain.append([('sale_order_id.name', 'ilike', search)])
        if search_in in ('sale_line', 'all'):
            search_domain.append([('sale_line_id.name', 'ilike', search)])
        if search_in in ('invoice', 'all'):
            search_domain.append([('sale_order_id.invoice_ids.name', 'ilike', search)])
        return OR(search_domain)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ProductTemplate(models.Model):
    _inherit = 'product.template'

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
        domain="[('company_id', '=', current_company_id)]",
        help='Select a billable project on which tasks can be created. This setting must be set for each company.')
    project_template_id = fields.Many2one(
        'project.project', 'Project Template', company_dependent=True, copy=True,
        domain="[('company_id', '=', current_company_id)]",
        help='Select a billable project to be the skeleton of the new created project when selling the current product. Its stages and tasks will be duplicated.')

    @api.depends('service_tracking', 'type')
    def _compute_product_tooltip(self):
        super()._compute_product_tooltip()
        for record in self.filtered(lambda record: record.type == 'service'):
            if record.service_tracking == 'no':
                record.product_tooltip = _(
                    "Invoice ordered quantities as soon as this service is sold."
                )
            elif record.service_tracking == 'task_global_project':
                record.product_tooltip = _(
                    "Invoice as soon as this service is sold, and create a task in an existing "
                    "project to track the time spent."
                )
            elif record.service_tracking == 'task_in_project':
                record.product_tooltip = _(
                    "Invoice ordered quantities as soon as this service is sold, and create a "
                    "project for the order with a task for each sales order line to track the time"
                    " spent."
                )
            elif record.service_tracking == 'project_only':
                record.product_tooltip = _(
                    "Invoice ordered quantities as soon as this service is sold, and create an "
                    "empty project for the order to track the time spent."
                )

    @api.constrains('project_id', 'project_template_id')
    def _check_project_and_template(self):
        """ NOTE 'service_tracking' should be in decorator parameters but since ORM check constraints twice (one after setting
            stored fields, one after setting non stored field), the error is raised when company-dependent fields are not set.
            So, this constraints does cover all cases and inconsistent can still be recorded until the ORM change its behavior.
        """
        for product in self:
            if product.service_tracking == 'no' and (product.project_id or product.project_template_id):
                raise ValidationError(_('The product %s should not have a project nor a project template since it will not generate project.') % (product.name,))
            elif product.service_tracking == 'task_global_project' and product.project_template_id:
                raise ValidationError(_('The product %s should not have a project template since it will generate a task in a global project.') % (product.name,))
            elif product.service_tracking in ['task_in_project', 'project_only'] and product.project_id:
                raise ValidationError(_('The product %s should not have a global project since it will generate a project.') % (product.name,))

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models, _, _lt
from odoo.exceptions import ValidationError, UserError
from odoo.osv import expression
from odoo.osv.query import Query


class Project(models.Model):
    _inherit = 'project.project'

    sale_line_id = fields.Many2one(
        'sale.order.line', 'Sales Order Item', copy=False,
        compute="_compute_sale_line_id", store=True, readonly=False, index=True,
        domain="[('is_service', '=', True), ('is_expense', '=', False), ('state', 'in', ['sale', 'done']), ('order_partner_id', '=?', partner_id), '|', ('company_id', '=', False), ('company_id', '=', company_id)]",
        help="Sales order item to which the project is linked. Link the timesheet entry to the sales order item defined on the project. "
        "Only applies on tasks without sale order item defined, and if the employee is not in the 'Employee/Sales Order Item Mapping' of the project.")
    sale_order_id = fields.Many2one(string='Sales Order', related='sale_line_id.order_id', help="Sales order to which the project is linked.")
    has_any_so_to_invoice = fields.Boolean('Has SO to Invoice', compute='_compute_has_any_so_to_invoice')

    @api.model
    def _map_tasks_default_valeus(self, task, project):
        defaults = super()._map_tasks_default_valeus(task, project)
        defaults['sale_line_id'] = False
        return defaults

    @api.depends('partner_id')
    def _compute_sale_line_id(self):
        self.filtered(
            lambda p:
                p.sale_line_id and (
                    not p.partner_id or p.sale_line_id.order_partner_id.commercial_partner_id != p.partner_id.commercial_partner_id
                )
        ).update({'sale_line_id': False})

    @api.depends('sale_order_id.invoice_status', 'tasks.sale_order_id.invoice_status')
    def _compute_has_any_so_to_invoice(self):
        """Has any Sale Order whose invoice_status is set as To Invoice"""
        if not self.ids:
            self.has_any_so_to_invoice = False
            return

        self.env.cr.execute("""
            SELECT id
              FROM project_project pp
             WHERE pp.active = true
               AND (   EXISTS(SELECT 1
                                FROM sale_order so
                                JOIN project_task pt ON pt.sale_order_id = so.id
                               WHERE pt.project_id = pp.id
                                 AND pt.active = true
                                 AND so.invoice_status = 'to invoice')
                    OR EXISTS(SELECT 1
                                FROM sale_order so
                                JOIN sale_order_line sol ON sol.order_id = so.id
                               WHERE sol.id = pp.sale_line_id
                                 AND so.invoice_status = 'to invoice'))
               AND id in %s""", (tuple(self.ids),))
        project_to_invoice = self.env['project.project'].browse([x[0] for x in self.env.cr.fetchall()])
        project_to_invoice.has_any_so_to_invoice = True
        (self - project_to_invoice).has_any_so_to_invoice = False

    def action_view_so(self):
        self.ensure_one()
        action_window = {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            "name": "Sales Order",
            "views": [[False, "form"]],
            "context": {"create": False, "show_sale": True},
            "res_id": self.sale_order_id.id
        }
        return action_window

    def action_create_invoice(self):
        if not self.has_any_so_to_invoice:
            raise UserError(_("There is nothing to invoice in this project."))
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_view_sale_advance_payment_inv")
        so_ids = (self.sale_order_id | self.task_ids.sale_order_id).filtered(lambda so: so.invoice_status == 'to invoice').ids
        action['context'] = {
            'active_id': so_ids[0] if len(so_ids) == 1 else False,
            'active_ids': so_ids
        }
        return action

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def _get_sale_order_stat_button(self):
        self.ensure_one()
        return {
            'icon': 'dollar',
            'text': _lt('Sales Order'),
            'action_type': 'object',
            'action': 'action_view_so',
            'show': bool(self.sale_order_id),
            'sequence': 1,
        }

    def _fetch_sale_order_items(self, domain_per_model=None, limit=None, offset=None):
        return self.env['sale.order.line'].browse(self._fetch_sale_order_item_ids(domain_per_model, limit, offset))

    def _fetch_sale_order_item_ids(self, domain_per_model=None, limit=None, offset=None):
        if not self:
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
        project_domain = [('id', 'in', self.ids), ('sale_line_id', '!=', False)]
        if 'project.project' in domain_per_model:
            project_domain = expression.AND([project_domain, domain_per_model['project.project']])
        project_query = self.env['project.project']._where_calc(project_domain)
        self._apply_ir_rules(project_query, 'read')
        project_query_str, project_params = project_query.select('id', 'sale_line_id')

        Task = self.env['project.task']
        task_domain = [('project_id', 'in', self.ids), ('sale_line_id', '!=', False)]
        if Task._name in domain_per_model:
            task_domain = expression.AND([task_domain, domain_per_model[Task._name]])
        task_query = Task._where_calc(task_domain)
        Task._apply_ir_rules(task_query, 'read')
        task_query_str, task_params = task_query.select(f'{Task._table}.project_id AS id', f'{Task._table}.sale_line_id')

        query = Query(self._cr, 'project_sale_order_item', ' UNION '.join([project_query_str, task_query_str]))
        query._where_params = project_params + task_params
        return query

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('sales_team.group_sale_salesman_all_leads'):
            buttons.append(self._get_sale_order_stat_button())
        return buttons

class ProjectTask(models.Model):
    _inherit = "project.task"

    sale_order_id = fields.Many2one('sale.order', 'Sales Order', compute='_compute_sale_order_id', store=True, help="Sales order to which the task is linked.")
    sale_line_id = fields.Many2one(
        'sale.order.line', 'Sales Order Item', domain="[('company_id', '=', company_id), ('is_service', '=', True), ('order_partner_id', 'child_of', commercial_partner_id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done'])]",
        compute='_compute_sale_line', recursive=True, store=True, readonly=False, copy=False, tracking=True, index=True,
        help="Sales Order Item to which the time spent on this task will be added, in order to be invoiced to your customer.")
    project_sale_order_id = fields.Many2one('sale.order', string="Project's sale order", related='project_id.sale_order_id')
    invoice_count = fields.Integer("Number of invoices", related='sale_order_id.invoice_count')
    task_to_invoice = fields.Boolean("To invoice", compute='_compute_task_to_invoice', search='_search_task_to_invoice', groups='sales_team.group_sale_salesman_all_leads')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | {'sale_order_id', 'sale_line_id'}

    @api.depends('sale_line_id', 'project_id', 'commercial_partner_id')
    def _compute_sale_order_id(self):
        for task in self:
            sale_order_id = task.sale_order_id or self.env["sale.order"]
            if task.sale_line_id:
                sale_order_id = task.sale_line_id.sudo().order_id
            elif task.project_id.sale_order_id:
                sale_order_id = task.project_id.sale_order_id
            if task.commercial_partner_id != sale_order_id.partner_id.commercial_partner_id:
                sale_order_id = False
            if sale_order_id and not task.partner_id:
                task.partner_id = sale_order_id.partner_id
            task.sale_order_id = sale_order_id

    @api.depends('commercial_partner_id', 'sale_line_id.order_partner_id.commercial_partner_id', 'parent_id.sale_line_id', 'project_id.sale_line_id')
    def _compute_sale_line(self):
        for task in self:
            if not task.sale_line_id:
                # if the display_project_id is set then it means the task is classic task or a subtask with another project than its parent.
                task.sale_line_id = task.display_project_id.sale_line_id or task.parent_id.sale_line_id or task.project_id.sale_line_id
            # check sale_line_id and customer are coherent
            if task.sale_line_id.order_partner_id.commercial_partner_id != task.partner_id.commercial_partner_id:
                task.sale_line_id = False

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

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_so(self):
        if any(task.sale_line_id for task in self):
            raise ValidationError(_('You have to unlink the task from the sale order item in order to delete it.'))

    # ---------------------------------------------------
    # Actions
    # ---------------------------------------------------

    def _get_action_view_so_ids(self):
        return self.sale_order_id.ids

    def action_view_so(self):
        self.ensure_one()
        so_ids = self._get_action_view_so_ids()
        action_window = {
            "type": "ir.actions.act_window",
            "res_model": "sale.order",
            "name": "Sales Order",
            "views": [[False, "tree"], [False, "form"]],
            "context": {"create": False, "show_sale": True},
            "domain": [["id", "in", so_ids]],
        }
        if len(so_ids) == 1:
            action_window["views"] = [[False, "form"]]
            action_window["res_id"] = so_ids[0]

        return action_window

    def rating_get_partner_id(self):
        partner = self.partner_id or self.sale_line_id.order_id.partner_id
        if partner:
            return partner
        return super().rating_get_partner_id()

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


class ProjectTaskRecurrence(models.Model):
    _inherit = 'project.task.recurrence'

    def _new_task_values(self, task):
        values = super(ProjectTaskRecurrence, self)._new_task_values(task)
        task = self.sudo().task_ids[0]
        values['sale_line_id'] = self._get_sale_line_id(task)
        return values

    def _get_sale_line_id(self, task):
        return task.sale_line_id.id

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, fields, models, _

from odoo.tools.misc import clean_context
from odoo.tools.safe_eval import safe_eval
from odoo.tools.sql import column_exists, create_column


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    tasks_ids = fields.Many2many('project.task', compute='_compute_tasks_ids', string='Tasks associated to this sale')
    tasks_count = fields.Integer(string='Tasks', compute='_compute_tasks_ids', groups="project.group_project_user")

    visible_project = fields.Boolean('Display project', compute='_compute_visible_project', readonly=True)
    project_id = fields.Many2one(
        'project.project', 'Project', readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        help='Select a non billable project on which tasks can be created.')
    project_ids = fields.Many2many('project.project', compute="_compute_project_ids", string='Projects', copy=False, groups="project.group_project_manager", help="Projects used in this sales order.")
    project_count = fields.Integer(string='Number of Projects', compute='_compute_project_ids', groups='project.group_project_manager')

    @api.depends('order_line.product_id.project_id')
    def _compute_tasks_ids(self):
        for order in self:
            order.tasks_ids = self.env['project.task'].search(['&', ('display_project_id', '!=', False), '|', ('sale_line_id', 'in', order.order_line.ids), ('sale_order_id', '=', order.id)])
            order.tasks_count = len(order.tasks_ids)

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
        for order in self:
            projects = order.order_line.mapped('product_id.project_id')
            projects |= order.order_line.mapped('project_id')
            projects |= order.project_id
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
        context = clean_context(self._context)
        if len(self.company_id) == 1:
            # All orders are in the same company
            self.order_line.sudo().with_context(context).with_company(self.company_id)._timesheet_service_generation()
        else:
            # Orders from different companies are confirmed together
            for order in self:
                order.order_line.sudo().with_context(context).with_company(order.company_id)._timesheet_service_generation()
        return result

    def action_view_task(self):
        self.ensure_one()

        list_view_id = self.env.ref('project.view_task_tree2').id
        form_view_id = self.env.ref('project.view_task_form2').id

        action = {'type': 'ir.actions.act_window_close'}
        task_projects = self.tasks_ids.mapped('project_id')
        if len(task_projects) == 1 and len(self.tasks_ids) > 1:  # redirect to task of the project (with kanban stage, ...)
            action = self.with_context(active_id=task_projects.id).env['ir.actions.actions']._for_xml_id(
                'project.act_project_project_2_project_task_all')
            action['domain'] = [('id', 'in', self.tasks_ids.ids)]
            if action.get('context'):
                eval_context = self.env['ir.actions.actions']._get_eval_context()
                eval_context.update({'active_id': task_projects.id})
                action_context = safe_eval(action['context'], eval_context)
                action_context.update(eval_context)
                action['context'] = action_context
        else:
            action = self.env["ir.actions.actions"]._for_xml_id("project.action_view_task")
            action['context'] = {}  # erase default context to avoid default filter
            if len(self.tasks_ids) > 1:  # cross project kanban task
                action['views'] = [[False, 'kanban'], [list_view_id, 'tree'], [form_view_id, 'form'], [False, 'graph'], [False, 'calendar'], [False, 'pivot']]
            elif len(self.tasks_ids) == 1:  # single task -> form view
                action['views'] = [(form_view_id, 'form')]
                action['res_id'] = self.tasks_ids.id
        # filter on the task of the current SO
        action.setdefault('context', {})
        action['context'].update({'search_default_sale_order_id': self.id})
        return action

    def action_view_project_ids(self):
        self.ensure_one()
        view_form_id = self.env.ref('project.edit_project').id
        view_kanban_id = self.env.ref('project.view_project_kanban').id
        action = {
            'type': 'ir.actions.act_window',
            'domain': [('id', 'in', self.with_context(active_test=False).project_ids.ids), ('active', 'in', [True, False])],
            'view_mode': 'kanban,form',
            'name': _('Projects'),
            'res_model': 'project.project',
        }
        if len(self.with_context(active_test=False).project_ids) == 1:
            action.update({'views': [(view_form_id, 'form')], 'res_id': self.project_ids.id})
        else:
            action['views'] = [(view_kanban_id, 'kanban'), (view_form_id, 'form')]
        return action

    def write(self, values):
        if 'state' in values and values['state'] == 'cancel':
            self.project_id.sudo().sale_line_id = False
        return super(SaleOrder, self).write(values)

    def _compute_line_data_for_template_change(self, line):
        data = super()._compute_line_data_for_template_change(line)
        # prevent the association of a related task on the SOL if a task would be generated when confirming the SO.
        if 'default_task_id' in self.env.context and line.product_id.service_tracking in ['task_in_project', 'task_global_project']:
            data['task_id'] = False
        return data


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    project_id = fields.Many2one(
        'project.project', 'Generated Project',
        index=True, copy=False, help="Project generated by the sales order item")
    task_id = fields.Many2one(
        'project.task', 'Generated Task',
        index=True, copy=False, help="Task generated by the sales order item")
    is_service = fields.Boolean("Is a Service", compute='_compute_is_service', store=True, compute_sudo=True, help="Sales Order item should generate a task and/or a project, depending on the product settings.")

    @api.depends('product_id.type')
    def _compute_is_service(self):
        for so_line in self:
            so_line.is_service = so_line.product_id.type == 'service'

    @api.depends('product_id.type')
    def _compute_product_updatable(self):
        for line in self:
            if line.product_id.type == 'service' and line.state == 'sale':
                line.product_updatable = False
            else:
                super(SaleOrderLine, line)._compute_product_updatable()

    def _auto_init(self):
        """
        Create column to stop ORM from computing it himself (too slow)
        """
        if not column_exists(self.env.cr, 'sale_order_line', 'is_service'):
            create_column(self.env.cr, 'sale_order_line', 'is_service', 'bool')
            self.env.cr.execute("""
                UPDATE sale_order_line line
                SET is_service = (pt.type = 'service')
                FROM product_product pp
                LEFT JOIN product_template pt ON pt.id = pp.product_tmpl_id
                WHERE pp.id = line.product_id
            """)
        return super()._auto_init()

    @api.model_create_multi
    def create(self, vals_list):
        lines = super().create(vals_list)
        # Do not generate task/project when expense SO line, but allow
        # generate task with hours=0.
        context = clean_context(self._context)
        for line in lines:
            if line.state == 'sale' and not line.is_expense:
                line.sudo().with_context(context)._timesheet_service_generation()
                # if the SO line created a task, post a message on the order
                if line.task_id:
                    msg_body = _("Task Created (%s): <a href=# data-oe-model=project.task data-oe-id=%d>%s</a>") % (line.product_id.name, line.task_id.id, line.task_id.name)
                    line.order_id.message_post(body=msg_body)
        return lines

    def write(self, values):
        result = super().write(values)
        # changing the ordered quantity should change the planned hours on the
        # task, whatever the SO state. It will be blocked by the super in case
        # of a locked sale order.
        if 'product_uom_qty' in values and not self.env.context.get('no_update_planned_hours', False):
            for line in self:
                if line.task_id and line.product_id.type == 'service':
                    planned_hours = line._convert_qty_company_hours(line.task_id.company_id)
                    line.task_id.write({'planned_hours': planned_hours})
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
            self.order_id._create_analytic_account(prefix=self.product_id.default_code or None)
            account = self.order_id.analytic_account_id

        # create the project or duplicate one
        return {
            'name': '%s - %s' % (self.order_id.client_order_ref, self.order_id.name) if self.order_id.client_order_ref else self.order_id.name,
            'analytic_account_id': account.id,
            'partner_id': self.order_id.partner_id.id,
            'sale_line_id': self.id,
            'active': True,
            'company_id': self.company_id.id,
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
            project = self.product_id.project_template_id.copy(values)
            project.tasks.write({
                'sale_line_id': self.id,
                'partner_id': self.order_id.partner_id.id,
                'email_from': self.order_id.partner_id.email,
            })
            # duplicating a project doesn't set the SO on sub-tasks
            project.tasks.filtered(lambda task: task.parent_id != False).write({
                'sale_line_id': self.id,
                'sale_order_id': self.order_id,
            })
        else:
            project = self.env['project.project'].create(values)

        # Avoid new tasks to go to 'Undefined Stage'
        if not project.type_ids:
            project.type_ids = self.env['project.task.type'].create({'name': _('New')})

        # link project as generated by current so line
        self.write({'project_id': project.id})
        return project

    def _timesheet_create_task_prepare_values(self, project):
        self.ensure_one()
        planned_hours = self._convert_qty_company_hours(self.company_id)
        sale_line_name_parts = self.name.split('\n')
        title = sale_line_name_parts[0] or self.product_id.name
        description = '<br/>'.join(sale_line_name_parts[1:])
        return {
            'name': title if project.sale_line_id else '%s: %s' % (self.order_id.name or '', title),
            'planned_hours': planned_hours,
            'partner_id': self.order_id.partner_id.id,
            'email_from': self.order_id.partner_id.email,
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
        task_msg = _("This task has been created from: <a href=# data-oe-model=sale.order data-oe-id=%d>%s</a> (%s)") % (self.order_id.id, self.order_id.name, self.product_id.name)
        task.message_post(body=task_msg)
        return task

    def _timesheet_service_generation(self):
        """ For service lines, create the task or the project. If already exists, it simply links
            the existing one to the line.
            Note: If the SO was confirmed, cancelled, set to draft then confirmed, avoid creating a
            new project/task. This explains the searches on 'sale_line_id' on project/task. This also
            implied if so line of generated task has been modified, we may regenerate it.
        """
        so_line_task_global_project = self.filtered(lambda sol: sol.is_service and sol.product_id.service_tracking == 'task_global_project')
        so_line_new_project = self.filtered(lambda sol: sol.is_service and sol.product_id.service_tracking in ['project_only', 'task_in_project'])

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

    def _prepare_invoice_line(self, **optional_values):
        """
            If the sale order line isn't linked to a sale order which already have a default analytic account,
            this method allows to retrieve the analytic account which is linked to project or task directly linked
            to this sale order line, or the analytic account of the project which uses this sale order line, if it exists.
        """
        values = super(SaleOrderLine, self)._prepare_invoice_line(**optional_values)
        if not values.get('analytic_account_id'):
            task_analytic_account = self.task_id._get_task_analytic_account_id() if self.task_id else False
            if task_analytic_account:
                values['analytic_account_id'] = task_analytic_account.id
            elif self.project_id.analytic_account_id:
                values['analytic_account_id'] = self.project_id.analytic_account_id.id
            elif self.is_service and not self.is_expense:
                task_analytic_account_id = self.env['project.task'].read_group([
                    ('sale_line_id', '=', self.id),
                    ('analytic_account_id', '!=', False),
                ], ['analytic_account_id'], ['analytic_account_id'])
                project_analytic_account_id = self.env['project.project'].read_group([
                    ('analytic_account_id', '!=', False),
                    '|',
                        ('sale_line_id', '=', self.id),
                        '&',
                            ('tasks.sale_line_id', '=', self.id),
                            ('tasks.analytic_account_id', '=', False)
                ], ['analytic_account_id'], ['analytic_account_id'])
                analytic_account_ids = {rec['analytic_account_id'][0] for rec in (task_analytic_account_id + project_analytic_account_id)}
                if len(analytic_account_ids) == 1:
                    values['analytic_account_id'] = analytic_account_ids.pop()
        analytic_tag_ids = [Command.link(tag_id.id) for tag_id in self.task_id.analytic_tag_ids]
        if self.is_service and not self.is_expense:
            tag_ids = self.env['account.analytic.tag'].search([
                ('task_ids.sale_line_id', '=', self.id)
            ])
            analytic_tag_ids += [Command.link(tag_id.id) for tag_id in tag_ids]
        if analytic_tag_ids:
            values['analytic_tag_ids'] = values.get('analytic_tag_ids', []) + analytic_tag_ids
        return values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product
from . import project
from . import sale_order

```

## File: report\project_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ReportProjectTaskUser(models.Model):
    _inherit = "report.project.task.user"

    sale_order_id = fields.Many2one('sale.order', string='Sales Order', readonly=True)

    def _select(self):
        return super()._select() + ", t.sale_order_id"

    def _group_by(self):
        return super()._group_by() + ", t.sale_order_id"

```

## File: report\project_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_task_project_user_search_inherited" model="ir.ui.view">
            <field name="name">report.project.task.user.search.inherited</field>
            <field name="model">report.project.task.user</field>
            <field name="inherit_id" ref="project.view_task_project_user_search" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='partner_id']" position="after">
                    <field name="sale_order_id"/>
                </xpath>
             </field>
        </record>
    </data>
</odoo>

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
        <field name="domain_force">['&amp;', '&amp;', ('state', 'in', ['sale', 'done']), ('is_service', '=', True), '|', ('project_id','!=', False), ('task_id','!=', False)]</field>
        <field name="groups" eval="[(4, ref('project.group_project_manager'))]"/>
        <field name="perm_create" eval="0"/>
        <field name="perm_write" eval="0"/>
        <field name="perm_unlink" eval="0"/>
        <field name="perm_read" eval="1"/>
    </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_template_form_view_invoice_policy_inherit_sale_project" model="ir.ui.view">
        <field name="name">product.template.inherit.sale.projectform</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale.product_template_form_view_invoice_policy"/>
        <field name="arch" type="xml">
            <field name="invoice_policy" position="after">
                <field name="service_tracking" required="1" attrs="{'invisible': [('type', '!=', 'service')]}"/>
            </field>
            <field name="product_tooltip" position="after">
                <field name="project_id" context="{'default_allow_billable': True}" attrs="{'invisible':[('service_tracking','!=','task_global_project')], 'required':[('service_tracking','==','task_global_project')]}"/>
                <field name="project_template_id" context="{'active_test': False, 'default_allow_billable': True}" attrs="{'invisible':[('service_tracking','not in',['task_in_project', 'project_only'])]}"/>
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
        <field name="inherit_id" ref="project.project_sharing_project_task_view_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <field name="sale_order_id" invisible="1"/>
                <button class="d-none d-md-inline oe_stat_button"
                        type="object" name="action_view_so" icon="fa-dollar"
                        invisible="1"
                        string="Sales Order"/>
            </div>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="options">{'no_open': True, 'no_create': True, 'no_edit': True}</attribute>
                <attribute name="context">{'res_partner_search_mode': 'customer'}</attribute>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="commercial_partner_id" invisible="1" />
                <field name="sale_line_id" string="Sales Order Item" attrs="{'invisible': [('partner_id', '=', False)]}" options='{"no_open": True}' readonly="1" context="{'create': False, 'edit': False, 'delete': False}"/>
            </xpath>
        </field>
    </record>

    <record id="project_sharing_inherit_project_task_view_search" model="ir.ui.view">
        <field name="name">project.task.search.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="before">
                <field name="sale_order_id" string="Sale Order" filter_domain="['|', ('sale_order_id', 'ilike', self), ('sale_line_id', 'ilike', self)]"/>
            </xpath>
            <xpath expr="//group/filter[@name='customer']" position="after">
                <!-- TODO: Remove me in master -->
                <filter name="sale_order_id" string="Sales Order Id" invisible="1" />
                <filter name="group_by_sale_order" string="Sales Order" context="{'group_by': 'sale_order_id'}" />
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

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
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_line_id" optional="hide" readonly="1"/>
            </xpath>
        </field>
    </record>

    <record id="view_edit_project_inherit_form" model="ir.ui.view">
        <field name="name">project.project.view.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="project.edit_project"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <field name="has_any_so_to_invoice" invisible="1"/>
                <button name="action_create_invoice" string="Create Invoice" type="object" class="oe_highlight oe_read_only" groups="sales_team.group_sale_salesman_all_leads" attrs="{'invisible': [('has_any_so_to_invoice', '=', False)]}"/>
            </xpath>
            <xpath expr="//button[@name='%(project.act_project_project_2_project_task_all)d']" position="before">
                <button class="d-none d-md-inline oe_stat_button"
                        type="object" name="action_view_so" icon="fa-dollar"
                        attrs="{'invisible': [('sale_order_id', '=', False)]}"
                        string="Sales Order"
                        groups="sales_team.group_sale_salesman_all_leads">
                    <field name="sale_order_id" attrs="{'invisible': True}"/> 
                </button>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="options">{'always_reload': True}</attribute>
                <attribute name="context">{'res_partner_search_mode': 'customer'}</attribute>
            </xpath>
        </field>
    </record>
    <record id="view_sale_project_inherit_form" model="ir.ui.view">
        <field name="name">project.task.view.inherit</field>
        <field name="model">project.task</field>
        <field name="groups_id" eval="[(4, ref('base.group_user'))]"/>
        <field name="inherit_id" ref="project.view_task_form2"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_recurring_tasks']" position="before">
                <button class="oe_stat_button"
                        type="object" name="action_view_so" icon="fa-dollar"
                        attrs="{'invisible': [('sale_order_id', '=', False)]}"
                        string="Sales Order"
                        groups="sales_team.group_sale_salesman_all_leads"/>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="attributes">
                <attribute name="options">{'always_reload': True}</attribute>
                <attribute name="context">{'res_partner_search_mode': 'customer'}</attribute>
            </xpath>
            <xpath expr="//field[@name='partner_phone']" position="after">
                <field name="sale_order_id" attrs="{'invisible': True}" groups="sales_team.group_sale_salesman"/>
                <field name="sale_line_id" string="Sales Order Item" attrs="{'invisible': [('partner_id', '=', False)]}" options='{"no_open": True}' readonly="1" context="{'create': False, 'edit': False, 'delete': False}"/>
                <field name="commercial_partner_id" invisible="1" />
            </xpath>
        </field>
    </record>

    <record id="project_task_view_form_inherit_sale_line_editable" model="ir.ui.view">
        <field name="name">project.task.form.inherit.sale.line.editable.salesman</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="view_sale_project_inherit_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='sale_line_id']" position="attributes">
                <attribute name="options">{"no_create": True}</attribute>
                <attribute name="readonly">0</attribute>
            </xpath>
        </field>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

    <record id="project_task_view_search" model="ir.ui.view">
        <field name="name">project.task.search.inherit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.view_task_search_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="sale_order_id" string="Sale Order" filter_domain="['|', ('sale_order_id', 'ilike', self), ('sale_line_id', 'ilike', self)]"/>
            </xpath>
            <xpath expr="//search/group/filter[@name='customer']" position="after">
                <!-- TODO: Remove me in master -->
                <filter string="Sales Order" name="sale_order_id" invisible="1"/>
                <filter string="Sales Order" name="group_by_sale_order" context="{'group_by': 'sale_order_id'}" invisible="1"/>
                <filter string="Sales Order Item" name="sale_line_id" context="{'group_by': 'sale_line_id'}"/>
            </xpath>
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
            <xpath expr="//button[@name='preview_sale_order']" position="before">
                <button type="object" name="action_view_project_ids" class="oe_stat_button" icon="fa-puzzle-piece" attrs="{'invisible': ['|', ('state', 'in', ['draft', 'sent']), ('project_ids', '=', [])]}" groups="project.group_project_manager">
                    <field name="project_ids" invisible="1"/>
                    <field name="project_count" widget="statinfo" string="Projects"/>
                </button>
            </xpath>
            <xpath expr="//button[@name='action_view_invoice']" position="before">
                <button type="object" name="action_view_task" class="oe_stat_button" icon="fa-tasks" attrs="{'invisible': [('tasks_count', '=', 0)]}" groups="project.group_project_user">
                    <field name="tasks_count" widget="statinfo" string="Tasks"/>
                </button>
            </xpath>
            <xpath expr="//field[@name='analytic_account_id']" position="after">
                <field name="visible_project" invisible="1"/>
                <field name="project_id" options="{'no_create': True}" attrs="{'invisible': [('visible_project', '=', False)]}"/>
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
        <xpath expr="//t[@t-call='portal.portal_table']//thead/tr/th[@name='project_portal_assignees']" position="before">
            <th t-if="groupby == 'sale_order'">
                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> for sale order:</em>
                <span t-if="tasks[0].sudo().sale_order_id" class="text-truncate" t-field="tasks[0].sudo().sale_order_id"/>
                <span t-else="">None</span>
            </th>
            <th t-if="groupby == 'sale_line'">
                <em class="font-weight-normal text-muted"><span t-field="tasks[0].sudo().project_id.label_tasks"/> for sale order item:</em>
                <span t-if="tasks[0].sudo().sale_line_id" class="text-truncate" t-field="tasks[0].sudo().sale_line_id"/>
                <span t-else="">None</span>
            </th>
        </xpath>
    </template>

</odoo>

```

