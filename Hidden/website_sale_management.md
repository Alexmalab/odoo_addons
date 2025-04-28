# Odoo Module: website_sale_management

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
{
    'name': 'Website - Sales Management',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
Display orders to invoice in website dashboard.
""",
    'depends': [
        'sale_management',
        'website_sale',
    ],
    'installable': True,
    'auto_install': True,
    'data': [
    ],
    'demo': [
    ],
    'qweb': ['static/src/xml/*.xml'],
    'license': 'LGPL-3',
}

```

## File: static\src\xml\website_sale_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="website.dashboard_header">
        <t t-jquery=".o_dashboard_common a.o_dashboard_action .o_primary" t-operation="after">
            <div t-if="widget.dashboards_data.sales.summary.order_to_invoice_count" class="o_inner_box o_dashboard_action" title="Generate an invoice from orders ready for invoicing." name="website_sale.sale_order_action_to_invoice">
                <div class="o_highlight"><t t-esc="widget.dashboards_data.sales.summary.order_to_invoice_count"/></div>
                Orders to Invoice
            </div>
        </t>
   </t>
</templates>

```

