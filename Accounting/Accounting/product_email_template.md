# Odoo Module: product_email_template

Category: Accounting/Accounting

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
    'name': 'Product Email Template',
    'depends': ['account'],
    'category': 'Accounting/Accounting',
    'description': """
Add email templates to products to be sent on invoice confirmation
==================================================================

With this module, link your products to a template to send complete information and tools to your customer.
For instance when invoicing a training, the training agenda and materials will automatically be sent to your customers.'
    """,
    'data': [
        'views/product_views.xml',
        'views/mail_template_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, SUPERUSER_ID


class AccountMove(models.Model):
    _inherit = 'account.move'

    def invoice_validate_send_email(self):
        if self.env.su:
            # sending mail in sudo was meant for it being sent from superuser
            self = self.with_user(SUPERUSER_ID)
        for invoice in self.filtered(lambda x: x.move_type == 'out_invoice'):
            # send template only on customer invoice
            # subscribe the partner to the invoice
            if invoice.partner_id not in invoice.message_partner_ids:
                invoice.message_subscribe([invoice.partner_id.id])
            for line in invoice.invoice_line_ids:
                if line.product_id.email_template_id:
                    invoice.message_post_with_template(
                        line.product_id.email_template_id.id,
                        composition_mode="comment",
                        email_layout_xmlid="mail.mail_notification_light"
                    )
        return True

    def _post(self, soft=True):
        # OVERRIDE
        posted = super()._post(soft)
        posted.invoice_validate_send_email()
        return posted

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ProductTemplate(models.Model):
    """ Product Template inheritance to add an optional email.template to a
    product.template. When validating an invoice, an email will be send to the
    customer based on this template. The customer will receive an email for each
    product linked to an email template. """
    _inherit = "product.template"

    email_template_id = fields.Many2one('mail.template', string='Product Email Template',
        help='When validating an invoice, an email will be sent to the customer '
        'based on this template. The customer will receive an email for each '
        'product linked to an email template.')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product
from . import account_move
```

## File: views\mail_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="email_template_form_simplified" model="ir.ui.view">
            <field name="name">mail.template.form.simplified</field>
            <field name="model">mail.template</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <form string="Email Template">
                    <group>
                        <field name="subject" invisible="1"/>
                        <field name="name" invisible="1"/>
                        <field name="model" invisible="1"/>
                        <h3 colspan="2">Body</h3>
                        <field name="body_html" nolabel="1" colspan="2" widget="html"
                            options="{'style-inline': true}" />
                        <field name="attachment_ids" nolabel="1" colspan="2"
                            widget="many2many_binary"/>
                    </group>
                </form>
            </field>
        </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="product_template_form_view" model="ir.ui.view">
            <field name="name">product.template.form.inherit.email.template</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='invoicing']//group[@name='accounting']" position="inside">
                    <group name="email_template" string="Automatic Email at Invoice">
                        <field name="email_template_id" string="Email Template" help="Send a product-specific email once the invoice is validated"
                            domain="[('model','=','account.move')]"
                            context="{
                                'form_view_ref':'product_email_template.email_template_form_simplified',
                                'default_model': 'account.move',
                                'default_subject': name,
                                'default_name': name,
                            }"/>
                    </group>
                </xpath>
            </field>
        </record>
</odoo>

```

