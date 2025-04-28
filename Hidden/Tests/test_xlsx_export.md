# Odoo Module: test_xlsx_export

Category: Hidden/Tests

This file contains the source code of the Odoo module.

## File: ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
access_export_group_operator,access_export_group_operator,model_export_group_operator,,1,1,1,1
access_export_group_operator_one2many,access_export_group_operator_one2many,model_export_group_operator_one2many,,1,1,1,1
access_export_integer,access_export_integer,model_export_integer,,1,1,1,1
access_export_computed_binary,access_export_computed_binary,model_export_computed_binary,,1,1,1,1

```

## File: models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

class NewModel(models.Model):
    _name = 'export.integer'
    _description = 'Export: Integer'

    value = fields.Integer(default=4)

    def name_get(self):
        return [(record.id, "%s:%s" % (self._name, record.value)) for record in self]

class GroupOperator(models.Model):
    _name = 'export.group_operator'
    _description = 'Export Group Operator'

    int_sum = fields.Integer(group_operator='sum')
    int_max = fields.Integer(group_operator='max')
    float_min = fields.Float(group_operator='min')
    float_avg = fields.Float(group_operator='avg')
    float_monetary = fields.Monetary(currency_field='currency_id',group_operator='sum')
    currency_id = fields.Many2one('res.currency')
    date_max = fields.Date(group_operator='max')
    bool_and = fields.Boolean(group_operator='bool_and')
    bool_or = fields.Boolean(group_operator='bool_or')
    many2one = fields.Many2one('export.integer')
    one2many = fields.One2many('export.group_operator.one2many', 'parent_id')

class GroupOperatorO2M(models.Model):
    _name = 'export.group_operator.one2many'
    _description = 'Export Group Operator One2Many'

    parent_id = fields.Many2one('export.group_operator')
    value = fields.Integer()

class ComputedBinary(models.Model):
    _name = 'export.computed.binary'
    _description = 'Export computed binary'

    binary_field = fields.Binary(compute='_compute_binary_field')

    def _compute_binary_field(self):
        # This kind of computed binary field is obviously a bad idea,
        # but since the ORM supports it, the export also needs to handle it.
        self.binary_field = ["computed value"]

```

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'test xlsx export',
    'version': '0.1',
    'category': 'Hidden/Tests',
    'description': """A module to test xlsx export.""",
    'depends': ['web', 'test_mail'],
    'data': ['ir.model.access.csv'],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

