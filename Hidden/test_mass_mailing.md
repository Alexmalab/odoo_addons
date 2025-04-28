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
are present in a separate module to use specific test models defined in
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

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import models

_logger = logging.getLogger(__name__)


class Mailing(models.Model):
    _inherit = 'mailing.mailing'

    def _get_opt_out_list(self):
        """Returns a set of emails opted-out in target model"""
        self.ensure_one()
        if self.mailing_model_real == 'mailing.test.optout':
            res_ids = self._get_recipients()
            opt_out_contacts = set(self.env['mailing.test.optout'].search([
                ('id', 'in', res_ids),
                ('opt_out', '=', True)
            ]).mapped('email_normalized'))
            _logger.info(
                "Mass-mailing %s targets %s, optout: %s emails",
                self, self.mailing_model_real, len(opt_out_contacts))
            return opt_out_contacts
        return super(Mailing, self)._get_opt_out_list()

```

## File: models\mailing_models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailingCustomer(models.Model):
    """ A model inheriting from mail.thread with a partner field, to test
    mass mailing flows involving checking partner email. """
    _description = 'Mailing with partner'
    _name = 'mailing.test.customer'
    _inherit = ['mail.thread']

    name = fields.Char()
    email_from = fields.Char(compute='_compute_email_from', readonly=False, store=True)
    customer_id = fields.Many2one('res.partner', 'Customer', tracking=True)

    @api.depends('customer_id')
    def _compute_email_from(self):
        for mailing in self.filtered(lambda rec: not rec.email_from and rec.customer_id):
            mailing.email_from = mailing.customer_id.email

    def _message_get_default_recipients(self):
        """ Default recipient checks for 'partner_id', here the field is named
        'customer_id'. """
        default_recipients = super()._message_get_default_recipients()
        for record in self:
            if record.customer_id:
                default_recipients[record.id] = {
                    'email_cc': False,
                    'email_to': False,
                    'partner_ids': record.customer_id.ids,
                }
        return default_recipients


class MailingSimple(models.Model):
    """ Model only inheriting from mail.thread to test base mailing features and
    performances. """
    _description = 'Simple Mailing'
    _name = 'mailing.test.simple'
    _inherit = ['mail.thread']

    name = fields.Char()
    email_from = fields.Char()


class MailingUTM(models.Model):
    """ Model inheriting from mail.thread and utm.mixin for checking utm of mailing
    is caught and set on reply """
    _description = 'Mailing: UTM enabled to test UTM sync with mailing'
    _name = 'mailing.test.utm'
    _inherit = ['mail.thread', 'utm.mixin']

    name = fields.Char()


class MailingBLacklist(models.Model):
    """ Model using blacklist mechanism for mass mailing features. """
    _description = 'Mailing Blacklist Enabled'
    _name = 'mailing.test.blacklist'
    _inherit = ['mail.thread.blacklist']
    _primary_email = 'email_from'

    name = fields.Char()
    email_from = fields.Char()
    customer_id = fields.Many2one('res.partner', 'Customer', tracking=True)
    user_id = fields.Many2one('res.users', 'Responsible', tracking=True)

    def _message_get_default_recipients(self):
        """ Default recipient checks for 'partner_id', here the field is named
        'customer_id'. """
        default_recipients = super()._message_get_default_recipients()
        for record in self:
            if record.customer_id:
                default_recipients[record.id] = {
                    'email_cc': False,
                    'email_to': False,
                    'partner_ids': record.customer_id.ids,
                }
        return default_recipients


class MailingOptOut(models.Model):
    """ Model using blacklist mechanism and a hijacked opt-out mechanism for
    mass mailing features. """
    _description = 'Mailing Blacklist / Optout Enabled'
    _name = 'mailing.test.optout'
    _inherit = ['mail.thread.blacklist']
    _primary_email = 'email_from'

    name = fields.Char()
    email_from = fields.Char()
    opt_out = fields.Boolean()
    customer_id = fields.Many2one('res.partner', 'Customer', tracking=True)
    user_id = fields.Many2one('res.users', 'Responsible', tracking=True)

    def _message_get_default_recipients(self):
        """ Default recipient checks for 'partner_id', here the field is named
        'customer_id'. """
        default_recipients = super()._message_get_default_recipients()
        for record in self:
            if record.customer_id:
                default_recipients[record.id] = {
                    'email_cc': False,
                    'email_to': False,
                    'partner_ids': record.customer_id.ids,
                }
        return default_recipients

class MailingTestPartner(models.Model):
    _description = 'Mailing Model with partner_id'
    _name = 'mailing.test.partner'
    _inherit = ['mail.thread.blacklist']
    _primary_email = 'email_from'

    name = fields.Char()
    email_from = fields.Char()
    partner_id = fields.Many2one('res.partner', 'Customer')


class MailingPerformance(models.Model):
    """ A very simple model only inheriting from mail.thread to test pure mass
    mailing performances. """
    _name = 'mailing.performance'
    _description = 'Mailing: base performance'
    _inherit = ['mail.thread']

    name = fields.Char()
    email_from = fields.Char()


class MailingPerformanceBL(models.Model):
    """ Model using blacklist mechanism for mass mailing performance. """
    _name = 'mailing.performance.blacklist'
    _description = 'Mailing: blacklist performance'
    _inherit = ['mail.thread.blacklist']
    _primary_email = 'email_from'  # blacklist field to check

    name = fields.Char()
    email_from = fields.Char()
    user_id = fields.Many2one(
        'res.users', 'Responsible',
        tracking=True)
    container_id = fields.Many2one(
        'mail.test.container', 'Meta Container Record',
        tracking=True)

```

## File: models\mailing_models_cornercase.py

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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_mailing
from . import mailing_models
from . import mailing_models_cornercase

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mailing_test_customer_all,access.mailing.test.customer.all,model_mailing_test_customer,,0,0,0,0
access_mailing_test_customer_user,access.mailing.test.customer.user,model_mailing_test_customer,base.group_user,1,1,1,1
access_mailing_test_simple_all,access.mailing.test.simple.all,model_mailing_test_simple,,0,0,0,0
access_mailing_test_simple_user,access.mailing.test.simple.user,model_mailing_test_simple,base.group_user,1,1,1,1
access_mailing_test_blacklist_all,access.mailing.test.blacklist.all,model_mailing_test_blacklist,,0,0,0,0
access_mailing_test_blacklist_user,access.mailing.test.blacklist.user,model_mailing_test_blacklist,base.group_user,1,1,1,1
access_mailing_test_optout_all,access.mailing.test.optout.all,model_mailing_test_optout,,0,0,0,0
access_mailing_test_optout_user,access.mailing.test.optout.user,model_mailing_test_optout,base.group_user,1,1,1,1
access_mailing_performance_all,access.mailing.performance.all,model_mailing_performance,,0,0,0,0
access_mailing_performance_user,access.mailing.performance.user,model_mailing_performance,base.group_user,1,1,1,1
access_mailing_performance_blacklist_all,access.mailing.performance.blacklist.all,model_mailing_performance_blacklist,,0,0,0,0
access_mailing_performance_blacklist_user,access.mailing.performance.blacklist.user,model_mailing_performance_blacklist,base.group_user,1,1,1,1
access_mailing_test_partner_all,access.mailing.test.partner.all,model_mailing_test_partner,,0,0,0,0
access_mailing_test_partner_user,access.mailing.test.partner.user,model_mailing_test_partner,base.group_user,1,1,1,1
access_mailing_test_partner_unstored_all,access.mailing.test.partner.unstored.all,model_mailing_test_partner_unstored,,0,0,0,0
access_mailing_test_partner_unstored_user,access.mailing.test.partner.unstored.user,model_mailing_test_partner_unstored,base.group_user,1,1,1,1
access_mailing_test_utm_all,access.mailing.test.utm.all,model_mailing_test_utm,,0,0,0,0
access_mailing_test_utm_user,access.mailing.test.utm.user,model_mailing_test_utm,base.group_user,1,1,1,1

```

