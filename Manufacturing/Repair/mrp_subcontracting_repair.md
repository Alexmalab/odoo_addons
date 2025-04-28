# Odoo Module: mrp_subcontracting_repair

Category: Manufacturing/Repair

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'MRP Subcontracting Repair',
    'version': '1.0',
    'category': 'Manufacturing/Repair',
    'description': """
        Bridge module between MRP subcontracting and Repair
    """,
    'depends': [
        'mrp_subcontracting', 'repair'
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/mrp_subcontracting_repair_security.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_subcontracting_portal_repair_line,subcontracting.portal.repair_line,repair.model_repair_line,base.group_portal,1,0,0,0

```

## File: security\mrp_subcontracting_repair_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="repair_line_subcontracting_rule" model="ir.rule">
        <field name="name">Repair Line Subcontractor</field>
        <field name="model_id" ref="repair.model_repair_line"/>
        <field name="domain_force">[
        '|',
            '|',
                ('product_id', 'in', user.partner_id.bom_ids.product_id.ids),
                ('product_id', 'in', user.partner_id.bom_ids.product_tmpl_id.product_variant_ids.ids),
            ('product_id', 'in', user.partner_id.bom_ids.bom_line_ids.product_id.ids),
            ]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

</odoo>

```

