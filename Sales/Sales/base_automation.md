# Odoo Module: base_automation

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Automation Rules',
    'version': '1.0',
    'category': 'Sales/Sales',
    'description': """
This module allows to implement automation rules for any object.
================================================================

Use automation rules to automatically trigger actions for various screens.

**Example:** A lead created by a specific user may be automatically set to a specific
Sales Team, or an opportunity which still has status pending after 14 days might
trigger an automatic reminder email.
    """,
    'depends': ['base', 'digest', 'resource', 'mail', 'sms'],
    'data': [
        'security/ir.model.access.csv',
        'data/base_automation_data.xml',
        'data/digest_data.xml',
        'views/ir_actions_server_views.xml',
        'views/base_automation_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'base_automation/static/src/**/*',
        ],
        'web.assets_unit_tests': [
            'base_automation/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
from odoo.http import request, route, Controller
from odoo.addons.base_automation.models.base_automation import get_webhook_request_payload

class BaseAutomationController(Controller):

    @route(['/web/hook/<string:rule_uuid>'], type='http', auth='public', methods=['GET', 'POST'], csrf=False, save_session=False)
    def call_webhook_http(self, rule_uuid, **kwargs):
        """ Execute an automation webhook """
        rule = request.env['base.automation'].sudo().search([('webhook_uuid', '=', rule_uuid)])
        if not rule:
            return request.make_json_response({'status': 'error'}, status=404)

        data = get_webhook_request_payload()
        try:
            rule._execute_webhook(data)
        except Exception: # noqa: BLE001
            return request.make_json_response({'status': 'error'}, status=500)
        return request.make_json_response({'status': 'ok'}, status=200)

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\base_automation_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="ir_cron_data_base_automation_check" model="ir.cron">
            <field name="name">Automation Rules: check and execute</field>
            <field name="model_id" ref="model_base_automation"/>
            <field name="state">code</field>
            <field name="code">model._check(True)</field>
            <field name="interval_number">4</field>
            <field name="interval_type">hours</field>
            <field name="active" eval="False" />
        </record>
    </data>
</odoo>

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="digest_tip_base_automation_0" model="digest.tip">
            <field name="name">Tip: Automate everything with Automation Rules</field>
            <field name="sequence">3700</field>
            <field name="group_id" ref="base.group_system"/>
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Automate everything with Automation Rules</p>
    <p class="tip_content">Send an email when an object changes state, archive records after a month of inactivity or remind yourself to follow-up on tasks when a specific tag is added.<br/>With Automation Rules, you can automate any workflow.</p>
    <img src="https://download.odoocdn.com/digests/base_automation/static/src/img/18-automation-rules.gif" width="540" class="illustration_border"/>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\base_automation.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
import re
import traceback
from collections import defaultdict
from uuid import uuid4

from dateutil.relativedelta import relativedelta
from odoo import _, api, exceptions, fields, models
from odoo.http import request
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, safe_eval

_logger = logging.getLogger(__name__)

DOMAIN_FIELDS_RE = re.compile(r"""
    [([]\s*                 # opening bracket with any whitespace
    (?P<quote>['"])         # opening quote
    (?P<field>[a-z]\w*)     # field name, should start with a letter then any [a-z0-9_]
    (?:\.[.\w]*)?           # dot followed by dots or text in between i.e. relation traversal (optional)
    (?P=quote)              # closing quote, matching the opening one
    (?:[^,]*?,){2}          # anything with two commas (to ensure that we are inside a triplet)
    [^,]*?[()[\]]           # anything except a comma followed by a closing bracket or another opening bracket
""", re.VERBOSE)

DATE_RANGE_FUNCTION = {
    'minutes': lambda interval: relativedelta(minutes=interval),
    'hour': lambda interval: relativedelta(hours=interval),
    'day': lambda interval: relativedelta(days=interval),
    'month': lambda interval: relativedelta(months=interval),
    False: lambda interval: relativedelta(0),
}

DATE_RANGE_FACTOR = {
    'minutes': 1,
    'hour': 60,
    'day': 24 * 60,
    'month': 30 * 24 * 60,
    False: 0,
}

CREATE_TRIGGERS = [
    'on_create',

    'on_create_or_write',
    'on_priority_set',
    'on_stage_set',
    'on_state_set',
    'on_tag_set',
    'on_user_set',
]

WRITE_TRIGGERS = [
    'on_write',
    'on_archive',
    'on_unarchive',

    'on_create_or_write',
    'on_priority_set',
    'on_stage_set',
    'on_state_set',
    'on_tag_set',
    'on_user_set',
]

MAIL_TRIGGERS = ("on_message_received", "on_message_sent")

CREATE_WRITE_SET = set(CREATE_TRIGGERS + WRITE_TRIGGERS)

TIME_TRIGGERS = [
    'on_time',
    'on_time_created',
    'on_time_updated',
]


def get_webhook_request_payload():
    if not request:
        return None
    try:
        payload = request.get_json_data()
    except ValueError:
        payload = {**request.httprequest.args}
    return payload


class BaseAutomation(models.Model):
    _name = 'base.automation'
    _description = 'Automation Rule'

    name = fields.Char(string="Automation Rule Name", required=True, translate=True)
    description = fields.Html(string="Description")
    model_id = fields.Many2one(
        "ir.model", string="Model", domain=[("field_id", "!=", False)], required=True, ondelete="cascade",
        help="Model on which the automation rule runs."
    )
    model_name = fields.Char(related="model_id.model", string="Model Name", readonly=True, inverse="_inverse_model_name")
    model_is_mail_thread = fields.Boolean(related="model_id.is_mail_thread")
    action_server_ids = fields.One2many("ir.actions.server", "base_automation_id",
        context={'default_usage': 'base_automation'},
        string="Actions",
        compute="_compute_action_server_ids",
        store=True,
        readonly=False,
    )
    url = fields.Char(compute='_compute_url')
    webhook_uuid = fields.Char(string="Webhook UUID", readonly=True, copy=False, default=lambda self: str(uuid4()))
    record_getter = fields.Char(default="model.env[payload.get('_model')].browse(int(payload.get('_id')))",
                                help="This code will be run to find on which record the automation rule should be run.")
    log_webhook_calls = fields.Boolean(string="Log Calls", default=False)
    active = fields.Boolean(default=True, help="When unchecked, the rule is hidden and will not be executed.")

    @api.constrains("trigger", "model_id")
    def _check_trigger(self):
        for automation in self:
            if automation.trigger in MAIL_TRIGGERS and not automation.model_id.is_mail_thread:
                raise exceptions.ValidationError(_("Mail event can not be configured on model %s. Only models with discussion feature can be used.", automation.model_id.name))

    trigger = fields.Selection(
        [
            ('on_stage_set', "Stage is set to"),
            ('on_user_set', "User is set"),
            ('on_tag_set', "Tag is added"),
            ('on_state_set', "State is set to"),
            ('on_priority_set', "Priority is set to"),
            ('on_archive', "On archived"),
            ('on_unarchive', "On unarchived"),
            ('on_create_or_write', "On save"),
            ('on_create', "On creation"),  # deprecated, use 'on_create_or_write' instead
            ('on_write', "On update"),  # deprecated, use 'on_create_or_write' instead

            ('on_unlink', "On deletion"),
            ('on_change', "On UI change"),

            ('on_time', "Based on date field"),
            ('on_time_created', "After creation"),
            ('on_time_updated', "After last update"),

            ("on_message_received", "On incoming message"),
            ("on_message_sent", "On outgoing message"),

            ('on_webhook', "On webhook"),
        ], string='Trigger',
        compute='_compute_trigger', readonly=False, store=True, required=True)
    trg_selection_field_id = fields.Many2one(
        'ir.model.fields.selection',
        string='Trigger Field',
        domain="[('field_id', 'in', trigger_field_ids)]",
        compute='_compute_trg_selection_field_id',
        readonly=False, store=True,
        help="Some triggers need a reference to a selection field. This field is used to store it.")
    trg_field_ref_model_name = fields.Char(
        string='Trigger Field Model',
        compute='_compute_trg_field_ref_model_name')
    trg_field_ref = fields.Many2oneReference(
        model_field='trg_field_ref_model_name',
        compute='_compute_trg_field_ref',
        string='Trigger Reference',
        readonly=False,
        store=True,
        help="Some triggers need a reference to another field. This field is used to store it.")
    trg_date_id = fields.Many2one(
        'ir.model.fields', string='Trigger Date',
        compute='_compute_trg_date_id',
        readonly=False, store=True,
        domain="[('model_id', '=', model_id), ('ttype', 'in', ('date', 'datetime'))]",
        help="""When should the condition be triggered.
                If present, will be checked by the scheduler. If empty, will be checked at creation and update.""")
    trg_date_range = fields.Integer(
        string='Delay after trigger date',
        compute='_compute_trg_date_range_data',
        readonly=False, store=True,
        help="Delay after the trigger date. "
        "You can put a negative number if you need a delay before the "
        "trigger date, like sending a reminder 15 minutes before a meeting.")
    trg_date_range_type = fields.Selection(
        [('minutes', 'Minutes'), ('hour', 'Hours'), ('day', 'Days'), ('month', 'Months')],
        string='Delay type',
        compute='_compute_trg_date_range_data',
        readonly=False, store=True)
    trg_date_calendar_id = fields.Many2one(
        "resource.calendar", string='Use Calendar',
        compute='_compute_trg_date_calendar_id',
        readonly=False, store=True,
        help="When calculating a day-based timed condition, it is possible"
             "to use a calendar to compute the date based on working days.")
    filter_pre_domain = fields.Char(
        string='Before Update Domain',
        compute='_compute_filter_pre_domain',
        readonly=False, store=True,
        help="If present, this condition must be satisfied before the update of the record. "
             "Not checked on record creation.")
    filter_domain = fields.Char(
        string='Apply on',
        help="If present, this condition must be satisfied before executing the automation rule.",
        compute='_compute_filter_domain',
        readonly=False, store=True
    )
    last_run = fields.Datetime(readonly=True, copy=False)
    on_change_field_ids = fields.Many2many(
        "ir.model.fields",
        relation="base_automation_onchange_fields_rel",
        compute='_compute_on_change_field_ids',
        readonly=False, store=True,
        string="On Change Fields Trigger",
        help="Fields that trigger the onchange.",
    )
    trigger_field_ids = fields.Many2many(
        'ir.model.fields', string='Trigger Fields',
        compute='_compute_trigger_field_ids', readonly=False, store=True,
        help="The automation rule will be triggered if and only if one of these fields is updated."
             "If empty, all fields are watched.")
    least_delay_msg = fields.Char(compute='_compute_least_delay_msg')

    # which fields have an impact on the registry and the cron
    CRITICAL_FIELDS = ['model_id', 'active', 'trigger', 'on_change_field_ids']
    RANGE_FIELDS = ['trg_date_range', 'trg_date_range_type']

    @api.constrains('model_id', 'action_server_ids')
    def _check_action_server_model(self):
        for rule in self:
            failing_actions = rule.action_server_ids.filtered(
                lambda action: action.model_id != rule.model_id
            )
            if failing_actions:
                raise exceptions.ValidationError(
                    _('Target model of actions %(action_names)s are different from rule model.',
                      action_names=', '.join(failing_actions.mapped('name'))
                     )
                )

    @api.depends("trigger", "webhook_uuid")
    def _compute_url(self):
        for automation in self:
            if automation.trigger != "on_webhook":
                automation.url = ""
            else:
                automation.url = "%s/web/hook/%s" % (automation.get_base_url(), automation.webhook_uuid)

    def _inverse_model_name(self):
        for rec in self:
            rec.model_id = self.env["ir.model"]._get(rec.model_name)

    @api.constrains('trigger', 'action_server_ids')
    def _check_trigger_state(self):
        for record in self:
            no_code_actions = record.action_server_ids.filtered(lambda a: a.state != 'code')
            if record.trigger == 'on_change' and no_code_actions:
                raise exceptions.ValidationError(
                    _('"On live update" automation rules can only be used with "Execute Python Code" action type.')
                )
            mail_actions = record.action_server_ids.filtered(
                lambda a: a.state in ['mail_post', 'followers', 'next_activity']
            )
            if record.trigger == 'on_unlink' and mail_actions:
                raise exceptions.ValidationError(
                    _('Email, follower or activity action types cannot be used when deleting records, '
                      'as there are no more records to apply these changes to!')
                )

    @api.depends('model_id')
    def _compute_action_server_ids(self):
        """ When changing / setting model, remove actions that are not targeting
        the same model anymore. """
        for rule in self.filtered('model_id'):
            actions_to_remove = rule.action_server_ids.filtered(
                lambda action: action.model_id != rule.model_id
            )
            if actions_to_remove:
                rule.action_server_ids = [(3, action.id) for action in actions_to_remove]

    @api.depends('trigger')
    def _compute_trg_date_id(self):
        to_reset = self.filtered(lambda a: a.trigger not in TIME_TRIGGERS)
        to_reset.trg_date_id = False
        for record in (self - to_reset):
            record.trg_date_id = record._get_trigger_specific_field()

    @api.depends('trigger')
    def _compute_trg_date_range_data(self):
        to_reset = self.filtered(lambda a: a.trigger not in TIME_TRIGGERS)
        to_reset.trg_date_range = False
        to_reset.trg_date_range_type = False
        (self - to_reset).filtered(lambda a: not a.trg_date_range_type).trg_date_range_type = 'hour'

    @api.depends('trigger', 'trg_date_id', 'trg_date_range_type')
    def _compute_trg_date_calendar_id(self):
        to_reset = self.filtered(
            lambda a: a.trigger not in TIME_TRIGGERS or not a.trg_date_id or a.trg_date_range_type != 'day'
        )
        to_reset.trg_date_calendar_id = False

    @api.depends('trigger')
    def _compute_trg_selection_field_id(self):
        self.trg_selection_field_id = False

    @api.depends('trigger')
    def _compute_trg_field_ref(self):
        self.trg_field_ref = False

    @api.depends('trigger', 'trg_field_ref')
    def _compute_trg_field_ref_model_name(self):
        to_compute = self.filtered(lambda a: a.trigger in ['on_stage_set', 'on_tag_set'] and a.trg_field_ref is not False)
        # wondering why we check based on 'is not'? Because the ref could be an empty recordset
        # and we still need to introspec on the model in that case - not just ignore it
        to_reset = (self - to_compute)
        to_reset.trg_field_ref_model_name = False
        for automation in to_compute:
            relation = automation._get_trigger_specific_field().relation
            if not relation:
                automation.trg_field_ref_model_name = False
                continue
            automation.trg_field_ref_model_name = relation

    @api.depends('trigger', 'trg_field_ref')
    def _compute_filter_pre_domain(self):
        to_reset = self.filtered(lambda a: a.trigger != 'on_tag_set')
        to_reset.filter_pre_domain = False
        for automation in (self - to_reset):
            field = automation._get_trigger_specific_field().name
            value = automation.trg_field_ref
            automation.filter_pre_domain = repr([(field, 'not in', [value])]) if value else False

    @api.depends('trigger', 'trg_selection_field_id', 'trg_field_ref')
    def _compute_filter_domain(self):
        for automation in self:
            field = (
                automation._get_trigger_specific_field()
                if automation.trigger not in ["on_create_or_write", *TIME_TRIGGERS]
                else False
            )
            if not field:
                automation.filter_domain = False
                continue

            # some triggers require a domain
            match automation.trigger:
                case 'on_state_set' | 'on_priority_set':
                    value = automation.trg_selection_field_id.value
                    automation.filter_domain = repr([(field.name, '=', value)]) if value else False
                case 'on_stage_set':
                    value = automation.trg_field_ref
                    automation.filter_domain = repr([(field.name, '=', value)]) if value else False
                case 'on_tag_set':
                    value = automation.trg_field_ref
                    automation.filter_domain = repr([(field.name, 'in', [value])]) if value else False
                case 'on_user_set':
                    automation.filter_domain = repr([(field.name, '!=', False)])
                case 'on_archive':
                    automation.filter_domain = repr([(field.name, '=', False)])
                case 'on_unarchive':
                    automation.filter_domain = repr([(field.name, '=', True)])

    @api.depends('model_id', 'trigger', 'filter_domain')
    def _compute_on_change_field_ids(self):
        to_reset = self.filtered(lambda a: a.trigger != 'on_change')
        to_reset.on_change_field_ids = False
        for automation in (self - to_reset):
            automation.on_change_field_ids |= automation._get_filter_domain_fields()

    @api.depends('model_id', 'trigger', 'filter_domain')
    def _compute_trigger_field_ids(self):
        for automation in self:
            if automation.trigger == "on_create_or_write":
                automation.trigger_field_ids |= automation._get_filter_domain_fields()
                continue
            automation._onchange_trigger()

    @api.depends('model_id')
    def _compute_trigger(self):
        self.trigger = False

    @api.onchange('trigger')
    def _onchange_trigger(self):
        field = (
            self._get_trigger_specific_field()
            if self.trigger not in TIME_TRIGGERS
            else False
        )
        self.trigger_field_ids = field


    @api.onchange('trigger', 'action_server_ids')
    def _onchange_trigger_or_actions(self):
        no_code_actions = self.action_server_ids.filtered(lambda a: a.state != 'code')
        if self.trigger == 'on_change' and len(no_code_actions) > 0:
            trigger_field = self._fields['trigger']
            action_states = dict(self.action_server_ids._fields['state']._description_selection(self.env))
            return {'warning': {
                'title': _("Warning"),
                'message': _(
                    "The \"%(trigger_value)s\" %(trigger_label)s can only be "
                    "used with the \"%(state_value)s\" action type",
                    trigger_value=dict(trigger_field._description_selection(self.env))['on_change'],
                    trigger_label=trigger_field._description_string(self.env),
                    state_value=action_states['code'])
            }}

        MAIL_STATES = ('mail_post', 'followers', 'next_activity')
        mail_actions = self.action_server_ids.filtered(lambda a: a.state in MAIL_STATES)
        if self.trigger == 'on_unlink' and len(mail_actions) > 0:
            return {'warning': {
                'title': _("Warning"),
                'message': _(
                    "You cannot send an email, add followers or create an activity "
                    "for a deleted record.  It simply does not work."
                ),
            }}

    def _has_trigger_onchange(self):
        return any(
            automation.active and automation.trigger == 'on_change' and automation.on_change_field_ids
            for automation in self
        )

    @api.model_create_multi
    def create(self, vals_list):
        base_automations = super(BaseAutomation, self).create(vals_list)
        self._update_cron()
        self._update_registry()
        if base_automations._has_trigger_onchange():
            # Invalidate templates cache to update on_change attributes if needed
            self.env.registry.clear_cache('templates')
        return base_automations

    def write(self, vals: dict):
        clear_templates = self._has_trigger_onchange()
        res = super(BaseAutomation, self).write(vals)
        if set(vals).intersection(self.CRITICAL_FIELDS):
            self._update_cron()
            self._update_registry()
            if clear_templates or self._has_trigger_onchange():
                # Invalidate templates cache to update on_change attributes if needed
                self.env.registry.clear_cache('templates')
        elif set(vals).intersection(self.RANGE_FIELDS):
            self._update_cron()
        return res

    def unlink(self):
        clear_templates = self._has_trigger_onchange()
        res = super(BaseAutomation, self).unlink()
        self._update_cron()
        self._update_registry()
        if clear_templates:
            # Invalidate templates cache to update on_change attributes if needed
            self.env.registry.clear_cache('templates')
        return res

    def copy(self, default=None):
        """Copy the actions of the automation while
        copying the automation itself."""
        actions = self.action_server_ids.copy()
        record_copy = super().copy(default)
        record_copy.action_server_ids = actions
        return record_copy

    def action_rotate_webhook_uuid(self):
        for automation in self:
            automation.webhook_uuid = str(uuid4())

    def action_view_webhook_logs(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Webhook Logs'),
            'res_model': 'ir.logging',
            'view_mode': 'list,form',
            'domain': [('path', '=', "base_automation(%s)" % self.id)],
        }

    def _get_filter_domain_fields(self):
        self.ensure_one()
        if not self.filter_domain or not self.model_id:
            return self.env['ir.model.fields']
        model = self.model_id.model
        fields = self.env["ir.model.fields"]
        # wondering why we use a regex instead of safe_eval?
        # because this method is called on a compute method hence could be triggered
        # from an onchange call (i.e. a manually crafted malicious one)
        # see: https://github.com/odoo/odoo/pull/189772#issuecomment-2548804283
        for match in DOMAIN_FIELDS_RE.finditer(self.filter_domain):
            if field := match.groupdict().get('field'):
                fields |= self.env["ir.model.fields"]._get(model, field)
        return fields

    def _get_trigger_specific_field(self):
        self.ensure_one()
        match self.trigger:
            case 'on_create_or_write':
                return self._get_filter_domain_fields()
            case 'on_stage_set':
                domain = [('ttype', '=', 'many2one'), ('name', 'in', ['stage_id', 'x_studio_stage_id'])]
            case 'on_tag_set':
                domain = [('ttype', '=', 'many2many'), ('name', 'in', ['tag_ids', 'x_studio_tag_ids'])]
            case 'on_priority_set':
                domain = [('ttype', '=', 'selection'), ('name', 'in', ['priority', 'x_studio_priority'])]
            case 'on_state_set':
                domain = [('ttype', '=', 'selection'), ('name', 'in', ['state', 'x_studio_state'])]
            case 'on_user_set':
                domain = [
                    ('relation', '=', 'res.users'),
                    ('ttype', 'in', ['many2one', 'many2many']),
                    ('name', 'in', ['user_id', 'user_ids', 'x_studio_user_id', 'x_studio_user_ids']),
                ]
            case 'on_archive' | 'on_unarchive':
                domain = [('ttype', '=', 'boolean'), ('name', 'in', ['active', 'x_active'])]
            case 'on_time_created':
                domain = [('ttype', '=', 'datetime'), ('name', '=', 'create_date')]
            case 'on_time_updated':
                domain = [('ttype', '=', 'datetime'), ('name', '=', 'write_date')]
            case _:
                return self.env['ir.model.fields']
        domain += [('model_id', '=', self.model_id.id)]
        return self.env['ir.model.fields'].search(domain, limit=1)

    def _prepare_loggin_values(self, **values):
        self.ensure_one()
        defaults = {
            'name': _("Webhook Log"),
            'type': 'server',
            'dbname': self._cr.dbname,
            'level': 'INFO',
            'path': "base_automation(%s)" % self.id,
            'func': '',
            'line': ''
        }
        defaults.update(**values)
        return defaults

    def _execute_webhook(self, payload):
        """ Execute the webhook for the given payload.
        The payload is a dictionnary that can be used by the `record_getter` to
        identify the record on which the automation should be run.
        """
        self.ensure_one()
        ir_logging_sudo = self.env['ir.logging'].sudo()

        # info logging is done by the ir.http logger
        msg = "Webhook #%s triggered with payload %s"
        msg_args = (self.id, payload)
        _logger.debug(msg, *msg_args)
        if self.log_webhook_calls:
            ir_logging_sudo.create(self._prepare_loggin_values(message=msg % msg_args))

        record = self.env[self.model_name]
        if self.record_getter:
            try:
                record = safe_eval.safe_eval(self.record_getter, self._get_eval_context(payload=payload))
            except Exception as e: # noqa: BLE001
                msg = "Webhook #%s could not be triggered because the record_getter failed:\n%s"
                msg_args = (self.id, traceback.format_exc())
                _logger.warning(msg, *msg_args)
                if self.log_webhook_calls:
                    ir_logging_sudo.create(self._prepare_loggin_values(message=msg % msg_args, level="ERROR"))
                raise e

        if not record.exists():
            msg = "Webhook #%s could not be triggered because no record to run it on was found."
            msg_args = (self.id,)
            _logger.warning(msg, *msg_args)
            if self.log_webhook_calls:
                ir_logging_sudo.create(self._prepare_loggin_values(message=msg % msg_args, level="ERROR"))
            raise exceptions.ValidationError(_("No record to run the automation on was found."))

        try:
            return self._process(record)
        except Exception as e: # noqa: BLE001
            msg = "Webhook #%s failed with error:\n%s"
            msg_args = (self.id, traceback.format_exc())
            _logger.warning(msg, *msg_args)
            if self.log_webhook_calls:
                ir_logging_sudo.create(self._prepare_loggin_values(message=msg % msg_args, level="ERROR"))
            raise e

    def _update_cron(self):
        """ Activate the cron job depending on whether there exists automation rules
        based on time conditions.  Also update its frequency according to
        the smallest automation delay, or restore the default 4 hours if there
        is no time based automation.
        """
        cron = self.env.ref('base_automation.ir_cron_data_base_automation_check', raise_if_not_found=False)
        if cron:
            automations = self.with_context(active_test=True).search([('trigger', 'in', TIME_TRIGGERS)])
            cron.try_write({
                'active': bool(automations),
                'interval_type': 'minutes',
                'interval_number': self._get_cron_interval(automations),
            })

    def _update_registry(self):
        """ Update the registry after a modification on automation rules. """
        if self.env.registry.ready and not self.env.context.get('import_file'):
            # re-install the model patches, and notify other workers
            self._unregister_hook()
            self._register_hook()
            self.env.registry.registry_invalidated = True

    def _get_actions(self, records, triggers):
        """ Return the automations of the given triggers for records' model. The
            returned automations' context contain an object to manage processing.
        """
        # Note: we keep the old action naming for the method and context variable
        # to avoid breaking existing code/downstream modules
        if '__action_done' not in self._context:
            self = self.with_context(__action_done={})
        domain = [('model_name', '=', records._name), ('trigger', 'in', triggers)]
        automations = self.with_context(active_test=True).sudo().search(domain)
        return automations.with_env(self.env)

    def _get_eval_context(self, payload=None):
        """ Prepare the context used when evaluating python code
            :returns: dict -- evaluation context given to safe_eval
        """
        self.ensure_one()
        model = self.env[self.model_name]
        eval_context = {
            'datetime': safe_eval.datetime,
            'dateutil': safe_eval.dateutil,
            'time': safe_eval.time,
            'uid': self.env.uid,
            'user': self.env.user,
            'model': model,
        }
        if payload is not None:
            eval_context['payload'] = payload
        return eval_context

    def _get_cron_interval(self, automations=None):
        """ Return the expected time interval used by the cron, in minutes. """
        def get_delay(rec):
            return abs(rec.trg_date_range) * DATE_RANGE_FACTOR[rec.trg_date_range_type]

        if automations is None:
            automations = self.with_context(active_test=True).search([('trigger', 'in', TIME_TRIGGERS)])

        # Minimum 1 minute, maximum 4 hours, 10% tolerance
        delay = min(automations.mapped(get_delay), default=0)
        return min(max(1, delay // 10), 4 * 60) if delay else 4 * 60

    def _compute_least_delay_msg(self):
        msg = _("Note that this automation rule can be triggered up to %d minutes after its schedule.")
        self.least_delay_msg = msg % self._get_cron_interval()

    def _filter_pre(self, records, feedback=False):
        """ Filter the records that satisfy the precondition of automation ``self``. """
        self_sudo = self.sudo()
        if self_sudo.filter_pre_domain and records:
            if feedback:
                # this context flag enables to detect the executions of
                # automations while evaluating their precondition
                records = records.with_context(__action_feedback=True)
            domain = safe_eval.safe_eval(self_sudo.filter_pre_domain, self._get_eval_context())
            return records.sudo().filtered_domain(domain).with_env(records.env)
        else:
            return records

    def _filter_post(self, records, feedback=False):
        return self._filter_post_export_domain(records, feedback)[0]

    def _filter_post_export_domain(self, records, feedback=False):
        """ Filter the records that satisfy the postcondition of automation ``self``. """
        self_sudo = self.sudo()
        if self_sudo.filter_domain and records:
            if feedback:
                # this context flag enables to detect the executions of
                # automations while evaluating their postcondition
                records = records.with_context(__action_feedback=True)
            domain = safe_eval.safe_eval(self_sudo.filter_domain, self._get_eval_context())
            return records.sudo().filtered_domain(domain).with_env(records.env), domain
        else:
            return records, None

    @api.model
    def _add_postmortem(self, e):
        if self.env.user._is_internal():
            e.context = {}
            e.context['exception_class'] = 'base_automation'
            e.context['base_automation'] = {
                'id': self.id,
                'name': self.sudo().name,
            }

    def _process(self, records, domain_post=None):
        """ Process automation ``self`` on the ``records`` that have not been done yet. """
        # filter out the records on which self has already been done
        automation_done = self._context.get('__action_done', {})
        records_done = automation_done.get(self, records.browse())
        records -= records_done
        if not records:
            return

        # mark the remaining records as done (to avoid recursive processing)
        if self.env.context.get('__action_feedback'):
            # modify the context dict in place: this is useful when fields are
            # computed during the pre/post filtering, in order to know which
            # automations have already been run by the computation itself
            automation_done[self] = records_done + records
        else:
            automation_done = dict(automation_done)
            automation_done[self] = records_done + records
            self = self.with_context(__action_done=automation_done)
            records = records.with_context(__action_done=automation_done)

        # we process the automation on the records for which any watched field
        # has been modified, and only mark the automation as done for those
        records = records.filtered(self._check_trigger_fields)
        automation_done[self] = records_done + records

        if records and 'date_automation_last' in records._fields:
            records.date_automation_last = fields.Datetime.now()

        # prepare the contexts for server actions
        contexts = [
            {
                'active_model': record._name,
                'active_ids': record.ids,
                'active_id': record.id,
                'domain_post': domain_post,
            }
            for record in records
        ]

        # execute server actions
        for action in self.sudo().action_server_ids:
            for ctx in contexts:
                try:
                    action.with_context(**ctx).run()
                except Exception as e:
                    self._add_postmortem(e)
                    raise

    def _check_trigger_fields(self, record):
        """ Return whether any of the trigger fields has been modified on ``record``. """
        self_sudo = self.sudo()
        if not self_sudo.trigger_field_ids:
            # all fields are implicit triggers
            return True

        if self._context.get('old_values') is None:
            # this is a create: all fields are considered modified
            return True

        # note: old_vals are in the record format
        old_vals = self._context['old_values'].get(record.id, {})

        def differ(name):
            return name in old_vals and record[name] != old_vals[name]

        return any(differ(field.name) for field in self_sudo.trigger_field_ids)

    def _register_hook(self):
        """ Patch models that should trigger action rules based on creation,
            modification, deletion of records and form onchanges.
        """
        #
        # Note: the patched methods must be defined inside another function,
        # otherwise their closure may be wrong. For instance, the function
        # create refers to the outer variable 'create', which you expect to be
        # bound to create itself. But that expectation is wrong if create is
        # defined inside a loop; in that case, the variable 'create' is bound to
        # the last function defined by the loop.
        #

        def make_create():
            """ Instanciate a create method that processes automation rules. """
            @api.model_create_multi
            def create(self, vals_list, **kw):
                # retrieve the automation rules to possibly execute
                automations = self.env['base.automation']._get_actions(self, CREATE_TRIGGERS)
                if not automations:
                    return create.origin(self, vals_list, **kw)
                # call original method
                records = create.origin(self.with_env(automations.env), vals_list, **kw)
                # check postconditions, and execute actions on the records that satisfy them
                for automation in automations.with_context(old_values=None):
                    automation._process(automation._filter_post(records, feedback=True))
                return records.with_env(self.env)

            return create

        def make_write():
            """ Instanciate a write method that processes automation rules. """
            def write(self, vals, **kw):
                # retrieve the automation rules to possibly execute
                automations = self.env['base.automation']._get_actions(self, WRITE_TRIGGERS)
                if not (automations and self):
                    return write.origin(self, vals, **kw)
                records = self.with_env(automations.env).filtered('id')
                # check preconditions on records
                pre = {a: a._filter_pre(records) for a in automations}
                # read old values before the update
                old_values = {
                    record.id: {field_name: record[field_name] for field_name in vals if field_name in record._fields and record._fields[field_name].store}
                    for record in records
                }
                # call original method
                write.origin(self.with_env(automations.env), vals, **kw)
                # check postconditions, and execute actions on the records that satisfy them
                for automation in automations.with_context(old_values=old_values):
                    records, domain_post = automation._filter_post_export_domain(pre[automation], feedback=True)
                    automation._process(records, domain_post=domain_post)
                return True

            return write

        def make_compute_field_value():
            """ Instanciate a compute_field_value method that processes automation rules. """
            #
            # Note: This is to catch updates made by field recomputations.
            #
            def _compute_field_value(self, field):
                # determine fields that may trigger an automation
                stored_fnames = [f.name for f in self.pool.field_computed[field] if f.store]
                if not stored_fnames:
                    return _compute_field_value.origin(self, field)
                # retrieve the action rules to possibly execute
                automations = self.env['base.automation']._get_actions(self, WRITE_TRIGGERS)
                records = self.filtered('id').with_env(automations.env)
                if not (automations and records):
                    _compute_field_value.origin(self, field)
                    return True
                # check preconditions on records
                pre = {a: a._filter_pre(records) for a in automations}
                # read old values before the update
                old_values = {
                    record.id: {fname: record[fname] for fname in stored_fnames}
                    for record in records
                }
                # call original method
                _compute_field_value.origin(self, field)
                # check postconditions, and execute automations on the records that satisfy them
                for automation in automations.with_context(old_values=old_values):
                    records, domain_post = automation._filter_post_export_domain(pre[automation], feedback=True)
                    automation._process(records, domain_post=domain_post)
                return True

            return _compute_field_value

        def make_unlink():
            """ Instanciate an unlink method that processes automation rules. """
            def unlink(self, **kwargs):
                # retrieve the action rules to possibly execute
                automations = self.env['base.automation']._get_actions(self, ['on_unlink'])
                records = self.with_env(automations.env)
                # check conditions, and execute actions on the records that satisfy them
                for automation in automations:
                    automation._process(automation._filter_post(records, feedback=True))
                # call original method
                return unlink.origin(self, **kwargs)

            return unlink

        def make_onchange(automation_rule_id):
            """ Instanciate an onchange method for the given automation rule. """
            def base_automation_onchange(self):
                automation_rule = self.env['base.automation'].browse(automation_rule_id)
                result = {}
                actions = automation_rule.sudo().action_server_ids.with_context(
                    active_model=self._name,
                    active_id=self._origin.id,
                    active_ids=self._origin.ids,
                    onchange_self=self,
                )
                for action in actions:
                    try:
                        res = action.run()
                    except Exception as e:
                        automation_rule._add_postmortem(e)
                        raise

                    if res:
                        if 'value' in res:
                            res['value'].pop('id', None)
                            self.update({key: val for key, val in res['value'].items() if key in self._fields})
                        if 'domain' in res:
                            result.setdefault('domain', {}).update(res['domain'])
                        if 'warning' in res:
                            result['warning'] = res["warning"]
                return result

            return base_automation_onchange

        def make_message_post():
            def _message_post(self, *args, **kwargs):
                message = _message_post.origin(self, *args, **kwargs)
                # Don't execute automations for a message emitted during
                # the run of automations for a real message
                # Don't execute if we know already that a message is only internal
                message_sudo = message.sudo().with_context(active_test=False)
                if "__action_done"  in self.env.context or message_sudo.is_internal or message_sudo.subtype_id.internal:
                    return message
                if message_sudo.message_type in ('notification', 'auto_comment', 'user_notification'):
                    return message

                # always execute actions when the author is a customer
                # if author is not set, it means the message is coming from outside
                mail_trigger = "on_message_received" if not message_sudo.author_id or message_sudo.author_id.partner_share else "on_message_sent"
                automations = self.env['base.automation']._get_actions(self, [mail_trigger])
                for automation in automations.with_context(old_values=None):
                    records = automation._filter_pre(self, feedback=True)
                    automation._process(records)

                return message
            return _message_post

        patched_models = defaultdict(set)

        def patch(model, name, method):
            """ Patch method `name` on `model`, unless it has been patched already. """
            if model not in patched_models[name]:
                patched_models[name].add(model)
                ModelClass = model.env.registry[model._name]
                method.origin = getattr(ModelClass, name)
                setattr(ModelClass, name, method)

        # retrieve all actions, and patch their corresponding model
        for automation_rule in self.with_context({}).search([]):
            Model = self.env.get(automation_rule.model_name)

            # Do not crash if the model of the base_action_rule was uninstalled
            if Model is None:
                _logger.warning(
                    "Automation rule with name '%s' (ID %d) depends on model %s (ID: %d)",
                    automation_rule.name,
                    automation_rule.id,
                    automation_rule.model_name,
                    automation_rule.model_id.id)
                continue

            if automation_rule.trigger in CREATE_WRITE_SET:
                if automation_rule.trigger in CREATE_TRIGGERS:
                    patch(Model, 'create', make_create())
                if automation_rule.trigger in WRITE_TRIGGERS:
                    patch(Model, 'write', make_write())
                    patch(Model, '_compute_field_value', make_compute_field_value())

            elif automation_rule.trigger == 'on_unlink':
                patch(Model, 'unlink', make_unlink())

            elif automation_rule.trigger == 'on_change':
                # register an onchange method for the automation_rule
                method = make_onchange(automation_rule.id)
                for field in automation_rule.on_change_field_ids:
                    Model._onchange_methods[field.name].append(method)

            if automation_rule.model_id.is_mail_thread and automation_rule.trigger in MAIL_TRIGGERS:
                patch(Model, "message_post", make_message_post())

    def _unregister_hook(self):
        """ Remove the patches installed by _register_hook() """
        NAMES = ['create', 'write', '_compute_field_value', 'unlink', '_onchange_methods', "message_post"]
        for Model in self.env.registry.values():
            for name in NAMES:
                try:
                    delattr(Model, name)
                except AttributeError:
                    pass

    @api.model
    def _get_calendar(self, automation, record):
        return automation.trg_date_calendar_id

    @api.model
    def _check(self, automatic=False, use_new_cursor=False):
        """ This Function is called by scheduler. """
        if '__action_done' not in self._context:
            self = self.with_context(__action_done={})

        # retrieve all the automation rules to run based on a timed condition
        for automation in self.with_context(active_test=True).search([('trigger', 'in', TIME_TRIGGERS)]):
            _logger.info("Starting time-based automation rule `%s`.", automation.name)
            last_run = fields.Datetime.from_string(automation.last_run) or datetime.datetime.fromtimestamp(0, tz=None)
            eval_context = automation._get_eval_context()

            # retrieve all the records that satisfy the automation's condition
            domain = []
            context = dict(self._context)
            if automation.filter_domain:
                domain = safe_eval.safe_eval(automation.filter_domain, eval_context)
            records = self.env[automation.model_name].with_context(context).search(domain)

            def get_record_dt(record):
                # determine when automation should occur for the records
                if automation.trg_date_id.name == "date_automation_last" and "create_date" in records._fields:
                    return record[automation.trg_date_id.name] or record.create_date
                else:
                    return record[automation.trg_date_id.name]

            # process action on the records that should be executed
            now = datetime.datetime.now()
            past_now = {}
            past_last_run = {}
            for record in records:
                record_dt = get_record_dt(record)
                if not record_dt:
                    continue
                if automation.trg_date_calendar_id and automation.trg_date_range_type == 'day':
                    calendar = self._get_calendar(automation, record)
                    if calendar.id not in past_now:
                        past_now[calendar.id] = calendar.plan_days(
                            - automation.trg_date_range,
                            now,
                            compute_leaves=True,
                        )
                        past_last_run[calendar.id] = calendar.plan_days(
                            - automation.trg_date_range,
                            last_run,
                            compute_leaves=True,
                        )
                    is_process_to_run = past_last_run[calendar.id] <= fields.Datetime.to_datetime(record_dt) < past_now[calendar.id]
                else:
                    is_process_to_run = (
                        last_run <=
                        fields.Datetime.from_string(record_dt) + DATE_RANGE_FUNCTION[automation.trg_date_range_type](automation.trg_date_range)
                        < now
                    )
                if is_process_to_run:
                    try:
                        automation._process(record)
                    except Exception:
                        _logger.error(traceback.format_exc())

            automation.write({'last_run': now.strftime(DEFAULT_SERVER_DATETIME_FORMAT)})
            _logger.info("Time-based automation rule `%s` done.", automation.name)

            if automatic:
                # auto-commit for batch processing
                self._cr.commit()

```

## File: models\ir_actions_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.tools.json import scriptsafe as json_scriptsafe

from odoo import api, exceptions, fields, models, _

from .base_automation import get_webhook_request_payload

class ServerAction(models.Model):
    _inherit = "ir.actions.server"

    name = fields.Char(compute='_compute_name', store=True, readonly=False)

    usage = fields.Selection(selection_add=[
        ('base_automation', 'Automation Rule')
    ], ondelete={'base_automation': 'cascade'})
    base_automation_id = fields.Many2one('base.automation', string='Automation Rule', ondelete='cascade')

    @api.constrains('model_id', 'base_automation_id')
    def _check_model_coherency_with_automation(self):
        for action in self.filtered('base_automation_id'):
            if action.model_id != action.base_automation_id.model_id:
                raise exceptions.ValidationError(
                    _("Model of action %(action_name)s should match the one from automated rule %(rule_name)s.",
                      action_name=action.name,
                      rule_name=action.base_automation_id.name
                     )
                )

    @api.depends('usage')
    def _compute_available_model_ids(self):
        """ Stricter model limit: based on automation rule """
        super()._compute_available_model_ids()
        rule_based = self.filtered(lambda action: action.usage == 'base_automation')
        for action in rule_based:
            rule_model = action.base_automation_id.model_id
            action.available_model_ids = rule_model.ids if rule_model in action.available_model_ids else []

    @api.depends('state', 'update_field_id', 'crud_model_id', 'value', 'evaluation_type', 'template_id', 'partner_ids', 'activity_summary', 'sms_template_id', 'webhook_url')
    def _compute_name(self):
        ''' Only server actions linked to a base_automation get an automatic name. '''
        to_update = self.filtered('base_automation_id')
        for action in to_update:
            match action.state:
                case 'object_write':
                    action_type = _("Update") if action.evaluation_type == 'value' else _("Compute")
                    action.name = f"{action_type} {action._stringify_path()}"
                case 'object_create':
                    action.name = _(
                    "Create %(model_name)s with name %(value)s",
                        model_name=action.crud_model_id.name,
                        value=action.value
                    )
                case 'webhook':
                    action.name = _("Send Webhook Notification")
                case 'sms':
                    action.name = _(
                    'Send SMS: %(template_name)s',
                    template_name=action.sms_template_id.name
                )
                case 'mail_post':
                    action.name = _(
                        'Send email: %(template_name)s',
                        template_name=action.template_id.name
                    )
                case 'followers':
                    action.name = _(
                        'Add followers: %(partner_names)s',
                        partner_names=', '.join(action.partner_ids.mapped('name'))
                    )
                case 'remove_followers':
                    action.name = _(
                        'Remove followers: %(partner_names)s',
                        partner_names=', '.join(action.partner_ids.mapped('name'))
                    )
                case 'next_activity':
                    action.name = _(
                        'Create activity: %(activity_name)s',
                        activity_name=action.activity_summary or action.activity_type_id.name
                    )
                case other:
                    action.name = dict(action._fields['state']._description_selection(self.env))[action.state]
        # Not sure, but IIRC assignation is mandatory and I don't want the name to be reset by accident
        for action in (self - to_update):
            action.name = action.name or ''

    def _get_eval_context(self, action=None):
        eval_context = super()._get_eval_context(action)
        if action and action.state == "code":
            eval_context['json'] = json_scriptsafe
            payload = get_webhook_request_payload()
            if payload:
                eval_context["payload"] = payload
        return eval_context

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_automation
from . import ir_actions_server

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_automation_config,base.automation config,model_base_automation,base.group_system,1,1,1,1

```

## File: static\img\automation.svg

```svg
<svg id="eseIaTOU95c1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="50 50 400 300" shape-rendering="geometricPrecision" text-rendering="geometricPrecision">
<style><![CDATA[
#eseIaTOU95c2_to {animation: eseIaTOU95c2_to__to 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c2_to__to { 0% {transform: translate(249.999992px,193.51566px)} 17.777778% {transform: translate(249.999992px,193.51566px)} 20% {transform: translate(249.999992px,240.772908px)} 31.111111% {transform: translate(249.999992px,240.772908px)} 33.333333% {transform: translate(249.999994px,287.87677px)} 42.222222% {transform: translate(249.999994px,287.87677px)} 54.444444% {transform: translate(249.999994px,287.87677px)} 75.555556% {transform: translate(249.999994px,193.42367px)} 100% {transform: translate(249.999994px,193.42367px)}} #eseIaTOU95c2_ts {animation: eseIaTOU95c2_ts__ts 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c2_ts__ts { 0% {transform: scale(1,1)} 15.555556% {transform: scale(1,1)} 16.666667% {transform: scale(1.1,1.1)} 17.777778% {transform: scale(1,1)} 28.888889% {transform: scale(1,1)} 30% {transform: scale(1.1,1.1)} 31.111111% {transform: scale(1,1)} 46.666667% {transform: scale(1,1)} 47.777778% {transform: scale(1.1,1.1)} 48.888889% {transform: scale(1,1)} 100% {transform: scale(1,1)}} #eseIaTOU95c2 {animation: eseIaTOU95c2_c_o 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c2_c_o { 0% {opacity: 1} 54.444444% {opacity: 1} 55.555556% {opacity: 0} 97.777778% {opacity: 0} 98.888889% {opacity: 1} 100% {opacity: 1}} #eseIaTOU95c16 {animation: eseIaTOU95c16_c_o 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c16_c_o { 0% {opacity: 0} 17.777778% {opacity: 0} 20% {opacity: 1} 92.222222% {opacity: 1} 97.777778% {opacity: 0} 100% {opacity: 0}} #eseIaTOU95c22 {animation: eseIaTOU95c22_c_o 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c22_c_o { 0% {opacity: 0} 31.111111% {opacity: 0} 33.333333% {opacity: 1} 92.222222% {opacity: 1} 97.777778% {opacity: 0} 100% {opacity: 0}} #eseIaTOU95c30 {animation: eseIaTOU95c30_c_o 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c30_c_o { 0% {opacity: 0} 48.888889% {opacity: 0} 51.111111% {opacity: 1} 92.222222% {opacity: 1} 97.777778% {opacity: 0} 100% {opacity: 0}} #eseIaTOU95c36_to {animation: eseIaTOU95c36_to__to 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c36_to__to { 0% {transform: translate(455.595461px,229.469635px)} 15.555556% {transform: translate(249.999999px,193.51566px);animation-timing-function: cubic-bezier(0,0,0.58,1)} 24.444444% {transform: translate(249.999999px,193.51566px)} 27.777778% {transform: translate(249.999999px,241.30167px)} 42.222222% {transform: translate(249.999999px,241.30167px)} 45.555556% {transform: translate(249.999999px,287.87677px)} 54.444444% {transform: translate(249.999999px,287.87677px);animation-timing-function: cubic-bezier(0.42,0,1,1)} 66.666667% {transform: translate(307.985766px,357.337544px)} 100% {transform: translate(307.985766px,357.337544px)}} #eseIaTOU95c36_ts {animation: eseIaTOU95c36_ts__ts 9000ms linear infinite normal forwards}@keyframes eseIaTOU95c36_ts__ts { 0% {transform: scale(0.575337,0.575337)} 15.555556% {transform: scale(0.575337,0.575337)} 16.666667% {transform: scale(0.5,0.5)} 17.777778% {transform: scale(0.58,0.58)} 28.888889% {transform: scale(0.58,0.58)} 30% {transform: scale(0.5,0.5)} 31.111111% {transform: scale(0.58,0.58)} 46.666667% {transform: scale(0.58,0.58)} 47.777778% {transform: scale(0.58,0.58)} 48.888889% {transform: scale(0.5,0.5)} 100% {transform: scale(0.5,0.5)}}
]]></style>
<defs><filter id="eseIaTOU95c2-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c2-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="3,3"/><feOffset id="eseIaTOU95c2-filter-drop-shadow-0-offset" dx="0" dy="0" result="tmp"/><feFlood id="eseIaTOU95c2-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.2)"/><feComposite id="eseIaTOU95c2-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c2-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c2-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c2-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter><filter id="eseIaTOU95c10-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c10-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="3,3"/><feOffset id="eseIaTOU95c10-filter-drop-shadow-0-offset" dx="0" dy="0" result="tmp"/><feFlood id="eseIaTOU95c10-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.2)"/><feComposite id="eseIaTOU95c10-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c10-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c10-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c10-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter><filter id="eseIaTOU95c17-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c17-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="3,3"/><feOffset id="eseIaTOU95c17-filter-drop-shadow-0-offset" dx="0" dy="0" result="tmp"/><feFlood id="eseIaTOU95c17-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.2)"/><feComposite id="eseIaTOU95c17-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c17-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c17-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c17-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter><filter id="eseIaTOU95c23-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c23-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="3,3"/><feOffset id="eseIaTOU95c23-filter-drop-shadow-0-offset" dx="0" dy="0" result="tmp"/><feFlood id="eseIaTOU95c23-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.2)"/><feComposite id="eseIaTOU95c23-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c23-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c23-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c23-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter><filter id="eseIaTOU95c31-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c31-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="3,3"/><feOffset id="eseIaTOU95c31-filter-drop-shadow-0-offset" dx="0" dy="0" result="tmp"/><feFlood id="eseIaTOU95c31-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.2)"/><feComposite id="eseIaTOU95c31-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c31-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c31-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c31-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter><filter id="eseIaTOU95c36-filter" x="-150%" width="400%" y="-150%" height="400%"><feGaussianBlur id="eseIaTOU95c36-filter-drop-shadow-0-blur" in="SourceAlpha" stdDeviation="5,5"/><feOffset id="eseIaTOU95c36-filter-drop-shadow-0-offset" dx="5" dy="5" result="tmp"/><feFlood id="eseIaTOU95c36-filter-drop-shadow-0-flood" flood-color="rgba(0,0,0,0.5)"/><feComposite id="eseIaTOU95c36-filter-drop-shadow-0-composite" operator="in" in2="tmp"/><feMerge id="eseIaTOU95c36-filter-drop-shadow-0-merge" result="result"><feMergeNode id="eseIaTOU95c36-filter-drop-shadow-0-merge-node-1"/><feMergeNode id="eseIaTOU95c36-filter-drop-shadow-0-merge-node-2" in="SourceGraphic"/></feMerge></filter></defs><g id="eseIaTOU95c2_to" transform="translate(249.999992,193.51566)"><g id="eseIaTOU95c2_ts" transform="scale(1,1)"><g id="eseIaTOU95c2" transform="translate(-264.846,-186.250955)" filter="url(#eseIaTOU95c2-filter)"><circle r="8.83883" transform="translate(264.846 186.250955)" fill="#714b67" stroke-linecap="round" stroke-linejoin="bevel"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="21" font-weight="300" transform="translate(258.84965 193.577679)" fill="#fff" stroke="#fff" stroke-linecap="round" stroke-linejoin="bevel"><tspan y="0" font-weight="300"><![CDATA[
+
]]></tspan></text></g></g></g><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" transform="translate(151.04633 86.432503)" fill="#fff" stroke="#d8dadd" stroke-width="0.702" stroke-linejoin="bevel"><tspan x="0" y="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" font-style="normal" fill="#8f8f8b" stroke="#8f8f8f" stroke-width="0.702" stroke-linejoin="bevel"><![CDATA[
When...
]]></tspan></text><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" transform="translate(151.29242 162.08133)" fill="#fff" stroke="#d8dadd" stroke-width="0.702" stroke-linejoin="bevel"><tspan x="0" y="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" font-style="normal" fill="#8f8f8b" stroke="#8f8f8f" stroke-width="0.702" stroke-linejoin="bevel"><![CDATA[
Then...
]]></tspan></text><g transform="translate(0-4)" filter="url(#eseIaTOU95c10-filter)"><path d="M170.95871,110.10226h143.01489c2.951189,0,5.3436,2.392411,5.3436,5.3436v30.70274c-.000004,2.956288-2.396552,5.352831-5.35284,5.35283h-143.00375c-1.416658.000002-2.775294-.562762-3.777023-1.564489s-1.564496-2.360363-1.564497-3.777021v-30.71804c0-2.948991,2.390629-5.33962,5.33962-5.33962Z" transform="matrix(1.39543 0 0 0.996657-79.4374-9.33412)" fill="#fff" stroke-width="1.00157" stroke-linecap="round"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" transform="translate(182.903 124.12703)" fill="#21272b" stroke="#21272b" stroke-width="0.3" stroke-linejoin="bevel"><tspan y="0" font-weight="300" stroke-width="0.3"><![CDATA[
Task moved to "In Progress"
]]></tspan></text><path d="M167.38343,121.01484h5.08035l3.49159-.007l1.44566-.003" fill="none" stroke="#8f8f8f" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="bevel"/><path d="M172.92996,116.57774l4.42926,4.42926-4.43496,4.43495" fill="none" stroke="#8f8f8f" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="bevel"/></g><g id="eseIaTOU95c16" opacity="0"><path d="M170.95871,110.10226h143.01489c2.951189,0,5.3436,2.392411,5.3436,5.3436v30.70274c-.000004,2.956288-2.396552,5.352831-5.35284,5.35283h-143.00375c-1.416658.000002-2.775294-.562762-3.777023-1.564489s-1.564496-2.360363-1.564497-3.777021v-30.71804c0-2.948991,2.390629-5.33962,5.33962-5.33962Z" transform="matrix(1.39543 0 0 0.996657-79.4374 62.2969)" filter="url(#eseIaTOU95c17-filter)" fill="#fff" stroke-width="1.00157" stroke-linecap="round"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" transform="translate(182.90302 195.77586)" fill="#21272b" stroke="#21272b" stroke-width="0.3" stroke-linejoin="bevel"><tspan y="0" font-weight="300" stroke-width="0.3"><![CDATA[
Send email to customer
]]></tspan></text><path d="M177.6078,195.97681h-13.31291v-8.41956h13.33484Z" fill="#fff" stroke="#8f8f8f" stroke-width="1.158" stroke-linecap="round" stroke-linejoin="round"/><path d="M164.50881,188.08763l6.42249,3.95297l6.48409-3.99089" fill="#fff" stroke="#8f8f8f" stroke-width="1.158" stroke-linecap="round" stroke-linejoin="round"/></g><g id="eseIaTOU95c22" opacity="0"><path d="M170.95871,110.10226h143.01489c2.951189,0,5.3436,2.392411,5.3436,5.3436v30.70274c-.000004,2.956288-2.396552,5.352831-5.35284,5.35283h-143.00375c-1.416658.000002-2.775294-.562762-3.777023-1.564489s-1.564496-2.360363-1.564497-3.777021v-30.71804c0-2.948991,2.390629-5.33962,5.33962-5.33962Z" transform="matrix(1.39543 0 0 0.996657-79.4374 109.523)" filter="url(#eseIaTOU95c23-filter)" fill="#fff" stroke-width="1.00157" stroke-linecap="round"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="100" transform="translate(182.903 243.01389)" fill="#21272b" stroke="#21272b" stroke-width="0.3" stroke-linejoin="bevel"><tspan y="0" font-weight="100" stroke-width="0.3"><![CDATA[
Add Anita Oliver as follower
]]></tspan></text><path d="M169.84289,230.5h-3.52181c-.760802.000001-1.42675.511002-1.62366,1.24588l-.59745,2.2297c-.22774.84994.30071,1.55792,1.18042,1.57723c1.53183.0336,3.83498.0755,5.36941.0632.88551-.007,1.4338-.80577,1.23338-1.67565-.16537-.71774-.3571-1.55325-.5102-2.22116-.15434-.67333-.83926-1.21922-1.53009-1.21922Z" transform="matrix(1.23338 0 0 1.23338-38.2249-46.1487)" fill="#8f8f8f" stroke-width="1.158" stroke-linecap="round" stroke-linejoin="round"/><circle r="2.89072" transform="translate(169.021 235.485)" fill="#8f8f8f" stroke="#fff" stroke-width="0.6" stroke-linecap="round" stroke-linejoin="round"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12.9644" font-weight="800" transform="translate(170.46661 245.19441)" fill="#8f8f8f" stroke-width="0.7" stroke-linecap="round" stroke-linejoin="round"><tspan x="0" y="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12.9644" font-weight="800" font-style="normal" fill="#8f8f8f" stroke="#fff" stroke-width="0.7" stroke-linecap="round" stroke-linejoin="round"><![CDATA[
+
]]></tspan></text></g><g id="eseIaTOU95c30" opacity="0"><path d="M170.95871,110.10226h143.01489c2.951189,0,5.3436,2.392411,5.3436,5.3436v30.70274c-.000004,2.956288-2.396552,5.352831-5.35284,5.35283h-143.00375c-1.416658.000002-2.775294-.562762-3.777023-1.564489s-1.564496-2.360363-1.564497-3.777021v-30.71804c0-2.948991,2.390629-5.33962,5.33962-5.33962Z" transform="matrix(1.39543 0 0 0.996657-79.4374 156.75)" filter="url(#eseIaTOU95c31-filter)" fill="#fff" stroke-width="1.00157" stroke-linecap="round"/><text dx="0" dy="0" font-family="&quot;eseIaTOU95c1:::Open Sans&quot;" font-size="12" font-weight="300" transform="translate(182.903 289.99326)" fill="#21272b" stroke="#21272b" stroke-width="0.3" stroke-linejoin="bevel"><tspan y="0" font-weight="300" stroke-width="0.3"><![CDATA[
Create activity "Follow Up"
]]></tspan></text><circle r="5.34677" transform="translate(170.962 286.888)" fill="none" stroke="#8f8f8f" stroke-width="1.00645" stroke-linecap="round" stroke-linejoin="round"/><path d="M168.22865,286.8819h2.73363v-3.50469" fill="none" stroke="#8f8f8f" stroke-width="0.83871" stroke-linecap="round" stroke-linejoin="round"/></g><g id="eseIaTOU95c36_to" transform="translate(455.595461,229.469635)"><g id="eseIaTOU95c36_ts" transform="scale(0.575337,0.575337)"><path d="M238.41873,141.99146l4.83741,8.37864-4.13635,2.38811-4.80868-8.39522-5.99562,6.31129-4.1736-28.62671l22.49349,18.12178Z" transform="translate(-224.141889,-122.04757)" filter="url(#eseIaTOU95c36-filter)" fill="#fff" stroke="#484848" stroke-width="1.96473" stroke-linecap="round" stroke-linejoin="round"/></g></g>
<style><![CDATA[
@font-face {font-family: 'eseIaTOU95c1:::Open Sans';font-style: normal;font-weight: 300;font-stretch: normal;src: url(data:font/ttf;charset=utf-8;base64,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) format('truetype');}@font-face {font-family: 'eseIaTOU95c1:::Open Sans';font-style: normal;font-weight: 800;font-stretch: normal;src: url(data:font/ttf;charset=utf-8;base64,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) format('truetype');}
]]></style>
</svg>

```

## File: static\src\base_automation_actions_one2many_field.js

```javascript
/** @odoo-module **/

import { Component, useExternalListener, useEffect, useRef } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { useThrottleForAnimation } from "@web/core/utils/timing";

class ActionsOne2ManyField extends Component {
    static props = ["*"];
    static template = "base_automation.ActionsOne2ManyField";
    static actionStates = {
        code: _t("Execute Python Code"),
        object_create: _t("Create a new Record"),
        object_write: _t("Update the Record"),
        multi: _t("Execute several actions"),
        mail_post: _t("Send email"),
        followers: _t("Add followers"),
        remove_followers: _t("Remove followers"),
        next_activity: _t("Create next activity"),
        sms: _t("Send SMS"),
    };
    setup() {
        this.root = useRef("root");

        let adaptCounter = 0;
        useEffect(
            () => {
                this.adapt();
            },
            () => [adaptCounter]
        );
        const throttledRenderAndAdapt = useThrottleForAnimation(() => {
            adaptCounter++;
            this.render();
        });
        useExternalListener(window, "resize", throttledRenderAndAdapt);
        this.currentActions = this.props.record.data[this.props.name].records;
        this.hiddenActionsCount = 0;
    }
    async adapt() {
        // --- Initialize ---
        // use getBoundingClientRect to get unrounded width
        // of the elements in order to avoid rounding issues
        const rootWidth = this.root.el.getBoundingClientRect().width;

        // remove all d-none classes (needed to get the real width of the elements)
        const actionsEls = Array.from(this.root.el.children).filter((el) => el.dataset.actionId);
        actionsEls.forEach((el) => el.classList.remove("d-none"));
        const actionsTotalWidth = actionsEls.reduce(
            (sum, el) => sum + el.getBoundingClientRect().width,
            0
        );

        // --- Check first overflowing action ---
        let overflowingActionId;
        if (actionsTotalWidth > rootWidth) {
            let width = 56; // for the ellipsis
            for (const el of actionsEls) {
                const elWidth = el.getBoundingClientRect().width;
                if (width + elWidth > rootWidth) {
                    // All the remaining elements are overflowing
                    overflowingActionId = el.dataset.actionId;
                    const firstOverflowingEl = actionsEls.find(
                        (el) => el.dataset.actionId === overflowingActionId
                    );
                    const firstOverflowingIndex = actionsEls.indexOf(firstOverflowingEl);
                    const overflowingEls = actionsEls.slice(firstOverflowingIndex);
                    // hide overflowing elements
                    overflowingEls.forEach((el) => el.classList.add("d-none"));
                    break;
                }
                width += elWidth;
            }
        }

        // --- Final rendering ---
        const initialHiddenActionsCount = this.hiddenActionsCount;
        this.hiddenActionsCount = overflowingActionId
            ? this.currentActions.length -
              this.currentActions.findIndex((action) => action.id === overflowingActionId)
            : 0;
        if (initialHiddenActionsCount !== this.hiddenActionsCount) {
            // Render only if hidden actions count has changed.
            return this.render();
        }
    }
    getActionType(action) {
        return this.constructor.actionStates[action.data.state] || action.data.state;
    }
    get moreText() {
        const isPlural = this.hiddenActionsCount > 1;
        return isPlural ? _t("%s actions", this.hiddenActionsCount) : _t("1 action");
    }
}

const actionsOne2ManyField = {
    component: ActionsOne2ManyField,
    relatedFields: [
        { name: "name", type: "char" },
        {
            name: "state",
            type: "selection",
            selection: [
                ["code", _t("Execute Python Code")],
                ["object_create", _t("Create a new Record")],
                ["object_write", _t("Update the Record")],
                ["multi", _t("Execute several actions")],
                ["mail_post", _t("Send email")],
                ["followers", _t("Add followers")],
                ["remove_followers", _t("Remove followers")],
                ["next_activity", _t("Create next activity")],
                ["sms", _t("Send SMS")],
            ],
        },
        // Execute Python Code
        { name: "code", type: "text" },
        // Create
        { name: "crud_model_id", type: "many2one" },
        { name: "crud_model_name", type: "char" },
        // Add Followers
        { name: "partner_ids", type: "many2many" },
        // Message Post / Email
        { name: "template_id", type: "many2one" },
        { name: "mail_post_autofollow", type: "boolean" },
        {
            name: "mail_post_method",
            type: "selection",
            selection: [
                ["email", _t("Email")],
                ["comment", _t("Post as Message")],
                ["note", _t("Post as Note")],
            ],
        },
        // Schedule Next Activity
        { name: "activity_type_id", type: "many2one" },
        { name: "activity_summary", type: "char" },
        { name: "activity_note", type: "html" },
        { name: "activity_date_deadline_range", type: "integer" },
        {
            name: "activity_date_deadline_range_type",
            type: "selection",
            selection: [
                ["days", _t("Days")],
                ["weeks", _t("Weeks")],
                ["months", _t("Months")],
            ],
        },
        {
            name: "activity_user_type",
            type: "selection",
            selection: [
                ["specific", _t("Specific User")],
                ["generic", _t("Generic User")],
            ],
        },
        { name: "activity_user_id", type: "many2one" },
        { name: "activity_user_field_name", type: "char" },
    ],
};

registry.category("fields").add("base_automation_actions_one2many", actionsOne2ManyField);

```

## File: static\src\base_automation_actions_one2many_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="base_automation.ActionsOne2ManyField">
        <div class="d-flex align-items-center" t-ref="root">
            <t t-if="currentActions.length === 0">
                <span class="text-muted">no action defined...</span>
            </t>
            <t t-foreach="currentActions" t-as="action" t-key="action.id">
                <div style="min-width: fit-content;" t-att-data-action-id="action.id">
                    <div class="fs-5 d-flex align-items-center">
                        <i
                            data-name="server_action_icon"
                            t-att-title="getActionType(action)"
                            class="fa"
                            t-att-class="{
                                'code': 'fa-file-code-o',
                                'object_create': 'fa-edit',
                                'object_write': 'fa-refresh',
                                'multi': 'fa-list-ul',
                                'mail_post': 'fa-envelope',
                                'followers': 'fa-user-o',
                                'remove_followers': 'fa-user-times',
                                'next_activity': 'fa-clock-o',
                                'sms': 'fa-comments-o',
                            }[action.data.state]"
                        />
                        <div class="ps-2" t-esc="action.data.name" />
                    </div>
                </div>
                <div t-if="!action_last" class="px-3 align-self-center" t-att-data-action-id="action.id">
                    <i class="fa fa-lg fa-plus text-primary"></i>
                </div>
            </t>
            <t t-if="hiddenActionsCount">
                <div class="fs-3 align-self-center text-muted">
                    <t t-out="moreText"/>
                </div>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\base_automation_error_dialog.js

```javascript
/** @odoo-module */

import { RPCErrorDialog } from "@web/core/errors/error_dialogs";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { user } from "@web/core/user";

export class BaseAutomationErrorDialog extends RPCErrorDialog {
    static template = "base_automation.ErrorDialog";
    setup() {
        super.setup(...arguments);
        const { id, name } = this.props.data.context.base_automation;
        this.automationId = id;
        this.automationName = name;
        this.isUserAdmin = user.isAdmin;
        this.actionService = useService("action");
        this.orm = useService("orm");
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * This method is called when the user clicks on the 'Disable Automation Rule' button
     * displayed when a crash occurs in the evaluation of an automation rule.
     * Then, we write `active` to `False` on the automation rule to disable it.
     *
     * @private
     * @param {MouseEvent} ev
     */
    async disableAutomation(ev) {
        await this.orm.write("base.automation", [this.automationId], { active: false });
        this.props.close();
    }
    /**
     * This method is called when the user clicks on the 'Edit action' button
     * displayed when a crash occurs in the evaluation of an automation rule.
     * Then, we redirect the user to the automation rule form.
     *
     * @private
     * @param {MouseEvent} ev
     */
    editAutomation(ev) {
        this.actionService.doAction({
            name: "Automation Rules",
            res_model: "base.automation",
            res_id: this.automationId,
            views: [[false, "form"]],
            type: "ir.actions.act_window",
            view_mode: "form",
            target: "new",
        });
        this.props.close();
    }
}

registry.category("error_dialogs").add("base_automation", BaseAutomationErrorDialog);

```

## File: static\src\base_automation_error_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="base_automation.ErrorDialog" t-inherit="web.ErrorDialog" t-inherit-mode="primary">
        <xpath expr="//div[@role='alert']" position="attributes">
            <attribute name="class" add="o_base_automation_error" separator=" "/>
        </xpath>
        <xpath expr="//div[@role='alert']" position="inside">
            <p class="mt-2">
                The error occurred during the execution of the automation rule
                "<t t-esc="automationName"/>"
                (ID: <t t-esc="automationId"/>).
                <br/>
            </p>
            <p t-if="isUserAdmin">
                You can disable this automation rule or edit it to solve the issue.<br/>
                Disabling this automation rule will enable you to continue your workflow
                but any data created after this could potentially be corrupted,
                as you are effectively disabling a customization that may set
                important and/or required fields.
            </p>
            <p t-else="">
                You can ask an administrator to disable or correct this automation rule.
            </p>
        </xpath>
        <xpath expr="//div[@role='alert']//button" position="after">
            <t t-if="isUserAdmin">
                <button class="btn btn-secondary mt4 o_disable_action_button me-3" t-on-click.prevent="disableAutomation">
                    <i class="fa fa-ban mr8"/>Disable Automation Rule
                </button>
                <button class="btn btn-secondary mt4 o_edit_action_button" t-on-click.prevent="editAutomation">
                    <i class="fa fa-edit mr8"/>Edit Automation Rule
                </button>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\base_automation_trigger_selection_field.js

```javascript
/** @odoo-module */

import { useState } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { useRecordObserver } from "@web/model/relational_model/utils";
import { selectionField, SelectionField } from "@web/views/fields/selection/selection_field";
import { TRIGGER_FILTERS } from "./utils";
import { useService } from "@web/core/utils/hooks";

const OPT_GROUPS = [
    {
        group: { sequence: 10, key: "values", name: _t("Values Updated") },
        triggers: [
            "on_stage_set",
            "on_user_set",
            "on_tag_set",
            "on_state_set",
            "on_priority_set",
            "on_archive",
            "on_unarchive",
        ],
    },
    {
        group: { sequence: 30, key: "timing", name: _t("Timing Conditions") },
        triggers: ["on_time", "on_time_created", "on_time_updated"],
    },
    {
        group: { sequence: 40, key: "custom", name: _t("Custom") },
        triggers: ["on_create_or_write", "on_unlink", "on_change"],
    },
    {
        group: { sequence: 50, key: "external", name: _t("External") },
        triggers: ["on_webhook"],
    },
    {
        group: { sequence: 20, key: "mail", name: _t("Email Events") },
        triggers: ["on_message_sent", "on_message_received"],
    },
    {
        group: { sequence: 60, key: "deprecated", name: _t("Deprecated (do not use)") },
        triggers: ["on_create", "on_write"],
    },
];

function computeDerivedOptions(options, fields, currentSelection, { excludeGroups = [] } = {}) {
    // filter options to display, derived from the current value and the model fields
    const derivedOptions = [];
    for (const [value, label] of options) {
        const { group, triggers } = OPT_GROUPS.find((g) => g.triggers.includes(value));
        if (
            (group.key === "deprecated" && !triggers.includes(currentSelection)) ||
            excludeGroups.includes(group.key)
        ) {
            // skip deprecated triggers if the current value is not deprecated
            continue;
        }
        const filterFn = TRIGGER_FILTERS[value];
        if (filterFn) {
            const triggerFields = fields.filter(filterFn);
            if (triggerFields.length === 0) {
                // skip triggers that don't have any corresponding field
                continue;
            }
        }

        const option = { group, value, label };
        derivedOptions.push(option);
    }
    return derivedOptions;
}

export class TriggerSelectionField extends SelectionField {
    static template = "base_automation.TriggerSelectionField";
    setup() {
        super.setup();
        this.groupedOptions = useState([]);

        const orm = useService("orm");
        let lastRelatedModelId;
        let relatedModelFields;
        useRecordObserver(async (record) => {
            const { data, fields } = record;
            const modelId = data.model_id?.[0];
            if (lastRelatedModelId !== modelId) {
                lastRelatedModelId = modelId;
                relatedModelFields = await orm.searchRead(
                    "ir.model.fields",
                    [["model_id", "=", modelId]],
                    ["field_description", "name", "ttype", "relation"]
                );
            }

            // first, compute the derived options
            const derivedOptions = computeDerivedOptions(
                fields[this.props.name].selection,
                relatedModelFields,
                data[this.props.name],
                { excludeGroups: data.model_is_mail_thread ? [] : ["mail"] }
            );

            // then group and sort them
            this.groupedOptions.length = 0;
            for (const option of derivedOptions) {
                const group = this.groupedOptions.find((g) => g.key === option.group.key) ?? {
                    ...option.group,
                    options: [],
                };
                group.options.push(option);
                if (!this.groupedOptions.includes(group)) {
                    this.groupedOptions.push(group);
                }
            }
            this.groupedOptions.sort((a, b) => a.sequence - b.sequence);
        });
    }
}

export const triggerSelectionField = {
    ...selectionField,
    component: TriggerSelectionField,
    fieldDependencies: [{ name: "model_is_mail_thread", type: "boolean" }],
};
registry.category("fields").add("base_automation_trigger_selection", triggerSelectionField);

```

## File: static\src\base_automation_trigger_selection_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="base_automation.TriggerSelectionField" t-inherit="web.SelectionField" t-inherit-mode="primary">
        <xpath expr="//t[@t-foreach='options']" position="replace">
            <t t-foreach="groupedOptions" t-as="group" t-key="group.key">
                <optgroup t-att-label="group.name">
                    <t t-foreach="group.options" t-as="option" t-key="option.value">
                        <option
                            t-att-selected="option.value === value"
                            t-att-value="stringify(option.value)"
                            t-esc="option.label"/>
                    </t>
                </optgroup>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\kanban_header_patch.js

```javascript
/* @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { user } from "@web/core/user";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";
import { KanbanHeader } from "@web/views/kanban/kanban_header";
import { TRIGGER_FILTERS } from "./utils";

const SUPPORTED_TRIGGERS = [
    "on_stage_set",
    "on_tag_set",
    "on_state_set",
    "on_priority_set",
    "on_user_set",
    "on_archive",
];

function enrichContext(context, group) {
    const { displayName, groupByField, value } = group;
    const { name, relation, type: ttype } = groupByField;
    for (const trigger of SUPPORTED_TRIGGERS) {
        if (!TRIGGER_FILTERS[trigger]({ name, relation, ttype })) {
            continue;
        }
        switch (trigger) {
            case "on_stage_set":
                return {
                    ...context,
                    default_trigger: trigger,
                    default_name: _t('Stage is set to "%s"', displayName),
                    default_trg_field_ref: value,
                };
            case "on_tag_set":
                return {
                    ...context,
                    default_trigger: trigger,
                    default_name: _t('"%s" tag is added', displayName),
                    default_trg_field_ref: value,
                };
            default:
                return { ...context, default_trigger: trigger };
        }
    }

    // Default trigger
    return { ...context, default_trigger: "on_create_or_write" };
}

patch(KanbanHeader.prototype, {
    setup() {
        super.setup();
        this.action = useService("action");
    },

    /**
     * @override
     */
    get permissions() {
        const permissions = super.permissions;
        Object.defineProperty(permissions, "canEditAutomations", {
            get: () => user.isAdmin,
            configurable: true,
        });
        return permissions;
    },

    async openAutomations() {
        return this._openAutomations();
    },

    async _openAutomations() {
        const domain = [["model", "=", this.props.list.resModel]];
        const modelId = await this.orm.search("ir.model", domain, { limit: 1 });
        const context = {
            active_test: false,
            default_model_id: modelId[0],
            search_default_model_id: modelId[0],
        };
        this.action.doAction("base_automation.base_automation_act", {
            additionalContext: enrichContext(context, this.group),
        });
    },
});

registry.category("kanban_header_config_items").add(
    "open_automations",
    {
        label: _t("Automations"),
        method: "openAutomations",
        isVisible: ({ permissions }) => permissions.canEditAutomations,
        class: "o_column_automations",
    },
    { sequence: 25, force: true }
);

```

## File: static\src\utils.js

```javascript
/** @odoo-module */

export const TRIGGER_FILTERS = {
    on_create_or_write: (f) => true,
    on_create: (f) => true,
    on_write: (f) => true,
    on_change: (f) => true,
    on_unlink: (f) => true,
    on_time: (f) => true,
    on_time_created: (f) => f.ttype === "datetime" && f.name === "create_date",
    on_time_updated: (f) => f.ttype === "datetime" && f.name === "write_date",
    on_stage_set: (f) =>
        f.ttype === "many2one" && ["stage_id", "x_studio_stage_id"].includes(f.name),
    on_user_set: (f) =>
        f.relation === "res.users" &&
        ["many2one", "many2many"].includes(f.ttype) &&
        ["user_id", "user_ids", "x_studio_user_id", "x_studio_user_ids"].includes(f.name),
    on_tag_set: (f) => f.ttype === "many2many" && ["tag_ids", "x_studio_tag_ids"].includes(f.name),
    on_state_set: (f) => f.ttype === "selection" && ["state", "x_studio_state"].includes(f.name),
    on_priority_set: (f) =>
        f.ttype === "selection" && ["priority", "x_studio_priority"].includes(f.name),
    on_archive: (f) => f.ttype === "boolean" && ["active", "x_active"].includes(f.name),
    on_unarchive: (f) => f.ttype === "boolean" && ["active", "x_active"].includes(f.name),
    on_webhook: (f) => true,
};

```

## File: views\base_automation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Automation Form View -->
        <record id="view_base_automation_form" model="ir.ui.view">
            <field name="name">Automations</field>
            <field name="model">base.automation</field>
            <field name="arch" type="xml">
                <form string="Automation Rule">
                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_view_webhook_logs" type="object" string="Logs" class="oe_stat_button" icon="fa-list" invisible="trigger != 'on_webhook'">
                            </button>
                        </div>
                        <field name="active" invisible="1" />
                        <field name="model_name" invisible="1" force_save="True" />
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" invisible="active"/>
                        <div class="oe_title">
                            <h1><field name="name" placeholder="e.g. Support flow"/></h1>
                        </div>
                        <group groups="!base.group_no_one" invisible="context.get('default_model_id')">
                            <group>
                                <field name="model_id" options="{'no_create': True}" />
                            </group>
                        </group>
                        <group groups="base.group_no_one">
                            <group>
                                <field name="model_id" options="{'no_create': True}" />
                            </group>
                        </group>
                        <group invisible="not model_id">
                            <group>
                                <label for="trigger"/>
                                <div>
                                    <div class="d-flex flex-row">
                                        <field name="trigger" widget="base_automation_trigger_selection" class="oe_inline me-3"/>
                                        <field name="trg_selection_field_id" placeholder="Select a value..." class="oe_inline"
                                            options="{'no_open': True, 'no_create': True}"
                                            invisible="trigger not in ['on_state_set', 'on_priority_set']"
                                            required="trigger in ['on_state_set', 'on_priority_set']"
                                        />
                                        <field name="trg_field_ref" placeholder="Select a value..." class="oe_inline"
                                            options="{'no_open': True, 'no_create': True}"
                                            invisible="trigger not in ['on_stage_set', 'on_tag_set']"
                                            required="trigger in ['on_stage_set', 'on_tag_set']"
                                        />
                                        <field name="trg_field_ref_model_name" invisible="1" />
                                        <field name="trg_date_id" class="oe_inline" string="Date Field"
                                            options="{'no_open': True, 'no_create': True}"
                                                invisible="trigger != 'on_time'"
                                                required="trigger in ['on_time', 'on_time_created', 'on_time_updated']"
                                        />
                                    </div>
                                    <div class="text-muted" invisible="trigger != 'on_change'"><i class="fa fa-warning"/> Automation rules triggered by UI changes will be executed <em>every time</em> the watched fields change, <em>whether you save or not</em>.</div>
                                </div>
                                <label for="url" string="URL" invisible="trigger != 'on_webhook'"/>
                                <div invisible="trigger != 'on_webhook'">
                                    <field name="url" widget="CopyClipboardURL"  placeholder="URL will be created once the rule is saved."/>
                                    <div class="alert alert-warning" role="status">
                                        <strong><i class="fa fa-lock"/> Keep it secret, keep it safe.</strong>
                                        <p>Your webhook URL contains a secret. Don't share it online or carelessly.</p>
                                        <button class="btn btn-seconadry" type="object" name="action_rotate_webhook_uuid" string="Rotate Secret" icon="fa-refresh" help="Change the URL's secret if you think the URL is no longer secure. You will have to update any automated system that calls this webhook to the new URL."/>
                                    </div>
                                </div>
                                <label for="trg_date_range" string="Delay" invisible="trigger not in ['on_time', 'on_time_created', 'on_time_updated']"/>
                                <div class="d-flex flex-row gap-2" invisible="trigger not in ['on_time', 'on_time_created', 'on_time_updated']">
                                    <field name="trg_date_range" class="oe_inline" required="trigger in ['on_time', 'on_time_created', 'on_time_updated']" />
                                    <field name="trg_date_range_type" class="oe_inline" required="trigger in ['on_time', 'on_time_created', 'on_time_updated']" />
                                    <span invisible="trigger != 'on_time_created'">after creation</span>
                                    <span invisible="trigger != 'on_time_updated'">after last update</span>
                                    <span invisible="trigger != 'on_time'">after</span>
                                    <field name="trg_date_id" class="oe_inline" string="Date Field" placeholder="Select a date field..."
                                        options="{'no_open': True, 'no_create': True}"
                                        context="{'hide_model': 1}"
                                        invisible="trigger != 'on_time'"
                                        required="trigger in ['on_time', 'on_time_created', 'on_time_updated']"/>
                                </div>
                                <field name="log_webhook_calls" widget="boolean_toggle" invisible="trigger != 'on_webhook'"/>
                                <field name="trg_date_calendar_id" class="oe_inline"
                                    options="{'no_open': True, 'no_create': True}"
                                    invisible="not trg_date_id or trg_date_range_type != 'day'"/>
                                <label for="least_delay_msg" invisible="trigger not in ['on_time', 'on_time_created', 'on_time_updated'] or not least_delay_msg" string="" />
                                <div class="alert alert-info" role="alert" invisible="trigger not in ['on_time', 'on_time_created', 'on_time_updated'] or not least_delay_msg">
                                    <field name="least_delay_msg"/>
                                </div>
                                <field name="filter_pre_domain" widget="domain" groups="base.group_no_one"
                                       options="{'model': 'model_name', 'in_dialog': True}"
                                       invisible="trigger in [
                                            'on_create',
                                            'on_unlink',
                                            'on_change',
                                            'on_webhook',
                                            'on_time',
                                            'on_time_created',
                                            'on_time_updated'
                                        ]"
                                />
                                <field name="filter_domain" widget="domain" groups="base.group_no_one"
                                    options="{'model': 'model_name', 'in_dialog': True}"
                                    invisible="trigger in ['on_change', 'on_webhook']"
                                />
                                <label for="filter_domain" groups="!base.group_no_one"
                                    invisible="trigger not in ['on_create_or_write', 'on_unlink']"
                                />
                                <label for="filter_domain" groups="!base.group_no_one"
                                    string="Extra Conditions"
                                    invisible="trigger not in ['on_time', 'on_time_created', 'on_time_updated']"
                                />
                                <field name="filter_domain" nolabel="1" widget="domain"
                                    groups="!base.group_no_one"
                                    options="{'model': 'model_name', 'in_dialog': False, 'foldable': True}"
                                    invisible="trigger not in ['on_create_or_write', 'on_unlink', 'on_time', 'on_time_created', 'on_time_updated']"
                                />
                                <field name="trigger_field_ids" string="When updating" placeholder="Select fields..."
                                    options="{'no_open': True, 'no_create': True}"
                                    domain="[('model_id', '=', model_id),('store','=',True)]"
                                    context="{'hide_model': 1}"
                                    invisible="trigger != 'on_create_or_write'" widget="many2many_tags" />
                                <field name="on_change_field_ids" string="When updating" placeholder="Select fields..."
                                    options="{'no_open': True, 'no_create': True}"
                                    domain="[('model_id', '=', model_id)]"
                                    context="{'hide_model': 1}"
                                    invisible="trigger != 'on_change'" widget="many2many_tags" />
                            </group>
                            <group>
                                <label for="record_getter" string="Target Record" invisible="trigger != 'on_webhook'" />
                                <div invisible="trigger != 'on_webhook'">
                                    <field name="record_getter" string="Target Record"/>
                                    <div>
                                        <div  class="text-muted"><i class="fa fa-info-circle"/> The default target record getter will work out-of-the-box for any webhook coming from another Odoo instance.</div>
                                        <span class="text-muted"> Available variables: </span>
                                        <ul class="text-muted">
                                            <li><code>env</code>: environment on which the action is triggered</li>
                                            <li><code>model</code>: model of the record on which the action is triggered; is a void recordset</li>
                                            <li><code>time</code>, <code>datetime</code>, <code>dateutil</code>, <code>timezone</code>: useful Python libraries</li>
                                            <li><code>payload</code>: the payload of the call (GET parameters, JSON body), as a dict.</li>
                                        </ul>
                                    </div>
                                </div>
                            </group>
                        </group>
                        <notebook invisible="not model_id">
                            <page string="Actions To Do" name="actions">
                                <field
                                    name="action_server_ids"
                                    widget="one2many"
                                    class="o_base_automation_actions_field"
                                    context="{'default_model_id': model_id, 'form_view_ref': 'base_automation.ir_actions_server_view_form_automation'}"
                                >
                                    <kanban>
                                        <control>
                                            <create string="Add an action" />
                                        </control>
                                        <field name="state"/>
                                        <field name="evaluation_type"/>
                                        <field name="value"/>
                                        <field name="value_field_to_show"/>
                                        <field name="update_field_type"/>
                                        <field name="update_m2m_operation"/>
                                        <templates>
                                            <t t-name="card" class="flex-row align-items-center gap-1">
                                                <field name="sequence" widget="handle" class="px-1" />
                                                <!-- Icon section -->
                                                <i
                                                    data-name="server_action_icon"
                                                    t-att-title="record.state.value"
                                                    class="fa fa-fw"
                                                    t-att-class="{
                                                        'code': 'fa-file-code-o',
                                                        'object_create': 'fa-edit',
                                                        'object_write': 'fa-refresh',
                                                        'multi': 'fa-list-ul',
                                                        'mail_post': 'fa-envelope',
                                                        'followers': 'fa-user-o',
                                                        'remove_followers': 'fa-user-times',
                                                        'next_activity': 'fa-clock-o',
                                                        'sms': 'fa-comments-o',
                                                        'webhook': 'fa-paper-plane',
                                                    }[record.state.raw_value]"
                                                />
                                                <field name="name" class="text-truncate" />
                                                <t invisible="state != 'object_write'">
                                                    <t invisible="not (update_field_type == 'many2many' and update_m2m_operation == 'clear') or evaluation_type != 'value'">
                                                        <span>by clearing it</span>
                                                    </t>
                                                    <t invisible="not (update_field_type == 'many2many' and update_m2m_operation == 'add') or evaluation_type != 'value'">
                                                        <span>by adding</span><field name="resource_ref"/>
                                                    </t>
                                                    <t invisible="not (update_field_type == 'many2many' and update_m2m_operation == 'remove') or evaluation_type != 'value'">
                                                        <span>by removing</span><field name="resource_ref"/>
                                                    </t>
                                                    <t invisible="not (update_field_type == 'many2many' and update_m2m_operation == 'set') or evaluation_type != 'value'">
                                                        <span>by setting it to</span><field name="resource_ref"/>
                                                        </t>
                                                    <t invisible="update_field_type == 'many2many' and evaluation_type == 'value'">
                                                        <span invisible="evaluation_type != 'value'">to</span>
                                                        <span invisible="evaluation_type != 'equation'">as</span>
                                                        <field name="resource_ref" invisible="not (value_field_to_show == 'resource_ref' and evaluation_type == 'value')" />
                                                        <field name="selection_value" invisible="not (value_field_to_show == 'selection_value' and evaluation_type == 'value')" class="d-inline"/>
                                                        <field name="update_boolean_value" invisible="not (value_field_to_show == 'update_boolean_value' and evaluation_type == 'value')" class="d-inline"/>
                                                        <em invisible="not (value_field_to_show == 'value' and evaluation_type == 'value')" class="d-inline"><field name="value"/></em>
                                                        <code invisible="not (evaluation_type == 'equation')"><field name="value" /></code>
                                                    </t>
                                                </t>
                                                <button type="delete" name="delete" class="btn fa fa-trash fa-xl px-3 ms-auto" title="Delete Action" />
                                            </t>
                                        </templates>
                                    </kanban>
                                </field>
                            </page>
                            <page string="Notes" name="notes">
                                <field name="description" placeholder="Keep track of what this automation does and why it exists..."/>
                            </page>
                        </notebook>
                    </sheet>
                </form>
            </field>
        </record>

        <!-- automation List View -->
        <record id="view_base_automation_tree" model="ir.ui.view">
            <field name="name">base.automation.list</field>
            <field name="model">base.automation</field>
            <field name="arch" type="xml">
                <list string="Automation Rules">
                    <field name="name"/>
                    <field name="trigger"/>
                    <field name="model_id"/>
                </list>
            </field>
        </record>

        <!-- automation Kanban View -->
        <record id="view_base_automation_kanban" model="ir.ui.view">
            <field name="name">base.automation.kanban</field>
            <field name="model">base.automation</field>
            <field name="arch" type="xml">
                <kanban
                    string="Automation Rules"
                    class="o_base_automation_kanban_view"
                    records_draggable="false"
                    groups_draggable="false"
                    quick_create="false"
                    group_create="false"
                    group_edit="false"
                    group_delete="false"
                >
                    <field name="active"/>
                    <templates>
                        <t t-name="card" class="flex-md-row">
                            <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active" />
                            <div class="d-flex align-items-center w-100 w-md-25 o_automation_base_info">
                                <div class="d-flex flex-column">
                                    <field name="name" class="fs-2 fw-bold"/>
                                    <field name="model_id" invisible="context.get('default_model_id')"/>
                                    <div class="d-flex align-items-center gap-1" invisible="trigger in ['on_time', 'on_time_created', 'on_time_updated']">
                                        <field name="trigger" />
                                        <field name="on_change_field_ids" invisible="trigger != 'on_change'" class="my-1" />
                                        <field name="trg_selection_field_id" invisible="trigger not in ['on_state_set', 'on_priority_set']" class="o_tag o_tag_color_0 rounded-pill p-1 px-2" />
                                        <field name="trg_field_ref" invisible="trigger not in ['on_stage_set', 'on_tag_set']" class="o_tag o_tag_color_0 rounded-pill p-1 px-2" />
                                        <field name="trigger_field_ids" invisible="trigger not in ['on_create_or_write']" class="my-1" />
                                    </div>
                                    <div class="d-flex align-items-center gap-1" invisible="trigger != 'on_time'">
                                        <field name="trg_date_range"/>
                                        <field name="trg_date_range_type" class="text-lowercase text-nowrap" />
                                        <span class="flex-shrink-0">
                                            based on <field name="trg_date_id" invisible="trigger != 'on_time'"  class="o_tag o_tag_color_0 rounded-pill p-1 px-2" />
                                        </span>
                                    </div>
                                    <div class="d-flex align-items-center gap-1" invisible="trigger not in ['on_time_created', 'on_time_updated']">
                                        <field name="trg_date_range"/>
                                        <field name="trg_date_range_type" class="text-lowercase" />
                                        <field name="trigger" class="text-lowercase" />
                                        <field name="trg_date_id" invisible="trigger != 'on_time'"  class="o_tag o_tag_color_0 rounded-pill p-1 px-2" />
                                    </div>
                                    <field name="trg_date_calendar_id" invisible="not trg_date_id or trg_date_range_type != 'day'" />
                                </div>
                            </div>
                            <div class="d-none d-md-flex flex-grow-1 align-items-center gap-3 o_automation_actions" data-name="more-info">
                                <i class="fa fa-2x fa-arrow-right text-primary" title="Actions" />
                                <field name="action_server_ids" widget="base_automation_actions_one2many" class="align-self-center w-100 me-md-3" />
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="view_base_automation_search" model="ir.ui.view">
            <field name="name">base.automation.search</field>
            <field name="model">base.automation</field>
            <field name="arch" type="xml">
                <search>
                    <field name="name"/>
                    <field name="model_id"/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <!-- automation Action -->
        <record id="base_automation_act" model="ir.actions.act_window">
            <field name="name">Automation Rules</field>
            <field name="res_model">base.automation</field>
            <field name="path">automations</field>
            <field name="view_mode">kanban,list,form</field>
            <field name="view_id" ref="view_base_automation_kanban"/>
            <field name="context">{'active_test': False}</field>
            <field name="help" type="html">
                <img class="w-100 w-md-75"
                src="/base_automation/static/img/automation.svg" />
              <p>
                Automate <em>everything</em> with Automation Rules
              </p><p>
                Send an email when an object changes state, archive records
                after a month of inactivity or remind yourself to follow-up on
                tasks when a specific tag is added.
              </p><p>With Automation Rules, you can automate
                <em>any</em> workflow.
              </p>
            </field>
        </record>

        <menuitem id="menu_base_automation_form"
            parent="base.menu_automation" action="base_automation_act" sequence="1"/>

        </odoo>

```

## File: views\ir_actions_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Automation: Server Action Form View -->
    <record id="ir_actions_server_view_form_automation" model="ir.ui.view">
        <field name="name">ir.actions.server.view.form.automation</field>
        <field name="model">ir.actions.server</field>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="inherit_id" ref="base.view_server_action_form"/>
        <field name="arch" type="xml">
            <xpath expr='//field[@name="name"]' position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr='//field[@name="state"]' position="attributes">
                <attribute name="string">Type</attribute>
            </xpath>
            <xpath expr="//header" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='model_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

