# Odoo Module: l10n_it_edi_doi

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def _l10n_it_edi_doi_post_init(env):
    for company in env['res.company'].search([('chart_template', '=', 'it'), ('parent_id', '=', False)]):
        template = env['account.chart.template'].with_company(company)
        template._load_data({
            'account.tax': template._get_it_edi_doi_account_tax(),
            'account.fiscal.position': template._get_it_edi_doi_account_fiscal_position(),
            'res.company': template._get_it_edi_doi_res_company(),
        })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Italy - Declaration of Intent',
    'countries': ['it'],
    'version': '0.1',
    'depends': [
        'l10n_it_edi',
        'sale',
    ],
    'description': """
    Add support for the Declaration of Intent (Dichiarazione di Intento) to the Italian localization.
    """,
    'category': 'Accounting/Localizations',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/italy.html',
    'data': [
        'security/ir.model.access.csv',
        'data/invoice_it_template.xml',
        'views/l10n_it_edi_doi_declaration_of_intent_views.xml',
        'views/account_move_views.xml',
        'views/report_invoice.xml',
        'views/res_partner_views.xml',
        'views/sale_ir_actions_report_templates.xml',
        'views/sale_order_views.xml',
    ],
    'license': 'LGPL-3',
    'post_init_hook': '_l10n_it_edi_doi_post_init',
}

```

## File: data\invoice_it_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <template id="account_invoice_line_it_FatturaPA" inherit_id="l10n_it_edi.account_invoice_line_it_FatturaPA">
            <DettaglioLinee position="inside">
                <AltriDatiGestionali t-if="record.l10n_it_edi_doi_id">
                    <TipoDato t-translation="off">INTENTO</TipoDato>
                    <RiferimentoTesto t-out="record.l10n_it_edi_doi_id.display_name"/>
                    <RiferimentoData t-out="format_date(record.l10n_it_edi_doi_id.issue_date)"/>
                </AltriDatiGestionali>
            </DettaglioLinee>
        </template>

    </data>
</odoo>

```

## File: data\template\account.fiscal.position-it.csv

```csv
"id","name",sequence,"auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@it","note","note@it"
"declaration_of_intent_fiscal_position","Declaration of Intent","10","0","0",,,"22v","00di","Dichiarazione di Intento","Not taxable (art. 8, c. 1, lett. c) D.P.R. 633/1972) letter intent","Non imponibile (art. 8, c. 1, lett. c) D.P.R. 633/1972) Lettera intento"
"","","","","","","","10v","00di","","",""
"","","","","","","","5v","00di","","",""
"","","","","","","","4v","00di","","",""

```

## File: data\template\account.tax-it.csv

```csv
"id","description","invoice_label","name","sequence","amount","amount_type","type_tax_use","price_include","tax_group_id","active","tax_scope","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@it","children_tax_ids","l10n_it_exempt_reason","l10n_it_law_reference","description@it"
"00di","Declaration of Intent","0%","0% E","950","0.0","percent","sale","False","tax_group_fuori","","","base","invoice","+02","","","","","N3.5","art. 8, c. 1, lett. c) D.P.R. 633/1972",""
"","","","","","","","","","","","","tax","invoice","","","","","","","",""
"","","","","","","","","","","","","base","refund","-02","","","","","","",""
"","","","","","","","","","","","","tax","refund","","","","","","","",""

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('it', 'account.tax')
    def _get_it_edi_doi_account_tax(self):
        tax_data = self._parse_csv('it', 'account.tax', module='l10n_it_edi_doi')
        self._deref_account_tags('it', tax_data)
        return tax_data

    @template('it', 'account.fiscal.position')
    def _get_it_edi_doi_account_fiscal_position(self):
        return self._parse_csv('it', 'account.fiscal.position', module='l10n_it_edi_doi')

    @template('it', 'res.company')
    def _get_it_edi_doi_res_company(self):
        return {
            self.env.company.id: {
                'l10n_it_edi_doi_tax_id': '00di',
                'l10n_it_edi_doi_fiscal_position_id': 'declaration_of_intent_fiscal_position',
            },
        }

```

## File: models\account_fiscal_position.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import UserError


class AccountFiscalPosition(models.Model):
    _inherit = 'account.fiscal.position'

    @api.ondelete(at_uninstall=False)
    def _never_unlink_declaration_of_intent_fiscal_position(self):
        for fiscal_position in self:
            if fiscal_position == fiscal_position.company_id.l10n_it_edi_doi_fiscal_position_id:
                raise UserError(_('You cannot delete the special fiscal position for Declarations of Intent.'))

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_edi_doi_date = fields.Date(
        string="Date on which Declaration of Intent is applied",
        compute='_compute_l10n_it_edi_doi_date',
    )

    l10n_it_edi_doi_use = fields.Boolean(
        string="Use Declaration of Intent",
        compute='_compute_l10n_it_edi_doi_use',
    )

    l10n_it_edi_doi_id = fields.Many2one(
        string="Declaration of Intent",
        compute='_compute_l10n_it_edi_doi_id',
        store=True,
        readonly=False,
        precompute=True,
        comodel_name='l10n_it_edi_doi.declaration_of_intent',
    )

    l10n_it_edi_doi_amount = fields.Monetary(
        string='Declaration of Intent Amount',
        compute='_compute_l10n_it_edi_doi_amount',
        store=True,
        readonly=True,
        help="Total amount of sales under the Declaration of Intent of this document",
    )

    l10n_it_edi_doi_warning = fields.Text(
        string="Declaration of Intent Threshold Warning",
        compute='_compute_l10n_it_edi_doi_warning',
    )

    @api.depends('invoice_date')
    def _compute_l10n_it_edi_doi_date(self):
        for move in self:
            move.l10n_it_edi_doi_date = move.invoice_date or fields.Date.context_today(self)

    @api.depends('l10n_it_edi_doi_id', 'country_code', 'move_type')
    def _compute_l10n_it_edi_doi_use(self):
        sale_types = self.env['account.move'].get_sale_types()
        for move in self:
            move.l10n_it_edi_doi_use = (
                move.l10n_it_edi_doi_id
                or (move.country_code == "IT" and move.move_type in sale_types)
            )

    @api.depends('company_id', 'partner_id.commercial_partner_id', 'l10n_it_edi_doi_date', 'currency_id')
    def _compute_l10n_it_edi_doi_id(self):
        for move in self:
            if not move.l10n_it_edi_doi_use or move.state != 'draft' and not move.l10n_it_edi_doi_id:
                move.l10n_it_edi_doi_id = False
                continue
            partner = move.partner_id.commercial_partner_id

            # Avoid a query or changing a manually set declaration of intent
            # (if the declaration is still valid).
            validity_warnings = move.l10n_it_edi_doi_id._get_validity_warnings(
                move.company_id, partner, move.currency_id, move.l10n_it_edi_doi_date
            )
            if move.l10n_it_edi_doi_id and not validity_warnings:
                continue

            declaration = self.env['l10n_it_edi_doi.declaration_of_intent']\
                ._fetch_valid_declaration_of_intent(move.company_id, partner, move.currency_id, move.l10n_it_edi_doi_date)
            move.l10n_it_edi_doi_id = declaration

    @api.depends('l10n_it_edi_doi_id', 'tax_totals', 'move_type')
    def _compute_l10n_it_edi_doi_amount(self):
        """
        Consider all the lines in self that belong to declaration of intent `declaration`
        and have the special declaration of intent tax applied.
        This function computes the signed sum of the price_total of all those lines
        (the tax amount of the lines is always 0).
        The direction_sign determines the sign: 1 (-1) for inbound (outbound) types.
        """
        for move in self:
            tax = move.company_id.l10n_it_edi_doi_tax_id
            if not tax or not move.l10n_it_edi_doi_id:
                move.l10n_it_edi_doi_amount = 0
                continue
            declaration_lines = move.invoice_line_ids.filtered(
                # The declaration tax cannot be used with other taxes on a single line
                # (checked in `_post`)
                lambda line: line.tax_ids.ids == tax.ids
            )
            move.l10n_it_edi_doi_amount = sum(declaration_lines.mapped('price_total')) * -move.direction_sign

    @api.depends('l10n_it_edi_doi_id', 'l10n_it_edi_doi_amount', 'state')
    def _compute_l10n_it_edi_doi_warning(self):
        for move in self:
            move.l10n_it_edi_doi_warning = ''
            declaration = move.l10n_it_edi_doi_id

            show_warning = (
                declaration
                and move.is_sale_document(include_receipts=False)
                and move.state != 'cancel'
            )
            if not show_warning:
                continue

            declaration_invoiced = declaration.invoiced
            declaration_not_yet_invoiced = declaration.not_yet_invoiced
            if move.state != 'posted':  # exactly the 'posted' invoices are included in declaration.invoiced
                # Here we replicate what would happen when posting the invoice.
                # Note: lines manually added to a move linked to a sales order are not added to the sales order
                declaration_invoiced += move.l10n_it_edi_doi_amount
                additional_invoiced_qty = {}
                linked_orders = self.env['sale.order']
                for invoice_line in move.invoice_line_ids:
                    for sale_line in invoice_line.sale_line_ids:
                        order = sale_line.order_id
                        if order.l10n_it_edi_doi_id == declaration:
                            linked_orders |= order
                        qty_invoiced = invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, sale_line.product_uom) * -move.direction_sign
                        sale_line_id = sale_line.ids[0]  # do not just use `id` in case of NewId
                        additional_invoiced_qty[sale_line_id] = additional_invoiced_qty.get(sale_line_id, 0) + qty_invoiced
                for order in linked_orders:
                    not_yet_invoiced = order.l10n_it_edi_doi_not_yet_invoiced
                    not_yet_invoiced_after_posting = order._l10n_it_edi_doi_get_amount_not_yet_invoiced(
                        declaration,
                        additional_invoiced_qty=additional_invoiced_qty,
                    )
                    declaration_not_yet_invoiced -= not_yet_invoiced - not_yet_invoiced_after_posting

            validity_warnings = declaration._get_validity_warnings(
                move.company_id, move.commercial_partner_id, move.currency_id, move.l10n_it_edi_doi_date,
                invoiced_amount=declaration_invoiced,
            )

            threshold_warning = declaration._build_threshold_warning_message(declaration_invoiced, declaration_not_yet_invoiced)

            move.l10n_it_edi_doi_warning = '{}\n\n{}'.format('\n'.join(validity_warnings), threshold_warning).strip()

    @api.depends('l10n_it_edi_doi_id')
    def _compute_fiscal_position_id(self):
        super()._compute_fiscal_position_id()
        for move in self:
            declaration_fiscal_position = move.company_id.l10n_it_edi_doi_fiscal_position_id
            if declaration_fiscal_position and move.l10n_it_edi_doi_id:
                move.fiscal_position_id = declaration_fiscal_position

    def copy_data(self, default=None):
        data_list = super().copy_data(default)
        for move, data in zip(self, data_list):
            date = fields.Date.context_today(self)
            validity_warnings = move.l10n_it_edi_doi_id._get_validity_warnings(
                move.company_id, move.commercial_partner_id, move.currency_id, date,
                only_blocking=True,
            )
            if validity_warnings:
                del data['l10n_it_edi_doi_id']
                del data['fiscal_position_id']
        return data_list

    @api.constrains('l10n_it_edi_doi_id')
    def _check_l10n_it_edi_doi_id(self):
        for move in self:
            validity_errors = move.l10n_it_edi_doi_id._get_validity_errors(
                move.company_id, move.partner_id.commercial_partner_id, move.currency_id
            )
            if validity_errors:
                raise UserError('\n'.join(validity_errors))

    def _post(self, soft=True):
        errors = []
        for move in self:
            declaration = move.l10n_it_edi_doi_id
            if declaration:
                validity_warnings = declaration._get_validity_warnings(
                    move.company_id, move.commercial_partner_id, move.currency_id, move.l10n_it_edi_doi_date,
                    invoiced_amount=move.l10n_it_edi_doi_amount,
                    only_blocking=True
                )
                errors.extend(validity_warnings)

            declaration_of_intent_tax = move.company_id.l10n_it_edi_doi_tax_id
            if not declaration_of_intent_tax:
                continue

            declaration_lines = move.invoice_line_ids.filtered(
                lambda line: declaration_of_intent_tax in line.tax_ids
            )
            if declaration_lines and not declaration:
                errors.append(_('Given the tax %s is applied, there should be a Declaration of Intent selected.',
                                declaration_of_intent_tax.name))
            if any(line.tax_ids != declaration_of_intent_tax for line in declaration_lines):
                errors.append(_('A line using tax %s should not contain any other taxes',
                                declaration_of_intent_tax.name))
        if errors:
            raise UserError('\n'.join(errors))

        return super()._post(soft)

    def action_open_declaration_of_intent(self):
        self.ensure_one()
        return {
            'name': _("Declaration of Intent for %s", self.display_name),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'l10n_it_edi_doi.declaration_of_intent',
            'res_id': self.l10n_it_edi_doi_id.id,
        }

```

## File: models\account_tax.py

```python
from odoo import api, models, _
from odoo.exceptions import UserError


class AccountTax(models.Model):
    _inherit = 'account.tax'

    @api.ondelete(at_uninstall=False)
    def _never_unlink_declaration_of_intent_tax(self):
        for tax in self:
            if tax == tax.company_id.l10n_it_edi_doi_tax_id:
                raise UserError(_('You cannot delete the special tax for Declarations of Intent.'))

```

## File: models\declaration_of_intent.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.misc import formatLang


class L10nItDeclarationOfIntent(models.Model):
    _name = "l10n_it_edi_doi.declaration_of_intent"
    _inherit = ['mail.thread.main.attachment', 'mail.activity.mixin']
    _description = "Declaration of Intent"
    _order = 'protocol_number_part1, protocol_number_part2'

    state = fields.Selection([
         ('draft', 'Draft'),
         ('active', 'Active'),
         ('revoked', 'Revoked'),
         ('terminated', 'Terminated'),
        ],
        string="State",
        tracking=True,
        default='draft',
        required=True,
        readonly=True,
        help="The state of this Declaration of Intent. \n"
        "- 'Draft' means that the Declaration of Intent still needs to be confirmed before being usable. \n"
        "- 'Active' means that the Declaration of Intent is usable. \n"
        "- 'Terminated' designates that the Declaration of Intent has been marked as not to use anymore without invalidating usages of it. \n"
        "- 'Revoked' means the Declaration of Intent should not have been used. You will probably need to revert previous usages of it, if any.\n")

    company_id = fields.Many2one(
        comodel_name='res.company',
        string='Company',
        index=True,
        required=True,
        default=lambda self: self.env.company._accessible_branches()[:1],
    )

    partner_id = fields.Many2one(
        comodel_name='res.partner',
        string='Partner',
        index=True,
        required=True,
        domain="['|', ('is_company', '=', True), ('parent_id', '=', False)]",
    )

    currency_id = fields.Many2one(
        comodel_name='res.currency',
        string='Currency',
        default=lambda self: self.env.ref('base.EUR', raise_if_not_found=False).id,
        required=True,
        readonly=True,
    )

    issue_date = fields.Date(
        string='Date of Issue',
        required=True,
        copy=False,
        default=fields.Date.context_today,
        help="Date on which the Declaration of Intent was issued",
    )

    start_date = fields.Date(
        string='Start Date',
        required=True,
        copy=False,
        help="First date on which the Declaration of Intent is valid",
    )

    end_date = fields.Date(
        string='End Date',
        required=True,
        copy=False,
        help="Last date on which the Declaration of Intent is valid",
    )

    threshold = fields.Monetary(
        string='Threshold',
        required=True,
        help="Total amount of allowed sales without VAT under this Declaration of Intent",
    )

    invoiced = fields.Monetary(
        string='Invoiced',
        compute='_compute_invoiced',
        store=True,
        readonly=True,
        help="Total amount of sales under this Declaration of Intent",
    )

    not_yet_invoiced = fields.Monetary(
        string='Not Yet Invoiced',
        compute='_compute_not_yet_invoiced',
        store=True,
        readonly=True,
        help="Total amount of planned sales under this Declaration of Intent (i.e. current quotation and sales orders) that can still be invoiced",
    )

    remaining = fields.Monetary(
        string='Remaining',
        compute='_compute_remaining',
        store=True,
        readonly=True,
        help="Remaining amount after deduction of the Invoiced and Not Yet Invoiced amounts.",
    )

    protocol_number_part1 = fields.Char(
        string='Protocol 1',
        required=True,
        readonly=False,
        copy=False,
    )

    protocol_number_part2 = fields.Char(
        string='Protocol 2',
        required=True,
        readonly=False,
        copy=False,
    )

    invoice_ids = fields.One2many(
        'account.move',
        'l10n_it_edi_doi_id',
        string="Invoices / Refunds",
        copy=False,
        readonly=True,
    )

    sale_order_ids = fields.One2many(
        'sale.order',
        'l10n_it_edi_doi_id',
        string="Sales Orders / Quotations",
        copy=False,
        readonly=True,
    )

    _sql_constraints = [
        ('protocol_number_unique',
         'unique(protocol_number_part1, protocol_number_part2)',
         "The Protocol Number of a Declaration of Intent must be unique! Please choose another one."),
        ('threshold_positive',
         'CHECK(threshold > 0)',
         "The Threshold of a Declaration of Intent must be positive."),
    ]

    @api.depends('protocol_number_part1', 'protocol_number_part2')
    def _compute_display_name(self):
        for record in self:
            record.display_name = f"{record.protocol_number_part1}-{record.protocol_number_part2}"

    @api.depends('invoice_ids', 'invoice_ids.state', 'invoice_ids.l10n_it_edi_doi_amount')
    def _compute_invoiced(self):
        for declaration in self:
            relevant_invoices = declaration.invoice_ids.filtered(
                lambda invoice: invoice.state == 'posted'
            )
            declaration.invoiced = sum(relevant_invoices.mapped('l10n_it_edi_doi_amount'))

    @api.depends('sale_order_ids', 'sale_order_ids.state', 'sale_order_ids.l10n_it_edi_doi_not_yet_invoiced')
    def _compute_not_yet_invoiced(self):
        for declaration in self:
            relevant_orders = declaration.sale_order_ids.filtered(
                lambda order: order.state == 'sale'
            )
            declaration.not_yet_invoiced = sum(relevant_orders.mapped('l10n_it_edi_doi_not_yet_invoiced'))

    @api.depends('threshold', 'not_yet_invoiced', 'invoiced')
    def _compute_remaining(self):
        for record in self:
            record.remaining = record.threshold - record.invoiced - record.not_yet_invoiced

    def _build_threshold_warning_message(self, invoiced, not_yet_invoiced):
        """
        Build a warning message that will be displayed in a yellow banner on top of a document
        if the `remaining` of the Declaration of Intent is less than 0 when including the document
        or the Declaration of Intent is revoked
            :param float invoiced:          The `declaration.invoiced` amount when including the document.
            :param float not_yet_invoiced:  The `declaration.not_yet_invoiced` amount when including the document.
            :return str:                    The warning message to be shown.
        """
        self.ensure_one()
        updated_remaining = self.threshold - invoiced - not_yet_invoiced
        if self.currency_id.compare_amounts(updated_remaining, 0) >= 0:
            return ''
        return _(
            'Pay attention, the threshold of your Declaration of Intent %s of %s is exceeded by %s, this document included.\n'
            'Invoiced: %s; Not Yet Invoiced: %s',
            self.display_name,
            formatLang(self.env, self.threshold, currency_obj=self.currency_id),
            formatLang(self.env, - updated_remaining, currency_obj=self.currency_id),
            formatLang(self.env, invoiced, currency_obj=self.currency_id),
            formatLang(self.env, not_yet_invoiced, currency_obj=self.currency_id),
        )

    def _get_validity_errors(self, company, partner, currency):
        """
        Check whether all declarations of intent in self are valid for the specified `company`, `partner`, `date` and `currency'.
        Violating these constraints leads to errors in the feature. They should not be ignored.
        Return all errors as a list of strings.
        """
        errors = []
        for declaration in self:
            if not company or declaration.company_id != company:
                errors.append(_("The Declaration of Intent belongs to company %s, not %s.",
                                declaration.company_id.name, company.name))
            if not currency or declaration.currency_id != currency:
                errors.append(_("The Declaration of Intent uses currency %s, not %s.",
                                declaration.currency_id.name, currency.name))
            if not partner or declaration.partner_id != partner.commercial_partner_id:
                errors.append(_("The Declaration of Intent belongs to partner %s, not %s.",
                                declaration.partner_id.name, partner.commercial_partner_id.name))
        return errors

    def _get_validity_warnings(self, company, partner, currency, date, invoiced_amount=0, only_blocking=False, sales_order=False):
        """
        Check whether all declarations of intent in self are valid for the specified `company`, `partner`, `date` and `currency'.
        The checks for `date` and state of the declaration (except draft) are not considered blocking in case `invoiced_amount` is not positive.
        All other checks are considered blocking (prevent posting).
        Includes all checks from `_get_validity_errors`.
        The checks are different for invoices and sales orders (toggled via kwarg `sales_order`).
        I.e. we do not care about the date for sales orders.
        Return all errors as a list of strings.
        """
        errors = []
        for declaration in self:
            errors.extend(declaration._get_validity_errors(company, partner, currency))
            if declaration.state == 'draft':
                errors.append(_("The Declaration of Intent is in draft."))
            if declaration.currency_id.compare_amounts(invoiced_amount, 0) > 0 or not only_blocking:
                if declaration.state != 'active':
                    errors.append(_("The Declaration of Intent must be active."))
                if not sales_order and (not date or declaration.start_date > date or declaration.end_date < date):
                    errors.append(_("The Declaration of Intent is valid from %s to %s, not on %s.",
                                    declaration.start_date, declaration.end_date, date))
        return errors

    @api.model
    def _fetch_valid_declaration_of_intent(self, company, partner, currency, date):
        """
        Fetch a declaration of intent that is valid for the specified `company`, `partner`, `date` and `currency`
        and has not reached the threshold yet.
        """
        return self.search([
            ('state', '=', 'active'),
            ('company_id', '=', company.id),
            ('currency_id', '=', currency.id),
            ('partner_id', '=', partner.commercial_partner_id.id),
            ('start_date', '<=', date),
            ('end_date', '>=', date),
            ('remaining', '>', 0),
        ], limit=1)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_to_document(self):
        if self.invoice_ids or self.sale_order_ids:
            raise UserError(_('You cannot delete Declarations of Intents that are already used on at least one Invoice or Sales Order.'))

    def action_validate(self):
        """ Move a 'draft' Declaration of Intent to 'active'."""
        for record in self:
            if record.state == 'draft':
                record.state = 'active'

    def action_reset_to_draft(self):
        """ Resets an 'active' Declaration of Intent back to 'draft'."""
        for record in self:
            if record.state == 'active':
                record.state = 'draft'

    def action_reactivate(self):
        """ Resets a not 'active' Declaration of Intent back to 'active'."""
        for record in self:
            if record.state != 'active':
                record.state = 'active'

    def action_revoke(self):
        """ Called by the 'revoke' button of the form view."""
        for record in self:
            record.state = 'revoked'

    def action_terminate(self):
        """ Called by the 'terminated' button of the form view."""
        for record in self:
            if record.state != 'revoked':
                record.state = 'terminated'

    def action_open_sale_order_ids(self):
        self.ensure_one()
        return {
            'name': _("Sales Orders using Declaration of Intent %s", self.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'sale.order',
            'domain': [('id', 'in', self.sale_order_ids.ids)],
            'views': [(self.env.ref('l10n_it_edi_doi.view_quotation_tree').id, 'tree'), (False, 'form')],
            'search_view_id': [self.env.ref('sale.sale_order_view_search_inherit_quotation').id],
            'context': {
                'search_default_sales': 1,
            },
        }

    def action_open_invoice_ids(self):
        self.ensure_one()
        return {
            'name': _("Invoices using Declaration of Intent %s", self.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'domain': [('id', 'in', self.invoice_ids.ids)],
            'views': [(self.env.ref('l10n_it_edi_doi.view_move_tree').id, 'tree'), (False, 'form')],
            'search_view_id': [self.env.ref('account.view_account_invoice_filter').id],
            'context': {
                'search_default_posted': 1,
            },
        }

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_it_edi_doi_tax_id = fields.Many2one(
        comodel_name='account.tax',
        string="Declaration of Intent Tax",
    )

    l10n_it_edi_doi_fiscal_position_id = fields.Many2one(
        comodel_name='account.fiscal.position',
        string="Declaration of Intent Fiscal Position",
    )

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_it_edi_doi_ids = fields.One2many(
        'l10n_it_edi_doi.declaration_of_intent',
        'partner_id',
        string="Available Declarations of Intent of this partner",
        domain=lambda self: [('company_id', '=', self.env.company.id)],
    )

    def l10n_it_edi_doi_action_open_declarations(self):
        self.ensure_one()
        return {
            'name': _("Declaration of Intent of %s", self.display_name),
            'type': 'ir.actions.act_window',
            'res_model': 'l10n_it_edi_doi.declaration_of_intent',
            'domain': [('partner_id', '=', self.commercial_partner_id.id)],
            'views': [(self.env.ref('l10n_it_edi_doi.view_l10n_it_edi_doi_tree').id, 'tree'),
                      (self.env.ref('l10n_it_edi_doi.view_l10n_it_edi_doi_form').id, 'form')],
            'context': {
                'default_partner_id': self.id,
            },
        }

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    l10n_it_edi_doi_date = fields.Date(
        string="Date on which Declaration of Intent is applied",
        compute='_compute_l10n_it_edi_doi_date',
    )

    l10n_it_edi_doi_use = fields.Boolean(
        string="Use Declaration of Intent",
        compute='_compute_l10n_it_edi_doi_use',
    )

    l10n_it_edi_doi_id = fields.Many2one(
        string="Declaration of Intent",
        compute='_compute_l10n_it_edi_doi_id',
        store=True,
        readonly=False,
        precompute=True,
        comodel_name='l10n_it_edi_doi.declaration_of_intent',
    )

    l10n_it_edi_doi_not_yet_invoiced = fields.Monetary(
        string='Declaration of Intent Amount Not Yet Invoiced',
        compute='_compute_l10n_it_edi_doi_not_yet_invoiced',
        store=True,
        readonly=True,
        help="Total under the Declaration of Intent of this document that can still be invoiced",
    )

    l10n_it_edi_doi_warning = fields.Text(
        string="Declaration of Intent Threshold Warning",
        compute='_compute_l10n_it_edi_doi_warning',
    )

    @api.depends('date_order')
    def _compute_l10n_it_edi_doi_date(self):
        for order in self:
            order.l10n_it_edi_doi_date = order.date_order or fields.Date.context_today(self)

    @api.depends('l10n_it_edi_doi_id', 'country_code')
    def _compute_l10n_it_edi_doi_use(self):
        for order in self:
            order.l10n_it_edi_doi_use = order.l10n_it_edi_doi_id \
                or order.country_code == "IT"

    @api.depends('company_id', 'partner_id.commercial_partner_id', 'l10n_it_edi_doi_date', 'currency_id')
    def _compute_l10n_it_edi_doi_id(self):
        for order in self:
            if not order.l10n_it_edi_doi_use or order.state != 'draft' and not order.l10n_it_edi_doi_id:
                order.l10n_it_edi_doi_id = False
                continue
            partner = order.partner_id.commercial_partner_id

            # Avoid a query or changing a manually set declaration of intent
            # (if the declaration is still valid).
            validity_warnings = order.l10n_it_edi_doi_id._get_validity_warnings(
                order.company_id, partner, order.currency_id, order.l10n_it_edi_doi_date, sales_order=True
            )
            if order.l10n_it_edi_doi_id and not validity_warnings:
                continue

            declaration = self.env['l10n_it_edi_doi.declaration_of_intent']\
                ._fetch_valid_declaration_of_intent(order.company_id, partner, order.currency_id, order.l10n_it_edi_doi_date)
            order.l10n_it_edi_doi_id = declaration

    @api.depends('l10n_it_edi_doi_id', 'tax_totals', 'order_line', 'order_line.qty_invoiced_posted')
    def _compute_l10n_it_edi_doi_not_yet_invoiced(self):
        for order in self:
            declaration = order.l10n_it_edi_doi_id
            order.l10n_it_edi_doi_not_yet_invoiced = order._l10n_it_edi_doi_get_amount_not_yet_invoiced(declaration)

    @api.depends('l10n_it_edi_doi_id', 'l10n_it_edi_doi_id.remaining', 'state', 'tax_totals')
    def _compute_l10n_it_edi_doi_warning(self):
        for order in self:
            order.l10n_it_edi_doi_warning = ''
            declaration = order.l10n_it_edi_doi_id

            show_warning = declaration and order.state != 'cancelled'
            if not show_warning:
                continue

            declaration_not_yet_invoiced = declaration.not_yet_invoiced
            # Exactly the confirmed SOs (state == 'sale') are included in `declaration.not_yet_invoiced`.
            # The amount of `declaration.not_yet_invoiced` may change due to confirming or saving `order`.
            #   * An unconfirmed order is being confirmed:
            #     We have to add the order amount to `declaration.not_yet_invoiced`.
            #   * A confirmed SO is being edited:
            #     The field `declaration.not_yet_invoiced` will be updated when saving.
            #     But we want to update the warning during the editing already (before saving).
            #     We first have to remove the "old amount" from `declaration.not_yet_invoiced`
            #     before adding the current amount.
            if order.state == 'sale':
                old_order_state = order._origin
                declaration_not_yet_invoiced -= old_order_state.l10n_it_edi_doi_not_yet_invoiced
            declaration_not_yet_invoiced += order.l10n_it_edi_doi_not_yet_invoiced

            validity_warnings = declaration._get_validity_warnings(
                order.company_id, order.partner_id.commercial_partner_id, order.currency_id, order.l10n_it_edi_doi_date,
                sales_order=True
            )

            threshold_warning = declaration._build_threshold_warning_message(declaration.invoiced, declaration_not_yet_invoiced)

            order.l10n_it_edi_doi_warning = '{}\n\n{}'.format('\n'.join(validity_warnings), threshold_warning).strip()

    @api.depends('l10n_it_edi_doi_id')
    def _compute_fiscal_position_id(self):
        super()._compute_fiscal_position_id()
        for order in self:
            declaration_fiscal_position = order.company_id.l10n_it_edi_doi_fiscal_position_id
            if declaration_fiscal_position and order.l10n_it_edi_doi_id:
                order.fiscal_position_id = declaration_fiscal_position

    def _prepare_invoice(self):
        """
        Prepare the dict of values to create the new invoice for a sales order. This method may be
        overridden to implement custom invoice generation (making sure to call super() to establish
        a clean extension chain).
        """
        vals = super()._prepare_invoice()
        declaration = self.l10n_it_edi_doi_id
        if declaration:
            date = fields.Date.context_today(self)
            validity_warnings = declaration._get_validity_warnings(
                self.company_id, self.partner_id.commercial_partner_id, self.currency_id, date, sales_order=True
            )
            if not validity_warnings:
                vals['l10n_it_edi_doi_id'] = declaration.id
        return vals

    def copy_data(self, default=None):
        data_list = super().copy_data(default)
        for order, data in zip(self, data_list):
            partner = order.partner_id.commercial_partner_id
            date = fields.Date.context_today(self)
            if order.l10n_it_edi_doi_id._get_validity_warnings(order.company_id, partner, order.currency_id, date, sales_order=True):
                del data['l10n_it_edi_doi_id']
                del data['fiscal_position_id']
        return data_list

    def _l10n_it_edi_doi_check_configuration(self):
        """
        Raise a UserError in case the configuration of the sale order is invalid.
        """
        errors = []
        for order in self:
            declaration = order.l10n_it_edi_doi_id
            if declaration:
                validity_warnings = declaration._get_validity_warnings(
                    order.company_id, order.partner_id.commercial_partner_id, order.currency_id, order.l10n_it_edi_doi_date,
                    only_blocking=True, sales_order=True,
                )
                errors.extend(validity_warnings)

            declaration_of_intent_tax = order.company_id.l10n_it_edi_doi_tax_id
            if not declaration_of_intent_tax:
                continue
            declaration_tax_lines = order.order_line.filtered(
                lambda line: declaration_of_intent_tax in line.tax_id
            )
            if declaration_tax_lines and not order.l10n_it_edi_doi_id:
                errors.append(_('Given the tax %s is applied, there should be a Declaration of Intent selected.',
                                declaration_of_intent_tax.name))
            if any(line.tax_id != declaration_of_intent_tax for line in declaration_tax_lines):
                errors.append(_('A line using tax %s should not contain any other taxes',
                                declaration_of_intent_tax.name))
        if errors:
            raise UserError('\n'.join(errors))

    def action_quotation_send(self):
        self._l10n_it_edi_doi_check_configuration()
        return super().action_quotation_send()

    def action_quotation_sent(self):
        self._l10n_it_edi_doi_check_configuration()
        return super().action_quotation_sent()

    def action_confirm(self):
        self._l10n_it_edi_doi_check_configuration()
        return super().action_confirm()

    @api.constrains('l10n_it_edi_doi_id')
    def _check_l10n_it_edi_doi_id(self):
        for order in self:
            declaration = order.l10n_it_edi_doi_id
            if not declaration:
                return
            partner = order.partner_id.commercial_partner_id
            errors = declaration._get_validity_warnings(
                order.company_id, partner, order.currency_id, order.l10n_it_edi_doi_date, only_blocking=True, sales_order=True
            )
            if errors:
                raise ValidationError('\n'.join(errors))

    def action_open_declaration_of_intent(self):
        self.ensure_one()
        return {
            'name': _("Declaration of Intent for %s", self.display_name),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'l10n_it_edi_doi.declaration_of_intent',
            'res_id': self.l10n_it_edi_doi_id.id,
        }

    def _l10n_it_edi_doi_get_amount_not_yet_invoiced(self, declaration, additional_invoiced_qty=None):
        """
        Consider sales orders in self that use declaration of intent `declaration`.
        For each sales order we compute the amount that is tax exempt due to the declaration of intent
        (line has special declaration of intent tax applied) but not yet invoiced.
        For each line of the SO we i.e. use the not yet invoiced quantity to compute this amount.
        The aforementioned quantity is computed from field `qty_invoiced_posted` and parameter `additional_invoiced_qty`
        Return the sum of all these amounts on the SOs.
        :param declaration:             We only consider sales orders using Declaration of Intent `declaration`.
        :param additional_invoiced_qty: Dictionary (sale order line id -> float)
                                        The float represents additional invoiced amount qty for the sale order.
                                        This can i.e. be used to simulate posting an already linked invoice.
        """
        if not declaration:
            return 0

        if additional_invoiced_qty is None:
            additional_invoiced_qty = {}

        tax = declaration.company_id.l10n_it_edi_doi_tax_id
        if not tax:
            return 0

        not_yet_invoiced = 0
        for order in self:
            if declaration != order.l10n_it_edi_doi_id:
                continue

            order_lines = order.order_line.filtered(
                # The declaration tax cannot be used with other taxes on a single line
                # (checked in `action_confirm`)
                lambda line: line.tax_id.ids == tax.ids
            )
            order_not_yet_invoiced = 0
            for line in order_lines:
                price_reduce = line.price_unit * (1 - (line.discount or 0.0) / 100.0)
                qty_invoiced = line.qty_invoiced_posted
                if line.ids and additional_invoiced_qty:
                    qty_invoiced += additional_invoiced_qty.get(line.ids[0], 0)
                qty_to_invoice = line.product_uom_qty - qty_invoiced
                order_not_yet_invoiced += price_reduce * qty_to_invoice
            if declaration.currency_id.compare_amounts(order_not_yet_invoiced, 0) > 0:
                not_yet_invoiced += order_not_yet_invoiced

        return not_yet_invoiced

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    qty_invoiced_posted = fields.Float(
        string="Invoiced Quantity (posted)",
        compute='_compute_qty_invoiced_posted',
        digits='Product Unit of Measure',
        store=True,
    )

    @api.depends('invoice_lines.move_id.state', 'invoice_lines.quantity')
    def _compute_qty_invoiced_posted(self):
        """
        This method is almost identical to '_compute_qty_invoiced()'. The only difference lies in the fact that
        for accounting purposes, we only want the quantities of the posted invoices.
        We need a dedicated computation because the triggers are different and could lead to incorrect values for
        'qty_invoiced' when computed together.
        """
        for line in self:
            qty_invoiced_posted = 0.0
            for invoice_line in line._get_invoice_lines():
                if invoice_line.move_id.state == 'posted' or invoice_line.move_id.payment_state == 'invoicing_legacy':
                    qty_unsigned = invoice_line.product_uom_id._compute_quantity(invoice_line.quantity, line.product_uom)
                    qty_signed = qty_unsigned * -invoice_line.move_id.direction_sign
                    qty_invoiced_posted += qty_signed
            line.qty_invoiced_posted = qty_invoiced_posted

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_chart_template
from . import account_fiscal_position
from . import account_move
from . import account_tax
from . import declaration_of_intent
from . import res_company
from . import res_partner
from . import sale_order
from . import sale_order_line

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_it_edi_doi.access_l10n_it_edi_doi_declaration_of_intent_readonly,access_l10n_it_edi_doi_declaration_of_intent,l10n_it_edi_doi.model_l10n_it_edi_doi_declaration_of_intent,account.group_account_readonly,1,0,0,0
l10n_it_edi_doi.access_l10n_it_edi_doi_declaration_of_intent_invoice,access_l10n_it_edi_doi_declaration_of_intent,l10n_it_edi_doi.model_l10n_it_edi_doi_declaration_of_intent,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='journal_id']" position="after">
                <field name="l10n_it_edi_doi_id"/>
            </xpath>
            <xpath expr="//filter[@name='to_check']" position="after">
                <filter string="Exceeded Declaration of Intent"
                        name="l10n_it_edi_doi_declaration_of_intent_exceeded"
                        domain="[('l10n_it_edi_doi_id.remaining','&lt;', 0)]"/>
            </xpath>
            <xpath expr="//group" position="inside">
                <filter string="Declaration of Intent"
                        name="l10n_it_edi_doi_declaration_of_intent"
                        context="{'group_by':'l10n_it_edi_doi_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="view_move_tree" model="ir.ui.view">
        <field name="name">account.move.tree</field>
        <field name="model">account.move</field>
        <field name="arch" type="xml">
            <tree string="Invoices" sample="1" decoration-info="state == 'draft'" expand="context.get('expand', False)">
                <field name="made_sequence_hole" column_invisible="True"/>
                <field name="name" decoration-bf="1" decoration-danger="made_sequence_hole"/>
                <field name="invoice_partner_display_name" string="Customer"/>
                <field name="invoice_date" string="Invoice Date"/>
                <field name="date" string="Accounting Date" optional="hidden"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-success="state == 'posted'"/>
                <field name="l10n_it_edi_doi_amount" decoration-bf="1" sum="Total" string="Tax excluded"/>
            </tree>
        </field>
    </record>

    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button groups="account.group_account_invoice"
                        type="object"
                        class="oe_stat_button"
                        name="action_open_declaration_of_intent"
                        icon="fa-list"
                        invisible="not l10n_it_edi_doi_id">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Declaration of Intent</span>
                    </div>
                </button>
            </div>
            <xpath expr="//header" position="after">
                <div class="alert alert-warning mb-0" role="alert"
                     invisible="not l10n_it_edi_doi_warning">
                    <field name="l10n_it_edi_doi_warning"/>
                </div>
            </xpath>
            <xpath expr="//field[@name='fiscal_position_id']" position="before">
                <field name="l10n_it_edi_doi_use" invisible="True"/>
                <field name="l10n_it_edi_doi_id"
                       invisible="not l10n_it_edi_doi_use"
                       readonly="state != 'draft'"
                       options='{"no_quick_create": True}'
                       domain="[
                           ('state', '!=', 'draft'),
                           ('company_id', '=', company_id),
                           ('currency_id', '=', currency_id),
                           ('partner_id', '=', commercial_partner_id)]"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\l10n_it_edi_doi_declaration_of_intent_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="view_l10n_it_edi_doi_tree" model="ir.ui.view">
        <field name="name">l10n_it_edi_doi.declaration_of_intent.tree</field>
        <field name="model">l10n_it_edi_doi.declaration_of_intent</field>
        <field name="arch" type="xml">
            <tree decoration-info="state == 'draft'"
                  decoration-muted="state == 'terminated'"
                  decoration-danger="state == 'revoked'">
                <control>
                    <create name="add_line_control" string="Add a Declaration of Intent"/>
                </control>
                <field name="currency_id" column_invisible="True"/>
                <field name="partner_id" readonly="state != 'draft'"/>
                <field name="company_id" groups="base.group_multi_company" optional="hidden"/>
                <field name="protocol_number_part1" readonly="state != 'draft'"/>
                <field name="protocol_number_part2" readonly="state != 'draft'"/>
                <field name="issue_date" readonly="state != 'draft'"/>
                <field name="start_date" readonly="state != 'draft'"/>
                <field name="end_date" readonly="state != 'draft'"/>
                <field name="threshold" readonly="state != 'draft'"/>
                <field name="not_yet_invoiced" optional="hidden"/>
                <field name="invoiced" optional="hidden"/>
                <field name="remaining"/>
                <field name="state"/>
            </tree>
        </field>
    </record>

    <record id="view_l10n_it_edi_doi_form" model="ir.ui.view">
        <field name="name">l10n_it_edi_doi.declaration_of_intent.form</field>
        <field name="model">l10n_it_edi_doi.declaration_of_intent</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <field name="state" widget="statusbar" statusbar_visible="draft,active"/>
                    <button string="Validate" invisible="state != 'draft'" class="btn-primary"
                            type="object" name="action_validate"/>
                    <button string="Terminate" invisible="state != 'active'" class="btn-primary"
                            type="object" name="action_terminate"/>
                    <button string="Reset to Draft" invisible="state != 'active'" class="btn-secondary"
                            type="object" name="action_reset_to_draft"/>
                    <button string="Reactivate" invisible="state not in ['terminated', 'revoked']" class="btn-secondary"
                            type="object" name="action_reactivate"/>
                    <button string="Revoke" invisible="state != 'active'" class="btn-secondary"
                            type="object" name="action_revoke"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <field name="invoice_ids" invisible="True"/>
                        <field name="sale_order_ids" invisible="True"/>
                        <button type="object"
                                class="oe_stat_button"
                                name="action_open_invoice_ids"
                                icon="fa-pencil-square-o"
                                invisible="not invoice_ids">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Invoices</span>
                            </div>
                        </button>
                        <button type="object"
                                class="oe_stat_button"
                                name="action_open_sale_order_ids"
                                icon="fa-pencil-square-o"
                                invisible="not sale_order_ids">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Sale Orders</span>
                            </div>
                        </button>
                    </div>
                    <group name="main_group">
                        <group name="left_column">
                            <field name="partner_id" readonly="state != 'draft'"/>
                            <field name="company_id" groups="base.group_multi_company" readonly="state != 'draft'"/>
                            <label for="protocol_number_part1" string="Protocol Number"/>
                            <div name="protocol_div" class="d-flex">
                                <field name="protocol_number_part1" readonly="state != 'draft'"/>
                                <span class="o_form_label mx-3">/</span>
                                <field name="protocol_number_part2" readonly="state != 'draft'"/>
                            </div>
                            <field name="issue_date" readonly="state != 'draft'"/>
                            <label string="Date Range" for="start_date"/>
                            <div name="date_range_div" class="d-flex">
                                <field name="start_date" readonly="state != 'draft'"/>
                                <span class="o_form_label mx-3"> to </span>
                                <field name="end_date" readonly="state != 'draft'"/>
                            </div>
                        </group>
                        <group name="right_column">
                            <div colspan="2" class="o_wrap_label">
                                <span class="o_form_label">Amounts:</span>
                            </div>
                            <field name="currency_id" invisible="True"/>
                            <field name="threshold" widget="monetary" readonly="state != 'draft'"/>
                            <field name="not_yet_invoiced" widget="monetary"/>
                            <field name="invoiced" widget="monetary"/>
                            <field name="remaining" widget="monetary"/>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" groups="base.group_user"/>
                    <field name="message_ids"/>
                    <field name="activity_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="view_l10n_it_edi_doi_declaration_of_intent_search" model="ir.ui.view">
        <field name="name">l10n_it_edi_doi.declaration_of_intent.search</field>
        <field name="model">l10n_it_edi_doi.declaration_of_intent</field>
        <field name="type">search</field>
        <field name="arch" type="xml">
            <search>
                <field name="protocol_number_part1"/>
                <field name="protocol_number_part2"/>
                <filter name="l10n_it_edi_doi_declaration_of_intent_active_filter"
                        string="Active"
                        domain="[('state','=', 'active')]"
                        help="Show active declarations of intent"/>
                <filter name="l10n_it_edi_doi_declaration_of_intent_draft_filter"
                        string="Draft"
                        domain="[('state','=', 'draft')]"
                        help="Show draft declarations of intent"/>
                <filter name="l10n_it_edi_doi_declaration_of_intent_terminated_filter"
                        string="Terminated / Revoked"
                        domain="[('state','in', ['terminated', 'revoked'])]"
                        help="Show terminated or revoked Declarations of Intent"/>
            </search>
        </field>
    </record>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <div name="comment" position="before">
            <div t-if="o.l10n_it_edi_doi_id">
                <span>Your Declaration of Intent number <span t-field="o.l10n_it_edi_doi_id"/> from <span t-field="o.l10n_it_edi_doi_id.issue_date"/>.</span>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="res_partner_view_search" model="ir.ui.view">
        <field name="name">res.partner.search.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.res_partner_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='supplier']" position="after">
                <filter string="Exceeded Declaration of Intent"
                        name="l10n_it_edi_doi_declaration_of_intent_exceeded"
                        domain="[('l10n_it_edi_doi_ids','any', [('remaining', '&lt;', 0)])]"/>
            </xpath>
        </field>
    </record>

    <record id="view_partner_l10n_form" model="ir.ui.view">
        <field name="name">view_partner_l10n_form</field>
        <field name="inherit_id" ref="base_vat.view_partner_base_vat_form"/>
        <field name="model">res.partner</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button groups="account.group_account_invoice"
                        type="object"
                        class="oe_stat_button"
                        name="l10n_it_edi_doi_action_open_declarations"
                        icon="fa-list"
                        invisible="'IT' not in fiscal_country_codes">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Declarations of Intent</span>
                    </div>
                </button>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\sale_ir_actions_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_saleorder_document" inherit_id="sale.report_saleorder_document">
        <p id="fiscal_position_remark" position="after">
            <div t-if="doc.l10n_it_edi_doi_id">
                <span>Your Declaration of Intent number <span t-field="doc.l10n_it_edi_doi_id"/> from <span t-field="doc.l10n_it_edi_doi_id.issue_date"/>.</span>
            </div>
        </p>
    </template>
</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_sales_order_filter" model="ir.ui.view">
        <field name="name">sale.order.list.select</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_sales_order_filter"/>
        <field name="arch" type="xml">
            <filter name="my_sale_orders_filter" position="after">
                <filter string="Exceeded Declaration of Intent"
                        name="l10n_it_edi_doi_declaration_of_intent_exceeded"
                        domain="[('l10n_it_edi_doi_id.remaining','&lt;', 0)]"/>
            </filter>
            <xpath expr="//search/group" position="inside">
                <filter string="Declaration of Intent"
                        name="l10n_it_edi_doi_declaration_of_intent"
                        domain="" context="{'group_by':'l10n_it_edi_doi_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="view_quotation_tree" model="ir.ui.view">
        <field name="name">sale.order.tree</field>
        <field name="model">sale.order</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <tree class="o_sale_order"
                  string="Sales Orders"
                  sample="1"
                  decoration-muted="state == 'cancel'">
                <field name="name" string="Number"/>
                <field name="date_order" widget="date"/>
                <field name="partner_id"/>
                <field name="currency_id" column_invisible="True"/>
                <field name="state"
                       decoration-success="state == 'sale'"
                       decoration-info="state == 'draft'"
                       decoration-primary="state == 'sent'"
                       widget="badge"/>
                <field name="l10n_it_edi_doi_not_yet_invoiced" decoration-bf="1" sum="Total" string="Not Yet Invoiced Amount"/>
            </tree>
        </field>
    </record>

    <record id="view_order_form" model="ir.ui.view">
        <field name="name">sale.order.form</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale.view_order_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button groups="account.group_account_invoice"
                        type="object"
                        class="oe_stat_button"
                        name="action_open_declaration_of_intent"
                        icon="fa-list"
                        invisible="not l10n_it_edi_doi_id">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Declaration of Intent</span>
                    </div>
                </button>
            </div>
            <xpath expr="//header" position="after">
                <div class="alert alert-warning mb-0" role="alert"
                     invisible="not l10n_it_edi_doi_warning">
                    <field name="l10n_it_edi_doi_warning"/>
                </div>
            </xpath>
            <xpath expr="//label[@for='fiscal_position_id']" position="before">
                <field name="l10n_it_edi_doi_use" invisible="True"/>
                <field name="l10n_it_edi_doi_id"
                       invisible="not l10n_it_edi_doi_use"
                       options='{"no_quick_create": True}'
                       domain="[
                           ('state', '!=', 'draft'),
                           ('company_id', '=', company_id),
                           ('currency_id', '=', currency_id),
                           ('partner_id', 'parent_of', partner_id)]"/>
            </xpath>
        </field>
    </record>

</odoo>

```

