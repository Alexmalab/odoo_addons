# Odoo Module: sale_sms

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Sale - SMS",
    'summary': "Ease SMS integration with sales capabilities",
    'description': "Ease SMS integration with sales capabilities",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['sale', 'sms'],
    'data': [
        'security/ir.model.access.csv',
        'security/security.xml',
    ],
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_sale_manager,access.sms.template.sale.manager,sms.model_sms_template,sales_team.group_sale_manager,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.rule" id="ir_rule_sms_template_so_sale_manager">
        <field name="name">SMS Template: sale manager CRUD on sale orders</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('sales_team.group_sale_manager'))]"/>
        <field name="domain_force">[('model_id.model', 'in', ('sale.order', 'res.partner'))]</field>
    </record>
</odoo>

```

