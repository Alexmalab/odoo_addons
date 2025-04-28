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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M3.484 23.012H2.81a.19.19 0 0 1-.195-.187v-1.733a.192.192 0 0 0-.195-.187H1.367a.192.192 0 0 0-.196.187v1.733a.19.19 0 0 1-.195.187h-.78a.191.191 0 0 0-.196.186v.827c0 .103.088.187.195.187h.78a.19.19 0 0 1 .196.186v3.816c0 1.103.744 1.703 1.75 1.703h.563a.192.192 0 0 0 .195-.188v-.923a.178.178 0 0 0-.178-.173H3.14c-.365 0-.525-.293-.525-.67v-3.565a.19.19 0 0 1 .195-.186h.674a.191.191 0 0 0 .195-.187v-.827a.191.191 0 0 0-.195-.186Zm46.479 6.58-2.17-3.089a.2.2 0 0 1 0-.232l2.069-2.95c.097-.137-.007-.323-.181-.323H48.49a.22.22 0 0 0-.186.098l-1.209 1.874a.223.223 0 0 1-.37 0l-1.209-1.873a.218.218 0 0 0-.185-.1H44.14c-.174 0-.278.187-.18.325l2.067 2.95a.2.2 0 0 1 0 .231l-2.17 3.089c-.096.139.007.325.181.325h1.193a.222.222 0 0 0 .184-.096l1.312-1.96a.223.223 0 0 1 .367 0l1.312 1.96a.22.22 0 0 0 .183.095h1.193c.174 0 .278-.185.18-.324Zm-8.236-3.735h-2.633c-.127 0-.233-.106-.217-.226.12-.928.73-1.42 1.541-1.42.811 0 1.41.492 1.527 1.42.016.12-.09.226-.218.226Zm-1.309-2.93c-1.4 0-3.003.866-3.003 3.558 0 2.832 1.735 3.515 3.222 3.515.865 0 1.668-.252 2.338-.834a.204.204 0 0 0 .013-.294l-.632-.67a.224.224 0 0 0-.304-.014c-.426.343-.94.529-1.444.529-.627 0-1.195-.251-1.502-.74a1.699 1.699 0 0 1-.24-.804.214.214 0 0 1 .216-.228h4.108c.12 0 .217-.093.217-.207v-.77c0-1.896-1.385-3.04-2.989-3.04Zm-3.72-2.719h-.72c-1.123 0-1.968.586-1.968 1.869v.73a.208.208 0 0 1-.213.205h-.746a.208.208 0 0 0-.213.203v.793a.21.21 0 0 0 .213.204h.745c.118 0 .214.09.214.203v5.297a.21.21 0 0 0 .212.204h1.017a.209.209 0 0 0 .214-.204v-5.297c0-.112.095-.203.213-.203h1.032a.209.209 0 0 0 .213-.204v-.793a.208.208 0 0 0-.213-.203h-1.032a.209.209 0 0 1-.213-.204v-.591c0-.433.16-.726.641-.726h.604a.208.208 0 0 0 .213-.204v-.875a.208.208 0 0 0-.213-.204ZM31.304 20h-.019c-.514 0-.931.4-.931.892s.417.89.931.89h.019c.515 0 .932-.398.932-.89 0-.493-.417-.892-.932-.892Zm.432 3.012h-.882c-.155 0-.28.12-.28.268v6.368c0 .148.125.268.28.268h.882c.154 0 .28-.12.28-.268V23.28a.275.275 0 0 0-.28-.268Zm-3.968 2.915c-.277-.07-.568-.098-.904-.14-.306-.041-.612-.041-.947-.055-.744-.042-.977-.349-.977-.67 0-.488.32-.85 1.268-.85.712 0 1.24.197 1.758.515a.25.25 0 0 0 .331-.059l.519-.707a.223.223 0 0 0-.05-.317c-.661-.46-1.44-.716-2.557-.716-1.517 0-2.712.711-2.712 2.12 0 1.06.714 1.702 1.763 1.91.525.099 1.065.029 1.59.113.51.083.933.223.933.739 0 .6-.613.907-1.444.907-.893 0-1.538-.238-2.116-.724a.251.251 0 0 0-.35.02l-.573.665a.228.228 0 0 0 .032.329 4.596 4.596 0 0 0 2.89.993c1.692 0 3.004-.795 3.004-2.12 0-1.228-.627-1.73-1.458-1.953Zm-7.795-3c-.801 0-1.435.259-1.895.81-.02.024-.043.03-.043 0v-.489a.256.256 0 0 0-.262-.25h-.92a.256.256 0 0 0-.262.25v6.418c0 .138.117.25.262.25h.92a.256.256 0 0 0 .262-.25V25.8c0-.962.627-1.506 1.458-1.506.83 0 1.458.544 1.458 1.506v3.865c0 .138.117.25.26.25h.92a.256.256 0 0 0 .263-.25v-4.353c0-1.563-1.298-2.385-2.42-2.385Zm-5.92 4.562c0 .405-.088.698-.278.879-.233.237-.7.349-1.385.349-.991 0-1.4-.405-1.4-.893 0-.516.409-.851 1.284-.851h1.779v.516Zm.597-4.073c-.524-.348-1.268-.488-2.186-.488-.935 0-1.655.238-2.173.715a.219.219 0 0 0 .006.32l.618.582a.244.244 0 0 0 .327.002c.263-.231.67-.335 1.294-.335 1.05 0 1.517.223 1.517.892v.753h-2.057c-1.559 0-2.347.92-2.347 2.037 0 .711.322 1.31.92 1.687.422.265.976.419 1.632.419.99 0 1.592-.362 1.793-.698.027-.044.059-.026.059 0v.387c0 .126.105.227.237.227h.954a.232.232 0 0 0 .237-.227v-4.515c0-.837-.306-1.381-.83-1.758Zm-5.124-.17a2.341 2.341 0 0 0-1.204-.318c-.802 0-1.429.265-1.895.809h-.044v-.54c0-.11-.093-.2-.21-.2H5.15c-.116 0-.21.09-.21.2v6.52c0 .11.094.2.21.2h1.025c.116 0 .21-.09.21-.2V25.76c0-1.047.713-1.465 1.428-1.465.328 0 .544.048.801.218.093.06.22.036.285-.052l.685-.931a.195.195 0 0 0-.057-.285Z" fill="#223657"/><path d="M43.105 4h.972V.818h1.108V0H42v.818h1.105V4Zm2.775 0h.858V1.453h.053L47.663 4h.555l.871-2.547h.056V4H50V0h-1.108l-.924 2.714h-.05L46.99 0h-1.11v4Z" fill="#D1D5DB"/></svg>

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
    <t t-name="transifex.CodeTranslationListView.Buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_list_buttons')]" position="inside">
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

