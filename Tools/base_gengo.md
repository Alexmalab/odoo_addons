# Odoo Module: base_gengo

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controller
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Automated Translations through Gengo API',
    'category': 'Tools',
    'description': """
Automated Translations through Gengo API
========================================

This module will install passive scheduler job for automated translations 
using the Gengo API. To activate it, you must
1) Configure your Gengo authentication parameters under `Settings > Companies > Gengo Parameters`
2) Launch the wizard under `Settings > Application Terms > Gengo: Manual Request of Translation` and follow the wizard.

This wizard will activate the CRON job and the Scheduler and will start the automatic translation via Gengo Services for all the terms where you requested it.
    """,
    'depends': ['base_setup'],
    'data': [
        'data/ir_cron_data.xml',
        'views/ir_translation_views.xml',
        'views/res_config_settings_views.xml',
        'wizard/base_gengo_translations_view.xml',
    ],
    'demo': ['data/res_company_demo.xml'],
    'test': [],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controller\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo.http import Controller, Response, request, route


class website_gengo(Controller):

    @route('/website/gengo_callback', type='http', auth='public', csrf=False)
    def gengo_callback(self, **post):
        IrTranslationSudo = request.env['ir.translation'].sudo()
        if post and post.get('job') and post.get('pgk'):
            if post.get('pgk') != request.env['base.gengo.translations'].sudo()._get_gengo_key():
                return Response("Bad authentication", status=104)
            job = json.loads(post['job'])
            tid = job.get('custom_data', False)
            if (job.get('status') == 'approved') and tid:
                term = IrTranslationSudo.browse(int(tid))
                if term.src != job.get('body_src'):
                    return Response("Text Altered - Not saved", status=418)
                domain = [
                    '|',
                    ('id', "=", int(tid)),
                    '&', '&', '&', '&', '&',
                    ('state', '=', term.state),
                    ('gengo_translation', '=', term.gengo_translation),
                    ('src', "=", term.src),
                    ('type', "=", term.type),
                    ('name', "=", term.name),
                    ('lang', "=", term.lang),
                    #('order_id', "=", term.order_id),
                ]

                all_ir_translations = IrTranslationSudo.search(domain)

                if all_ir_translations:
                    all_ir_translations.write({
                        'state': 'translated',
                        'value': job.get('body_tgt')
                    })
                    return Response("OK", status=200)
                else:
                    return Response("No terms found", status=412)
        return Response("Not saved", status=418)

```

## File: controller\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!--Scheduler sync Receive Request-->
        <record id="gengo_sync_receive_request_scheduler" model="ir.cron">
            <field name="name">Gengo: Sync translation (Response)</field>
            <field name="model_id" ref="base_gengo.model_base_gengo_translations"/>
            <field name="state">code</field>
            <field name="code">model._sync_response(20)</field>
            <field name="interval_number">6</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False" />
        </record>

        <!--Scheduler Sync Send Request-->
        <record id="gengo_sync_send_request_scheduler" model="ir.cron">
            <field name="name">Gengo: Sync translation (Request)</field>
            <field name="model_id" ref="base_gengo.model_base_gengo_translations"/>
            <field name="state">code</field>
            <field name="code">model._sync_request(20)</field>
            <field name="interval_number">6</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: data\res_company_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="base.main_company" model="res.company">
            <field name="gengo_public_key">dummy</field>
            <field name="gengo_private_key">dummy</field>
            <field name="gengo_sandbox">True</field>
        </record>

    </data>
</odoo>

```

## File: doc\changelog.rst

```rst
========================
``base_gengo`` changelog
========================

******
saas-4
******

- Library update: ``mygengo`` (https://pypi.python.org/pypi/mygengo/1.3.3) was outdated and has been replaced by ``gengo`` (https://pypi.python.org/pypi/gengo).

```

## File: models\ir_translation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.exceptions import UserError

LANG_CODE_MAPPING = {
    'ar_SY': ('ar', 'Arabic'),
    'id_ID': ('id', 'Indonesian'),
    'nl_NL': ('nl', 'Dutch'),
    'fr_CA': ('fr-ca', 'French (Canada)'),
    'pl_PL': ('pl', 'Polish'),
    'zh_TW': ('zh-tw', 'Chinese (Traditional)'),
    'sv_SE': ('sv', 'Swedish'),
    'ko_KR': ('ko', 'Korean'),
    'pt_PT': ('pt', 'Portuguese (Europe)'),
    'en_US': ('en', 'English'),
    'ja_JP': ('ja', 'Japanese'),
    'es_ES': ('es', 'Spanish (Spain)'),
    'zh_CN': ('zh', 'Chinese (Simplified)'),
    'de_DE': ('de', 'German'),
    'fr_FR': ('fr', 'French'),
    'fr_BE': ('fr', 'French'),
    'ru_RU': ('ru', 'Russian'),
    'it_IT': ('it', 'Italian'),
    'pt_BR': ('pt-br', 'Portuguese (Brazil)'),
    'th_TH': ('th', 'Thai'),
    'nb_NO': ('no', 'Norwegian'),
    'ro_RO': ('ro', 'Romanian'),
    'tr_TR': ('tr', 'Turkish'),
    'bg_BG': ('bg', 'Bulgarian'),
    'da_DK': ('da', 'Danish'),
    'en_GB': ('en-gb', 'English (British)'),
    'el_GR': ('el', 'Greek'),
    'vi_VN': ('vi', 'Vietnamese'),
    'he_IL': ('he', 'Hebrew'),
    'hu_HU': ('hu', 'Hungarian'),
    'fi_FI': ('fi', 'Finnish')
}


class IrTranslation(models.Model):
    _inherit = "ir.translation"

    gengo_comment = fields.Text("Comments & Activity Linked to Gengo")
    order_id = fields.Char('Gengo Order ID')
    gengo_translation = fields.Selection([
        ('machine', 'Translation By Machine'),
        ('standard', 'Standard'),
        ('pro', 'Pro'),
        ('ultra', 'Ultra')
        ], "Gengo Translation Service Level",
        help='You can select here the service level you want for an automatic translation using Gengo.')

    @api.model
    def _get_all_supported_languages(self):
        flag, gengo = self.env['base.gengo.translations'].gengo_authentication()
        if not flag:
            raise UserError(gengo)
        supported_langs = {}
        lang_pair = gengo.getServiceLanguagePairs(lc_src='en')
        if lang_pair['opstat'] == 'ok':
            for g_lang in lang_pair['response']:
                if g_lang['lc_tgt'] not in supported_langs:
                    supported_langs[g_lang['lc_tgt']] = []
                supported_langs[g_lang['lc_tgt']] += [g_lang['tier']]
        return supported_langs

    def _get_gengo_corresponding_language(self, lang):
        return lang in LANG_CODE_MAPPING and LANG_CODE_MAPPING[lang][0] or lang

    @api.model
    def _get_source_query(self, name, types, lang, source, res_id):
        query, params = super(IrTranslation, self)._get_source_query(name, types, lang, source, res_id)

        # disable gengo during module installation and uninstallation
        if not self.pool.ready:
            return query, params

        query += """
                    ORDER BY
                        CASE
                            WHEN gengo_translation=%s then 10
                            WHEN gengo_translation=%s then 20
                            WHEN gengo_translation=%s then 30
                            WHEN gengo_translation=%s then 40
                            ELSE 0
                        END DESC
                 """
        params += ('machine', 'standard', 'ultra', 'pro',)
        return (query, params)

    @api.model
    def _get_terms_query(self, field, records):
        query, params = super(IrTranslation, self)._get_terms_query(field, records)

        # disable gengo during module installation and uninstallation
        if not self.pool.ready:
            return query, params

        # order translations from worst to best
        query += """
                    ORDER BY
                        CASE
                            WHEN gengo_translation=%s then 10
                            WHEN gengo_translation=%s then 20
                            WHEN gengo_translation=%s then 30
                            WHEN gengo_translation=%s then 40
                            ELSE 0
                        END ASC
                 """
        params += ('machine', 'standard', 'ultra', 'pro')
        return query, params

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class res_company(models.Model):
    _inherit = "res.company"

    gengo_private_key = fields.Char(string="Gengo Private Key", copy=False, groups="base.group_system")
    gengo_public_key = fields.Text(string="Gengo Public Key", copy=False, groups="base.group_user")
    gengo_comment = fields.Text(string="Comments", groups="base.group_user",
      help="This comment will be automatically be enclosed in each an every request sent to Gengo")
    gengo_auto_approve = fields.Boolean(string="Auto Approve Translation ?", groups="base.group_user", default=True,
      help="Jobs are Automatically Approved by Gengo.")
    gengo_sandbox = fields.Boolean(string="Sandbox Mode",
      help="Check this box if you're using the sandbox mode of Gengo, mainly used for testing purpose.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    gengo_private_key = fields.Char(string="Gengo Private Key", related="company_id.gengo_private_key", readonly=False)
    gengo_public_key = fields.Text(string="Gengo Public Key", related="company_id.gengo_public_key", readonly=False)
    gengo_comment = fields.Text(string="Comments", related="company_id.gengo_comment",
      help="This comment will be automatically be enclosed in each an every request sent to Gengo")
    gengo_auto_approve = fields.Boolean(string="Auto Approve Translation ?", related="company_id.gengo_auto_approve", readonly=False,
      help="Jobs are Automatically Approved by Gengo.")
    gengo_sandbox = fields.Boolean(string="Sandbox Mode", related="company_id.gengo_sandbox", readonly=False,
      help="Check this box if you're using the sandbox mode of Gengo, mainly used for testing purpose.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import ir_translation
from . import res_company

```

## File: views\ir_translation_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_translation_search" model="ir.ui.view">
            <field name="name">ir.translation.search.inherit</field>
            <field name="model">ir.translation</field>
            <field name="inherit_id" ref="base.view_translation_search"/>
            <field name="arch" type="xml">
                <xpath expr="//search">
                     <filter string="To Approve In Gengo" name="to_approve_gengo" domain="[('state','=','inprogress'),('gengo_translation','=',True)]"></filter>
                </xpath>
            </field>
        </record>

        <!-- ir.translation form view -->
        <record id="view_ir_translation_inherit_base_gengo_form" model="ir.ui.view">
            <field name="name">ir.translation.form.inherit</field>
            <field name="inherit_id" ref="base.view_translation_form"/>
            <field name="model">ir.translation</field>
            <field name="arch" type="xml">
                 <xpath expr="//form/sheet" position="inside">
                    <group string="Gengo Translation Service" col="4" colspan="4">
                        <field name="gengo_translation" />
                        <span class="o_form_label">Note: If the translation state is 'In Progress', it means that the translation has to be approved to be uploaded in this system. You are supposed to do that directly by using your Gengo Account</span>
                        <field name="gengo_comment" nolabel="1" placeholder="Gengo Comments &amp; Activity..." colspan="4"/>
                    </group>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.base.gengo</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='base_gengo_right_pane']" position="inside">
                <div name="base_gengo_right_pane" position="replace">
                    <label for="id" string="Public Key" attrs="{'invisible': [('module_base_gengo','=',False)]}"/>
                    <div name="gengo_public_key" attrs="{'invisible': [('module_base_gengo','=',False)]}">
                        <div name="gengo_public_key">
                            <field name="gengo_public_key" nolabel="1" placeholder="Add Gengo login Public Key..."/>
                        </div>
                    </div>
                    <label for="id" string="Private Key" attrs="{'invisible': [('module_base_gengo','=',False)]}"/>
                    <div name="gengo_private_key" attrs="{'invisible': [('module_base_gengo','=',False)]}">
                        <div name="gengo_private_key">
                            <field name="gengo_private_key" password="True" nolabel="1" placeholder="Add Gengo login Private Key..."/>
                        </div>
                    </div>
                    <div name="gengo_auto_approve" attrs="{'invisible': [('module_base_gengo','=',False)]}">
                        <div name="gengo_auto_approve">
                            <field name="gengo_auto_approve" class="oe_inline"/>
                            <label for="id" string="Auto Approve Translation"/>
                        </div>
                    </div>
                    <div name="gengo_sandbox" attrs="{'invisible': [('module_base_gengo','=',False)]}">
                        <div name="gengo_sandbox">
                            <field name="gengo_sandbox" class="oe_inline"/>
                            <label for="id" string="Sandbox Mode"/>
                        </div>
                    </div>
                    <label for="id" string="Comment" attrs="{'invisible': [('module_base_gengo','=',False)]}"/>
                    <div name="gengo_comment" attrs="{'invisible': [('module_base_gengo','=',False)]}">
                        <div name="gengo_comment">
                            <field name="gengo_comment" nolabel="1" placeholder="Add your comments here for translator...."/>
                        </div>
                    </div>
                </div>
            </xpath>
            <xpath expr="//div[@name='base_gengo_warning']" position="replace"/>
        </field>
    </record>
</odoo>
```

## File: wizard\base_gengo_translations.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re
import time
import uuid

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.misc import get_lang

_logger = logging.getLogger(__name__)

try:
    from gengo import Gengo
except ImportError:
    Gengo = None
    _logger.warning('Gengo library not found, Gengo features disabled. If you plan to use it, please install the gengo library from http://pypi.python.org/pypi/gengo')

GENGO_DEFAULT_LIMIT = 20


class BaseGengoTranslations(models.TransientModel):
    GENGO_KEY = "Gengo.UUID"
    GROUPS = ['base.group_system']

    _name = 'base.gengo.translations'
    _description = 'Base Gengo Translations'

    @api.model
    def default_get(self, fields):
        res = super(BaseGengoTranslations, self).default_get(fields)
        res['authorized_credentials'], gengo = self.gengo_authentication()
        if 'lang_id' in fields:
            res['lang_id'] = get_lang(self.env).id
        return res

    sync_type = fields.Selection([
        ('send', 'Send New Terms'),
        ('receive', 'Receive Translation'),
        ('both', 'Both')
        ], "Sync Type", default='both', required=True)
    lang_id = fields.Many2one('res.lang', 'Language', required=True)
    sync_limit = fields.Integer("No. of terms to sync", default=20)
    authorized_credentials = fields.Boolean('The private and public keys are valid')

    def init(self):
        icp = self.env['ir.config_parameter'].sudo()
        if not icp.get_param(self.GENGO_KEY):
            icp.set_param(self.GENGO_KEY, str(uuid.uuid4()))

    def _get_gengo_key(self):
        icp = self.env['ir.config_parameter'].sudo()
        return icp.get_param(self.GENGO_KEY, default="Undefined")

    def open_company(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'res.config.settings',
            'target': 'inline',
            'context': {'module' : 'general_settings'},
            }

    @api.model
    def gengo_authentication(self):
        '''
        This method tries to open a connection with Gengo. For that, it uses the Public and Private
        keys that are linked to the company (given by Gengo on subscription). It returns a tuple with
         * as first element: a boolean depicting if the authentication was a success or not
         * as second element: the connection, if it was a success, or the error message returned by
            Gengo when the connection failed.
            This error message can either be displayed in the server logs (if the authentication was called
            by the cron) or in a dialog box (if requested by the user), thus it's important to return it
            translated.
        '''
        user = self.env.user
        if not user.company_id.gengo_public_key or not user.company_id.gengo_private_key:
            return (False, _("Gengo `Public Key` or `Private Key` are missing. Enter your Gengo authentication parameters under `Settings > Companies > Gengo Parameters`."))
        if not Gengo:
            return (False, _("Gengo library not installed. Contact your system administrator to use it."))
        try:
            gengo = Gengo(
                public_key=user.company_id.gengo_public_key.encode('ascii'),
                private_key=user.company_id.gengo_private_key.encode('ascii'),
                sandbox=user.company_id.gengo_sandbox,
            )
            gengo.getAccountStats()
            return (True, gengo)
        except Exception as e:
            _logger.exception('Gengo connection failed')
            return (False, _("Gengo connection failed with this message:\n``%s``") % e)

    def act_update(self):
        '''
        Function called by the wizard.
        '''
        flag, gengo = self.gengo_authentication()
        if not flag:
            raise UserError(gengo)
        for wizard in self:
            supported_langs = self.env['ir.translation']._get_all_supported_languages()
            language = self.env['ir.translation']._get_gengo_corresponding_language(wizard.lang_id.code)
            if language not in supported_langs:
                raise UserError(_('This language is not supported by the Gengo translation services.'))

            ctx = self.env.context.copy()
            ctx['gengo_language'] = wizard.lang_id.id
            if wizard.sync_limit > 200 or wizard.sync_limit < 1:
                raise UserError(_('The number of terms to sync should be between 1 to 200 to work with Gengo translation services.'))
            if wizard.sync_type in ['send', 'both']:
                self.with_context(ctx)._sync_request(wizard.sync_limit)
            if wizard.sync_type in ['receive', 'both']:
                self.with_context(ctx)._sync_response(wizard.sync_limit)
        return {'type': 'ir.actions.act_window_close'}

    @api.model
    def _sync_response(self, limit=GENGO_DEFAULT_LIMIT):
        """
        This method will be called by cron services to get translations from
        Gengo. It will read translated terms and comments from Gengo and will
        update respective ir.translation in Odoo.
        """
        IrTranslation = self.env['ir.translation']
        flag, gengo = self.gengo_authentication()
        if not flag:
            _logger.warning("%s", gengo)
        else:
            offset = 0
            all_translation_ids = IrTranslation.search([
                ('state', '=', 'inprogress'),
                ('gengo_translation', 'in', ('machine', 'standard', 'pro', 'ultra')),
                ('order_id', "!=", False)])
            while True:
                translation_ids = all_translation_ids[offset:offset + limit]
                offset += limit
                if not translation_ids:
                    break

                terms_progress = {
                    'gengo_order_ids': set(),
                    'ir_translation_ids': set(),
                }
                for term in translation_ids:
                    terms_progress['gengo_order_ids'].add(term.order_id)
                    terms_progress['ir_translation_ids'].add(tools.ustr(term.id))

                for order_id in terms_progress['gengo_order_ids']:
                    order_response = gengo.getTranslationOrderJobs(id=order_id)
                    jobs_approved = order_response.get('response', []).get('order', []).get('jobs_approved', [])
                    gengo_ids = ','.join(jobs_approved)

                if gengo_ids:  # Need to check, because getTranslationJobBatch don't catch this case and so call the getTranslationJobs because no ids in url
                    try:
                        job_response = gengo.getTranslationJobBatch(id=gengo_ids)
                    except:
                        continue
                    if job_response['opstat'] == 'ok':
                        for job in job_response['response'].get('jobs', []):
                            if job.get('custom_data') in terms_progress['ir_translation_ids']:
                                self._update_terms_job(job)
        return True

    @api.model
    def _update_terms_job(self, job):
        translation = self.env['ir.translation'].browse(int(job['custom_data']))
        vals = {}
        if job.get('status', False) in ('queued', 'available', 'pending', 'reviewable'):
            vals['state'] = 'inprogress'
        if job.get('body_tgt', False) and job.get('status', False) == 'approved':
            vals['value'] = job['body_tgt']
        if job.get('status', False) in ('approved', 'canceled'):
            vals['state'] = 'translated'
        if vals:
            try:
                translation.write(vals)
            except ValidationError:
                pass

    @api.model
    def _update_terms(self, response, term_ids):
        """
        Update the terms after their translation were requested to Gengo
        """
        vals = {
            'order_id': response.get('order_id', ''),
            'state': 'inprogress'
        }
        term_ids.write(vals)
        jobs = response.get('jobs', [])
        if jobs:
            for t_id, job in jobs.items():
                self._update_terms_job(job)

        return

    @api.model
    def pack_jobs_request(self, term_ids, context=None):
        ''' prepare the terms that will be requested to gengo and returns them in a dictionary with following format
            {'jobs': {
                'term1.id': {...}
                'term2.id': {...}
                }
            }'''
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        IrTranslation = self.env['ir.translation']
        jobs = {}
        user = self.env.user
        auto_approve = 1 if user.company_id.gengo_auto_approve else 0
        for term in term_ids:
            if re.search(r"\w", term.src or ""):
                comment = user.company_id.gengo_comment or ''
                if term.gengo_comment:
                    comment += '\n' + term.gengo_comment
                jobs[time.strftime('%Y%m%d%H%M%S') + '-' + str(term.id)] = {
                    'type': 'text',
                    'slug': 'Single :: English to ' + term.lang,
                    'tier': tools.ustr(term.gengo_translation),
                    'custom_data': str(term.id),
                    'body_src': term.src,
                    'lc_src': 'en',
                    'lc_tgt': IrTranslation._get_gengo_corresponding_language(term.lang),
                    'auto_approve': auto_approve,
                    'comment': comment,
                    'callback_url': "%s/website/gengo_callback?pgk=%s&db=%s" % (base_url, self._get_gengo_key(), self.env.cr.dbname)
                }
        return {'jobs': jobs, 'as_group': 0}

    @api.model
    def _send_translation_terms(self, term_ids):
        """
        Send a request to Gengo with all the term_ids in a different job, get the response and update the terms in
        database accordingly.
        """
        flag, gengo = self.gengo_authentication()
        if flag:
            request = self.pack_jobs_request(term_ids)
            if request['jobs']:
                result = gengo.postTranslationJobs(jobs=request)
                if result['opstat'] == 'ok':
                    self._update_terms(result['response'], term_ids)
        else:
            _logger.error(gengo)
        return True

    @api.model
    def _sync_request(self, limit=GENGO_DEFAULT_LIMIT):
        """
        This scheduler will send a job request to the gengo , which terms are
        waiing to be translated and for which gengo_translation is enabled.

        A special key 'gengo_language' can be passed in the context in order to
        request only translations of that language only. Its value is the language
        ID in Odoo.
        """
        domain = [
            ('state', '=', 'to_translate'),
            ('gengo_translation', 'in', ('machine', 'standard', 'pro', 'ultra')),
            ('order_id', "=", False)]
        if self.env.context.get('gengo_language', False):
            lc = self.env['res.lang'].browse(self.env.context['gengo_language']).code
            domain.append(('lang', '=', lc))

        all_term_ids = self.env['ir.translation'].search(domain)
        try:
            offset = 0
            while True:
                #search for the n first terms to translate
                term_ids = all_term_ids[offset:offset + limit]
                if term_ids:
                    offset += limit
                    self._send_translation_terms(term_ids)
                    _logger.info("%s Translation terms have been posted to Gengo successfully", len(term_ids))
                if not len(term_ids) == limit:
                    break
        except Exception as e:
            _logger.error("%s", e)

```

## File: wizard\base_gengo_translations_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record model="ir.ui.view" id="base_gengo_translation_wizard_from">
            <field name="name">base.gengo.translation.form</field>
            <field name="model">base.gengo.translations</field>
            <field name="arch" type="xml">
                <form string="Gengo Request Form">
                    <div class="alert alert-warning text-center" attrs="{'invisible': [('authorized_credentials', '=', True)]}" role="alert">
                        <span class="o_form_label">Gengo Public or Private keys are wrong or missing.</span>
                        <button type="object" name="open_company" string="Click here to Configure Gengo Parameters" icon="fa-cogs" class="oe_inline oe_link"/>
                        <field name="authorized_credentials" invisible="1"/>
                    </div>
                    <group>
                        <field name="lang_id"/>
                    </group>
                    <group>
	                    <group>
	                            <field name="sync_type" widget="radio"/>
	                    </group>
	                    <group>
	                            <field name="sync_limit" required="1"/>
	                    </group>
                    </group>
                    <footer>
                        <button name="act_update" string="Send" type="object" class="btn-primary"/>
                        <button name="act_cancel" special="cancel" string="Cancel" type="object" class="btn-secondary"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_wizard_base_gengo_translations" model="ir.actions.act_window">
            <field name="name">Gengo: Manual Request of Translation</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">base.gengo.translations</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>
        <menuitem id="menu_action_wizard_base_gengo_translations" action="action_wizard_base_gengo_translations" parent="base.menu_translation_app"/>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_gengo_translations

```

