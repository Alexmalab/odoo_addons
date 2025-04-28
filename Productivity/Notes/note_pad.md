# Odoo Module: note_pad

Category: Productivity/Notes

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

{
    'name': 'Memos pad',
    'version': '0.1',
    'category': 'Productivity/Notes',
    'description': """
This module update memos inside Odoo for using an external pad
=================================================================

Use for update your text memo in real time with the following user that you invite.

""",
    'summary': 'Sticky memos, Collaborative',
    'depends': [
        'mail',
        'pad',
        'note',
    ],
    'data': [
        'views/note_views.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\note.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class NotePad(models.Model):

    _name = 'note.note'
    _inherit = ['pad.common', 'note.note']

    _pad_fields = ['note_pad']

    note_pad_url = fields.Char('Pad Url', pad_content_field='memo', copy=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import note

```

## File: views\note_views.xml

```xml
<?xml version="1.0" ?>
<odoo>

    <record id="view_note_note_pad_form" model="ir.ui.view">
      <field name="name">note_pad.view.form</field>
      <field name="model">note.note</field>
      <field name="inherit_id" ref="note.view_note_note_form"/>
      <field name="arch" type="xml">
        <field name="memo" position="replace">
           <field name="note_pad_url" widget="pad" class="oe_memo"/>
        </field>
      </field>
    </record>

</odoo>

```

