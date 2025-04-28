# Odoo Module: l10n_tr_nilvera

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models


def _l10n_tr_nilvera_post_init(env):
    env['res.lang']._activate_lang('tr_TR')

```

## File: __manifest__.py

```python
{
    'name': 'Türkiye - Nilvera',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
Base module containing core functionalities required by other Nilvera modules.
    """,
    'depends': ['l10n_tr'],
    'data': [
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'data/uom_data.xml',
    ],
    'post_init_hook': '_l10n_tr_nilvera_post_init',
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
-- disable l10n_tr_nilvera integration
UPDATE res_company
   SET l10n_tr_nilvera_api_key = NULL,
       l10n_tr_nilvera_environment = 'sandbox',
       l10n_tr_nilvera_purchase_journal_id = NULL;

```

## File: data\uom_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="product_uom_categ_energy" model="uom.category">
        <field name="name">Energy</field>
    </record>

    <record id="product_uom_pk" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Parcel</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_pf" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Pallet</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_cr" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Crate</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_standard_cubic_meter" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="name">Standard Cubic Meter</field>
        <field name="factor_inv" eval="1000.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_sa" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Bags</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_cmq" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="name">Cubic Centimeter - cm³</field>
        <field name="factor" eval="1000.0"/>
        <field name="uom_type">smaller</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_mlt" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="name">Milliliter - ml</field>
        <field name="factor" eval="1000.0"/>
        <field name="uom_type">smaller</field>
    </record>
    <record id="product_uom_mmq" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="name">Cubic Millimeter - mm³</field>
        <field name="factor" eval="1000000.0"/>
        <field name="uom_type">smaller</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_cmk" model="uom.uom">
        <field name="category_id" ref="uom.uom_categ_surface"/>
        <field name="name">Square Centimeter - cm²</field>
        <field name="factor" eval="10000.0"/>
        <field name="uom_type">smaller</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_bg" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Pack</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_bx" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Box</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_pr" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Pair</field>
        <field name="factor" eval="0.5"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_mgm" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="name">Milligram - mg</field>
        <field name="factor" eval="1000000.0"/>
        <field name="uom_type">smaller</field>
    </record>
    <record id="product_uom_mon" model="uom.uom">
        <field name="category_id" ref="uom.uom_categ_wtime"/>
        <field name="name">Month</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_gt" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="name">Gross Ton</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_ann" model="uom.uom">
        <field name="category_id" ref="uom.uom_categ_wtime"/>
        <field name="name">Year</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_d61" model="uom.uom">
        <field name="category_id" ref="uom.uom_categ_wtime"/>
        <field name="name">Minute</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">smaller</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_d62" model="uom.uom">
        <field name="category_id" ref="uom.uom_categ_wtime"/>
        <field name="name">Second</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">smaller</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_pa" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Package</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_kwt" model="uom.uom">
        <field name="category_id" ref="l10n_tr_nilvera.product_uom_categ_energy"/>
        <field name="name">Kilowatt</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">reference</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_mwh" model="uom.uom">
        <field name="category_id" ref="l10n_tr_nilvera.product_uom_categ_energy"/>
        <field name="name">Megawatt Hour</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_kwh" model="uom.uom">
        <field name="category_id" ref="l10n_tr_nilvera.product_uom_categ_energy"/>
        <field name="name">Kilowatt Hour</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
    <record id="product_uom_set" model="uom.uom">
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="name">Set</field>
        <field name="factor" eval="1.0"/>
        <field name="uom_type">bigger</field>
        <field name="active">False</field>
    </record>
</odoo>

```

## File: lib\nilvera_client.py

```python
import logging
import requests
from datetime import datetime
from json import JSONDecodeError
from pprint import pformat

from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

def _get_nilvera_client(company, timeout_limit=None):
    return NilveraClient(
        environment=company.l10n_tr_nilvera_environment,
        api_key=company.l10n_tr_nilvera_api_key,
        timeout_limit=timeout_limit,
    )


class NilveraClient:
    def __init__(self, environment=None, api_key=None, timeout_limit=None):
        self.is_production = environment and environment == 'production'
        self.base_url = 'https://api.nilvera.com' if self.is_production else 'https://apitest.nilvera.com'
        self.timeout_limit = min(timeout_limit or 10, 30)

        self.__session = requests.Session()
        self.__session.headers.update({'Accept': 'application/json'})
        if api_key:
            self.__session.headers['Authorization'] = 'Bearer ' + api_key

    def __enter__(self):
        return self

    def __exit__(self, type, value, traceback):
        if hasattr(self, '_NilveraClient__session'):
            self.__session.close()

    def request(self, method, endpoint, params=None, json=None, files=None, handle_response=True):
        start = datetime.utcnow()
        url = self.base_url + endpoint

        try:
            response = self.__session.request(
                method, url,
                timeout=self.timeout_limit,
                params=params,
                json=json,
                files=files,
            )
        except requests.exceptions.RequestException as e:
            _logger.error("Network error during request: %s", e)
            raise UserError("Network connectivity issue. Please check your internet connection and try again.")

        end = datetime.utcnow()
        self._log_request(method, start, end, url, params, json, response)

        if handle_response:
            return self.handle_response(response)
        return response

    def _log_request(self, method, start, end, url, params, json, response):
        _logger.info(
            "%(method)s\nstart=%(start)s\nend=%(end)s\nurl=%(url)s\nparams=%(params)s\njson=%(json)s\nresponse=%(response)s",
            {
                "method": method,
                "start": start,
                "end": end,
                "url": pformat(url),
                "params": pformat(params),
                "json": pformat(json),
                "response": pformat(response),
            },
        )

    def handle_response(self, response):
        if response.status_code in {401, 403}:
            raise UserError("Oops, seems like you're unauthorised to do this. Try another API key with more rights or contact Nilvera.")
        elif 403 < response.status_code < 600:
            raise UserError("Odoo could not perform this action at the moment, try again later.\n%s - %s" % (response.reason, response.code))

        try:
            return response.json()
        except JSONDecodeError:
            _logger.exception("Invalid JSON response: %s", response.text)
            raise UserError("An error occurred. Try again later.")

```

## File: models\account_journal.py

```python
from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    l10n_tr_nilvera_api_key = fields.Char(related='company_id.l10n_tr_nilvera_api_key')
    is_nilvera_journal = fields.Boolean(string="Journal used for Nilvera")

```

## File: models\l10n_tr_nilvera_alias.py

```python
from odoo import fields, models


class L10nTrNilveraAlias(models.Model):
    _name = 'l10n_tr.nilvera.alias'
    _description = "Customer Alias on Nilvera"

    name = fields.Char()
    partner_id = fields.Many2one('res.partner')

```

## File: models\res_company.py

```python
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_tr_nilvera_api_key = fields.Char(string="Nilvera API key", groups='base.group_system')
    l10n_tr_nilvera_environment = fields.Selection(
        string="Nilvera Environment",
        selection=[
            ('sandbox', "Test"),
            ('production', "Production"),
        ],
        required=True,
        default='sandbox',
    )
    l10n_tr_nilvera_purchase_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="Nilvera Purchase Journal",
        domain=[('type', '=', 'purchase')],
        store=True,
        compute='_compute_l10n_tr_nilvera_purchase_journal_id',
        inverse='_inverse_l10n_tr_nilvera_purchase_journal_id',
    )

    def _compute_l10n_tr_nilvera_purchase_journal_id(self):
        purchase_journals = self.env['account.journal'].search([('type', '=', 'purchase')])
        for company in self:
            if not company.l10n_tr_nilvera_purchase_journal_id:
                company.l10n_tr_nilvera_purchase_journal_id = purchase_journals.filtered_domain(self.env['account.journal']._check_company_domain(company))[:1]
                company.l10n_tr_nilvera_purchase_journal_id.is_nilvera_journal = True

    def _inverse_l10n_tr_nilvera_purchase_journal_id(self):
        # dict(company: journals)
        journals_to_reset_grouped = self.env['account.journal'].search([
            ('company_id', 'in', self.ids),
            ('is_nilvera_journal', '=', True),
        ]).grouped('company_id')
        for company in self:
            # This avoids having 2 or more journals from the same company with
            # `is_nilvera_journal` set to True (which could occur after changes).
            if journals_to_reset := journals_to_reset_grouped.get(company):
                journals_to_reset.is_nilvera_journal = False
            company.l10n_tr_nilvera_purchase_journal_id.is_nilvera_journal = True

```

## File: models\res_config_settings.py

```python
from odoo import fields, models, _
from odoo.addons.l10n_tr_nilvera.lib.nilvera_client import _get_nilvera_client


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_tr_nilvera_api_key = fields.Char(
        related='company_id.l10n_tr_nilvera_api_key',
        string="Nilvera API key",
        readonly=False,
    )
    l10n_tr_nilvera_environment = fields.Selection(
        related='company_id.l10n_tr_nilvera_environment',
        string="Nilvera Environment",
        required=True,
        readonly=False,
    )
    l10n_tr_nilvera_purchase_journal_id = fields.Many2one(
        related='company_id.l10n_tr_nilvera_purchase_journal_id',
        readonly=False,
    )

    def nilvera_ping(self):
        """ Test the connection and the API key. """
        self.check_access_rule('read')  # To make sure not everyone can call this method as it's public.
        with _get_nilvera_client(self.env.company) as client:
            # As there is no endpoint to ping Nilvera to make sure the connection works, try an endpoint to get the
            # company's data and this way we can verify the connection and the tax ID in the same step.
            response = client.request("GET", "/general/Company", handle_response=False)
            if response.status_code == 200:
                nilvera_registered_tax_number = response.json().get('TaxNumber')
                if self.env.company.vat == nilvera_registered_tax_number:
                    self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                        'type': 'success',
                        'message': _("Nilvera connection successful!"),
                    })
                else:
                    self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                        'type': 'success',
                        'message': _("Nilvera connection successful but the tax number on Nilvera and Odoo doesn't match. Check Nilvera."),
                    })
            elif response.status_code == 401:
                self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                    'type': 'danger',
                    'message': _("Nilvera connection was unsuccessful, check the API key."),
                })
            else:
                self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                    'type': 'danger',
                    'message': _("An error occurred. Try again later."),
                })

```

## File: models\res_partner.py

```python
import logging
import urllib.parse

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.addons.l10n_tr_nilvera.lib.nilvera_client import _get_nilvera_client


_logger = logging.getLogger(__name__)


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = ['res.partner']

    invoice_edi_format = fields.Selection(selection_add=[('ubl_tr', "UBL TR 1.2")])
    l10n_tr_nilvera_customer_status = fields.Selection(
        selection=[
            ('not_checked', "Not Checked"),
            ('earchive', "E-Archive"),
            ('einvoice', "E-Invoice"),
        ],
        string="Nilvera Status",
        compute='_compute_nilvera_customer_status_and_alias_id',
        store=True,
        copy=False,
        default='not_checked',
    )
    l10n_tr_nilvera_customer_alias_id = fields.Many2one(
        comodel_name='l10n_tr.nilvera.alias',
        string="Alias",
        compute='_compute_nilvera_customer_status_and_alias_id',
        domain="[('partner_id', '=', id)]",
        copy=False,
        store=True,
        readonly=False,
    )

    # This field is only used technically for optimisation purposes. It's needed for check_nilvera_customer.
    l10n_tr_nilvera_customer_alias_ids = fields.One2many(
        comodel_name='l10n_tr.nilvera.alias',
        inverse_name="partner_id",
    )

    @api.depends('vat', 'invoice_edi_format')
    def _compute_nilvera_customer_status_and_alias_id(self):
        for partner in self:
            if partner.vat and partner.invoice_edi_format == 'ubl_tr':
                try:
                    partner.check_nilvera_customer()
                except UserError:
                    # In case of an internet connection issue, exit silently.
                    continue
            else:
                # Reset the alias if no VAT or UBL format changed.
                partner.l10n_tr_nilvera_customer_status = 'not_checked'
                partner.l10n_tr_nilvera_customer_alias_id = False

    def check_nilvera_customer(self):
        self.ensure_one()
        if not self.vat:
            return

        with _get_nilvera_client(self.env.company) as client:
            response = client.request("GET", "/general/GlobalCompany/Check/TaxNumber/" + urllib.parse.quote(self.vat), handle_response=False)
            if response.status_code == 200:
                query_result = response.json()

                if not query_result:
                    self.l10n_tr_nilvera_customer_status = 'earchive'
                    self.l10n_tr_nilvera_customer_alias_id = False
                else:
                    self.l10n_tr_nilvera_customer_status = 'einvoice'

                    # We need to sync the data from the API with the records in database.
                    aliases = {result.get('Name') for result in query_result}
                    persisted_aliases = self.l10n_tr_nilvera_customer_alias_ids
                    # Find aliases to add (in query result but not in database).
                    aliases_to_add = aliases - set(persisted_aliases.mapped('name'))
                    # Find aliases to remove (in database but not in query result).
                    aliases_to_remove = set(persisted_aliases.mapped('name')) - aliases

                    newly_persisted_aliases = self.env['l10n_tr.nilvera.alias'].create([{
                        'name': alias_name,
                        'partner_id': self.id,
                    } for alias_name in aliases_to_add])
                    to_keep = persisted_aliases.filtered(lambda a: a.name not in aliases_to_remove)
                    (persisted_aliases - to_keep).unlink()

                    # If no alias was previously selected, automatically select the first alias.
                    remaining_aliases = newly_persisted_aliases | to_keep
                    if not self.l10n_tr_nilvera_customer_alias_id and remaining_aliases:
                        self.l10n_tr_nilvera_customer_alias_id = remaining_aliases[0]

    def _get_edi_builder(self, invoice_edi_format):
        # EXTENDS 'account_edi_ubl_cii'
        if invoice_edi_format == 'ubl_tr':
            return self.env['account.edi.xml.ubl.tr']
        return super()._get_edi_builder(invoice_edi_format)

    def _get_ubl_cii_formats_info(self):
        # EXTENDS 'account_edi_ubl_cii'
        formats_info = super()._get_ubl_cii_formats_info()
        formats_info['ubl_tr'] = {'countries': ['TR']}
        return formats_info

```

## File: models\uom_uom.py

```python
from odoo import models

UOM_TO_UNECE_CODE = {
    'l10n_tr_nilvera.product_uom_pk': 'PK',
    'l10n_tr_nilvera.product_uom_pf': 'PF',
    'l10n_tr_nilvera.product_uom_cr': 'CR',
    'l10n_tr_nilvera.product_uom_standard_cubic_meter': 'SM3',
    'l10n_tr_nilvera.product_uom_sa': 'SA',
    'l10n_tr_nilvera.product_uom_cmq': 'CMQ',
    'l10n_tr_nilvera.product_uom_mlt': 'MLT',
    'l10n_tr_nilvera.product_uom_mmq': 'MMQ',
    'l10n_tr_nilvera.product_uom_cmk': 'CMK',
    'l10n_tr_nilvera.product_uom_bg': 'BG',
    'l10n_tr_nilvera.product_uom_bx': 'BX',
    'l10n_tr_nilvera.product_uom_pr': 'PR',
    'l10n_tr_nilvera.product_uom_mgm': 'MGM',
    'l10n_tr_nilvera.product_uom_mon': 'MON',
    'l10n_tr_nilvera.product_uom_gt': 'GT',
    'l10n_tr_nilvera.product_uom_ann': 'ANN',
    'l10n_tr_nilvera.product_uom_d61': 'D61',
    'l10n_tr_nilvera.product_uom_d62': 'D62',
    'l10n_tr_nilvera.product_uom_pa': 'PA',
    'l10n_tr_nilvera.product_uom_mwh': 'MWH',
    'l10n_tr_nilvera.product_uom_kwh': 'KWH',
    'l10n_tr_nilvera.product_uom_kwt': 'KWT',
    'l10n_tr_nilvera.product_uom_set': 'SET',
}


class Uom(models.Model):
    _inherit = 'uom.uom'

    def _get_unece_code(self):
        """ This depends on the mapping from https://developer.nilvera.com/en/code-lists#birim-kodlari """
        unece_code = super()._get_unece_code()
        if unece_code == 'C62':
            xml_id = self.get_external_id()
            if xml_id and self.id in xml_id:
                return UOM_TO_UNECE_CODE.get(xml_id[self.id], 'C62')
        return unece_code

```

## File: models\__init__.py

```python
from . import account_journal
from . import l10n_tr_nilvera_alias
from . import res_company
from . import res_config_settings
from . import res_partner
from . import uom_uom

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_tr_nilvera_alias_readonly,l10n_tr.nilvera.alias.readonly,model_l10n_tr_nilvera_alias,account.group_account_readonly,1,0,0,0
access_l10n_tr_nilvera_alias,l10n_tr.nilvera.alias,model_l10n_tr_nilvera_alias,account.group_account_user,1,1,1,1

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n.tr.nilvera</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block name="integration" position="inside">
                <field name="country_code" invisible="True"/>
                <setting id="nilvera_settings" string="Nilvera Electronic Document Invoicing" help="Configure Nilvera settings" invisible="country_code != 'TR'">
                    <div class="content-group">
                        <div class="row mt16">
                            <label string="Environment" for="l10n_tr_nilvera_environment" class="col-lg-6 o_light_label"/>
                            <field name="l10n_tr_nilvera_environment"/>
                        </div>
                        <div class="row">
                            <label string="API KEY" for="l10n_tr_nilvera_api_key" class="col-lg-6 o_light_label" />
                            <field name="l10n_tr_nilvera_api_key"/>
                        </div>
                        <div class="row">
                            <label string="Incoming Invoices Journal"
                                   for="l10n_tr_nilvera_purchase_journal_id"
                                   class="col-lg-6 o_light_label"/>
                            <field name="l10n_tr_nilvera_purchase_journal_id"/>
                        </div>
                        <div class="mt16" invisible="not l10n_tr_nilvera_api_key">
                            <a href="https://portal.nilvera.com/" target="_new">
                                <i title="Go to Nilvera portal" role="img" aria-label="Go to Nilvera portal" class="fa fa-external-link-square fa-fw"/>
                                Nilvera portal
                            </a>
                            <button name="nilvera_ping" type="object" class="btn-link">
                                <i title="Test connection" role="img" aria-label="Test connection" class="fa fa-plug fa-fw"/>
                                Test connection
                            </button>
                        </div>
                    </div>
                </setting>
            </block>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<odoo>
    <record id="view_partner_property_form_inherit_ubl_tr" model="ir.ui.view">
        <field name="name">res.partner.property.form.inherit.ubl.tr</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account_edi_ubl_cii.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='invoice_edi_format']" position="after">
                <label for="l10n_tr_nilvera_customer_status" invisible="invoice_edi_format != 'ubl_tr'"/>
                <div class="row" invisible="invoice_edi_format != 'ubl_tr'">
                        <div class="col-4">
                            <field name="l10n_tr_nilvera_customer_status"/>
                        </div>
                        <div class="col-8 pt-0">
                            <button name="check_nilvera_customer"
                                    class="btn btn-secondary"
                                    type="object"
                                    string="Verify"
                                    help="Verify partner on Nilvera"/>
                        </div>
                    </div>
                <field name="l10n_tr_nilvera_customer_alias_id" invisible="invoice_edi_format != 'ubl_tr' or l10n_tr_nilvera_customer_status != 'einvoice'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

