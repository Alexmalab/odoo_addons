# Odoo Module: l10n_es_edi_sii

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from odoo import api, SUPERUSER_ID


def _l10n_es_edi_post_init(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    companies = env['res.company'].search([('partner_id.country_id.code', '=', 'ES')])

    all_chart_templates = companies.chart_template_id
    current_chart_template = all_chart_templates
    while current_chart_template.parent_id:
        all_chart_templates |= current_chart_template.parent_id
        current_chart_template = current_chart_template.parent_id

    if all_chart_templates:
        tax_templates = env['account.tax.template'].search([
            ('chart_template_id', 'in', all_chart_templates.ids),
            '|', '|', '|',
            ('l10n_es_type', '!=', False),
            ('l10n_es_exempt_reason', '!=', False),
            ('tax_scope', '!=', False),
            ('l10n_es_bien_inversion', '!=', False),
        ])
        xml_ids = tax_templates.get_external_id()
        for company in companies:
            for tax_template in tax_templates:
                module, xml_id = xml_ids.get(tax_template.id).split('.')
                tax = env.ref('%s.%s_%s' % (module, company.id, xml_id), raise_if_not_found=False)
                if tax:
                    tax.write({
                        'l10n_es_exempt_reason': tax_template.l10n_es_exempt_reason,
                        'tax_scope': tax_template.tax_scope,
                        'l10n_es_type': tax_template.l10n_es_type,
                        'l10n_es_bien_inversion': tax_template.l10n_es_bien_inversion,
                    })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Thanks to AEOdoo and the Spanish community
# Specially among others Ignacio Ibeas, Pedro Baeza and Landoo

{
    'name': "Spain - SII EDI Suministro de Libros",
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'description': """
        This module sends the taxes information (mostly VAT) of the 
        vendor bills and customer invoices to the SII.  It is called 
        Procedimiento G417 - IVA. Llevanza de libros registro.  It is 
        required for every company with a turnover of +6M€ and others can 
        already make use of it.  The invoices are automatically 
        sent after validation.  
        
        How the information is sent to the SII depends on the 
        configuration that is put in the taxes.  The taxes 
        that were in the chart template (l10n_es) are automatically 
        configured to have the right type.  It is possible however 
        that extra taxes need to be created for certain exempt/no sujeta reasons.   
        
        You need to configure your certificate and the tax agency.  
    """,
    'depends': [
        'l10n_es',
        'account_edi_extended',
    ],
    'data': [
        'data/account_tax_data.xml',
        'data/account_edi_data.xml',
        'data/res_partner_data.xml',

        'security/ir.model.access.csv',

        'views/account_tax_views.xml',
        'views/l10n_es_edi_certificate_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml'
    ],
    'external_dependencies': {
        'python': ['pyOpenSSL'],
    },
    'post_init_hook': '_l10n_es_edi_post_init',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_es_sii" model="account.edi.format">
            <field name="name">SII IVA Llevanza de libros registro (ES)</field>
            <field name="code">es_sii</field>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_es.account_tax_template_s_iva21b" model="account.tax.template">
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva21s" model="account.tax.template">
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva21isp" model="account.tax.template">
        <field name="l10n_es_type">sujeto</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_bc" model="account.tax.template">
        <field name="name">21% IVA soportado (bienes corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_sc" model="account.tax.template">
        <field name="name">21% IVA soportado (servicios corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_sp_in" model="account.tax.template">
        <field name="name">IVA 21% Adquisición de servicios intracomunitarios</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_ic_bc" model="account.tax.template">
        <field name="name">IVA 21% Adquisición Intracomunitaria. Bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_ic_bi" model="account.tax.template">
        <field name="name">IVA 21% Adquisición Intracomunitaria. Bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_ibc" model="account.tax.template">
        <field name="name">IVA 21% Importaciones bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_ibi" model="account.tax.template">
        <field name="name">IVA 21% Importaciones bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf21td" model="account.tax.template">
        <field name="name">Retenciones IRPF (Trabajadores) dinerarios</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_sp_ex" model="account.tax.template">
        <field name="name">IVA 4% Adquisición de servicios extracomunitarios</field>
        <field name="l10n_es_type">sujeto_isp</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_sp_ex" model="account.tax.template">
        <field name="name">IVA 10% Adquisición de servicios extracomunitarios</field>
        <field name="l10n_es_type">sujeto_isp</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_sp_ex" model="account.tax.template">
        <field name="name">IVA 21% Adquisición de servicios extracomunitarios</field>
        <field name="l10n_es_type">sujeto_isp</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_ic_bc" model="account.tax.template">
        <field name="name">IVA 4% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_ic_bi" model="account.tax.template">
        <field name="name">IVA 4% Adquisición Intracomunitario. Bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_ic_bc" model="account.tax.template">
        <field name="name">IVA 10% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_ic_bi" model="account.tax.template">
        <field name="name">IVA 10% Adquisición Intracomunitario. Bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva0_sp_i" model="account.tax.template">
        <field name="name">IVA 0% Prestación de servicios intracomunitario</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva_ns" model="account.tax.template">
        <field name="name">No sujeto Repercutido (Servicios)</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva_ns_b" model="account.tax.template">
        <field name="name">No sujeto Repercutido (Bienes)</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva_e" model="account.tax.template">
        <field name="name">IVA 0% Prestación de servicios extracomunitaria</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_ibc" model="account.tax.template">
        <field name="name">IVA 4% Importaciones bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_ibi" model="account.tax.template">
        <field name="name">IVA 4% Importaciones bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_ibc" model="account.tax.template">
        <field name="name">IVA 10% Importaciones bienes corrientes</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_ibi" model="account.tax.template">
        <field name="name">IVA 10% Importaciones bienes de inversión</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_bi" model="account.tax.template">
        <field name="name">4% IVA Soportado (bienes de inversión)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
        <field name="l10n_es_bien_inversion">True</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_sc" model="account.tax.template">
        <field name="name">4% IVA soportado (servicios corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_bi" model="account.tax.template">
        <field name="name">10% IVA Soportado (bienes de inversión)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
        <field name="l10n_es_bien_inversion">True</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_bi" model="account.tax.template">
        <field name="name">21% IVA Soportado (bienes de inversión)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
        <field name="l10n_es_bien_inversion">True</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_bc" model="account.tax.template">
        <field name="name">10% IVA soportado (bienes corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_bc" model="account.tax.template">
        <field name="name">4% IVA soportado (bienes corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_sc" model="account.tax.template">
        <field name="name">10% IVA soportado (servicios corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva0" model="account.tax.template">
        <field name="name">IVA Exento Repercutido Sujeto</field>
        <field name="l10n_es_type">exento</field>
        <field name="l10n_es_exempt_reason">E1</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva0_ns" model="account.tax.template">
        <field name="name">IVA Exento Repercutido No Sujeto</field>
        <field name="l10n_es_type">ignore</field>
    </record>
    <record id="l10n_es.account_tax_template_s_req05" model="account.tax.template">
        <field name="name">0.50% Recargo Equivalencia Ventas</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva4b" model="account.tax.template">
        <field name="name">IVA 4% (Bienes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva10b" model="account.tax.template">
        <field name="name">IVA 10% (Bienes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva0_nd" model="account.tax.template">
        <field name="name">21% IVA Soportado no deducible</field>
        <field name="l10n_es_type">no_deducible</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_nd" model="account.tax.template">
        <field name="name">10% IVA Soportado no deducible</field>
        <field name="l10n_es_type">no_deducible</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva5_nd" model="account.tax.template">
        <field name="name">5% IVA Soportado no deducible</field>
        <field name="l10n_es_type">no_deducible</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_nd" model="account.tax.template">
        <field name="name">4% IVA Soportado no deducible</field>
        <field name="l10n_es_type">no_deducible</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva4s" model="account.tax.template">
        <field name="name">IVA 4% (Servicios)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva10s" model="account.tax.template">
        <field name="name">IVA 10% (Servicios)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_s_req014" model="account.tax.template">
        <field name="name">1.4% Recargo Equivalencia Ventas</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_s_req52" model="account.tax.template">
        <field name="name">5.2% Recargo Equivalencia Ventas</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva0_bc" model="account.tax.template">
        <field name="name">IVA Soportado exento (operaciones corrientes)</field>
        <field name="l10n_es_type">sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva0_ns" model="account.tax.template">
        <field name="name">IVA Soportado no sujeto (Servicios)</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">service</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva0_ns_b" model="account.tax.template">
        <field name="name">IVA Soportado no sujeto (Bienes)</field>
        <field name="l10n_es_type">no_sujeto</field>
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf9" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 9%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf18" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 18%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf19" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 19%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf19a" model="account.tax.template">
        <field name="name">Retenciones a cuenta 19% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf195a" model="account.tax.template">
        <field name="name">Retenciones a cuenta 19,5% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf19" model="account.tax.template">
        <field name="name">Retenciones IRPF 19%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf20a" model="account.tax.template">
        <field name="name">Retenciones 20% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf18" model="account.tax.template">
        <field name="name">Retenciones IRPF 18%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf19a" model="account.tax.template">
        <field name="name">Retenciones 19% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf195a" model="account.tax.template">
        <field name="name">Retenciones 19,5% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf7" model="account.tax.template">
        <field name="name">Retenciones IRPF 7%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf9" model="account.tax.template">
        <field name="name">Retenciones IRPF 9%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf24" model="account.tax.template">
        <field name="name">Retenciones IRPF 14%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf20" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 20%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf20a" model="account.tax.template">
        <field name="name">Retenciones a cuenta 20% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf24" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 24%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva12_agr" model="account.tax.template">
        <field name="name">12% IVA Soportado régimen agricultura</field>
        <field name="l10n_es_type">sujeto_agricultura</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva105_gan" model="account.tax.template">
        <field name="name">10,5% IVA Soportado régimen ganadero o pesca</field>
        <!-- TODO: could not find anything back -->
    </record>
    <record id="l10n_es.account_tax_template_s_iva0_e" model="account.tax.template">
        <field name="name">IVA 0% Exportaciones</field>
        <field name="l10n_es_type">exento</field>
        <field name="l10n_es_exempt_reason">E2</field> <!--E2 for exportation-->
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva0_ic" model="account.tax.template">
        <field name="name">IVA 0% Entregas Intracomunitarias exentas</field>
        <field name="l10n_es_type">exento</field>
        <field name="l10n_es_exempt_reason">E5</field> <!--E5  for intra-community-->
        <field name="tax_scope">consu</field>
    </record>
    <record id="l10n_es.account_tax_template_p_req014" model="account.tax.template">
        <field name="name">1.4% Recargo Equivalencia Compras</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_p_req05" model="account.tax.template">
        <field name="name">0.50% Recargo Equivalencia Compras</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_p_req52" model="account.tax.template">
        <field name="name">5.2% Recargo Equivalencia Compras</field>
        <field name="l10n_es_type">recargo</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf1" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 1%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf2" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 2%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf21" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 21%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf21a" model="account.tax.template">
        <field name="name">Retenciones a cuenta 21% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf7" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 7%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_irpf15" model="account.tax.template">
        <field name="name">Retenciones a cuenta IRPF 15%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf1" model="account.tax.template">
        <field name="name">Retenciones IRPF 1%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf15" model="account.tax.template">
        <field name="name">Retenciones IRPF 15%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf21t" model="account.tax.template">
        <field name="name">Retenciones IRPF (Trabajadores)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_sp_in" model="account.tax.template">
        <field name="name">IVA 10% Adquisición de servicios intracomunitarios</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_sp_in" model="account.tax.template">
        <field name="name">IVA 4% Adquisición de servicios intracomunitarios</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf21te" model="account.tax.template">
        <field name="name">Retenciones IRPF (Trabajadores) en especie</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf20" model="account.tax.template">
        <field name="name">Retenciones IRPF 20%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf21a" model="account.tax.template">
        <field name="name">Retenciones 21% (Arrendamientos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf21p" model="account.tax.template">
        <field name="name">Retenciones IRPF 21%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_irpf2" model="account.tax.template">
        <field name="name">Retenciones IRPF 2%</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_s_iva0_isp" model="account.tax.template">
        <field name="name">IVA 0% Venta con Inversión del Sujeto Pasivo</field>
        <field name="l10n_es_type">sujeto_isp</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva4_isp" model="account.tax.template">
        <field name="name">IVA 4% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="l10n_es_type">sujeto_isp</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva10_isp" model="account.tax.template">
        <field name="name">IVA 10% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="l10n_es_type">sujeto_isp</field>
    </record>
    <record id="l10n_es.account_tax_template_p_iva21_isp" model="account.tax.template">
        <field name="name">IVA 21% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="l10n_es_type">sujeto_isp</field>
    </record>
    <record id="l10n_es.account_tax_template_p_rp19" model="account.tax.template">
        <field name="name">Retenciones 19% (préstamos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
    <record id="l10n_es.account_tax_template_p_rrD19" model="account.tax.template">
        <field name="name">Retenciones 19% (reparto de dividendos)</field>
        <field name="l10n_es_type">retencion</field>
    </record>
</odoo>

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="partner_simplified" model="res.partner">
        <field name="name">Simplified Invoice Partner (ES)</field>
    </record>
</odoo>

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict
from urllib3.util.ssl_ import create_urllib3_context, DEFAULT_CIPHERS
from OpenSSL.crypto import load_certificate, load_privatekey, FILETYPE_PEM
from zeep.transports import Transport

from odoo import fields
from odoo.exceptions import UserError
from odoo.tools import html_escape

import math
import json
import requests
import zeep

from odoo import models, _


# Custom patches to perform the WSDL requests.

EUSKADI_CIPHERS = f"{DEFAULT_CIPHERS}:!DH"


class PatchedHTTPAdapter(requests.adapters.HTTPAdapter):
    """ An adapter to block DH ciphers which may not work for the tax agencies called"""

    def init_poolmanager(self, *args, **kwargs):
        # OVERRIDE
        kwargs['ssl_context'] = create_urllib3_context(ciphers=EUSKADI_CIPHERS)
        return super().init_poolmanager(*args, **kwargs)

    def cert_verify(self, conn, url, verify, cert):
        # OVERRIDE
        # The last parameter is only used by the super method to check if the file exists.
        # In our case, cert is an odoo record 'l10n_es_edi.certificate' so not a path to a file.
        # By putting 'None' as last parameter, we ensure the check about TLS configuration is
        # still made without checking temporary files exist.
        super().cert_verify(conn, url, verify, None)
        conn.cert_file = cert
        conn.key_file = None

    def get_connection(self, url, proxies=None):
        # OVERRIDE
        # Patch the OpenSSLContext to decode the certificate in-memory.
        conn = super().get_connection(url, proxies=proxies)
        context = conn.conn_kw['ssl_context']

        def patched_load_cert_chain(l10n_es_odoo_certificate, keyfile=None, password=None):
            cert_file, key_file, dummy = l10n_es_odoo_certificate.sudo()._decode_certificate()
            cert_obj = load_certificate(FILETYPE_PEM, cert_file)
            pkey_obj = load_privatekey(FILETYPE_PEM, key_file)

            context._ctx.use_certificate(cert_obj)
            context._ctx.use_privatekey(pkey_obj)

        context.load_cert_chain = patched_load_cert_chain

        return conn


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # ES EDI
    # -------------------------------------------------------------------------

    def _l10n_es_edi_get_invoices_tax_details_info(self, invoice, filter_invl_to_apply=None):

        def grouping_key_generator(tax_values):
            tax = tax_values['tax_id']
            return {
                'applied_tax_amount': tax.amount,
                'l10n_es_type': tax.l10n_es_type,
                'l10n_es_exempt_reason': tax.l10n_es_exempt_reason if tax.l10n_es_type == 'exento' else False,
                'l10n_es_bien_inversion': tax.l10n_es_bien_inversion,
            }

        def filter_to_apply(tax_values):
            # For intra-community, we do not take into account the negative repartition line
            return tax_values['tax_repartition_line_id'].factor_percent > 0.0

        def full_filter_invl_to_apply(invoice_line):
            if 'ignore' in invoice_line.tax_ids.flatten_taxes_hierarchy().mapped('l10n_es_type'):
                return False
            return filter_invl_to_apply(invoice_line) if filter_invl_to_apply else True

        tax_details = invoice._prepare_edi_tax_details(
            grouping_key_generator=grouping_key_generator,
            filter_invl_to_apply=full_filter_invl_to_apply,
            filter_to_apply=filter_to_apply,
        )
        sign = -1 if invoice.is_sale_document() else 1

        tax_details_info = defaultdict(dict)

        # Detect for which is the main tax for 'recargo'. Since only a single combination tax + recargo is allowed
        # on the same invoice, this can be deduced globally.

        recargo_tax_details = {} # Mapping between main tax and recargo tax details
        invoice_lines = invoice.invoice_line_ids.filtered(lambda x: not x.display_type)
        if filter_invl_to_apply:
            invoice_lines = invoice_lines.filtered(filter_invl_to_apply)
        for line in invoice_lines:
            taxes = line.tax_ids.flatten_taxes_hierarchy()
            recargo_tax = [t for t in taxes if t.l10n_es_type == 'recargo']
            if recargo_tax and taxes:
                recargo_main_tax = taxes.filtered(lambda x: x.l10n_es_type in ('sujeto', 'sujeto_isp'))[:1]
                if not recargo_tax_details.get(recargo_main_tax):
                    recargo_tax_details[recargo_main_tax] = [
                        x for x in tax_details['tax_details'].values()
                        if x['group_tax_details'][0]['tax_id'] == recargo_tax[0]
                    ][0]

        tax_amount_deductible = 0.0
        tax_amount_retention = 0.0
        base_amount_not_subject = 0.0
        base_amount_not_subject_loc = 0.0
        tax_subject_info_list = []
        tax_subject_isp_info_list = []
        for tax_values in tax_details['tax_details'].values():

            if invoice.is_sale_document():
                # Customer invoices

                if tax_values['l10n_es_type'] in ('sujeto', 'sujeto_isp'):
                    tax_amount_deductible += tax_values['tax_amount']

                    base_amount = sign * tax_values['base_amount']
                    tax_info = {
                        'TipoImpositivo': tax_values['applied_tax_amount'],
                        'BaseImponible': round(base_amount, 2),
                        'CuotaRepercutida': round(math.copysign(tax_values['tax_amount'], base_amount), 2),
                    }

                    recargo = recargo_tax_details.get(tax_values['group_tax_details'][0]['tax_id'])
                    if recargo:
                        tax_info['CuotaRecargoEquivalencia'] = round(sign * recargo['tax_amount'], 2)
                        tax_info['TipoRecargoEquivalencia'] = recargo['applied_tax_amount']

                    if tax_values['l10n_es_type'] == 'sujeto':
                        tax_subject_info_list.append(tax_info)
                    else:
                        tax_subject_isp_info_list.append(tax_info)

                elif tax_values['l10n_es_type'] == 'exento':
                    tax_details_info['Sujeta'].setdefault('Exenta', {'DetalleExenta': []})
                    tax_details_info['Sujeta']['Exenta']['DetalleExenta'].append({
                        'BaseImponible': round(sign * tax_values['base_amount'], 2),
                        'CausaExencion': tax_values['l10n_es_exempt_reason'],
                    })
                elif tax_values['l10n_es_type'] == 'retencion':
                    tax_amount_retention += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto':
                    base_amount_not_subject += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto_loc':
                    base_amount_not_subject_loc += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'ignore':
                    continue

                if tax_subject_isp_info_list and not tax_subject_info_list:
                    tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S2'}
                elif not tax_subject_isp_info_list and tax_subject_info_list:
                    tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S1'}
                elif tax_subject_isp_info_list and tax_subject_info_list:
                    tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S3'}

                if tax_subject_info_list:
                    tax_details_info['Sujeta']['NoExenta'].setdefault('DesgloseIVA', {})
                    tax_details_info['Sujeta']['NoExenta']['DesgloseIVA'].setdefault('DetalleIVA', [])
                    tax_details_info['Sujeta']['NoExenta']['DesgloseIVA']['DetalleIVA'] += tax_subject_info_list
                if tax_subject_isp_info_list:
                    tax_details_info['Sujeta']['NoExenta'].setdefault('DesgloseIVA', {})
                    tax_details_info['Sujeta']['NoExenta']['DesgloseIVA'].setdefault('DetalleIVA', [])
                    tax_details_info['Sujeta']['NoExenta']['DesgloseIVA']['DetalleIVA'] += tax_subject_isp_info_list

            else:
                # Vendor bills
                if tax_values['l10n_es_type'] in ('sujeto', 'sujeto_isp', 'no_sujeto', 'no_sujeto_loc'):
                    tax_amount_deductible += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'retencion':
                    tax_amount_retention += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto':
                    base_amount_not_subject += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto_loc':
                    base_amount_not_subject_loc += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'ignore':
                    continue

                if tax_values['l10n_es_type'] not in ['retencion', 'recargo']: # = in sujeto/sujeto_isp/no_deducible
                    base_amount = sign * tax_values['base_amount']
                    tax_details_info.setdefault('DetalleIVA', [])
                    tax_info = {
                        'BaseImponible': round(base_amount, 2),
                    }
                    if tax_values['applied_tax_amount'] > 0.0:
                        tax_info.update({
                            'TipoImpositivo': tax_values['applied_tax_amount'],
                            'CuotaSoportada': round(math.copysign(tax_values['tax_amount'], base_amount), 2),
                        })
                    if tax_values['l10n_es_bien_inversion']:
                        tax_info['BienInversion'] = 'S'
                    recargo = recargo_tax_details.get(tax_values['group_tax_details'][0]['tax_id'])
                    if recargo:
                        tax_info['CuotaRecargoEquivalencia'] = round(sign * recargo['tax_amount'], 2)
                        tax_info['TipoRecargoEquivalencia'] = recargo['applied_tax_amount']
                    tax_details_info['DetalleIVA'].append(tax_info)

        if not invoice.company_id.currency_id.is_zero(base_amount_not_subject) and invoice.is_sale_document():
            tax_details_info['NoSujeta']['ImportePorArticulos7_14_Otros'] = round(sign * base_amount_not_subject, 2)
        if not invoice.company_id.currency_id.is_zero(base_amount_not_subject_loc) and invoice.is_sale_document():
            tax_details_info['NoSujeta']['ImporteTAIReglasLocalizacion'] = round(sign * base_amount_not_subject_loc, 2)

        return {
            'tax_details_info': tax_details_info,
            'tax_details': tax_details,
            'tax_amount_deductible': tax_amount_deductible,
            'tax_amount_retention': tax_amount_retention,
            'base_amount_not_subject': base_amount_not_subject,
        }

    def _l10n_es_edi_get_partner_info(self, partner):
        eu_country_codes = set(self.env.ref('base.europe').country_ids.mapped('code'))

        partner_info = {}
        IDOtro_ID = partner.vat or 'NO_DISPONIBLE'

        if (not partner.country_id or partner.country_id.code == 'ES') and partner.vat:
            # ES partner with VAT.
            partner_info['NIF'] = partner.vat[2:] if partner.vat.startswith('ES') else partner.vat
            if self.env.context.get('error_1117'):
                partner_info['IDOtro'] = {'IDType': '07', 'ID': IDOtro_ID}

        elif partner.country_id.code in eu_country_codes and partner.vat:
            # European partner.
            partner_info['IDOtro'] = {'IDType': '02', 'ID': IDOtro_ID}
        else:
            partner_info['IDOtro'] = {'ID': IDOtro_ID}
            if partner.vat:
                partner_info['IDOtro']['IDType'] = '04'
            else:
                partner_info['IDOtro']['IDType'] = '06'
            if partner.country_id:
                partner_info['IDOtro']['CodigoPais'] = partner.country_id.code
        return partner_info

    def _l10n_es_edi_get_invoices_info(self, invoices):
        eu_country_codes = set(self.env.ref('base.europe').country_ids.mapped('code'))

        simplified_partner = self.env.ref("l10n_es_edi_sii.partner_simplified")

        info_list = []
        for invoice in invoices:
            com_partner = invoice.commercial_partner_id
            is_simplified = invoice.partner_id == simplified_partner

            info = {
                'PeriodoLiquidacion': {
                    'Ejercicio': str(invoice.date.year),
                    'Periodo': str(invoice.date.month).zfill(2),
                },
                'IDFactura': {
                    'FechaExpedicionFacturaEmisor': invoice.invoice_date.strftime('%d-%m-%Y'),
                },
            }

            if invoice.is_sale_document():
                invoice_node = info['FacturaExpedida'] = {}
            else:
                invoice_node = info['FacturaRecibida'] = {}

            # === Partner ===

            partner_info = self._l10n_es_edi_get_partner_info(com_partner)

            # === Invoice ===

            invoice_node['DescripcionOperacion'] = invoice.invoice_origin[:500] if invoice.invoice_origin else 'manual'
            if invoice.is_sale_document():
                info['IDFactura']['IDEmisorFactura'] = {'NIF': invoice.company_id.vat[2:]}
                info['IDFactura']['NumSerieFacturaEmisor'] = invoice.name[:60]
                if not is_simplified:
                    invoice_node['Contraparte'] = {
                        **partner_info,
                        'NombreRazon': com_partner.name[:120],
                    }

                if not com_partner.country_id or com_partner.country_id.code in eu_country_codes:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '01'
                else:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '02'
            else:
                info['IDFactura']['IDEmisorFactura'] = partner_info
                info['IDFactura']['NumSerieFacturaEmisor'] = invoice.ref[:60]
                if not is_simplified:
                    invoice_node['Contraparte'] = {
                        **partner_info,
                        'NombreRazon': com_partner.name[:120],
                    }

                if invoice.l10n_es_registration_date:
                    invoice_node['FechaRegContable'] = invoice.l10n_es_registration_date.strftime('%d-%m-%Y')
                else:
                    invoice_node['FechaRegContable'] = fields.Date.context_today(self).strftime('%d-%m-%Y')

                country_code = com_partner.country_id.code
                if not country_code or country_code == 'ES' or country_code not in eu_country_codes:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '01'
                else:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '09' # For Intra-Com

            if invoice.move_type == 'out_invoice':
                invoice_node['TipoFactura'] = 'F2' if is_simplified else 'F1'
            elif invoice.move_type == 'out_refund':
                invoice_node['TipoFactura'] = 'R5' if is_simplified else 'R1'
                invoice_node['TipoRectificativa'] = 'I'
            elif invoice.move_type == 'in_invoice':
                invoice_node['TipoFactura'] = 'F1'
            elif invoice.move_type == 'in_refund':
                invoice_node['TipoFactura'] = 'R4'
                invoice_node['TipoRectificativa'] = 'I'

            # === Taxes ===

            sign = -1 if invoice.is_sale_document() else 1

            if invoice.is_sale_document():
                # Customer invoices

                if com_partner.country_id.code in ('ES', False) and not (com_partner.vat or '').startswith("ESN"):
                    tax_details_info_vals = self._l10n_es_edi_get_invoices_tax_details_info(invoice)
                    invoice_node['TipoDesglose'] = {'DesgloseFactura': tax_details_info_vals['tax_details_info']}

                    invoice_node['ImporteTotal'] = round(sign * (
                        tax_details_info_vals['tax_details']['base_amount']
                        + tax_details_info_vals['tax_details']['tax_amount']
                        - tax_details_info_vals['tax_amount_retention']
                    ), 2)
                else:
                    tax_details_info_service_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                        invoice,
                        filter_invl_to_apply=lambda x: any(t.tax_scope == 'service' for t in x.tax_ids)
                    )
                    tax_details_info_consu_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                        invoice,
                        filter_invl_to_apply=lambda x: any(t.tax_scope == 'consu' for t in x.tax_ids)
                    )

                    if tax_details_info_service_vals['tax_details_info']:
                        invoice_node.setdefault('TipoDesglose', {})
                        invoice_node['TipoDesglose'].setdefault('DesgloseTipoOperacion', {})
                        invoice_node['TipoDesglose']['DesgloseTipoOperacion']['PrestacionServicios'] = tax_details_info_service_vals['tax_details_info']
                    if tax_details_info_consu_vals['tax_details_info']:
                        invoice_node.setdefault('TipoDesglose', {})
                        invoice_node['TipoDesglose'].setdefault('DesgloseTipoOperacion', {})
                        invoice_node['TipoDesglose']['DesgloseTipoOperacion']['Entrega'] = tax_details_info_consu_vals['tax_details_info']
                    if not invoice_node.get('TipoDesglose'):
                        raise UserError(_(
                            "In case of a foreign customer, you need to configure the tax scope on taxes:\n%s",
                            "\n".join(invoice.line_ids.tax_ids.mapped('name'))
                        ))

                    invoice_node['ImporteTotal'] = round(sign * (
                        tax_details_info_service_vals['tax_details']['base_amount']
                        + tax_details_info_service_vals['tax_details']['tax_amount']
                        - tax_details_info_service_vals['tax_amount_retention']
                        + tax_details_info_consu_vals['tax_details']['base_amount']
                        + tax_details_info_consu_vals['tax_details']['tax_amount']
                        - tax_details_info_consu_vals['tax_amount_retention']
                    ), 2)

            else:
                # Vendor bills

                tax_details_info_isp_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                    invoice,
                    filter_invl_to_apply=lambda x: any(t for t in x.tax_ids if t.l10n_es_type == 'sujeto_isp'),
                )
                tax_details_info_other_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                    invoice,
                    filter_invl_to_apply=lambda x: not any(t for t in x.tax_ids if t.l10n_es_type == 'sujeto_isp'),
                )

                invoice_node['DesgloseFactura'] = {}
                if tax_details_info_isp_vals['tax_details_info']:
                    invoice_node['DesgloseFactura']['InversionSujetoPasivo'] = tax_details_info_isp_vals['tax_details_info']
                if tax_details_info_other_vals['tax_details_info']:
                    invoice_node['DesgloseFactura']['DesgloseIVA'] = tax_details_info_other_vals['tax_details_info']

                invoice_node['ImporteTotal'] = round(sign * (
                    tax_details_info_isp_vals['tax_details']['base_amount']
                    + tax_details_info_isp_vals['tax_details']['tax_amount']
                    - tax_details_info_isp_vals['tax_amount_retention']
                    + tax_details_info_other_vals['tax_details']['base_amount']
                    + tax_details_info_other_vals['tax_details']['tax_amount']
                    - tax_details_info_other_vals['tax_amount_retention']
                ), 2)

                invoice_node['CuotaDeducible'] = round(sign * (
                    tax_details_info_isp_vals['tax_amount_deductible']
                    + tax_details_info_other_vals['tax_amount_deductible']
                ), 2)

            info_list.append(info)
        return info_list

    def _l10n_es_edi_web_service_aeat_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://www2.agenciatributaria.gob.es/static_files/common/internet/dep/aplicaciones/es/aeat/ssii_1_1/fact/ws/SuministroFactEmitidas.wsdl',
                'test_url': 'https://prewww1.aeat.es/wlpl/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://www2.agenciatributaria.gob.es/static_files/common/internet/dep/aplicaciones/es/aeat/ssii_1_1/fact/ws/SuministroFactRecibidas.wsdl',
                'test_url': 'https://prewww1.aeat.es/wlpl/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_web_service_bizkaia_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://www.bizkaia.eus/ogasuna/sii/documentos/SuministroFactEmitidas.wsdl',
                'test_url': 'https://pruapps.bizkaia.eus/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://www.bizkaia.eus/ogasuna/sii/documentos/SuministroFactRecibidas.wsdl',
                'test_url': 'https://pruapps.bizkaia.eus/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_web_service_gipuzkoa_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://egoitza.gipuzkoa.eus/ogasuna/sii/ficheros/v1.1/SuministroFactEmitidas.wsdl',
                'test_url': 'https://sii-prep.egoitza.gipuzkoa.eus/JBS/HACI/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://egoitza.gipuzkoa.eus/ogasuna/sii/ficheros/v1.1/SuministroFactRecibidas.wsdl',
                'test_url': 'https://sii-prep.egoitza.gipuzkoa.eus/JBS/HACI/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_call_web_service_sign(self, invoices, info_list):
        company = invoices.company_id

        # All are sharing the same value, see '_get_batch_key'.
        csv_number = invoices.mapped('l10n_es_edi_csv')[0]

        # Set registration date
        invoices.filtered(lambda inv: not inv.l10n_es_registration_date).write({
            'l10n_es_registration_date': fields.Date.context_today(self),
        })

        # === Call the web service ===

        # Get connection data.
        l10n_es_edi_tax_agency = company.mapped('l10n_es_edi_tax_agency')[0]
        connection_vals = getattr(self, f'_l10n_es_edi_web_service_{l10n_es_edi_tax_agency}_vals')(invoices)

        header = {
            'IDVersionSii': '1.1',
            'Titular': {
                'NombreRazon': company.name[:120],
                'NIF': company.vat[2:],
            },
            'TipoComunicacion': 'A1' if csv_number else 'A0',
        }

        session = requests.Session()
        session.cert = company.l10n_es_edi_certificate_id
        session.mount('https://', PatchedHTTPAdapter())

        transport = Transport(operation_timeout=60, timeout=60, session=session)
        client = zeep.Client(connection_vals['url'], transport=transport)

        if invoices[0].is_sale_document():
            service_name = 'SuministroFactEmitidas'
        else:
            service_name = 'SuministroFactRecibidas'
        if company.l10n_es_edi_test_env and not connection_vals.get('test_url'):
            service_name += 'Pruebas'

        # Establish the connection.
        serv = client.bind('siiService', service_name)
        if company.l10n_es_edi_test_env and connection_vals.get('test_url'):
            serv._binding_options['address'] = connection_vals['test_url']

        msg = ''
        try:
            if invoices[0].is_sale_document():
                res = serv.SuministroLRFacturasEmitidas(header, info_list)
            else:
                res = serv.SuministroLRFacturasRecibidas(header, info_list)
        except requests.exceptions.SSLError as error:
            msg = _("The SSL certificate could not be validated.")
        except zeep.exceptions.Error as error:
            msg = _("Networking error:\n%s") % error
        except Exception as error:
            msg = str(error)
        finally:
            if msg:
                return {inv: {
                    'error': msg,
                    'blocking_level': 'warning',
                } for inv in invoices}

        # Process response.

        if not res or not res.RespuestaLinea:
            return {inv: {
                'error': _("The web service is not responding"),
                'blocking_level': 'warning',
            } for inv in invoices}

        resp_state = res["EstadoEnvio"]
        l10n_es_edi_csv = res['CSV']

        if resp_state == 'Correcto':
            invoices.write({'l10n_es_edi_csv': l10n_es_edi_csv})
            return {inv: {'success': True} for inv in invoices}

        results = {}
        for respl in res.RespuestaLinea:
            invoice_number = respl.IDFactura.NumSerieFacturaEmisor

            # Retrieve the corresponding invoice.
            # Note: ref can be the same for different partners but there is no enough information on the response
            # to match the partner.

            # Note: Invoices are batched per move_type.
            if invoices[0].is_sale_document():
                inv = invoices.filtered(lambda x: x.name[:60] == invoice_number)
            else:
                # 'ref' can be the same for different partners.
                candidates = invoices.filtered(lambda x: x.ref[:60] == invoice_number)
                if len(candidates) >= 1:
                    respl_partner_info = respl.IDFactura.IDEmisorFactura
                    inv = None
                    for candidate in candidates:
                        partner_info = self._l10n_es_edi_get_partner_info(candidate.commercial_partner_id)
                        if partner_info.get('NIF') and partner_info['NIF'] == respl_partner_info.NIF:
                            inv = candidate
                            break
                        if partner_info.get('IDOtro') and all(getattr(respl_partner_info.IDOtro, k) == v
                                                           for k, v in partner_info['IDOtro'].items()):
                            inv = candidate
                            break

                    if not inv:
                        # This case shouldn't happen and means there is something wrong in this code. However, we can't
                        # raise anything since the document has already been approved by the government. The result
                        # will only be a badly logged message into the chatter so, not a big deal.
                        inv = candidates[0]
                else:
                    inv = candidates

            resp_line_state = respl.EstadoRegistro
            if resp_line_state in ('Correcto', 'AceptadoConErrores'):
                inv.l10n_es_edi_csv = l10n_es_edi_csv
                results[inv] = {'success': True}
                if resp_line_state == 'AceptadoConErrores':
                    inv.message_post(body=_("This was accepted with errors: ") + html_escape(respl.DescripcionErrorRegistro))
            elif respl.RegistroDuplicado:
                results[inv] = {'success': True}
                inv.message_post(body=_("We saw that this invoice was sent correctly before, but we did not treat "
                                        "the response.  Make sure it is not because of a wrong configuration."))
            elif respl.CodigoErrorRegistro == 1117 and not self.env.context.get('error_1117'):
                return self.with_context(error_1117=True)._post_invoice_edi(invoices)
            else:
                results[inv] = {
                    'error': _("[%s] %s", respl.CodigoErrorRegistro, respl.DescripcionErrorRegistro),
                    'blocking_level': 'error',
                }

        return results

    # -------------------------------------------------------------------------
    # EDI OVERRIDDEN METHODS
    # -------------------------------------------------------------------------

    def _get_invoice_edi_content(self, move):
        if self.code != 'es_sii':
            return super()._get_invoice_edi_content(move)
        return json.dumps(self._l10n_es_edi_get_invoices_info(move)).encode()

    def _is_required_for_invoice(self, invoice):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._is_required_for_invoice(invoice)

        return invoice.l10n_es_edi_is_required

    def _needs_web_services(self):
        # OVERRIDE
        return self.code == 'es_sii' or super()._needs_web_services()

    def _support_batching(self, move=None, state=None, company=None):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._support_batching(move=move, state=state, company=company)

        return state == 'to_send' and move.is_invoice()

    def _get_batch_key(self, move, state):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._get_batch_key(move, state)

        return move.move_type, move.l10n_es_edi_csv

    def _check_move_configuration(self, move):
        # OVERRIDE
        res = super()._check_move_configuration(move)
        if self.code != 'es_sii':
            return res

        if not move.company_id.vat:
            res.append(_("VAT number is missing on company %s", move.company_id.display_name))
        for line in move.invoice_line_ids.filtered(lambda line: not line.display_type):
            taxes = line.tax_ids.flatten_taxes_hierarchy()
            recargo_count = taxes.mapped('l10n_es_type').count('recargo')
            retention_count = taxes.mapped('l10n_es_type').count('retencion')
            sujeto_count = taxes.mapped('l10n_es_type').count('sujeto')
            no_sujeto_count = taxes.mapped('l10n_es_type').count('no_sujeto')
            no_sujeto_loc_count = taxes.mapped('l10n_es_type').count('no_sujeto_loc')
            if retention_count > 1:
                res.append(_("Line %s should only have one retention tax.", line.display_name))
            if recargo_count > 1:
                res.append(_("Line %s should only have one recargo tax.", line.display_name))
            if sujeto_count > 1:
                res.append(_("Line %s should only have one sujeto tax.", line.display_name))
            if no_sujeto_count > 1:
                res.append(_("Line %s should only have one no sujeto tax.", line.display_name))
            if no_sujeto_loc_count > 1:
                res.append(_("Line %s should only have one no sujeto (localizations) tax.", line.display_name))
            if sujeto_count + no_sujeto_loc_count + no_sujeto_count > 1:
                res.append(_("Line %s should only have one main tax.", line.display_name))
        if move.move_type in ('in_invoice', 'in_refund'):
            if not move.ref:
                res.append(_("You should put a vendor reference on this vendor bill. "))
        return res

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._is_compatible_with_journal(journal)

        return journal.country_code == 'ES'

    def _post_invoice_edi(self, invoices, test_mode=False):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._post_invoice_edi(invoices, test_mode=test_mode)

        # Ensure a certificate is available.
        certificate = invoices.company_id.l10n_es_edi_certificate_id
        if not certificate:
            return {inv: {
                'error': _("Please configure the certificate for SII."),
                'blocking_level': 'error',
            } for inv in invoices}

        # Ensure a tax agency is available.
        l10n_es_edi_tax_agency = invoices.company_id.mapped('l10n_es_edi_tax_agency')[0]
        if not l10n_es_edi_tax_agency:
            return {inv: {
                'error': _("Please specify a tax agency on your company for SII."),
                'blocking_level': 'error',
            } for inv in invoices}

        # Generate the JSON.
        info_list = self._l10n_es_edi_get_invoices_info(invoices)

        # Call the web service.
        if test_mode:
            res = {inv: {'success': True} for inv in invoices}
        else:
            res = self._l10n_es_edi_call_web_service_sign(invoices, info_list)

        for inv in invoices:
            if res.get(inv, {}).get('success'):
                attachment = self.env['ir.attachment'].create({
                    'type': 'binary',
                    'name': 'jsondump.json',
                    'raw': json.dumps(info_list),
                    'mimetype': 'application/json',
                    'res_model': inv._name,
                    'res_id': inv.id,
                })
                res[inv]['attachment'] = attachment
        return res

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_edi_is_required = fields.Boolean(
        string="Is the Spanish EDI needed",
        compute='_compute_l10n_es_edi_is_required'
    )
    l10n_es_edi_csv = fields.Char(string="CSV return code", copy=False)
    l10n_es_registration_date = fields.Date(
        string="Registration Date", copy=False,
        help="Technical field to keep the date the invoice was sent the first time as the date the invoice was "
             "registered into the system.",
    )

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('move_type', 'company_id')
    def _compute_l10n_es_edi_is_required(self):
        for move in self:
            move.l10n_es_edi_is_required = move.is_invoice() \
                                           and move.country_code == 'ES' \
                                           and move.company_id.l10n_es_edi_tax_agency

    @api.depends('l10n_es_edi_is_required')
    def _compute_edi_show_cancel_button(self):
        super()._compute_edi_show_cancel_button()
        for move in self.filtered('l10n_es_edi_is_required'):
            move.edi_show_cancel_button = False

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class SIIAccountTaxMixin(models.AbstractModel):
    _name = 'l10n_es.sii.account.tax.mixin'
    _description = 'SII Fields'

    l10n_es_exempt_reason = fields.Selection(
        selection=[
            ('E1', 'Art. 20'),
            ('E2', 'Art. 21'),
            ('E3', 'Art. 22'),
            ('E4', 'Art. 23 y 24'),
            ('E5', 'Art. 25'),
            ('E6', 'Otros'),
        ],
        string="Exempt Reason (Spain)",
    )
    l10n_es_type = fields.Selection(
        selection=[
            ('exento', 'Exento'),
            ('sujeto', 'Sujeto'),
            ('sujeto_agricultura', 'Sujeto Agricultura'),
            ('sujeto_isp', 'Sujeto ISP'),
            ('no_sujeto', 'No Sujeto'),
            ('no_sujeto_loc', 'No Sujeto por reglas de Localization'),
            ('no_deducible', 'No Deducible'),
            ('retencion', 'Retencion'),
            ('recargo', 'Recargo de Equivalencia'),
            ('ignore', 'Ignore even the base amount'),
        ],
        string="Tax Type (Spain)", default='sujeto'
    )
    l10n_es_bien_inversion = fields.Boolean('Bien de Inversion', default=False)


class AccountTax(models.Model):
    _inherit = ['account.tax', 'l10n_es.sii.account.tax.mixin']
    _name = 'account.tax'


class AccountTaxTemplate(models.Model):
    _inherit = ['account.tax.template', 'l10n_es.sii.account.tax.mixin']
    _name = 'account.tax.template'

    def _get_tax_vals(self, company, tax_template_to_tax):
        # OVERRIDE
        # Copy values from 'account.tax.template' to vals will be used to create a new 'account.tax'.
        vals = super()._get_tax_vals(company, tax_template_to_tax)
        vals['l10n_es_exempt_reason'] = self.l10n_es_exempt_reason
        vals['l10n_es_type'] = self.l10n_es_type
        vals['l10n_es_bien_inversion'] = self.l10n_es_bien_inversion
        return vals

```

## File: models\l10n_es_edi_certificate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from pytz import timezone
from datetime import datetime
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.serialization import Encoding, NoEncryption, PrivateFormat, pkcs12


from odoo import _, api, fields, models, tools
from odoo.exceptions import ValidationError


class Certificate(models.Model):
    _name = 'l10n_es_edi.certificate'
    _description = 'Personal Digital Certificate'
    _order = 'date_start desc, id desc'
    _rec_name = 'date_start'

    content = fields.Binary(string="File", required=True, help="PFX Certificate")
    password = fields.Char(help="Passphrase for the PFX certificate", groups="base.group_system")
    date_start = fields.Datetime(readonly=True, help="The date on which the certificate starts to be valid")
    date_end = fields.Datetime(readonly=True, help="The date on which the certificate expires")
    company_id = fields.Many2one(comodel_name='res.company', required=True, default=lambda self: self.env.company)

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    @api.model
    def _get_es_current_datetime(self):
        """Get the current datetime with the Peruvian timezone. """
        return datetime.now(timezone('Europe/Madrid'))

    @tools.ormcache('self.content', 'self.password')
    def _decode_certificate(self):
        """Return the content (DER encoded) and the certificate decrypted based in the point 3.1 from the RS 097-2012
        http://www.vauxoo.com/r/manualdeautorizacion#page=21
        """
        self.ensure_one()

        if not self.password:
            return None, None, None

        private_key, certificate, dummy = pkcs12.load_key_and_certificates(
            b64decode(self.content),
            self.password.encode(),
            backend=default_backend(),
        )

        pem_certificate = certificate.public_bytes(Encoding.PEM)
        pem_private_key = private_key.private_bytes(
            Encoding.PEM,
            format=PrivateFormat.TraditionalOpenSSL,
            encryption_algorithm=NoEncryption(),
        )
        return pem_certificate, pem_private_key, certificate

    # -------------------------------------------------------------------------
    # LOW-LEVEL METHODS
    # -------------------------------------------------------------------------

    @api.model
    def create(self, vals):
        record = super().create(vals)

        spain_tz = timezone('Europe/Madrid')
        spain_dt = self._get_es_current_datetime()
        try:
            pem_certificate, pem_private_key, certificate = record._decode_certificate()
            cert_date_start = spain_tz.localize(certificate.not_valid_before)
            cert_date_end = spain_tz.localize(certificate.not_valid_after)
        except Exception:
            raise ValidationError(_(
                "There has been a problem with the certificate, some usual problems can be:\n"
                "- The password given or the certificate are not valid.\n"
                "- The certificate content is invalid."
            ))
        # Assign extracted values from the certificate
        record.write({
            'date_start': fields.Datetime.to_string(cert_date_start),
            'date_end': fields.Datetime.to_string(cert_date_end),
        })
        if spain_dt > cert_date_end:
            raise ValidationError(_("The certificate is expired since %s", record.date_end))
        return record

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_es_edi_certificate_id = fields.Many2one(
        string="Certificate (ES)",
        store=True,
        readonly=False,
        comodel_name='l10n_es_edi.certificate',
        compute="_compute_l10n_es_edi_certificate",
    )
    l10n_es_edi_certificate_ids = fields.One2many(
        comodel_name='l10n_es_edi.certificate',
        inverse_name='company_id',
    )
    l10n_es_edi_tax_agency = fields.Selection(
        string="Tax Agency for SII",
        selection=[
            ('aeat', "Agencia Tributaria española"),
            ('gipuzkoa', "Hacienda Foral de Gipuzkoa"),
            ('bizkaia', "Hacienda Foral de Bizkaia"),
        ],
        default=False,
    )
    l10n_es_edi_test_env = fields.Boolean(
        string="Test Mode",
        help="Use the test environment",
    )

    @api.depends('country_id', 'l10n_es_edi_certificate_ids')
    def _compute_l10n_es_edi_certificate(self):
        for company in self:
            if company.country_code == 'ES':
                company.l10n_es_edi_certificate_id = self.env['l10n_es_edi.certificate'].search(
                    [('company_id', '=', company.id)],
                    order='date_end desc',
                    limit=1,
                )
            else:
                company.l10n_es_edi_certificate_id = False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_es_edi_certificate_ids = fields.One2many(related='company_id.l10n_es_edi_certificate_ids', readonly=False)
    l10n_es_edi_tax_agency = fields.Selection(related='company_id.l10n_es_edi_tax_agency', readonly=False)
    l10n_es_edi_test_env = fields.Boolean(related='company_id.l10n_es_edi_test_env', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_edi_format
from . import account_move
from . import account_tax
from . import l10n_es_edi_certificate
from . import res_company
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_es_edi_certificate,access_l10n_es_edi_certificate,model_l10n_es_edi_certificate,base.group_system,1,1,1,1

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_tax_form_inherit_l10n_es_edi" model="ir.ui.view">
            <field name="name">account.tax.form.inherit.l10n_es_edi</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='tax_scope']" position="after">
                    <field name="l10n_es_type"
                           attrs="{'invisible': [('country_code', '!=', 'ES')]}"/>
                    <field name="l10n_es_exempt_reason"
                           attrs="{'invisible': [('country_code', '!=', 'ES'), ('l10n_es_type', '!=', 'exento')], 'required': [('l10n_es_type', '=', 'exento'), ('type_tax_use', '=', 'sale')]}"/>
                    <field name="l10n_es_bien_inversion"
                           attrs="{'invisible': [('country_code', '!=', 'ES'), ('l10n_es_type', '!=', 'exento')], 'required': [('l10n_es_type', '=', 'exento'), ('type_tax_use', '=', 'sale')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\l10n_es_edi_certificate_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>

        <record id="l10n_es_edi_certificate_form" model="ir.ui.view">
            <field name="name">l10n_es_edi.certificate.form</field>
            <field name="model">l10n_es_edi.certificate</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <group>
                            <field name="content"/>
                            <field name="password" password="True"/>
                            <label for="date_start" string="Validity"/>
                            <div>
                                <field name="date_start"/> -
                                <field name="date_end"/>
                            </div>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="l10n_es_edi_certificate_tree" model="ir.ui.view">
            <field name="name">l10n_es_edi.certificate.tree</field>
            <field name="model">l10n_es_edi.certificate</field>
            <field name="arch" type="xml">
                <tree>
                    <field name="date_start"/>
                    <field name="date_end"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="l10n_es_edi_certificate_action" model="ir.actions.act_window">
            <field name="name">Certificates for EDI invoices on Spain</field>
            <field name="res_model">l10n_es_edi.certificate</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first certificate</p>
            </field>
        </record>

        <menuitem id="menu_l10n_es_edi_root"
                  name="Spain"
                  sequence="110"
                  groups="account.group_account_manager"
                  parent="account.menu_finance_configuration">
            <menuitem id="menu_l10n_es_edi_certificates"
                      name="Certificates (ES)"
                      action="l10n_es_edi_certificate_action"
                      sequence="100"
                      groups="account.group_account_manager"/>
        </menuitem>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n.es</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='account']/div" position="after">
                <h2 attrs="{'invisible': [('country_code', '!=', 'ES')]}">Spain Localization</h2>
                <div class="row mt16 o_settings_container"
                     name="spain_localization"
                     attrs="{'invisible': [('country_code', '!=', 'ES')]}">
                    <div class="col-xs-12 col-md-6 o_setting_box">

                        <!-- Invisible fields -->
                        <field name="l10n_es_edi_certificate_ids" invisible="1"/>

                        <div class="o_setting_left_pane"/>

                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Registro de Libros connection SII</span>
                            <span class="fa fa-lg fa-building-o"
                                  title="Values set here are company-specific."
                                  groups="base.group_multi_company"/>
                            <div class="content-group">
                                <div class="mt16">
                                    <field name="l10n_es_edi_tax_agency"/>
                                    <br/>
                                    Check this box if test env: <field name="l10n_es_edi_test_env"/>
                                    <p attrs="{'invisible': [('l10n_es_edi_certificate_ids', '!=', [])]}">
                                        Go to Configuration > Certificates [ES] to add your certificate.
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>

        </field>
    </record>
</odoo>

```

