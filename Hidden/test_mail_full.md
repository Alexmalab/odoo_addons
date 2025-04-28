# Odoo Module: test_mail_full

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import tests

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
and mail-related sub modules. Those tests are contained in a separate module as it
contains models used only to perform tests independently to functional aspects of
real applications. """,
    'depends': [
        'test_mail',
        'mail',
        'mail_bot',
        # 'snailmail',
        'mass_mailing',
        'mass_mailing_sms',  # adds portal
        'phone_validation',
        'sms',
    ],
    'data': [
        'security/ir.model.access.csv',
    ],
    'demo': [
    ],
    'installable': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: models\test_mail_models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailTestPortal(models.Model):
    """ A model intheriting from mail.thread with some fields used for portal
    sharing, like a partner, ..."""
    _description = 'Chatter Model for Portal'
    _name = 'mail.test.portal'
    _inherit = [
        'mail.thread',
        'portal.mixin',
    ]

    name = fields.Char()
    partner_id = fields.Many2one('res.partner', 'Customer')

    def _compute_access_url(self):
        self.access_url = False
        for record in self.filtered('id'):
            record.access_url = '/my/test_portal/%s' % self.id


class MailTestSMS(models.Model):
    """ A model inheriting from mail.thread with some fields used for SMS
    gateway, like a partner, a specific mobile phone, ... """
    _description = 'Chatter Model for SMS Gateway'
    _name = 'mail.test.sms'
    _inherit = ['mail.thread']
    _order = 'name asc, id asc'

    name = fields.Char()
    subject = fields.Char()
    email_from = fields.Char()
    phone_nbr = fields.Char()
    mobile_nbr = fields.Char()
    customer_id = fields.Many2one('res.partner', 'Customer')

    def _sms_get_partner_fields(self):
        return ['customer_id']

    def _sms_get_number_fields(self):
        return ['phone_nbr', 'mobile_nbr']


class MailTestSMSBL(models.Model):
    """ A model inheriting from mail.thread with some fields used for SMS
    gateway, like a partner, a specific mobile phone, ... """
    _description = 'Chatter Model for SMS Gateway'
    _name = 'mail.test.sms.bl'
    _inherit = ['mail.thread.phone']
    _order = 'name asc, id asc'

    name = fields.Char()
    subject = fields.Char()
    email_from = fields.Char()
    phone_nbr = fields.Char()
    mobile_nbr = fields.Char()
    customer_id = fields.Many2one('res.partner', 'Customer')

    def _sms_get_partner_fields(self):
        return ['customer_id']

    def _sms_get_number_fields(self):
        return ['phone_nbr', 'mobile_nbr']


class MailTestSMSSoLike(models.Model):
    """ A model like sale order having only a customer, not specific phone
    or mobile fields. """
    _description = 'Chatter Model for SMS Gateway (Partner only)'
    _name = 'mail.test.sms.partner'
    _inherit = ['mail.thread']

    name = fields.Char()
    partner_id = fields.Many2one('res.partner', 'Customer')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import test_mail_models

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mail_test_portal_all,mail.test.portal.all,model_mail_test_portal,,1,0,0,0
access_mail_test_portal_user,mail.test.portal.user,model_mail_test_portal,base.group_user,1,1,1,1
access_mail_test_sms_all,mail.test.sms.all,model_mail_test_sms,,0,0,0,0
access_mail_test_sms_user,mail.test.sms.user,model_mail_test_sms,base.group_user,1,1,1,1
access_mail_test_sms_bl_all,mail.test.sms.bl.all,model_mail_test_sms_bl,,0,0,0,0
access_mail_test_sms_bl_user,mail.test.sms.bl.user,model_mail_test_sms_bl,base.group_user,1,1,1,1
access_mail_test_sms_partner_all,mail.test.sms.partner.all,model_mail_test_sms_partner,,0,0,0,0
access_mail_test_sms_partner_user,mail.test.sms.partner.user,model_mail_test_sms_partner,base.group_user,1,1,1,1

```

