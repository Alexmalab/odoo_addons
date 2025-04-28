# Odoo Module: website_theme_install

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
{
    'name': 'Website Theme Install',
    'summary': 'Select a theme for your website',
    'description': "",
    'category': 'Hidden',
    'version': '1.0',
    'data': [
        'views/assets.xml',
        'views/views.xml',
        'views/res_config_settings_views.xml',
        'security/ir.model.access.csv',
    ],
    'depends': ['website'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\ir_module_module.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import os
from collections import OrderedDict

from odoo import api, fields, models
from odoo.addons.base.models.ir_model import MODULE_UNINSTALL_FLAG
from odoo.exceptions import MissingError
from odoo.http import request

_logger = logging.getLogger(__name__)


class IrModuleModule(models.Model):
    _name = "ir.module.module"
    _description = 'Module'
    _inherit = _name

    # The order is important because of dependencies (page need view, menu need page)
    _theme_model_names = OrderedDict([
        ('ir.ui.view', 'theme.ir.ui.view'),
        ('website.page', 'theme.website.page'),
        ('website.menu', 'theme.website.menu'),
        ('ir.attachment', 'theme.ir.attachment'),
    ])
    _theme_translated_fields = {
        'theme.ir.ui.view': [('theme.ir.ui.view,arch', 'ir.ui.view,arch_db')],
        'theme.website.menu': [('theme.website.menu,name', 'website.menu,name')],
    }

    image_ids = fields.One2many('ir.attachment', 'res_id',
                                domain=[('res_model', '=', _name), ('mimetype', '=like', 'image/%')],
                                string='Screenshots', readonly=True)
    # for kanban view
    is_installed_on_current_website = fields.Boolean(compute='_compute_is_installed_on_current_website')

    def _compute_is_installed_on_current_website(self):
        """
            Compute for every theme in ``self`` if the current website is using it or not.

            This method does not take dependencies into account, because if it did, it would show
            the current website as having multiple different themes installed at the same time,
            which would be confusing for the user.
        """
        for module in self:
            module.is_installed_on_current_website = module == self.env['website'].get_current_website().theme_id

    def write(self, vals):
        """
            Override to correctly upgrade themes after upgrade/installation of modules.

            # Install

                If this theme wasn't installed before, then load it for every website
                for which it is in the stream.

                eg. The very first installation of a theme on a website will trigger this.

                eg. If a website uses theme_A and we install sale, then theme_A_sale will be
                    autoinstalled, and in this case we need to load theme_A_sale for the website.

            # Upgrade

                There are 2 cases to handle when upgrading a theme:

                * When clicking on the theme upgrade button on the interface,
                    in which case there will be an http request made.

                    -> We want to upgrade the current website only, not any other.

                * When upgrading with -u, in which case no request should be set.

                    -> We want to upgrade every website using this theme.
        """
        for module in self:
            if module.name.startswith('theme_') and vals.get('state') == 'installed':
                _logger.info('Module %s has been loaded as theme template (%s)' % (module.name, module.state))

                if module.state in ['to install', 'to upgrade']:
                    websites_to_update = module._theme_get_stream_website_ids()

                    if module.state == 'to upgrade' and request:
                        Website = self.env['website']
                        current_website = Website.get_current_website()
                        websites_to_update = current_website if current_website in websites_to_update else Website

                    for website in websites_to_update:
                        module._theme_load(website)

        return super(IrModuleModule, self).write(vals)

    def _get_module_data(self, model_name):
        """
            Return every theme template model of type ``model_name`` for every theme in ``self``.

            :param model_name: string with the technical name of the model for which to get data.
                (the name must be one of the keys present in ``_theme_model_names``)
            :return: recordset of theme template models (of type defined by ``model_name``)
        """
        theme_model_name = self._theme_model_names[model_name]
        IrModelData = self.env['ir.model.data']
        records = self.env[theme_model_name]

        for module in self:
            imd_ids = IrModelData.search([('module', '=', module.name), ('model', '=', theme_model_name)]).mapped('res_id')
            records |= self.env[theme_model_name].with_context(active_test=False).browse(imd_ids)
        return records

    def _update_records(self, model_name, website):
        """
            This method:

            - Find and update existing records.

                For each model, overwrite the fields that are defined in the template (except few
                cases such as active) but keep inherited models to not lose customizations.

            - Create new records from templates for those that didn't exist.

            - Remove the models that existed before but are not in the template anymore.

                See _theme_cleanup for more information.


            There is a special 'while' loop around the 'for' to be able queue back models at the end
            of the iteration when they have unmet dependencies. Hopefully the dependency will be
            found after all models have been processed, but if it's not the case an error message will be shown.


            :param model_name: string with the technical name of the model to handle
                (the name must be one of the keys present in ``_theme_model_names``)
            :param website: ``website`` model for which the records have to be updated

            :raise MissingError: if there is a missing dependency.
        """
        self.ensure_one()

        remaining = self._get_module_data(model_name)
        last_len = -1
        while (len(remaining) != last_len):
            last_len = len(remaining)
            for rec in remaining:
                rec_data = rec._convert_to_base_model(website)
                if not rec_data:
                    _logger.info('Record queued: %s' % rec.display_name)
                    continue

                find = rec.with_context(active_test=False).mapped('copy_ids').filtered(lambda m: m.website_id == website)

                # special case for attachment
                # if module B override attachment from dependence A, we update it
                if not find and model_name == 'ir.attachment':
                    find = rec.copy_ids.search([('key', '=', rec.key), ('website_id', '=', website.id)])

                if find:
                    imd = self.env['ir.model.data'].search([('model', '=', find._name), ('res_id', '=', find.id)])
                    if imd and imd.noupdate:
                        _logger.info('Noupdate set for %s (%s)' % (find, imd))
                    else:
                        # at update, ignore active field
                        if 'active' in rec_data:
                            rec_data.pop('active')
                        if model_name == 'ir.ui.view' and (find.arch_updated or find.arch == rec_data['arch']):
                            rec_data.pop('arch')
                        find.update(rec_data)
                        self._post_copy(rec, find)
                else:
                    new_rec = self.env[model_name].create(rec_data)
                    self._post_copy(rec, new_rec)

                remaining -= rec

        if len(remaining):
            error = 'Error - Remaining: %s' % remaining.mapped('display_name')
            _logger.error(error)
            raise MissingError(error)

        self._theme_cleanup(model_name, website)

    def _post_copy(self, old_rec, new_rec):
        self.ensure_one()
        translated_fields = self._theme_translated_fields.get(old_rec._name, [])
        for (src_field, dst_field) in translated_fields:
            self._cr.execute("""INSERT INTO ir_translation (lang, src, name, res_id, state, value, type, module)
                                SELECT t.lang, t.src, %s, %s, t.state, t.value, t.type, t.module
                                FROM ir_translation t
                                WHERE name = %s
                                  AND res_id = %s
                                ON CONFLICT DO NOTHING""",
                             (dst_field, new_rec.id, src_field, old_rec.id))

    def _theme_load(self, website):
        """
            For every type of model in ``self._theme_model_names``, and for every theme in ``self``:
            create/update real models for the website ``website`` based on the theme template models.

            :param website: ``website`` model on which to load the themes
        """
        for module in self:
            _logger.info('Load theme %s for website %s from template.' % (module.mapped('name'), website.id))

            for model_name in self._theme_model_names:
                module._update_records(model_name, website)

            self.env['theme.utils']._post_copy(module, website)

    def _theme_unload(self, website):
        """
            For every type of model in ``self._theme_model_names``, and for every theme in ``self``:
            remove real models that were generated based on the theme template models
            for the website ``website``.

            :param website: ``website`` model on which to unload the themes
        """
        for module in self:
            _logger.info('Unload theme %s for website %s from template.' % (self.mapped('name'), website.id))

            for model_name in self._theme_model_names:
                template = self._get_module_data(model_name)
                models = template.with_context(**{'active_test': False, MODULE_UNINSTALL_FLAG: True}).mapped('copy_ids').filtered(lambda m: m.website_id == website)
                models.unlink()
                self._theme_cleanup(model_name, website)

    def _theme_cleanup(self, model_name, website):
        """
            Remove orphan models of type ``model_name`` from the current theme and
            for the website ``website``.

            We need to compute it this way because if the upgrade (or deletion) of a theme module
            removes a model template, then in the model itself the variable
            ``theme_template_id`` will be set to NULL and the reference to the theme being removed
            will be lost. However we do want the ophan to be deleted from the website when
            we upgrade or delete the theme from the website.

            ``website.page`` and ``website.menu`` don't have ``key`` field so we don't clean them.
            TODO in master: add a field ``theme_id`` on the models to more cleanly compute orphans.

            :param model_name: string with the technical name of the model to cleanup
                (the name must be one of the keys present in ``_theme_model_names``)
            :param website: ``website`` model for which the models have to be cleaned

        """
        self.ensure_one()
        model = self.env[model_name]

        if model_name in ('website.page', 'website.menu'):
            return model
        # use active_test to also unlink archived models
        # and use MODULE_UNINSTALL_FLAG to also unlink inherited models
        orphans = model.with_context(**{'active_test': False, MODULE_UNINSTALL_FLAG: True}).search([
            ('key', '=like', self.name + '.%'),
            ('website_id', '=', website.id),
            ('theme_template_id', '=', False),
        ])
        orphans.unlink()

    def _theme_get_upstream(self):
        """
            Return installed upstream themes.

            :return: recordset of themes ``ir.module.module``
        """
        self.ensure_one()
        return self.upstream_dependencies(exclude_states=('',)).filtered(lambda x: x.name.startswith('theme_'))

    def _theme_get_downstream(self):
        """
            Return installed downstream themes that starts with the same name.

            eg. For theme_A, this will return theme_A_sale, but not theme_B even if theme B
                depends on theme_A.

            :return: recordset of themes ``ir.module.module``
        """
        self.ensure_one()
        return self.downstream_dependencies().filtered(lambda x: x.name.startswith(self.name))

    def _theme_get_stream_themes(self):
        """
            Returns all the themes in the stream of the current theme.

            First find all its downstream themes, and all of the upstream themes of both
            sorted by their level in hierarchy, up first.

            :return: recordset of themes ``ir.module.module``
        """
        self.ensure_one()
        all_mods = self + self._theme_get_downstream()
        for down_mod in self._theme_get_downstream() + self:
            for up_mod in down_mod._theme_get_upstream():
                all_mods = up_mod | all_mods
        return all_mods

    def _theme_get_stream_website_ids(self):
        """
            Websites for which this theme (self) is in the stream (up or down) of their theme.

            :return: recordset of websites ``website``
        """
        self.ensure_one()
        websites = self.env['website']
        for website in websites.search([('theme_id', '!=', False)]):
            if self in website.theme_id._theme_get_stream_themes():
                websites |= website
        return websites

    def _theme_upgrade_upstream(self):
        """ Upgrade the upstream dependencies of a theme, and install it if necessary. """
        def install_or_upgrade(theme):
            if theme.state != 'installed':
                theme.button_install()
            themes = theme + theme._theme_get_upstream()
            themes.filtered(lambda m: m.state == 'installed').button_upgrade()

        self._button_immediate_function(install_or_upgrade)

    @api.model
    def _theme_remove(self, website):
        """
            Remove from ``website`` its current theme, including all the themes in the stream.

            The order of removal will be reverse of installation to handle dependencies correctly.

            :param website: ``website`` model for which the themes have to be removed
        """

        # _theme_remove is the entry point of any change of theme for a website
        # (either removal or installation of a theme and its dependencies). In
        # either case, we need to reset some default configuration before.
        self.env['theme.utils'].with_context(website_id=website.id)._reset_default_config()

        if not website.theme_id:
            return

        for theme in reversed(website.theme_id._theme_get_stream_themes()):
            theme._theme_unload(website)
        website.theme_id = False

    def button_choose_theme(self):
        """
            Remove any existing theme on the current website and install the theme ``self`` instead.

            The actual loading of the theme on the current website will be done
            automatically on ``write`` thanks to the upgrade and/or install.

            When installating a new theme, upgrade the upstream chain first to make sure
            we have the latest version of the dependencies to prevent inconsistencies.

            :return: dict with the next action to execute
        """
        self.ensure_one()
        website = self.env['website'].get_current_website()

        self._theme_remove(website)

        # website.theme_id must be set before upgrade/install to trigger the load in ``write``
        website.theme_id = self

        # this will install 'self' if it is not installed yet
        self._theme_upgrade_upstream()

        return website.button_go_website()

    def button_remove_theme(self):
        """Remove the current theme of the current website."""
        website = self.env['website'].get_current_website()
        self._theme_remove(website)

    def button_refresh_theme(self):
        """
            Refresh the current theme of the current website.

            To refresh it, we only need to upgrade the modules.
            Indeed the (re)loading of the theme will be done automatically on ``write``.
        """
        website = self.env['website'].get_current_website()
        website.theme_id._theme_upgrade_upstream()

    @api.model
    def update_list(self):
        res = super(IrModuleModule, self).update_list()

        IrAttachment = self.env['ir.attachment']
        existing_urls = IrAttachment.search_read([['res_model', '=', self._name], ['type', '=', 'url']], ['url'])
        existing_urls = [url_wrapped['url'] for url_wrapped in existing_urls]

        for app in self.search([]):
            terp = self.get_module_info(app.name)
            images = terp.get('images', [])
            for image in images:
                image_path = os.path.join(app.name, image)
                if image_path not in existing_urls:
                    image_name = os.path.basename(image_path)
                    IrAttachment.create({
                        'type': 'url',
                        'name': image_name,
                        'url': image_path,
                        'res_model': self._name,
                        'res_id': app.id,
                    })
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from ast import literal_eval


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _compute_website_theme_onboarding_done(self):
        """ The step is marked as done if one theme is installed. """
        # we need the same domain as the existing action
        action = self.env.ref('website_theme_install.theme_install_kanban_action').read()[0]
        domain = literal_eval(action['domain'])
        domain.append(('state', '=', 'installed'))
        installed_themes_count = self.env['ir.module.module'].sudo().search_count(domain)
        for record in self:
            record.website_theme_onboarding_done = (installed_themes_count > 0)

    website_theme_onboarding_done = fields.Boolean("Onboarding website theme step done",
                                                   compute='_compute_website_theme_onboarding_done')

    @api.model
    def action_open_website_theme_selector(self):
        action = self.env.ref('website_theme_install.theme_install_kanban_action').read()[0]
        action['target'] = 'new'
        return action

```

## File: models\res_config_settings.py

```python
# coding: utf-8
from odoo import models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def install_theme_on_current_website(self):
        self.website_id._force()
        action = self.env.ref('website_theme_install.theme_install_kanban_action')
        return action.read()[0]

    def action_website_create_new(self):
        res = super(ResConfigSettings, self).action_website_create_new()
        res['view_id'] = self.env.ref('website_theme_install.view_website_form_view_themes_modal').id
        return res

```

## File: models\theme_models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import api, fields, models
from odoo.tools.translate import xml_translate
from odoo.modules.module import get_resource_from_path

_logger = logging.getLogger(__name__)


class ThemeView(models.Model):
    _name = 'theme.ir.ui.view'
    _description = 'Theme UI View'

    def compute_arch_fs(self):
        if 'install_filename' not in self._context:
            return ''
        path_info = get_resource_from_path(self._context['install_filename'])
        if path_info:
            return '/'.join(path_info[0:2])

    name = fields.Char(required=True)
    key = fields.Char()
    type = fields.Char()
    priority = fields.Integer(default=16, required=True)
    mode = fields.Selection([('primary', "Base view"), ('extension', "Extension View")])
    active = fields.Boolean(default=True)
    arch = fields.Text(translate=xml_translate)
    arch_fs = fields.Char(default=compute_arch_fs)
    inherit_id = fields.Reference(selection=[('ir.ui.view', 'ir.ui.view'), ('theme.ir.ui.view', 'theme.ir.ui.view')])
    copy_ids = fields.One2many('ir.ui.view', 'theme_template_id', 'Views using a copy of me', copy=False, readonly=True)

    # TODO master add missing field: customize_show

    def _convert_to_base_model(self, website, **kwargs):
        self.ensure_one()
        inherit = self.inherit_id
        if self.inherit_id and self.inherit_id._name == 'theme.ir.ui.view':
            inherit = self.inherit_id.with_context(active_test=False).copy_ids.filtered(lambda x: x.website_id == website)
            if not inherit:
                # inherit_id not yet created, add to the queue
                return False

        if inherit and inherit.website_id != website:
            website_specific_inherit = self.env['ir.ui.view'].with_context(active_test=False).search([
                ('key', '=', inherit.key),
                ('website_id', '=', website.id)
            ], limit=1)
            if website_specific_inherit:
                inherit = website_specific_inherit

        new_view = {
            'type': self.type or 'qweb',
            'name': self.name,
            'arch': self.arch,
            'key': self.key,
            'inherit_id': inherit and inherit.id,
            'arch_fs': self.arch_fs,
            'priority': self.priority,
            'active': self.active,
            'theme_template_id': self.id,
            'website_id': website.id,
        }

        if self.mode:  # if not provided, it will be computed automatically (if inherit_id or not)
            new_view['mode'] = self.mode

        return new_view


class ThemeAttachment(models.Model):
    _name = 'theme.ir.attachment'
    _description = 'Theme Attachments'

    name = fields.Char(required=True)
    key = fields.Char(required=True)
    url = fields.Char()
    copy_ids = fields.One2many('ir.attachment', 'theme_template_id', 'Attachment using a copy of me', copy=False, readonly=True)


    def _convert_to_base_model(self, website, **kwargs):
        self.ensure_one()
        new_attach = {
            'key': self.key,
            'public': True,
            'res_model': 'ir.ui.view',
            'type': 'url',
            'name': self.name,
            'url': self.url,
            'website_id': website.id,
            'theme_template_id': self.id,
        }
        return new_attach


class ThemeMenu(models.Model):
    _name = 'theme.website.menu'
    _description = 'Website Theme Menu'

    name = fields.Char(required=True, translate=True)
    url = fields.Char(default='')
    page_id = fields.Many2one('theme.website.page', ondelete='cascade')
    new_window = fields.Boolean('New Window')
    sequence = fields.Integer()
    parent_id = fields.Many2one('theme.website.menu', index=True, ondelete="cascade")
    copy_ids = fields.One2many('website.menu', 'theme_template_id', 'Menu using a copy of me', copy=False, readonly=True)

    def _convert_to_base_model(self, website, **kwargs):
        self.ensure_one()
        page_id = self.page_id.copy_ids.filtered(lambda x: x.website_id == website)
        parent_id = self.copy_ids.filtered(lambda x: x.website_id == website)
        new_menu = {
            'name': self.name,
            'url': self.url,
            'page_id': page_id and page_id.id or False,
            'new_window': self.new_window,
            'sequence': self.sequence,
            'parent_id': parent_id and parent_id.id or False,
            'theme_template_id': self.id,
        }
        return new_menu


class ThemePage(models.Model):
    _name = 'theme.website.page'
    _description = 'Website Theme Page'

    url = fields.Char()
    view_id = fields.Many2one('theme.ir.ui.view', required=True, ondelete="cascade")
    website_indexed = fields.Boolean('Page Indexed', default=True)
    copy_ids = fields.One2many('website.page', 'theme_template_id', 'Page using a copy of me', copy=False, readonly=True)

    def _convert_to_base_model(self, website, **kwargs):
        self.ensure_one()
        view_id = self.view_id.copy_ids.filtered(lambda x: x.website_id == website)
        if not view_id:
            # inherit_id not yet created, add to the queue
            return False

        new_page = {
            'url': self.url,
            'view_id': view_id.id,
            'website_indexed': self.website_indexed,
            'theme_template_id': self.id,
        }
        return new_page


class Theme(models.AbstractModel):
    _name = 'theme.utils'
    _description = 'Theme Utils'
    _auto = False

    def _post_copy(self, mod, website=False):
        # deprecated: to remove in master
        if not website:  # remove optional website in master
            website = self.env['website'].get_current_website()

        # Call specific theme post copy
        theme_post_copy = '_%s_post_copy' % mod.name
        if hasattr(self, theme_post_copy):
            _logger.info('Executing method %s' % theme_post_copy)
            method = getattr(self.with_context(website_id=website.id), theme_post_copy)
            return method(mod)
        return False

    @api.model
    def _reset_default_config(self):
        # Reinitialize font customizations
        self.env['web_editor.assets'].make_scss_customization(
            '/website/static/src/scss/options/user_values.scss',
            {
                'font-number': 'null',
                'headings-font-number': 'null',
                'navbar-font-number': 'null',
                'buttons-font-number': 'null',
            }
        )

    @api.model
    def _toggle_view(self, xml_id, active):
        obj = self.env.ref(xml_id)
        website = self.env['website'].get_current_website()
        if obj._name == 'theme.ir.ui.view':
            obj = obj.with_context(active_test=False)
            obj = obj.copy_ids.filtered(lambda x: x.website_id == website)
        else:
            # If a theme post copy wants to enable/disable a view, this is to
            # enable/disable a given functionality which is disabled/enabled
            # by default. So if a post copy asks to enable/disable a view which
            # is already enabled/disabled, we would not consider it otherwise it
            # would COW the view for nothing.
            View = self.env['ir.ui.view'].with_context(active_test=False)
            has_specific = obj.key and View.search_count([
                ('key', '=', obj.key),
                ('website_id', '=', website.id)
            ]) >= 1
            if not has_specific and active == obj.active:
                return
        obj.write({'active': active})

    @api.model
    def enable_view(self, xml_id):
        self._toggle_view(xml_id, True)

    @api.model
    def disable_view(self, xml_id):
        self._toggle_view(xml_id, False)


class IrUiView(models.Model):
    _inherit = 'ir.ui.view'

    theme_template_id = fields.Many2one('theme.ir.ui.view', copy=False)

    def write(self, vals):
        no_arch_updated_views = other_views = self.env['ir.ui.view']
        for record in self:
            # Do not mark the view as user updated if original view arch is similar
            arch = vals.get('arch', vals.get('arch_base'))
            if record.theme_template_id and record.theme_template_id.arch == arch:
                no_arch_updated_views += record
            else:
                other_views += record
        res = super(IrUiView, other_views).write(vals)
        if no_arch_updated_views:
            vals['arch_updated'] = False
            res &= super(IrUiView, no_arch_updated_views).write(vals)
        return res

class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    key = fields.Char(copy=False)
    theme_template_id = fields.Many2one('theme.ir.attachment', copy=False)


class WebsiteMenu(models.Model):
    _inherit = 'website.menu'

    theme_template_id = fields.Many2one('theme.website.menu', copy=False)


class WebsitePage(models.Model):
    _inherit = 'website.page'

    theme_template_id = fields.Many2one('theme.website.page', copy=False)

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class Website(models.Model):
    _name = "website"
    _inherit = _name

    @api.model
    def create_and_redirect_to_theme(self, vals):
        self.browse(vals)._force()
        action = self.env.ref('website_theme_install.theme_install_kanban_action')
        return action.read()[0]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_module_module
from . import res_company
from . import res_config_settings
from . import theme_models
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_theme_ir_ui_view,access_theme_ir_ui_view,model_theme_ir_ui_view,base.group_system,1,1,1,1
access_theme_ir_attachment,access_theme_ir_attachment,model_theme_ir_attachment,base.group_system,1,1,1,1
access_theme_website_menu,access_theme_website_menu,model_theme_website_menu,base.group_system,1,1,1,1
access_theme_website_page,access_theme_website_page,model_theme_website_page,base.group_system,1,1,1,1


```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="70" height="70" viewBox="0 0 70 70"><defs><linearGradient id="a" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><path fill="url(#a)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M4 69c-2 0-4-1-4-4V35.319l18.052-20.27L48 16l1 22-5.578 7h.992l2.384-2.976L52 45l1.373 8.198L40.598 69H4z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M49 40.05a10.12 10.12 0 0 0-1-.05c-5.523 0-10 4.477-10 10 0 .337.017.671.05 1H18V17h31v23.05zM26 19v3h21v-3H26zm-6 3h5v-3h-5v3zm13.543 17.242a4.846 4.846 0 0 1-3.234-2.676c-1.813.036-3.489.594-4.41 3.024a.633.633 0 0 1-.606.414c-.461 0-1.887-1.149-2.293-1.426C23 41.996 24.621 45 28.36 45c3.148 0 5.265-1.816 5.265-4.988a3.6 3.6 0 0 0-.082-.77zM40.993 25c-.563 0-1.09.25-1.493.613-7.586 6.809-8.375 6.969-8.375 8.973 0 1.906 1.582 3.539 3.523 3.539 2.305 0 3.641-1.695 7.836-9.563.274-.535.516-1.113.516-1.714C43 25.77 42.035 25 40.992 25zm12.73 26.471l1.18.68a.333.333 0 0 1 .15.388 6.851 6.851 0 0 1-1.512 2.617.332.332 0 0 1-.41.062l-1.179-.68a5.3 5.3 0 0 1-1.681.972v1.36c0 .156-.108.29-.26.325a6.916 6.916 0 0 1-3.022 0 .333.333 0 0 1-.26-.324V55.51a5.3 5.3 0 0 1-1.681-.972l-1.178.68a.332.332 0 0 1-.41-.062 6.852 6.852 0 0 1-1.514-2.617.333.333 0 0 1 .151-.387l1.18-.68a5.353 5.353 0 0 1 0-1.943l-1.18-.68a.333.333 0 0 1-.15-.388 6.851 6.851 0 0 1 1.512-2.617.332.332 0 0 1 .41-.062l1.179.68a5.3 5.3 0 0 1 1.681-.972v-1.36c0-.156.108-.29.26-.325a6.916 6.916 0 0 1 3.022 0c.152.034.26.169.26.324v1.361a5.3 5.3 0 0 1 1.681.972l1.178-.68a.332.332 0 0 1 .41.062 6.852 6.852 0 0 1 1.514 2.617.333.333 0 0 1-.151.387l-1.18.68c.12.643.12 1.301 0 1.943zm-3.01-.971c0-1.22-.992-2.214-2.213-2.214-1.22 0-2.214.993-2.214 2.214 0 1.22.993 2.214 2.214 2.214 1.22 0 2.214-.993 2.214-2.214z" opacity=".3"/><path fill="#FFF" d="M49 38.05a10.12 10.12 0 0 0-1-.05c-5.523 0-10 4.477-10 10 0 .337.017.671.05 1H18V15h31v23.05zM26 17v3h21v-3H26zm-6 3h5v-3h-5v3zm13.543 17.242a4.846 4.846 0 0 1-3.234-2.676c-1.813.036-3.489.594-4.41 3.024a.633.633 0 0 1-.606.414c-.461 0-1.887-1.149-2.293-1.426C23 39.996 24.621 43 28.36 43c3.148 0 5.265-1.816 5.265-4.988a3.6 3.6 0 0 0-.082-.77zM40.993 23c-.563 0-1.09.25-1.493.613-7.586 6.809-8.375 6.969-8.375 8.973 0 1.906 1.582 3.539 3.523 3.539 2.305 0 3.641-1.695 7.836-9.563.274-.535.516-1.113.516-1.714C43 23.77 42.035 23 40.992 23zm12.73 26.471l1.18.68a.333.333 0 0 1 .15.388 6.851 6.851 0 0 1-1.512 2.617.332.332 0 0 1-.41.062l-1.179-.68a5.3 5.3 0 0 1-1.681.972v1.36c0 .156-.108.29-.26.325a6.916 6.916 0 0 1-3.022 0 .333.333 0 0 1-.26-.324V53.51a5.3 5.3 0 0 1-1.681-.972l-1.178.68a.332.332 0 0 1-.41-.062 6.852 6.852 0 0 1-1.514-2.617.333.333 0 0 1 .151-.387l1.18-.68a5.353 5.353 0 0 1 0-1.943l-1.18-.68a.333.333 0 0 1-.15-.388 6.851 6.851 0 0 1 1.512-2.617.332.332 0 0 1 .41-.062l1.179.68a5.3 5.3 0 0 1 1.681-.972v-1.36c0-.156.108-.29.26-.325a6.916 6.916 0 0 1 3.022 0c.152.034.26.169.26.324v1.361a5.3 5.3 0 0 1 1.681.972l1.178-.68a.332.332 0 0 1 .41.062 6.852 6.852 0 0 1 1.514 2.617.333.333 0 0 1-.151.387l-1.18.68c.12.643.12 1.301 0 1.943zm-3.01-.971c0-1.22-.992-2.214-2.213-2.214-1.22 0-2.214.993-2.214 2.214 0 1.22.993 2.214 2.214 2.214 1.22 0 2.214-.993 2.214-2.214z"/></g></svg>
```

## File: static\src\js\res_config_settings.js

```javascript
odoo.define('website_theme_install.settings', function (require) {

var BaseSettingController = require('base.settings').Controller;
var FormController = require('web.FormController');

BaseSettingController.include({

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Bypasses the discard confirmation dialog when selecting a theme because
     * the theme will be installed on the selected website.
     *
     * Without this override, it is impossible to install a theme on a website
     * other than the first because discarding will revert it back to the
     * default value.
     *
     * @override
     */
    _onButtonClicked: function (ev) {
        if (ev.data.attrs.name === 'install_theme_on_current_website') {
            FormController.prototype._onButtonClicked.apply(this, arguments);
        } else {
            this._super.apply(this, arguments);
        }
    },
});
});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="website_theme_install_assets" inherit_id="web.assets_backend" name="Website Theme Install Assets">
        <xpath expr="//link[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/website_theme_install/static/src/scss/website_theme_install.scss"/>
        </xpath>
        <xpath expr="//script[last()]" position="after">
	        <script type="text/javascript" src="/website_theme_install/static/src/js/res_config_settings.js"/>
	    </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website_theme_install</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='website_go_to']" position="after">
                <button name="install_theme_on_current_website" type="object" string="Choose a theme" class="ml-2 btn btn-primary" icon="fa-paint-brush"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Custom module kanban : install button (even if already installed) which redirects to website after (fake or not) installation + live preview button -->
    <record model="ir.ui.view" id="theme_view_kanban">
        <field name="name">Themes Kanban</field>
        <field name="model">ir.module.module</field>
        <field name="arch" type="xml">
            <kanban create="false" class="o_theme_kanban" default_order="state,sequence,name">
                <field name="icon"/>
                <field name="name"/>
                <field name="state"/>
                <field name="url"/>
                <field name="image_ids"/>
                <field name="category_id"/>
                <field name="display_name"/>
                <field name="is_installed_on_current_website"/>
                <templates>
                    <div t-name="kanban-box" t-attf-class="o_theme_preview mb16 mt16 #{record.is_installed_on_current_website.raw_value? 'o_theme_installed' : ''}">
                        <t t-set="has_image" t-value="record.image_ids.raw_value.length > 0"/>
                        <t t-set="has_screenshot" t-value="record.image_ids.raw_value.length > 1"/>
                        <t t-set="image_url" t-value="has_image ? 'web/image/' + record.image_ids.raw_value[0] : record.icon.value"/>

                        <div class="o_theme_preview_top bg-white mb4">
                            <div t-attf-class="bg-gray-lighter #{has_screenshot? 'o_theme_screenshot' : (has_image ? 'o_theme_cover' : 'o_theme_logo')}" t-attf-style="background-image: url(#{image_url});"/>
                            <div t-if="record.is_installed_on_current_website.raw_value" class="o_button_area">
                                <button type="object" name="button_refresh_theme" class="btn btn-primary">Update theme</button>
                                <hr />
                                <button type="object" name="button_remove_theme" class="btn btn-secondary">Remove theme</button>
                            </div>
                            <div t-else="" class="o_button_area">
                                <button type="object" name="button_choose_theme" class="btn btn-primary">Use this theme</button>
                                <hr t-if="record.url.value"/>
                                <a role="button" t-if="record.url.value" class="btn btn-secondary" t-att-href="record.url.value" target="_blank">Live Preview</a>
                            </div>
                            <i states="installed" t-if="record.is_installed_on_current_website.raw_value"
                                class="fa fa-check position-absolute p-1 m-2 rounded-circle bg-primary shadow"
                                style="top: 0; right: 0;"
                                role="img" aria-label="Installed" title="Installed"/>
                        </div>
                        <div class="o_theme_preview_bottom clearfix">
                            <h5 t-if="record.display_name.value" class="text-uppercase float-left">
                                <img class="float-left mr4"  t-att-src="record.icon.value" height="16" width="16" alt="Theme preview"/>
                                <b><t t-esc="record.display_name.value.replace(('Theme'), '').replace(('theme'), '')"/></b>
                            </h5>
                            <h6 t-if="record.category_id.value" class="text-muted float-right">
                                <b><t t-esc="record.category_id.value"/></b>
                            </h6>
                        </div>
                    </div>
                </templates>
            </kanban>
        </field>
    </record>
    <record model="ir.ui.view" id="theme_view_search">
        <field name="name">Themes Search</field>
        <field name="model">ir.module.module</field>
        <field name="priority">50</field>
        <field name="arch" type="xml">
            <search>
                <field name="name" filter_domain="['|', '|', ('summary', 'ilike', self), ('shortdesc', 'ilike', self), ('name', 'ilike', self)]" string="Theme"/>
                <field name="category_id" filter_domain="['|', '|', ('summary', 'ilike', self), ('shortdesc', 'ilike', self), ('category_id', 'ilike', self)]" string="Category"/>
                <group>
                    <filter string="Author" name="author" domain="[]" context="{'group_by':'author'}"/>
                    <filter string="Category" name="category" domain="[]" context="{'group_by':'category_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <!-- themes should be installed through website_theme_install -->
    <record id="base.open_module_tree" model="ir.actions.act_window">
        <field name="domain">['!', ('name', '=like', 'theme_%')]</field>
    </record>

    <!-- Actions to list themes with custom kanban (launched on module installation) -->
    <record id="theme_install_kanban_action" model="ir.actions.act_window">
        <field name="name">Choose a theme for your website</field>
        <field name="res_model">ir.module.module</field>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="theme_view_kanban"/>
        <field name="search_view_id" ref="theme_view_search"/>
        <field name="domain" eval="[
            ('category_id', 'not in', [ref('base.module_category_hidden', False), ref('base.module_category_theme_hidden', False)]),
            '|', ('category_id', '=', ref('base.module_category_theme', False)), ('category_id.parent_id', '=', ref('base.module_category_theme', False))
        ]"/>
    </record>

    <record id="theme_install_todo_action" model="ir.actions.server">
        <field name="name">Config: Choose Your Theme</field>
        <field name="model_id" ref="model_ir_module_module"/>
        <field name="state">code</field>
        <field name="code">
model.update_list()
action = {
    'type': 'ir.actions.act_url',
    'url': '/web?reload#action=website_theme_install.theme_install_kanban_action', # the ?reload option is there to fool the webclient into thinking it is a different location and so to force a reload
    'target': 'self',
}
        </field>
    </record>
    <record id="theme_install_todo" model="ir.actions.todo">
        <field name="action_id" ref="theme_install_todo_action"/>
        <field name="sequence">0</field>
    </record>

    <!-- Add link to theme install in default customize modal -->
    <template id="customize_modal" inherit_id="website.theme_customize" priority="99">
        <xpath expr="//div" position="replace">
            <div>
                Please <a href="/web#action=website_theme_install.theme_install_kanban_action">install a theme</a> to customize your website.
            </div>
        </xpath>
    </template>

    <record id="view_website_form_view_themes_modal" model="ir.ui.view">
        <field name="name">website.form</field>
        <field name="model">website</field>
        <field name="inherit_id" ref="website.view_website_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//form" position='inside'>
                <footer>
                    <button name="create_and_redirect_to_theme" type="object" string="Choose a theme" class="ml-2 btn btn-primary" icon="fa-paint-brush"/>
                    <button string="Cancel" class="btn btn-default" special="cancel"/>
                </footer>
            </xpath>
        </field>
    </record>

    <!-- ONBOARDING -->
    <template id="onboarding_website_theme_step">
        <t t-call="base.onboarding_step">
            <t t-set="title">Website Theme</t>
            <t t-set="description">Select a theme for your website.</t>
            <t t-set="btn_text">Let's start!</t>
            <t t-set="method" t-value="'action_open_website_theme_selector'" />
            <t t-set="model" t-value="'res.company'" />
            <t t-set="done" t-value="company.website_theme_onboarding_done" />
        </t>
    </template>

</odoo>

```

