# Odoo Module: mass_mailing_sale

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Mass mailing on sale orders',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add sale order UTM info on mass mailing',
    'description': """UTM and mass mailing on sale orders""",
    'depends': ['sale', 'mass_mailing'],
    'data': [
        'views/mailing_mailing_views.xml',
    ],
    'demo': [
        'data/mass_mailing_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mass_mailing_demo.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data noupdate="0">
    <record id="mass_mail_sale_order_0" model="mailing.mailing">
        <field name="name">Sale Promotion 1</field>
        <field name="subject">Our last promotions, just for you !</field>
        <field name="state">draft</field>
        <field name="user_id" ref="base.user_admin"/>
        <field name="schedule_date" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        <field name="campaign_id" ref="mass_mailing.mass_mail_campaign_1"/>
        <field name="source_id" ref="sale.utm_source_sale_order_0"/>
        <field name="mailing_model_id" ref="sale.model_sale_order"/>
        <field name="mailing_domain">[]</field>
        <field name="reply_to_mode">new</field>
        <field name="reply_to">{{ object.company_id.email }}</field>
        <field name="body_arch" type="html">
<div class="oe_structure o_editable">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 24px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="100%" style="background-color: white; padding: 0; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Hello</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-out="object.name" />
                    </span>
                </td><td valign="middle" align="right">
                    <img t-att-src="'/logo.png?company=%s' % object.company_id.id" style="padding: 0px; margin: 0px; height: 48px;" t-att-alt="object.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:4px 0px 32px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td style="min-width: 590px;">
            <p>Dear <t t-out="object.name" /></p>
            <p>Great stories have personality. Consider telling a great story that provides personality. Writing a story with personality for potential clients will assist with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.</p>
            <p>Great stories are for everyone even when only written for just one person. If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.</p>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px; padding: 0 8px 0 8px; font-size:11px;">
            <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 4px 0px;"/>
            <b><t t-out="object.company_id.name" /></b><br/>
            <div style="color: #999999;">
                <t t-out="object.company_id.phone" />
                <t t-if="object.company_id.email">
                |
                    <a t-att-href="'mailto:%s' % object.company_id.email" style="text-decoration:none; color: #999999;">
                        <t t-out="object.company_id.email" />
                    </a>
                </t>
                <t t-if="object.company_id.website">
                |
                    <a t-att-href="'%s' % object.company_id.website" style="text-decoration:none; color: #999999;">
                        <t t-out="object.company_id.website" />
                    </a>
                </t>
            </div>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=email" style="color: #875A7B;">Odoo</a>
</td></tr>
</table></div></field>
    </record>
</data>
</odoo>

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup
from odoo import api, fields, models, _, tools
from odoo.osv import expression


class MassMailing(models.Model):
    _name = 'mailing.mailing'
    _inherit = 'mailing.mailing'

    sale_quotation_count = fields.Integer('Quotation Count', compute='_compute_sale_quotation_count')
    sale_invoiced_amount = fields.Integer('Invoiced Amount', compute='_compute_sale_invoiced_amount')

    @api.depends('mailing_domain')
    def _compute_sale_quotation_count(self):
        quotation_data = self.env['sale.order'].sudo()._read_group(
            [('source_id', 'in', self.source_id.ids), ('order_line', '!=', False)],
            ['source_id'], ['source_id'],
        )
        mapped_data = {datum['source_id'][0]: datum['source_id_count'] for datum in quotation_data}
        for mass_mailing in self:
            mass_mailing.sale_quotation_count = mapped_data.get(mass_mailing.source_id.id, 0)

    @api.depends('mailing_domain')
    def _compute_sale_invoiced_amount(self):
        domain = expression.AND([
            [('source_id', 'in', self.source_id.ids)],
            [('state', 'not in', ['draft', 'cancel'])]
        ])
        moves_data = self.env['account.move'].sudo()._read_group(
            domain, ['source_id', 'amount_untaxed_signed'], ['source_id'],
        )
        mapped_data = {datum['source_id'][0]: datum['amount_untaxed_signed'] for datum in moves_data}
        for mass_mailing in self:
            mass_mailing.sale_invoiced_amount = mapped_data.get(mass_mailing.source_id.id, 0)

    def action_redirect_to_quotations(self):
        helper_header = _("No Quotations yet!")
        helper_message = _("Quotations will appear here once your customers add "
                           "products to their Carts or when your sales reps assign this mailing.")
        return {
            'context': {
                'create': False,
                'search_default_group_by_date_day': True,
                'sale_report_view_hide_date': True,
            },
            'domain': [('source_id', '=', self.source_id.id)],
            'help': Markup('<p class="o_view_nocontent_smiling_face">%s</p><p>%s</p>') % (
                helper_header, helper_message,
            ),
            'name': _("Sales Analysis"),
            'res_model': 'sale.report',
            'type': 'ir.actions.act_window',
            'view_mode': 'graph,pivot,tree,form',
        }

    def action_redirect_to_invoiced(self):
        domain = expression.AND([
            [('source_id', '=', self.source_id.id)],
            [('state', 'not in', ['draft', 'cancel'])]
        ])
        moves = self.env['account.move'].search(domain)
        helper_header = _("No Revenues yet!")
        helper_message = _("Revenues will appear here once orders are turned into invoices.")
        return {
            'context': {
                'create': False,
                'edit': False,
                'view_no_maturity': True,
                'search_default_group_by_invoice_date_week': True,
                'invoice_report_view_hide_invoice_date': True,
            },
            'domain': [('move_id', 'in', moves.ids)],
            'help': Markup('<p class="o_view_nocontent_smiling_face">%s</p><p>%s</p>') % (
                helper_header, helper_message,
            ),
            'name': _("Invoices Analysis"),
            'res_model': 'account.invoice.report',
            'type': 'ir.actions.act_window',
            'view_mode': 'graph,pivot,tree,form',
        }

    def _prepare_statistics_email_values(self):
        self.ensure_one()
        values = super(MassMailing, self)._prepare_statistics_email_values()
        if not self.user_id:
            return values

        self_with_company = self.with_company(self.user_id.company_id)
        currency = self.user_id.company_id.currency_id
        formated_amount = tools.format_decimalized_amount(self_with_company.sale_invoiced_amount, currency)

        values['kpi_data'][1]['kpi_col2'] = {
            'value': self.sale_quotation_count,
            'col_subtitle': _('QUOTATIONS'),
        }
        values['kpi_data'][1]['kpi_col3'] = {
            'value': formated_amount,
            'col_subtitle': _('INVOICED'),
        }
        values['kpi_data'][1]['kpi_name'] = 'sale'
        return values

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrder(models.Model):
    _inherit = 'sale.order'
    _mailing_enabled = True

    def _mailing_get_default_domain(self, mailing):
        """ Exclude by default canceled orders when performing a mass mailing. """
        return [('state', '!=', 'cancel')]

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    ab_testing_winner_selection = fields.Selection(selection_add=[
        ('sale_quotation_count', 'Quotations'),
        ('sale_invoiced_amount', 'Revenues'),
    ])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_mailing
from . import sale_order
from . import utm

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_mailing_view_form" model="ir.ui.view">
        <field name="name">mailing.mailing.view.form.inherit.sale</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
        <field name="arch" type="xml">
    		<xpath expr="//button[@id='button_view_delivered']" position="before">
                <button name="action_redirect_to_quotations"
                    type="object"
                    icon="fa-pencil-square-o"
                    class="oe_stat_button"
                    attrs="{'invisible': [('state', '=', 'draft')]}">
                    <field name="sale_quotation_count" string="Quotations" widget="statinfo"/>
                </button>
                <button name="action_redirect_to_invoiced"
                    type="object"
                    icon="fa-dollar"
                    class="oe_stat_button"
                    attrs="{'invisible': [('state', '=', 'draft')]}" >
                    <field name="sale_invoiced_amount" string="Invoiced" widget="statinfo"/>
                </button>
    		</xpath>
        </field>
    </record>
</odoo>

```

