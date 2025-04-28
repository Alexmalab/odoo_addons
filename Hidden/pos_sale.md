# Odoo Module: pos_sale

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'pos_sale',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between Point of Sale and Sales',
    'description': """

This module adds a custom Sales Team for the Point of Sale. This enables you to view and manage your point of sale sales with more ease.
""",
    'depends': ['point_of_sale', 'sale_management'],
    'data': [
        'data/pos_sale_data.xml',
        'security/pos_sale_security.xml',
        'security/ir.model.access.csv',
        'views/sales_team_views.xml',
        'views/pos_config_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\pos_sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="sales_team.pos_sales_team" model="crm.team">
            <field name="active" eval="True"/>
        </record>
    </data>
</odoo>


```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from datetime import datetime
import pytz


class CrmTeam(models.Model):
    _inherit = 'crm.team'

    pos_config_ids = fields.One2many('pos.config', 'crm_team_id', string="Point of Sales")
    pos_sessions_open_count = fields.Integer(string='Open POS Sessions', compute='_compute_pos_sessions_open_count')
    pos_order_amount_total = fields.Float(string="Session Sale Amount", compute='_compute_pos_order_amount_total')

    def _compute_pos_sessions_open_count(self):
        for team in self:
            team.pos_sessions_open_count = self.env['pos.session'].search_count([('config_id.crm_team_id', '=', team.id), ('state', '=', 'opened')])

    def _compute_pos_order_amount_total(self):
        for team in self:
            res = self.env['report.pos.order'].read_group(
                [("config_id.crm_team_id", "=", team.id), ("session_id.state", "=", "opened")],
                ["price_total"],
                [],
            )
            team.pos_order_amount_total = sum(r["price_total"] for r in res if r["price_total"])

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.exceptions import AccessError


class PosConfig(models.Model):
    _inherit = 'pos.config'


    crm_team_id = fields.Many2one(
        'crm.team', string="Sales Team",
        help="This Point of sale's sales will be related to this Sales Team.")

    @api.onchange('company_id')
    def _get_default_pos_team(self):
        default_sale_team = self.env.ref('sales_team.pos_sales_team', raise_if_not_found=False)
        if default_sale_team and (not default_sale_team.company_id or default_sale_team.company_id == self.company_id):
            self.crm_team_id = default_sale_team
        else:
            self.crm_team_id = self.env['crm.team'].search(['|', ('company_id', '=', self.company_id.id), ('company_id', '=', False)], limit=1)

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    currency_rate = fields.Float("Currency Rate", compute='_compute_currency_rate', store=True, digits=0, readonly=True, help='The rate of the currency to the currency of rate applicable at the date of the order')
    crm_team_id = fields.Many2one('crm.team', string="Sales Team")

    @api.model
    def _complete_values_from_session(self, session, values):
        values = super(PosOrder, self)._complete_values_from_session(session, values)
        values.setdefault('crm_team_id', session.config_id.crm_team_id.id)
        return values

    @api.depends('pricelist_id.currency_id', 'date_order', 'company_id')
    def _compute_currency_rate(self):
        for order in self:
            date_order = order.date_order or fields.Datetime.now()
            order.currency_rate = self.env['res.currency']._get_conversion_rate(order.company_id.currency_id, order.pricelist_id.currency_id, order.company_id, date_order)

    def _prepare_invoice(self):
        invoice_vals = super(PosOrder, self)._prepare_invoice()
        invoice_vals['team_id'] = self.crm_team_id
        return invoice_vals

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PosSession(models.Model):
    _inherit = 'pos.session'

    crm_team_id = fields.Many2one('crm.team', related='config_id.crm_team_id', string="Sales Team", readonly=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import crm_team
from . import pos_session

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import tools
from odoo import api, fields, models


class SaleReport(models.Model):
    _inherit = "sale.report"

    @api.model
    def _get_done_states(self):
        done_states = super(SaleReport, self)._get_done_states()
        done_states.extend(['pos_done', 'invoiced'])
        return done_states

    state = fields.Selection(selection_add=[('pos_draft', 'New'),
                                            ('paid', 'Paid'),
                                            ('pos_done', 'Posted'),
                                            ('invoiced', 'Invoiced')], string='Status', readonly=True)

    def _query(self, with_clause='', fields={}, groupby='', from_clause=''):
        res = super(SaleReport, self)._query(with_clause, fields, groupby, from_clause)

        select_ = '''
            -MIN(l.id) AS id,
            l.product_id AS product_id,
            t.uom_id AS product_uom,
            sum(l.qty) AS product_uom_qty,
            sum(l.qty) AS qty_delivered,
            CASE WHEN pos.state = 'invoiced' THEN sum(l.qty) ELSE 0 END AS qty_invoiced,
            CASE WHEN pos.state != 'invoiced' THEN sum(l.qty) ELSE 0 END AS qty_to_invoice,
            SUM(l.price_subtotal_incl) / MIN(CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END) AS price_total,
            SUM(l.price_subtotal) / MIN(CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END) AS price_subtotal,
            (CASE WHEN pos.state != 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END) / MIN(CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END) AS amount_to_invoice,
            (CASE WHEN pos.state = 'invoiced' THEN SUM(l.price_subtotal) ELSE 0 END) / MIN(CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END) AS amount_invoiced,
            count(*) AS nbr,
            pos.name AS name,
            pos.date_order AS date,
            CASE WHEN pos.state = 'draft' THEN 'pos_draft' WHEN pos.state = 'done' THEN 'pos_done' else pos.state END AS state,
            pos.partner_id AS partner_id,
            pos.user_id AS user_id,
            pos.company_id AS company_id,
            NULL AS campaign_id,
            NULL AS medium_id,
            NULL AS source_id,
            extract(epoch from avg(date_trunc('day',pos.date_order)-date_trunc('day',pos.create_date)))/(24*60*60)::decimal(16,2) AS delay,
            t.categ_id AS categ_id,
            pos.pricelist_id AS pricelist_id,
            NULL AS analytic_account_id,
            pos.crm_team_id AS team_id,
            p.product_tmpl_id,
            partner.country_id AS country_id,
            partner.industry_id AS industry_id,
            partner.commercial_partner_id AS commercial_partner_id,
            (select sum(t.weight*l.qty/u.factor) from pos_order_line l
               join product_product p on (l.product_id=p.id)
               left join product_template t on (p.product_tmpl_id=t.id)
               left join uom_uom u on (u.id=t.uom_id)) AS weight,
            (select sum(t.volume*l.qty/u.factor) from pos_order_line l
               join product_product p on (l.product_id=p.id)
               left join product_template t on (p.product_tmpl_id=t.id)
               left join uom_uom u on (u.id=t.uom_id)) AS volume,
            l.discount as discount,
            sum((l.price_unit * l.discount * l.qty / 100.0 / CASE COALESCE(pos.currency_rate, 0) WHEN 0 THEN 1.0 ELSE pos.currency_rate END)) as discount_amount,
            NULL as order_id
        '''

        for field in fields.keys():
            select_ += ', NULL AS %s' % (field)

        from_ = '''
            pos_order_line l
                  join pos_order pos on (l.order_id=pos.id)
                  left join res_partner partner ON (pos.partner_id = partner.id OR pos.partner_id = NULL)
                    left join product_product p on (l.product_id=p.id)
                    left join product_template t on (p.product_tmpl_id=t.id)
                    LEFT JOIN uom_uom u ON (u.id=t.uom_id)
                    LEFT JOIN pos_session session ON (session.id = pos.session_id)
                    LEFT JOIN pos_config config ON (config.id = session.config_id)
                left join product_pricelist pp on (pos.pricelist_id = pp.id)
        '''

        groupby_ = '''
            l.order_id,
            l.product_id,
            l.price_unit,
            l.discount,
            l.qty,
            t.uom_id,
            t.categ_id,
            pos.name,
            pos.date_order,
            pos.partner_id,
            pos.user_id,
            pos.state,
            pos.company_id,
            pos.pricelist_id,
            p.product_tmpl_id,
            partner.country_id,
            partner.industry_id,
            partner.commercial_partner_id,
            u.factor,
            pos.crm_team_id
        '''
        current = '(SELECT %s FROM %s GROUP BY %s)' % (select_, from_, groupby_)

        return '%s UNION ALL %s' % (res, current)

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_team_pos_manager,crm.team.pos.manager,model_crm_team,point_of_sale.group_pos_manager,1,1,1,1

```

## File: security\pos_sale_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_sale_rule_pos_channel_pos_manager" model="ir.rule">
        <field name="name">POS Sales Team</field>
        <field name="model_id" ref="sales_team.model_crm_team"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('point_of_sale.group_pos_manager'))]"/>
    </record>
</odoo>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="pos_config_view_form_pos_sale" model="ir.ui.view">
        <field name="name">pos.config.form.pos.sale</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <div id="accounting_section" position="after">
                <h2>Sales Reporting</h2>
                <div class="row mt16 o_settings_container">
                    <div class="col-12 col-lg-6 o_setting_box">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Sales Team</span>
                            <div class="text-muted">
                                Sales are reported to the following sales team
                            </div>
                            <field name="crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]"/>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\sales_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_session_search_inherit_pos_sale" model="ir.ui.view">
        <field name="name">pos.session.search.view</field>
        <field name="model">pos.session</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_session_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='config_id']" position="after">
                <field name="crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
            </xpath>
        </field>
    </record>

    <record id="pos_session_action_from_crm_team" model="ir.actions.act_window">
        <field name="name">Open Sessions</field>
        <field name="binding_model_id" ref="sales_team.model_crm_team"/>
        <field name="res_model">pos.session</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'search_default_open_sessions': True, 'search_default_crm_team_id': active_id}</field>
    </record>

    <record id="view_pos_config_search_inherit_pos_sale" model="ir.ui.view">
        <field name="name">pos.config.search.view</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_config_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position="after">
                <field name="crm_team_id" kanban_view_ref="%(sales_team.crm_team_view_kanban)s"/>
            </xpath>
        </field>
    </record>

    <record id="crm_team_salesteams_view_kanban_inherit_pos_sale" model="ir.ui.view"> 
        <field name="name">crm.team.kanban</field>
        <field name="model">crm.team</field>
        <field name="inherit_id" ref="sales_team.crm_team_salesteams_view_kanban"/>
        <field name="groups_id" eval="[(4, ref('point_of_sale.group_pos_user'))]"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//t[@name='first_options']" position="before">
                <div class="row" t-if="record.pos_sessions_open_count.raw_value">
                    <div class="col-8">
                        <a name="%(pos_session_action_from_crm_team)d" type="action">
                            <field name="pos_sessions_open_count"/>
                            <t t-if="record.pos_sessions_open_count.raw_value == 1">Session Running</t>
                            <t t-else="">Sessions Running</t>
                        </a>
                    </div>
                    <div class="col-4 text-right">
                        <field name="pos_order_amount_total" widget="monetary"/>
                    </div>
                </div>
            </xpath>

        </data>
        </field>
    </record>

</odoo>

```

