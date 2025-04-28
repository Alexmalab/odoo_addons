# Odoo Module: test_mass_mailing

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import data
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Mass Mail Tests',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 8765,
    'summary': 'Mass Mail Tests: feature and performance tests for mass mailing',
    'description': """This module contains tests related to mass mailing. Those
are contained in a separate module to use specific test models defined in
test_mail. """,
    'depends': ['test_mail', 'mass_mailing'],
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

## File: data\mail_test_data.py

```python

MAIL_TEMPLATE = """Return-Path: <whatever-2a840@postmaster.twitter.com>
To: {to}
cc: {cc}
Received: by mail1.openerp.com (Postfix, from userid 10002)
    id 5DF9ABFB2A; Fri, 10 Aug 2012 16:16:39 +0200 (CEST)
From: {email_from}
Subject: {subject}
MIME-Version: 1.0
Content-Type: multipart/alternative;
    boundary="----=_Part_4200734_24778174.1344608186754"
Date: Fri, 10 Aug 2012 14:16:26 +0000
Message-ID: {msg_id}
{extra}
------=_Part_4200734_24778174.1344608186754
Content-Type: text/plain; charset=utf-8
Content-Transfer-Encoding: quoted-printable

I would gladly answer to your mass mailing !

--
Your Dear Customer
------=_Part_4200734_24778174.1344608186754
Content-Type: text/html; charset=utf-8
Content-Transfer-Encoding: quoted-printable

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
 <head>=20
  <meta http-equiv=3D"Content-Type" content=3D"text/html; charset=3Dutf-8" />
 </head>=20
 <body style=3D"margin: 0; padding: 0; background: #ffffff;-webkit-text-size-adjust: 100%;">=20

  <p>I would gladly answer to your mass mailing !</p>

  <p>--<br/>
     Your Dear Customer
  <p>
 </body>
</html>
------=_Part_4200734_24778174.1344608186754--
"""

```

## File: data\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_test_data

```

## File: models\mailing_test_models_cornercases.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailingTestPartnerUnstored(models.Model):
    """ Check mailing with unstored fields """
    _description = 'Mailing Model without stored partner_id'
    _name = 'mailing.test.partner.unstored'
    _inherit = ['mail.thread.blacklist']
    _primary_email = 'email_from'

    name = fields.Char()
    email_from = fields.Char()
    partner_id = fields.Many2one(
        'res.partner', 'Customer',
        compute='_compute_partner_id',
        store=False)

    @api.depends('email_from')
    def _compute_partner_id(self):
        partners = self.env['res.partner'].search(
            [('email_normalized', 'in', self.filtered('email_from').mapped('email_normalized'))]
        )
        self.partner_id = False
        for record in self.filtered('email_from'):
            record.partner_id = next(
                (partner.id for partner in partners
                 if partner.email_normalized == record.email_normalized),
                False
            )

```

## File: models\mass_mail_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MassMailTest(models.Model):
    """ A very simple model only inheriting from mail.thread to test pure mass
    mailing features and base performances. """
    _description = 'Simple Mass Mailing Model'
    _name = 'mass.mail.test'
    _inherit = ['mail.thread', 'mail.address.mixin']
    _primary_email = 'email_from'

    name = fields.Char()
    email_from = fields.Char()


class MassMailTestBlacklist(models.Model):
    """ Model using blacklist mechanism for mass mailing. """
    _description = 'Mass Mailing Model w Blacklist'
    _name = 'mass.mail.test.bl'
    _inherit = ['mail.thread.blacklist']

    _primary_email = 'email_from'  # blacklist field to check

    name = fields.Char()
    email_from = fields.Char()
    user_id = fields.Many2one(
        'res.users', 'Responsible',
        tracking=True)
    umbrella_id = fields.Many2one(
        'mail.test', 'Meta Umbrella Record',
        tracking=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_test_models_cornercases
from . import mass_mail_test

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mass_mail_test_all,mass.mail.test.all,model_mass_mail_test,,0,0,0,0
access_mass_mail_test_user,mass.mail.test.user,model_mass_mail_test,base.group_user,1,1,1,1
access_mass_mail_test_bl_all,mass.mail.test.bl.all,model_mass_mail_test_bl,,0,0,0,0
access_mass_mail_test_bl_user,mass.mail.test.bl.user,model_mass_mail_test_bl,base.group_user,1,1,1,1
access_mailing_test_partner_unstored_all,access.mailing.test.partner.unstored.all,model_mailing_test_partner_unstored,,0,0,0,0
access_mailing_test_partner_unstored_user,access.mailing.test.partner.unstored.user,model_mailing_test_partner_unstored,base.group_user,1,1,1,1

```

