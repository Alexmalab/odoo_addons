# Odoo Module: l10n_fr_pos_cert

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from odoo import api, SUPERUSER_ID


def _setup_inalterability(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    # enable ping for this module
    env['publisher_warranty.contract'].update_notification(cron_mode=True)

    fr_companies = env['res.company'].search([('partner_id.country_id.code', 'in', env['res.company']._get_unalterable_country())])
    if fr_companies:
        fr_companies._create_secure_sequence(['l10n_fr_pos_cert_sequence_id'])

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'France - VAT Anti-Fraud Certification for Point of Sale (CGI 286 I-3 bis)',
    'version': '1.0',
    'category': 'Accounting/Localizations/Point of Sale',
    'description': """
This add-on brings the technical requirements of the French regulation CGI art. 286, I. 3° bis that stipulates certain criteria concerning the inalterability, security, storage and archiving of data related to sales to private individuals (B2C).
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Install it if you use the Point of Sale app to sell to individuals.

The module adds following features:

    Inalterability: deactivation of all the ways to cancel or modify key data of POS orders, invoices and journal entries

    Security: chaining algorithm to verify the inalterability

    Storage: automatic sales closings with computation of both period and cumulative totals (daily, monthly, annually)

    Access to download the mandatory Certificate of Conformity delivered by Odoo SA (only for Odoo Enterprise users)
""",
    'depends': ['l10n_fr', 'point_of_sale'],
    'installable': True,
    'auto_install': True,
    'application': False,
    'data': [
        'views/account_views.xml',
        'views/l10n_fr_pos_cert_templates.xml',
        'views/pos_views.xml',
        'views/account_sale_closure.xml',
        'views/pos_inalterability_menuitem.xml',
        'report/pos_hash_integrity.xml',
        'data/account_sale_closure_cron.xml',
        'security/ir.model.access.csv',
        'security/account_closing_intercompany.xml',
    ],
    'qweb': ['static/src/xml/OrderReceipt.xml'],
    'post_init_hook': '_setup_inalterability',
    'license': 'LGPL-3',
}

```

## File: data\account_sale_closure_cron.xml

```xml
<odoo>
    <record model="ir.cron" id="account_sale_closing_daily">
        <field name="name">Generate Daily Sales Closing</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_account_sale_closing"/>
        <field name="state">code</field>
        <field name="code">model._automated_closing('daily')</field>
    </record>

    <record model="ir.cron" id="account_sale_closing_monthly">
        <field name="name">Generate Monthly Sales Closing</field>
        <field name="interval_number">1</field>
        <field name="interval_type">months</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_account_sale_closing"/>
        <field name="state">code</field>
        <field name="code">model._automated_closing('monthly')</field>
    </record>

    <record model="ir.cron" id="account_sale_closing_annually">
        <field name="name">Generate Annual Sales Closing</field>
        <field name="interval_number">12</field>
        <field name="interval_type">months</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
        <field name="model_id" ref="model_account_sale_closing"/>
        <field name="state">code</field>
        <field name="code">model._automated_closing('annually')</field>
    </record>
</odoo>

```

## File: models\account_bank_statement.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api
from odoo.tools.translate import _
from odoo.exceptions import UserError


class AccountBankStatement(models.Model):
    _inherit = 'account.bank.statement'

    def unlink(self):
        for statement in self:
            if not statement.company_id._is_accounting_unalterable() or not statement.journal_id.pos_payment_method_ids:
                continue
            if statement.state != 'open':
                raise UserError(_('You cannot modify anything on a bank statement (name: %s) that was created by point of sale operations.') % (statement.name))
        return super(AccountBankStatement, self).unlink()


class AccountBankStatementLine(models.Model):
    _inherit = 'account.bank.statement.line'

    def unlink(self):
        for line in self.filtered(lambda s: s.company_id._is_accounting_unalterable() and s.journal_id.pos_payment_method_ids):
            raise UserError(_('You cannot modify anything on a bank statement line (name: %s) that was created by point of sale operations.') % (line.name,))
        return super(AccountBankStatementLine, self).unlink()

```

## File: models\account_closing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime, timedelta

from odoo import models, api, fields
from odoo.fields import Datetime as FieldDateTime
from odoo.tools.translate import _
from odoo.exceptions import UserError
from odoo.osv.expression import AND


class AccountClosing(models.Model):
    """
    This object holds an interval total and a grand total of the accounts of type receivable for a company,
    as well as the last account_move that has been counted in a previous object
    It takes its earliest brother to infer from when the computation needs to be done
    in order to compute its own data.
    """
    _name = 'account.sale.closing'
    _order = 'date_closing_stop desc, sequence_number desc'
    _description = "Sale Closing"

    name = fields.Char(help="Frequency and unique sequence number", required=True)
    company_id = fields.Many2one('res.company', string='Company', readonly=True, required=True)
    date_closing_stop = fields.Datetime(string="Closing Date", help='Date to which the values are computed', readonly=True, required=True)
    date_closing_start = fields.Datetime(string="Starting Date", help='Date from which the total interval is computed', readonly=True, required=True)
    frequency = fields.Selection(string='Closing Type', selection=[('daily', 'Daily'), ('monthly', 'Monthly'), ('annually', 'Annual')], readonly=True, required=True)
    total_interval = fields.Monetary(string="Period Total", help='Total in receivable accounts during the interval, excluding overlapping periods', readonly=True, required=True)
    cumulative_total = fields.Monetary(string="Cumulative Grand Total", help='Total in receivable accounts since the beginnig of times', readonly=True, required=True)
    sequence_number = fields.Integer('Sequence #', readonly=True, required=True)
    last_order_id = fields.Many2one('pos.order', string='Last Pos Order', help='Last Pos order included in the grand total', readonly=True)
    last_order_hash = fields.Char(string='Last Order entry\'s inalteralbility hash', readonly=True)
    currency_id = fields.Many2one('res.currency', string='Currency', help="The company's currency", readonly=True, related='company_id.currency_id', store=True)

    def _query_for_aml(self, company, first_move_sequence_number, date_start):
        params = {'company_id': company.id}
        query = '''WITH aggregate AS (SELECT m.id AS move_id,
                    aml.balance AS balance,
                    aml.id as line_id
            FROM account_move_line aml
            JOIN account_journal j ON aml.journal_id = j.id
            JOIN account_account acc ON acc.id = aml.account_id
            JOIN account_account_type t ON (t.id = acc.user_type_id AND t.type = 'receivable')
            JOIN account_move m ON m.id = aml.move_id
            WHERE j.type = 'sale'
                AND aml.company_id = %(company_id)s
                AND m.state = 'posted' '''

        if first_move_sequence_number is not False and first_move_sequence_number is not None:
            params['first_move_sequence_number'] = first_move_sequence_number
            query += '''AND m.secure_sequence_number > %(first_move_sequence_number)s'''
        elif date_start:
            #the first time we compute the closing, we consider only from the installation of the module
            params['date_start'] = date_start
            query += '''AND m.date >= %(date_start)s'''

        query += " ORDER BY m.secure_sequence_number DESC) "
        query += '''SELECT array_agg(move_id) AS move_ids,
                           array_agg(line_id) AS line_ids,
                           sum(balance) AS balance
                    FROM aggregate'''

        self.env.cr.execute(query, params)
        return self.env.cr.dictfetchall()[0]

    def _compute_amounts(self, frequency, company):
        """
        Method used to compute all the business data of the new object.
        It will search for previous closings of the same frequency to infer the move from which
        account move lines should be fetched.
        @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)
            frequencies are literal (daily means 24 hours and so on)
        @param {recordset} company: the company for which the closing is done
        @return {dict} containing {field: value} for each business field of the object
        """
        interval_dates = self._interval_dates(frequency, company)
        previous_closing = self.search([
            ('frequency', '=', frequency),
            ('company_id', '=', company.id)], limit=1, order='sequence_number desc')

        first_order = self.env['pos.order']
        date_start = interval_dates['interval_from']
        cumulative_total = 0
        if previous_closing:
            first_order = previous_closing.last_order_id
            date_start = previous_closing.create_date
            cumulative_total += previous_closing.cumulative_total

        domain = [('company_id', '=', company.id), ('state', 'in', ('paid', 'done', 'invoiced'))]
        if first_order.l10n_fr_secure_sequence_number is not False and first_order.l10n_fr_secure_sequence_number is not None:
            domain = AND([domain, [('l10n_fr_secure_sequence_number', '>', first_order.l10n_fr_secure_sequence_number)]])
        elif date_start:
            #the first time we compute the closing, we consider only from the installation of the module
            domain = AND([domain, [('date_order', '>=', date_start)]])

        orders = self.env['pos.order'].search(domain, order='date_order desc')

        total_interval = sum(orders.mapped('amount_total'))
        cumulative_total += total_interval

        # We keep the reference to avoid gaps (like daily object during the weekend)
        last_order = first_order
        if orders:
            last_order = orders[0]

        return {'total_interval': total_interval,
                'cumulative_total': cumulative_total,
                'last_order_id': last_order.id,
                'last_order_hash': last_order.l10n_fr_secure_sequence_number,
                'date_closing_stop': interval_dates['date_stop'],
                'date_closing_start': date_start,
                'name': interval_dates['name_interval'] + ' - ' + interval_dates['date_stop'][:10]}

    def _interval_dates(self, frequency, company):
        """
        Method used to compute the theoretical date from which account move lines should be fetched
        @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)
            frequencies are literal (daily means 24 hours and so on)
        @param {recordset} company: the company for which the closing is done
        @return {dict} the theoretical date from which account move lines are fetched.
            date_stop date to which the move lines are fetched, always now()
            the dates are in their Odoo Database string representation
        """
        date_stop = datetime.utcnow()
        interval_from = None
        name_interval = ''
        if frequency == 'daily':
            interval_from = date_stop - timedelta(days=1)
            name_interval = _('Daily Closing')
        elif frequency == 'monthly':
            month_target = date_stop.month > 1 and date_stop.month - 1 or 12
            year_target = month_target < 12 and date_stop.year or date_stop.year - 1
            interval_from = date_stop.replace(year=year_target, month=month_target)
            name_interval = _('Monthly Closing')
        elif frequency == 'annually':
            year_target = date_stop.year - 1
            interval_from = date_stop.replace(year=year_target)
            name_interval = _('Annual Closing')

        return {'interval_from': FieldDateTime.to_string(interval_from),
                'date_stop': FieldDateTime.to_string(date_stop),
                'name_interval': name_interval}

    def write(self, vals):
        raise UserError(_('Sale Closings are not meant to be written or deleted under any circumstances.'))

    def unlink(self):
        raise UserError(_('Sale Closings are not meant to be written or deleted under any circumstances.'))

    @api.model
    def _automated_closing(self, frequency='daily'):
        """To be executed by the CRON to create an object of the given frequency for each company that needs it
        @param {string} frequency: a valid value of the selection field on the object (daily, monthly, annually)
            frequencies are literal (daily means 24 hours and so on)
        @return {recordset} all the objects created for the given frequency
        """
        res_company = self.env['res.company'].search([])
        account_closings = self.env['account.sale.closing']
        for company in res_company.filtered(lambda c: c._is_accounting_unalterable()):
            new_sequence_number = company.l10n_fr_closing_sequence_id.next_by_id()
            values = self._compute_amounts(frequency, company)
            values['frequency'] = frequency
            values['company_id'] = company.id
            values['sequence_number'] = new_sequence_number
            account_closings |= account_closings.create(values)

        return account_closings

```

## File: models\account_fiscal_position.py

```python
# -*- coding: utf-8 -*-

from odoo import _, models
from odoo.exceptions import UserError


class AccountFiscalPosition(models.Model):
    _inherit = "account.fiscal.position"

    def write(self, vals):
        if "tax_ids" in vals:
            if self.env["pos.order"].sudo().search_count([("fiscal_position_id", "in", self.ids)]):
                raise UserError(
                    _(
                        "You cannot modify a fiscal position used in a POS order. "
                        "You should archive it and create a new one."
                    )
                )
        return super(AccountFiscalPosition, self).write(vals)

```

## File: models\pos.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import datetime, timedelta
from hashlib import sha256
from json import dumps

from odoo import models, api, fields
from odoo.fields import Datetime
from odoo.tools.translate import _, _lt
from odoo.exceptions import UserError


class pos_config(models.Model):
    _inherit = 'pos.config'

    def open_ui(self):
        for config in self:
            if not config.company_id.country_id:
                raise UserError(_("You have to set a country in your company setting."))
            if config.company_id._is_accounting_unalterable():
                if config.current_session_id:
                    config.current_session_id._check_session_timing()
        return super(pos_config, self).open_ui()


class pos_session(models.Model):
    _inherit = 'pos.session'

    def _check_session_timing(self):
        self.ensure_one()
        date_today = datetime.utcnow()
        session_start = Datetime.from_string(self.start_at)
        if not date_today - timedelta(hours=24) <= session_start:
            raise UserError(_("This session has been opened another day. To comply with the French law, you should close sessions on a daily basis. Please close session %s and open a new one.", self.name))
        return True

    def open_frontend_cb(self):
        sessions_to_check = self.filtered(lambda s: s.config_id.company_id._is_accounting_unalterable())
        sessions_to_check.filtered(lambda s: s.state == 'opening_control').start_at = fields.Datetime.now()
        for session in sessions_to_check:
            session._check_session_timing()
        return super(pos_session, self).open_frontend_cb()


ORDER_FIELDS = ['date_order', 'user_id', 'lines', 'payment_ids', 'pricelist_id', 'partner_id', 'session_id', 'pos_reference', 'sale_journal', 'fiscal_position_id']
LINE_FIELDS = ['notice', 'product_id', 'qty', 'price_unit', 'discount', 'tax_ids', 'tax_ids_after_fiscal_position']
ERR_MSG = _lt('According to the French law, you cannot modify a %s. Forbidden fields: %s.')


class pos_order(models.Model):
    _inherit = 'pos.order'

    l10n_fr_hash = fields.Char(string="Inalteralbility Hash", readonly=True, copy=False)
    l10n_fr_secure_sequence_number = fields.Integer(string="Inalteralbility No Gap Sequence #", readonly=True, copy=False)
    l10n_fr_string_to_hash = fields.Char(compute='_compute_string_to_hash', readonly=True, store=False)

    def _get_new_hash(self, secure_seq_number):
        """ Returns the hash to write on pos orders when they get posted"""
        self.ensure_one()
        #get the only one exact previous order in the securisation sequence
        prev_order = self.search([('state', 'in', ['paid', 'done', 'invoiced']),
                                 ('company_id', '=', self.company_id.id),
                                 ('l10n_fr_secure_sequence_number', '!=', 0),
                                 ('l10n_fr_secure_sequence_number', '=', int(secure_seq_number) - 1)])
        if prev_order and len(prev_order) != 1:
            raise UserError(
               _('An error occured when computing the inalterability. Impossible to get the unique previous posted point of sale order.'))

        #build and return the hash
        return self._compute_hash(prev_order.l10n_fr_hash if prev_order else u'')

    def _compute_hash(self, previous_hash):
        """ Computes the hash of the browse_record given as self, based on the hash
        of the previous record in the company's securisation sequence given as parameter"""
        self.ensure_one()
        hash_string = sha256((previous_hash + self.l10n_fr_string_to_hash).encode('utf-8'))
        return hash_string.hexdigest()

    def _compute_string_to_hash(self):
        def _getattrstring(obj, field_str):
            field_value = obj[field_str]
            if obj._fields[field_str].type == 'many2one':
                field_value = field_value.id
            if obj._fields[field_str].type in ['many2many', 'one2many']:
                field_value = field_value.sorted().ids
            return str(field_value)

        for order in self:
            values = {}
            for field in ORDER_FIELDS:
                values[field] = _getattrstring(order, field)

            for line in order.lines:
                for field in LINE_FIELDS:
                    k = 'line_%d_%s' % (line.id, field)
                    values[k] = _getattrstring(line, field)
            #make the json serialization canonical
            #  (https://tools.ietf.org/html/draft-staykov-hu-json-canonical-form-00)
            order.l10n_fr_string_to_hash = dumps(values, sort_keys=True,
                                                ensure_ascii=True, indent=None,
                                                separators=(',',':'))

    def write(self, vals):
        has_been_posted = False
        for order in self:
            if order.company_id._is_accounting_unalterable():
                # write the hash and the secure_sequence_number when posting or invoicing an pos.order
                if vals.get('state') in ['paid', 'done', 'invoiced']:
                    has_been_posted = True

                # restrict the operation in case we are trying to write a forbidden field
                if (order.state in ['paid', 'done', 'invoiced'] and set(vals).intersection(ORDER_FIELDS)):
                    raise UserError(_('According to the French law, you cannot modify a point of sale order. Forbidden fields: %s.') % ', '.join(ORDER_FIELDS))
                # restrict the operation in case we are trying to overwrite existing hash
                if (order.l10n_fr_hash and 'l10n_fr_hash' in vals) or (order.l10n_fr_secure_sequence_number and 'l10n_fr_secure_sequence_number' in vals):
                    raise UserError(_('You cannot overwrite the values ensuring the inalterability of the point of sale.'))
        res = super(pos_order, self).write(vals)
        # write the hash and the secure_sequence_number when posting or invoicing a pos order
        if has_been_posted:
            for order in self.filtered(lambda o: o.company_id._is_accounting_unalterable() and
                                                not (o.l10n_fr_secure_sequence_number or o.l10n_fr_hash)):
                new_number = order.company_id.l10n_fr_pos_cert_sequence_id.next_by_id()
                vals_hashing = {'l10n_fr_secure_sequence_number': new_number,
                                'l10n_fr_hash': order._get_new_hash(new_number)}
                res |= super(pos_order, order).write(vals_hashing)
        return res

    def unlink(self):
        for order in self:
            if order.company_id._is_accounting_unalterable():
                raise UserError(_("According to French law, you cannot delet a point of sale order."))
        return super(pos_order, self).unlink()
    
    def _export_for_ui(self, order):
        res = super()._export_for_ui(order)
        res['l10n_fr_hash'] = order.l10n_fr_hash
        return res


class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    def write(self, vals):
        # restrict the operation in case we are trying to write a forbidden field
        if set(vals).intersection(LINE_FIELDS):
            if any(l.company_id._is_accounting_unalterable() and l.order_id.state in ['done', 'invoiced'] for l in self):
                raise UserError(_('According to the French law, you cannot modify a point of sale order line. Forbidden fields: %s.') % ', '.join(LINE_FIELDS))
        return super(PosOrderLine, self).write(vals)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api, fields, _
from odoo.exceptions import UserError
from datetime import datetime
from odoo.fields import Datetime, Date
from odoo.tools.misc import format_date
import pytz


def ctx_tz(record, field):
    res_lang = None
    ctx = record._context
    tz_name = pytz.timezone(ctx.get('tz') or record.env.user.tz)
    timestamp = Datetime.from_string(record[field])
    if ctx.get('lang'):
        res_lang = record.env['res.lang']._lang_get(ctx['lang'])
    if res_lang:
        timestamp = pytz.utc.localize(timestamp, is_dst=False)
        return datetime.strftime(timestamp.astimezone(tz_name), res_lang.date_format + ' ' + res_lang.time_format)
    return Datetime.context_timestamp(record, timestamp)


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_pos_cert_sequence_id = fields.Many2one('ir.sequence')

    @api.model
    def create(self, vals):
        company = super(ResCompany, self).create(vals)
        #when creating a new french company, create the securisation sequence as well
        if company._is_accounting_unalterable():
            sequence_fields = ['l10n_fr_pos_cert_sequence_id']
            company._create_secure_sequence(sequence_fields)
        return company

    def write(self, vals):
        res = super(ResCompany, self).write(vals)
        #if country changed to fr, create the securisation sequence
        for company in self:
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_pos_cert_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return res

    def _action_check_pos_hash_integrity(self):
        return self.env.ref('l10n_fr_pos_cert.action_report_pos_hash_integrity').report_action(self.id)

    def _check_pos_hash_integrity(self):
        """Checks that all posted or invoiced pos orders have still the same data as when they were posted
        and raises an error with the result.
        """
        def build_order_info(order):
            entry_reference = _('(Receipt ref.: %s)')
            order_reference_string = order.pos_reference and entry_reference % order.pos_reference or ''
            return [ctx_tz(order, 'date_order'), order.l10n_fr_hash, order.name, order_reference_string, ctx_tz(order, 'write_date')]

        msg_alert = ''
        report_dict = {}
        if self._is_accounting_unalterable():
            orders = self.env['pos.order'].search([('state', 'in', ['paid', 'done', 'invoiced']), ('company_id', '=', self.id),
                                    ('l10n_fr_secure_sequence_number', '!=', 0)], order="l10n_fr_secure_sequence_number ASC")

            if not orders:
                msg_alert = (_('There isn\'t any order flagged for data inalterability yet for the company %s. This mechanism only runs for point of sale orders generated after the installation of the module France - Certification CGI 286 I-3 bis. - POS', self.env.company.name))
                raise UserError(msg_alert)

            previous_hash = u''
            corrupted_orders = []
            for order in orders:
                if order.l10n_fr_hash != order._compute_hash(previous_hash=previous_hash):
                    corrupted_orders.append(order.name)
                    msg_alert = (_('Corrupted data on point of sale order with id %s.', order.id))
                previous_hash = order.l10n_fr_hash

            orders_sorted_date = orders.sorted(lambda o: o.date_order)
            start_order_info = build_order_info(orders_sorted_date[0])
            end_order_info = build_order_info(orders_sorted_date[-1])

            report_dict.update({
                'first_order_name': start_order_info[2],
                'first_order_hash': start_order_info[1],
                'first_order_date': start_order_info[0],
                'last_order_name': end_order_info[2],
                'last_order_hash': end_order_info[1],
                'last_order_date': end_order_info[0],
            })
            corrupted_orders = ', '.join([o for o in corrupted_orders])
            return {
                'result': report_dict or 'None',
                'msg_alert': msg_alert or 'None',
                'printing_date': format_date(self.env,  Date.to_string( Date.today())),
                'corrupted_orders': corrupted_orders or 'None'
            }
        else:
            raise UserError(_('Accounting is not unalterable for the company %s. This mechanism is designed for companies where accounting is unalterable.') % self.env.company.name)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_bank_statement
from . import account_fiscal_position
from . import res_company
from . import pos
from . import account_closing

```

## File: report\pos_hash_integrity.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ReportPosHashIntegrity(models.AbstractModel):
    _name = 'report.l10n_fr_pos_cert.report_pos_hash_integrity'
    _description = 'Get french pos hash integrity result as PDF.'

    @api.model
    def _get_report_values(self, docids, data=None):
        data = data or {}
        data.update(self.env.company._check_pos_hash_integrity() or {})
        return {
            'doc_ids' : docids,
            'doc_model' : self.env['res.company'],
            'data' : data,
            'docs' : self.env['res.company'].browse(self.env.company.id),
        }

```

## File: report\pos_hash_integrity.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <data>
        <template id="report_pos_hash_integrity">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="company">
                    <t t-call="web.external_layout">
                        <div class="page">
                            <div class="row" id="hash_header">
                                <div class="col-12">
                                    <br/>
                                    <h2>Résultat du test d'intégrité - <span t-esc="data['printing_date']"/></h2>
                                    <br/>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-12" id="hash_config_review">
                                    <h6>
                                        Selon l’article 286 du code général des impôts français, toute livraison de bien ou prestation
                                        de services ne donnant pas lieu à facturation et étant enregistrée au moyen d’un logiciel ou
                                        d’un système de caisse doit satisfaire à des conditions d’inaltérabilité et de sécurisation des
                                        données en vue d’un contrôle de l’administration fiscale.
                                        <br/>
                                        <br/>
                                        Ces conditions sont respectées via une fonction de hachage des ventes du Point de Vente.
                                        <br/>
                                        <br/>
                                    </h6>
                                </div>
                            </div>
                            <t t-if="data['result'] != 'None'">
                                <div class="row">
                                    <div class="col-12" id="hash_data_consistency">
                                        <br/>
                                        <h3>Contrôle des données du point de vente</h3>
                                        <br/>
                                        <t t-if="data['result'] != 'None' and data['corrupted_orders'] == 'None'">
                                            <h5>
                                                Toutes les ventes effectuées via le Point de Vente
                                                sont bien dans la chaîne de hachage.
                                            </h5>
                                            <br/>
                                        </t>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-12" id="hash_data_consistency_table">
                                        <table class="table table-bordered" style="table-layout: fixed">
                                            <thead style="display: table-row-group">
                                                <tr>
                                                    <th class="text-center" style="width: 25%" scope="col">First Hash</th>
                                                    <th class="text-center" style="width: 25%" scope="col">First Entry</th>
                                                    <th class="text-center" style="width: 25%" scope="col">Last Hash</th>
                                                    <th class="text-center" style="width: 25%" scope="col">Last Entry</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                <t t-if="data['result'] != 'None'">
                                                    <t t-if="data['result']['first_order_hash'] != 'None'">
                                                        <tr>
                                                            <td><span t-esc="data['result']['first_order_hash']"/></td>
                                                            <td>
                                                                <span t-esc="data['result']['first_order_name']"/> <br/>
                                                                <span t-esc="data['result']['first_order_date']"/>
                                                            </td>
                                                            <td><span t-esc="data['result']['last_order_hash']"/></td>
                                                            <td>
                                                                <span t-esc="data['result']['last_order_name']"/> <br/>
                                                                <span t-esc="data['result']['last_order_date']"/>
                                                            </td>
                                                        </tr>
                                                    </t>
                                                </t>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                                <div>
                                    <t t-if="data['corrupted_orders'] != 'None'">
                                        <h5>
                                            Données corrompues sur la commande du point de vente:
                                        </h5>
                                        <span t-esc="data['corrupted_orders']"/> <br/>
                                    </t>
                                </div>
                                <div class="row" id="hash_last_div">
                                    <div class="col-12" id="hash_chain_compliant">
                                        <br/>
                                        <h6>
                                            La chaîne de hachage est conforme: il n’est pas possible d’altérer les données
                                            sans casser la chaîne de hachage pour les pièces ultérieures.
                                        </h6>
                                        <br/>
                                    </div>
                                </div>
                            </t>
                        </div>
                    </t>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_hash_integrity

```

## File: security\account_closing_intercompany.xml

```xml
<odoo noupdate="1">

    <record model="ir.rule" id="account_sale_closing_multi_company">
      <field name="name">Sale Closing multi-company</field>
      <field name="model_id" ref="model_account_sale_closing"/>
      <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_fr_pos_cert_account_sale_closing_user,l10n_fr_pos_cert.account.sale.closing.user,l10n_fr_pos_cert.model_account_sale_closing,base.group_user,1,0,0,0

```

## File: static\src\js\NumpadWidget.js

```javascript
odoo.define('l10n_fr_pos_cert.NumpadWidget', function(require) {
    'use strict';

    const NumpadWidget = require('point_of_sale.NumpadWidget');
    const Registries = require('point_of_sale.Registries');

    const PosFrNumpadWidget = NumpadWidget => class extends NumpadWidget {
        get hasPriceControlRights() {
            if (this.env.pos.is_french_country()) {
                return false;
            } else {
                return super.hasPriceControlRights;
            }
        }
    };

    Registries.Component.extend(NumpadWidget, PosFrNumpadWidget);

    return NumpadWidget;
 });

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('l10n_fr_pos_cert.PaymentScreen', function(require) {

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');
    const session = require('web.session');

    const PosFrPaymentScreen = PaymentScreen => class extends PaymentScreen {
        async _postPushOrderResolve(order, order_server_ids) {
            try {
                if(this.env.pos.is_french_country()) {
                    let result = await this.rpc({
                        model: 'pos.order',
                        method: 'search_read',
                        domain: [['id', 'in', order_server_ids]],
                        fields: ['l10n_fr_hash'],
                        context: session.user_context,
                    });
                    order.set_l10n_fr_hash(result[0].l10n_fr_hash || false);
                }
            } finally {
                return super._postPushOrderResolve(...arguments);
            }
        }
    };

    Registries.Component.extend(PaymentScreen, PosFrPaymentScreen);

    return PaymentScreen;
});

```

## File: static\src\js\pos.js

```javascript
odoo.define('l10n_fr_pos_cert.pos', function (require) {
"use strict";

const { Gui } = require('point_of_sale.Gui');
var models = require('point_of_sale.models');
var rpc = require('web.rpc');
var session = require('web.session');
var core = require('web.core');
var utils = require('web.utils');

var _t = core._t;
var round_di = utils.round_decimals;

var _super_posmodel = models.PosModel.prototype;
models.PosModel = models.PosModel.extend({
    is_french_country: function(){
      var french_countries = ['FR', 'MF', 'MQ', 'NC', 'PF', 'RE', 'GF', 'GP', 'TF'];
      if (!this.company.country) {
        Gui.showPopup("ErrorPopup", {
            'title': _t("Missing Country"),
            'body':  _.str.sprintf(_t('The company %s doesn\'t have a country set.'), this.company.name),
        });
        return false;
      }
      return _.contains(french_countries, this.company.country.code);
    },
    delete_current_order: function () {
        if (this.is_french_country() && this.get_order().get_orderlines().length) {
            Gui.showPopup("ErrorPopup", {
                'title': _t("Fiscal Data Module error"),
                'body':  _t("Deleting of orders is not allowed."),
            });
        } else {
            _super_posmodel.delete_current_order.apply(this, arguments);
        }
    },

    disallowLineQuantityChange() {
        let result = _super_posmodel.disallowLineQuantityChange.bind(this)();
        return this.is_french_country() || result;
    }
});


var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    initialize: function() {
        _super_order.initialize.apply(this,arguments);
        this.l10n_fr_hash = this.l10n_fr_hash || false;
        this.save_to_db();
    },
    export_for_printing: function() {
      var result = _super_order.export_for_printing.apply(this,arguments);
      result.l10n_fr_hash = this.get_l10n_fr_hash();
      return result;
    },
    set_l10n_fr_hash: function (l10n_fr_hash){
      this.l10n_fr_hash = l10n_fr_hash;
    },
    get_l10n_fr_hash: function() {
      return this.l10n_fr_hash;
    },
    wait_for_push_order: function() {
      var result = _super_order.wait_for_push_order.apply(this,arguments);
      result = Boolean(result || this.pos.is_french_country());
      return result;
    }
});

var orderline_super = models.Orderline.prototype;
models.Orderline = models.Orderline.extend({
    can_be_merged_with: function(orderline) {
        if (this.pos.is_french_country()) {
            const order = this.pos.get_order();
            const lastId = order.orderlines.last().cid;
            if ((order.orderlines._byId[lastId].product.id !== orderline.product.id || order.orderlines._byId[lastId].quantity < 0)) {
                return false;
            }
            return orderline_super.can_be_merged_with.apply(this, arguments);
        }
        return orderline_super.can_be_merged_with.apply(this, arguments);
    }
});

});

```

## File: static\src\xml\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('pos-receipt-order-data')]" position="inside">
            <t t-if="receipt.l10n_fr_hash !== false">
                <br/>
                <div style="word-wrap:break-word;"><t t-esc="receipt.l10n_fr_hash"/></div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: views\account_sale_closure.xml

```xml
<odoo>
    <record id="list_view_account_sale_closing" model="ir.ui.view">
        <field name="name">Sales Closings</field>
        <field name="model">account.sale.closing</field>
        <field name="arch" type="xml">
            <tree create="false" import="false">
                <field name="date_closing_start"/>
                <field name="date_closing_stop"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="currency_id" invisible="1"/>
                <field name="frequency"/>
                <field name="sequence_number" groups="base.group_no_one"/>
                <field name="total_interval"/>
                <field name="cumulative_total"/>
            </tree>
        </field>
    </record>

    <record id="form_view_account_sale_closing" model="ir.ui.view">
        <field name="name">Sales Closings</field>
        <field name="model">account.sale.closing</field>
        <field name="arch" type="xml">
            <form create="false" edit="false" string="Account Closing">
                <sheet>
                    <div class="oe_title">
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="date_closing_start"/>
                            <field name="date_closing_stop"/>
                            <field name="frequency"/>
                            <field name="sequence_number" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="total_interval"/>
                            <field name="cumulative_total"/>
                            <field name="last_order_id" groups="account.group_account_readonly"/>
                            <field name="last_order_hash" groups="account.group_account_readonly"/>
                        </group>
                        <group>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="currency_id" invisible="1"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="action_list_view_account_sale_closing" model="ir.actions.act_window">
        <field name="name">Sales Closings</field>
        <field name="res_model">account.sale.closing</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_nocreate">
                The closings are created by Odoo
                </p><p>
                Sales closings run automatically on a daily, monthly and annual basis. It computes both period and cumulative totals from all the sales entries posted in the system after the previous closing.
            </p>
        </field>
    </record>

    <menuitem action="action_list_view_account_sale_closing" id="menu_account_closing_reporting" parent="l10n_fr.account_reports_fr_statements_menu" sequence="90"/>
</odoo>

```

## File: views\account_views.xml

```xml
<odoo>
    <record id="view_bank_statement_form_readonly" model="ir.ui.view">
        <field name="name">account.bank.statement.form</field>
        <field name="model">account.bank.statement</field>
        <field name="inherit_id" ref="account.view_bank_statement_form"/>
        <field name="mode">extension</field>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <field name="pos_session_id" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='name']" position="attributes">
                <attribute name="attrs">{'readonly': [('pos_session_id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//field[@name='journal_id']" position="attributes">
                <attribute name="attrs">{'readonly': ['|', ('move_line_count','!=', 0), ('pos_session_id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//field[@name='date']" position="attributes">
                <attribute name="attrs">{'readonly': ['|', ('state', '!=', 'open'), ('pos_session_id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//field[@name='line_ids']" position="attributes">
                <attribute name="attrs">{'readonly': ['|', ('state', '!=', 'open'), ('pos_session_id', '!=', False)]}</attribute>
            </xpath>

        </field>
    </record>
</odoo>

```

## File: views\l10n_fr_pos_cert_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/l10n_fr_pos_cert/static/src/js/pos.js"></script>
            <script type="text/javascript" src="/l10n_fr_pos_cert/static/src/js/NumpadWidget.js"></script>
            <script type="text/javascript" src="/l10n_fr_pos_cert/static/src/js/PaymentScreen.js"></script>
        </xpath>
    </template>

</odoo>

```

## File: views\pos_inalterability_menuitem.xml

```xml
<odoo>
    <record id="action_report_pos_hash_integrity" model="ir.actions.report">
        <field name="name">Hash integrity result PDF</field>
        <field name="model">res.company</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">l10n_fr_pos_cert.report_pos_hash_integrity</field>
        <field name="report_file">l10n_fr_pos_cert.report_pos_hash_integrity</field>
    </record>
    <record model="ir.actions.server" id="action_check_pos_hash_integrity">
        <field name="name">POS Inalterability Check</field>
        <field name="model_id" ref="account.model_res_company"/>
        <field name="type">ir.actions.server</field>
        <field name="state">code</field>
        <field name="code">
            action = env.company._action_check_pos_hash_integrity()
        </field>
    </record>
        <menuitem id="pos_fr_statements_menu" name="French Statements" parent="point_of_sale.menu_point_rep" sequence="9" />
        <menuitem action="l10n_fr_pos_cert.action_list_view_account_sale_closing" id="menu_account_closing" parent="pos_fr_statements_menu" sequence="80"/>
        <menuitem action="l10n_fr_pos_cert.action_check_pos_hash_integrity" id="menu_check_move_integrity_reporting" parent="pos_fr_statements_menu" sequence="90"/>
</odoo>

```

## File: views\pos_views.xml

```xml
<odoo>
    <record id="pos_order_form_inherit" model="ir.ui.view">
        <field name="name">pos.order.form.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='pos_reference']" position='after'>
                <field string='Hash' name="l10n_fr_hash" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>
</odoo>

```

