# Odoo Module: mass_mailing_crm

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
    'name': 'Mass mailing on lead / opportunities',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add lead / opportunities UTM info on mass mailing',
    'description': """UTM and mass mailing on lead / opportunities""",
    'depends': ['crm', 'mass_mailing'],
    'data': [
        'views/mailing_mailing_views.xml',
    ],
    'demo': [
        'demo/mailing_mailing.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class CrmLead(models.Model):
    _inherit = 'crm.lead'
    _mailing_enabled = True

```

## File: models\mailing_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup
from odoo import fields, models, _, tools


class MassMailing(models.Model):
    _name = 'mailing.mailing'
    _inherit = 'mailing.mailing'

    use_leads = fields.Boolean('Use Leads', compute='_compute_use_leads')
    crm_lead_count = fields.Integer('Leads/Opportunities Count', compute='_compute_crm_lead_count')

    def _compute_use_leads(self):
        self.use_leads = self.env.user.has_group('crm.group_use_lead')

    def _compute_crm_lead_count(self):
        lead_data = self.env['crm.lead'].with_context(active_test=False).sudo()._read_group(
            [('source_id', 'in', self.source_id.ids)],
            ['source_id'], ['__count'],
        )
        mapped_data = {source.id: count for source, count in lead_data}
        for mass_mailing in self:
            mass_mailing.crm_lead_count = mapped_data.get(mass_mailing.source_id.id, 0)

    def action_redirect_to_leads_and_opportunities(self):
        text = _("Leads") if self.use_leads else _("Opportunities")
        helper_header = _("No %s yet!", text)
        helper_message = _("Note that Odoo cannot track replies if they are sent towards email addresses to this database.")
        return {
            'context': {
                'active_test': False,
                'create': False,
                'search_default_group_by_create_date_day': True,
                'crm_lead_view_hide_month': True,
            },
            'domain': [('source_id', 'in', self.source_id.ids)],
            'help': Markup('<p class="o_view_nocontent_smiling_face">%s</p><p>%s</p>') % (
                helper_header, helper_message,
            ),
            'name': _("Leads Analysis"),
            'res_model': 'crm.lead',
            'type': 'ir.actions.act_window',
            'view_mode': 'tree,pivot,graph,form',
        }

    def _prepare_statistics_email_values(self):
        self.ensure_one()
        values = super(MassMailing, self)._prepare_statistics_email_values()
        if not self.user_id:
            return values
        if not self.env['crm.lead'].check_access_rights('read', raise_exception=False):
            return values
        values['kpi_data'][1]['kpi_col1'] = {
            'value': tools.format_decimalized_number(self.crm_lead_count, decimal=0),
            'col_subtitle': _('LEADS'),
        }
        values['kpi_data'][1]['kpi_name'] = 'lead'
        return values

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    ab_testing_winner_selection = fields.Selection(selection_add=[('crm_lead_count', 'Leads')])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import mailing_mailing
from . import utm

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_mailing_view_form" model="ir.ui.view">
        <field name="name">mailing.mailing.view.form.inherit.crm</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
        <field name="arch" type="xml">
    		<xpath expr="//button[@id='button_view_delivered']" position="before">
                <button name="action_redirect_to_leads_and_opportunities"
                    type="object"
                    icon="fa-star"
                    class="oe_stat_button"
                    invisible="state == 'draft'">
                    <div class="o_field_widget o_stat_info">
                        <field name="use_leads" invisible="1"/>
                        <span class="o_stat_value"><field nolabel="1" name="crm_lead_count"/></span>
                        <span class="o_stat_text" invisible="not use_leads">Leads</span>
                        <span class="o_stat_text" invisible="use_leads">Opportunities</span>
                    </div>
                </button>
    		</xpath>
        </field>
    </record>
</odoo>

```

