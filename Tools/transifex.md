# Odoo Module: transifex

Category: Tools

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
    'name': 'Transifex integration',
    'version': '1.0',
    'summary': 'Add a link to edit a translation in Transifex',
    'category': 'Tools',
    'description':
    """
Transifex integration
=====================
This module will add a link to the Transifex project in the translation view.
The purpose of this module is to speed up translations of the main modules.

To work, Odoo uses Transifex configuration files `.tx/config` to detec the
project source. Custom modules will not be translated (as not published on
the main Transifex project).

The language the user tries to translate must be activated on the Transifex
project.
        """,
    'data': [
        'data/transifex_data.xml',
        'data/ir_translation_view.xml',
    ],
    'depends': ['base'],
    'license': 'LGPL-3',
}

```

## File: data\ir_translation_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_translation_view_tree_transifex" model="ir.ui.view">
        <field name="name">ir.translation.transifex</field>
        <field name="model">ir.translation</field>
        <field name="inherit_id" ref="base.view_translation_tree"/>
        <field name="arch" type="xml">
            <field name="lang" position="after">
                <field name="transifex_url"
                       widget="link_button"
                       string="Transifex" />
            </field>
        </field>
    </record>

    <record id="ir_translation_dialog_view_tree_transifex" model="ir.ui.view">
        <field name="name">ir.translation.transifex</field>
        <field name="model">ir.translation</field>
        <field name="inherit_id" ref="base.view_translation_dialog_tree"/>
        <field name="arch" type="xml">
            <field name="lang" position="after">
                <field name="transifex_url"
                       widget="link_button"
                       string="Transifex" />
            </field>
        </field>
    </record>
</odoo>

```

## File: data\transifex_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="transifex_project_url" model="ir.config_parameter">
        <field name="key">transifex.project_url</field>
        <field name="value">https://www.transifex.com/odoo</field>
    </record>

</odoo>

```

## File: models\ir_translation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

try:
    from configparser import ConfigParser
except ImportError:
    # python2 import
    from ConfigParser import ConfigParser
from os.path import join as opj
import os
import werkzeug

import odoo
from odoo import models, fields


class IrTranslation(models.Model):

    _inherit = 'ir.translation'

    transifex_url = fields.Char("Transifex URL", compute='_get_transifex_url')

    def _get_transifex_url(self):
        """ Construct transifex URL based on the module on configuration """
        # e.g. 'https://www.transifex.com/odoo/'
        base_url = self.env['ir.config_parameter'].sudo().get_param('transifex.project_url')

        tx_config_file = ConfigParser()
        tx_sections = []
        for addon_path in odoo.addons.__path__:
            tx_path = opj(addon_path, '.tx', 'config')
            if os.path.isfile(tx_path):
                tx_config_file.read(tx_path)
                # first section is [main], after [odoo-11.sale]
                tx_sections.extend(tx_config_file.sections()[1:])

            # parent directory ad .tx/config is root directory in odoo/odoo
            tx_path = opj(addon_path, os.pardir, '.tx', 'config')
            if os.path.isfile(tx_path):
                tx_config_file.read(tx_path)
                tx_sections.extend(tx_config_file.sections()[1:])

        if not base_url or not tx_sections:
            self.update({'transifex_url': False})
        else:
            base_url = base_url.rstrip('/')

            # will probably be the same for all terms, avoid multiple searches
            translation_languages = list(set(self.mapped('lang')))
            languages = self.env['res.lang'].with_context(active_test=False).search(
                [('code', 'in', translation_languages)])

            language_codes = dict((l.code, l.iso_code) for l in languages)

            # .tx/config files contains the project reference
            # using ini files like '[odoo-master.website_sale]'
            translation_modules = set(self.mapped('module'))
            project_modules = {}
            for module in translation_modules:
                for section in tx_sections:
                    tx_project, tx_mod = section.split('.')
                    if tx_mod == module:
                        project_modules[module] = tx_project

            for translation in self:
                if not translation.module or not translation.src or translation.lang == 'en_US':
                    # custom or source term
                    translation.transifex_url = False
                    continue

                lang_code = language_codes.get(translation.lang)
                if not lang_code:
                    translation.transifex_url = False
                    continue

                project = project_modules.get(translation.module)
                if not project:
                    translation.transifex_url = False
                    continue

                # e.g. https://www.transifex.com/odoo/odoo-10/translate/#fr/sale/42?q=text'Sale+Order'
                translation.transifex_url = "%(url)s/%(project)s/translate/#%(lang)s/%(module)s/42?q=%(src)s" % {
                    'url': base_url,
                    'project': project,
                    'lang': lang_code,
                    'module': translation.module,
                    'src': "text:'" + werkzeug.url_quote_plus(
                               translation.src[:50].replace("\n", "").replace("'", "")
                           ) + "'",
                }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_translation

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.095L18.936 13l5.025 8H30.5L41 28l7.5-7-5.564 10.127L55.577 49 36 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" fill-rule="nonzero" d="M19.077 15h5.5v8h4v5h-4v18h4v5H23.54c-3.026 0-4.513-1.667-4.462-5V28h-4.452v-5h4.452v-8zm11.5 36l9-14-9.5-14h7l5.5 9 6-9h7l-9.5 14 9.5 14h-7l-5.5-8.445-5.5 8.445h-7zm-15.54 2.642H14V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17 53h1.07l.809 2.389h.01L19.652 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H17V53z" opacity=".3"/><path fill="#FFF" fill-rule="nonzero" d="M19.077 13h5.5v8h4v5h-4v18h4v5H23.54c-3.026 0-4.513-1.667-4.462-5V26h-4.452v-5h4.452v-8zm11.5 36l9-14-9.5-14h7l5.5 9 6-9h7l-9.5 14 9.5 14h-7l-5.5-8.445-5.5 8.445h-7zm-15.54 2.642H14V51h2.833v.642h-1.037v2.832h-.76v-2.832zM17 51h1.07l.809 2.389h.01L19.652 51h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H17V51z"/></g></g></svg>
```

