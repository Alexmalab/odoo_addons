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
    'version': '1.1',
    'category': 'Sales/Sales',
    'summary': 'Sales Teams',
    'description': """
Using this application you can manage Sales Teams with CRM and/or Sales
=======================================================================
 """,
    'website': 'https://www.odoo.com/app/crm',
    'depends': ['base', 'mail'],
    'data': [
        'security/sales_team_security.xml',
        'security/ir.model.access.csv',
        'data/crm_team_data.xml',
        'views/crm_tag_views.xml',
        'views/crm_team_views.xml',
        'views/crm_team_member_views.xml',
        'views/mail_activity_views.xml',
        'views/res_partner_views.xml',
        ],
    'demo': [
        'data/crm_team_demo.xml',
        'data/crm_tag_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'sales_team/static/**/*',
        ],
    },
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
            <field name="company_id" eval="False"/>
        </record>
        <record id="crm_team_member_admin_sales" model="crm.team.member" forcecreate="0">
            <field name="crm_team_id" ref="team_sales_department"/>
            <field name="user_id" ref="base.user_admin"/>
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
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[
                (3, ref('sales_team.group_sale_manager')),
                (3, ref('sales_team.group_sale_salesman_all_leads')),
                (4, ref('sales_team.group_sale_salesman'))]"/>
        </record>

        <record model="crm.team" id="team_sales_department">
            <field name="name">Sales</field>
        </record>

        <record model="crm.team" id="crm_team_1">
            <field name="name">Pre-Sales</field>
            <field name="company_id" eval="False"/>
        </record>

        <record id="crm_team_member_demo_team_1" model="crm.team.member">
            <field name="user_id" ref="base.user_demo"/>
            <field name="crm_team_id" ref="sales_team.crm_team_1"/>
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
        ('name_uniq', 'unique (name)', "Tag name already exists!"),
    ]

```

## File: models\crm_team.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import random

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
    _order = "sequence ASC, create_date DESC, id DESC"
    _check_company_auto = True

    def _get_default_team_id(self, user_id=False, domain=False):
        """ Compute default team id for sales related documents. Note that this
        method is not called by default_get as it takes some additional
        parameters and is meant to be called by other default methods.

        Heuristic (when multiple match: take from default context value or first
        sequence ordered)

          1- any of my teams (member OR responsible) matching domain, either from
             context or based on _order;
          2- any of my teams (member OR responsible), either from context or based
             on _order;
          3- default from context
          4- any team matching my company and domain (based on company rule)
          5- any team matching my company (based on company rule)

        Note: ResPartner.team_id field is explicitly not taken into account. We
        think this field causes a lot of noises compared to its added value.
        Think notably: team not in responsible teams, team company not matching
        responsible or lead company, asked domain not matching, ...

        :param user_id: salesperson to target, fallback on env.uid;
        :domain: optional domain to filter teams (like use_lead = True);
        """
        if not user_id:
            user = self.env.user
        else:
            user = self.env['res.users'].sudo().browse(user_id)
        default_team = self.env['crm.team'].browse(
            self.env.context['default_team_id']
        ) if self.env.context.get('default_team_id') else self.env['crm.team']
        valid_cids = [False] + [c for c in user.company_ids.ids if c in self.env.companies.ids]

        # 1- find in user memberships - note that if current user in C1 searches
        # for team belonging to a user in C1/C2 -> only results for C1 will be returned
        team = self.env['crm.team']
        teams = self.env['crm.team'].search([
            ('company_id', 'in', valid_cids),
             '|', ('user_id', '=', user.id), ('member_ids', 'in', [user.id])
        ])
        if teams and domain:
            filtered_teams = teams.filtered_domain(domain)
            if default_team and default_team in filtered_teams:
                team = default_team
            else:
                team = filtered_teams[:1]

        # 2- any of my teams
        if not team:
            if default_team and default_team in teams:
                team = default_team
            else:
                team = teams[:1]

        # 3- default: context
        if not team and default_team:
            team = default_team

        if not team:
            teams = self.env['crm.team'].search([('company_id', 'in', valid_cids)])
            # 4- default: based on company rule, first one matching domain
            if teams and domain:
                team = teams.filtered_domain(domain)[:1]
            # 5- default: based on company rule, first one
            if not team:
                team = teams[:1]

        return team

    def _get_default_favorite_user_ids(self):
        return [(6, 0, [self.env.uid])]

    # description
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
    is_membership_multi = fields.Boolean(
        'Multiple Memberships Allowed', compute='_compute_is_membership_multi',
        help='If True, users may belong to several sales teams. Otherwise membership is limited to a single sales team.')
    member_ids = fields.Many2many(
        'res.users', string='Salespersons',
        domain="['&', ('share', '=', False), ('company_ids', 'in', member_company_ids)]",
        compute='_compute_member_ids', inverse='_inverse_member_ids', search='_search_member_ids',
        help="Users assigned to this team.")
    member_company_ids = fields.Many2many(
        'res.company', compute='_compute_member_company_ids',
        help='UX: Limit to team company or all if no company')
    member_warning = fields.Text('Membership Issue Warning', compute='_compute_member_warning')
    crm_team_member_ids = fields.One2many(
        'crm.team.member', 'crm_team_id', string='Sales Team Members',
        context={'active_test': True},
        help="Add members to automatically assign their documents to this sales team.")
    crm_team_member_all_ids = fields.One2many(
        'crm.team.member', 'crm_team_id', string='Sales Team Members (incl. inactive)',
        context={'active_test': False})
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

    @api.depends('sequence')  # TDE FIXME: force compute in new mode
    def _compute_is_membership_multi(self):
        multi_enabled = self.env['ir.config_parameter'].sudo().get_param('sales_team.membership_multi', False)
        self.is_membership_multi = multi_enabled

    @api.depends('crm_team_member_ids.active')
    def _compute_member_ids(self):
        for team in self:
            team.member_ids = team.crm_team_member_ids.user_id

    def _inverse_member_ids(self):
        for team in self:
            # pre-save value to avoid having _compute_member_ids interfering
            # while building membership status
            memberships = team.crm_team_member_ids
            users_current = team.member_ids
            users_new = users_current - memberships.user_id

            # add missing memberships
            self.env['crm.team.member'].create([{'crm_team_id': team.id, 'user_id': user.id} for user in users_new])

            # activate or deactivate other memberships depending on members
            for membership in memberships:
                membership.active = membership.user_id in users_current

    @api.depends('is_membership_multi', 'member_ids')
    def _compute_member_warning(self):
        """ Display a warning message to warn user they are about to archive
        other memberships. Only valid in mono-membership mode and take into
        account only active memberships as we may keep several archived
        memberships. """
        self.member_warning = False
        if all(team.is_membership_multi for team in self):
            return
        # done in a loop, but to be used in form view only -> not optimized
        for team in self:
            member_warning = False
            other_memberships = self.env['crm.team.member'].search([
                ('crm_team_id', '!=', team.id if team.ids else False),  # handle NewID
                ('user_id', 'in', team.member_ids.ids)
            ])
            if other_memberships and len(other_memberships) == 1:
                member_warning = _("Adding %(user_name)s in this team would remove him/her from its current team %(team_name)s.",
                                   user_name=other_memberships.user_id.name,
                                   team_name=other_memberships.crm_team_id.name
                                  )
            elif other_memberships:
                member_warning = _("Adding %(user_names)s in this team would remove them from their current teams (%(team_names)s).",
                                   user_names=", ".join(other_memberships.mapped('user_id.name')),
                                   team_names=", ".join(other_memberships.mapped('crm_team_id.name'))
                                  )
            if member_warning:
                team.member_warning = member_warning + " " + _("To add a Salesperson into multiple Teams, activate the Multi-Team option in settings.")

    def _search_member_ids(self, operator, value):
        return [('crm_team_member_ids.user_id', operator, value)]

    # 'name' should not be in the trigger, but as 'company_id' is possibly not present in the view
    # because it depends on the multi-company group, we use it as fake trigger to force computation
    @api.depends('company_id', 'name')
    def _compute_member_company_ids(self):
        """ Available companies for members. Either team company if set, either
        any company if not set on team. """
        all_companies = self.env['res.company'].search([])
        for team in self:
            team.member_company_ids = team.company_id or all_companies

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

    @api.model_create_multi
    def create(self, vals_list):
        teams = super(CrmTeam, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        teams.filtered(lambda t: t.member_ids)._add_members_to_favorites()
        return teams

    def write(self, values):
        res = super(CrmTeam, self).write(values)
        # manually launch company sanity check
        if values.get('company_id'):
            self.crm_team_member_ids._check_company(fnames=['crm_team_id'])

        if values.get('member_ids'):
            self._add_members_to_favorites()
        return res

    @api.ondelete(at_uninstall=False)
    def _unlink_except_default(self):
        default_teams = [
            self.env.ref('sales_team.salesteam_website_sales'),
            self.env.ref('sales_team.pos_sales_team'),
            self.env.ref('sales_team.ebay_sales_team')
        ]
        for team in self:
            if team in default_teams:
                raise UserError(_('Cannot delete default team "%s"', team.name))

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

    def _graph_get_table(self, GraphModel):
        return GraphModel._table

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
        graph_table = self._graph_get_table(GraphModel)
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
        week_count = (end_date.isocalendar()[1] - start_date.isocalendar()[1]) % weeks_in_start_year + 1
        for week in range(week_count):
            short_name = get_week_name(start_date + relativedelta(days=7 * week), locale)
            values.append({x_field: short_name, y_field: 0, 'type': 'future' if week + 1 == week_count else 'past'})

        for data_item in graph_data:
            index = int((data_item.get('x_value') - start_date.isocalendar()[1]) % weeks_in_start_year)
            values[index][y_field] = data_item.get('y_value')

        [graph_title, graph_key] = self._graph_title_and_key()
        color = '#875A7B' if '+e' in version else '#7c7bad'

        # If no actual data available, show some sample data
        if not graph_data:
            graph_key = _('Sample data')
            for value in values:
                value['type'] = 'o_sample_data'
                # we use unrealistic values for the sample data
                value['value'] = random.randint(0, 20)
        return [{'values': values, 'area': True, 'title': graph_title, 'key': graph_key, 'color': color}]

```

## File: models\crm_team_member.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, exceptions, fields, models, _


class CrmTeamMember(models.Model):
    _name = 'crm.team.member'
    _inherit = ['mail.thread']
    _description = 'Sales Team Member'
    _rec_name = 'user_id'
    _order = 'create_date ASC, id'
    _check_company_auto = True

    crm_team_id = fields.Many2one(
        'crm.team', string='Sales Team', group_expand='_read_group_crm_team_id',
        default=False,  # TDE: temporary fix to activate depending computed fields
        check_company=True, index=True, ondelete="cascade", required=True)
    user_id = fields.Many2one(
        'res.users', string='Salesperson',  # TDE FIXME check responsible field
        check_company=True, index=True, ondelete='cascade', required=True,
        domain="[('share', '=', False), ('id', 'not in', user_in_teams_ids), ('company_ids', 'in', user_company_ids)]")
    user_in_teams_ids = fields.Many2many(
        'res.users', compute='_compute_user_in_teams_ids',
        help='UX: Give users not to add in the currently chosen team to avoid duplicates')
    user_company_ids = fields.Many2many(
        'res.company', compute='_compute_user_company_ids',
        help='UX: Limit to team company or all if no company')
    active = fields.Boolean(string='Active', default=True)
    is_membership_multi = fields.Boolean(
        'Multiple Memberships Allowed', compute='_compute_is_membership_multi',
        help='If True, users may belong to several sales teams. Otherwise membership is limited to a single sales team.')
    member_warning = fields.Text(compute='_compute_member_warning')
    # salesman information
    image_1920 = fields.Image("Image", related="user_id.image_1920", max_width=1920, max_height=1920)
    image_128 = fields.Image("Image (128)", related="user_id.image_128", max_width=128, max_height=128)
    name = fields.Char(string='Name', related='user_id.display_name', readonly=False)
    email = fields.Char(string='Email', related='user_id.email')
    phone = fields.Char(string='Phone', related='user_id.phone')
    mobile = fields.Char(string='Mobile', related='user_id.mobile')
    company_id = fields.Many2one('res.company', string='Company', related='user_id.company_id')

    @api.constrains('crm_team_id', 'user_id', 'active')
    def _constrains_membership(self):
        # In mono membership mode: check crm_team_id / user_id is unique for active
        # memberships. Inactive memberships can create duplicate pairs which is whyy
        # we don't use a SQL constraint. Include "self" in search in case we use create
        # multi with duplicated user / team pairs in it. Use an explicit active leaf
        # in domain as we may have an active_test in context that would break computation
        existing = self.env['crm.team.member'].search([
            ('crm_team_id', 'in', self.crm_team_id.ids),
            ('user_id', 'in', self.user_id.ids),
            ('active', '=', True)
        ])
        duplicates = self.env['crm.team.member']

        active_records = dict(
            (membership.user_id.id, membership.crm_team_id.id)
            for membership in self if membership.active
        )
        for membership in self:
            potential = existing.filtered(lambda m: m.user_id == membership.user_id and \
                m.crm_team_id == membership.crm_team_id and m.id != membership.id
            )
            if not potential or len(potential) > 1:
                duplicates += potential
                continue
            if active_records.get(potential.user_id.id):
                duplicates += potential
            else:
                active_records[potential.user_id.id] = potential.crm_team_id.id

        if duplicates:
            raise exceptions.ValidationError(
                _("You are trying to create duplicate membership(s). We found that %(duplicates)s already exist(s).",
                  duplicates=", ".join("%s (%s)" % (m.user_id.name, m.crm_team_id.name) for m in duplicates)
                 ))

    @api.depends('crm_team_id', 'is_membership_multi', 'user_id')
    @api.depends_context('default_crm_team_id')
    def _compute_user_in_teams_ids(self):
        """ Give users not to add in the currently chosen team to avoid duplicates.
        In multi membership mode this field is empty as duplicates are allowed. """
        if all(m.is_membership_multi for m in self):
            member_user_ids = self.env['res.users']
        elif self.ids:
            member_user_ids = self.env['crm.team.member'].search([('id', 'not in', self.ids)]).user_id
        else:
            member_user_ids = self.env['crm.team.member'].search([]).user_id
        for member in self:
            if member_user_ids:
                member.user_in_teams_ids = member_user_ids
            elif member.crm_team_id:
                member.user_in_teams_ids = member.crm_team_id.member_ids
            elif self.env.context.get('default_crm_team_id'):
                member.user_in_teams_ids = self.env['crm.team'].browse(self.env.context['default_crm_team_id']).member_ids
            else:
                member.user_in_teams_ids = self.env['res.users']

    @api.depends('crm_team_id')
    def _compute_user_company_ids(self):
        all_companies = self.env['res.company'].search([])
        for member in self:
            member.user_company_ids = member.crm_team_id.company_id or all_companies

    @api.depends('crm_team_id')
    def _compute_is_membership_multi(self):
        multi_enabled = self.env['ir.config_parameter'].sudo().get_param('sales_team.membership_multi', False)
        self.is_membership_multi = multi_enabled

    @api.depends('is_membership_multi', 'active', 'user_id', 'crm_team_id')
    def _compute_member_warning(self):
        """ Display a warning message to warn user they are about to archive
        other memberships. Only valid in mono-membership mode and take into
        account only active memberships as we may keep several archived
        memberships. """
        if all(m.is_membership_multi for m in self):
            self.member_warning = False
        else:
            active = self.filtered('active')
            (self - active).member_warning = False
            if not active:
                return
            existing = self.env['crm.team.member'].search([('user_id', 'in', active.user_id.ids)])
            user_mapping = dict.fromkeys(existing.user_id, self.env['crm.team'])
            for membership in existing:
                user_mapping[membership.user_id] |= membership.crm_team_id

            for member in active:
                teams = user_mapping.get(member.user_id, self.env['crm.team'])
                remaining = teams - (member.crm_team_id | member._origin.crm_team_id)
                if remaining:
                    member.member_warning = _("Adding %(user_name)s in this team would remove him/her from its current teams %(team_names)s.",
                                              user_name=member.user_id.name,
                                              team_names=", ".join(remaining.mapped('name'))
                                             )
                else:
                    member.member_warning = False

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    @api.model_create_multi
    def create(self, values_list):
        """ Specific behavior implemented on create

          * mono membership mode: other user memberships are automatically
            archived (a warning already told it in form view);
          * creating a membership already existing as archived: do nothing as
            people can manage them from specific menu "Members";

        Also remove autofollow on create. No need to follow team members
        when creating them as chatter is mainly used for information purpose
        (tracked fields).
        """
        is_membership_multi = self.env['ir.config_parameter'].sudo().get_param('sales_team.membership_multi', False)
        if not is_membership_multi:
            self._synchronize_memberships(values_list)
        return super(CrmTeamMember, self.with_context(
            mail_create_nosubscribe=True
        )).create(values_list)

    def write(self, values):
        """ Specific behavior about active. If you change user_id / team_id user
        get warnings in form view and a raise in constraint check. We support
        archive / activation of memberships that toggles other memberships. But
        we do not support manual creation or update of user_id / team_id. This
        either works, either crashes). Indeed supporting it would lead to complex
        code with low added value. Users should create or remove members, and
        maybe archive / activate them. Updating manually memberships by
        modifying user_id or team_id is advanced and does not benefit from our
        support. """
        is_membership_multi = self.env['ir.config_parameter'].sudo().get_param('sales_team.membership_multi', False)
        if not is_membership_multi and values.get('active'):
            self._synchronize_memberships([
                dict(user_id=membership.user_id.id, crm_team_id=membership.crm_team_id.id)
                for membership in self
            ])
        return super(CrmTeamMember, self).write(values)

    @api.model
    def _read_group_crm_team_id(self, teams, domain, order):
        """Read group customization in order to display all the teams in
        Kanban view, even if they are empty.
        """
        return self.env['crm.team'].search([], order=order)

    def _synchronize_memberships(self, user_team_ids):
        """ Synchronize memberships: archive other memberships.

        :param user_team_ids: list of pairs (user_id, crm_team_id)
        """
        existing = self.search([
            ('active', '=', True),  # explicit search on active only, whatever context
            ('user_id', 'in', [values['user_id'] for values in user_team_ids])
        ])
        user_memberships = dict.fromkeys(existing.user_id.ids, self.env['crm.team.member'])
        for membership in existing:
            user_memberships[membership.user_id.id] += membership

        existing_to_archive = self.env['crm.team.member']
        for values in user_team_ids:
            existing_to_archive += user_memberships.get(values['user_id'], self.env['crm.team.member']).filtered(
                lambda m: m.crm_team_id.id != values['crm_team_id']
            )

        if existing_to_archive:
            existing_to_archive.action_archive()

        return existing_to_archive

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

from odoo import api, fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    crm_team_ids = fields.Many2many(
        'crm.team', 'crm_team_member', 'user_id', 'crm_team_id', string='Sales Teams',
        check_company=True, copy=False, readonly=True,
        compute='_compute_crm_team_ids', search='_search_crm_team_ids')
    crm_team_member_ids = fields.One2many('crm.team.member', 'user_id', string='Sales Team Members')
    sale_team_id = fields.Many2one(
        'crm.team', string='User Sales Team', compute='_compute_sale_team_id',
        readonly=True, store=True,
        help="Main user sales team. Used notably for pipeline, or to set sales team in invoicing or subscription.")

    @api.depends('crm_team_member_ids.active')
    def _compute_crm_team_ids(self):
        for user in self:
            user.crm_team_ids = user.crm_team_member_ids.crm_team_id

    def _search_crm_team_ids(self, operator, value):
        return [('crm_team_member_ids.crm_team_id', operator, value)]

    @api.depends('crm_team_member_ids.crm_team_id', 'crm_team_member_ids.create_date', 'crm_team_member_ids.active')
    def _compute_sale_team_id(self):
        for user in self:
            if not user.crm_team_member_ids.ids:
                user.sale_team_id = False
            else:
                sorted_memberships = user.crm_team_member_ids  # sorted by create date
                user.sale_team_id = sorted_memberships[0].crm_team_id if sorted_memberships else False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_team_member
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
access_crm_team_member_all,access.crm.team.member.all,model_crm_team_member,,0,0,0,0
access_crm_team_member_user,access.crm.team.member.user,model_crm_team_member,base.group_user,1,0,0,0
access_crm_team_member_manager,access.crm.team.member.manager,model_crm_team_member,sales_team.group_sale_manager,1,1,1,1
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
            <field name="implied_ids" eval="[(4, ref('group_sale_salesman_all_leads')),
                (4, ref('mail.group_mail_template_editor'))]"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
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
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Services"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="color" required="True" widget="color_picker"/>
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
            <tree string="Tags" editable="bottom" sample="1">
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
            Create CRM Tags
            </p><p>
            Use Tags to manage and track your Opportunities (product structure, sales type, ...)
            </p>
        </field>
    </record>
</odoo>

```

## File: views\crm_team_member_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="crm_team_member_view_search" model="ir.ui.view">
        <field name="name">crm.team.member.view.search</field>
        <field name="model">crm.team.member</field>
        <field name="arch" type="xml">
            <search string="Sales Person">
                <field name="user_id"/>
                <field name="crm_team_id"/>
                <separator/>
                <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Sales Team" name="groupby_crm_team_id" context="{'group_by': 'crm_team_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="crm_team_member_view_tree" model="ir.ui.view">
        <field name="name">crm.team.member.view.tree</field>
        <field name="model">crm.team.member</field>
        <field name="arch" type="xml">
            <tree string="Sales Men" sample="1">
                <field name="crm_team_id"/>
                <field name="user_id"/>
            </tree>
        </field>
    </record>

    <record id="crm_team_member_view_tree_from_team" model="ir.ui.view">
        <field name="name">crm.team.member.view.tree.from.team</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_tree"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='crm_team_id']" position="replace">
            </xpath>
        </field>
    </record>

    <record id="crm_team_member_view_kanban" model="ir.ui.view">
        <field name="name">crm.team.member.view.kanban</field>
        <field name="model">crm.team.member</field>
        <field name="arch" type="xml">
            <kanban quick_create="false"
                group_create="0"
                records_draggable="0"
                sample="1"
                default_group_by="crm_team_id"
                class="o_crm_team_member_kanban">
                <field name="user_id"/>
                <field name="active"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="ribbon ribbon-top-right" invisible="active">
                                <span class="text-bg-danger">Archived</span>
                            </div>
                            <div class="o_kanban_card_content d-flex">
                                <div>
                                    <img t-att-src="kanban_image('res.users', 'avatar_128', record.user_id.raw_value)" class="o_kanban_image o_image_64_cover" alt="Avatar"/>
                                </div>
                                <div class="oe_kanban_details d-flex flex-column ms-3">
                                    <strong class="o_kanban_record_title oe_partner_heading"><field name="user_id"/></strong>
                                    <a type="open" class="nav-link p-0"><field name="crm_team_id"/></a>
                                    <div class="d-flex align-items-baseline text-break">
                                        <i class="fa fa-envelope me-1" role="img" aria-label="Email" title="Email"/><field name="email"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="crm_team_member_view_kanban_from_team" model="ir.ui.view">
        <field name="name">crm.team.member.view.kanban.from.team</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_kanban"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='crm_team_id']" position="replace">
            </xpath>
        </field>
    </record>

    <record id="crm_team_member_view_form" model="ir.ui.view">
        <field name="name">crm.team.member.view.form</field>
        <field name="model">crm.team.member</field>
        <field name="arch" type="xml">
            <form string="Sales Men">
                <field name="active" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="is_membership_multi" invisible="1"/>
                <field name="member_warning" invisible="1"/>
                <field name="user_in_teams_ids" invisible="1"/>
                <field name="user_company_ids" invisible="1"/>
                <sheet>
                    <div class="alert alert-info text-center" role="alert"
                        invisible="not member_warning">
                        <field name="member_warning"/>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="image_1920" widget='image' class="oe_avatar"
                        invisible="not user_id"
                        options='{"preview_image": "image_128"}'/>
                    <div class="oe_title">
                        <label for="user_id" class="oe_edit_only"/>
                        <h1 class="d-flex">
                            <field name="user_id" class="flex-grow-1"
                                options="{'no_quick_create': True}"/>
                        </h1>
                    </div>
                    <group name="member_partner_info">
                        <group name="group_owner">
                            <field name="crm_team_id" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
                        </group>
                        <group class="ps-0" name="group_user">
                            <field name="company_id" invisible="not user_id"
                                groups="base.group_multi_company"/>
                            <field name="email" invisible="not user_id"/>
                            <field name="mobile" invisible="not user_id"/>
                            <field name="phone" invisible="not user_id"/>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="crm_team_member_view_form_from_team" model="ir.ui.view">
        <field name="name">crm.team.member.view.form.from.team</field>
        <field name="model">crm.team.member</field>
        <field name="inherit_id" ref="sales_team.crm_team_member_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='group_owner']" position="replace"/>
        </field>
    </record>

    <record id="crm_team_member_action" model="ir.actions.act_window">
        <field name="name">Team Members</field>
        <field name="res_model">crm.team.member</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new salesman
            </p><p>
                Link salespersons to sales teams.
            </p>
        </field>
    </record>

</data></odoo>

```

## File: views\crm_team_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="crm_team_view_search" model="ir.ui.view">
        <field name="name">crm.team.view.search</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <search string="Salesteams Search">
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <field name="name"/>
                <field name="user_id"/>
                <field name="member_ids"/>
                <group expand="0" string="Group By...">
                    <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Team Leader" name="team_leader" domain="[]" context="{'group_by': 'user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="crm_team_view_form" model="ir.ui.view">
        <field name="name">crm.team.form</field>
        <field name="model">crm.team</field>
        <field name="arch" type="xml">
            <form string="Sales Team" class="o_crm_team_form_view">
                <div class="alert alert-info text-center" role="alert"
                    invisible="is_membership_multi or not member_warning">
                    <field name="member_warning"/>
                </div>
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <label for="name" string="Sales Team"/>
                        <h1>
                            <field class="text-break" name="name" placeholder="e.g. North America"/>
                        </h1>
                        <div name="options_active" class="o_row"/>
                    </div>
                    <group>
                        <group name="left" string="Team Details">
                            <field name="active" invisible="1"/>
                            <field name="sequence" invisible="1"/>
                            <field name="is_membership_multi" invisible="1"/>
                            <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
                            <field name="company_id" options="{'no_create': True}" groups="base.group_multi_company"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="member_company_ids" invisible="1"/>
                        </group>
                        <group name="right">
                        </group>
                    </group>
                    <notebook>
                        <page string="Members" name="members_users">
                            <field name="member_ids" mode="kanban"
                                class="w-100">
                                <kanban>
                                    <field name="id"/>
                                    <field name="name"/>
                                    <field name="email"/>
                                    <field name="avatar_128"/>
                                    <templates>
                                        <t t-name="kanban-box">
                                            <div class="oe_kanban_card oe_kanban_global_click">
                                                <div class="o_kanban_card_content d-flex">
                                                    <div>
                                                        <img t-att-src="kanban_image('res.users', 'avatar_128', record.id.raw_value)"
                                                            class="o_kanban_image o_image_64_cover" alt="Avatar"/>
                                                    </div>
                                                    <div class="oe_kanban_details d-flex flex-column ms-3">
                                                        <strong class="o_kanban_record_title oe_partner_heading"><field name="name"/></strong>
                                                        <div class="d-flex align-items-baseline text-break">
                                                            <i class="fa fa-envelope me-1" role="img" aria-label="Email" title="Email"/><field name="email"/>
                                                        </div>
                                                    </div>
                                                </div>
                                            </div>
                                        </t>
                                    </templates>
                                </kanban>
                            </field>
                            <field name="crm_team_member_ids" mode="kanban"
                                class="w-100"
                                invisible="is_membership_multi or not is_membership_multi"
                                context="{
                                    'kanban_view_ref': 'sales_team.crm_team_member_view_kanban_from_team',
                                    'form_view_ref': 'sales_team.crm_team_member_view_form_from_team',
                                    'tree_view_ref': 'sales_team.crm_team_member_view_tree_from_team',
                                    'default_crm_team_id': id,
                                }"/>
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
        <field name="arch" type="xml">
            <tree string="Sales Team" sample="1" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="name" readonly="1"/>
                <field name="active" column_invisible="True"/>
                <field name="user_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="crm_team_view_kanban" model="ir.ui.view">
        <field name="name">crm.team.view.kanban</field>
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
                                    <span class="float-end"><field name="user_id"/></span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <!-- Case Teams Salesteams dashboard view -->
   <record id="crm_team_view_kanban_dashboard" model="ir.ui.view" >
        <field name="name">crm.team.view.kanban.dashboard</field>
        <field name="model">crm.team</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_dashboard o_crm_team_kanban" create="0" sample="1">
                <field name="name"/>
                <field name="user_id"/>
                <field name="member_ids"/>
                <field name="color"/>
                <field name="currency_id"/>
                <field name="is_favorite"/>
                <templates>
                    <t t-name="kanban-menu">
                        <div class="container">
                            <div class="row">
                                <div class="col-4 o_kanban_card_manage_section o_kanban_manage_view">
                                    <h5 role="menuitem" class=" o_kanban_card_manage_title">
                                        <span>View</span>
                                    </h5>
                                </div>
                                <div class="col-4 o_kanban_card_manage_section o_kanban_manage_new">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>New</span>
                                    </h5>
                                </div>
                                <div class="col-4 o_kanban_card_manage_section o_kanban_manage_reports">
                                    <h5 role="menuitem" class="o_kanban_card_manage_title">
                                        <span>Reporting</span>
                                    </h5>
                                    <div name="o_team_kanban_report_separator"></div>
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
                    </t>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''}">
                            <div t-attf-class="o_kanban_card_header">
                                <div class="o_kanban_card_header_title">
                                    <div class="o_primary o_text_overflow"><field name="name"/></div>
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
    <record id="crm_team_action_sales" model="ir.actions.act_window">
        <field name="name">Sales Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">kanban,form</field>
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

      <record id="crm_team_action_pipeline" model="ir.actions.act_window">
        <field name="name">Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">kanban,form</field>
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

    <record id="crm_team_action_config" model="ir.actions.act_window">
        <field name="name">Sales Teams</field>
        <field name="res_model">crm.team</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Sales Team
            </p><p>
                Use Sales Teams to organize your sales departments and draw up reports.
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
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'res.partner')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create an Activity Type
            </p><p>
                Those represent the different categories of things you have to do (e.g. "Call" or "Prepare meeting").
            </p>
        </field>
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
                <field name="team_id" invisible="1"/>
                <field name="team_id" groups="base.group_no_one" context="{'kanban_view_ref': 'sales_team.crm_team_view_kanban'}"/>
            </xpath>
            <field name="parent_id" position="attributes">
                <attribute name="context">{'default_is_company': True, 'show_vat': True, 'default_user_id': user_id, 'default_team_id': team_id}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

