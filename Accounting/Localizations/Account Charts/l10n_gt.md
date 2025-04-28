# Odoo Module: l10n_gt

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Guatemala - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['gt'],
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Guatemala.
=====================================================================

Agrega una nomenclatura contable para Guatemala. También icluye impuestos y
la moneda del Quetzal. -- Adds accounting chart for Guatemala. It also includes
taxes and the Quetzal currency.""",
    'author': 'José Rodrigo Fernández Menegazzo',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'depends': [
        'base',
        'account',
    ],
    'auto_install': ['account'],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\template\account.account-gt.csv

```csv
"id","name","code","account_type","reconcile","name@es"
"cta110201","General Accounts Receivable","1.1.02.01","asset_receivable","True","Cuentas por Cobrar Generales"
"cta110202","Accounts Receivable Affiliated Companies","1.1.02.02","asset_receivable","True","Cuentas por Cobrar Empresas Afilidas"
"cta110203","Loans to Personnel","1.1.02.03","asset_receivable","True","Prestamos al Personal"
"cta110204","Other Accounts Receivable","1.1.02.04","asset_receivable","True","Otras Cuentas por Cobrar"
"cta110205","General Accounts Receivable (PoS)","1.1.02.05","asset_receivable","True","Cuentas por Cobrar Generales (PoS)"
"cta110301","Unpaid VAT","1.1.03.01","asset_current","False","IVA por Cobrar"
"cta110302","VAT withholdings received","1.1.03.02","asset_current","False","Retenciones de IVA recibidas"
"cta120101","Property, Plant and Equipment","1.2.01.01","asset_current","False","Propiedad, Planta y Equipo"
"cta120201","Accumulated Depreciation","1.2.02.01","asset_current","False","Depreciaciones Acumuladas"
"cta130101","Unamortized Expenses","1.3.01.01","asset_current","False","Gastos por Amortizar"
"cta130201","Prepaid Expenses","1.3.02.01","asset_current","False","Gastos Anticipados"
"cta130501","Organizational Expenses","1.3.03.01","asset_current","False","Gastos de Organización"
"cta130601","Other Assets","1.3.04.01","asset_current","False","Otros Activos"
"cta210101","Accounts and notes payable","2.1.01.01","liability_payable","True","Cuentas y Documentos por Pagar"
"cta210201","VAT Payable","2.1.02.01","liability_current","False","IVA por Pagar"
"cta210301","Taxes","2.1.03.01","liability_current","False","Impuestos"
"cta220101","Provision for severance indemnities","2.2.01.01","liability_current","False","Provisión para Indemnizaciones"
"cta230101","Advance payments","2.3.01.01","liability_current","False","Anticipos"
"cta310101","Authorized, Subscribed and Paid-in Capital","3.1.01.01","equity","False","Capital Autorizado, Suscríto y Pagado"
"cta310102","Reservations","3.1.01.02","equity","False","Reservas"
"cta310103","Profit and loss","3.1.01.03","equity","False","Perdidas y Ganancias"
"cta410101","Sales","4.1.01.01","income","False","Ventas"
"cta410102","Sales Discounts","4.1.01.02","income","False","Descuentos Sobre Ventas"
"cta410103","Earn an account","4.1.01.03","income_other","False","Ganar cuenta"
"cta420101","Other Income","4.2.01.01","income","False","Otros Ingresos"
"cta510101","Cost of sales","5.1.01.01","expense","False","Costos de Ventas"
"cta610101","Expense of sales","6.1.01.01","expense","False","Gastos de Ventas"
"cta620101","Administrative Expenses","6.2.01.01","expense","False","Gastos de Administración"
"cta620201","Other Operating Expenses","6.2.02.01","expense","False","Otros Gastos de Operación"
"cta630101","Non-Deductible Expenses","6.3.01.01","expense","False","Gastos no Deducibles"
"cta710101","Other Financial Expenses","7.1.01.01","expense","False","Otros Gastos Financieros"
"cta710102","Interests","7.1.01.02","expense","False","Intereses"

```

## File: data\template\account.tax-gt.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","description@es","invoice_label@es","price_include_override"
"impuestos_plantilla_iva_por_cobrar","12%","Unpaid VAT","Unpaid VAT","12.0","percent","purchase","tax_group_iva_12","base","invoice","","IVA por Cobrar","IVA por Cobrar","tax_included"
"","","","","","","","","tax","invoice","cta110301","","",""
"","","","","","","","","base","refund","","","",""
"","","","","","","","","tax","refund","cta110301","","",""
"impuestos_plantilla_iva_por_pagar","12%","VAT Payable","VAT Payable","12.0","percent","sale","tax_group_iva_12","base","invoice","","IVA por Pagar","IVA por Pagar","tax_included"
"","","","","","","","","tax","invoice","cta210201","","",""
"","","","","","","","","base","refund","","","",""
"","","","","","","","","tax","refund","cta210201","","",""

```

## File: data\template\account.tax.group-gt.csv

```csv
"id","name","country_id","name@es"
"tax_group_iva_12","VAT 12%","base.gt","IVA 12%"

```

## File: models\template_gt.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gt')
    def _get_gt_template_data(self):
        return {
            'code_digits': '9',
            'property_account_receivable_id': 'cta110201',
            'property_account_payable_id': 'cta210101',
            'property_account_income_categ_id': 'cta410101',
            'property_account_expense_categ_id': 'cta510101',
        }

    @template('gt', 'res.company')
    def _get_gt_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.gt',
                'bank_account_code_prefix': '1.0.01.0',
                'cash_account_code_prefix': '1.0.02.0',
                'transfer_account_code_prefix': '1.0.03.01',
                'account_default_pos_receivable_account_id': 'cta110205',
                'income_currency_exchange_account_id': 'cta410103',
                'expense_currency_exchange_account_id': 'cta710101',
                'account_sale_tax_id': 'impuestos_plantilla_iva_por_pagar',
                'account_purchase_tax_id': 'impuestos_plantilla_iva_por_cobrar',

            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_gt

```

