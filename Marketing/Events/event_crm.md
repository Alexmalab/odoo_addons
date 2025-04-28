# Odoo Module: event_crm

Category: Marketing/Events

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
    'name': 'Event CRM',
    'version': '1.0',
    'category': 'Marketing/Events',
    'website': 'https://www.odoo.com/app/events',
    'description': "Create leads from event registrations.",
    'depends': ['event', 'crm'],
    'data': [
        'security/event_crm_security.xml',
        'security/ir.model.access.csv',
        'data/crm_lead_merge_template.xml',
        'views/crm_lead_views.xml',
        'views/event_registration_views.xml',
        'views/event_lead_rule_views.xml',
        'views/event_event_views.xml',
    ],
    'demo': [
        'data/event_crm_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\crm_lead_merge_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="crm_lead_merge_summary_inherit_event_crm" inherit_id="crm.crm_lead_merge_summary">
    <xpath expr="//div[@name='marketing']" position="after">
        <div class="mt-3 mb-3" name="event" t-if="lead.event_lead_rule_id or lead.event_id">
            <div class="fw-bold">
                Event:
            </div>
            <div t-if="lead.event_id">
                Event: <span t-field="lead.event_id"/>
            </div>
            <div t-if="lead.event_lead_rule_id">
                Registration Rule: <span t-field="lead.event_lead_rule_id"/>
            </div>
        </div>
    </xpath>
</template>

</odoo>

```

## File: data\event_crm_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Event CRM Rule 0 -->
        <record id="event_lead_rule_0" model="event.lead.rule">
            <field name="name">Rule on @example.com</field>
            <field name="event_id" ref="event.event_5"/>
            <field name="event_registration_filter">[('email','ilike','@example.com')]</field>
            <field name="lead_user_id" ref="base.user_admin"/>
            <field name="lead_tag_ids" eval="[(6, 0, [ref('sales_team.categ_oppor8')])]"/>
        </record>

        <record id="event_registration_0_rule_0" model="event.registration">
            <field name="name">Barney Lonny</field>
            <field name="email">barney.lonny@example.com</field>
            <field name="phone">+1 202 555 0122</field>
            <field name="event_id" ref="event.event_5"/>
        </record>
        <record id="event_registration_1_rule_0" model="event.registration">
            <field name="name">Tom Harper</field>
            <field name="email">tom.harper@example.com</field>
            <field name="phone">+1 202 555 0161</field>
            <field name="event_id" ref="event.event_5"/>
        </record>

    </data>
</odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class Lead(models.Model):
    _inherit = 'crm.lead'

    event_lead_rule_id = fields.Many2one('event.lead.rule', string="Registration Rule", help="Rule that created this lead")
    event_id = fields.Many2one('event.event', string="Source Event", help="Event triggering the rule that created this lead")
    registration_ids = fields.Many2many(
        'event.registration', string="Source Registrations",
        groups='event.group_event_registration_desk',
        help="Registrations triggering the rule that created this lead")
    registration_count = fields.Integer(
        string="# Registrations", compute='_compute_registration_count',
        groups='event.group_event_registration_desk',
        help="Counter for the registrations linked to this lead")

    @api.depends('registration_ids')
    def _compute_registration_count(self):
        for record in self:
            record.registration_count = len(record.registration_ids)

    def _merge_dependences(self, opportunities):
        super(Lead, self)._merge_dependences(opportunities)

        # merge registrations as sudo, as crm people may not have access to event rights
        self.sudo().write({
            'registration_ids': [(4, registration.id) for registration in opportunities.sudo().registration_ids]
        })

    def _merge_get_fields(self):
        return super(Lead, self)._merge_get_fields() + ['event_lead_rule_id', 'event_id']

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class EventEvent(models.Model):
    _name = "event.event"
    _inherit = "event.event"

    lead_ids = fields.One2many(
        'crm.lead', 'event_id', string="Leads", groups='sales_team.group_sale_salesman',
        help="Leads generated from this event")
    lead_count = fields.Integer(
        string="# Leads", compute='_compute_lead_count', groups='sales_team.group_sale_salesman')

    @api.depends('lead_ids')
    def _compute_lead_count(self):
        lead_data = self.env['crm.lead']._read_group(
            [('event_id', 'in', self.ids)],
            ['event_id'], ['event_id']
        )
        mapped_data = {item['event_id'][0]: item['event_id_count'] for item in lead_data}
        for event in self:
            event.lead_count = mapped_data.get(event.id, 0)

```

## File: models\event_lead_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from collections import defaultdict

from odoo import fields, models, _


class EventLeadRule(models.Model):
    """ Rule model for creating / updating leads from event registrations.

    SPECIFICATIONS: CREATION TYPE

    There are two types of lead creation:

      * per attendee: create a lead for each registration;
      * per order: create a lead for a group of registrations;

    The last one is only available through interface if it is possible to register
    a group of attendees in one action (when event_sale or website_event are
    installed). Behavior itself is implemented directly in event_crm.

    Basically a group is either a list of registrations belonging to the same
    event and created in batch (website_event flow). With event_sale this
    definition will be improved to be based on sale_order.

    SPECIFICATIONS: CREATION TRIGGERS

    There are three options to trigger lead creation. We consider basically that
    lead quality increases if attendees confirmed or went to the event. Triggers
    allow therefore to run rules:

      * at attendee creation;
      * at attendee confirmation;
      * at attendee venue;

    This trigger defines when the rule will run.

    SPECIFICATIONS: FILTERING REGISTRATIONS

    When a batch of registrations matches the rule trigger we filter them based
    on conditions and rules defines on event_lead_rule model. Heuristic is the
    following:

      * the rule is active;
      * if a filter is set: filter registrations based on this filter. This is
        done like a search, and filter is a domain;
      * if a company is set on the rule, it must match event's company. Note
        that multi-company rules apply on event_lead_rule;
      * if an event category it set, it must match;
      * if an event is set, it must match;
      * if both event and category are set, one of them must match (OR). If none
        of those are set, it is considered as OK;

    If conditions are met, leads are created with pre-filled informations defined
    on the rule (type, user_id, team_id). Contact information coming from the
    registrations are computed (customer, name, email, phone, mobile, contact_name).

    SPECIFICATIONS: OTHER POINTS

    Note that all rules matching their conditions are applied. This means more
    than one lead can be created depending on the configuration. This is
    intended in order to give more freedom to the user using the automatic
    lead generation.
    """
    _name = "event.lead.rule"
    _description = "Event Lead Rules"

    # Definition
    name = fields.Char('Rule Name', required=True, translate=True)
    active = fields.Boolean('Active', default=True)
    lead_ids = fields.One2many(
        'crm.lead', 'event_lead_rule_id', string='Created Leads',
        groups='sales_team.group_sale_salesman')
    # Triggers
    lead_creation_basis = fields.Selection([
        ('attendee', 'Per Attendee'), ('order', 'Per Order')],
        string='Create', default='attendee', required=True,
        help='Per Attendee : A Lead is created for each Attendee (B2C).\n'
             'Per Order : A single Lead is created per Ticket Batch/Sale Order (B2B)')
    lead_creation_trigger = fields.Selection([
        ('create', 'Attendees are created'),
        ('confirm', 'Attendees are confirmed'),
        ('done', 'Attendees attended')],
        string='When', default='create', required=True,
        help='Creation: at attendee creation;\n'
             'Confirmation: when attendee is confirmed, manually or automatically;\n'
             'Attended: when attendance is confirmed and registration set to done;')
    # Filters
    event_type_ids = fields.Many2many(
        'event.type', string='Event Categories',
        help='Filter the attendees to include those of this specific event category. If not set, no event category restriction will be applied.')
    event_id = fields.Many2one(
        'event.event', string='Event',
        domain="[('company_id', 'in', [company_id or current_company_id, False])]",
        help='Filter the attendees to include those of this specific event. If not set, no event restriction will be applied.')
    company_id = fields.Many2one(
        'res.company', string='Company',
        help="Restrict the trigger of this rule to events belonging to a specific company.\nIf not set, no company restriction will be applied.")
    event_registration_filter = fields.Text(string="Registrations Domain", help="Filter the attendees that will or not generate leads.")
    # Lead default_value fields
    lead_type = fields.Selection([
        ('lead', 'Lead'), ('opportunity', 'Opportunity')], string="Lead Type", required=True,
        default=lambda self: 'lead' if self.env['res.users'].has_group('crm.group_use_lead') else 'opportunity',
        help="Default lead type when this rule is applied.")
    lead_sales_team_id = fields.Many2one(
        'crm.team', string='Sales Team', ondelete="set null",
        help="Automatically assign the created leads to this Sales Team.")
    lead_user_id = fields.Many2one('res.users', string='Salesperson', help="Automatically assign the created leads to this Salesperson.")
    lead_tag_ids = fields.Many2many('crm.tag', string='Tags', help="Automatically add these tags to the created leads.")

    def _run_on_registrations(self, registrations):
        """ Create or update leads based on rule configuration. Two main lead
        management type exists

          * per attendee: each registration creates a lead;
          * per order: registrations are grouped per group and one lead is created
            or updated with the batch (used mainly with sale order configuration
            in event_sale);

        Heuristic

          * first, check existing lead linked to registrations to ensure no
            duplication. Indeed for example attendee status change may trigger
            the same rule several times;
          * then for each rule, get the subset of registrations matching its
            filters;
          * then for each order-based rule, get the grouping information. This
            give a list of registrations by group (event, sale_order), with maybe
            an already-existing lead to update instead of creating a new one;
          * finally apply rules. Attendee-based rules create a lead for each
            attendee, group-based rules use the grouping information to create
            or update leads;

        :param registrations: event.registration recordset on which rules given by
          self have to run. Triggers should already be checked, only filters are
          applied here.

        :return leads: newly-created leads. Updated leads are not returned.
        """
        # order by ID, ensure first created wins
        registrations = registrations.sorted('id')

        # first: ensure no duplicate by searching existing registrations / rule
        existing_leads = self.env['crm.lead'].search([
            ('registration_ids', 'in', registrations.ids),
            ('event_lead_rule_id', 'in', self.ids)
        ])
        rule_to_existing_regs = defaultdict(lambda: self.env['event.registration'])
        for lead in existing_leads:
            rule_to_existing_regs[lead.event_lead_rule_id] += lead.registration_ids

        # second: check registrations matching rules (in batch)
        new_registrations = self.env['event.registration']
        rule_to_new_regs = dict()
        for rule in self:
            new_for_rule = registrations.filtered(lambda reg: reg not in rule_to_existing_regs[rule])
            rule_registrations = rule._filter_registrations(new_for_rule)
            new_registrations |= rule_registrations
            rule_to_new_regs[rule] = rule_registrations
        new_registrations.sorted('id')  # as an OR was used, re-ensure order

        # third: check grouping
        order_based_rules = self.filtered(lambda rule: rule.lead_creation_basis == 'order')
        rule_group_info = new_registrations._get_lead_grouping(order_based_rules, rule_to_new_regs)

        lead_vals_list = []
        for rule in self:
            if rule.lead_creation_basis == 'attendee':
                matching_registrations = rule_to_new_regs[rule].sorted('id')
                for registration in matching_registrations:
                    lead_vals_list.append(registration._get_lead_values(rule))
            else:
                # check if registrations are part of a group, for example a sale order, to know if we update or create leads
                for (toupdate_leads, group_key, group_registrations) in rule_group_info[rule]:
                    if toupdate_leads:
                        additionnal_description = group_registrations._get_lead_description(_("New registrations"), line_counter=True)
                        for lead in toupdate_leads:
                            lead.write({
                                'description': "%s<br/>%s" % (lead.description, additionnal_description),
                                'registration_ids': [(4, reg.id) for reg in group_registrations],
                            })
                    elif group_registrations:
                        lead_vals_list.append(group_registrations._get_lead_values(rule))

        return self.env['crm.lead'].create(lead_vals_list)

    def _filter_registrations(self, registrations):
        """ Keep registrations matching rule conditions. Those are

          * if a filter is set: filter registrations based on this filter. This is
            done like a search, and filter is a domain;
          * if a company is set on the rule, it must match event's company. Note
            that multi-company rules apply on event_lead_rule;
          * if an event category it set, it must match;
          * if an event is set, it must match;
          * if both event and category are set, one of them must match (OR). If none
            of those are set, it is considered as OK;

        :param registrations: event.registration recordset on which rule filters
          will be evaluated;
        :return: subset of registrations matching rules
        """
        self.ensure_one()
        if self.event_registration_filter and self.event_registration_filter != '[]':
            registrations = registrations.filtered_domain(literal_eval(self.event_registration_filter))

        # check from direct m2o to linked m2o / o2m to filter first without inner search
        company_ok = lambda registration: registration.company_id == self.company_id if self.company_id else True
        event_or_event_type_ok = \
            lambda registration: \
                registration.event_id == self.event_id or registration.event_id.event_type_id in self.event_type_ids \
                if (self.event_id or self.event_type_ids) else True

        return registrations.filtered(lambda r: company_ok(r) and event_or_event_type_ok(r))

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from markupsafe import Markup

from odoo import api, fields, models, tools, _
from odoo.addons.phone_validation.tools import phone_validation


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    lead_ids = fields.Many2many(
        'crm.lead', string='Leads', copy=False, readonly=True,
        groups='sales_team.group_sale_salesman')
    lead_count = fields.Integer(
        '# Leads', compute='_compute_lead_count', compute_sudo=True)

    @api.depends('lead_ids')
    def _compute_lead_count(self):
        for record in self:
            record.lead_count = len(record.lead_ids)

    @api.model_create_multi
    def create(self, vals_list):
        """ Trigger rules based on registration creation, and check state for
        rules based on confirmed / done attendees. """
        registrations = super(EventRegistration, self).create(vals_list)

        # handle triggers based on creation, then those based on confirm and done
        # as registrations can be automatically confirmed, or even created directly
        # with a state given in values
        if not self.env.context.get('event_lead_rule_skip'):
            self.env['event.lead.rule'].search([('lead_creation_trigger', '=', 'create')]).sudo()._run_on_registrations(registrations)
            open_registrations = registrations.filtered(lambda reg: reg.state == 'open')
            if open_registrations:
                self.env['event.lead.rule'].search([('lead_creation_trigger', '=', 'confirm')]).sudo()._run_on_registrations(open_registrations)
            done_registrations = registrations.filtered(lambda reg: reg.state == 'done')
            if done_registrations:
                self.env['event.lead.rule'].search([('lead_creation_trigger', '=', 'done')]).sudo()._run_on_registrations(done_registrations)

        return registrations

    def write(self, vals):
        """ Update the lead values depending on fields updated in registrations.
        There are 2 main use cases

          * first is when we update the partner_id of multiple registrations. It
            happens when a public user fill its information when they register to
            an event;
          * second is when we update specific values of one registration like
            updating question answers or a contact information (email, phone);

        Also trigger rules based on confirmed and done attendees (state written
        to open and done).
        """
        to_update, event_lead_rule_skip = False, self.env.context.get('event_lead_rule_skip')
        if not event_lead_rule_skip:
            to_update = self.filtered(lambda reg: reg.lead_count)
        if to_update:
            lead_tracked_vals = to_update._get_lead_tracked_values()

        res = super(EventRegistration, self).write(vals)

        if not event_lead_rule_skip and to_update:
            self.env.flush_all()  # compute notably partner-based fields if necessary
            to_update.sudo()._update_leads(vals, lead_tracked_vals)

        # handle triggers based on state
        if not event_lead_rule_skip:
            if vals.get('state') == 'open':
                self.env['event.lead.rule'].search([('lead_creation_trigger', '=', 'confirm')]).sudo()._run_on_registrations(self)
            elif vals.get('state') == 'done':
                self.env['event.lead.rule'].search([('lead_creation_trigger', '=', 'done')]).sudo()._run_on_registrations(self)

        return res

    def _load_records_create(self, values):
        """ In import mode: do not run rules those are intended to run when customers
        buy tickets, not when bootstrapping a database. """
        return super(EventRegistration, self.with_context(event_lead_rule_skip=True))._load_records_create(values)

    def _load_records_write(self, values):
        """ In import mode: do not run rules those are intended to run when customers
        buy tickets, not when bootstrapping a database. """
        return super(EventRegistration, self.with_context(event_lead_rule_skip=True))._load_records_write(values)

    def _update_leads(self, new_vals, lead_tracked_vals):
        """ Update leads linked to some registrations. Update is based depending
        on updated fields, see ``_get_lead_contact_fields()`` and ``_get_lead_
        description_fields()``. Main heuristic is

          * check attendee-based leads, for each registration recompute contact
            information if necessary (changing partner triggers the whole contact
            computation); update description if necessary;
          * check order-based leads, for each existing group-based lead, only
            partner change triggers a contact and description update. We consider
            that group-based rule works mainly with the main contact and less
            with further details of registrations. Those can be found in stat
            button if necessary.

        :param new_vals: values given to write. Used to determine updated fields;
        :param lead_tracked_vals: dict(registration_id, registration previous values)
          based on new_vals;
        """
        for registration in self:
            leads_attendee = registration.lead_ids.filtered(
                lambda lead: lead.event_lead_rule_id.lead_creation_basis == 'attendee'
            )
            if not leads_attendee:
                continue

            old_vals = lead_tracked_vals[registration.id]
            # if partner has been updated -> update registration contact information
            # as they are computed (and therefore not given to write values)
            if 'partner_id' in new_vals:
                new_vals.update(**dict(
                    (field, registration[field])
                    for field in self._get_lead_contact_fields()
                    if field != 'partner_id')
                )

            lead_values = {}
            # update contact fields: valid for all leads of registration
            upd_contact_fields = [field for field in self._get_lead_contact_fields() if field in new_vals.keys()]
            if any(new_vals[field] != old_vals[field] for field in upd_contact_fields):
                lead_values = registration._get_lead_contact_values()

            # update description fields: each lead has to be updated, otherwise
            # update in batch
            upd_description_fields = [field for field in self._get_lead_description_fields() if field in new_vals.keys()]
            if any(new_vals[field] != old_vals[field] for field in upd_description_fields):
                for lead in leads_attendee:
                    lead_values['description'] = "%s<br/>%s" % (
                        lead.description,
                        registration._get_lead_description(_("Updated registrations"), line_counter=True)
                    )
                    lead.write(lead_values)
            elif lead_values:
                leads_attendee.write(lead_values)

        leads_order = self.lead_ids.filtered(lambda lead: lead.event_lead_rule_id.lead_creation_basis == 'order')
        for lead in leads_order:
            lead_values = {}
            if new_vals.get('partner_id'):
                lead_values.update(lead.registration_ids._get_lead_contact_values())
                if not lead.partner_id:
                    lead_values['description'] = lead.registration_ids._get_lead_description(_("Participants"), line_counter=True)
                elif new_vals['partner_id'] != lead.partner_id.id:
                    lead_values['description'] = (lead.description or '') + "<br/>" + lead.registration_ids._get_lead_description(_("Updated registrations"), line_counter=True, line_suffix=_("(updated)"))
            if lead_values:
                lead.write(lead_values)

    def _get_lead_values(self, rule):
        """ Get lead values from registrations. Self can contain multiple records
        in which case first found non void value is taken. Note that all
        registrations should belong to the same event.

        :return dict lead_values: values used for create / write on a lead
        """
        lead_values = {
            # from rule
            'type': rule.lead_type,
            'user_id': rule.lead_user_id.id,
            'team_id': rule.lead_sales_team_id.id,
            'tag_ids': rule.lead_tag_ids.ids,
            'event_lead_rule_id': rule.id,
            # event and registration
            'event_id': self.event_id.id,
            'referred': self.event_id.name,
            'registration_ids': self.ids,
            'campaign_id': self._find_first_notnull('utm_campaign_id'),
            'source_id': self._find_first_notnull('utm_source_id'),
            'medium_id': self._find_first_notnull('utm_medium_id'),
        }
        lead_values.update(self._get_lead_contact_values())
        lead_values['description'] = self._get_lead_description(_("Participants"), line_counter=True)
        return lead_values

    def _get_lead_contact_values(self):
        """ Specific management of contact values. Rule creation basis has some
        effect on contact management

          * in attendee mode: keep registration partner only if partner phone and
            email match. Indeed lead are synchronized with their contact and it
            would imply rewriting on partner, and therefore on other documents;
          * in batch mode: if a customer is found use it as main contact. Registrations
            details are included in lead description;

        :return dict: values used for create / write on a lead
        """
        valid_partner = next(
            (reg.partner_id for reg in self if reg.partner_id != self.env.ref('base.public_partner')),
            self.env['res.partner']
        )  # CHECKME: broader than just public partner

        # mono registration mode: keep partner only if email and phone matches;
        # otherwise registration > partner. Note that email format and phone
        # formatting have to taken into account in comparison
        if len(self) == 1 and valid_partner:
            # compare emails: email_normalized or raw
            if self.email and valid_partner.email:
                if valid_partner.email_normalized and tools.email_normalize(self.email) != valid_partner.email_normalized:
                    valid_partner = self.env['res.partner']
                elif not valid_partner.email_normalized and valid_partner.email != self.email:
                    valid_partner = self.env['res.partner']

            # compare phone, taking into account formatting
            if valid_partner and self.phone and valid_partner.phone:
                phone_formatted = phone_validation.phone_format(
                    self.phone,
                    valid_partner.country_id.code or None,
                    valid_partner.country_id.phone_code or None,
                    force_format='E164',
                    raise_exception=False
                )
                partner_phone_formatted = valid_partner._phone_format(valid_partner.phone)
                if phone_formatted and partner_phone_formatted and phone_formatted != partner_phone_formatted:
                    valid_partner = self.env['res.partner']
                if (not phone_formatted or not partner_phone_formatted) and self.phone != valid_partner.phone:
                    valid_partner = self.env['res.partner']

        if valid_partner:
            contact_vals = self.env['crm.lead']._prepare_values_from_partner(valid_partner)
            # force email_from / phone only if not set on partner because those fields are now synchronized automatically
            if not valid_partner.email:
                contact_vals['email_from'] = self._find_first_notnull('email')
            if not valid_partner.phone:
                contact_vals['phone'] = self._find_first_notnull('phone')
        else:
            # don't force email_from + partner_id because those fields are now synchronized automatically
            contact_vals = {
                'contact_name': self._find_first_notnull('name'),
                'email_from': self._find_first_notnull('email'),
                'phone': self._find_first_notnull('phone'),
                'lang_id': False,
            }
        contact_name = valid_partner.name or self._find_first_notnull('name') or self._find_first_notnull('email')
        contact_vals.update({
            'name': f'{self.event_id[:1].name} - {contact_name}',
            'partner_id': valid_partner.id,
            'mobile': valid_partner.mobile or self._find_first_notnull('mobile'),
        })
        return contact_vals

    def _get_lead_description(self, prefix='', line_counter=True, line_suffix=''):
        """ Build the description for the lead using a prefix for all generated
        lines. For example to enumerate participants or inform of an update in
        the information of a participant.

        :return string description: complete description for a lead taking into
          account all registrations contained in self
        """
        reg_lines = [
            registration._get_lead_description_registration(
                line_suffix=line_suffix
            ) for registration in self
        ]
        description = (prefix if prefix else '') + Markup("<br/>")
        if line_counter:
            description += Markup("<ol>") + Markup('').join(reg_lines) + Markup("</ol>")
        else:
            description += Markup("<ul>") + Markup('').join(reg_lines) + Markup("</ul>")
        return description

    def _get_lead_description_registration(self, line_suffix=''):
        """ Build the description line specific to a given registration. """
        self.ensure_one()
        return Markup("<li>") + "%s (%s)%s" % (
            self.name or self.partner_id.name or self.email,
            " - ".join(self[field] for field in ('email', 'phone') if self[field]),
            f" {line_suffix}" if line_suffix else "",
        ) + Markup("</li>")

    def _get_lead_tracked_values(self):
        """ Tracked values are based on two subset of fields to track in order
        to fill or update leads. Two main use cases are

          * description fields: registration contact fields: email, phone, ...
            on registration. Other fields are added by inheritance like
            question answers;
          * contact fields: registration contact fields + partner_id field as
            contact of a lead is managed specifically. Indeed email and phone
            synchronization of lead / partner_id implies paying attention to
            not rewrite partner values from registration values.

        Tracked values are therefore the union of those two field sets. """
        tracked_fields = list(set(self._get_lead_contact_fields()) | set(self._get_lead_description_fields()))
        return dict(
            (registration.id,
             dict((field, self._convert_value(registration[field], field)) for field in tracked_fields)
            ) for registration in self
        )

    def _get_lead_grouping(self, rules, rule_to_new_regs):
        """ Perform grouping of registrations in order to enable order-based
        lead creation and update existing groups with new registrations.

        Heuristic in event is the following. Registrations created in multi-mode
        are grouped by event. Customer use case: website_event flow creates
        several registrations in a create-multi.

        Update is not supported as there is no way to determine if a registration
        is part of an existing batch.

        :param rules: lead creation rules to run on registrations given by self;
        :param rule_to_new_regs: dict: for each rule, subset of self matching
          rule conditions. Used to speedup batch computation;

        :return dict: for each rule, rule (key of dict) gives a list of groups.
          Each group is a tuple (
            existing_lead: existing lead to update;
            group_record: record used to group;
            registrations: sub record set of self, containing registrations
                           belonging to the same group;
          )
        """
        event_to_reg_ids = defaultdict(lambda: self.env['event.registration'])
        for registration in self:
            event_to_reg_ids[registration.event_id] += registration

        return dict(
            (rule, [(False, event, (registrations & rule_to_new_regs[rule]).sorted('id'))
                    for event, registrations in event_to_reg_ids.items()])
            for rule in rules
        )

    # ------------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------------

    @api.model
    def _get_lead_contact_fields(self):
        """ Get registration fields linked to lead contact. Those are used notably
        to see if an update of lead is necessary or to fill contact values
        in ``_get_lead_contact_values())`` """
        return ['name', 'email', 'phone', 'mobile', 'partner_id']

    @api.model
    def _get_lead_description_fields(self):
        """ Get registration fields linked to lead description. Those are used
        notablyto see if an update of lead is necessary or to fill description
        in ``_get_lead_description())`` """
        return ['name', 'email', 'phone']

    def _find_first_notnull(self, field_name):
        """ Small tool to extract the first not nullvalue of a field: its value
        or the ids if this is a relational field. """
        value = next((reg[field_name] for reg in self if reg[field_name]), False)
        return self._convert_value(value, field_name)

    def _convert_value(self, value, field_name):
        """ Small tool because convert_to_write is touchy """
        if isinstance(value, models.BaseModel) and self._fields[field_name].type in ['many2many', 'one2many']:
            return value.ids
        if isinstance(value, models.BaseModel) and self._fields[field_name].type == 'many2one':
            return value.id
        return value

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_lead_rule
from . import event_registration
from . import crm_lead
from . import event_event

```

## File: security\event_crm_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="ir_rule_event_crm" model="ir.rule">
            <field name="name">Event CRM: Multi Company</field>
            <field name="model_id" ref="model_event_lead_rule"/>
            <field name="groups" eval="[(4, ref('base.group_multi_company'))]"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_crm_registration,event.lead.rule.user,model_event_lead_rule,event.group_event_registration_desk,1,0,0,0
access_event_crm_user,event.lead.rule.user,model_event_lead_rule,event.group_event_user,1,0,0,0
access_event_crm_manager,event.lead.rule.manager,model_event_lead_rule,event.group_event_manager,1,1,1,1
access_event_crm_salesman,event.lead.rule.salesman,model_event_lead_rule,sales_team.group_sale_salesman,1,0,0,0

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_registration_action_from_lead" model="ir.actions.act_window">
        <field name="name">Event registrations</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">event.registration</field>
        <field name="view_mode">tree,kanban,form,calendar,graph</field>
        <field name="domain">[('lead_ids', '=', active_id)]</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No registration found
            </p>
        </field>
    </record>

    <record id="crm_lead_view_form" model="ir.ui.view">
        <field name="name">crm.lead.view.form.inherit.event.crm</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_schedule_meeting']" position="after">
                <field name="registration_ids" invisible="1"/>
                <button name="%(event_registration_action_from_lead)d" type="action" class="oe_stat_button" icon="fa-ticket"
                    attrs="{'invisible': [('registration_count', '=', 0)]}" groups="event.group_event_user">
                    <div class="o_stat_info">
                        <field name="registration_count"/>
                        <span class="o_stat_text"> Attendees</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <record id="crm_lead_action_from_registration" model="ir.actions.act_window">
        <field name="name">Leads</field>
        <field name="res_model">crm.lead</field>
        <field name="view_mode">tree,kanban,graph,pivot,calendar,form,activity</field>
        <field name="domain">[('registration_ids', 'in', active_id)]</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No leads found
            </p>
        </field>
    </record>

    <record id="crm_lead_action_from_event" model="ir.actions.act_window">
        <field name="name">Leads</field>
        <field name="res_model">crm.lead</field>
        <field name="view_mode">tree,kanban,graph,pivot,calendar,form,activity</field>
        <field name="domain">[('event_id', '=', active_id)]</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No leads found
            </p>
        </field>
    </record>
</odoo>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_view_form" model="ir.ui.view">
        <field name="name">event.event.form.inherit.event.crm</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_form"/>
        <field name="priority" eval="30"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(event.act_event_registration_from_event)d']" position="after">
                <button name="%(crm_lead_action_from_event)d" class="oe_stat_button" type="action" icon="fa-star"
                    attrs="{'invisible': [('lead_count', '=' , 0)]}" groups="sales_team.group_sale_salesman">
                    <div class="o_stat_info">
                        <field name="lead_count"/>
                        <span class="o_stat_text"> Leads</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
    
    <record id="event_view_tree" model="ir.ui.view">
        <field name="name">event.event.tree.inherit.event.crm</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="event.view_event_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="lead_count" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\event_lead_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <record id="event_lead_rule_view_search" model="ir.ui.view">
        <field name="name">event.lead.rule.view.search</field>
        <field name="model">event.lead.rule</field>
        <field name="arch" type="xml">
            <search string="Search Lead Generation Rules">
                <field name="name" string="Name"/>
                <separator/>
                <filter string="Archived" name="filter_inactive" domain="[('active', '=', False)]"/>
                <filter string="Creation Type" name="filter_lead_creation_basis" context="{'group_by': 'lead_creation_basis'}"/>
                <filter string="Trigger Type" name="filter_lead_creation_trigger" context="{'group_by': 'lead_creation_trigger'}"/>
            </search>
        </field>
    </record>

    <record id="event_lead_rule_view_tree" model="ir.ui.view">
        <field name="name">event.lead.rule.view.tree</field>
        <field name="model">event.lead.rule</field>
        <field name="arch" type="xml">
            <tree string="Lead Generation Rules">
                <field name="name"/>
                <field name="lead_creation_basis" string="Lead Creation Type" invisible="1"/>
                <field name="lead_creation_trigger"/>
                <field name="event_type_ids" widget="many2many_tags"/>
                <field name="event_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="event_lead_rule_view_form" model="ir.ui.view">
        <field name="name">event.lead.rule.view.form</field>
        <field name="model">event.lead.rule</field>
        <field name="arch" type="xml">
            <form string="Lead Generation Rule">
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1><field name="name" placeholder="e.g. B2B Fairs"/></h1>
                    </div>
                    <field name="active" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                    <group name="lead_creation_configuration">
                        <group name="lead_creation_basis" invisible="1">
                            <field name="lead_creation_basis" widget="radio"/>
                        </group>
                        <group>
                            <field name="lead_creation_trigger" widget="radio"/>
                        </group>
                    </group>
                    <group string="For any of these Events">
                        <group>
                            <field name="event_type_ids" widget="many2many_tags"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                        </group>
                        <group>
                            <field name="event_id" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <group string="If the Attendees meet these Conditions">
                        <field name="event_registration_filter" widget="domain" options="{'model': 'event.registration'}" nolabel="1" colspan="2"/>
                    </group>
                    <group string="Lead Default Values">
                        <group class="col">
                            <field name="lead_type" groups="crm.group_use_lead"/>
                            <field name="lead_sales_team_id"/>
                            <field name="lead_user_id"/>
                        </group>
                        <group class="col">
                            <field name="lead_tag_ids" widget="many2many_tags" options="{'color_field': 'color'}"/>
                        </group>                         
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="event_lead_rule_action" model="ir.actions.act_window">
        <field name="name">Lead Generation Rule</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">event.lead.rule</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">Create a Lead Generation Rule</p>
            <p>Those automatically create leads when attendees register.</p>
        </field>
    </record>

    <menuitem name="Lead Generation"
        id="event_lead_rule_menu" 
        action="event_lead_rule_action"
        parent="event.menu_event_configuration"
        sequence="10"
        groups="event.group_event_manager"/>
</data>
</odoo>

```

## File: views\event_registration_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="event_registration_view_form" model="ir.ui.view">
        <field name="name">event.registration.form.inherit.event.crm</field>
        <field name="model">event.registration</field>
        <field name="inherit_id" ref="event.view_event_registration_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="%(crm_lead_action_from_registration)d" class="oe_stat_button" type="action" icon="fa-star"
                    attrs="{'invisible': [('lead_count', '=' , 0)]}" groups="sales_team.group_sale_salesman">
                    <div class="o_stat_info">
                        <field name="lead_count"/>
                        <span class="o_stat_text"> Leads</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

