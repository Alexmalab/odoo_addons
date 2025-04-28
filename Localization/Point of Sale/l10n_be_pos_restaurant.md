# Odoo Module: l10n_be_pos_restaurant

Category: Localization/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

def post_init_hook(env):
    for company in env['res.company'].search([('chart_template', '=like', 'be%'), ('parent_id', '=', False)]):
        Template = env['account.chart.template'].with_company(company)
        Template._load_data({
            'account.tax': Template._get_be_pos_restaurant_account_tax(),
        })

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Belgian POS Restaurant Localization',
    'version': '1.0',
    'category': 'Localization/Point of Sale',
    'depends': ['pos_restaurant', 'l10n_be'],
    'auto_install': True,
    'installable': True,
    'license': 'LGPL-3',
    'post_init_hook': 'post_init_hook',
}

```

## File: data\template\account.tax-be.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","tax_scope","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@fr","name@nl"
"tax_alcohol_luxury","10","21% VAT (Alcohol, luxury)","21%","21% Alcohol / luxury","21.0","percent","sale","tax_group_tva_21","consu","","base","invoice","+03","","","21% Alcool / luxe","21% Alcohol / luxe"
"","","","","","","","","","","","tax","invoice","+54","a451","","",""
"","","","","","","","","","","","base","refund","+49","","","",""
"","","","","","","","","","","","tax","refund","+64","a451","","",""

```

## File: models\pos_config.py

```python
from odoo import api, models

class PosConfig(models.Model):
    _inherit = 'pos.config'

    def _create_takeaway_fiscal_position(self, config):
        ChartTemplate = self.env['account.chart.template'].with_company(self.env.company)
        tax_21 = ChartTemplate.ref('attn_VAT-OUT-21-L', raise_if_not_found=False)
        tax_12 = ChartTemplate.ref('attn_VAT-OUT-12-L', raise_if_not_found=False)
        tax_6 = ChartTemplate.ref('attn_VAT-OUT-06-L', raise_if_not_found=False)

        if tax_21 and tax_12 and tax_6:
            fp = self.env['account.fiscal.position'].create({
                'name': 'Take out',
            })
            self.env['account.fiscal.position.tax'].create({
                'tax_src_id': tax_21.id,
                'tax_dest_id': tax_6.id,
                'position_id': fp.id
            })
            self.env['account.fiscal.position.tax'].create({
                'tax_src_id': tax_12.id,
                'tax_dest_id': tax_6.id,
                'position_id': fp.id
            })
            config.write({'takeaway': True, 'takeaway_fp_id': fp.id})

    @api.model
    def load_onboarding_bar_scenario(self):
        super().load_onboarding_bar_scenario()
        if (self.env.company.chart_template or '').startswith('be'):
            ChartTemplate = self.env['account.chart.template'].with_company(self.env.company)
            tax_alcohol = ChartTemplate.ref('tax_alcohol_luxury')
            cocktails_category = self.env.ref('pos_restaurant.pos_category_cocktails', raise_if_not_found=False)
            if cocktails_category:
                self.env['product.template'].search([
                    ('pos_categ_ids', 'in', [cocktails_category.id])
                ]).write({'taxes_id': [(6, 0, [tax_alcohol.id])]})

    @api.model
    def load_onboarding_restaurant_scenario(self):
        super().load_onboarding_restaurant_scenario()
        if (self.env.company.chart_template or '').startswith('be'):
            config = self.env.ref(self._get_suffixed_ref_name('pos_restaurant.pos_config_main_restaurant'), raise_if_not_found=False)
            if config:
                self._create_takeaway_fiscal_position(config)

```

## File: models\template_be.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models

from odoo.addons.account.models.chart_template import template

class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('be', 'account.tax')
    def _get_be_pos_restaurant_account_tax(self):
        be_restaurant_tax = self._parse_csv('be', 'account.tax', module='l10n_be_pos_restaurant')
        existing_taxes = self.env['account.tax'].search([('company_id', 'child_of', self.env.company.root_id.id)])
        # Filter out taxes that already exist
        existing_tax_names = set(existing_taxes.mapped('name'))
        taxes_to_create = {name: tax for name, tax in be_restaurant_tax.items() if tax['name'] not in existing_tax_names}
        self._deref_account_tags('be_comp', be_restaurant_tax)
        return taxes_to_create

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import pos_config
from . import template_be

```

