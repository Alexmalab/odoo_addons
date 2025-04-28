# Odoo Module: base_automation

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from .tests import test_models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Automated Action Rules',
    'version': '1.0',
    'category': 'Sales/Sales',
    'description': """
This module allows to implement action rules for any object.
============================================================

Use automated actions to automatically trigger actions for various screens.

**Example:** A lead created by a specific user may be automatically set to a specific
Sales Team, or an opportunity which still has status pending after 14 days might
trigger an automatic reminder email.
    """,
    'depends': ['base', 'resource', 'mail'],
    'data': [
        'security/ir.model.access.csv',
        'data/base_automation_data.xml',
        'views/base_automation_view.xml',
    ],
    'demo': [
        'data/base_automation_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\base_automation_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="ir_cron_data_base_automation_check" model="ir.cron">
            <field name="name">Base Action Rule: check and execute</field>
            <field name="model_id" ref="model_base_automation"/>
            <field name="state">code</field>
            <field name="code">model._check(True)</field>
            <field name="interval_number">4</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
            <field name="active" eval="False" />
        </record>
    </data>
</odoo>

```

## File: data\base_automation_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="test_rule_on_create" model="base.automation">
            <field name="name">Base Automation: test rule on create</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="state">code</field>
            <field name="code" eval="'records.write({\'user_id\': %s})' % ref('base.user_demo')"/>
            <field name="trigger">on_create</field>
            <field name="active" eval="True"/>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="filter_domain">[('state', '=', 'draft')]</field>
        </record>

        <record id="test_rule_on_write" model="base.automation">
            <field name="name">Base Automation: test rule on write</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="state">code</field>
            <field name="code" eval="'records.write({\'user_id\': %s})' % ref('base.user_demo')"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="filter_domain">[('state', '=', 'done')]</field>
            <field name="filter_pre_domain">[('state', '=', 'open')]</field>
        </record>

        <record id="test_rule_on_recompute" model="base.automation">
            <field name="name">Base Automation: test rule on recompute</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="state">code</field>
            <field name="code" eval="'records.write({\'user_id\': %s})' % ref('base.user_demo')"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="filter_domain">[('employee', '=', True)]</field>
        </record>

        <record id="test_rule_recursive" model="base.automation">
            <field name="name">Base Automation: test recursive rule</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="state">code</field>
            <field name="code">
record = model.browse(env.context['active_id'])
if 'partner_id' in env.context['old_values'][record.id]:
    record.write({'state': 'draft'})
            </field>
        </record>

        <record id="test_rule_on_line" model="base.automation">
            <field name="name">Base Automation: test rule on secondary model</field>
            <field name="model_id" ref="base_automation.model_base_automation_line_test"/>
            <field name="state">code</field>
            <field name="code" eval="'records.write({\'user_id\': %s})' % ref('base.user_demo')"/>
            <field name="trigger">on_create</field>
            <field name="active" eval="True"/>
            <field name="model_id" ref="base_automation.model_base_automation_line_test"/>
        </record>

        <record id="test_rule_on_write_check_context" model="base.automation">
            <field name="name">Base Automation: test rule on write check context</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="state">code</field>
            <field name="code">
record = model.browse(env.context['active_id'])
if 'user_id' in env.context['old_values'][record.id]:
    record.write({'is_assigned_to_admin': (record.user_id.id == 1)})
            </field>
        </record>

        <record id="test_rule_with_trigger" model="base.automation">
            <field name="name">Base Automation: test rule with trigger</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="trigger_field_ids" eval="[(4,ref('base_automation.field_base_automation_lead_test__state'))]"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="state">code</field>
            <field name="code">
record = model.browse(env.context['active_id'])
record['name'] = record.name + 'X'
            </field>
        </record>

        <record id="test_mail_template_automation" model="mail.template">
            <field name="name">Template Automation</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="body_html">&lt;div&gt;Email automation&lt;/div&gt;</field>
        </record>
        <record id="test_rule_on_write_recompute_send_email" model="base.automation">
            <field name="name">Base Automation: test send an email</field>
            <field name="model_id" ref="base_automation.model_base_automation_lead_test"/>
            <field name="template_id" ref="base_automation.test_mail_template_automation"/>
            <field name="state">email</field>
            <field name="trigger_field_ids" eval="[(4,ref('base_automation.field_base_automation_lead_test__deadline'))]"/>
            <field name="trigger">on_write</field>
            <field name="active" eval="True"/>
            <field name="filter_domain">[('deadline', '!=', False)]</field>
            <field name="filter_pre_domain">[('deadline', '=', False)]</field>
        </record>
</odoo>

```

## File: models\base_automation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
import time
import traceback
from collections import defaultdict

import dateutil
from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models, SUPERUSER_ID
from odoo.modules.registry import Registry
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.safe_eval import safe_eval

_logger = logging.getLogger(__name__)

DATE_RANGE_FUNCTION = {
    'minutes': lambda interval: relativedelta(minutes=interval),
    'hour': lambda interval: relativedelta(hours=interval),
    'day': lambda interval: relativedelta(days=interval),
    'month': lambda interval: relativedelta(months=interval),
    False: lambda interval: relativedelta(0),
}


class BaseAutomation(models.Model):
    _name = 'base.automation'
    _description = 'Automated Action'
    _order = 'sequence'

    action_server_id = fields.Many2one(
        'ir.actions.server', 'Server Actions',
        domain="[('model_id', '=', model_id)]",
        delegate=True, required=True, ondelete='restrict')
    active = fields.Boolean(default=True, help="When unchecked, the rule is hidden and will not be executed.")
    trigger = fields.Selection([
        ('on_create', 'On Creation'),
        ('on_write', 'On Update'),
        ('on_create_or_write', 'On Creation & Update'),
        ('on_unlink', 'On Deletion'),
        ('on_change', 'Based on Form Modification'),
        ('on_time', 'Based on Timed Condition')
        ], string='Trigger Condition', required=True)
    trg_date_id = fields.Many2one('ir.model.fields', string='Trigger Date',
                                  help="""When should the condition be triggered.
                                  If present, will be checked by the scheduler. If empty, will be checked at creation and update.""",
                                  domain="[('model_id', '=', model_id), ('ttype', 'in', ('date', 'datetime'))]")
    trg_date_range = fields.Integer(string='Delay after trigger date',
                                    help="""Delay after the trigger date.
                                    You can put a negative number if you need a delay before the
                                    trigger date, like sending a reminder 15 minutes before a meeting.""")
    trg_date_range_type = fields.Selection([('minutes', 'Minutes'), ('hour', 'Hours'), ('day', 'Days'), ('month', 'Months')],
                                           string='Delay type', default='day')
    trg_date_calendar_id = fields.Many2one("resource.calendar", string='Use Calendar',
                                            help="When calculating a day-based timed condition, it is possible to use a calendar to compute the date based on working days.")
    filter_pre_domain = fields.Char(string='Before Update Domain',
                                    help="If present, this condition must be satisfied before the update of the record.")
    filter_domain = fields.Char(string='Apply on', help="If present, this condition must be satisfied before executing the action rule.")
    last_run = fields.Datetime(readonly=True, copy=False)
    on_change_fields = fields.Char(string="On Change Fields Trigger", help="Comma-separated list of field names that triggers the onchange.")
    trigger_field_ids = fields.Many2many('ir.model.fields', string='Watched fields',
                                        help="The action will be triggered if and only if one of these fields is updated."
                                             "If empty, all fields are watched.")

    # which fields have an impact on the registry
    CRITICAL_FIELDS = ['model_id', 'active', 'trigger', 'on_change_fields']

    @api.onchange('model_id')
    def onchange_model_id(self):
        self.model_name = self.model_id.model

    @api.onchange('trigger')
    def onchange_trigger(self):
        if self.trigger in ['on_create', 'on_create_or_write', 'on_unlink']:
            self.filter_pre_domain = self.trg_date_id = self.trg_date_range = self.trg_date_range_type = False
        elif self.trigger in ['on_write', 'on_create_or_write']:
            self.trg_date_id = self.trg_date_range = self.trg_date_range_type = False
        elif self.trigger == 'on_time':
            self.filter_pre_domain = False

    @api.onchange('trigger', 'state')
    def _onchange_state(self):
        if self.trigger == 'on_change' and self.state != 'code':
            ff = self.fields_get(['trigger', 'state'])
            return {'warning': {
                'title': _("Warning"),
                'message': _("The \"%(trigger_value)s\" %(trigger_label)s can only be used with the \"%(state_value)s\" action type") % {
                    'trigger_value': dict(ff['trigger']['selection'])['on_change'],
                    'trigger_label': ff['trigger']['string'],
                    'state_value': dict(ff['state']['selection'])['code'],
                }
            }}

        MAIL_STATES = ('email', 'followers', 'next_activity')
        if self.trigger == 'on_unlink' and self.state in MAIL_STATES:
            return {'warning': {
                'title': _("Warning"),
                'message': _(
                    "You cannot send an email, add followers or create an activity "
                    "for a deleted record.  It simply does not work."
                ),
            }}

    @api.model
    def create(self, vals):
        vals['usage'] = 'base_automation'
        base_automation = super(BaseAutomation, self).create(vals)
        self._update_cron()
        self._update_registry()
        return base_automation

    def write(self, vals):
        res = super(BaseAutomation, self).write(vals)
        if set(vals).intersection(self.CRITICAL_FIELDS):
            self._update_cron()
            self._update_registry()
        return res

    def unlink(self):
        res = super(BaseAutomation, self).unlink()
        self._update_cron()
        self._update_registry()
        return res

    def _update_cron(self):
        """ Activate the cron job depending on whether there exists action rules
            based on time conditions.
        """
        cron = self.env.ref('base_automation.ir_cron_data_base_automation_check', raise_if_not_found=False)
        return cron and cron.toggle(model=self._name, domain=[('trigger', '=', 'on_time')])

    def _update_registry(self):
        """ Update the registry after a modification on action rules. """
        if self.env.registry.ready and not self.env.context.get('import_file'):
            # re-install the model patches, and notify other workers
            self._unregister_hook()
            self._register_hook()
            self.env.registry.registry_invalidated = True

    def _get_actions(self, records, triggers):
        """ Return the actions of the given triggers for records' model. The
            returned actions' context contain an object to manage processing.
        """
        if '__action_done' not in self._context:
            self = self.with_context(__action_done={})
        domain = [('model_name', '=', records._name), ('trigger', 'in', triggers)]
        actions = self.with_context(active_test=True).search(domain)
        return actions.with_env(self.env)

    def _get_eval_context(self):
        """ Prepare the context used when evaluating python code
            :returns: dict -- evaluation context given to safe_eval
        """
        return {
            'datetime': datetime,
            'dateutil': dateutil,
            'time': time,
            'uid': self.env.uid,
            'user': self.env.user,
        }

    def _filter_pre(self, records):
        """ Filter the records that satisfy the precondition of action ``self``. """
        if self.filter_pre_domain and records:
            domain = safe_eval(self.filter_pre_domain, self._get_eval_context())
            return records.sudo().filtered_domain(domain).with_env(records.env)
        else:
            return records

    def _filter_post(self, records):
        return self._filter_post_export_domain(records)[0]

    def _filter_post_export_domain(self, records):
        """ Filter the records that satisfy the postcondition of action ``self``. """
        if self.filter_domain and records:
            domain = safe_eval(self.filter_domain, self._get_eval_context())
            return records.sudo().filtered_domain(domain).with_env(records.env), domain
        else:
            return records, None

    @api.model
    def _add_postmortem_action(self, e):
        if self.user_has_groups('base.group_user'):
            e.context = {}
            e.context['exception_class'] = 'base_automation'
            e.context['base_automation'] = {
                'id': self.id,
                'name': self.name,
            }

    def _process(self, records, domain_post=None):
        """ Process action ``self`` on the ``records`` that have not been done yet. """
        # filter out the records on which self has already been done
        action_done = self._context['__action_done']
        records_done = action_done.get(self, records.browse())
        records -= records_done
        if not records:
            return

        # mark the remaining records as done (to avoid recursive processing)
        action_done = dict(action_done)
        action_done[self] = records_done + records
        self = self.with_context(__action_done=action_done)
        records = records.with_context(__action_done=action_done)

        # modify records
        values = {}
        if 'date_action_last' in records._fields:
            values['date_action_last'] = fields.Datetime.now()
        if values:
            records.write(values)

        # execute server actions
        if self.action_server_id:
            for record in records:
                # we process the action if any watched field has been modified
                if self._check_trigger_fields(record):
                    ctx = {
                        'active_model': record._name,
                        'active_ids': record.ids,
                        'active_id': record.id,
                        'domain_post': domain_post,
                    }
                    try:
                        self.action_server_id.with_context(**ctx).run()
                    except Exception as e:
                        self._add_postmortem_action(e)
                        raise e

    def _check_trigger_fields(self, record):
        """ Return whether any of the trigger fields has been modified on ``record``. """
        if not self.trigger_field_ids:
            # all fields are implicit triggers
            return True

        if not self._context.get('old_values'):
            # this is a create: all fields are considered modified
            return True

        # Note: old_vals are in the format of read()
        old_vals = self._context['old_values'].get(record.id, {})

        def differ(name):
            field = record._fields[name]
            return (
                name in old_vals and
                field.convert_to_cache(record[name], record, validate=False) !=
                field.convert_to_cache(old_vals[name], record, validate=False)
            )
        return any(differ(field.name) for field in self.trigger_field_ids)

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
            """ Instanciate a create method that processes action rules. """
            @api.model_create_multi
            def create(self, vals_list, **kw):
                # retrieve the action rules to possibly execute
                actions = self.env['base.automation']._get_actions(self, ['on_create', 'on_create_or_write'])
                # call original method
                records = create.origin(self.with_env(actions.env), vals_list, **kw)
                # check postconditions, and execute actions on the records that satisfy them
                for action in actions.with_context(old_values=None):
                    action._process(action._filter_post(records))
                return records.with_env(self.env)

            return create

        def make_write():
            """ Instanciate a write method that processes action rules. """
            def write(self, vals, **kw):
                # retrieve the action rules to possibly execute
                actions = self.env['base.automation']._get_actions(self, ['on_write', 'on_create_or_write'])
                records = self.with_env(actions.env)
                # check preconditions on records
                pre = {action: action._filter_pre(records) for action in actions}
                # read old values before the update
                old_values = {
                    old_vals.pop('id'): old_vals
                    for old_vals in (records.read(list(vals)) if vals else [])
                }
                # call original method
                write.origin(records, vals, **kw)
                # check postconditions, and execute actions on the records that satisfy them
                for action in actions.with_context(old_values=old_values):
                    records, domain_post = action._filter_post_export_domain(pre[action])
                    action._process(records, domain_post=domain_post)
                return True

            return write

        def make_compute_field_value():
            """ Instanciate a compute_field_value method that processes action rules. """
            #
            # Note: This is to catch updates made by field recomputations.
            #
            def _compute_field_value(self, field):
                # determine fields that may trigger an action
                stored_fields = [f for f in self._field_computed[field] if f.store]
                if not any(stored_fields):
                    return _compute_field_value.origin(self, field)
                # retrieve the action rules to possibly execute
                actions = self.env['base.automation']._get_actions(self, ['on_write', 'on_create_or_write'])
                records = self.filtered('id').with_env(actions.env)
                # check preconditions on records
                pre = {action: action._filter_pre(records) for action in actions}
                # read old values before the update
                old_values = {
                    old_vals.pop('id'): old_vals
                    for old_vals in (records.read([f.name for f in stored_fields]))
                }
                # call original method
                _compute_field_value.origin(self, field)
                # check postconditions, and execute actions on the records that satisfy them
                for action in actions.with_context(old_values=old_values):
                    records, domain_post = action._filter_post_export_domain(pre[action])
                    action._process(records, domain_post=domain_post)
                return True

            return _compute_field_value

        def make_unlink():
            """ Instanciate an unlink method that processes action rules. """
            def unlink(self, **kwargs):
                # retrieve the action rules to possibly execute
                actions = self.env['base.automation']._get_actions(self, ['on_unlink'])
                records = self.with_env(actions.env)
                # check conditions, and execute actions on the records that satisfy them
                for action in actions:
                    action._process(action._filter_post(records))
                # call original method
                return unlink.origin(self, **kwargs)

            return unlink

        def make_onchange(action_rule_id):
            """ Instanciate an onchange method for the given action rule. """
            def base_automation_onchange(self):
                action_rule = self.env['base.automation'].browse(action_rule_id)
                result = {}
                server_action = action_rule.action_server_id.with_context(
                    active_model=self._name,
                    active_id=self._origin.id,
                    active_ids=self._origin.ids,
                    onchange_self=self,
                )
                try:
                    res = server_action.run()
                except Exception as e:
                    action_rule._add_postmortem_action(e)
                    raise e

                if res:
                    if 'value' in res:
                        res['value'].pop('id', None)
                        self.update({key: val for key, val in res['value'].items() if key in self._fields})
                    if 'domain' in res:
                        result.setdefault('domain', {}).update(res['domain'])
                    if 'warning' in res:
                        result['warning'] = res['warning']
                return result

            return base_automation_onchange

        patched_models = defaultdict(set)
        def patch(model, name, method):
            """ Patch method `name` on `model`, unless it has been patched already. """
            if model not in patched_models[name]:
                patched_models[name].add(model)
                model._patch_method(name, method)

        # retrieve all actions, and patch their corresponding model
        for action_rule in self.with_context({}).search([]):
            Model = self.env.get(action_rule.model_name)

            # Do not crash if the model of the base_action_rule was uninstalled
            if Model is None:
                _logger.warning("Action rule with ID %d depends on model %s" %
                                (action_rule.id,
                                 action_rule.model_name))
                continue

            if action_rule.trigger == 'on_create':
                patch(Model, 'create', make_create())

            elif action_rule.trigger == 'on_create_or_write':
                patch(Model, 'create', make_create())
                patch(Model, 'write', make_write())
                patch(Model, '_compute_field_value', make_compute_field_value())

            elif action_rule.trigger == 'on_write':
                patch(Model, 'write', make_write())
                patch(Model, '_compute_field_value', make_compute_field_value())

            elif action_rule.trigger == 'on_unlink':
                patch(Model, 'unlink', make_unlink())

            elif action_rule.trigger == 'on_change':
                # register an onchange method for the action_rule
                method = make_onchange(action_rule.id)
                for field_name in action_rule.on_change_fields.split(","):
                    Model._onchange_methods[field_name.strip()].append(method)

    def _unregister_hook(self):
        """ Remove the patches installed by _register_hook() """
        NAMES = ['create', 'write', '_compute_field_value', 'unlink', '_onchange_methods']
        for Model in self.env.registry.values():
            for name in NAMES:
                try:
                    delattr(Model, name)
                except AttributeError:
                    pass

    @api.model
    def _check_delay(self, action, record, record_dt):
        if action.trg_date_calendar_id and action.trg_date_range_type == 'day':
            return action.trg_date_calendar_id.plan_days(
                action.trg_date_range,
                fields.Datetime.from_string(record_dt),
                compute_leaves=True,
            )
        else:
            delay = DATE_RANGE_FUNCTION[action.trg_date_range_type](action.trg_date_range)
            return fields.Datetime.from_string(record_dt) + delay

    @api.model
    def _check(self, automatic=False, use_new_cursor=False):
        """ This Function is called by scheduler. """
        if '__action_done' not in self._context:
            self = self.with_context(__action_done={})

        # retrieve all the action rules to run based on a timed condition
        eval_context = self._get_eval_context()
        for action in self.with_context(active_test=True).search([('trigger', '=', 'on_time')]):
            last_run = fields.Datetime.from_string(action.last_run) or datetime.datetime.utcfromtimestamp(0)

            # retrieve all the records that satisfy the action's condition
            domain = []
            context = dict(self._context)
            if action.filter_domain:
                domain = safe_eval(action.filter_domain, eval_context)
            records = self.env[action.model_name].with_context(context).search(domain)

            # determine when action should occur for the records
            if action.trg_date_id.name == 'date_action_last' and 'create_date' in records._fields:
                get_record_dt = lambda record: record[action.trg_date_id.name] or record.create_date
            else:
                get_record_dt = lambda record: record[action.trg_date_id.name]

            # process action on the records that should be executed
            now = datetime.datetime.now()
            for record in records:
                record_dt = get_record_dt(record)
                if not record_dt:
                    continue
                action_dt = self._check_delay(action, record, record_dt)
                if last_run <= action_dt < now:
                    try:
                        action._process(record)
                    except Exception:
                        _logger.error(traceback.format_exc())

            action.write({'last_run': now.strftime(DEFAULT_SERVER_DATETIME_FORMAT)})

            if automatic:
                # auto-commit for batch processing
                self._cr.commit()

```

## File: models\ir_actions.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ServerAction(models.Model):
    _inherit = "ir.actions.server"

    usage = fields.Selection(selection_add=[('base_automation', 'Automated Action')])

```

## File: models\ir_demo.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrDemo(models.TransientModel):
    _inherit = 'ir.demo'

    def install_demo(self):
        # Prevent the registry to reload while loading demo data, since `create` calls
        # `_update_registry`.
        self.pool.ready = False
        try:
            result = super().install_demo()
        finally:
            self.pool.ready = True

        # Reload the registry
        self.env['base.automation']._update_registry()

        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_automation
from . import ir_actions
from . import ir_demo

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_automation,base.automation,model_base_automation,,1,0,0,0
access_base_automation_config,base.automation config,model_base_automation,base.group_system,1,1,1,1
access_base_automation_lead_test,access_base_automation_lead_test,model_base_automation_lead_test,base.group_system,1,1,1,1
access_base_automation_line_test,access_base_automation_line_test,model_base_automation_line_test,base.group_system,1,1,1,1
```

## File: static\src\js\base_automation_error_dialog.js

```javascript
odoo.define('base_automation.BaseAutomatioErrorDialog', function (require) {
    "use strict";

    const CrashManager = require('web.CrashManager');
    const ErrorDialog = CrashManager.ErrorDialog;
    const ErrorDialogRegistry = require('web.ErrorDialogRegistry');
    const session = require('web.session');

    const BaseAutomationErrorDialog = ErrorDialog.extend({
        xmlDependencies: (ErrorDialog.prototype.xmlDependencies || []).concat(
            ['/base_automation/static/src/xml/base_automation_error_dialog.xml']
        ),
        template: 'CrashManager.BaseAutomationError',
        events: {
            'click .o_disable_action_button': '_onDisableAction',
            'click .o_edit_action_button': '_onEditAction',
        },
        /**
        * Assign the `base_automation` object based on the error data,
        * which is then used by the `CrashManager.BaseAutomationError` template
        * and the events defined above.
        * @override
        * @param {Object} error
        * @param {string} error.data.context.base_automation.id  the ID of the failing automated action
        * @param {string} error.data.context.base_automation.name  the name of the failing automated action
        */
        init: function (parent, options, error) {
            this._super.apply(this, arguments);
            this.base_automation = error.data.context.base_automation;
            this.is_admin = session.is_admin;
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
        * This method is called when the user clicks on the 'Disable action' button
        * displayed when a crash occurs in the evaluation of an automated action.
        * Then, we write `active` to `False` on the automated action to disable it.
        *
        * @private
        * @param {MouseEvent} ev
        */
        _onDisableAction: function (ev) {
            ev.preventDefault();
            this._rpc({
                model: 'base.automation',
                method: 'write',
                args: [[this.base_automation.id], {
                    active: false,
                }],
            }).then(this.destroy.bind(this));
        },
        /**
        * This method is called when the user clicks on the 'Edit action' button
        * displayed when a crash occurs in the evaluation of an automated action.
        * Then, we redirect the user to the automated action form.
        *
        * @private
        * @param {MouseEvent} ev
        */
        _onEditAction: function (ev) {
            ev.preventDefault();
            this.do_action({
                name: 'Automated Actions',
                res_model: 'base.automation',
                res_id: this.base_automation.id,
                views: [[false, 'form']],
                type: 'ir.actions.act_window',
                view_mode: 'form',
            });
        },
    });

    ErrorDialogRegistry.add('base_automation', BaseAutomationErrorDialog);

});

```

## File: static\src\xml\base_automation_error_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="CrashManager.BaseAutomationError" t-extend="CrashManager.error">
        <t t-jquery=".alert.alert-warning" t-operation="append">
            <t t-if="widget.base_automation.id">
                <p>
                    The error occurred during the execution of the automated action
                    "<t t-esc="widget.base_automation.name"/>"
                    (ID: <t t-esc="widget.base_automation.id"/>).
                    <br/>
                </p>
                <p t-if="!widget.is_admin">
                    You can ask an administrator to disable or correct this automated action.
                </p>
                <p t-if="widget.is_admin">
                    You can disable this automated action or edit it to solve the issue.<br/>
                    Disabling this automated action will enable you to continue your workflow
                    but any data created after this could potentially be corrupted,
                    as you are effectively disabling a customization that may set
                    important and/or required fields.
                </p>
            </t>
        </t>
        <t t-jquery=".alert.alert-warning button" t-operation="after">
            <t t-if="widget.base_automation.id &amp;&amp; widget.is_admin">
                <button class="btn btn-secondary mt4 o_disable_action_button">
                    <i class="fa fa-ban mr8"/>Disable Action
                </button>
                <button class="btn btn-secondary mt4 o_edit_action_button">
                    <i class="fa fa-edit mr8"/>Edit action
                </button>
            </t>
        </t>
    </t>
</templates>

```

## File: views\base_automation_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Automation Form View -->
        <record id="view_base_automation_form" model="ir.ui.view">
            <field name="name">Automations</field>
            <field name="model">base.automation</field>
            <field name="mode">primary</field>
            <field name="inherit_id" ref="base.view_server_action_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='create_action']" position="replace">
                </xpath>
                <xpath expr="//button[@name='unlink_action']" position="replace">
                </xpath>
                <xpath expr="//group[@name='action_content']" position="inside">
                    <field name="active"/>
                    <field name="trigger"/>
                    <field name="trigger_field_ids" domain="[('model_id', '=', model_id)]"
                                    attrs="{'invisible': [('trigger', 'not in', ['on_write', 'on_create_or_write'])]}"/>

                    <field name="filter_pre_domain" widget="domain" options="{'model': 'model_name', 'in_dialog': True}" attrs="{'invisible': [('trigger', 'not in', ['on_write','on_create_or_write'])]}"/>
                    <field name="filter_domain" widget="domain" options="{'model': 'model_name', 'in_dialog': True}"/>

                    <field name="on_change_fields"
                        attrs="{'invisible': [('trigger', '!=', 'on_change')], 'required': [('trigger', '=', 'on_change')]}"/>
                    <field name="trg_date_id"
                        attrs="{'invisible': [('trigger', '!=', 'on_time')], 'required': [('trigger', '=', 'on_time')]}"/>
                    <label for="trg_date_range"
                        attrs="{'invisible': [('trigger', '!=', 'on_time')]}"/>
                    <div class="o_row" attrs="{'invisible': [('trigger', '!=', 'on_time')]}">
                        <field name="trg_date_range" attrs="{'required': [('trigger','=','on_time')]}"/>
                        <field name="trg_date_range_type" attrs="{'required': [('trigger','=','on_time')]}"/>
                    </div>
                    <field name="trg_date_calendar_id"
                        attrs="{'invisible': ['|', ('trg_date_id','=',False), ('trg_date_range_type', '!=', 'day')]}"/>
                </xpath>
            </field>
        </record>

        <!-- automation Tree View -->
        <record id="view_base_automation_tree" model="ir.ui.view">
            <field name="name">base.automation.tree</field>
            <field name="model">base.automation</field>
            <field name="arch" type="xml">
                <tree string="Automation">
                    <field name="sequence" widget="handle"/>
                    <field name="name"/>
                    <field name="trigger"/>
                    <field name="model_id"/>
                </tree>
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
            <field name="name">Automated Actions</field>
            <field name="res_model">base.automation</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_base_automation_tree"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Setup a new automated automation
              </p><p>
                Use automated actions to automatically trigger actions for
                various screens. Example: a lead created by a specific user may
                be automatically set to a specific Sales Team, or an
                opportunity which still has status pending after 14 days might
                trigger an automatic reminder email.
              </p>
            </field>
        </record>

        <menuitem id="menu_base_automation_form"
            parent="base.menu_automation" action="base_automation_act" sequence="1"/>

        <template id="assets_backend" name="base_automation assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/base_automation/static/src/js/base_automation_error_dialog.js"/>
            </xpath>
        </template>

        <template id="qunit_suite" name="base_automation_tests" inherit_id="web.qunit_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/base_automation/static/tests/base_automation_error_dialog.js"/>
            </xpath>
        </template>

</odoo>

```

