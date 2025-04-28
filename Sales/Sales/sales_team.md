# Odoo Module: sales_team

Category: Sales/Sales

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
    'name': 'Sales Teams',
    'version': '1.0',
    'category': 'Sales/Sales',
    'summary': 'Sales Teams',
    'description': """
Using this application you can manage Sales Teams with CRM and/or Sales
=======================================================================
 """,
    'website': 'https://www.odoo.com/page/crm',
    'depends': ['base', 'mail'],
    'data': [
        'security/sales_team_security.xml',
        'security/ir.model.access.csv',
        'data/crm_team_data.xml',
        'views/assets.xml',
        'views/crm_tag_views.xml',
        'views/crm_team_views.xml',
        'views/mail_activity_views.xml',
        'views/res_partner_views.xml',
        ],
    'demo': [
        'data/crm_team_demo.xml',
        'data/crm_tag_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\crm_tag_demo.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

    <!--  crm categories -->
    <record id="categ_oppor1" model="crm.tag">
        <field name="name">Product</field>
        <field name="color" eval="1"/>
    </record>
    <record id="categ_oppor2" model="crm.tag">
        <field name="name">Software</field>
        <field name="color" eval="2"/>
    </record>
    <record id="categ_oppor3" model="crm.tag">
        <field name="name">Services</field>
        <field name="color" eval="3"/>
    </record>
    <record id="categ_oppor4" model="crm.tag">
        <field name="name">Information</field>
        <field name="color" eval="4"/>
    </record>
    <record id="categ_oppor5" model="crm.tag">
        <field name="name">Design</field>
        <field name="color" eval="5"/>
    </record>
    <record id="categ_oppor6" model="crm.tag">
        <field name="name">Training</field>
        <field name="color" eval="6"/>
    </record>
    <record id="categ_oppor7" model="crm.tag">
        <field name="name">Consulting</field>
        <field name="color" eval="7"/>
    </record>
    <record id="categ_oppor8" model="crm.tag">
        <field name="name">Other</field>
        <field name="color" eval="8"/>
    </record>

</data></odoo>

```

## File: data\crm_team_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="team_sales_department" model="crm.team">
            <field name="name">Sales</field>
            <field name="sequence">0</field>
            <field name="member_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="company_id" eval="False"/>
        </record>

        <record id="salesteam_website_sales" model="crm.team">
            <field name="name">Website</field>
            <field name="company_id" eval="False"/>
            <field name="active" eval="False"/>
        </record>

        <record id="pos_sales_team" model="crm.team">
            <field name="name">Point of Sale</field>
            <field name="company_id" eval="False"/>
            <field name="active" eval="False"/>
        </record>

        <record id="ebay_sales_team" model="crm.team">
            <field name="name">eBay</field>
            <field name="active" eval="False"/>
        </record>

        <function model="crm.team" name="message_unsubscribe"
            eval="[
            ref('sales_team.team_sales_department'),
            ref('sales_team.salesteam_website_sales'),
            ref('sales_team.ebay_sales_team'),
            ref('sales_team.pos_sales_team')],
            [ref('base.partner_root')]"/>
    </data>
</odoo>

```

## File: data\crm_team_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- rename for demo-data -->
        <record model="crm.team" id="team_sales_department">
            <field name="name">Europe</field>
            <field name="member_ids" eval="[(4, ref('base.user_demo'))]"/>
            <field name="company_id" eval="False"/>
        </record>
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(4,ref('sales_team.group_sale_salesman'))]"/>
        </record>

        <record id="team_sales_department" model="crm.team">
            <field name="member_ids" eval="[(4, ref('base.user_demo'))]"/>
        </record>

        <record model="crm.team" id="crm_team_1">
            <field name="name">America</field>
            <field name="company_id" eval="False"/>
            <field name="member_ids" eval="[(4, ref('base.user_demo'))]"/>
        </record>
    </data>
</odoo>

```

## File: models\crm_tag.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from random import randint

from odoo import fields, models


class Tag(models.Model):
    _name = "crm.tag"
    _description = "CRM Tag"

    def _get_default_color(self):
        return randint(1, 11)

    name = fields.Char('Tag Name', required=True, translate=True)
    color = fields.Integer('Color', default=_get_default_color)

    _sql_constraints = [
        ('name_uniq', 'unique (name)', "Tag name already exists !"),
    ]

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from babel.dates import format_date
from datetime import date
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.release import version


class CrmTeam(models.Model):
    _name = "crm.team"
    _inherit = ['mail.thread']
    _description = "Sales Team"
    _order = "sequence"
    _check_company_auto = True

    def _get_default_team_id(self, user_id=None, domain=None):
        user_id = user_id or self.env.uid
        user_salesteam_id = self.env['res.users'].browse(user_id).sale_team_id.id
        # Avoid searching on member_ids (+1 query) when we may have the user salesteam already in cache.
        team = self.env['crm.team'].search([
            ('company_id', 'in', [False, self.env.company.id]),
            '|', ('user_id', '=', user_id), ('id', '=', user_salesteam_id),
        ], limit=1)
        if not team and 'default_team_id' in self.env.context:
            team = self.env['crm.team'].browse(self.env.context.get('default_team_id'))
        return team or self.env['crm.team'].search(domain or [], limit=1)

    def _get_default_favorite_user_ids(self):
        return [(6, 0, [self.env.uid])]

    name = fields.Char('Sales Team', required=True, translate=True)
    sequence = fields.Integer('Sequence', default=10)
    active = fields.Boolean(default=True, help="If the active field is set to false, it will allow you to hide the Sales Team without removing it.")
    company_id = fields.Many2one(
        'res.company', string='Company', index=True,
        default=lambda self: self.env.company)
    currency_id = fields.Many2one(
        "res.currency", string="Currency",
        related='company_id.currency_id', readonly=True)
    user_id = fields.Many2one('res.users', string='Team Leader', check_company=True)
    # memberships
    member_ids = fields.One2many(
        'res.users', 'sale_team_id', string='Channel Members',
        check_company=True, domain=[('share', '=', False)],
        help="Add members to automatically assign their documents to this sales team. You can only be member of one team.")
    # UX options
    color = fields.Integer(string='Color Index', help="The color of the channel")
    favorite_user_ids = fields.Many2many(
        'res.users', 'team_favorite_user_rel', 'team_id', 'user_id',
        string='Favorite Members', default=_get_default_favorite_user_ids)
    is_favorite = fields.Boolean(
        string='Show on dashboard', compute='_compute_is_favorite', inverse='_inverse_is_favorite',
        help="Favorite teams to display them in the dashboard and access them easily.")
    dashboard_button_name = fields.Char(string="Dashboard Button", compute='_compute_dashboard_button_name')
    dashboard_graph_data = fields.Text(compute='_compute_dashboard_graph')

    def _compute_is_favorite(self):
        for team in self:
            team.is_favorite = self.env.user in team.favorite_user_ids

    def _inverse_is_favorite(self):
        sudoed_self = self.sudo()
        to_fav = sudoed_self.filtered(lambda team: self.env.user not in team.favorite_user_ids)
        to_fav.write({'favorite_user_ids': [(4, self.env.uid)]})
        (sudoed_self - to_fav).write({'favorite_user_ids': [(3, self.env.uid)]})
        return True

    def _compute_dashboard_button_name(self):
        """ Sets the adequate dashboard button name depending on the Sales Team's options
        """
        for team in self:
            team.dashboard_button_name = _("Big Pretty Button :)") # placeholder

    def _compute_dashboard_graph(self):
        for team in self:
            team.dashboard_graph_data = json.dumps(team._get_dashboard_graph_data())

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model
    def create(self, values):
        team = super(CrmTeam, self.with_context(mail_create_nosubscribe=True)).create(values)
        if values.get('member_ids'):
            team._add_members_to_favorites()
        return team

    def write(self, values):
        res = super(CrmTeam, self).write(values)
        if values.get('member_ids'):
            self._add_members_to_favorites()
        return res

    def unlink(self):
        default_teams = [
            self.env.ref('sales_team.salesteam_website_sales'),
            self.env.ref('sales_team.pos_sales_team'),
            self.env.ref('sales_team.ebay_sales_team')
        ]
        for team in self:
            if team in default_teams:
                raise UserError(_('Cannot delete default team "%s"', team.name))
        return super(CrmTeam,self).unlink()

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def action_primary_channel_button(self):
        """ Skeleton function to be overloaded It will return the adequate action
        depending on the Sales Team's options. """
        return False

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    def _add_members_to_favorites(self):
        for team in self:
            team.favorite_user_ids = [(4, member.id) for member in team.member_ids]

    # ------------------------------------------------------------
    # GRAPH
    # ------------------------------------------------------------

    def _graph_get_model(self):
        """ skeleton function defined here because it'll be called by crm and/or sale
        """
        raise UserError(_('Undefined graph model for Sales Team: %s', self.name))

    def _graph_get_dates(self, today):
        """ return a coherent start and end date for the dashboard graph covering a month period grouped by week.
        """
        start_date = today - relativedelta(months=1)
        # we take the start of the following week if we group by week
        # (to avoid having twice the same week from different month)
        start_date += relativedelta(days=8 - start_date.isocalendar()[2])
        return [start_date, today]

    def _graph_date_column(self):
        return 'create_date'

    def _graph_x_query(self):
        return 'EXTRACT(WEEK FROM %s)' % self._graph_date_column()

    def _graph_y_query(self):
        raise UserError(_('Undefined graph model for Sales Team: %s', self.name))

    def _extra_sql_conditions(self):
        return ''

    def _graph_title_and_key(self):
        """ Returns an array containing the appropriate graph title and key respectively.

            The key is for lineCharts, to have the on-hover label.
        """
        return ['', '']

    def _graph_data(self, start_date, end_date):
        """ return format should be an iterable of dicts that contain {'x_value': ..., 'y_value': ...}
            x_values should be weeks.
            y_values are floats.
        """
        query = """SELECT %(x_query)s as x_value, %(y_query)s as y_value
                     FROM %(table)s
                    WHERE team_id = %(team_id)s
                      AND DATE(%(date_column)s) >= %(start_date)s
                      AND DATE(%(date_column)s) <= %(end_date)s
                      %(extra_conditions)s
                    GROUP BY x_value;"""

        # apply rules
        dashboard_graph_model = self._graph_get_model()
        GraphModel = self.env[dashboard_graph_model]
        graph_table = GraphModel._table
        extra_conditions = self._extra_sql_conditions()
        where_query = GraphModel._where_calc([])
        GraphModel._apply_ir_rules(where_query, 'read')
        from_clause, where_clause, where_clause_params = where_query.get_sql()
        if where_clause:
            extra_conditions += " AND " + where_clause

        query = query % {
            'x_query': self._graph_x_query(),
            'y_query': self._graph_y_query(),
            'table': graph_table,
            'team_id': "%s",
            'date_column': self._graph_date_column(),
            'start_date': "%s",
            'end_date': "%s",
            'extra_conditions': extra_conditions
        }

        self._cr.execute(query, [self.id, start_date, end_date] + where_clause_params)
        return self.env.cr.dictfetchall()

    def _get_dashboard_graph_data(self):
        def get_week_name(start_date, locale):
            """ Generates a week name (string) from a datetime according to the locale:
                E.g.: locale    start_date (datetime)      return string
                      "en_US"      November 16th           "16-22 Nov"
                      "en_US"      December 28th           "28 Dec-3 Jan"
            """
            if (start_date + relativedelta(days=6)).month == start_date.month:
                short_name_from = format_date(start_date, 'd', locale=locale)
            else:
                short_name_from = format_date(start_date, 'd MMM', locale=locale)
            short_name_to = format_date(start_date + relativedelta(days=6), 'd MMM', locale=locale)
            return short_name_from + '-' + short_name_to

        self.ensure_one()
        values = []
        today = fields.Date.from_string(fields.Date.context_today(self))
        start_date, end_date = self._graph_get_dates(today)
        graph_data = self._graph_data(start_date, end_date)
        x_field = 'label'
        y_field = 'value'

        # generate all required x_fields and update the y_values where we have data for them
        locale = self._context.get('lang') or 'en_US'

        weeks_in_start_year = int(date(start_date.year, 12, 28).isocalendar()[1]) # This date is always in the last week of ISO years
        for week in range(0, (end_date.isocalendar()[1] - start_date.isocalendar()[1]) % weeks_in_start_year + 1):
            short_name = get_week_name(start_date + relativedelta(days=7 * week), locale)
            values.append({x_field: short_name, y_field: 0})

        for data_item in graph_data:
            index = int((data_item.get('x_value') - start_date.isocalendar()[1]) % weeks_in_start_year)
            values[index][y_field] = data_item.get('y_value')

        [graph_title, graph_key] = self._graph_title_and_key()
        color = '#875A7B' if '+e' in version else '#7c7bad'
        return [{'values': values, 'area': True, 'title': graph_title, 'key': graph_key, 'color': color}]

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    team_id = fields.Many2one(
        'crm.team', 'Sales Team',
        help='If set, this Sales Team will be used for sales and assignments related to this partner')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    sale_team_id = fields.Many2one(
        'crm.team', "User's Sales Team",
        help='Sales Team the user is member of. Used to compute the members of a Sales Team through the inverse one2many')
```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_team
from . import crm_tag
from . import res_partner
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_crm_team,crm.team,model_crm_team,base.group_user,1,0,0,0
access_crm_team_user,crm.team.user,model_crm_team,sales_team.group_sale_salesman,1,0,0,0
access_crm_team_manager,crm.team.manager,model_crm_team,sales_team.group_sale_manager,1,1,1,1
access_crm_tag,crm_tag,model_crm_tag,base.group_user,0,0,0,0
access_crm_tag_salesman,crm_tag salesman,model_crm_tag,sales_team.group_sale_salesman,1,1,1,0
access_crm_tag_manager,crm_tag manager,model_crm_tag,sales_team.group_sale_manager,1,1,1,1

```

## File: security\sales_team_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="base.module_category_sales_sales" model="ir.module.category">
            <field name="description">Helps you handle your quotations, sale orders and invoicing.</field>
            <field name="sequence">1</field>
        </record>

        <record id="group_sale_salesman" model="res.groups">
            <field name="name">User: Own Documents Only</field>
            <field name="category_id" ref="base.module_category_sales_sales"/>
            <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
            <field name="comment">the user will have access to his own data in the sales application.</field>
        </record>

        <record id="group_sale_salesman_all_leads" model="res.groups">
            <field name="name">User: All Documents</field>
            <field name="category_id" ref="base.module_category_sales_sales"/>
            <field name="implied_ids" eval="[(4, ref('group_sale_salesman'))]"/>
            <field name="comment">the user will have access to all records of everyone in the sales application.</field>
        </record>

        <record id="group_sale_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="comment">the user will have an access to the sales configuration as well as statistic reports.</field>
            <field name="category_id" ref="base.module_category_sales_sales"/>
            <field name="implied_ids" eval="[(4, ref('group_sale_salesman_all_leads'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

        <record model="ir.ui.menu" id="sales_team.menu_sale_config">
            <field name="name">Configuration</field>
            <field eval="[(6,0,[ref('base.group_system')])]" name="groups_id"/>
        </record>

    <data noupdate="1">
        <record id="crm_rule_all_salesteam" model="ir.rule">
            <field name="name">All Salesteam</field>
            <field ref="sales_team.model_crm_team" name="model_id"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4, ref('sales_team.group_sale_salesman_all_leads'))]"/>
        </record>

        <record model="ir.rule" id="sale_team_comp_rule">
            <field name="name">Sales Team multi-company</field>
            <field name="model_id" ref="model_crm_team"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('sales_team.group_sale_manager'))]"/>
        </record>
    </data>
</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="sales_team assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/sales_team/static/src/scss/sales_team_dashboard.scss"/>
        </xpath>
    </template>
</odoo>

```

## File: views\crm_tag_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!--
        crm.tag views
    -->
    <record id="sales_team_crm_tag_view_form" model="ir.ui.view">
        <field name="name">sales.team.crm.tag.view.form</field>
        <field name="model">crm.tag</field>
        <field name="arch" type="xml">
            <form string="Tags">
                <sheet>
                    <div class="oe_title">
                        <div class="oe_edit_only">
                            <label for="name"/>
                        </div>
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="color" required="True"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sales_team_crm_tag_view_tree" model="ir.ui.view">
        <field name="name">sales.team.crm.tag.view.tree</field>
        <field name="model">crm.tag</field>
        <field name="arch" type="xml">
            <tree string="Tags" editable="bottom">
                <field name="name"/>
                <field name="color" widget="color_picker" />
            </tree>
        </field>
    </record>

    <!-- Tags Configuration -->
    <record id="sales_team_crm_tag_action" model="ir.actions.act_window">
        <field name="name">Tags</field>
        <field name="res_model">crm.tag</field>
        <field name="view_id" ref="sales_team_crm_tag_view_tree"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create new tags for your opportunities
            </p><p>
            Create tags that fit your business (product structure, sales type, etc.) to better manage and track your opportunities.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="crm_team_salesteams_search" model="ir.ui.view">
        <field name="name">Sales Teams - Search</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <search string="Salesteams Search">
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <field name="name"/>
                <field name="user_id"/>
                <group expand="0" string="Group By...">
                    <filter string="Team Leader" name="team_leader" domain="[]" context="{'group_by':'user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="crm_team_view_form" model="ir.ui.view">
        <field name="name">crm.team.form</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <form string="Sales Team">
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only" string="Sales Team"/>
                        <h1>
                            <field name="name" placeholder="Sales Team name..."/>
                        </h1>
                        <div name="options_active" />
                    </div>
                    <group>
                        <group name="left">
                            <field name="active" invisible="1"/>
                            <field name="user_id" domain="[('share', '=', False)]"/>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                            <field name="currency_id" invisible="1"/>
                        </group>
                        <group name="right">
                        </group>
                    </group>
                    <notebook>
                        <page name="members" string="Team Members" >
                            <field name="member_ids" widget="many2many" options="{'not_delete': True}">
                                <kanban quick_create="false" create="true" delete="true">
                                    <field name="id"/>
                                    <field name="name"/>
                                    <templates>
                                        <t t-name="kanban-box">
                                            <div class="oe_kanban_global_click" style="max-width: 200px">
                                                <div class="o_kanban_record_top">
                                                    <img t-att-src="kanban_image('res.users', 'image_128', record.id.raw_value)" class="oe_avatar oe_kanban_avatar_smallbox o_image_40_cover mb0" alt="Avatar"/>
                                                    <div class="o_kanban_record_headings ml8">
                                                        <strong class="o_kanban_record_title"><field name="name"/></strong>
                                                    </div>
                                                </div>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" help="Follow this salesteam to automatically track the events associated to users of this team."/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <!-- SALES TEAMS TREE VIEW + MUTI_EDIT -->
    <record id="crm_team_view_tree" model="ir.ui.view">
        <field name="name">crm.team.tree</field>
        <field name="model">crm.team</field>
        <field name="field_parent">child_ids</field>
        <field name="arch" type="xml">
            <tree string="Sales Team" sample="1" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="name" readonly="1"/>
                <field name="active" invisible="1"/>
                <field name="user_id" domain="[('share', '=', False)]"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="crm_team_view_kanban" model="ir.ui.view">
        <field name="name">crm.team.kanban</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" sample="1">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_content oe_kanban_global_click">
                            <div class="row">
                                <div class="col-6">
                                    <strong><field name="name"/></strong>
                                </div>
                                <div class="col-6">
                                    <span class="float-right"><field name="user_id"/></span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- Case Teams Salesteams dashboard view -->
   <record id="crm_team_salesteams_view_kanban" model="ir.ui.view" >
        <field name="name">crm.team.dashboard</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <kanban class="oe_background_grey o_kanban_dashboard o_salesteam_kanban" create="0" sample="1">
                <field name="name"/>
                <field name="user_id"/>
                <field name="member_ids"/>
                <field name="color"/>
                <field name="currency_id"/>
                <field name="is_favorite"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div t-attf-class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary"><field name="name"/></div>
                                </div>
                                <div class="o_kanban_manage_button_section">
                                    <a class="o_kanban_manage_toggle_button" href="#"><i class="fa fa-ellipsis-v" role="img" aria-label="Manage" title="Manage"/></a>
                                </div>
                            </div>
                            <div class="container o_kanban_card_content">
                                <div class="row o_kanban_card_upper_content">
                                    <div class="col-4 o_kanban_primary_left" name="to_replace_in_sale_crm">
                                        <button type="object" class="btn btn-primary" name="action_primary_channel_button"><field name="dashboard_button_name"/></button>
                                    </div>
                                    <div class="col-8 o_kanban_primary_right" style="padding-bottom:0;">
                                        <t name="first_options"/>
                                        <t name="second_options"/>
                                        <t name="third_options"/>
                                    </div>
                                </div>
                                <div class="row">
                                    <div class="col-12 o_kanban_primary_bottom">
                                        <t t-call="SalesTeamDashboardGraph"/>
                                    </div>
                                    <div class="col-12 o_kanban_primary_bottom bottom_block">
                                    </div>
                                </div>
                            </div><div class="container o_kanban_card_manage_pane dropdown-menu" role="menu">
                                <div class="row">
                                    <div class="col-4 o_kanban_card_manage_section o_kanban_manage_view">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>View</span>
                                        </div>
                                    </div>
                                    <div class="col-4 o_kanban_card_manage_section o_kanban_manage_new">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>New</span>
                                        </div>
                                    </div>
                                    <div class="col-4 o_kanban_card_manage_section o_kanban_manage_reports">
                                        <div role="menuitem" class="o_kanban_card_manage_title">
                                            <span>Reporting</span>
                                        </div>
                                    </div>
                                </div>

                                <div t-if="widget.editable" class="o_kanban_card_manage_settings row" groups="sales_team.group_sale_manager">
                                    <div role="menuitem" aria-haspopup="true" class="col-8">
                                        <ul class="oe_kanban_colorpicker" data-field="color" role="menu"/>
                                    </div>
                                    <div role="menuitem" class="col-4">
                                        <a class="dropdown-item" type="edit">Configuration</a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                    <t t-name="SalesTeamDashboardGraph">
                        <div t-if="record.dashboard_graph_data.raw_value" class="o_sales_team_kanban_graph_section">
                            <field name="dashboard_graph_data" widget="dashboard_graph" t-att-graph_type="'bar'"/>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- Sales Teams Actions -->
    <record id="crm_team_salesteams_act" model="ir.actions.act_window">
        <field name="name">Sales Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">kanban,form</field>
        <field name="view_id" ref="crm_team_salesteams_search"/>
        <field name="context">{'in_sales_app': True}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Define a new sales team
            </p><p>
                Use Sales Teams to organize your sales departments.
                Each team will work with a separate pipeline.
            </p>
        </field>
    </record>

      <record id="crm_team_salesteams_pipelines_act" model="ir.actions.act_window">
        <field name="name">Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">kanban,form</field>
        <field name="view_id" ref="crm_team_salesteams_search"/>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Define a new sales team
            </p><p>
                Use Sales Teams to organize your sales departments.
                Each team will work with a separate pipeline.
            </p>
        </field>
    </record>

    <record id="sales_team_config_action" model="ir.actions.act_window">
        <field name="name">Sales Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{}</field>
        <field name="view_id" ref="crm_team_salesteams_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Define a new sales team
            </p><p>
                Use Sales Teams to organize your sales departments.
                Each team will work with a separate pipeline.
            </p>
        </field>
    </record>

</data>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_activity_type_action_config_sales" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', '=', 'res.partner')]</field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_partner_view_team" model="ir.ui.view">
        <field name="name">res.partner.view.team</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form" />
        <field name="arch" type="xml">
            <xpath expr="//page[@name='sales_purchases']//field[@name='user_id']" position="after">
                <field name="team_id" groups="base.group_no_one" />
            </xpath>
        </field>
    </record>
</odoo>

```

