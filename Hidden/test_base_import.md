# Odoo Module: test_base_import

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Test - Base Import',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 3843,
    'summary': 'Base Import Tests: Ensure Flow Robustness',
    'description': """This module contains tests related to base import.""",
    'depends': ['base_import'],
    'data': [
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: models\test_base_import.py

```python
# -*- coding: utf-8 -*-
from odoo import fields, models


def model(suffix_name):
    return 'base_import.%s' % suffix_name


class Char(models.Model):
    _name = model('char')
    _description = 'Tests: Base Import Model, Character'

    value = fields.Char()
class CharRequired(models.Model):
    _name = model('char.required')
    _description = 'Tests: Base Import Model, Character required'

    value = fields.Char(required=True)

class CharReadonly(models.Model):
    _name = model('char.readonly')
    _description = 'Tests: Base Import Model, Character readonly'

    value = fields.Char(readonly=True)

class CharNoreadonly(models.Model):
    _name = model('char.noreadonly')
    _description = 'Tests: Base Import Model, Character No readonly'

    value = fields.Char(readonly=True)

class CharStillreadonly(models.Model):
    _name = model('char.stillreadonly')
    _description = 'Tests: Base Import Model, Character still readonly'

    value = fields.Char(readonly=True)

# TODO: complex field (m2m, o2m, m2o)
class M2o(models.Model):
    _name = model('m2o')
    _description = 'Tests: Base Import Model, Many to One'

    value = fields.Many2one(model('m2o.related'))

class M2oRelated(models.Model):
    _name = model('m2o.related')
    _description = 'Tests: Base Import Model, Many to One related'

    value = fields.Integer(default=42)

class M2oRequired(models.Model):
    _name = model('m2o.required')
    _description = 'Tests: Base Import Model, Many to One required'

    value = fields.Many2one(model('m2o.required.related'), required=True)

class M2oRequiredRelated(models.Model):
    _name = model('m2o.required.related')
    _description = 'Tests: Base Import Model, Many to One required related'

    value = fields.Integer(default=42)

class O2m(models.Model):
    _name = model('o2m')
    _description = 'Tests: Base Import Model, One to Many'

    name = fields.Char()
    value = fields.One2many(model('o2m.child'), 'parent_id')

class O2mChild(models.Model):
    _name = model('o2m.child')
    _description = 'Tests: Base Import Model, One to Many child'

    parent_id = fields.Many2one(model('o2m'))
    value = fields.Integer()

class PreviewModel(models.Model):
    _name = model('preview')
    _description = 'Tests: Base Import Model Preview'

    name = fields.Char('Name')
    somevalue = fields.Integer(string='Some Value', required=True)
    othervalue = fields.Integer(string='Other Variable')

class FloatModel(models.Model):
    _name = model('float')
    _description = 'Tests: Base Import Model Float'

    value = fields.Float()
    value2 = fields.Monetary()
    currency_id = fields.Many2one('res.currency')

class ComplexModel(models.Model):
    _name = model('complex')
    _description = 'Tests: Base Import Model Complex'

    f = fields.Float()
    m = fields.Monetary()
    c = fields.Char()
    currency_id = fields.Many2one('res.currency')
    d = fields.Date()
    dt = fields.Datetime()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import test_base_import

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
access_test_base_import_char,base.import.tests.models.char,model_base_import_char,base.group_user,1,1,1,1
access_test_base_import_char_required,base.import.tests.models.char.required,model_base_import_char_required,base.group_user,1,1,1,1
access_test_base_import_char_readonly,base.import.tests.models.char.readonly,model_base_import_char_readonly,base.group_user,1,1,1,1
access_test_base_import_char_noreadonly,base.import.tests.models.char.noreadonly,model_base_import_char_noreadonly,base.group_user,1,1,1,1
access_test_base_import_char_stillreadonly,base.import.tests.models.char.stillreadonly,model_base_import_char_stillreadonly,base.group_user,1,1,1,1
access_test_base_import_m2o,base.import.tests.models.m2o,model_base_import_m2o,base.group_user,1,1,1,1
access_test_base_import_m2o_related,base.import.tests.models.m2o.related,model_base_import_m2o_related,base.group_user,1,1,1,1
access_test_base_import_m2o_required,base.import.tests.models.m2o.required,model_base_import_m2o_required,base.group_user,1,1,1,1
access_test_base_import_m2o_required_related,base.import.tests.models.m2o.required.related,model_base_import_m2o_required_related,base.group_user,1,1,1,1
access_test_base_import_o2m,base.import.tests.models.o2m,model_base_import_o2m,base.group_user,1,1,1,1
access_test_base_import_o2m_child,base.import.tests.models.o2m.child,model_base_import_o2m_child,base.group_user,1,1,1,1
access_test_base_import_float,base.import.tests.models.float,model_base_import_float,base.group_user,1,1,1,1
access_test_base_import_preview,base.import.tests.models.preview,model_base_import_preview,base.group_user,1,1,1,1
access_test_base_import_complex,access_test_base_import_complex,model_base_import_complex,base.group_user,1,0,0,0

```

