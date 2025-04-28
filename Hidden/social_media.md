# Odoo Module: social_media

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
    'name': "Social Media",

    'summary': "Social media connectors for company settings.",

    'description': """
The purpose of this technical module is to provide a front for
social media configuration for any other module that might need it.
    """,
    'category': 'Hidden',
    'version': '0.1',
    'depends': ['base'],

    'data': [
        'views/res_company_views.xml',
    ],
    'demo': [
        'demo/res_company_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Company(models.Model):
    _inherit = "res.company"

    social_twitter = fields.Char('Twitter Account')
    social_facebook = fields.Char('Facebook Account')
    social_github = fields.Char('GitHub Account')
    social_linkedin = fields.Char('LinkedIn Account')
    social_youtube = fields.Char('Youtube Account')
    social_instagram = fields.Char('Instagram Account')
    social_tiktok = fields.Char('TikTok Account')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_company

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_company_form_inherit_social_media" model="ir.ui.view">
        <field name="name">res.company.form.inherit.social.media</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='social_media']" position="replace">
                <group string="Social Media" name="social_media" groups="base.group_no_one">
                    <field name="social_twitter"/>
                    <field name="social_facebook"/>
                    <field name="social_github"/>
                    <field name="social_linkedin"/>
                    <field name="social_youtube"/>
                    <field name="social_instagram"/>
                    <field name="social_tiktok"/>
                </group>
            </xpath>
        </field>
    </record>

</odoo>

```

