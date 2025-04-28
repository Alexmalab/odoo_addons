# Odoo Module: hr_livechat

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
{
    'name': 'HR - Livechat',
    'version': '1.0',
    'category': 'Human Resources',
    'description': """
Bridge between HR and Livechat.""",
    'depends': ['hr', 'im_livechat'],
    'data': [
        'views/discuss_channel_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: views\discuss_channel_views.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <record id="discuss_channel_view_search" model="ir.ui.view">
            <field name="name">discuss.channel.search.inherit.hr.livechat</field>
            <field name="model">discuss.channel</field>
            <field name="inherit_id" ref="im_livechat.discuss_channel_view_search"/>
            <field name="arch" type="xml">
                <xpath expr="//*[@name='filter_my_sessions']" position="after">
                    <filter name="filter_my_team" domain="[('channel_member_ids.partner_id.user_ids.employee_id.member_of_department', '=', True)]" string="My Team"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

