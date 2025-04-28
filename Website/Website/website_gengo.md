# Odoo Module: website_gengo

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Gengo Translator',
    'category': 'Website/Website',
    'summary': 'Translate website in one-click',
    'description': """
This module allows to send website content to Gengo translation service in a single click. Gengo then gives back the translated terms in the destination language.
    """,
    'depends': [
        'website',
        'base_gengo'
    ],
    'data': [
        'views/website_gengo_templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request

GENGO_DEFAULT_LIMIT = 20


class WebsiteGengo(http.Controller):

    @http.route('/website/get_translated_length', type='json', auth='user', website=True)
    def get_translated_length(self, translated_ids, lang):
        result = {"done": 0}
        gengo_translation_ids = request.env['ir.translation'].search([('id', 'in', translated_ids), ('gengo_translation', '!=', False)])
        for trans in gengo_translation_ids:
            result['done'] += len(trans.src.split())
        return result

    @http.route('/website/check_gengo_set', type='json', auth='user', website=True)
    def check_gengo_set(self):
        company = request.env.company.sudo()
        company_flag = 0
        if not company.gengo_public_key or not company.gengo_private_key:
            company_flag = company.id
        return company_flag

    @http.route('/website/set_gengo_config', type='json', auth='user', website=True)
    def set_gengo_config(self, config):
        request.env.company.write(config)
        return True

    @http.route('/website/post_gengo_jobs', type='json', auth='user', website=True)
    def post_gengo_jobs(self):
        request.env['base.gengo.translations']._sync_request(limit=GENGO_DEFAULT_LIMIT)
        return True

    @http.route('/website_gengo/set_translations', type='json', auth='user', website=True)
    def set_translations(self, data, lang):
        IrTranslation = request.env['ir.translation']
        for term in data:
            initial_content = term['initial_content'].strip()
            translation_ids = term['translation_id']
            if not translation_ids:
                translations = IrTranslation.search_read([('lang', '=', lang), ('src', '=', initial_content)], fields=['id'])
                if translations:
                    translation_ids = [t_id['id'] for t_id in translations]

            vals = {
                'gengo_comment': term['gengo_comment'],
                'gengo_translation': term['gengo_translation'],
                'state': 'to_translate',
            }
            if translation_ids:
                IrTranslation.browse(translation_ids).write(vals)
            else:
                vals.update({
                    'name': 'website',
                    'lang': lang,
                    'src': initial_content,
                })
                IrTranslation.create(vals)
        return True

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\base_gengo_translations.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class BaseGengoTranslations(models.TransientModel):
    _inherit = 'base.gengo.translations'
    # update GROUPS, that are the groups allowing to access the gengo key.
    # this is done here because in the base_gengo module, the various website
    # groups do not exist, limiting the access to the admin group.
    GROUPS = ['website.group_website_designer', 'website.group_website_publisher']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_gengo_translations

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#94B6C8"/><stop offset="100%" stop-color="#6A9EBA"/></linearGradient><path id="d" d="M25.032 41.074a19.727 19.727 0 0 1-5.488-1.294c-2.237 1.814-4.98 2.92-7.896 3.237a.592.592 0 0 1-.635-.445c-.07-.292.15-.473.366-.687 1.07-1.068 2.368-1.907 2.875-5.493C12.213 34.363 11 31.83 11 29.081 11 22.408 18.15 17 26.968 17c7.685 0 14.103 4.108 15.625 9.58a19.384 19.384 0 0 0-1.407-.05C32.246 26.53 25 32.654 25 40.209c0 .29.01.579.032.865zm.989-16.996c6.788-4.37-5.117-.712 1.12.722.313.072 1.874.619 2.097.773 1.235.855-1.19.232-4.763 2.67-2.547 1.74 4.968.546 9.525-4.877l-7.98.712zm-4.815 7.276c-1.047.45-1.449-.334-1.206-2.354.364-3.03-1.553-4.378 2-2.496 3.553 1.882-3.438-4.606-5.26-2.026-1.821 2.58 2.588.043 2.223.28-.366.238-2.239 3.277-.963 5.242 1.276 1.965 1.566 2.419 3.024 2.869.971.3 1.032-.205.182-1.515zm20.02 22.48C32.819 53.833 26 47.873 26 40.51c0-7.359 6.818-13.323 15.227-13.323 8.408 0 15.226 5.965 15.226 13.323 0 3.032-1.157 5.826-3.103 8.063.484 3.955 1.721 4.88 2.742 6.057.206.237.415.436.348.759-.061.29-.29.494-.545.494a.49.49 0 0 1-.06-.004c-2.78-.35-5.397-1.569-7.53-3.57a16.928 16.928 0 0 1-7.078 1.524zm-1.613-21.258L33.83 47.64h2.722l1.414-3.988h6.014l1.413 3.988h2.828l-5.803-15.065h-2.806zm-.929 9.073l2.28-6.478h.063l2.257 6.478h-4.6z"/><path id="e" d="M25.032 39.074a19.727 19.727 0 0 1-5.488-1.294c-2.237 1.814-4.98 2.92-7.896 3.237a.592.592 0 0 1-.635-.445c-.07-.292.15-.473.366-.687 1.07-1.068 2.368-1.907 2.875-5.493C12.213 32.363 11 29.83 11 27.081 11 20.408 18.15 15 26.968 15c7.685 0 14.103 4.108 15.625 9.58a19.384 19.384 0 0 0-1.407-.05C32.246 24.53 25 30.654 25 38.209c0 .29.01.579.032.865zm.989-16.996c6.788-4.37-5.117-.712 1.12.722.313.072 1.874.619 2.097.773 1.235.855-1.19.232-4.763 2.67-2.547 1.74 4.968.546 9.525-4.877l-7.98.712zm-4.815 7.276c-1.047.45-1.449-.334-1.206-2.354.364-3.03-1.553-4.378 2-2.496 3.553 1.882-3.438-4.606-5.26-2.026-1.821 2.58 2.588.043 2.223.28-.366.238-2.239 3.277-.963 5.242 1.276 1.965 1.566 2.419 3.024 2.869.971.3 1.032-.205.182-1.515zm20.02 22.48C32.819 51.833 26 45.873 26 38.51c0-7.359 6.818-13.323 15.227-13.323 8.408 0 15.226 5.965 15.226 13.323 0 3.032-1.157 5.826-3.103 8.063.484 3.955 1.721 4.88 2.742 6.057.206.237.415.436.348.759-.061.29-.29.494-.545.494a.49.49 0 0 1-.06-.004c-2.78-.35-5.397-1.569-7.53-3.57a16.928 16.928 0 0 1-7.078 1.524zm-1.613-21.258L33.83 45.64h2.722l1.414-3.988h6.014l1.413 3.988h2.828l-5.803-15.065h-2.806zm-.929 9.073l2.28-6.478h.063l2.257 6.478h-4.6z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M45.111 69H4c-2 0-4-1-4-4V37.883L14 20l11-5 17 9 10 24 4.36 5.623L45.11 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\src\js\website_gengo.js

```javascript
odoo.define('website_gengo.website_gengo', function (require) {
'use strict';

var ajax = require('web.ajax');
var core = require('web.core');
var Dialog = require('web.Dialog');
var Widget = require('web.Widget');
var WysiwygTranslate = require('web_editor.wysiwyg.multizone.translate');
var TranslatorMenu = require('website.editor.menu.translate');

var qweb = core.qweb;
var _t = core._t;

TranslatorMenu.include({
    xmlDependencies: (TranslatorMenu.prototype.xmlDependencies || [])
        .concat(['/website_gengo/static/src/xml/website.gengo.xml']),
    events: _.extend({}, TranslatorMenu.prototype.events, {
        'click a[data-action=translation_gengo_post]': 'translation_gengo_post',
        'click a[data-action=translation_gengo_info]': 'translation_gengo_info',
    }),
    start: function () {
        var def = this._super.apply(this, arguments);

        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        var gengo_langs = ["ar_SY","id_ID","nl_NL","fr_CA","pl_PL","zh_TW","sv_SE","ko_KR","pt_PT","en_US","ja_JP","es_ES","zh_CN","de_DE","fr_FR","fr_BE","ru_RU","it_IT","pt_BR","pt_BR","th_TH","nb_NO","ro_RO","tr_TR","bg_BG","da_DK","en_GB","el_GR","vi_VN","he_IL","hu_HU","fi_FI"];
        if (gengo_langs.indexOf(context.lang) >= 0) {
            this.$('.gengo_post,.gengo_wait,.gengo_inprogress,.gengo_info').remove();
            this.$('button[data-action=save]')
                .after(qweb.render('website.ButtonGengoTranslator'));
        }
        this.translation_gengo_display();

        return def;
    },
    translation_gengo_display: function () {
        var self = this;
        if ($('[data-oe-translation-state="to_translate"], [data-oe-translation-state="None"]').length === 0){
            self.$el.find('.gengo_post').addClass('d-none');
            self.$el.find('.gengo_inprogress').removeClass('d-none');
        }
    },
    translation_gengo_post: function () {
        var self = this;
        this.new_words =  0;
        $('[data-oe-translation-state="to_translate"], [data-oe-translation-state="None"]').each(function () {
            self.new_words += $(this).text().trim().replace(/ +/g," ").split(" ").length;
        });
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        ajax.jsonRpc('/website/check_gengo_set', 'call', {
        }).then(function (res) {
            var dialog;
            if (res === 0){
                dialog = new GengoTranslatorPostDialog(self.new_words);
                dialog.appendTo($(document.body));
                dialog.on('service_level', this, function () {
                    var gengo_service_level = dialog.$el.find(".form-control").val();
                    dialog.$el.modal('hide');
                    self.$el.find('.gengo_post').addClass('d-none');
                    self.$el.find('.gengo_wait').removeClass('d-none');
                    var trans = [];
                    $('[data-oe-translation-state="to_translate"], [data-oe-translation-state="None"]').each(function () {
                        var $node = $(this);
                        var data = $node.data();

                        var val = ($node.is('img')) ? $node.attr('alt') : $node.text();
                        trans.push({
                            initial_content: qweb.tools.html_escape(val),
                            translation_id: data.oeTranslationId || null,
                            gengo_translation: gengo_service_level,
                            gengo_comment:"\nOriginal Page: " + document.URL
                        });
                    });
                    ajax.jsonRpc('/website_gengo/set_translations', 'call', {
                        'data': trans,
                        'lang': context.lang,
                    }).then(function () {
                        ajax.jsonRpc('/website/post_gengo_jobs', 'call', {});
                        self.save();
                    }).guardedCatch(function () {
                        Dialog.alert(null, _t("Could not Post translation"));
                    });
                });
            } else {
                dialog = new GengoApiConfigDialog(res);
                dialog.appendTo($(document.body));
                dialog.on('set_config', this, function () {
                    dialog.$el.modal('hide');
                });
            }
        });
    },
    translation_gengo_info: function () {
        var translated_ids = [];
        $('[data-oe-translation-state="translated"]').each(function () {
            translated_ids.push($(this).attr('data-oe-translation-id'));
        });
        var context;
        this.trigger_up('context_get', {
            callback: function (ctx) {
                context = ctx;
            },
        });
        ajax.jsonRpc('/website/get_translated_length', 'call', {
            'translated_ids': translated_ids,
            'lang': context.lang,
        }).then(function (res){
            var dialog = new GengoTranslatorStatisticDialog(res);
            dialog.appendTo($(document.body));
        });
    },
});

var GengoTranslatorPostDialog = Widget.extend({
    xmlDependencies: ['/website_gengo/static/src/xml/website.gengo.xml'],
    events: {
        'hidden.bs.modal': 'destroy',
        'click button[data-action=service_level]': function () {
            this.trigger('service_level');
        },
    },
    template: 'website.GengoTranslatorPostDialog',
    init: function (new_words){
        this.new_words = new_words;
        return this._super.apply(this, arguments);
    },
    start: function () {
        this.$el.modal();
    },
});

var GengoTranslatorStatisticDialog = Widget.extend({
    xmlDependencies: ['/website_gengo/static/src/xml/website.gengo.xml'],
    events: {
        'hidden.bs.modal': 'destroy',
    },
    template: 'website.GengoTranslatorStatisticDialog',
    init: function (res) {
        var self = this;
        this.inprogess =  0;
        this.new_words =  0;
        this.done =  res.done;
        $('[data-oe-translation-state="to_translate"], [data-oe-translation-state="None"]').each(function () {
            self.new_words += $(this).text().trim().replace(/ +/g," ").split(" ").length;
        });
        $('[data-oe-translation-state="inprogress"]').each(function () {
            self.inprogess += $(this).text().trim().replace(/ +/g," ").split(" ").length;
        });
        this.total = this.done + this.inprogess;
        return this._super.apply(this, arguments);
    },
    start: function (res) {
        this.$el.modal(this.res);
    },
});

var GengoApiConfigDialog = Widget.extend({
    xmlDependencies: ['/website_gengo/static/src/xml/website.gengo.xml'],
    events: {
        'hidden.bs.modal': 'destroy',
        'click button[data-action=set_config]': 'set_config'
    },
    template: 'website.GengoApiConfigDialog',
    init:function (company_id){
        this.company_id =  company_id;
        return this._super.apply(this, arguments);
    },
    start: function (res) {
        this.$el.modal(this.res);
    },
    set_config: function () {
       var self = this;
       var public_key = this.$el.find("#gengo_public_key")[0].value;
       var private_key = this.$el.find("#gengo_private_key")[0].value;
       var auto_approve = this.$el.find("#gengo_auto_approve")[0].checked;
       var sandbox = this.$el.find("#gengo_sandbox")[0].checked;
       var pub_el = this.$el.find(".gengo_group_public")[0];
       var pri_el = this.$el.find(".gengo_group_private")[0];
       if (! public_key){
           $(pub_el).addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
       }
       else {
           $(pub_el).removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
       }
       if (! private_key){
           $(pri_el).addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
       }
       else {
           $(pri_el).removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
       }
       if (public_key && private_key){
           ajax.jsonRpc('/website/set_gengo_config', 'call', {
               'config': {'gengo_public_key':public_key,'gengo_private_key':private_key,'gengo_auto_approve':auto_approve,'gengo_sandbox':sandbox},
           }).then(function () {
               self.trigger('set_config');
           }).guardedCatch(function () {
               Dialog.alert(null, _t("Could not submit ! Try Again"));
           });
       }
    }
});

});

```

## File: static\src\xml\website.gengo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
<t t-name="website.ButtonGengoTranslator">
    <a role="button" class="btn btn-danger gengo_post" data-action="translation_gengo_post" href="#">Auto Translate</a>
    <a role="button" class="btn btn-danger d-none gengo_wait disabled"  href="#"><i class="fa fa-spinner fa-spin"></i> Wait</a>
    <a role="button" class="btn btn-danger d-none gengo_inprogress disabled"  href="#"> <i class="fa fa-clock-o"></i> Translation in Progress</a>
    <a role="button" class="btn btn-link gengo_info" data-action="translation_gengo_info">Count Words</a>
</t>
<t t-extend="website.TranslatorInfoDialog">
    <t t-jquery=".oe_translate_examples" t-operation="replace">
        <ul class="oe_translate_examples">
            <li style="background:#ffffb6;">
                Content to translate or you can post them to <b><a href="http://gengo.com/" >Gengo</a></b> for translation.
            </li>
            <li data-oe-translation-state="inprogress">
                Translation in process (Gengo)
            </li>
            <li data-oe-translation-state="translated">
                Already translated content
            </li>
        </ul>
    </t>
    <t t-jquery="p:last" t-operation="replace">
        <p> <!-- Keep wrong indent for translation --> 
                            In this mode, you can translate texts or post texts to Gengo for translation.
                            To change the structure of the page, you must edit the
                            master page.
        </p>
    </t>
</t>

<t t-name="website.GengoTranslatorPostDialog">
    <div role="dialog" class="modal o_technical_modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <header class="modal-header">
                    <h4 class="modal-title">Select Gengo Translation Service Level</h4>
                    <button title="Close" aria-label="Close" type="button" class="close" data-dismiss="modal">×</button>
                </header>
                <main class="modal-body">
                    <section>
                    <select class="form-control" required="required" autofocus="autofocus">
                        <option value="machine">By Machine (Free)</option>
                        <option value="standard">Standard - $ <t t-esc="(widget.new_words * 0.05).toFixed(2)"></t> </option>
                        <option value="pro">Pro - $ <t t-esc="(widget.new_words * 0.10).toFixed(2)"></t></option>
                        <option value="ultra">Ultra - $ <t t-esc="(widget.new_words * 0.15).toFixed(2)"></t></option>
                    </select>
                    </section>
                </main>
                <footer class="modal-footer">
                    <button type="button" data-action="service_level" class="btn btn-primary">Post</button>
                    <button type="button" class="btn btn-secondary" data-action="discard" data-dismiss="modal">Cancel</button>
                </footer>
            </div>
        </div>
    </div>
</t>

<t t-name="website.GengoTranslatorStatisticDialog">
    <div role="dialog" class="modal o_technical_modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <header class="modal-header">
                    <h5 class="modal-title">Translator statistics for this page</h5>
                    <button title="Close" aria-label="Close" type="button" class="close" data-dismiss="modal">×</button>
                </header>
                <main class="modal-body">
                    <b>
                        <div class="text-muted mb16"> <i class="fa fa-search-plus"></i> <t t-esc="widget.new_words"></t> new words found on this page.</div>
                        <h5><i class="fa fa-tachometer"></i> Gengo Statistics <a href="https://gengo.com/c/dashboard" class="float-right" target="new">Gengo Dashboard</a></h5>
                        <hr class="mt8"/>
                        <div class="text-info mb8"> <i class="fa fa-align-left"></i> Words posted for translate <t t-esc="widget.total"></t></div>
                        <div class="text-warning mb8"> <i class="fa fa-cogs"></i> Words in progress <t t-esc="widget.inprogess"></t></div>
                        <div class="text-success mb8"> <i class="fa fa-check"></i> Translated words <t t-esc="widget.done"></t></div>
                    </b>
                </main>
                <footer class="modal-footer">
                    <button type="button" class="btn btn-primary" data-action="discard" data-dismiss="modal">Close</button>
                </footer>
            </div>
        </div>
    </div>
</t>
<t t-name="website.GengoApiConfigDialog">
    <div role="dialog" class="modal o_technical_modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <header class="modal-header">
                    <h4 class="modal-title">Gengo API is not configured</h4>
                    <button title="Close" aria-label="Close" type="button" class="close" data-dismiss="modal">×</button>
                </header>
                <main class="modal-body">
                    <b> <h4>Steps for configure Gengo </h4>
                        <div class="mb16"> 1. Go To your <b><a target="new" href="https://gengo.com/account/api_settings/">Gengo account</a></b> and generate API Keys.</div>
                        <div class="mb16"> 2. Then paste generated keys in given form</div>
                        <ul class="list-group">
                            <li class="list-group-item form-group gengo_group_public">
                                <h4 class="list-group-item-heading">
                                    <label  class="col-form-label">Public key</label>
                                </h4>
                                <input type="text" class="form-control url url-source" id="gengo_public_key" placeholder="Paste public key here"/>

                            </li>
                            <li class="list-group-item form-group gengo_group_private">
                                <h4 class="list-group-item-heading">
                                    <label for="link-external" class="col-form-label">Private key</label>
                                </h4>
                                <input type="text" class="form-control url url-source" id="gengo_private_key" placeholder="Paste private key here"/>
                            </li>
                            <li class="list-group-item form-group">
                                <div>
                                    <label>
                                        <input type="checkbox" id="gengo_auto_approve" checked="1"/>
                                        Auto Approve Translation  <small class="text-muted">- Jobs are Automatically Approved by Gengo.</small>
                                    </label>
                                </div>
                                <div>
                                    <label>
                                        <input type="checkbox" id="gengo_sandbox"/>
                                        Sandbox  <small class="text-muted">- Enable if you using testing account</small>
                                    </label>
                                </div>
                            </li>
                        </ul>
                    </b>
                </main>
                <footer class="modal-footer">
                    <button type="button" data-action="set_config" class="btn btn-primary">Submit</button>
                    <button type="button" class="btn btn-secondary" data-action="discard" data-dismiss="modal">Close</button>
                </footer>
            </div>
        </div>
    </div>
</t>

</templates>

```

## File: views\website_gengo_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="gengo_assets_editor" inherit_id="website.assets_wysiwyg" name="Editor Head">
        <xpath expr='.' position="inside">
            <link rel="stylesheet" href="/website_gengo/static/src/css/website_gengo.css"/>
            <script type="text/javascript" src="/website_gengo/static/src/js/website_gengo.js" />
        </xpath>
    </template>

</odoo>

```

