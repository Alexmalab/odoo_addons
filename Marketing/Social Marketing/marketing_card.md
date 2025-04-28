# Odoo Module: marketing_card

Category: Marketing/Social Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
from . import models
from . import wizards

```

## File: __manifest__.py

```python
{
    'name': 'Marketing Card',
    'version': '1.1',
    'category': 'Marketing/Social Marketing',
    'summary': 'Generate dynamic shareable cards',
    'depends': ['link_tracker', 'mass_mailing', 'website'],
    'data': [
        'security/marketing_card_groups.xml',
        'security/ir.model.access.csv',
        'views/card_card_templates.xml',
        'data/card_template_data.xml',
        'views/card_card_views.xml',
        'views/card_campaign_views.xml',
        'views/card_frontend_templates.xml',
        'views/card_template_views.xml',
        'views/card_menus.xml',
        'views/mailing_mailing_views.xml',
        'views/website_templates.xml',
    ],
    'demo': [
        'demo/card_campaign_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'marketing_card/static/src/scss/*',
        ],
        'web_editor.backend_assets_wysiwyg': [
            'marketing_card/static/src/scss/mass_mailing.scss'
        ],
    },
    'application': True,
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\marketing_card.py

```python
import base64
from urllib.parse import quote
from werkzeug.exceptions import BadRequest

from odoo.http import Controller, content_disposition, request, route

# from https://github.com/monperrus/crawler-user-agents
SOCIAL_NETWORK_USER_AGENTS = (
    # Facebook
    'Facebot',
    'facebookexternalhit',
    # Twitter
    'Twitterbot',
    # LinkedIn
    'LinkedInBot',
    # Whatsapp
    'WhatsApp',
    # Pinterest
    'Pinterest',
    'Pinterestbot',
)


def _is_crawler(request):
    """Returns True if the request is made by a social network crawler."""
    return any(
        short_crawler_name in request.httprequest.user_agent.string
        for short_crawler_name in SOCIAL_NETWORK_USER_AGENTS
    )


def _get_card_from_url(card_id, card_slug):
    """Helper to support both legacy card id url and new slug urls"""
    if card_slug:
        card_id = request.env['ir.http']._unslug(card_slug)[1]
    if not card_id:
        raise request.not_found()
    card = request.env['card.card'].browse(card_id).exists()
    if not card:
        raise BadRequest()
    return card


class MarketingCardController(Controller):

    @route([
        '/cards/<string:card_slug>/card.jpg',
        '/cards/<int:card_id>/card.jpg',
    ], type='http', auth='public', sitemap=False, website=True)
    def card_campaign_image(self, card_id=None, card_slug=None):
        card = _get_card_from_url(card_id, card_slug)
        if _is_crawler(request) and card.share_status != 'shared':
            card.sudo().share_status = 'shared'
        if not card.image:
            raise request.not_found()

        image_bytes = base64.b64decode(card.image)
        return request.make_response(image_bytes, [
            ('Content-Type', ' image/jpeg'),
            ('Content-Length', len(image_bytes)),
            ('Content-Disposition', content_disposition('card.jpg')),
        ])

    @route([
        '/cards/<string:card_slug>/preview',
        '/cards/<int:card_id>/preview',
    ], type='http', auth='public', sitemap=False, website=True)
    def card_campaign_preview(self, card_id=None, card_slug=None):
        """Route for users to preview their card and share it on their social platforms."""
        card = _get_card_from_url(card_id, card_slug)
        if not card.share_status:
            card.sudo().share_status = 'visited'

        campaign_sudo = card.sudo().campaign_id
        return request.render('marketing_card.card_campaign_preview', {
            'card': card,
            'campaign': campaign_sudo,
            'quote': quote,
        })

    @route([
        '/cards/<string:card_slug>/redirect',
        '/cards/<int:card_id>/redirect',
    ], type='http', auth='public', sitemap=False, website=True)
    def card_campaign_redirect(self, card_id=None, card_slug=None):
        """Route to redirect users to the target url, or display the opengraph embed text for web crawlers.

        When a user posts a link on an application supporting opengraph, the application will follow
        the link to fetch specific meta tags on the web page to get preview information such as a preview card.
        The "crawler" performing that action usually has a specific user agent.

        As we cannot necessarily control the target url of the campaign we must return a different
        result when a social network crawler is visiting the URL to get preview information.
        From the perspective of the crawler, this url is an empty page with opengraph tags.
        For all other user agents, it's a simple redirection url.

        Keeping an up-to-date list of user agents for each supported target website is imperative
        for this app to work.
        """
        card = _get_card_from_url(card_id, card_slug)

        campaign_sudo = card.sudo().campaign_id
        redirect_url = campaign_sudo.link_tracker_id.short_url or campaign_sudo.target_url or campaign_sudo.get_base_url()

        if _is_crawler(request):
            return request.render('marketing_card.card_campaign_crawler', {
                'image_url': card._get_card_url(),
                'post_text': campaign_sudo.post_suggestion,
                'target_name': card.display_name or '',
            })

        return request.redirect(redirect_url)

```

## File: controllers\__init__.py

```python
from . import marketing_card

```

## File: data\card_template_data.xml

```xml
<odoo noupdate="1">
    <record id="card_template_light" model="card.template">
        <field name="name">Light</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_5"/>
        </field>
    </record>
    <record id="card_template_dark" model="card.template">
        <field name="name">Dark</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_5"/>
        </field>
        <field name="primary_color">#161315</field>
        <field name="secondary_color">#dedede</field>
        <field name="primary_text_color">#ffffff</field>
        <field name="secondary_text_color">#161315</field>
    </record>
    <record id="card_template_user_image" model="card.template">
        <field name="name">Center</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_1"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Image.jpg"/>
        <field name="primary_color">#161315</field>
        <field name="secondary_color">#dedede</field>
        <field name="primary_text_color">#ffffff</field>
        <field name="secondary_text_color">#161315</field>
    </record>
    <record id="card_template_world_map" model="card.template">
        <field name="name">World Map</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_2"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/WorldMap.png"/>
        <field name="primary_color">#161315</field>
        <field name="secondary_color">#dedede</field>
        <field name="primary_text_color">#ffffff</field>
        <field name="secondary_text_color">#161315</field>
    </record>
    <record id="card_template_lila" model="card.template">
        <field name="name">Lila</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_3"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Lila.png"/>
    </record>
    <record id="card_template_safari" model="card.template">
        <field name="name">Safari</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_4"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Desert.png"/>
        <field name="primary_color">#161315</field>
        <field name="secondary_color">#dedede</field>
        <field name="primary_text_color">#ffffff</field>
        <field name="secondary_text_color">#161315</field>
    </record>
    <record id="card_template_waves" model="card.template">
        <field name="name">Waves</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_5"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Waves.png"/>
    </record>
    <record id="card_template_lines" model="card.template">
        <field name="name">Lines</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_2"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Lines.png"/>
        <field name="primary_color">#161315</field>
        <field name="secondary_color">#dedede</field>
        <field name="primary_text_color">#ffffff</field>
        <field name="secondary_text_color">#161315</field>
    </record>
    <record id="card_template_organic" model="card.template">
        <field name="name">Organic</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_4"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Organic.png"/>
    </record>
    <record id="card_template_blur" model="card.template">
        <field name="name">Blur</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_1"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Blur.png"/>
    </record>
    <record id="card_template_geometric" model="card.template">
        <field name="name">Geometric</field>
        <field name="body" type="html">
            <t t-call="marketing_card.template_4"/>
        </field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Geometric.png"/>
    </record>
    <record id="card_template_circles" model="card.template">
        <field name="name">Circles</field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Circles.png"/>
        <field name="body" type="html">
            <t t-call="marketing_card.template_5"/>
        </field>
    </record>
    <record id="card_template_avatar_highlight" model="card.template">
        <field name="name">Avatar Highlight</field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Highlight.png"/>
        <field name="body" type="html">
            <t t-call="marketing_card.template_3"/>
        </field>
    </record>
    <record id="card_template_drawings" model="card.template">
        <field name="name">Drawings</field>
        <field name="default_background" type="base64" file="marketing_card/static/card_backgrounds/Drawings.png"/>
        <field name="body" type="html">
            <t t-call="marketing_card.template_5"/>
        </field>
    </record>
</odoo>

```

## File: data\utm_source_data.xml

```xml
<odoo>
    <record id="utm_source_marketing_card" model="utm.source">
        <field name="name">Marketing Card</field>
    </record>
</odoo>

```

## File: models\card_campaign.py

```python
import base64

from odoo import _, api, fields, models, exceptions

from .card_template import TEMPLATE_DIMENSIONS


class CardCampaign(models.Model):
    _name = 'card.campaign'
    _description = 'Marketing Card Campaign'
    _inherit = ['mail.activity.mixin', 'mail.render.mixin', 'mail.thread']
    _order = 'id DESC'
    _unrestricted_rendering = True

    def _default_card_template_id(self):
        return self.env['card.template'].search([], limit=1)

    def _get_model_selection(self):
        """Hardcoded list of models, checked against actually-present models."""
        allowed_models = ['res.partner', 'event.track', 'event.booth', 'event.registration']
        models = self.env['ir.model'].sudo().search_fetch([('model', 'in', allowed_models)], ['model', 'name'])
        return [(model.model, model.name) for model in models]

    name = fields.Char(required=True)
    active = fields.Boolean(default=True)
    body_html = fields.Html(related='card_template_id.body', render_engine="qweb")

    card_count = fields.Integer(compute='_compute_card_stats')
    card_click_count = fields.Integer(compute='_compute_card_stats')
    card_share_count = fields.Integer(compute='_compute_card_stats')

    mailing_ids = fields.One2many('mailing.mailing', 'card_campaign_id')
    mailing_count = fields.Integer(compute='_compute_mailing_count')

    card_ids = fields.One2many('card.card', inverse_name='campaign_id')
    card_template_id = fields.Many2one('card.template', string="Design", default=_default_card_template_id, required=True)
    image_preview = fields.Image(compute='_compute_image_preview', compute_sudo=False, readonly=True, store=True, attachment=False)
    link_tracker_id = fields.Many2one('link.tracker', ondelete="restrict")
    res_model = fields.Selection(
        string="Model Name", compute='_compute_res_model', selection='_get_model_selection',
        precompute=True, readonly=True, required=True, store=True,
    )

    post_suggestion = fields.Text(help="Description below the card and default text when sharing on X")
    preview_record_ref = fields.Reference(string="Preview On", selection="_get_model_selection", required=True)
    tag_ids = fields.Many2many('card.campaign.tag', string='Tags')
    target_url = fields.Char(string='Post Link')
    target_url_click_count = fields.Integer(related="link_tracker_id.count")

    user_id = fields.Many2one('res.users', string='Responsible', default=lambda self: self.env.user, domain="[('share', '=', False)]")

    reward_message = fields.Html(string='Thank You Message')
    reward_target_url = fields.Char(string='Reward Link')
    request_title = fields.Char('Request', default=lambda self: _('Help us share the news'))
    request_description = fields.Text('Request Description')

    # Static Content fields
    content_background = fields.Image('Background')
    content_button = fields.Char('Button')

    # Dynamic Content fields
    content_header = fields.Char('Header')
    content_header_dyn = fields.Boolean('Is Dynamic Header')
    content_header_path = fields.Char('Header Path')
    content_header_color = fields.Char('Header Color')

    content_sub_header = fields.Char('Sub-Header')
    content_sub_header_dyn = fields.Boolean('Is Dynamic Sub-Header')
    content_sub_header_path = fields.Char('Sub-Header Path')
    content_sub_header_color = fields.Char('Sub Header Color')

    content_section = fields.Char('Section')
    content_section_dyn = fields.Boolean('Is Dynamic Section')
    content_section_path = fields.Char('Section Path')

    content_sub_section1 = fields.Char('Sub-Section 1')
    content_sub_section1_dyn = fields.Boolean('Is Dynamic Sub-Section 1')
    content_sub_section1_path = fields.Char('Sub-Section 1 Path')

    content_sub_section2 = fields.Char('Sub-Section 2')
    content_sub_section2_dyn = fields.Boolean('Is Dynamic Sub-Section 2')
    content_sub_section2_path = fields.Char('Sub-Section 2 Path')

    # images are always dynamic
    content_image1_path = fields.Char('Dynamic Image 1')
    content_image2_path = fields.Char('Dynamic Image 2')

    @api.depends('card_ids')
    def _compute_card_stats(self):
        cards_by_status_count = self.env['card.card']._read_group(
            domain=[('campaign_id', 'in', self.ids)],
            groupby=['campaign_id', 'share_status'],
            aggregates=['__count'],
            order='campaign_id ASC',
        )
        self.update({
            'card_count': 0,
            'card_click_count': 0,
            'card_share_count': 0,
        })
        for campaign, status, card_count in cards_by_status_count:
            # shared cards are implicitly visited
            if status == 'shared':
                campaign.card_share_count += card_count
            if status in ('shared', 'visited'):
                campaign.card_click_count += card_count
            campaign.card_count += card_count

    @api.model
    def _get_render_fields(self):
        return [
            'body_html', 'content_background', 'content_image1_path', 'content_image2_path', 'content_button', 'content_header',
            'content_header_dyn', 'content_header_path', 'content_header_color', 'content_sub_header',
            'content_sub_header_dyn', 'content_sub_header_path', 'content_section', 'content_section_dyn',
            'content_section_path', 'content_sub_section1', 'content_sub_section1_dyn', 'content_sub_header_color',
            'content_sub_section1_path', 'content_sub_section2', 'content_sub_section2_dyn', 'content_sub_section2_path',
            'card_template_id',
        ]

    def _check_access_right_dynamic_template(self):
        """ `_unrestricted_rendering` being True means we trust the value on model
        when rendering. This means once created, rendering is done without restriction.
        But this attribute triggers a check at create / write / translation update that
        current user is an admin or has full edition rights (group_mail_template_editor).

         However here a Marketing Card Manager must be able to edit the fields other
         than the rendering fields. The qweb rendered field `body_html` cannot be
         modified by users other than the `base.group_system` users, as
        - it's a related field to `card.template.body`,
        - store=False
        - the model `card.template` can only be altered by `base.group_system`

        Hence the security is delegated to the 'card.template' model, hence the
        check done by `_check_access_right_dynamic_template` can be bypassed.
        """
        return

    @api.depends(lambda self: self._get_render_fields() + ['preview_record_ref'])
    def _compute_image_preview(self):
        for campaign in self:
            if campaign.preview_record_ref and campaign.preview_record_ref.exists():
                image = campaign._get_image_b64(campaign.preview_record_ref)
            else:
                image = False
            campaign.image_preview = image

    @api.depends('mailing_ids')
    def _compute_mailing_count(self):
        self.mailing_count = 0
        mailing_counts = self.env['mailing.mailing']._read_group(
            [('card_campaign_id', 'in', self.ids)], ['card_campaign_id'], ['__count']
        )
        for campaign, mailing_count in mailing_counts:
            campaign.mailing_count = mailing_count

    @api.depends('preview_record_ref')
    def _compute_res_model(self):
        for campaign in self:
            preview_model = campaign.preview_record_ref and campaign.preview_record_ref._name
            campaign.res_model = preview_model or campaign.res_model or 'res.partner'

    @api.model_create_multi
    def create(self, create_vals):
        utm_source = self.env.ref('marketing_card.utm_source_marketing_card', raise_if_not_found=False)
        link_trackers = self.env['link.tracker'].sudo().create([
            {
                'url': vals.get('target_url') or self.env['card.campaign'].get_base_url(),
                'title': vals['name'],  # not having this will trigger a request in the create
                'source_id': utm_source.id if utm_source else None,
                'label': f"marketing_card_campaign_{vals.get('name', '')}_{fields.Datetime.now()}",
            }
            for vals in create_vals
        ])
        return super().create([{
            **vals,
            'link_tracker_id': link_tracker_id,
        } for vals, link_tracker_id in zip(create_vals, link_trackers.ids)])

    def write(self, vals):
        link_tracker_vals = {}
        if vals.keys() & set(self._get_render_fields()):
            self.env['card.card'].search([('campaign_id', 'in', self.ids)]).requires_sync = True
        if 'target_url' in vals:
            link_tracker_vals['url'] = vals['target_url'] or self.env['card.campaign'].get_base_url()
        if link_tracker_vals:
            self.link_tracker_id.sudo().write(link_tracker_vals)

        # write and detect model changes on actively-used campaigns
        original_models = self.mapped('res_model')

        write_res = super().write(vals)

        updated_model_campaigns = self.env['card.campaign'].browse([
            campaign.id for campaign, new_model, old_model
            in zip(self, self.mapped('res_model'), original_models)
            if new_model != old_model
        ])
        for campaign in updated_model_campaigns:
            if campaign.card_count:
                raise exceptions.ValidationError(_(
                    "Model of campaign %(campaign)s may not be changed as it already has cards",
                    campaign=campaign.display_name,
                ))
        return write_res

    def action_view_cards(self):
        self.ensure_one()
        return self.env["ir.actions.actions"]._for_xml_id("marketing_card.cards_card_action") | {
            'context': {},
            'domain': [('campaign_id', '=', self.id)],
        }

    def action_view_cards_clicked(self):
        self.ensure_one()
        return self.env["ir.actions.actions"]._for_xml_id("marketing_card.cards_card_action") | {
            'context': {'search_default_filter_visited': True},
            'domain': [('campaign_id', '=', self.id)],
        }

    def action_view_cards_shared(self):
        self.ensure_one()
        return self.env["ir.actions.actions"]._for_xml_id("marketing_card.cards_card_action") | {
            'context': {'search_default_filter_shared': True},
            'domain': [('campaign_id', '=', self.id)],
        }

    def action_view_mailings(self):
        self.ensure_one()
        return {
            'name': _('%(card_campaign_name)s Mailings', card_campaign_name=self.name),
            'type': 'ir.actions.act_window',
            'res_model': 'mailing.mailing',
            'domain': [('card_campaign_id', '=', self.id)],
            'view_mode': 'list,form',
            'target': 'current',
        }

    def action_preview(self):
        self.ensure_one()
        card = self.env['card.card'].with_context(active_test=False).search([
            ('campaign_id', '=', self.id),
            ('res_id', '=', self.preview_record_ref.id),
        ])
        if card:
            card.image = self.image_preview
        else:
            card = self.env['card.card'].create({
                'campaign_id': self.id,
                'res_id': self.preview_record_ref.id,
                'image': self.image_preview,
                'active': False,
            })
        return {'type': 'ir.actions.act_url', 'url': card._get_path('preview'), 'target': 'new'}

    def action_share(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Send Cards'),
            'res_model': 'mailing.mailing',
            'context': {
                'default_subject': self.name,
                'default_card_campaign_id': self.id,
                'default_mailing_model_id': self.env['ir.model']._get_id(self.res_model),
                'default_body_arch': f"""
<div class="o_layout oe_unremovable oe_unmovable bg-200 o_empty_theme" data-name="Mailing">
<style id="design-element"></style>
<div class="container o_mail_wrapper o_mail_regular oe_unremovable">
<div class="row">
<div class="col o_mail_no_options o_mail_wrapper_td bg-white oe_structure o_editable theme_selection_done">

<div class="s_text_block o_mail_snippet_general pt24 pb24" style="padding-left: 15px; padding-right: 15px;" data-snippet="s_text_block" data-name="Text">
    <div class="container s_allow_columns">
        <p class="o_default_snippet_text">Hello everyone</p>
        <p class="o_default_snippet_text">Here's the link to advertise your participation.
        <br> Your help with this promotion would be greatly appreciated!`</p>
        <p class="o_default_snippet_text">Many thanks</p>
    </div>
</div>

<div class="s_call_to_share_card o_mail_snippet_general" style="padding-top: 10px; padding-bottom: 10px;">
    <table width="100%" border="0" cellspacing="0" cellpadding="0">
        <tbody>
            <tr>
                <td align="center">
                    <a href="/cards/{self.id}/preview" style="padding-left: 3px !important; padding-right: 3px !important">
                        <img src="/web/image/card.campaign/{self.id}/image_preview" alt="Card Preview" class="img-fluid" style="width: 540px;"/>
                    </a>
                </td>
            </tr>
        </tbody>
    </table>
</div>

</div></div></div></div>
""",
            },
            'views': [[False, 'form']],
            'target': 'new',
        }

    # ==========================================================================
    # Image generation
    # ==========================================================================

    def _get_image_b64(self, record):
        if not self.card_template_id.body:
            return ''

        image_bytes = self.env['ir.actions.report']._run_wkhtmltoimage(
            [self._render_field('body_html', record.ids, add_context={'card_campaign': self})[record.id]],
            *TEMPLATE_DIMENSIONS
        )[0]
        return image_bytes and base64.b64encode(image_bytes)

    # ==========================================================================
    # Card creation
    # ==========================================================================

    def _update_cards(self, domain, auto_commit=False):
        """Create missing cards and update cards if necessary based for the domain."""
        self.ensure_one()
        TargetModel = self.env[self.res_model]
        res_ids = TargetModel.search(domain).ids
        cards = self.env['card.card'].with_context(active_test=False).search_fetch([
            ('campaign_id', '=', self.id),
            ('res_id', 'in', res_ids),
        ], ['res_id', 'requires_sync'])
        # update active and res_model for preview cards
        cards.active = True
        self.env['card.card'].create([
            {'campaign_id': self.id, 'res_id': res_id}
            for res_id in set(res_ids) - set(cards.mapped('res_id'))
        ])

        # render by batch of 100 to avoid losing progress in case of time out
        updated_cards = self.env['card.card']
        while cards := self.env['card.card'].search_fetch([
            ('requires_sync', '=', True),
            ('campaign_id', '=', self.id),
            ('res_id', 'in', res_ids),
        ], ['res_id'], limit=100):
            # no need to autocommit if it can be done in one batch
            if auto_commit and updated_cards:
                self.env.cr.commit()
                # avoid keeping hundreds of jpegs in memory
                self.env['card.card'].invalidate_model(['image'])
            TargetModelPrefetch = TargetModel.with_prefetch(cards.mapped('res_id'))
            for card in cards.filtered('requires_sync'):
                card.write({
                    'image': self._get_image_b64(TargetModelPrefetch.browse(card.res_id)),
                    'requires_sync': False,
                    'active': True,
                })
            cards.flush_recordset()
            updated_cards += cards
        return updated_cards

    def _get_url_from_res_id(self, res_id, suffix='preview'):
        card = self.env['card.card'].search([('campaign_id', '=', self.id), ('res_id', '=', res_id)])
        return card and card._get_path(suffix) or self.target_url

    # ==========================================================================
    # Mail render mixin / Render utils
    # ==========================================================================

    @api.depends('res_model')
    def _compute_render_model(self):
        """ override for mail.render.mixin """
        for campaign in self:
            campaign.render_model = campaign.res_model

    def _get_card_element_values(self, record):
        """Helper to get the right value for dynamic fields."""
        self.ensure_one()
        result = {
            'image1': images[0] if (images := self.content_image1_path and self.content_image1_path in record and record.mapped(self.content_image1_path)) else False,
            'image2': images[0] if (images := self.content_image2_path and self.content_image2_path in record and record.mapped(self.content_image2_path)) else False,
        }
        campaign_text_element_fields = (
            ('header', 'content_header', 'content_header_dyn', 'content_header_path'),
            ('sub_header', 'content_sub_header', 'content_sub_header_dyn', 'content_sub_header_path'),
            ('section', 'content_section', 'content_section_dyn', 'content_section_path'),
            ('sub_section1', 'content_sub_section1', 'content_sub_section1_dyn', 'content_sub_section1_path'),
            ('sub_section2', 'content_sub_section2', 'content_sub_section2_dyn', 'content_sub_section2_path'),
        )
        for el, text_field, dyn_field, path_field in campaign_text_element_fields:
            if not self[dyn_field]:
                result[el] = self[text_field]
            else:
                try:
                    m = record.mapped(self[path_field])
                    result[el] = m and m[0] or False
                except (AttributeError, KeyError):
                    # for generic image, or if field incorrect, return name of field
                    result[el] = self[path_field]
        return result

```

## File: models\card_campaign_tag.py

```python
import random

from odoo import fields, models


class CardCampaignTag(models.Model):
    _name = 'card.campaign.tag'
    _description = 'Marketing Card Campaign Tag'

    def _get_default_color(self):
        return random.randint(1, 11)

    name = fields.Char(required=True)
    color = fields.Integer(default=_get_default_color)

    _sql_constraints = [('name_uniq', "unique(name)", "Tags may not reuse existing names.")]

```

## File: models\card_card.py

```python
from datetime import datetime, timedelta

from odoo import api, fields, models


class MarketingCard(models.Model):
    """Mapping from a unique ID to a 'sharer' of a campaign. Storing state of sharing and their specific card."""
    _name = 'card.card'
    _description = 'Marketing Card'

    active = fields.Boolean('Active', default=True)
    campaign_id = fields.Many2one('card.campaign', required=True, ondelete="cascade")
    res_model = fields.Selection(related='campaign_id.res_model')
    res_id = fields.Many2oneReference('Record ID', model_field='res_model', required=True)
    image = fields.Image()
    requires_sync = fields.Boolean(help="Whether the image needs to be updated to match the campaign template.", default=True)
    share_status = fields.Selection([
        ('shared', 'Shared'),
        ('visited', 'Visited'),
    ])

    _sql_constraints = [
        ('campaign_record_unique', 'unique(campaign_id, res_id)',
         'Each record should be unique for a campaign'),
    ]

    @api.depends('res_model', 'res_id')
    def _compute_display_name(self):
        for model, cards in self.grouped('res_model').items():
            if not model:
                cards.display_name = ""
                continue
            self.env[model].browse(cards.mapped('res_id')).sudo().fetch(['display_name'])
            for card in cards:
                card.display_name = self.env[model].browse(card.res_id).sudo().display_name

    @api.depends('campaign_id')
    def _compute_res_model(self):
        """Compute the res_model once and never update it again."""
        for campaign, cards in self.grouped('campaign_id').items():
            cards.res_model = campaign.res_model

    @api.autovacuum
    def _gc_card(self):
        """Remove cards. Social networks are expected to cache the images on their side."""
        timedelta_days = self.env['ir.config_parameter'].get_param('marketing_card.card_image_cleanup_interval_days', 60)
        if not timedelta_days:
            return
        self.with_context({"active_test": False}).search([('write_date', '<=', datetime.now() - timedelta(days=timedelta_days))]).unlink()

    def _get_card_url(self):
        return self._get_path('card.jpg')

    def _get_redirect_url(self):
        return self._get_path('redirect')

    def _get_path(self, suffix):
        self.ensure_one()
        card_slug = self.env['ir.http']._slug(self)
        return f'{self.get_base_url()}/cards/{card_slug}/{suffix}'

```

## File: models\card_template.py

```python
from odoo import fields, models

# Good ratio to have a large image still small enough to stay under 5MB (common limit)
# Close to the 2:1 ratio recommended by twitter and these dimensions are recommended by meta
# https://developers.facebook.com/docs/sharing/webmasters/images/
# https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/summary-card-with-large-image
TEMPLATE_DIMENSIONS = (600, 315)
TEMPLATE_RATIO = 40 / 21


class CardCampaignTemplate(models.Model):
    _name = 'card.template'
    _description = 'Marketing Card Template'

    name = fields.Char(required=True)
    default_background = fields.Image()
    body = fields.Html(sanitize_tags=False, sanitize_attributes=False)

    primary_color = fields.Char(default='#f9f9f9', required=True)
    secondary_color = fields.Char(default='#000000', required=True)
    primary_text_color = fields.Char(default='#000000', required=True)
    secondary_text_color = fields.Char(default='#ffffff', required=True)

```

## File: models\mailing_mailing.py

```python
from odoo import _, api, exceptions, fields, models, osv


class MassMailing(models.Model):
    _name = 'mailing.mailing'
    _inherit = 'mailing.mailing'

    mailing_model_id = fields.Many2one(compute="_compute_mailing_model_id", store=True, readonly=False)
    card_requires_sync_count = fields.Integer(compute="_compute_card_requires_sync_count")
    card_campaign_id = fields.Many2one('card.campaign')

    @api.constrains('card_campaign_id', 'mailing_domain', 'mailing_model_id')
    def _check_mailing_domain(self):
        for mailing in self:
            if mailing.card_campaign_id:
                if mailing.sudo().mailing_model_id.model != mailing.card_campaign_id.res_model:
                    raise exceptions.ValidationError(_(
                        "Card Campaign Mailing should target model %(model_name)s",
                        model_name=self.env['ir.model']._get(mailing.card_campaign_id.res_model).display_name
                    ))

    @api.depends('card_campaign_id')
    def _compute_mailing_model_id(self):
        for mailing in self.filtered('card_campaign_id'):
            mailing.mailing_model_id = self.env['ir.model']._get_id(mailing.card_campaign_id.res_model)

    @api.depends('card_campaign_id')
    def _compute_card_requires_sync_count(self):
        """Check if there's any missing or outdated card."""
        self.card_requires_sync_count = 0
        # no point in updating sent mailings
        card_mailings = self.filtered(lambda mailing: mailing.card_campaign_id and mailing.state == 'draft')
        for mailing in card_mailings:
            recipients = self.env[mailing.mailing_model_real].search(self._parse_mailing_domain())
            out_of_date_count = self.env['card.card'].search_count([
                ('campaign_id', '=', mailing.card_campaign_id.id),
                ('res_id', 'in', recipients.ids),
                ('requires_sync', '=', False)
            ])
            mailing.card_requires_sync_count = len(recipients) - out_of_date_count

    def action_put_in_queue(self):
        """Detect mismatches before scheduling."""
        for mailing in self.filtered('card_campaign_id'):
            if mailing.card_requires_sync_count:
                raise exceptions.UserError(_(
                    'You should update all the cards for %(mailing)s before scheduling a mailing.',
                    mailing=mailing.display_name
                ))
        super().action_put_in_queue()

    def action_send_mail(self, res_ids=None):
        for mailing in self.filtered('card_campaign_id'):
            if mailing.card_requires_sync_count:
                raise exceptions.UserError(_(
                    'You should update all the cards for %(mailing)s before scheduling a mailing.',
                    mailing=mailing.display_name
                ))
        return super().action_send_mail(res_ids)

    def action_update_cards(self):
        """Update the cards in batches, commiting after each batch."""
        for campaign in self.filtered(lambda mailing: mailing.state == 'draft').card_campaign_id:
            campaign._update_cards(self._parse_mailing_domain(), auto_commit=True)
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'mailing.mailing',
            'res_id': self[0].id,
            'view_mode': 'form',
            'target': 'current',
        }

    def _get_recipients_domain(self):
        """Domain with an additional condition that the card must exist for the records."""
        domain = super()._get_recipients_domain()
        if self.card_campaign_id:
            res_ids = self.env['card.card'].search_fetch([('campaign_id', '=', self.card_campaign_id.id)], ['res_id']).mapped('res_id')
            domain = osv.expression.AND([domain, [('id', 'in', res_ids)]])
        return domain

```

## File: models\utm_source.py

```python
from odoo import _, api, models, exceptions


class UtmSource(models.Model):
    _inherit = 'utm.source'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_utm_source_marketing_card(self):
        utm_source_marketing_card = self.env.ref('marketing_card.utm_source_marketing_card', raise_if_not_found=False)
        if utm_source_marketing_card and utm_source_marketing_card in self:
            raise exceptions.UserError(_(
                "The UTM source '%s' cannot be deleted as it is used to promote marketing cards campaigns.",
                utm_source_marketing_card.name
            ))

```

## File: models\__init__.py

```python
from . import card_campaign
from . import card_campaign_tag
from . import card_template
from . import card_card
from . import mailing_mailing
from . import utm_source

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_card_campaign_tag_user,card.campaign.tag.user,model_card_campaign_tag,marketing_card.marketing_card_group_user,1,0,0,0
access_card_campaign_tag_manager,card.campaign.tag.manager,model_card_campaign_tag,marketing_card.marketing_card_group_manager,1,1,1,1
access_card_campaign_user,card.campaign.user,model_card_campaign,marketing_card.marketing_card_group_user,1,1,1,1
access_card_template_user,card.template.user,model_card_template,marketing_card.marketing_card_group_user,1,0,0,0
access_card_template_system,card.template.system,model_card_template,base.group_system,1,1,1,1
access_card_card_user,card.card.user,model_card_card,base.group_user,1,1,1,0
access_card_card_public,card.card.user,model_card_card,base.group_public,1,0,0,0
access_card_card_manager,card.card.manager,model_card_card,marketing_card.marketing_card_group_manager,1,1,1,1

```

## File: security\marketing_card_groups.xml

```xml
<?xml version="1.0"?>
<odoo noupdate="1">
    <record id="module_category_marketing_card" model="ir.module.category">
        <field name="name">Marketing Card</field>
        <field name="description">Helps you manage marketing card campaigns.</field>
        <field name="sequence">18</field>
    </record>

    <record id="marketing_card_group_user" model="res.groups">
        <field name="name">Marketing Card User</field>
        <field name="category_id" ref="module_category_marketing_card"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user')), (4, ref('mass_mailing.group_mass_mailing_user'))]"/>
    </record>

    <record id="marketing_card_group_manager" model="res.groups">
        <field name="name">Marketing Card Manager</field>
        <field name="category_id" ref="module_category_marketing_card"/>
        <field name="implied_ids" eval="[(4, ref('marketing_card_group_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="marketing_card_campaign_manager_own_rule" model="ir.rule">
        <field name="name">Manager may access and edit any card campaign</field>
        <field name="model_id" ref="model_card_campaign"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('marketing_card_group_manager'))]"></field>
    </record>

    <record id="marketing_card_campaign_user_owl_rule" model="ir.rule">
        <field name="name">Users may only edit their own card campaigns</field>
        <field name="model_id" ref="model_card_campaign"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
        <field name="groups" eval="[(4, ref('marketing_card_group_user'))]"></field>
        <field name="perm_create" eval="False"/>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="True"/>
        <field name="perm_unlink" eval="True"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 27a4 4 0 0 1 4-4h31v15a4 4 0 0 1-4 4H0V27Z" fill="#1AD3BB"/><path d="M15 18a4 4 0 0 1 4-4h24v11a4 4 0 0 1-4 4H15V18Z" fill="#FC868B"/><path d="M35 29H15v-6h20v6Z" fill="#1A6F66"/><path d="M28 12a4 4 0 0 1 4-4h18v8a4 4 0 0 1-4 4H28v-8Z" fill="#2EBCFA"/><path d="M43 20H28v-6h15v6Z" fill="#2D6388"/></svg>

```

## File: views\card_campaign_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="card_campaign_view_form" model="ir.ui.view">
        <field name="name">card.campaign.view.form</field>
        <field name="model">card.campaign</field>
        <field name="arch" type="xml">
            <form string="Share Campaign" class="o_card_campaign_form">
                <header invisible="not active" class="mb-2">
                    <button name="action_share" type="object" class="btn-primary">Send</button>
                    <button name="action_preview" type="object" class="btn-secondary" invisible="not preview_record_ref">Preview</button>
                </header>
                <sheet>
                <div class="oe_button_box" name="button_box">
                    <button name="action_view_mailings" type="object" icon="fa-envelope">
                        <div class="o_stat_info">
                            <span class="o_stat_text">
                                Mailings
                            </span>
                            <span class="o_stat_value">
                                <field name="mailing_count" readonly="1"/>
                            </span>
                        </div>
                    </button>
                    <button icon="fa-mouse-pointer">
                        <div class="o_stat_info">
                            <span class="o_stat_text">
                                Clicks
                            </span>
                            <span class="o_stat_value">
                                <field name="target_url_click_count" readonly="1"/>
                            </span>
                        </div>
                    </button>
                    <button name="action_view_cards" type="object" icon="fa-paper-plane">
                        <div class="o_stat_info">
                            <span class="o_stat_text">
                                Cards
                            </span>
                            <span class="o_stat_value">
                                <field name="card_count" readonly="1"/>
                            </span>
                        </div>
                    </button>
                    <button name="action_view_cards_clicked" type="object" icon="fa-eye">
                        <div class="o_stat_info">
                            <span class="o_stat_text">
                                Opened
                            </span>
                            <span class="o_stat_value">
                                <field name="card_click_count" readonly="1"/>
                            </span>
                        </div>
                    </button>
                    <button name="action_view_cards_shared" type="object" icon="fa-share">
                        <div class="o_stat_info">
                            <span class="o_stat_text">
                                Shared
                            </span>
                            <span class="o_stat_value">
                                <field name="card_share_count" readonly="1"/>
                            </span>
                        </div>
                    </button>
                </div>
                <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                <div class="oe_title">
                    <h1>
                        <field name="name" placeholder="e.g. Odoo Experience Talks"/>
                    </h1>
                </div>
                <group>
                    <group>
                        <field name="preview_record_ref" string="Recipients" placeholder="Preview on..." options="{'no_create': True}"/>
                        <field name="target_url" placeholder="Your Home Page"/>
                        <field name="res_model" invisible="1"/><!--Get the default model for field pickers-->
                        <field name="res_model" groups="base.group_no_one"/>
                        <field name="post_suggestion" placeholder="Join me at this event!" widget="text_emojis"/>
                    </group>
                    <group>
                        <field name="user_id" widget="many2one_avatar"/>
                        <field name="tag_ids" widget="many2many_tags"  options="{'color_field': 'color', 'no_create_edit': True}"/>
                    </group>
                </group>
                <notebook>
                    <page name="Card Layout" string="Card Layout">
                        <group>
                        <group>
                            <field name="content_background" widget="image" options="{'img_class': 'w-25 object-fit-contain'}"/>
                            <label for="content_header"/>
                            <div class="d-flex">
                                <field name="content_header_dyn" title="Dynamic Field?"/>
                                <field name="content_header" invisible="content_header_dyn" placeholder="e.g. Join Odoo Experience 2024"/>
                                <field name="content_header_path" invisible="not content_header_dyn" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                                <field name="content_header_color" widget="color" style="width: 40px"/>
                            </div>
                            <label for="content_sub_header"/>
                            <div class="d-flex">
                                <field name="content_sub_header_dyn"/>
                                <field name="content_sub_header" invisible="content_sub_header_dyn" placeholder="e.g. Aug 24, Brussels Expo"/>
                                <field name="content_sub_header_path" invisible="not content_sub_header_dyn" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                                <field name="content_sub_header_color" widget="color" style="width: 40px"/>
                            </div>
                            <label for="content_section"/>
                            <div class="d-flex">
                                <field name="content_section_dyn"/>
                                <field name="content_section" invisible="content_section_dyn" placeholder="e.g. Sample Talk"/>
                                <field name="content_section_path" invisible="not content_section_dyn" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                            </div>
                            <label for="content_sub_section1"/>
                            <div class="d-flex">
                                <field name="content_sub_section1_dyn"/>
                                <field name="content_sub_section1" invisible="content_sub_section1_dyn" placeholder="e.g. By Lionel Messy"/>
                                <field name="content_sub_section1_path" invisible="not content_sub_section1_dyn" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                            </div>
                             <label for="content_sub_section2"/>
                            <div class="d-flex">
                                <field name="content_sub_section2_dyn"/>
                                <field name="content_sub_section2" invisible="content_sub_section2_dyn" placeholder="e.g. CFO Chief Football Officer"/>
                                <field name="content_sub_section2_path" invisible="not content_sub_section2_dyn" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                            </div>
                            <field name="content_image1_path" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                            <field name="content_image2_path" placeholder="Select a field" widget="DynamicModelFieldSelectorChar" options="{'model': 'res_model'}"/>
                            <field name="content_button" placeholder="No button"/>
                        </group>
                        <group>
                            <field name="card_template_id" required="1" nolabel="1" colspan="2"
                            widget="selection_badge" options="{'size': 'sm'}"/>
                            <field name="image_preview" class="o_marketing_card_image_preview" nolabel="1" colspan="2"
                            invisible="not card_template_id" widget="image" options="{'size': [0, 500]}"/>
                        </group>
                        </group>
                    </page>
                    <page name="Recipient Message" string="Recipient Message">
                        <group>
                            <group>
                                <field name="request_title"/>
                                <field name="request_description" placeholder="e.g. Why people should share on their network?"/>
                            </group>
                            <group>
                                <field name="reward_target_url" placeholder='e.g. "https://www.mycompany.com/reward"'/>
                                <field name="reward_message" placeholder='e.g. "Thanks for sharing, here is your reward!"'/>
                            </group>
                        </group>
                    </page>
                </notebook>
                </sheet>
                <chatter reload_on_post="True"/>
            </form>
        </field>
    </record>

    <record id="card_campaign_view_kanban" model="ir.ui.view">
        <field name="name">card.campaign.view.kanban</field>
        <field name="model">card.campaign</field>
        <field name="arch" type="xml">
            <kanban sample="1">
                <templates>
                    <t t-name="card" class="o_marketing_card_campaign_kanban">
                        <field name="name" class="fw-bolder fs-5"/>
                        <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" class="mt-2"/>
                        <footer class="pt-0">
                            <div>
                                <a type="object" name="action_view_cards_shared" href="#" class="me-1">
                                    <span class="badge rounded-pill">
                                        <i class="fa fa-fw fa-share" aria-label="Shares" role="img" title="Shares"/>
                                        <field name="card_share_count"/>
                                    </span>
                                </a>
                                <a type="object" name="action_view_cards_clicked" href="#" class="me-1">
                                    <span class="badge rounded-pill">
                                        <i class="fa fa-fw fa-mouse-pointer" aria-label="Clicks" role="img" title="Clicks"/>
                                        <field name="target_url_click_count"/>
                                    </span>
                                </a>
                            </div>
                            <field name="user_id" class="ms-auto" widget="many2one_avatar_user"/>
                        </footer>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="card_campaign_view_tree" model="ir.ui.view">
        <field name="name">card.campaign.view.list</field>
        <field name="model">card.campaign</field>
        <field name="arch" type="xml">
            <list sample="1">
                <field name="create_date"/>
                <field name="name"/>
                <field name="user_id" widget="many2one_avatar"/>
                <field name="res_model" optional="hide"/>
                <field name="target_url" optional="hide"/>
                <field name="tag_ids" widget="many2many_tags"  options="{'color_field': 'color'}"/>
            </list>
        </field>
    </record>

    <record id="card_campaign_view_search" model="ir.ui.view">
        <field name="name">card.campaign.view.search</field>
        <field name="model">card.campaign</field>
        <field name="arch" type="xml">
            <search string="Search Share Campaign">
                <filter string="My Campaigns" name="my_campaigns" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter string="Archived" name="archived" domain="[('active', '=', False)]"/>
                <field name="name"/>
                <field name="tag_ids"/>
                <separator/>
                <group expand="0" string="Group By">
                    <filter string="Responsible" name="by_responsible" domain="[]" context="{'group_by': 'user_id'}"/>
                    <filter string="Tags" name="by_tags" domain="[]" context="{'group_by': 'tag_ids'}"/>
                </group>
            </search>
        </field>
    </record>


    <record id="card_campaign_action" model="ir.actions.act_window">
        <field name="name">Card Campaign</field>
        <field name="res_model">card.campaign</field>
        <field name="search_view_id" ref="card_campaign_view_search"/>
        <field name="view_mode">list,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">Create a Sharing Campaign!</p>
            <p>Prepare a design and some content and let your community spread the word!</p>
        </field>
    </record>

</odoo>

```

## File: views\card_card_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
<!--Notable pitfalls-->
<!--Unpatched wkhtmltoimage uses an old version of webkit, meaning things like standard flexbox
    are not supported and -webkit flags should be used.-->
<!--Your local fonts may diverge from production fonts, make sure the designs work as intended
    in their final production environment.-->
<template id="common_template">
    <html>
        <head>
            <meta charset="utf-8"/>
            <style>
            body {
                @charset "UTF-8";
                padding: 35px;
                font-family: Roboto, Ubuntu, "Noto Sans", Arial, sans-serif, "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
                font-size: 1em;
                background: <t t-out="card_campaign.card_template_id.primary_color"/>;
                color: <t t-out="card_campaign.card_template_id.primary_text_color"/>;
                background-size: cover;
                background-attachment: fixed;
                display: -webkit-box;
                display: flex;
                -webkit-box-orient: vertical;
                flex-direction: column;
                -webkit-box-pack: justify;
                justify-content: space-around;
            }
            .button_background {
                background: <t t-out="card_campaign.card_template_id.secondary_color"/>;
                color: <t t-out="card_campaign.card_template_id.secondary_text_color"/>;
            }
            .header {
                line-height: 1.2em;
            }
            .header, .subheader, .cta_button h1 {
                display: -webkit-box;
                -webkit-box-orient: vertical;
            }
            .header, .subheader, .cta_button h1, .cta_button h2 {
                margin: 0.1em;
            }
            .subheader, .cta_button h1{
                line-height: 1.2em;
            }
            .cta_button {
                text-align: center;
                width: fit-content;
                min-width: 30%;
                max-width: 70%;
                padding: 0.2rem 0.2rem;
            }
            .rounded_button {
                border-radius: 50px;
            }
            .rounded_rectangle_button {
                border-radius: 8px;
            }
            .footer {
                width: 100%;
                display: -webkit-box;
                display: flex;
                -webkit-box-orient: horizontal;
                flex-direction: row;
                -webkit-box-pack: justify;
                justify-content: space-between;
            }
            .footer &gt; div {
                height: 100%;
            }
            .profile_section {
                margin-right: 0.25rem;
                font-size: 1.25rem;
                display: -webkit-box;
                display: flex;
                -webkit-box-orient: horizontal;
                flex-direction: row;
                -webkit-box-align: center;
                align-items: center;
            }
            .text_section {
                display: -webkit-box;
                display: flex;
                -webkit-box-orient: vertical;
                flex-direction: column;
                -webkit-box-align: start;
                align-items: flex-start;
                align-content: flex-start;
                -webkit-box-pack: center;
                justify-content: center;
            }
            .profile_section .text_section .text_subsections {
                font-size: 1rem;
            }
            </style>
        </head>
        <t t-set="values" t-value="card_campaign._get_card_element_values(object)"/>
        <body t-attf-style="background-image: url('data:image/png;base64,{{card_campaign.content_background or card_campaign.card_template_id.default_background}}');">
        </body>
    </html>
</template>

<template id="template_1" inherit_id="common_template" primary="True">
    <xpath expr="//html/head/style" position="after">
    <style>
        body {
            -webkit-box-align: center;
            align-items: center;
        }
        .center_column {
            text-align: center;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: vertical;
            flex-direction: column;
            -webkit-box-align: center;
            align-items: center;
            -webkit-box-pack: justify;
            justify-content: space-around;
            line-height: 2rem;
            height: 60%;
            width: 100%;
        }
        .center_column .cta_button {
            min-width: 50%;
            max-width: 70%;
        }
        .footer {
            -webkit-box-align: center;
            align-items: center;
            height: 30%;
        }
        .profile_section img {
            object-fit: cover;
            border-radius: 50px;
        }
        .profile_section .text_section span {
            display: block;
        }
        .logo {
            max-height: 40px;
            width: auto;
        }
    </style>
    </xpath>
    <xpath expr="//html/body" position="inside">
        <div class="center_column">
            <div class="title">
                <h1 class="header" t-out="values['header']" t-att-style="'color: %s;' % card_campaign.content_header_color"></h1>
                <h2 class="subheader" t-out="values['sub_header']" t-att-style="'color: %s;' % card_campaign.content_sub_header_color"></h2>
            </div>
            <div class="button_background rounded_button cta_button" t-if="card_campaign.content_button">
                <h1 t-out="card_campaign.content_button">Button</h1>
            </div>
        </div>
        <div class="footer">
            <div class="profile_section">
                <img
                t-attf-src="data:image/png;base64,{{values['image1']}}"
                alt="Profile Picture"
                width="64px"
                height="64px"
                style="margin-right: 10px"
                t-if="values['image1']"
                />
                <div t-else=""></div>
                <div class="text_section">
                    <span t-out="values['section']"/>
                    <div class="text_subsections">
                        <span t-out="values['sub_section1']"/>
                        <span t-out="values['sub_section2']"/>
                    </div>
                </div>
            </div>
            <img
                t-attf-src="data:image/png;base64,{{values['image2']}}"
                alt="Profile Picture"
                height="auto"
                class="logo"
                t-if="values['image2']"
            />
        </div>
    </xpath>
</template>

<template id="template_2" inherit_id="common_template" primary="True">
    <xpath expr="//html/head/style" position="after">
    <style>
        body {
            -webkit-box-align: center;
            align-items: center;
            justify-content: space-between;
        }
        .title {
            line-height: 2rem;
            width: 100%;
        }
        .footer {
            -webkit-box-align: center;
            align-items: center;
            height: 30%;
        }
        .cta_button {
            text-align: center;
            width: fit-content;
            max-height: 3rem;
        }
        .profile_section img {
            object-fit: cover;
            border-radius: 50px;
        }
        .profile_section .text_section {
            margin-left: 0.25rem;
        }
        .profile_section .text_section span {
            display: block;
        }
        .logo_line {
            width: 100%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: horizontal;
            flex-direction: row;
            -webkit-box-align: center;
            align-items: center;
            align-content: center;
            -webkit-box-pack: end;
            justify-content: flex-end;
        }
        .logo {
            margin: 0.1rem;
            align-self: flex-end;
            max-height: 40px;
            width: auto;
        }
    </style>
    </xpath>
    <xpath expr="//html/body" position="inside">
        <div class="logo_line">
            <img
                t-attf-src="data:image/png;base64,{{values['image2']}}"
                alt="Profile Picture"
                class="logo"
                t-if="values['image2']"
            />
        </div>
        <div class="title">
            <h1 class="header" t-out="values['header']" t-att-style="'color: %s;' % card_campaign.content_header_color"/>
            <h2 class="subheader" t-out="values['sub_header']" t-att-style="'color: %s;' % card_campaign.content_sub_header_color"/>
        </div>
        <div class="footer">
            <div class="profile_section">
                <img
                    t-attf-src="data:image/png;base64,{{values['image1']}}"
                    alt="Profile Picture"
                    width="64px"
                    height="64px"
                    t-if="values['image1']"
                />
                <div t-else=""></div>
                <div class="text_section">
                    <span t-out="values['section']"/>
                    <div class="text_subsections">
                        <span t-out="values['sub_section1']"/>
                        <span t-out="values['sub_section2']"/>
                    </div>
                </div>
            </div>
            <div class="button_background rounded_button cta_button" t-if="card_campaign.content_button">
                <h1 t-out="card_campaign.content_button">button text</h1>
            </div>
        </div>
    </xpath>
</template>

<template id="template_3" inherit_id="common_template" primary="True">
    <xpath expr="//html/head/style" position="after">
    <style>
        body {
            -webkit-box-pack: justify;
            justify-content: space-between;
        }
        .title {
            line-height: 2rem;
            width: 80%;
        }
        .footer {
            -webkit-box-align: end;
            align-items: flex-end;
            height: 50%;
        }
        .footer &gt; div {
            min-width: 50%;
        }
        .cta_button {
            margin-top: 20px;
        }
        .profile_image {
            object-fit: cover;
            border-radius: 50px;
        }
        .cta_section {
            max-width: 75%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: vertical;
            flex-direction: column;
            -webkit-box-align: center;
            align-items: center;
            -webkit-box-pack: justify;
            justify-content: space-between;
        }
        .cta_section .text_section {
            width: 100%;
            font-size: 1.25rem;
        }
        .cta_section .text_subsections {
            font-size: 0.85rem;
        }
        .logo {
            position: absolute;
            right: 35px;
            top: 35px;
            max-height: 40px;
            width: auto;
        }
    </style>
    </xpath>
    <xpath expr="//html/body" position="inside">
        <img
            t-attf-src="data:image/png;base64,{{values['image2']}}"
            alt="Profile Picture"
            class="logo"
            t-if="values['image2']"
        />
        <div class="title">
            <h2 class="subheader" t-out="values['sub_header']" t-att-style="'color: %s;' % card_campaign.content_sub_header_color"></h2>
            <h1 class="header" t-out="values['header']" t-att-style="'color: %s;' % card_campaign.content_header_color"></h1>
        </div>
        <div class="footer">
            <div>
                <div class="text_section">
                    <span t-out="values['section']"/>
                    <div class="text_subsections">
                        <span t-out="values['sub_section1']"/>
                        <t t-if="values['sub_section1'] and values['sub_section2']">-</t>
                        <span t-out="values['sub_section2']"/>
                    </div>
                </div>
                <div class="button_background rounded_rectangle_button cta_button" t-if="card_campaign.content_button">
                    <h2 t-out="card_campaign.content_button">button text</h2>
                </div>
            </div>
            <img
                class="profile_image"
                t-attf-src="data:image/png;base64,{{values['image1']}}"
                alt="Profile Picture"
                width="106px"
                height="106px"
                t-if="values['image1']"
            />
        </div>
    </xpath>
</template>


<template id="template_4" inherit_id="common_template" primary="True">
    <xpath expr="//html/head/style" position="after">
    <style>
        body {
            justify-content: space-between;
        }
        .header {
            -webkit-line-clamp: 3;
            max-height: 4.5em;
            line-height: 1.2em;
        }
        .title {
            line-height: 2rem;
            width: 80%;
        }
        .footer {
            height: 50%;
        }
        .cta_button {
            text-align: center;
            max-height: 3rem;
        }
        .profile_image {
            object-fit: cover;
            border-radius: 50px;
        }
        .cta_section {
            max-width: 75%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: vertical;
            flex-direction: column;
            -webkit-box-align: center;
            align-items: center;
        }
        .text_section {
            width: 55%;
            font-size: 1.25rem;
        }
        .text_subsections {
            font-size: 0.85rem;
        }
        .text_subsections span {
            text-wrap: nowrap;
            white-space: nowrap;
        }
        .cta_button {
            width: fit-content;
            margin-top: 30px;
        }
        .logo_line {
            width: 100%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: horizontal;
            flex-direction: row;
            -webkit-box-align: center;
            align-items: center;
            align-content: center;
            -webkit-box-pack: start;
            justify-content: flex-start;
        }
        .logo {
            max-height: 40px;
            width: auto;
        }
        .profile_image {
            background-size: cover;
            position: absolute;
            right: 35px;
            top: 35px;
        }
    </style>
    </xpath>
    <xpath expr="//html/body" position="inside">
        <div class="logo_line">
            <img
                t-attf-src="data:image/png;base64,{{values['image2']}}"
                alt="Profile Picture"
                class="logo"
                t-if="values['image2']"
            />
        </div>
        <img
            class="profile_image"
            t-attf-src="data:image/png;base64,{{values['image1']}}"
            alt="Profile Picture"
            width="106px"
            height="106px"
            t-if="values['image1']"
        />
        <div class="title">
            <h2 class="subheader" t-out="values['sub_header']" t-att-style="'color: %s;' % card_campaign.content_sub_header_color"></h2>
            <h1 class="header" t-out="values['header']" t-att-style="'color: %s;' % card_campaign.content_header_color"></h1>
        </div>
        <div class="footer">
            <div class="text_section">
                <span t-out="values['section']"/>
                <div class="text_subsections">
                    <span t-out="values['sub_section1']"/>
                    <t t-if="values['sub_section1'] and values['sub_section2']">-</t>
                    <span t-out="values['sub_section2']"/>
                </div>
            </div>
            <div class="button_background rounded_button cta_button" t-if="card_campaign.content_button">
                <h1 t-out="card_campaign.content_button">button text</h1>
            </div>
        </div>
    </xpath>
</template>

<template id="template_5" inherit_id="common_template" primary="True">
    <xpath expr="//html/head/style" position="after">
    <style>
        body {
            height: 100%;
        }
        .footer {
            margin-top: 5%;
            height: 40%;
        }
        .cta_button {
            text-align: center;
        }
        .profile_image {
            object-fit: cover;
            border-radius: 50px;
        }
        .cta_section {
            max-width: 75%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: vertical;
            flex-direction: column;
            -webkit-box-align: center;
            align-items: center;
        }
        .text_section {
            font-size: 1.25rem;
        }
        .text_subsections {
            height: 100%;
            width: 100%;
            font-size: 0.85rem;
        }
        .cta_button {
            width: fit-content;
            min-width: 30%;
            max-width: 50%;
        }
        .logo_line {
            width: 100%;
            display: -webkit-box;
            display: flex;
            -webkit-box-orient: horizontal;
            flex-direction: row;
            -webkit-box-align: center;
            align-items: center;
            align-content: center;
            -webkit-box-pack: justify;
            justify-content: space-between;
        }
        .logo {
            max-height: 40px;
            width: auto;
        }
    </style>
    </xpath>
    <xpath expr="//html/body" position="inside">
        <div class="logo_line">
            <img
                class="profile_image"
                t-attf-src="data:image/png;base64,{{values['image1']}}"
                alt="Profile Picture"
                width="60px"
                height="60px"
                t-if="values['image1']"
            />
            <!--spacing filler in case there is no image-->
            <div t-if="not values['image1']"></div>
            <div class="button_background rounded_button cta_button" t-if="card_campaign.content_button">
                <h1 t-out="card_campaign.content_button">button text</h1>
            </div>
        </div>
        <div class="title">
            <h2 class="subheader" t-out="values['sub_header']" t-att-style="'color: %s;' % card_campaign.content_sub_header_color"></h2>
            <h1 class="header" t-out="values['header']" t-att-style="'color: %s;' % card_campaign.content_header_color"></h1>
        </div>
        <div class="footer">
            <div class="text_section">
                <span t-out="values['section']"/>
                <div class="text_subsections">
                    <span t-out="values['sub_section1']"/>
                    <t t-if="values['sub_section1'] and values['sub_section2']">-</t>
                    <span t-out="values['sub_section2']"/>
                </div>
            </div>
            <img
                t-attf-src="data:image/png;base64,{{values['image2']}}"
                alt="Profile Picture"
                class="logo"
                t-if="values['image2']"
            />
        </div>
    </xpath>
</template>
</odoo>

```

## File: views\card_card_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="card_card_view_list" model="ir.ui.view">
        <field name="name">card.card.view.list</field>
        <field name="model">card.card</field>
        <field name="arch" type="xml">
            <list string="Share Card">
                <field name="create_date"/>
                <field name="create_uid"/>
                <field name="display_name"/>
                <field name="res_model" optional="hidden"/>
                <field name="campaign_id" optional="hidden"/>
                <field name="share_status"/>
            </list>
        </field>
    </record>


    <record id="card_card_view_search" model="ir.ui.view">
        <field name="name">card.card.view.search</field>
        <field name="model">card.card</field>
        <field name="arch" type="xml">
            <search string="Search Card">
                <group expand="0" string="Filter By">
                    <field name="share_status"/>
                    <filter string="Shared" name="filter_shared" domain="[('share_status', '=', 'shared')]"/>
                    <filter string="Visited" name="filter_visited" domain="[('share_status', '=', 'visited')]"/>
                </group>
                <group expand="0" string="Group By">
                    <field name="campaign_id"/>
                    <filter string="Campaign" name="by_campaign" context="{'group_by': 'campaign_id'}"/>
                </group>
            </search>
        </field>
    </record>


    <record id="cards_card_action" model="ir.actions.act_window">
        <field name="name">Card</field>
        <field name="res_model">card.card</field>
        <field name="search_view_id" ref="card_card_view_search"></field>
        <field name="context">{'search_default_by_campaign': True, 'search_default_filter_visited': True}</field>
        <field name="view_mode">list</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a Card Campaign to send cards to your partners
            </p>
        </field>
    </record>
</odoo>

```

## File: views\card_frontend_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="card_campaign_preview" name="Share campaign">
        <t t-call="web.frontend_layout">
            <t t-set="title" t-value="card.display_name"/>
            <t t-set="body_classname" t-value="'o_card_campaign_preview_frontend'"/>
            <t t-set="no_header" t-value="1"/>
            <t t-set="no_footer" t-value="1"/>
            <t t-set="no_livechat" t-value="1"/>
            <t t-set="share_url" t-value="card._get_redirect_url()"/>
            <t t-set="share_url_quoted" t-value="quote(share_url)"/>
            <div class="d-flex flex-column align-items-center justify-content-center h-100">
                <div class="mb-5 d-flex flex-column align-items-center justify-content-center">
                    <h1 t-out="campaign.request_title">Share with your community!</h1>
                    <p t-out="campaign.request_description or None" class="mb-5"/>
                    <a t-att-href="card._get_card_url()" class="border border-3"
                       style="max-width: 80%; height: auto; max-height: 80%;">
                        <img t-att-src="card._get_card_url()" class="w-100"/>
                    </a>
                    <a t-att-href="share_url" target="_blank"><small>Where does this link to?</small></a>
                </div>
                <div class="d-flex flex-column align-items-center">
                    <h4>Select where to share</h4>
                    <div class="text-start o_no_link_popover">
                        <div id="url-div">
                            <a class="fa fa-2x fa-facebook rounded m-2" t-attf-href="https://www.facebook.com/sharer/sharer.php?u={{share_url_quoted}}" target="_blank"/>
                            <a class="fa fa-2x fa-twitter rounded m-2" t-attf-href="https://twitter.com/intent/tweet?url={{share_url_quoted}}{{'&amp;text=%s' % quote(campaign.post_suggestion) if campaign.post_suggestion else ''}}" target="_blank"/>
                            <a class="fa fa-2x fa-linkedin rounded m-2" t-attf-href="https://www.linkedin.com/sharing/share-offsite/?url={{share_url_quoted}}" target="_blank"/>
                            <a class="fa fa-2x fa-whatsapp rounded m-2" t-attf-href="https://wa.me/?text={{share_url_quoted}}" target="_blank"/>
                            <a class="fa fa-2x fa-pinterest rounded m-2" t-attf-href="https://pinterest.com/pin/create/button/?url={{share_url_quoted}}" target="_blank"/>
                        </div>
                    </div>
                </div>
                <br/>
                <div id="thanks-section" t-if="not is_html_empty(campaign.reward_message) or campaign.reward_target_url" style="width: 50%;"
                t-attf-class="alert alert-info d-flex flex-column align-items-center {{'d-none' if not card.share_status == 'shared' else ''}}">
                    <p t-if="campaign.reward_message" t-out="campaign.reward_message"/>
                    <p t-if="campaign.reward_target_url"><a t-att-href="campaign.reward_target_url">Click here for your reward!</a></p>
                </div>
            </div>
            <footer class="d-flex flex-column align-items-center fixed-bottom">
                <span>Powered By <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo</a></span>
            </footer>
            <script>
                document.querySelectorAll("#url-div a").forEach((anchor) =&gt; {
                    anchor.addEventListener("click", (event) =&gt; {
                        event.preventDefault();
                        event.stopPropagation();
                        window.open(
                            event.target.href,
                            event.target.target,
                            "menubar=no,toolbar=no,resizable=yes,scrollbars=yes,height=550,width=600",
                        );
                        const thanksSection = document.getElementById("thanks-section")
                        if (thanksSection) {
                            thanksSection.classList.remove("d-none");
                        }
                    });
                });
            </script>
        </t>
    </template>

    <template id="card_campaign_crawler" name="Share campaign Crawler View">
        <t t-call="web.frontend_layout">
            <t t-set="head">
                <meta name="twitter:card" content="summary_large_image"/>
                <meta property="og:image" t-att-content="image_url"/>
                <meta property="og:image:alt" t-attf-content="{{ target_name }}"/>
                <meta property="og:title" t-att-content="target_name"/>
                <meta property="og:type" content="website"/>
                <meta property="og:description" t-att-content="post_suggestion"/>
            </t>
            <t t-set="title" t-value="target_name"/>
            <t t-set="no_header" t-value="1"/>
            <t t-set="no_footer" t-value="1"/>
        </t>
    </template>
</odoo>

```

## File: views\card_menus.xml

```xml
<?xml version="1.0"?>
<odoo>
    <menuitem name="Marketing Card"
        id="card_menu"
        web_icon="marketing_card,static/description/icon.png"
        sequence="270"
        groups="marketing_card.marketing_card_group_user"/>

    <menuitem name="Campaigns"
        id="card_campaign_menu"
        parent="card_menu"
        sequence="0"
        action="card_campaign_action"
        groups="marketing_card.marketing_card_group_user"/>

    <!-- Technical -->
    <menuitem name="Marketing Card"
        id="marketing_card_menu_technical"
        parent="base.menu_custom"
        groups="marketing_card.marketing_card_group_manager,base.group_no_one"
        sequence="4"/>

    <menuitem name="Card Template"
        id="cards_template_menu"
        parent="marketing_card.marketing_card_menu_technical"
        groups="marketing_card.marketing_card_group_manager,base.group_no_one"
        action="card_template_action"/>
</odoo>

```

## File: views\card_template_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="card_template_view_form" model="ir.ui.view">
        <field name="name">card.template.view.form</field>
        <field name="model">card.template</field>
        <field name="arch" type="xml">
            <form string="Share Template">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="default_background"/>
                        </group>
                        <group>
                            <field name="primary_color" widget="color"/>
                            <field name="secondary_color" widget="color"/>
                            <field name="primary_text_color" widget="color"/>
                            <field name="secondary_text_color" widget="color"/>
                        </group>
                        <field name="body" widget="code" options="{'mode': 'xml'}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="card_template_action" model="ir.actions.act_window">
        <field name="name">Card Template</field>
        <field name="res_model">card.template</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a design to use in Card Campaigns
            </p>
        </field>
    </record>

</odoo>

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_mailing_view_form_inherit_marketing_card" model="ir.ui.view">
        <field name="name">mailing.mailing.view.form.inherit.marketing.card</field>
        <field name="model">mailing.mailing</field>
        <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="attributes">
                <attribute name="invisible" separator=" or " add="card_requires_sync_count"/>
            </xpath>
            <xpath expr="//header" position="after">
                <header invisible="not card_requires_sync_count">
                    <button type="object" name="action_update_cards" class="btn-primary"
                    confirm="Are you sure you want to update all cards of the campaign?"
                    confirm-title="Confirm Cards Update" confirm-label="Update Cards">Update <field name="card_requires_sync_count"/> Cards</button>
                </header>
            </xpath>
            <xpath expr="//header//button[@name='action_set_favorite']" position="attributes">
                <attribute name="invisible" separator=" or " add="card_campaign_id"/>
            </xpath>
            <xpath expr="//field[@name='mailing_model_id']" position="attributes">
                <attribute name="readonly" separator=" or " add="card_campaign_id"/>
            </xpath>
            <xpath expr="//label[@for='mailing_model_id']" position="before">
                <field name="card_campaign_id" invisible="not card_campaign_id" readonly="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_templates.xml

```xml
<odoo>
    <data>
        <template id="robots" inherit_id="website.robots">
            <xpath expr="//t[@t-out='request.website.sudo().robots_txt']" position="before">
                User-agent: *
                Allow: /cards/
            </xpath>
        </template>
    </data>
</odoo>

```

## File: wizards\mail_compose_message.py

```python
import re
from markupsafe import Markup

from odoo import api, models

CARD_IMAGE_URL = re.compile(r'src=".*?/web/image/card.campaign/[0-9]+/image_preview"')
CARD_PREVIEW_URL = re.compile(r'href=".*?/cards/[0-9]+/preview"')


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    def _prepare_mail_values_dynamic(self, res_ids):
        """Replace generic card urls with the specific res_id url."""
        mail_values_all = super()._prepare_mail_values_dynamic(res_ids)

        if campaign := self.mass_mailing_id.card_campaign_id:
            card_from_res_id = self.env['card.card'].search_fetch(
                [('campaign_id', '=', campaign.id), ('res_id', 'in', res_ids)],
                ['res_id'],
            ).grouped('res_id')

            processed_bodies = self._process_generic_card_url_body([
                (card_from_res_id[res_id], mail_values.get('body_html'))
                for res_id, mail_values in mail_values_all.items()
            ])
            for mail_values, body in zip(mail_values_all.values(), processed_bodies):
                if body is not None:
                    mail_values['body_html'] = body

        return mail_values_all

    @api.model
    def _process_generic_card_url_body(self, card_body_pairs: list[tuple[models.Model, str]]) -> list[str]:
        """Update the bodies with the specific card url for that res_id and create a card.

        example: (1, "/cards/9/preview") -> (1, "/cards/9/1/abchashtoken/preview") + new card as side-effect

        :return: processed bodies in the order they were received
        """
        bodies = []
        for card, body in card_body_pairs:
            if body:
                def fill_card_image_url(match):
                    return Markup('src="{}"').format(card._get_path('card.jpg'))

                def fill_card_preview_url(match):
                    return Markup('href="{}"').format(card._get_path('preview'))

                body_is_markup = False
                if isinstance(body, Markup):
                    body_is_markup = True
                body = re.sub(CARD_IMAGE_URL, fill_card_image_url, body)
                body = re.sub(CARD_PREVIEW_URL, fill_card_preview_url, body)
                if body_is_markup:
                    body = Markup(body)
            bodies.append(body)
        return bodies

```

## File: wizards\__init__.py

```python
from . import mail_compose_message

```

