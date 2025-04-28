# Odoo Module: transifex

Category: Hidden/Tools

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
    'category': 'Hidden/Tools',
    'description':
    """
Transifex integration
=====================
This module will add a link to the Transifex project in the translation view.
The purpose of this module is to speed up translations of the main modules.

To work, Odoo uses Transifex configuration files `.tx/config` to detect the
project source. Custom modules will not be translated (as not published on
the main Transifex project).

The language the user tries to translate must be activated on the Transifex
project.
        """,
    'data': [
        'data/transifex_data.xml',
        'views/code_translation_views.xml',
        'security/ir.model.access.csv'
    ],
    'assets': {
        'web.assets_backend': [
            'transifex/static/src/views/fields/translation_dialog.xml',
            'transifex/static/src/views/*.js',
            'transifex/static/src/views/*.xml',
        ],
    },
    'depends': ['base', 'web'],
    'license': 'LGPL-3',
}

```

## File: data\transifex_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="transifex_project_url" model="ir.config_parameter">
        <field name="key">transifex.project_url</field>
        <field name="value">https://app.transifex.com/odoo</field>
    </record>

    <record id="transifex_code_translation_reload" model="ir.cron">
        <field name="name">Transifex: Reload code translations</field>
        <field name="model_id" ref="model_transifex_code_translation"/>
        <field name="state">code</field>
        <field name="code">model.reload()</field>
        <field name='interval_number'>7</field>
        <field name='interval_type'>days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="True"/>
    </record>

</odoo>

```

## File: models\models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class BaseModel(models.AbstractModel):
    _inherit = 'base'

    def get_field_translations(self, field_name, langs=None):
        """ get model/model_term translations for records with transifex url
        :param str field_name: field name
        :param list langs: languages

        :return: (translations, context) where
            translations: list of dicts like [{"lang": lang, "source": source_term, "value": value_term,
                    "module": module, "transifexURL": transifex_url}]
            context: {"translation_type": "text"/"char", "translation_show_source": True/False}
        """
        translations, context = super().get_field_translations(field_name, langs=langs)
        external_id = self.get_external_id().get(self.id)
        if not external_id:
            return translations, context

        module = external_id.split('.')[0]
        if module not in self.pool._init_modules:
            return translations, context

        for translation in translations:
            translation['module'] = module
        self.env['transifex.translation']._update_transifex_url(translations)
        return translations, context

```

## File: models\transifex_code_translation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import psycopg2

from odoo import api, models, fields
from odoo.tools.translate import CodeTranslations


class TransifexCodeTranslation(models.Model):
    _name = "transifex.code.translation"
    _description = "Code Translation"
    _log_access = False

    source = fields.Text(string='Code')
    value = fields.Text(string='Translation Value')
    module = fields.Char(help="Module this term belongs to")
    lang = fields.Selection(selection='_get_languages', string='Language', validate=False)
    transifex_url = fields.Char("Transifex URL", compute='_compute_transifex_url',
                                help="Propose a modification in the official version of Odoo")

    def _get_languages(self):
        return self.env['res.lang'].get_installed()

    def _compute_transifex_url(self):
        self.transifex_url = False
        self.env['transifex.translation']._update_transifex_url(self)

    def _load_code_translations(self, module_names=None, langs=None):
        try:
            # the table lock promises translations for a (module, language) will only be created once
            self.env.cr.execute(f'LOCK TABLE {self._table} IN EXCLUSIVE MODE NOWAIT')

            if module_names is None:
                module_names = self.env['ir.module.module'].search([('state', '=', 'installed')]).mapped('name')
            if langs is None:
                langs = [lang for lang, _ in self._get_languages() if lang != 'en_US']
            self.env.cr.execute(f'SELECT DISTINCT module, lang FROM {self._table}')
            loaded_code_translations = set(self.env.cr.fetchall())
            create_value_list = [
                {
                    'source': src,
                    'value': value,
                    'module': module_name,
                    'lang': lang,
                }
                for module_name in module_names
                for lang in langs
                if (module_name, lang) not in loaded_code_translations
                for src, value in CodeTranslations._get_code_translations(module_name, lang, lambda x: True).items()
            ]
            self.sudo().create(create_value_list)

        except psycopg2.errors.LockNotAvailable:
            return False

        return True

    def _open_code_translations(self):
        self._load_code_translations()
        return {
            'name': 'Code Translations',
            'type': 'ir.actions.act_window',
            'res_model': 'transifex.code.translation',
            'view_mode': 'list',
        }

    @api.model
    def reload(self):
        self.env.cr.execute(f'DELETE FROM {self._table}')
        return self._load_code_translations()

```

## File: models\transifex_translation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import werkzeug.urls
from configparser import ConfigParser
from os import pardir
from os.path import isfile, join as opj

import odoo
from odoo import models, tools


class TransifexTranslation(models.AbstractModel):
    _name = "transifex.translation"
    _description = "Transifex Translation"

    @tools.ormcache()
    def _get_transifex_projects(self):
        """ get the transifex project name for each module

        .tx/config files contains the project reference
        first section is [main], after '[odoo-16.sale]'

        :rtype: dict
        :return: {module_name: tx_project_name}
        """
        tx_config_file = ConfigParser()
        projects = {}
        for addon_path in odoo.addons.__path__:
            for tx_path in (
                    opj(addon_path, '.tx', 'config'),
                    opj(addon_path, pardir, '.tx', 'config'),
            ):
                if isfile(tx_path):
                    tx_config_file.read(tx_path)
                    for sec in tx_config_file.sections()[1:]:
                        if len(sec.split(":")) != 6:
                            # old format ['main', 'odoo-16.base', ...]
                            tx_project, tx_mod = sec.split(".")
                        else:
                            # tx_config_file.sections(): ['main', 'o:odoo:p:odoo-16:r:base', ...]
                            _, _, _, tx_project, _, tx_mod = sec.split(':')
                        projects[tx_mod] = tx_project
        return projects

    def _update_transifex_url(self, translations):
        """ Update translations' Transifex URL

        :param translations: the translations to update, may be a recordset or a list of dicts.
            The elements of `translations` must have the fields/keys 'source', 'module', 'lang',
            and the field/key 'transifex_url' is updated on them.
        """

        # e.g. 'https://www.transifex.com/odoo/'
        base_url = self.env['ir.config_parameter'].sudo().get_param('transifex.project_url')
        if not base_url:
            return
        base_url = base_url.rstrip('/')

        res_langs = self.env['res.lang'].search([])
        lang_to_iso = {l.code: l.iso_code for l in res_langs}
        if not lang_to_iso:
            return

        projects = self._get_transifex_projects()
        if not projects:
            return

        for translation in translations:
            if not translation['source'] or translation['lang'] == 'en_US':
                continue

            lang_iso = lang_to_iso.get(translation['lang'])
            if not lang_iso:
                continue

            project = projects.get(translation['module'])
            if not project:
                continue

            # e.g. https://www.transifex.com/odoo/odoo-16/translate/#fr_FR/sale/42?q=text:'Sale+Order'
            # 42 is an arbitrary number to satisfy the transifex URL format
            source = werkzeug.urls.url_quote_plus(translation['source'][:50].replace("\n", "").replace("'", "\\'"))
            source = f"'{source}'" if "+" in source else source
            translation['transifex_url'] = f"{base_url}/{project}/translate/#{lang_iso}/{translation['module']}/42?q=text%3A{source}"

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import transifex_code_translation
from . import transifex_translation

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_transifex_code_translation,transifex.code.translation,model_transifex_code_translation,base.group_system,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V36.095L18.936 13l5.025 8H30.5L41 28l7.5-7-5.564 10.127L55.577 49 36 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" fill-rule="nonzero" d="M19.077 15h5.5v8h4v5h-4v18h4v5H23.54c-3.026 0-4.513-1.667-4.462-5V28h-4.452v-5h4.452v-8zm11.5 36l9-14-9.5-14h7l5.5 9 6-9h7l-9.5 14 9.5 14h-7l-5.5-8.445-5.5 8.445h-7zm-15.54 2.642H14V53h2.833v.642h-1.037v2.832h-.76v-2.832zM17 53h1.07l.809 2.389h.01L19.652 53h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H17V53z" opacity=".3"/><path fill="#FFF" fill-rule="nonzero" d="M19.077 13h5.5v8h4v5h-4v18h4v5H23.54c-3.026 0-4.513-1.667-4.462-5V26h-4.452v-5h4.452v-8zm11.5 36l9-14-9.5-14h7l5.5 9 6-9h7l-9.5 14 9.5 14h-7l-5.5-8.445-5.5 8.445h-7zm-15.54 2.642H14V51h2.833v.642h-1.037v2.832h-.76v-2.832zM17 51h1.07l.809 2.389h.01L19.652 51h1.07v3.474h-.711v-2.462h-.01l-.847 2.462h-.586l-.848-2.438h-.01v2.438H17V51z"/></g></g></svg>
```

## File: static\src\views\reload_code_translations_views.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { ListController } from "@web/views/list/list_controller";
import { listView } from "@web/views/list/list_view";
import { browser } from "@web/core/browser/browser";
import { useService } from "@web/core/utils/hooks";

export class TransifexCodeTranslationListController extends ListController {
    setup() {
        super.setup();
        this.orm = useService("orm");
    }

    async onClickReloadCodeTranslations() {
        await this.orm.call("transifex.code.translation", "reload", [], {});
        browser.location.reload();
    }
}

registry.category("views").add("transifex_code_translation_tree", {
    ...listView,
    Controller: TransifexCodeTranslationListController,
    buttonTemplate: "transifex.CodeTranslationListView.Buttons",
});

```

## File: static\src\views\reload_code_translations_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="transifex.CodeTranslationListView.Buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary" owl="1">
        <!-- Before the export button -->
        <xpath expr="//t[contains(@t-if, 'isExportEnable')]" position="before">
            <button type="button" class="btn btn-primary" t-on-click="onClickReloadCodeTranslations">Reload</button>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\translation_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

<t t-inherit="web.TranslationDialog" t-inherit-mode="extension">
    <xpath expr="//div[hasclass('row')]/div/t" position="after">
        <a t-if="term.transifex_url" t-attf-href="#{term.transifex_url}" title="Contribute" target="_blank"> <i class="fa fa-globe"/></a>
    </xpath>
</t>

</templates>
```

## File: views\code_translation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="transifex_code_translation_tree_view" model="ir.ui.view">
        <field name="name">transifex.code.translation.tree</field>
        <field name="model">transifex.code.translation</field>
        <field name="arch" type="xml">
            <tree string="Transifex Code Translation" js_class="transifex_code_translation_tree">
                <field name="source"/>
                <field name="value"/>
                <field name="module"/>
                <field name="lang"/>
                <field name="transifex_url"
                       widget="url"
                       text="Contribute"
                       string="Transifex" />
            </tree>
        </field>
    </record>

    <record id="transifex_code_translation_view_search" model="ir.ui.view">
        <field name="name">transifex.code.translation.view.search</field>
        <field name="model">transifex.code.translation</field>
        <field name="arch" type="xml">
            <search string="Search Code Translations">
                <field name="module"/>
                <field name="lang"/>
                <field name="source"/>
                <field name="value"/>
                <separator/>
                <filter string="Not Translated" name="not_translated" domain="[('value', '=', '')]"/>
                <group string="Group By">
                    <filter string="Module" name="group_by_module" context="{'group_by': 'module'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_code_translations" model="ir.actions.server">
        <field name="name">Transifex Code Translations</field>
        <field name="model_id" ref="transifex.model_transifex_code_translation"/>
        <field name="groups_id" eval="[(4, ref('base.group_system'))]"/>
        <field name="state">code</field>
        <field name="code">action = model._open_code_translations()</field>
    </record>
    <menuitem action="action_code_translations" id="menu_transifex_code_translations" parent="base.menu_translation_app"/>

</odoo>

```

