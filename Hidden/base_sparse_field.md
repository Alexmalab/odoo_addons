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

@monkey_patch(fields.Field)
def _get_attrs(self, model, name):
    attrs = _get_attrs.super(self, model, name)
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
        value = self.convert_to_read(record[self.name], record, use_name_get=False)
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
    _slots = {
        'prefetch': False,              # not prefetched by default
    }
    column_type = ('text', 'text')

    def convert_to_column(self, value, record, values=None, validate=True):
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

from odoo import models, fields, api, _
from odoo.exceptions import UserError


class IrModelFields(models.Model):
    _inherit = 'ir.model.fields'

    ttype = fields.Selection(selection_add=[('serialized', 'serialized')])
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
                    raise UserError(_('Changing the storing system for field "%s" is not allowed.') % field.name)
                if field.serialization_field_id and (field.name != vals['name']):
                    raise UserError(_('Renaming sparse field "%s" is not allowed') % field.name)

        return super(IrModelFields, self).write(vals)

    def _reflect_model(self, model):
        super(IrModelFields, self)._reflect_model(model)

        # set 'serialization_field_id' on sparse fields; it is done here to
        # ensure that the serialized field is reflected already
        cr = self._cr
        query = """ UPDATE ir_model_fields
                    SET serialization_field_id=%s
                    WHERE model=%s AND name=%s
                    RETURNING id
                """
        fields_data = self._existing_field_data(model._name)

        for field in model._fields.values():
            ser_field_id = None
            ser_field_name = getattr(field, 'sparse', None)
            if ser_field_name:
                if ser_field_name not in fields_data:
                    msg = _("Serialization field `%s` not found for sparse field `%s`!")
                    raise UserError(msg % (ser_field_name, field.name))
                ser_field_id = fields_data[ser_field_name]['id']

            if fields_data[field.name]['serialization_field_id'] != ser_field_id:
                cr.execute(query, (ser_field_id, model._name, field.name))
                record = self.browse(cr.fetchone())
                self.pool.post_init(record.modified, ['serialization_field_id'])
                self.clear_caches()

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
                        attrs="{'readonly': [('state','=','base')]}"/>
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
                        attrs="{'readonly': [('state','=','base')]}"/>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

