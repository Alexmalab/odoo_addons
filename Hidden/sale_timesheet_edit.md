# Odoo Module: sale_timesheet_edit

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

# TODO: [XBO] merge with sale_timesheet module in master
{
    'name': 'Sales Timesheet Edit',
    'category': 'Hidden',
    'summary': 'Edit the sale order line linked in the timesheets',
    'description': """
Allow to edit sale order line in the timesheets
===============================================

This module adds the edition of the sale order line
set in the timesheets. This allows adds more flexibility
to the user to easily change the sale order line on a
timesheet in task form view when it is needed.
""",
    'depends': ['sale_timesheet'],
    'data': [
        'views/assets.xml',
        'views/project_task.xml',
    ],
    'demo': [],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_analytic_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


# TODO: [XBO] merge with account.analytic.line in the sale_timesheet module in master.
class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    is_so_line_edited = fields.Boolean()

    @api.depends('task_id.sale_line_id', 'project_id.sale_line_id', 'project_id.allow_billable', 'employee_id')
    def _compute_so_line(self):
        super(AccountAnalyticLine, self.filtered(lambda t: not t.is_so_line_edited))._compute_so_line()

    def _check_sale_line_in_project_map(self):
        # TODO: [XBO] remove me in master, now we authorize to manually edit the so_line, then this so_line can be different of the one in task/project/map_entry
        # !!! Override of the method in sale_timesheet !!!
        return

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Project(models.Model):
    _inherit = 'project.project'

    def _get_not_billed_timesheets(self):
        """ Get the timesheets not invoiced and the SOL has not manually been edited
            FIXME: [XBO] this change must be done in the _update_timesheets_sale_line_id
                rather than this method in master to keep the initial behaviour of this method.
        """
        return super(Project, self)._get_not_billed_timesheets() - self.mapped('timesheet_ids').filtered('is_so_line_edited')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_analytic_line
from . import project

```

## File: static\src\js\so_line_one2many.js

```javascript
odoo.define('sale_timesheet_edit.so_line_many2one', function (require) {
"use strict";

const fieldRegistry = require('web.field_registry');
const FieldOne2Many = require('web.relational_fields').FieldOne2Many;

const SoLineOne2Many = FieldOne2Many.extend({
    _onFieldChanged: function (ev) {
        if (
            ev.data.changes &&
            ev.data.changes.hasOwnProperty('timesheet_ids') &&
            ev.data.changes.timesheet_ids.operation === 'UPDATE' &&
            ev.data.changes.timesheet_ids.data &&
            ev.data.changes.timesheet_ids.data.hasOwnProperty('so_line')) {
            const line = this.value.data.find(line => {
                return line.id === ev.data.changes.timesheet_ids.id;
            });
            if (!line.is_so_line_edited) {
                ev.data.changes.timesheet_ids.data.is_so_line_edited = true;
            }
        }
        this._super.apply(this, arguments);
    }
});


fieldRegistry.add('so_line_one2many', SoLineOne2Many);

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_backend" inherit_id="web.assets_backend">
        <xpath expr="script[last()]" position="after">
            <script type="text/javascript" src="/sale_timesheet_edit/static/src/js/so_line_one2many.js"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\project_task.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- This view can be removed to change only if it is needed in the inherit view in sale_timesheet -->
    <record id="project_task_view_form_inherit_sale_timesheet_edit" model="ir.ui.view">
        <field name="name">project.task.form.view.form.inherit.sale.timesheet.edit</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="sale_timesheet.project_task_view_form_inherit_sale_timesheet"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='timesheet_ids']" position="attributes">
                <attribute name="widget">so_line_one2many</attribute>
            </xpath>
            <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='so_line']" position="attributes">
                <attribute name="readonly">0</attribute>
                <attribute name="domain">[('is_service', '=', True), ('order_partner_id', 'child_of', parent.commercial_partner_id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done']), ('order_id', '=?', parent.project_sale_order_id)]</attribute>
                <attribute name="options">{'no_create': True, 'no_open': True}</attribute>
            </xpath>
        </field>
    </record>

    <!--
        TODO: [XBO] In master, add this view in the sale_timesheet when we will merge of the both modules
        Don't forget to change the inherit_id to have the correct view in sale_timesheet,
        since the view above can be merged with project_task_view_form_inherit_sale_timesheet view.
    -->
    <record id="project_task_view_form_inherit_sale_timesheet_editable" model="ir.ui.view">
        <field name="name">project.task.form.view.form.inherit.sale.timesheet.editable</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project_task_view_form_inherit_sale_timesheet_edit"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='so_line']" position="attributes">
                <attribute name="options">{'no_create': True}</attribute>
            </xpath>
            <xpath expr="//field[@name='timesheet_ids']/tree" position="inside">
                <field name="is_so_line_edited" invisible="1" />
            </xpath>
        </field>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
    </record>

</odoo>

```

