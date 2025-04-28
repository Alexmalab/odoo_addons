# Odoo Module: test_mail_full

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Mail Tests (Full)',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 9876,
    'summary': 'Mail Tests: performances and tests specific to mail with all sub-modules',
    'description': """This module contains tests related to various mail features
and mail-related sub modules. Those tests are present in a separate module as it
contains models used only to perform tests independently to functional aspects of
real applications. """,
    'depends': [
        'mail',
        'mail_bot',
        'portal',
        'rating',
        # 'snailmail',
        'mass_mailing',
        'mass_mailing_sms',  # adds portal
        'phone_validation',
        'sms',
        'test_mail',
        'test_mail_sms',
        'test_mass_mailing',
    ],
    'data': [
        'data/mail_message_subtype_data.xml',
        'security/ir.model.access.csv',
        'security/ir_rule_data.xml',
        'views/test_portal_template.xml',
    ],
    'assets': {
        'web.assets_unit_tests': [
            'test_mail_full/static/tests/**/*',
            ('remove', 'test_mail_full/static/tests/tours/**/*'),
        ],
        'web.assets_tests': [
            'test_mail_full/static/tests/tours/**/*',
        ],
    },
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class PortalTest(http.Controller):
    """Implements some test portal routes (ex.: for viewing a record)."""

    @http.route('/my/test_portal/<int:res_id>', type='http', auth='public', methods=['GET'])
    def test_portal_record_view(self, res_id, access_token=None, **kwargs):
        return request.make_response(f'Record view of test_portal {res_id} ({access_token}, {kwargs})')

    @http.route("/my/test_portal_records/<int:res_id>", type="http", auth="public", website=True)
    def test_portal_record_page(self, res_id, **kwargs):
        record = request.env["mail.test.portal"]._get_thread_with_access(res_id, **kwargs)
        values = {
            "object": record,
            "token": kwargs.get("access_token", None),
            "hash": kwargs.get("hash", None),
            "pid": kwargs.get("pid", None),
        }
        return request.render("test_mail_full.test_portal_template", values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mt_mail_test_rating_rating_done" model="mail.message.subtype">
        <field name="name">Rating Done</field>
        <field name="description">Rating Done</field>
        <field name="res_model">mail.test.rating</field>
        <field name="default" eval="True"/>
        <field name="internal" eval="False"/>
    </record>
</odoo>

```

## File: models\test_mail_models_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailTestPortal(models.Model):
    """ A model inheriting from mail.thread and portal.mixin with some fields
    used for portal sharing, like a partner, ..."""
    _description = 'Chatter Model for Portal'
    _name = 'mail.test.portal'
    _inherit = [
        'portal.mixin',
        'mail.thread',
    ]

    name = fields.Char('Name')
    partner_id = fields.Many2one('res.partner', 'Customer')
    user_id = fields.Many2one(comodel_name='res.users', string="Salesperson")

    def _compute_access_url(self):
        super()._compute_access_url()
        for record in self.filtered('id'):
            record.access_url = '/my/test_portal/%s' % self.id


class MailTestPortalNoPartner(models.Model):
    """ A model inheriting from portal, but without any partner field """
    _description = 'Chatter Model for Portal (no partner field)'
    _name = 'mail.test.portal.no.partner'
    _inherit = [
        'mail.thread',
        'portal.mixin',
    ]

    name = fields.Char()

    def _compute_access_url(self):
        self.access_url = False
        for record in self.filtered('id'):
            record.access_url = '/my/test_portal_no_partner/%s' % self.id


class MailTestRating(models.Model):
    """ A model inheriting from rating.mixin (which inherits from mail.thread) with some fields used for SMS
    gateway, like a partner, a specific mobile phone, ... """
    _description = 'Rating Model (ticket-like)'
    _name = 'mail.test.rating'
    _inherit = [
        'rating.mixin',
        'mail.activity.mixin',
        'portal.mixin',
    ]
    _mailing_enabled = True
    _order = 'name asc, id asc'

    name = fields.Char('Name')
    subject = fields.Char('Subject')
    company_id = fields.Many2one('res.company', 'Company')
    customer_id = fields.Many2one('res.partner', 'Customer')
    email_from = fields.Char('From', compute='_compute_email_from', precompute=True, readonly=False, store=True)
    mobile_nbr = fields.Char('Mobile', compute='_compute_mobile_nbr', precompute=True, readonly=False, store=True)
    phone_nbr = fields.Char('Phone Number', compute='_compute_phone_nbr', precompute=True, readonly=False, store=True)
    user_id = fields.Many2one('res.users', 'Responsible', tracking=1)

    @api.depends('customer_id')
    def _compute_email_from(self):
        for rating in self:
            if rating.customer_id.email_normalized:
                rating.email_from = rating.customer_id.email_normalized
            elif not rating.email_from:
                rating.email_from = False

    @api.depends('customer_id')
    def _compute_mobile_nbr(self):
        for rating in self:
            if rating.customer_id.mobile:
                rating.mobile_nbr = rating.customer_id.mobile
            elif not rating.mobile_nbr:
                rating.mobile_nbr = False

    @api.depends('customer_id')
    def _compute_phone_nbr(self):
        for rating in self:
            if rating.customer_id.phone:
                rating.phone_nbr = rating.customer_id.phone
            elif not rating.phone_nbr:
                rating.phone_nbr = False

    def _mail_get_partner_fields(self, introspect_fields=False):
        return ['customer_id']

    def _phone_get_number_fields(self):
        return ['phone_nbr', 'mobile_nbr']

    def _rating_apply_get_default_subtype_id(self):
        return self.env['ir.model.data']._xmlid_to_res_id("test_mail_full.mt_mail_test_rating_rating_done")

    def _rating_get_partner(self):
        return self.customer_id


class MailTestRatingThread(models.Model):
    """A model inheriting from mail.thread with minimal fields for testing
     rating submission without the rating mixin but with the same test code:

     - partner_id: value returned by the base _rating_get_partner method
     - user_id: value returned by the base _rating_get_operator method
     """
    _description = 'Model for testing rating without the rating mixin'
    _name = 'mail.test.rating.thread'
    _inherit = 'mail.thread'
    _order = 'name asc, id asc'

    name = fields.Char('Name')
    customer_id = fields.Many2one('res.partner', 'Customer')
    user_id = fields.Many2one('res.users', 'Responsible', tracking=1)

    def _mail_get_partner_fields(self, introspect_fields=False):
        return ['customer_id']

    def _rating_get_partner(self):
        return self.customer_id or super()._rating_get_partner()


class MailTestRatingThreadRead(models.Model):
    """Same as MailTestRatingThread but post accessible on read by portal users."""

    _description = "Read-post rating model"
    _name = "mail.test.rating.thread.read"
    _inherit = "mail.test.rating.thread"
    _order = "name asc, id asc"
    _mail_post_access = "read"

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import test_mail_models_mail

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mail_test_portal_user,mail.test.portal.user,model_mail_test_portal,base.group_user,1,1,1,1
access_mail_test_portal_no_partner_portal,mail.test.portal.no.partner.all,model_mail_test_portal_no_partner,base.group_portal,1,0,0,0
access_mail_test_portal_no_partner_user,mail.test.portal.no.partner.user,model_mail_test_portal_no_partner,base.group_user,1,1,1,1
access_mail_test_rating_all,mail.test.rating.all,model_mail_test_rating,base.group_user,0,0,0,0
access_mail_test_rating_portal,mail.test.rating.portal,model_mail_test_rating,base.group_portal,1,0,0,0
access_mail_test_rating_user,mail.test.rating.user,model_mail_test_rating,base.group_user,1,1,1,1
access_mail_test_rating_thread_all,mail.test.rating.thread.all,model_mail_test_rating_thread,,0,0,0,0
access_mail_test_rating_thread_portal,mail.test.rating.thread.portal,model_mail_test_rating_thread,base.group_portal,1,0,0,0
access_mail_test_rating_thread_user,mail.test.rating.thread.user,model_mail_test_rating_thread,base.group_user,1,1,1,1
access_mail_test_rating_thread_read_portal,mail.test.rating.thread.read.portal,model_mail_test_rating_thread_read,base.group_portal,1,0,0,0
access_mail_test_rating_thread_read_user,mail.test.rating.thread.read.user,model_mail_test_rating_thread_read,base.group_user,1,1,1,1

```

## File: security\ir_rule_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="mail_test_rating_rule_mc" model="ir.rule">
        <field name="name">TestRating: Multi Company</field>
        <field name="model_id" ref="test_mail_full.model_mail_test_rating"/>
        <field eval="True" name="global"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>
    <record id="mail_test_rating_rule_portal" model="ir.rule">
        <field name="name">TestRating: Portal should follow</field>
        <field name="model_id" ref="test_mail_full.model_mail_test_rating"/>
        <field name="domain_force">[('message_partner_ids', 'in', [user.partner_id.id])]</field>
        <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
    </record>

</odoo>

```

## File: views\test_portal_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="test_portal_template" name="Test Portal" inherit_id="portal.portal_sidebar" primary="True">
        <xpath expr="//div[hasclass('o_portal_sidebar')]" position="inside">
            <!-- chatter -->
            <div>
                <h3>Communication history</h3>
                <t t-call="portal.message_thread"/>
            </div>
        </xpath>
    </template>

</odoo>

```

