# Odoo Module: base_sparse_field

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Sparse Fields",
    'summary': """Implementation of sparse fields.""",
    'description': """
The purpose of this module is to implement "sparse" fields, i.e., fields
that are mostly null. This implementation circumvents the PostgreSQL
limitation on the number of columns in a table. The values of all sparse
fields are stored in a "serialized" field in the form of a JSON mapping.
    """,
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\fields.py

```python
# -*- coding: utf-8 -*-

import json

from odoo import fields


def monkey_patch(cls):
    """ Return a method decorator to monkey-patch the given class. """
    def decorate(func):
        name = func.__name__
        func.super = getattr(cls, name, None)
        setattr(cls, name, func)
        return func
    return decorate


#
# Implement sparse fields by monkey-patching fields.Field
#

fields.Field.__doc__ += """

        .. _field-sparse:

        .. rubric:: Sparse fields

        Sparse fields have a very small probability of being not null. Therefore
        many such fields can be serialized compactly into a common location, the
        latter being a so-called "serialized" field.

        :param sparse: the name of the field where the value of this field must
            be stored.
"""
fields.Field.sparse = None

@monkey_patch(fields.Field)
def _get_attrs(self, model_class, name):
    attrs = _get_attrs.super(self, model_class, name)
    if attrs.get('sparse'):
        # by default, sparse fields are not stored and not copied
        attrs['store'] = False
        attrs['copy'] = attrs.get('copy', False)
        attrs['compute'] = self._compute_sparse
        if not attrs.get('readonly'):
            attrs['inverse'] = self._inverse_sparse
    return attrs

@monkey_patch(fields.Field)
def _compute_sparse(self, records):
    for record in records:
        values = record[self.sparse]
        record[self.name] = values.get(self.name)
    if self.relational:
        for record in records:
            record[self.name] = record[self.name].exists()

@monkey_patch(fields.Field)
def _inverse_sparse(self, records):
    for record in records:
        values = record[self.sparse]
        value = self.convert_to_read(record[self.name], record, use_display_name=False)
        if value:
            if values.get(self.name) != value:
                values[self.name] = value
                record[self.sparse] = values
        else:
            if self.name in values:
                values.pop(self.name)
                record[self.sparse] = values


#
# Definition and implementation of serialized fields
#

class Serialized(fields.Field):
    """ Serialized fields provide the storage for sparse fields. """
    type = 'serialized'
    column_type = ('text', 'text')

    prefetch = False                    # not prefetched by default

    def convert_to_column_insert(self, value, record, values=None, validate=True):
        return self.convert_to_cache(value, record, validate=validate)

    def convert_to_cache(self, value, record, validate=True):
        # cache format: json.dumps(value) or None
        return json.dumps(value) if isinstance(value, dict) else (value or None)

    def convert_to_record(self, value, record):
        return json.loads(value or "{}")


fields.Serialized = Serialized

```

## File: models\models.py

```python
# -*- coding: utf-8 -*-

from collections import defaultdict

from odoo import models, fields, api, _
from odoo.exceptions import UserError


class Base(models.AbstractModel):
    _inherit = 'base'

    def _valid_field_parameter(self, field, name):
        return name == 'sparse' or super()._valid_field_parameter(field, name)


class IrModelFields(models.Model):
    _inherit = 'ir.model.fields'

    ttype = fields.Selection(selection_add=[
        ('serialized', 'serialized'),
    ], ondelete={'serialized': 'cascade'})
    serialization_field_id = fields.Many2one('ir.model.fields', string='Serialization Field',
        ondelete='cascade', domain="[('ttype','=','serialized'), ('model_id', '=', model_id)]",
        help="If set, this field will be stored in the sparse structure of the "
             "serialization field, instead of having its own database column. "
             "This cannot be changed after creation.",
    )

    def write(self, vals):
        # Limitation: renaming a sparse field or changing the storing system is
        # currently not allowed
        if 'serialization_field_id' in vals or 'name' in vals:
            for field in self:
                if 'serialization_field_id' in vals and field.serialization_field_id.id != vals['serialization_field_id']:
                    raise UserError(_('Changing the storing system for field "%s" is not allowed.', field.name))
                if field.serialization_field_id and (field.name != vals['name']):
                    raise UserError(_('Renaming sparse field "%s" is not allowed', field.name))

        return super(IrModelFields, self).write(vals)

    def _reflect_fields(self, model_names):
        super()._reflect_fields(model_names)

        # set 'serialization_field_id' on sparse fields; it is done here to
        # ensure that the serialized field is reflected already
        cr = self._cr

        # retrieve existing values
        query = """
            SELECT model, name, id, serialization_field_id
            FROM ir_model_fields
            WHERE model IN %s
        """
        cr.execute(query, [tuple(model_names)])
        existing = {row[:2]: row[2:] for row in cr.fetchall()}

        # determine updates, grouped by value
        updates = defaultdict(list)
        for model_name in model_names:
            for field_name, field in self.env[model_name]._fields.items():
                field_id, current_value = existing[(model_name, field_name)]
                try:
                    value = existing[(model_name, field.sparse)][0] if field.sparse else None
                except KeyError:
                    raise UserError(_(
                        'Serialization field "%(serialization_field)s" not found for sparse field %(sparse_field)s!',
                        serialization_field=field.sparse,
                        sparse_field=field,
                    ))
                if current_value != value:
                    updates[value].append(field_id)

        if not updates:
            return

        # update fields
        query = "UPDATE ir_model_fields SET serialization_field_id=%s WHERE id IN %s"
        for value, ids in updates.items():
            cr.execute(query, [value, tuple(ids)])

        records = self.browse(id_ for ids in updates.values() for id_ in ids)
        self.pool.post_init(records.modified, ['serialization_field_id'])

    def _instanciate_attrs(self, field_data):
        attrs = super(IrModelFields, self)._instanciate_attrs(field_data)
        if attrs and field_data.get('serialization_field_id'):
            serialization_record = self.browse(field_data['serialization_field_id'])
            attrs['sparse'] = serialization_record.name
        return attrs


class TestSparse(models.TransientModel):
    _name = 'sparse_fields.test'
    _description = 'Sparse fields Test'

    data = fields.Serialized()
    boolean = fields.Boolean(sparse='data')
    integer = fields.Integer(sparse='data')
    float = fields.Float(sparse='data')
    char = fields.Char(sparse='data')
    selection = fields.Selection([('one', 'One'), ('two', 'Two')], sparse='data')
    partner = fields.Many2one('res.partner', sparse='data')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import fields
from . import models

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_sparse_fields_test","access.sparse_fields.test","model_sparse_fields_test","base.group_system",1,1,1,0

```

## File: views\views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Add 'serialization_field_id' in ir.model form view -->
        <record model="ir.ui.view" id="model_form_view">
            <field name="model">ir.model</field>
            <field name="inherit_id" ref="base.view_model_form"/>
            <field name="arch" type="xml">
                <field name="related" position="before">
                    <field name="serialization_field_id"
                        domain="[('ttype','=','serialized'), ('model_id','=',parent.model)]"
                        readonly="state == 'base'"/>
                </field>
            </field>
        </record>

        <!-- Add 'serialization_field_id' in ir.model.fields form view -->
        <record model="ir.ui.view" id="field_form_view">
            <field name="model">ir.model.fields</field>
            <field name="inherit_id" ref="base.view_model_fields_form"/>
            <field name="arch" type="xml">
                <field name="related" position="before">
                    <field name="serialization_field_id"
                        context="{'default_model_id': model_id, 'default_ttype': 'serialized'}"
                        readonly="state == 'base'"/>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

